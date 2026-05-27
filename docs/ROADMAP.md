# Roadmap

> **Strategic direction:** The NSM engine is the product. Curse of Strahd is the flagship demo universe. Every tool phase is built universe-agnostic first — CoS data ships as the free starter dataset. This decouples the tool from WotC IP and broadens the addressable market to any TTRPG module, not just CoS.

---

## Strategic North Star

A DM campaign companion powered by the Narrative State Machine framework:
- Accepts any NSM-format universe file (`.serialize.json`)
- CoS is the bundled demo; custom universes are user-supplied or community-contributed
- The tool is the commercial asset — universe data files are free/open
- Story Complete (upstream) is the authoring environment; this repo is the player/consumer layer

---

## Phase 1 — Data Foundation ✓ (complete)

Graph exported, visualization published, data model documented.

---

## Phase 2 — Universe-Agnostic Engine Core

The architectural foundation everything else builds on. The engine must be universe-neutral before Phase 3.

- [ ] Define the `Universe` loader interface — accepts any `.serialize.json` file matching the NSM schema
- [ ] Validate CoS graph loads cleanly through the loader (regression baseline)
- [ ] Write a minimal second test universe (stub, even 10 nodes) to prove universe-agnosticism
- [ ] Build Campaign State model: played scenes → derived world-state (active conditions, unlocked scenes, story status)
- [ ] Expose a query API: `what_is_unlocked(played_scenes)`, `active_conditions(played_scenes)`, `stories_in_progress(played_scenes)`
- [ ] Decision: engine language — Python CLI first (fastest), or JS/TS for browser-native?

---

## Phase 3 — Campaign State Tracker (first tool)

Given a session log (played scenes + confirmed conditions), surface:
- Active world-state facts
- Scenes with satisfied preconditions (ready to play)
- In-progress vs. blocked vs. complete stories
- Session prep brief: key NPCs, locations, recommended next scenes

**Output target:** shareable static HTML snapshot a DM can bookmark or print.

Design decisions:
- [ ] Input UX: scene checklist vs. condition toggles vs. natural language session log
- [ ] Output format: HTML report, Markdown, JSON handoff for further tooling
- [ ] Hosting model: local CLI tool first; hosted web app later as a separate phase

**Commercial note:** this tool works for any NSM universe — the CoS version is just the one with the prettiest data.

---

## Phase 4 — NPC Relationship Map

Interactive graph of character connections, motivations, and alliances — filterable by arc, location, or faction. Universe-agnostic: driven entirely by `character` nodes and their edges.

- [ ] Identify which edge types encode character relationships in the NSM schema
- [ ] Build filterable force-directed graph view (extend `viz/cos.viz.html` or new component)
- [ ] Export: SVG/PNG for DM use at the table

---

## Phase 5 — Session Prep Assistant (AI-powered)

Given campaign state, generate a structured session brief via Claude API:
- Scene recommendations with difficulty and narrative fit
- NPC motivations relevant to current party position
- Foreshadowed events now approaching
- Adversarial variants for likely upcoming scenes

**Input:** `cos.context.json` (or equivalent per-universe context file) + campaign state snapshot  
**Output:** structured Markdown brief, optionally saved as static HTML

- [ ] Design the prompt template (universe context + state snapshot → brief)
- [ ] Evaluate `cos.context.json` quality: is it dense enough for zero-shot session prep?
- [ ] Add prompt caching (Anthropic SDK) — context file is large and static per session
- [ ] DM feedback loop: thumbs up/down on recommendations feeds quality signal

**Commercial note:** API calls are a natural usage-based cost to pass through or absorb in a subscription tier.

---

## Phase 6 — Monetisation & Distribution

To be designed once Phase 3 ships and real DMs use it.

Candidate models:
- **Free tier:** local tool + CoS starter data, open source
- **Paid tier:** hosted web app, custom universe uploads, AI-powered session prep, saved campaign history
- **DMs Guild complement:** CoS-specific content packs (encounter variants, NPC expansions) distributed there, tool itself separate

Open questions:
- [ ] Is the right vehicle a web app, a desktop Electron app, or a VS Code / Obsidian plugin?
- [ ] Does the upstream Story Complete editor become a paid authoring tier?
- [ ] Community marketplace for universe files — does that need a legal wrapper?

---

## Phase 7 — Community Features

To be shaped by community feedback from Phases 3–5. Candidates:

- Campaign journal: auto-generated session recap from state changes
- Encounter variant library: DM-contributed alternate scene outcomes
- Multi-universe library: Curse of Strahd, Tomb of Annihilation, Rime of the Frostmaiden...
- New DM onboarding: guided walkthrough of the Barovia narrative for first-time CoS DMs

---

## Open Architecture Decisions

| Decision | Options | Status |
|---|---|---|
| Engine language | Python CLI / TypeScript / Python + TS bindings | Open |
| Hosting model | Static HTML / hosted web app / Electron / plugin | Open |
| Universe file ownership | User-supplied only / community hub / built-in bundle | Open |
| Commercial structure | Freemium SaaS / one-time tool / DMs Guild hybrid | Open |
| Story Complete coupling | Tight (same schema, same repo owner) / loose (published spec) | Open |
