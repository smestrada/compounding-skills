---
name: gmail-daily-briefing
description: >
  Reads your Gmail inbox, cuts through the clutter, and delivers a focused
  daily briefing. Use this skill whenever you say "check my email", "what's
  in my inbox", "give me my daily briefing", "triage my email", "what did I
  miss", "catch me up on my emails", or anything about wanting a summary or
  digest of your Gmail. Also trigger when you ask what AI news, tech news, or
  marketing news came in. This skill separates signal from noise: it surfaces
  the emails worth reading and flags everything else as clutter to dismiss.
---

# Gmail Daily Briefing Skill

You get a high volume of newsletters, news digests, and promotional emails
every day. Your goal is to stay informed on AI, tech, news, and marketing —
without spending hours in your inbox. This skill handles the triage.

## What this skill does

1. Pulls today's unread emails from Gmail
2. Classifies each one as SIGNAL or CLUTTER
3. Outputs a clean briefing: key takeaways from the keepers, a list of clutter
   to dismiss
4. **Runs an operator-level analysis** on the full inbox: what a top 1%
   operator would stop/start/double down on, and where you are being the
   bottleneck

---

## Step 1: Fetch today's unread emails

Search Gmail for unread messages from the past 24 hours:

```
q: "is:unread newer_than:1d"
maxResults: 50
```

If you say "yesterday" or "last 2 days", adjust the `newer_than` value
accordingly. Grab message IDs and subjects from the results.

---

## Step 2: Classify each email

Use the subject line and snippet to make an initial SIGNAL vs. CLUTTER call.
Then, for every email you classify as SIGNAL, you MUST open the full email
using `gmail_read_message` before writing the summary. Do not summarize from
the snippet alone — the snippet is only enough to decide whether to open it,
not enough to write a useful summary.

Clutter emails do not need to be opened. The snippet is enough to confirm
clutter classification.

### SIGNAL — worth reading/summarizing
Apply these criteria:

**Content categories that count as signal:**
- AI news: new model releases, AI tool launches, research breakthroughs,
  policy/regulation, notable AI use cases
- Tech news: meaningful product launches, developer tools, platform changes
- Marketing & business: strategy insights, platform algorithm changes, data/
  research, trends relevant to your industry
- World news: significant geopolitical events, economic shifts, policy changes
- Substantive newsletters: long reads, analysis pieces, curated digests with
  real editorial judgment

**Examples of high-signal newsletter types:**
- AI-focused newsletters (model releases, research, tooling)
- Geopolitics analysis (Foreign Affairs, etc.)
- World news summaries (1440, etc.)
- Marketing strategy and platform change coverage

### CLUTTER — skip/archive
Flag as clutter if ANY of these apply:

- **Promotional/sales**: discount codes, product sales, "shop now", seasonal
  offers
- **Duplicate story coverage**: same AI/tech story covered by 3+ newsletters —
  keep the most substantive one, flag the rest as redundant
- **Low-signal digests**: vague subject lines, countdown urgency emails
- **Course/product pitches disguised as newsletters**: emails primarily selling
  a course, community, or product rather than delivering information
- **Platform/account notices**: storage warnings, account verifications, order
  confirmations

**Tip:** Add senders that consistently land in CLUTTER to your clutter list
in a `references/senders.md` file so this skill can skip them faster over
time.

---

## Step 3: Build the briefing as an HTML artifact

Output a self-contained HTML file — NOT plain text. The HTML must include
all styles inline (no external dependencies except Google Fonts). Structure:

### Layout
- **Masthead**: Dark header with title "📬 Daily Briefing", date, and pill
  counts (X Signal / Y Clutter / Z FYI). The Operator tab is always present
  — no pill count needed since it's an analysis, not a count.
