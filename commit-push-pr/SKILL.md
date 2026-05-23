---
name: commit-push-pr
description: >
  Reviews uncommitted git changes, runs available project validation (tests, lint, typecheck), flags hazards, drafts a Conventional Commit message AND a PR package (title, summary, validation notes, risk notes), then waits for explicit approval before staging, committing, pushing, or opening a pull request. Uses a two-stage approval gate: approve commit first, then approve push + PR. Refuses force-push and requires explicit confirmation before pushing to main or master. Use this skill whenever the user says "commit-push-pr", "commit and push", "ship this", "open a PR for this", "prepare a PR", "draft a PR", "wrap this up and push it", "I'm ready to push", or describes finished work and wants it landed. Always use this skill before running git push or opening a PR — even if the changes look simple. Pairs with commit-safe (which stops at the commit; this skill extends to push + PR).
---

# commit-push-pr

Extends `commit-safe` with validation, push, and PR drafting. Two approval gates: one before commit, one before push + PR. Never force-pushes. Refuses to push to main/master without explicit confirmation.

---

## Hard rules (read first)

1. **Never run `git add`, `git commit`, `git push`, or open a PR without explicit approval from the user in chat.** "Go ahead", "yes", "commit it", "push it", or "approved" counts. Silence does not.
2. **Two approval gates, not one.** Gate 1: approve the commit message + staged files → run `git add` + `git commit`. Gate 2: approve the PR package + push target → run `git push` + open PR. Don't combine them.
3. **Never force-push.** Not with `--force`, not with `--force-with-lease`, not ever. If the user asks, refuse and explain that this skill doesn't do force-push.
4. **Pushing to `main` or `master` requires explicit confirmation.** Surface the branch name in the Branch section. If it's main/master, ask: "This will push directly to main. Confirm?"
5. **Never invent validation commands.** Only run commands the project defines (in `package.json` scripts, `pyproject.toml`, `Makefile`, etc.). If none exist, say so — don't guess.
6. **One logical change per commit.** Mixed diffs get a split recommendation, not a merged message.

---

## Step 1: Inspect the repo

Run these commands in order using bash. Inject their output into context before drafting anything.

```bash
git status --short
git branch --show-current
git diff --stat
git diff
git diff --staged
```

Then check upstream status — this handles new branches, missing upstream, and commits-ahead:

```bash
BRANCH=$(git branch --show-current)
if git rev-parse --verify "origin/$BRANCH" >/dev/null 2>&1; then
  git log "origin/$BRANCH..HEAD" --oneline
else
  echo "NEW_BRANCH: no upstream yet — first push will use 'git push -u origin $BRANCH'"
fi
```

If `git status --short` returns nothing AND the upstream check shows no new commits AND it's not a new branch, stop. Tell the user there's nothing to commit or push.

---

## Step 2: Detect project type and run validation

Check for these files to figure out what validation to run:

| File found | Try these commands (only if defined) |
|---|---|
| `package.json` | `npm test`, `npm run lint`, `npm run typecheck`, `npm run build` |
| `pyproject.toml` or `setup.py` | `pytest`, `ruff check`, `mypy` (only if configured) |
| `Makefile` | `make test`, `make lint` (only if targets exist) |
| `Cargo.toml` | `cargo test`, `cargo clippy` |

**Rules:**
- For npm scripts: check `package.json` `scripts` block first. Don't run what isn't defined.
- For Python: only run tools if config exists (`pyproject.toml` mentions them, or config files like `.ruff.toml` are present).
- For Make: check `make -n <target>` doesn't error before running.
- If nothing is defined, write "No project validation commands found" in the Validation section. Don't apologize for it.

Capture pass/fail for each command. If something fails, surface it in the Safety check — don't bury it.

---

## Step 3: Identify the change

Look at the diff and decide:

- **One clean logical change** → proceed to draft a single commit + PR
- **Multiple unrelated changes** → recommend a split before drafting anything else. Stop and ask the user how to proceed.
- **Accidental / generated / debug / secret files in the diff** → flag immediately, recommend removing before proceeding

---

## Step 4: Hazard check

Scan the diff for any of these. List them in the Safety check section.

| Hazard | What to look for |
|---|---|
| Secrets | API keys, tokens, passwords, private keys, OAuth client secrets |
| `.env` files | Any `.env`, `.env.local`, `.env.production`, etc. |
| Local config | `config.local.*`, IDE settings (`.vscode/`, `.idea/`), OS files (`.DS_Store`, `Thumbs.db`) |
| Generated artifacts | `dist/`, `build/`, `node_modules/`, `__pycache__/`, `.next/`, compiled binaries |
| Debug code | `console.log`, `print()` for debugging, commented-out code blocks |
| Unrelated churn | Whitespace-only changes in files unrelated to the main work |
| Large files | Anything over ~1 MB that isn't obviously intentional |
| Local data | Exported CSVs, cached API responses, database dumps, fixture data not meant for the repo |
| Personal tool config | Editor/notebook/vault config folders that shouldn't be shared (e.g., `.obsidian/`, `.jupyter/`) |
| Push target | Pushing to `main` or `master` — flag for explicit confirmation |
| Force-push request | Refuse outright |
| Failed validation | Tests or lint failed in Step 2 |

If nothing suspicious: write "No obvious hazards found."

---

## Step 5: Draft the Conventional Commit message

Format:

```text
type(scope): imperative summary

Optional body explaining why the change was made, tradeoffs,
or anything important for future maintainers.
```

### Allowed types

