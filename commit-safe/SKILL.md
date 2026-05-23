---
name: commit-safe
description: >
  Reviews uncommitted git changes, flags hazards (secrets, .env files, generated artifacts, debug code, unrelated churn), drafts a Conventional Commit message, and waits for explicit approval before running any git command. Use this skill whenever the user says "review my changes", "draft a commit message", "commit-safe", "what should I commit", "is this safe to commit", "check my diff before I commit", "is this commit clean", or describes uncommitted work and asks what to do next. Also trigger when the user pastes git status or git diff output, or says they're about to commit or push. Always use this skill before drafting any commit message or running git add/commit — even if the change seems simple. Pairs with commit-push-pr (which adds push + PR drafting on top of this review).
---

# commit-safe

A conservative pre-commit review. Inspects the repo, flags hazards, drafts a Conventional Commit message, and waits for approval. Never runs `git add` or `git commit` without explicit "yes" from the user.

---

## Hard rules (read first)

1. **Never run `git add`, `git commit`, `git push`, or `git reset` without explicit approval from the user in chat.** "Go ahead", "yes", "commit it", or "approved" counts. Silence does not.
2. **Always run the inspection commands in Step 1 before drafting anything.** No exceptions. Even if the user pasted a diff already — run them yourself to get the current state.
3. **If any hazard is found, surface it before drafting the message.** Don't bury hazards inside the recommendation section.
4. **One logical change per commit.** If the diff is mixed, recommend a split — don't merge unrelated work into one message.

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

If `git status --short` returns nothing, stop. Tell the user there's nothing to commit. Don't proceed.

---

## Step 2: Identify the change

Look at the diff and decide which of these applies:

- **One clean logical change** → proceed to draft a single commit message
- **Multiple unrelated changes** → recommend a split, with suggested file groupings and a commit message for each group
- **Accidental / generated / debug / secret files in the diff** → flag immediately, recommend removing from the commit before proceeding

---

## Step 3: Hazard check

Scan the diff for any of these. If found, list them in the Safety check section.

| Hazard | What to look for |
|---|---|
| Secrets | API keys, tokens, passwords, private keys, OAuth client secrets |
| `.env` files | Any `.env`, `.env.local`, `.env.production`, etc. |
| Local config | `config.local.*`, IDE settings (`.vscode/`, `.idea/`), OS files (`.DS_Store`, `Thumbs.db`) |
| Generated artifacts | `dist/`, `build/`, `node_modules/`, `__pycache__/`, `.next/`, compiled binaries |
| Debug code | `console.log`, `print()` for debugging, commented-out code blocks, TODO/FIXME left over |
| Unrelated churn | Whitespace-only changes in files unrelated to the main work, accidental reformatting |
| Large files | Anything over ~1 MB that isn't obviously intentional |
| Local data | Exported CSVs, cached API responses, database dumps, fixture data not meant for the repo |
| Personal tool config | Editor/notebook/vault config folders that shouldn't be shared (e.g., `.obsidian/`, `.jupyter/`) |

If nothing suspicious: say "No obvious commit hazards found."

---

## Step 4: Draft the Conventional Commit message

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
- Do not describe every file changed — the diff already shows that
- Use the body to explain **why**, not **what**
- Mention breaking changes clearly with `BREAKING CHANGE:` in the footer

### Edge cases — fetch the spec

If the change involves any of: breaking changes, multi-scope commits, footer conventions (`Refs:`, `Closes:`, `BREAKING CHANGE:`), or anything unusual — fetch the current spec before drafting:

- Fetch `https://www.conventionalcommits.org/en/v1.0.0/`
- If the fetched spec conflicts with this file, follow the spec.

---

## Step 5: Output

Return exactly these five sections, in this order, with these headers.

### Change summary
One paragraph. What changed and why, based on the diff.

### Safety check
List every hazard found, or write "No obvious commit hazards found."

### Suggested commit
```text
type(scope): imperative summary

Optional body.
```

### Files to include
Bullet list of the files that should be staged. If recommending a split, group them by proposed commit.

### Recommendation
One of:
- **Commit as-is** — diff is clean, message is drafted, ready when the user approves
- **Split into multiple commits** — list the groups and their messages
- **Fix something first** — list what to fix (remove `.env`, gitignore the artifact, etc.) before committing

---

## Step 6: Wait for approval

After delivering the output, stop. Do not run `git add` or `git commit`.

Wait for the user to say one of:
- "Go ahead" / "commit it" / "yes" / "approved" → then run `git add <files>` and `git commit -m "..."` with the approved message
- "Change the message to..." → redraft and show again, wait for approval
- "Split it like you suggested" → run the staging and commits in order, one at a time, confirming each
- "Wait, let me fix X first" → stop entirely, let the user come back

If the response is ambiguous ("looks good"), ask: "Commit now, or did you want to change anything first?"

---

## Anti-patterns (do not do)

- Don't run `git add .` — always stage specific files
- Don't auto-amend a previous commit
- Don't force-push, ever
- Don't drop hazards into the recommendation section without flagging them in Safety check
- Don't draft a commit message before running the inspection commands
- Don't assume the staged diff and unstaged diff are the same — check both

---

## Scope conventions (style note, not a hard rule)

Use the component, module, or package name as the scope. Examples:

- `feat(auth): add OAuth token refresh`
- `fix(api): correct timezone offset in scheduling`
- `docs(readme): clarify install steps`
- `chore(deps): bump SDK to latest`

If the scope isn't obvious from the diff, ask before guessing.
