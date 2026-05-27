# cos-campaign-tools

> DM tools for Curse of Strahd — built on a narrative state machine model of Barovia. Tracks campaign world state, surfaces what's unlocked, and helps new DMs prep sessions without losing the thread. Powered by the Story Complete NSM framework.

---

## What This Is

Curse of Strahd is one of the best D&D modules ever written — and one of the hardest to run. The Tarokka reading branches the narrative, NPCs move between locations, and new DMs routinely lose the thread around session 3.

This project builds tools on top of a structured, machine-readable model of the entire Barovia narrative graph: 208 nodes, 726 edges, every precondition and world-state change explicit. The goal is a campaign companion that tells you what your party has done, what's now unlocked, and what to prep for next session.

---

## What's Here Now

| Path | Contents |
|---|---|
| `data/cos.context.json` | LLM-ready authoring context — 181 KB |
| `data/cos.serialize.json` | Full narrative graph — 208 nodes, 726 edges, 1.4 MB |
| `viz/cos.viz.html` | Standalone swimlane visualization — open in any browser |

Open `viz/cos.viz.html` locally — no server needed.

---

## Data Model

The graph is produced by [Story Complete](https://github.com/marc-a-gardiner/story-complete), a narrative modelling tool built on the Narrative State Machine (NSM) architecture. The Curse of Strahd universe is modelled as:

- **Characters** — Strahd, Ireena, Van Richten, Madam Eva, and 25+ NPCs
- **Locations** — 25 locations across Barovia
- **Scenes** — atomic interactions with preconditions, world changes, and expected outcomes
- **Stories** — user journeys through the narrative
- **Conditions** — named world-state facts (e.g. `sunsword-location-known`, `vallaki-political-crisis`)
- **Rules** — world constraints that govern the narrative (the Curse, the Mists, Strahd's Dominion)

See [`docs/DATA-MODEL.md`](docs/DATA-MODEL.md) for a full reference.

---

## Fan Content Notice

This project is unofficial Fan Content permitted under the [Wizards of the Coast Fan Content Policy](https://company.wizards.com/en/legal/fancontentpolicy). Not approved or endorsed by Wizards. Portions of the materials used are property of Wizards of the Coast LLC. © Wizards of the Coast LLC.

**Curse of Strahd** is a trademark of Wizards of the Coast. This project is non-commercial.

---

## License

The code and tooling in this repository are licensed under the [MIT License](LICENSE).

Content derived from the Curse of Strahd module remains the intellectual property of Wizards of the Coast and is used here under the Fan Content Policy (non-commercial).
