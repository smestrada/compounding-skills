---
name: weekly-tune-up
description: Analyzes the user's AI assistant usage from the past 7 days and produces a "Weekly Tune-Up" — a premium HTML dashboard that surfaces friction, leverage, skill gaps, and strategic drift with specific, actionable recommendations. Use this skill whenever the user asks to "run the weekly tune-up", "audit my usage", "run my Sunday review", "analyze what I did this week", "find friction in my workflows", "find what skills to build", "review my performance", or any request to review, audit, or optimize past assistant usage. Also triggers on "run the tune-up", "self-review", or "weekly audit". Always use this skill before writing any usage audit — even if the request seems simple. The skill also tracks last week's recommendations and holds the user accountable to applying them.
---

# Weekly Tune-Up — Self-Improvement Audit

This skill turns the user's past 7 days of assistant usage into a decision document. It is not a summary. It surfaces what needs to change.

## First-run check

Before doing anything else, check whether `references/user-context.md` exists and has been filled in.

If the file is missing OR contains only the template placeholder text (`[ONBOARDING NOT YET COMPLETED]`), run the **Onboarding Interview** (see below) before doing any analysis. Do not skip this.

If the file exists and is filled in, read it and apply it throughout the analysis and report.

## Onboarding Interview

Tell the user: "Before I can run the weekly tune-up for you, I need to understand your priorities and communication style. This is a one-time interview — I'll save your answers so future runs skip this step."

Ask the following questions using `ask_user_input_v0` where options make sense, and plain text for open-ended ones. Batch them 2–3 at a time so it doesn't feel overwhelming.

**Round 1 — Identity & priorities**
1. "What's your name?" (open)
2. "What are your top 2–4 priority areas right now? Examples: a side business, a day job, a health goal, a creative project." (open — ask for 2–4 short labels)
3. "For each priority area, what rough percentage of your assistant time would you *want* to go there in a healthy week?" (open — should sum to ~100%)

**Round 2 — Tone & accountability**
4. "How direct do you want the feedback?" (single_select)
   - "Plain and direct — no hype, no soft language"
   - "Balanced — direct but warm"
   - "Gentle — frame everything constructively"
5. "When you ignore a recommendation for 2+ weeks, how should I escalate?" (single_select)
   - "Call it out directly and name the pattern"
   - "Keep suggesting it with the same tone"
   - "Drop it after 2 weeks of being ignored"

**Round 3 — Output preferences**
6. "What reading level / complexity do you want in explanations?" (single_select)
   - "High school (plain English)"
   - "College-educated general reader"
   - "Expert / technical — no need to simplify"
7. "Anything you want me to never bring up unsolicited? (e.g. sensitive personal topics, specific life events)" (open — okay if blank)
8. "Is there anyone else whose work is tracked in your chats that I should know about? (e.g. a business partner, client, team member whose name will come up)" (open — okay if blank)

**Round 4 — Brand (optional)**
9. "Do you want the report styled in a specific color palette? Paste up to 4 hex codes, or say 'default' to use the built-in deep navy / orange palette." (open)
10. "Anything else I should know that would make the weekly tune-up more useful to you?" (open — okay if blank)

After they finish answering, write everything to `references/user-context.md` using the template structure below. Confirm the write, then continue to Step 1 of the main workflow.

## What this skill produces

A single HTML dashboard saved to `/mnt/user-data/outputs/` named `weekly-tune-up-YYYY-MM-DD.html`, containing:

1. **The Week in One Sentence** — what the user actually spent their assistant time on
2. **Top 3 Moves for Next Week** — the only things that matter (P1 priorities)
3. **Friction Log** — conversations that went sideways, with the evidence
4. **Leverage Map** — which 20% of usage drove 80% of output
5. **Skill Performance** — which skills fired, which missed, which to kill
6. **Strategic Drift Check** — alignment with user's stated priorities
7. **Kill List** — what to stop doing, delete, or archive
8. **Last Week's Scorecard** — did the user actually apply last week's recommendations?
9. **Proposed Changes** — copy-paste ready skill edits, memory edits, and new routines

## The workflow

Follow these steps in order. Do not skip.

### Step 0 — Read user context

Read `references/user-context.md`. Internalize:
- The user's name (use it naturally, not performatively)
- Their priority areas and target percentages
- Their tone preference (apply it to the entire report)
- Topics to avoid
- Named people whose work might appear in chats
- Brand colors (default to deep navy / orange if none specified)

### Step 1 — Pull the data

Use these tools to build the raw dataset:

1. `recent_chats` with `n=20`, `sort_order=desc` — get the most recent 20 chats
2. If the oldest chat is still within the last 7 days, paginate with `before` set to that chat's `updated_at` and pull 20 more. Repeat until you reach 7 days back or hit 5 total calls.
3. For each chat, read the snippet carefully. Note:
   - The task type (coding, content writing, research, strategy, automation, admin)
   - Which skill(s) seemed to fire (or should have fired)
   - Whether the user had to re-prompt or correct ("no, I meant", "try again", "that's not right")
   - Whether the task was part of a known priority area from the user context
   - Whether it was a repeat of something they've done before (candidate for automation)

### Step 2 — Check last week's recommendations

