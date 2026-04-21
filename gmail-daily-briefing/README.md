# Gmail Daily Briefing

A Claude skill that reads your inbox every morning and tells you what actually matters.

Most people open Gmail and get pulled into whatever landed last. You skim half a dozen newsletters, delete a bunch of promos, lose 20 minutes, and still aren't sure you caught anything important. This skill fixes that.

Once a day, you tell Claude "check my email" or "give me my daily briefing." It pulls your last 24 hours of unread mail, runs everything through a signal vs. clutter filter, and produces a clean HTML briefing with:

- The emails worth reading — summarized with actual substance, not vibes
- Everything else flagged and categorized so you can archive in bulk
- An **Operator View** tab that reads your inbox as a whole and surfaces patterns: where you're leaking attention, where you're the bottleneck, and the one thing worth doing today

## How it works

1. **Fetch** — Pulls unread email from the past 24 hours (up to 50)
2. **Classify** — Each email is SIGNAL, CLUTTER, or FYI
3. **Summarize** — Signal emails are opened and summarized with real detail
4. **Build the briefing** — A tabbed HTML artifact: Signal / Clutter / FYI / Operator
5. **Operator analysis** — Reads the inbox as a system, not a list

## What you get

- Signal cards with source links, key takeaways, and a read-it-if recommendation
- Clutter rows grouped by type (promo, course pitch, duplicate, low-signal)
- FYI rows for things that don't need action but are worth knowing
- Operator View with Stop / Start / Double Down, bottleneck findings, and The One Thing

## Setup

1. Connect your Gmail account to Claude via the Gmail MCP connector
2. Copy `SKILL.md` into your Claude skills folder
3. Customize `references/senders.md` with your known signal and clutter senders
4. Customize `references/priorities.md` with your current top priorities (used by the Operator View)

## Trigger phrases

- "Check my email"
- "What's in my inbox"
- "Give me my daily briefing"
- "Triage my email"
- "What did I miss"
- "Catch me up on my emails"
- "What AI / tech / marketing news came in"

## Customization

The skill is designed to be personalized. The `references/` folder is where you store:

- **`senders.md`** — Your known signal senders (always open) and known clutter senders (always skip). The more you maintain this, the faster and more accurate the briefing gets.
- **`priorities.md`** — Your current top 2-3 priorities. The Operator View uses this to flag strategic drift when your inbox attention doesn't match your goals.

## Design

The briefing uses a clean editorial layout with four tabs. The Operator tab uses a distinct navy color scheme to signal it's the strategic layer, not the tactical one. Design tokens are defined in `SKILL.md` if you want to restyle it.

## Requirements

- Claude with Gmail MCP connector enabled
- Claude skills support (claude.ai or Claude Code)
