# Analysis Prompts

Deeper questions for each of the seven analyses. Pull from these when the surface-level pass doesn't yield enough.

## Analysis 1 — Usage Forensics

- What time of day did most chats happen? Does that match the user's energy patterns (morning deep work vs. afternoon admin)?
- Were there days with zero usage? Why?
- Which chats were single-shot (one question, one answer) vs. multi-turn? Ratio?
- Did any chat span multiple sessions (came back to it later)?
- Were there chats that started one topic and drifted into another? That's a sign of unclear intent at the start.
- Which skills fired without being needed? (Over-triggering is as bad as under-triggering.)

## Analysis 2 — Friction Detection

Signals to look for in chat snippets:
- "No, I meant" / "That's not what I wanted" / "Try again"
- "Let me clarify" / "What I actually need is"
- "Can you start over" / "Redo this"
- Repeated asks for the same output with slight tweaks (agent didn't get it right the first 3 times)
- Long back-and-forth on small details (indicates specification gap)
- The user explaining the same context twice in one chat
- Skill produced something, the user edited heavily → quality gap in the skill

Root cause options:
1. **Prompt quality** — the user's initial ask was unclear
2. **Skill quality** — skill fired but output missed
3. **Skill description** — wrong skill fired or no skill fired
4. **Missing context** — assistant needed info it didn't have (memory gap, preference gap)
5. **Tool mismatch** — wrong tool choice (e.g., should have searched, didn't)
6. **Model capability** — genuine limitation, no easy fix

## Analysis 3 — Leverage Analysis

High-leverage signals:
- Output the user used directly (published, shipped, sent)
- Tasks that unlocked something else downstream
- One-shot tasks that would have taken hours manually

Low-leverage signals:
- Exploration that didn't converge
- Chats that ended without a clear output
- Research the user didn't act on

Automation candidates (3+ repeats = strong signal):
- Same task type, different inputs, same output structure
- Could be a script, a scheduled automation, or a new skill
- Estimate: how many minutes per run × how many times per month = hours saved per year

Stale skills check:
- Pull the `available_skills` list from context
- For each skill not used in 30+ days, ask: is this waiting for a trigger that will come, or is it dead weight?
- Dead weight clutters the skill router and hurts other skills' triggering

## Analysis 4 — Strategic Drift

Read the priority areas from `references/user-context.md`.

Healthy allocation check:
- Use the user's stated target percentages as the baseline
- Any priority at 0% for the week → flag it (was the silence intentional?)
- Any priority taking 2x its target → flag it (crowding everything else?)
- All reactive work, no strategic → no future pipeline being built

What's NOT happening?
- Is the user building the next thing, or only running today's thing?
- Any known upcoming deadlines that haven't surfaced?
- Any stated goals that haven't been touched recently?

## Analysis 5 — Repeated Context Leaks

When the user types the same context into a fresh chat multiple times, that context should move to:
- **User preferences** — if it's a behavioral instruction (tone, format, always/never rules)
- **Memory edit** — if it's a factual update (new role, new project, changed status)
- **Skill** — if it's a workflow (same sequence of steps, same kind of output)
- **Project instructions** — if it only applies in a specific context

Examples of leaks:
- A person's name + role typed into multiple chats → memory edit
- "Use short sentences, no corny language" typed repeatedly → already in preferences, so why did the user type it? Preference is being ignored — investigate.
- "Here's how our [workflow] works..." repeated in chats → skill candidate

## Analysis 6 — Accountability

For each P1 from last week:
- **Applied** — evidence exists (new skill referenced, behavior changed, routine triggered)
- **Partial** — some but not all of the recommendation landed
- **Ignored** — no evidence either way, the user just didn't do it
- **Rejected** — the user actively pushed back in a chat ("I don't want to build that")

Escalation rules (adjust based on the user's accountability preference in `user-context.md`):
- 1st week ignored: note it matter-of-factly
- 2nd week ignored: ask "is this worth doing?" — maybe the recommendation was wrong
- 3rd week ignored: either demote to P3 or drop entirely. The recommendation failed.

Never shame. But don't pretend. Direct users asked for direct treatment.

## Analysis 7 — The Three Moves

Selection criteria:
1. **Evidence** — tied to something specific from this week
2. **Leverage** — high ratio of impact to effort
3. **Achievable** — under 30 minutes to apply
4. **Concrete** — exact diff, not "improve X"
5. **Trackable** — next week we can tell if it was applied

Bad moves:
- "Build better skills" — too vague
- "Consider adding a trigger phrase" — hedge words
- "Set up a scheduled automation for [complex thing]" — too big

Good moves:
- "Add the phrase 'weekly review' to [skill-name]'s description to fix the 2 missed triggers this week"
- "Delete the memory line about [stale event] — it already happened; update to reflect current state"
- "Create a user preference: 'When asked to update a database, always show the diff before applying' — would have prevented Wednesday's rollback"
