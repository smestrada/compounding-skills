# Compounding Skills

Opinionated Claude skills built on one idea: the right skill earns its keep by making next week's work easier than this week's.

Most AI tools give you the same output every time you use them. These don't. Each skill here is designed to compound — either by automating something you'd otherwise redo manually, by using past data to make future decisions sharper, or by holding you accountable to the workflow changes you said you'd make.

Different skills, same principle: the longer you use them, the more leverage they give you.

---

## Skills in this repo

### [weekly-tune-up](./weekly-tune-up)

Audits your last 7 days of Claude usage and produces a premium HTML dashboard with the three highest-leverage changes to make next week. Tracks whether you applied last week's recommendations and holds you accountable when you don't.

*One-time onboarding interview personalizes the skill to your priorities and tone. Every run after that skips straight to analysis.*

[→ Read the full docs](./weekly-tune-up/README.md) · [→ Download the .skill file](../../releases)

---

## How to install a skill

1. Download the `.skill` file for the skill you want (from Releases, or zip the skill folder yourself)
2. Open Claude in a browser → **Settings → Capabilities → Skills**
3. Click **Upload skill** and select the file
4. It appears in your skills list, enabled by default

## What makes a skill belong here

Not every useful skill compounds. A skill belongs in this repo if it meets at least one of these:

- **Uses past data to make future work better** — each run gets sharper because the last one happened
- **Automates something repetitive** — the second time you'd do it by hand, this skill has already done it
- **Holds you accountable** — tracks whether you did what you said you'd do, and calls it out when you didn't
- **Replaces a recurring manual workflow** — Monday inbox triage, Sunday reviews, weekly content batches

If a skill is a one-shot utility (write me a poem, translate this paragraph), it doesn't belong here. Plenty of other repos do that well.

## Contributing

If you improve a skill, open a PR. If you break one, open an issue. If you build your own compounding skill and want it added here, open a discussion.

## License

MIT. Use it, modify it, ship it. Attribution appreciated but not required.

---

Built by [Sally Estrada](https://advisercmo.com).