| Type | Use for |
|---|---|
| `feat` | New user-facing functionality |
| `fix` | Bug fix |
| `docs` | Documentation-only change |
| `refactor` | Code restructuring without behavior change |
| `test` | Test additions or corrections |
| `chore` | Maintenance work (dependency bumps, tooling) |
| `style` | Formatting-only changes |
| `perf` | Performance improvement |
| `build` | Build system or dependency changes |
| `ci` | CI/CD changes |

### Rules

- Imperative mood: "add", "fix", "update", "remove" — not "added", "fixes", "updating"
- Subject specific and outcome-focused
- Prefer 50 characters or fewer when practical
- Do not end the subject with a period
- Use the body to explain **why**, not **what**
- Mention breaking changes clearly with `BREAKING CHANGE:` in the footer

### Scope conventions

Use the component, module, or package name as the scope. Examples:
- `feat(auth): add OAuth token refresh`
- `fix(api): correct timezone offset in scheduling`
- `chore(deps): bump SDK to latest`

If the scope isn't obvious from the diff, ask before guessing.

### Edge cases — fetch the spec

If the change involves any of: breaking changes, multi-scope commits, footer conventions (`Refs:`, `Closes:`, `BREAKING CHANGE:`), or anything unusual — fetch the current spec before drafting:

- Fetch `https://www.conventionalcommits.org/en/v1.0.0/`
- If the fetched spec conflicts with this file, follow the spec.

---

## Step 6: Draft the PR package

**PR title:** Same as the commit subject (`type(scope): summary`). If multiple commits, use a higher-level summary.

**PR body template:**

```markdown
## Summary
- [Bullet: what this PR does and why]
- [Bullet: any context a reviewer needs]

## Validation
- [What was run, what passed/failed]
- [Manual verification steps, if any]

## Risk / Notes
- [What could break]
- [What to watch for after merge]
- [Migration notes, if any]
```

Keep bullets concise — one line each when possible. The diff shows the what; the PR explains the why and the risk.

---

## Step 7: Output (before any git action)

Return exactly these seven sections, in this order, with these headers.

### Branch
Current branch name. If it's `main` or `master`, add a warning line: "⚠️ This will push directly to main. Confirm in your approval."

### Change summary
One paragraph. What changed and why, based on the diff.

### Validation
What was run, what passed, what failed. Or "No project validation commands found."

### Safety check
List every hazard found, or "No obvious hazards found."

### Proposed commit
```text
type(scope): imperative summary

Optional body.
```

### Proposed PR
```markdown
## Summary
- 

## Validation
- 

## Risk / Notes
- 
```

### Next step
Explicit ask: "Approve the commit message + staged files? Once committed, I'll show the push + PR plan for a second approval."

---

## Step 8: Gate 1 — Commit approval

Wait for the user to respond:

- **"Approved" / "go ahead" / "commit it" / "yes"** → run `git add <specific files>` and `git commit -m "..."` with the approved message. Then proceed to Step 9.
- **"Change the commit message to..."** → redraft, show again, wait for approval.
- **"Split it"** → stop here. Recommend the split groups. Don't commit anything until the user restarts with a single clean change.
- **"Commit only, don't push yet"** → run the commit, then stop. Skip Step 9. Tell the user to run the skill again when ready to push.
- **Ambiguous ("looks good", "ok")** → ask: "Commit and proceed to push approval, or commit only?"

---

## Step 9: Gate 2 — Push + PR approval

After the commit is made, show:

```
Commit made: <hash> <subject>

Ready to push to <remote>/<branch> and open a PR with the title and body shown above.
Approve?
```

Wait for the user:

- **"Approved" / "push it" / "yes"** → push using the logic below. Then open the PR.
- **"Don't push yet"** → stop. Commit stays local. Tell the user how to push manually when ready.
- **"Change the PR body to..."** → redraft, show again, wait.

### Pushing

Check whether the branch has upstream tracking set:

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{upstream} 2>/dev/null
```

- If a tracking branch is returned → run `git push` (no force, no force-with-lease).
- If the command fails (no upstream) → run `git push -u origin <branch-name>` to set tracking and push.

If push fails with "non-fast-forward" or "rejected", do NOT retry with force flags. Tell the user the remote has diverged and recommend they pull and rebase manually before re-running this skill: `git pull --rebase origin <branch>`. Then stop.

If push fails for any other reason, stop and report the error. Do not retry with force flags.

### Opening the PR

Check if `gh` CLI is available AND authenticated:

```bash
which gh && gh auth status >/dev/null 2>&1
```

- If both succeed: run `gh pr create --title "..." --body "..."` with the approved title and body.
- If either fails: print the PR title and body in a copyable block, plus the GitHub URL for the branch (`https://github.com/<owner>/<repo>/compare/<branch>?expand=1`). Tell the user to paste and create manually.

---

## Push safety

Refuse all of these — no exceptions:

- `git push --force` or `git push -f`
- `git push --force-with-lease`
- Pushing without explicit approval
- Pushing to main/master without the explicit branch-name confirmation in approval

If the user asks for any of these, explain that this skill doesn't do force-push or unconfirmed main-push, and suggest they run the command manually if they're sure.

---

## Anti-patterns (do not do)

- Don't run `git add .` — always stage specific files
- Don't auto-amend a previous commit
- Don't force-push, ever
- Don't combine Gate 1 and Gate 2 approvals into a single ask
- Don't invent validation commands the project hasn't defined
- Don't bury hazards (failed tests, .env files, main-branch push) in the recommendation section — surface them in Safety check
- Don't draft the commit or PR before running inspection and validation
- Don't open a PR if the push failed — stop and report
