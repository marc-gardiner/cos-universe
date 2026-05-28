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
- **Decision: TypeScript, browser-native.** Minimal friction for DMs — no install, no CLI, just a URL. The existing `viz/cos.viz.html` pattern (self-contained, open in browser) extends naturally into a full app.

### Schema Evolution & Migration

Story Complete will keep updating the NSM schema. A saved campaign state recorded against schema `1.4` must survive a graph update to `1.5` without data loss or silent corruption. This is an engine-layer concern — the tracker and AI brief should never need to know the schema version.

- [ ] Embed `schema_version` in every `campaign-state.json` snapshot at write time
- [ ] On load, detect version mismatch between saved state and current graph; surface a clear warning rather than silently breaking
- [ ] Define a migration interface: `migrate(state, fromVersion, toVersion) → CampaignState` — even if the first implementation is identity (no-op), the hook must exist
- [ ] Document the schema change contract: what changes in `.serialize.json` are breaking vs. additive, so upstream Story Complete updates can be evaluated before merging

---

## Phase 3 — Campaign State Tracker (first tool)

**Problem it solves:** DMs running CoS stitch together 3–5 unrelated tools to track world state. The core failure mode is sandbox overwhelm — they don't know which scenes are available, which are blocked, or what the party's choices have changed. "What's available next" is universally desired and universally unmet.

**Target output:** A single browser-native HTML file a DM opens before each session. No install, no account, no server. Loads `cos.serialize.json` from the same directory. State persists in `localStorage`.

### Input UX — Resolved

Scene checklist as the primary input, organised by location/arc (matching how DMs mentally index the module — not alphabetically, not by node type). Secondary: condition overrides for edge cases (e.g. DM ran a scene off-book). No natural language input — too ambiguous at this stage, and state must be deterministic for the AI layer in Phase 5.

### Output Panels

The tool renders five panels from the computed campaign state:

