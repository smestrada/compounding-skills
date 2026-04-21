# Sender Reference

This file helps the Gmail Daily Briefing skill classify emails faster.
Update it over time as you identify patterns.

---

## Signal Senders (always open)

These senders consistently deliver high-value content. Open and summarize.

| Sender | Domain | Category | Notes |
|--------|--------|----------|-------|
| DeepLearning.AI / The Batch | deeplearning.ai | AI | Weekly AI research digest |
| AlphaSignal | alphasignal.ai | AI | Daily AI/ML signal |
| The Rundown AI | therundown.ai | AI | Daily AI news |
| 1440 Daily Digest | join1440.com | News | World news summary |
| Foreign Affairs | foreignaffairs.com | Geopolitics | Long-form analysis |

*Add your own high-signal senders here.*

---

## Clutter Senders (skip unless noted)

These senders consistently land in clutter. The skill will flag them without opening.

| Sender | Reason |
|--------|--------|
| *(add senders here)* | *(sales / course pitch / low-signal / duplicate)* |

*Build this list over time. The more entries, the faster your briefing runs.*

---

## Notes

- "Signal" doesn't mean every email from that sender is worth reading —
  it means open and check before classifying.
- "Clutter" means skip the open step. Classify from subject + snippet.
- If a clutter sender occasionally sends something genuinely useful,
  note it in the Notes column so the skill knows to spot exceptions.