- **Tab bar**: Sticky navigation with four tabs — Signal, Clutter, FYI,
  Operator. Clicking a tab shows only that section. Do NOT include a section
  header below the tab bar — go straight into category labels and cards.
  The Operator tab should have a subtle visual distinction (e.g., a small
  ⚡ icon) because it's the highest-value section.
- **Content**: Max-width centered layout with cards

### Signal cards
Each signal email gets its own card with:
- Sender badge (styled pill in green)
- Bold headline (rewritten if needed — make it informative, not clickbait)
- **Source line**: Show the sender email address and a direct Gmail link using
  the thread ID captured during Step 1. Format:
  `<a href="https://mail.google.com/mail/u/0/#inbox/THREAD_ID">Open in Gmail →</a>`
  This must always be present so you can open the actual email.
- **"What it says"** label + body: 3-5 sentences of actual substance —
  specific tools, models, numbers, findings, arguments. Multiple stories in
  one newsletter each get a bolded sub-label and a sentence or two.
- **"Read it if"** label + body: direct yes/no recommendation with a reason.
  End with either a green "→ Open for X" pill or a gray "✓ Summary covers it"
  pill. Do NOT use a non-functional button for this — the Gmail link above is
  the actual open action.

Group signal cards under category labels:
- AI & Tech
- News & World
- Marketing & Business

### Clutter rows
Compact cards in a red-tinted style. Each shows:
- Sender name (bold)
- Subject line
- Reason tag (sales promo / course pitch / duplicate / low-signal)

Group under: Promotional/Sales, Course Pitches, Duplicate Coverage, Low-Signal

### FYI rows
Compact cards in a warm amber/gold style. Each shows:
- Sender + subject
- One-line note (what to do with it, if anything)

### Operator View cards
The Operator tab uses a distinct **navy/ink** color scheme to signal this is
strategic, not tactical. Three sections, each in its own card:

**1. Stop / Start / Double Down**
A 3-column layout (stack on mobile). Each column has:
- Bold column header: ⛔ STOP / ▶️ START / 🔁 DOUBLE DOWN
- 2-4 bullet items, each one sentence
- Every item must cite which emails revealed the pattern — not just a vague
  description
- Items are PATTERN-level, not one-off. One promo email is not a pattern.

**2. Where You're the Bottleneck**
A vertical list of 2-5 bottleneck findings. Each one has:
- Bold headline naming the bottleneck
- One sentence of evidence from the inbox
- One sentence of the smallest action that unblocks it today
- A ⏱ time estimate for the unblock action ("~10 min", "~2 min reply")

**3. The One Thing**
A single highlighted callout — the ONE action that would compound most if
done today. Must be concrete, not vague.

### Design tokens (use these exactly)
```
--ink: #1a1a1a
--paper: #f5f2ec
--card: #ffffff
--signal: #0f5c3a  (green)
--signal-bg: #edf7f2
--signal-border: #6dbf96
--clutter: #7a2020  (red)
--clutter-bg: #fdf2f2
--clutter-border: #e8a0a0
--fyi: #5a4a1a  (amber)
--fyi-bg: #fdf8ee
--fyi-border: #d4b86a
--operator: #0a1628  (deep navy)
--operator-bg: #eef1f6
--operator-border: #0171bb
--operator-accent: #fa8128  (reserved for "The One Thing")
--muted: #888
--rule: #ddd8cf
```
Fonts: Playfair Display (headings), DM Sans (body), DM Mono (badges/counts).
Load from Google Fonts.

### Tab switching
Include a small JS function to show/hide sections and update active tab styling.

Present the output as an artifact the user can open and read — not as a
code block in the chat.

---

## Step 4: Duplicate story detection

Before finalizing, scan signal emails for overlapping stories. If 3 or more
newsletters covered the same story, keep the most detailed/substantive one in
SIGNAL and add the others to CLUTTER with note: *"duplicate — [better source]
covers this better"*

---

## Step 5: Operator-level analysis

