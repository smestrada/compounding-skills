# Weekly Tune-Up

A Claude skill that audits your own AI assistant usage every week and tells you what to fix.

Most people set up Claude once and never update anything. You keep running the same broken workflows, re-typing the same context into fresh chats, ignoring skills you built months ago, and drifting from your actual priorities without noticing. This skill catches all of that.

Once a week, you tell Claude "run the weekly tune-up." It pulls your last 7 days of chats, analyzes them across seven dimensions, and produces a premium HTML dashboard with:

- The three highest-leverage changes you could make next week (each under 30 minutes to apply)
- A friction log showing which chats went sideways and why
- A skill performance report — what fired, what should have fired, what's gathering dust
- A strategic drift check against your stated priorities
- A kill list of things to stop doing
- An accountability scorecard grading whether you applied last week's recommendations

It's self-improvement for your AI setup. It gets better every week because it tracks whether you actually did what it told you.

---

## What it looks like

The output is a single HTML file styled with a dark premium aesthetic — deep navy, serif headlines, mono for evidence blocks. Designed to be read on a Sunday morning with coffee and taken seriously.

Sections:

1. **The Week in One Sentence** — what you actually spent your assistant time on
2. **The Three Moves** — the only changes worth making next week
3. **Last Week's Scorecard** — Applied / Partial / Ignored / Rejected for each prior recommendation
4. **Friction Log** — evidence-first, cause-named
5. **Leverage Map** — what drove output, what to automate
6. **Skill Performance** — firing / missed / stale table
7. **Strategic Drift Check** — % of time per priority area vs. your target
8. **Kill List** — stop doing these
9. **Proposed Changes** — copy-paste ready skill edits, memory edits, new routines

---

## Install

1. Download the [latest .skill release](../../../releases) or zip this folder
2. Open Claude in a browser → **Settings → Capabilities → Skills**
3. Click **Upload skill** and select the file
4. It appears in your skills list, enabled by default

## First run

Say any of:

- "Run the weekly tune-up"
- "Run my Sunday review"
- "Audit my Claude usage"

**The first run does a short onboarding interview** — your name, your priority areas, your tone preference, how direct you want the feedback. These answers get saved into the skill's `references/user-context.md` so you never answer them again. After onboarding, it runs the full analysis.

Every run after that skips the interview and goes straight to analysis.

## Requirements

- Claude.ai with the `recent_chats` and `conversation_search` tools (both on by default)
- 7 days of chat history to be worth analyzing — if you just started, wait a week
- A sense of humor about being held accountable by your own tool

---

## Philosophy

This skill is opinionated. It will:

- Call out patterns you're avoiding
- Ask you to pick one thing rather than fifteen
- Escalate language on recommendations you keep ignoring
- Refuse to include filler, soft praise, or "great job this week" decoration

If you want a cheerful weekly roundup, this isn't the skill. Set the tone preference to "Gentle" during onboarding and it'll soften, but the underlying logic stays the same: evidence before recommendation, specificity over generality, actionability over insight.

## What it will not do

- Read your email, files, or anything outside the chat history
- Share your data anywhere — runs entirely in your Claude session
- Require any API keys, subscriptions, or third-party services
- Write to your Notion, Google Drive, or other tools (unless you ask, and only with MCP you've already connected)