1. **Scene Navigator** — three columns: *Ready to Play* (preconditions met, not yet played), *Blocked* (preconditions unmet, with a one-line explanation of what's missing), *Complete* (played scenes, collapsible). Sorted by narrative prominence (major > moderate > minor).

2. **World State** — active conditions displayed as named fact-cards (e.g. *"Vallaki Political Crisis — active"*, *"Sunsword Location Known — active"*). Each card shows which scenes satisfied or could violate it. This is the single most requested missing artefact — DMs currently track this on index cards or lose it entirely.

3. **Story Dashboard** — each story node shown with: entry conditions (met/unmet), scenes completed vs. total, current status (locked / in progress / complete). Gives the DM a narrative arc view rather than a location view.

4. **NPC Disposition Board** — character nodes with a session-delta indicator: which NPCs were affected by scenes played since the last session checkpoint. Surfaces the #1 continuity failure DMs report: NPC relationship drift between sessions going unnoticed.

5. **Quest Log** — active sidequests (story nodes with `prominence: minor`) listed with current precondition status and nearest unblocking scene. CoS has many sidequests buried in location text; this makes them visible.

### Tarokka Setup Wizard

The Tarokka card reading is a CoS-specific mechanic that determines where the Sunsword, Holy Symbol of Ravenkind, and Tome of Strahd are hidden, and which NPC becomes the party's ally. These outcomes gate large portions of the scene graph — the tracker cannot accurately compute what's unlocked without them. Running the tool without Tarokka inputs would surface incorrect scene availability.

The Tarokka wizard runs once per campaign, before the scene checklist. The DM either inputs the drawn cards (if already played) or pre-draws them before Session 1 (the standard DM advice). The wizard maps card outcomes to the NSM condition nodes they satisfy, seeding initial world state.

- [ ] Map all Tarokka card outcomes to their corresponding NSM condition and scene nodes
- [ ] Build setup wizard UI: card draw input (or random draw option) → confirm placements → seeds campaign state
- [ ] Persist Tarokka outcomes in `campaign-state.json` as part of the initial world state
- [ ] Handle the "reading not yet done" state gracefully — tracker still loads but flags affected scenes as indeterminate

### Multi-Campaign & Save Slots

A DM may run CoS for multiple groups simultaneously, or finish one campaign and start a second. `localStorage` keyed to a single state silently overwrites the previous campaign. Save slots must exist before any public release — losing a 20-session campaign record is an unrecoverable trust failure.

- [ ] Replace single `localStorage` key with named campaign slots (e.g. "Thursday Group", "Online Campaign")
- [ ] Campaign picker screen on launch — create new, rename, delete, switch
- [ ] Export/import campaign state as a portable `.json` file — protects against browser storage clearing and enables backups
- [ ] Warn before destructive actions (delete campaign, reset state)

### JSON State Snapshot

Every render produces a `campaign-state.json` export. This is the handoff contract to Phase 5 — the AI brief reads this file, not the raw graph. Keeps the AI layer stateless and the state layer deterministic.

### Build Tasks

- [ ] `engine/loader.ts` — `Universe` class: load and validate `.serialize.json`, index nodes by id and type
- [ ] `engine/state.ts` — `CampaignState` class: `played: Set<SceneId>`, derives `activeConditions`, `unlockedScenes`, `blockedScenes`, `storiesInProgress`
- [ ] `engine/queries.ts` — `whatIsUnlocked(played)`, `activeConditions(played)`, `storiesInProgress(played)`, `npcDelta(prevPlayed, currPlayed)`
- [ ] `engine/snapshot.ts` — serialise/deserialise `CampaignState` to/from `campaign-state.json`
- [ ] `ui/checklist.ts` — scene checklist component, grouped by location, with search/filter
- [ ] `ui/navigator.ts` — Scene Navigator panel (ready / blocked / complete columns)
- [ ] `ui/worldstate.ts` — World State condition cards
- [ ] `ui/stories.ts` — Story Dashboard panel
- [ ] `ui/npcs.ts` — NPC Disposition Board with session-delta highlights
- [ ] `ui/questlog.ts` — Quest Log panel
- [ ] `ui/export.ts` — HTML snapshot export + JSON state export
- [ ] `tracker.html` — single-file entry point bundled with esbuild; loads CoS data from `data/` directory
- [ ] Smoke test: play through 10 CoS scenes manually, verify all five panels update correctly

**Commercial note:** this tool works for any NSM universe — the CoS version ships as the free demo dataset.

---

## Phase 4 — NPC Relationship Map

Interactive graph of character connections, motivations, and alliances — filterable by arc, location, or faction. Universe-agnostic: driven entirely by `character` nodes and their edges.

- [ ] Identify which edge types encode character relationships in the NSM schema
- [ ] Build filterable force-directed graph view (extend `viz/cos.viz.html` or new component)
- [ ] Export: SVG/PNG for DM use at the table

---

## Phase 5 — Session Prep Assistant (AI-powered)

**Problem it solves:** DMs spend 2–4 hours per session on prep that is largely re-deriving context the campaign state already contains. The community accepts AI as a *collaborator*, not an oracle — but distrusts free-generation tools because of hallucinated lore. The moat here is graph-grounded output: the AI cannot invent scene availability or NPC facts because the graph already computed them. Claude only writes prose around deterministic inputs.

**Inputs:**
- `campaign-state.json` from Phase 3 (deterministic — what is true right now)
- `cos.context.json` (or universe equivalent) — LLM-optimised narrative context, injected via prompt cache
- Optional DM directive: "next session focus" (free text — e.g. "party is heading to Argynvostholt" or "I want to push the Vallaki arc")

**Output:** A single-page HTML session brief the DM can print or read on a second screen at the table.

### Brief Structure (six sections)

1. **Last Session Recap** — auto-generated 3–5 sentence summary of what changed since the previous state snapshot (scenes played, conditions flipped, stories advanced). Replaces the DM's manual notes; gives players a recap handout.

2. **Ready to Run — Top Picks** — 3–5 recommended scenes for the next session, selected from the *Ready to Play* list in campaign state. Each pick includes: why it's available now, dramatic weight, estimated length, NPCs present. AI ranks by narrative momentum (foreshadowed > orphaned; major > minor).

3. **NPC Briefing Cards** — for each NPC likely to appear in the recommended scenes: current goal, key secret, relationship to party (from disposition board delta), suggested voice/mannerism note. One card per NPC, generated from `cos.context.json` anchored to current world state — not free-invented.

4. **Location Setup** — for each likely venue: atmosphere note, key interactive elements, any condition-gated details now visible to the party. Replaces mid-session book-diving.

5. **Foreshadowing Now Landing** — hooks planted in earlier scenes that are now paying off given current state. Helps DMs call back earlier moments without re-reading 200 pages.

6. **Watch List** — conditions currently one scene away from being satisfied or violated. Gives the DM a short list of "things that could tip this session" — the narrative pressure points.

### Prompt Architecture

```
[System]   NSM framework explainer + brief format spec (static, ~500 tokens)
[Cached]   cos.context.json — full narrative context (static per universe, ~180 KB)
[Dynamic]  campaign-state.json snapshot (~2–5 KB)
[Dynamic]  DM directive (optional free text, ~50–200 tokens)
```

`cos.context.json` is injected as a cache-control block (Anthropic prompt caching). It is identical across all calls for the same universe, so it hits cache on every call after the first. This keeps per-call cost low and latency fast — only the state snapshot and directive are uncached.

### Build Tasks

- [ ] Evaluate `cos.context.json` fidelity: manually verify NPC goals, secrets, and scene descriptions are accurate enough for zero-shot briefing
- [ ] Design and iterate prompt template: system prompt + output format spec → test against 5 different campaign state snapshots
- [ ] `prep/brief.ts` — assembles prompt from state snapshot + directive, calls Anthropic SDK, streams response
- [ ] `prep/cache.ts` — wraps `cos.context.json` in a prompt cache block; measures cache hit rate
- [ ] `prep/render.ts` — parses structured brief output → renders to HTML with the six section layout
- [ ] `prep/export.ts` — save brief as standalone HTML file (printable, no external deps)
- [ ] Add "Generate Session Brief" button to `tracker.html` — passes current state snapshot to brief generator
- [ ] DM feedback: thumbs up/down on each recommended scene feeds a `feedback.json` log (local only at this stage) — foundation for future ranking tuning
- [ ] Test: generate briefs for three distinct campaign states (early game / mid-game / endgame); review for hallucinated lore or invented scene availability

### Player-Facing Brief

The DM session brief contains spoilers — scene outcomes, NPC secrets, foreshadowing. Players want their own artifact: a recap of last session and a spoiler-free quest log. This is a distinct output generated from the same campaign state, but filtered through a different prompt that withholds DM-only information.

- [ ] Define the spoiler boundary in the NSM schema: which fields are DM-only vs. player-visible
- [ ] Add a "Player View" prompt variant — same state snapshot, different system prompt, omits scene preconditions and NPC secrets
- [ ] Output: shareable HTML page (one per session) — the DM sends a link or pastes it into Discord
- [ ] Session recap section only (no recommendations, no watch list) — players get a "what happened" not a "what's coming"

**Pricing note:** `cos.context.json` is ~180 KB ≈ 45K tokens. With prompt caching, cache writes cost 25% more than base input; cache reads cost 10% of base input. After the first call per session the context is free to re-inject. At Claude Sonnet pricing this is <$0.05/brief at steady state — easily absorbable in a $5–10/month tier.

---

## Pre-Launch Gate — Fan Content Compliance Review

**Must complete before any public release of Phase 3 or later.**

This project operates under the [WotC Fan Content Policy](https://company.wizards.com/en/legal/fancontentpolicy). The policy permits non-commercial fan tools but has specific restrictions on what can be reproduced, sold, or distributed. Violating it — even accidentally — exposes the project to takedown or legal action and would destroy community trust.

- [ ] Audit all user-facing text: does any output reproduce substantial verbatim content from the CoS sourcebook? (Scene descriptions, NPC quotes, read-aloud text.) If so, replace with paraphrased or AI-generated equivalents.
- [ ] Confirm the tool itself (code, engine, UI) contains no WotC IP — only the data files reference CoS content, and those are clearly attributed
- [ ] Review AI-generated output: does the session brief or player recap reproduce copyrighted text from the module? Add a prompt instruction to paraphrase, not quote.
- [ ] Add required attribution to all public-facing surfaces: *"Unofficial Fan Content permitted under the Wizards of the Coast Fan Content Policy. Not approved/endorsed by Wizards. Curse of Strahd © Wizards of the Coast LLC."*
- [ ] Evaluate: does offering a paid tier trigger the policy's non-commercial requirement? The tool is the commercial asset; the CoS data is fan content. This distinction may need legal review before Phase 6.

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
| Engine language | **TypeScript** — browser-native, no install friction | **Resolved** |
| Primary output format | **Single-page HTML brief** — printable, no server, bookmarkable | **Resolved** |
| Input UX (Phase 3) | **Scene checklist by location/arc** + condition overrides | **Resolved** |
| Hosting model | Static HTML start → hosted web app for paid tier | Open |
| Universe file ownership | User-supplied only / community hub / built-in bundle | Open |
| Commercial structure | Freemium SaaS / one-time tool / DMs Guild hybrid | Open |
| Story Complete coupling | Tight (same schema, same repo owner) / loose (published spec) | Open |
| Mobile / tablet support | Responsive web app vs. dedicated PWA vs. desktop-only | Open — decide before Phase 3 UI build |