This is the highest-value step. After classifying everything, step back and
analyze the **inbox as a whole** — not email by email. The inbox is a
signal about how you are spending attention. A top 1% operator would read
these patterns, not the individual messages.

### What to look for

Before writing anything, scan across ALL emails (signal, clutter, FYI, AND
anything that looked like genuine correspondence) and identify:

**Attention leaks** — places where time is leaking:
- Newsletters you read but never act on (consumption without application)
- Topics covered by 5+ overlapping sources (redundant information diet)
- Course/community pitches you haven't unsubscribed from (decision debt)
- Promotional senders that appear daily or weekly (subscription bloat)

**Signal-to-noise ratio per sender** — which senders earn their inbox slot:
- Senders whose emails consistently appear in CLUTTER → candidates to
  unsubscribe
- Senders whose emails consistently appear in SIGNAL → candidates to
  elevate (filter to a priority label, read first)

**Bottleneck signals** — places where you are the chokepoint:
- Threads with direct questions to you, especially 2+ days old
- Colleague names in threads you haven't replied to
- Decisions waiting on your input
- Follow-ups from senders you initiated contact with (you asked, they
  answered, you haven't closed the loop)
- Subscriptions/renewals/trials expiring soon that need a decision
- Recurring "checking in" emails from the same sender — means the original
  ask wasn't resolved

**Strategic drift** — is your attention matched to your stated priorities?
- Define your top 2-3 priorities in `references/priorities.md`
- If the inbox is dominated by unrelated content, that's drift worth surfacing

### How to write the output

**Stop / Start / Double Down rules:**
- Every recommendation cites specific evidence from today's inbox
- "Stop" = a behavior pattern to end (not a single email to delete)
- "Start" = a new behavior the inbox suggests would pay off
- "Double Down" = a signal source or practice already working that
  deserves more weight
- Keep each item to ONE sentence. If you need two, it's not sharp enough.
- No generic advice. Cite actual senders and counts.

**Bottleneck rules:**
- Only list bottlenecks with concrete evidence in the inbox. Do NOT speculate.
- If no bottlenecks are visible in today's inbox, say so plainly:
  "No bottlenecks visible in today's inbox." Do not fabricate.
- Rank by severity: client-waiting > colleague-waiting > vendor-waiting >
  self-imposed
- The unblock action must be doable today in under 30 minutes

**The One Thing rules:**
- Pick from the bottleneck list OR from a "start" item — whichever has the
  highest leverage if done today
- If the inbox genuinely has nothing of strategic weight, The One Thing
  should be a clutter-cleanup action
- Never fluff this. If there's no high-leverage action, say so.

### Tone for Operator View

Direct, pattern-focused, no cheerleading. No "great job today!" or "you've
got this!". State the pattern, state the action, move on.

---

## Tone & format guidelines

- Write at a high school reading level — clear and direct
- No filler phrases like "It's worth noting that..." or "Interestingly..."
- Summaries should be dense with actual information, not vibes
- Use the sender's actual name (e.g., "DeepLearning.AI" not "a newsletter")
- Output is always an HTML artifact — never plain text or a code block
- Each signal card should be readable in 30 seconds; each clutter row in 3 seconds
- Keep the whole briefing skimmable — you should be able to triage in under 5 minutes

---

## Edge cases

- **No new signal emails**: "Nothing worth your time today. [X] clutter emails
  to archive."
- **Massive volume (50+ emails)**: Process the 50 most recent. Note if more
  exist.
- **Ambiguous sender**: If a sender is unfamiliar, open the email and classify
  based on content.
- **Important personal emails mixed in**: Flag immediately at the top as
  ⚡ PRIORITY before the briefing sections.
- **Thin inbox for Operator View**: If fewer than 10 emails came in, the
  Operator tab may have limited patterns to surface. Show only what the
  evidence supports — a short Operator View is better than a padded one.
- **Operator view default tab on open**: Keep Signal as the default tab.
  Land on the content first, then click into Operator for the meta-view.