Use `conversation_search` with query `"Weekly Tune-Up"` or `"tune-up recommendations"` to find last week's report. If found, extract the P1 recommendations. Cross-reference against this week's activity:

- Did they apply them? (Evidence: new skill file referenced, new routine triggered, memory edit made, behavior changed)
- Which are still pending?
- Which did they explicitly reject or ignore?

If no prior report exists, note this as "baseline run — no prior recommendations to evaluate."

### Step 3 — Run the seven analyses

Before writing the HTML, work through each of these in order. Write conclusions as short bullet notes — not prose.

**Analysis 1: Usage forensics**
- Total chats opened in the period
- Rough breakdown by task type (percentages)
- Time-of-day patterns if discernible from timestamps
- Which skills fired successfully
- Which tasks should have triggered a skill but didn't (skill description problem)
- Which tasks were manual that could be a skill

**Analysis 2: Friction detection**
- Flag any chat with evidence of re-prompting, correction, or frustration
- For each, name the root cause: unclear prompt, wrong skill, missing context, skill quality issue
- Count how many minutes of friction this likely represented

**Analysis 3: Leverage analysis**
- Which 20% of chats drove the most meaningful output?
- Which tasks repeated 3+ times in the week (automation candidates)?
- Which skills haven't been used in 30+ days? (Cross-reference available_skills against recent usage)

**Analysis 4: Strategic drift check**
Compare the week's activity against the priority areas in `user-context.md`.

For each priority area, estimate rough percentage of assistant time spent. Flag any imbalance against the user's stated target percentages. Call out what's NOT happening that should be.

**Analysis 5: Repeated context leaks**
- Context the user typed into fresh chats that belongs in memory or user preferences
- Instructions given multiple times in different chats

**Analysis 6: Accountability check**
- For each P1 recommendation from last week, verdict: Applied / Partial / Ignored / Rejected
- Escalation follows the user's preference from onboarding:
  - "Call it out directly" → 2nd week ignored = stronger language; 3rd week = demote or reframe
  - "Keep suggesting same tone" → flag it but don't escalate
  - "Drop after 2 weeks" → don't repeat a recommendation that's been ignored twice

**Analysis 7: The Three Moves**
Pick the three highest-leverage changes the user could make next week. These must be:
- Specific (not "improve your skill" — actual diff-level: "add this trigger phrase")
- Actionable in under 30 minutes
- Evidence-backed (tied to something that happened this week)

### Step 4 — Build the HTML dashboard

Read `references/dashboard-template.html` for the full styling system. The design follows these principles:

- **Dark premium aesthetic** — deep navy background, generous whitespace, serif headlines for weight
- **One-screen-per-section** — sections don't cram; each analysis gets breathing room
- **Evidence-first** — every recommendation shows the evidence that generated it
- **Copy-paste ready** — skill edits, memory edits, and new routine prompts in code blocks
- **Scorecard-forward** — last week's accountability is near the top, not buried
- **No filler** — if a section has nothing meaningful, replace it with a short "Nothing notable here" line

**Brand colors:** Use the hex codes from `user-context.md` if the user provided them. Otherwise use the default palette:
- `#0A1628` — deep navy (background)
- `#0171bb` — blue (accents, links)
- `#fa8128` — orange (callouts, P1 priority, warnings)
- `#929292` — medium grey (secondary text)
- `#F5F5F0` — warm off-white (body text on dark)

Typography stays the same regardless of brand colors:
- Headlines: serif (`'Playfair Display'`, `Georgia`, serif)
- Body: sans (`-apple-system`, `'Inter'`, sans-serif)
- Data/code: mono (`'JetBrains Mono'`, monospace)

### Step 5 — Save and present

1. Save the HTML file to `/mnt/user-data/outputs/weekly-tune-up-YYYY-MM-DD.html` where the date is today's date
2. Use `present_files` to deliver it
3. After presenting, give the user a 3-sentence summary of the top finding — not a rehash of the report

## What this skill does NOT do

- Does not write to external tools (Notion, Slack, etc.) unless the user asks — keep it local and fast
- Does not recommend changes that require more than 30 minutes to apply
- Does not skip the accountability check even if it's uncomfortable
- Does not include every finding — only what matters

## Tone rules

Apply the tone preference from `user-context.md` throughout. Regardless of tone setting:

- Evidence before recommendation, always
- Match the reading level from the user context
- No self-congratulation: skip "great job this week" filler unless the user specifically asked for warm tone
- No hype words: "unlock", "supercharge", "level up", "game-changer", "revolutionary" — these never belong here

## Edge cases

- **Fewer than 5 chats in the week:** Note it as a low-activity week. Focus on strategic drift (was the lack of activity intentional?) rather than pattern analysis.
- **No prior tune-up found:** Skip the scorecard section; add a line in the intro that this is the baseline.
- **One task type dominated (>70%):** Don't dilute the findings trying to be balanced. Call it out as the dominant pattern and analyze accordingly.
- **Sensitive content in chats (health, personal stress, topics the user listed as off-limits):** Do not include it in the report or recommendations. Focus only on work/productivity patterns.

## References

- `references/user-context.md` — User's priorities, tone, and preferences (created by onboarding)
- `references/dashboard-template.html` — The full HTML template with placeholders
- `references/analysis-prompts.md` — Deeper questions for each analysis stage (optional)
