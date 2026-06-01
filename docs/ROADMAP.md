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

## Phase 4 — Visualization & Analysis Layer

A suite of graph-powered views that ship as tabs inside the tracker. Each view exploits a specific edge type or node property that has no equivalent in any existing DM tool. Universe-agnostic throughout.

### NPC Relationship Map

Interactive force-directed graph of character connections, motivations, and alliances — filterable by arc, location, or faction. Driven entirely by `character` nodes and their edges.

- [ ] Identify which edge types encode character relationships in the NSM schema
- [ ] Build filterable force-directed graph view (extend `viz/cos.viz.html` or new component)
- [ ] Export: SVG/PNG for DM use at the table

### Dramatic Tension Heatmap

Score each scene by: number of active conditions at stake, `foreshadows` edges landing, `contrasts_with` alternatives available, and `rules_in_effect` complexity. Render as a heatmap overlay on the swimlane visualization. Surfaces the campaign's dramatic peaks and — critically — *dramatic deserts*: stretches of low-tension scenes that will lose players at the table.

- [ ] Define tension scoring formula across the four signal types
- [ ] Render heatmap as colour overlay on existing swimlane viz
- [ ] Allow DM to filter by arc or story to see local tension curves

### Pacing Advisor

Map `scene_type` values (`revelation`, `threshold`, `resolution`, `interlude`, `standard`) across the played campaign timeline as a waveform. Detect structural problems automatically: consecutive thresholds with no resolution (player fatigue), stretches of interludes (pacing sag), act transitions with no emotional release. Recommend scene reordering to fix detected issues.

- [ ] Compute pacing waveform from played scene sequence and scene_type values
- [ ] Define detection rules for each anti-pattern
- [ ] Render as annotated timeline with flagged problem regions and suggested fixes

### Adversarial Variant Explorer

The `contrasts_with` edges link scenes that share the same setting but produce opposite world outcomes. For every ready-to-play scene, show the DM both forks side-by-side: world state if the party succeeds vs. fails, with downstream consequences of each branch. Lets DMs prep both outcomes in one pass without re-reading the sourcebook.

- [ ] Index all `contrasts_with` edge pairs
- [ ] Build split-panel view: success branch vs. failure branch, with world_change diff
- [ ] Surface from Scene Navigator: each ready scene links to its variant explorer view

### Faction Power Tracker

Map each played scene to its impact on Barovia's political factions (Lady Wachter vs. Baron Vallakovich, the Church vs. secular power, Keepers of the Feather vs. Strahd's agents). Render a live balance chart that shifts as scenes are checked off. Makes the aggregate political consequence of party choices visible — a campaign dimension that currently lives only in the DM's head.

- [ ] Define faction tags and scene → faction impact mappings in the data layer
- [ ] Build balance chart component: per-faction power score with session-delta indicators
- [ ] Exportable snapshot for player-facing "state of Barovia" handout

---

## Butterfly Effect — Curated Scenario Backlog

Pre-computed, narrative-focused consequence scenarios for `viz/butterfly-effect-v2.html`. Each scenario presents a decision point, two or more paths, and three cascading consequence levels (That Night / The Sessions That Follow / Campaign Echo). Content is authored from graph data — `world_change` signals, scene prose, and character arcs — not live inference.

5 scenarios shipped: Doru's Fate, Vallaki's Soul, Van Richten's Allegiance, The Dark Gift, Strahd's End.

Tier 1 = direct graph branching · Tier 2 = graph-supported + canon extrapolation · Tier 3 = canon-primary

### Phase 1 — Entering the Curse ✓ (7 scenarios shipped)

### Phase 2 — Vallaki ✓ (10 scenarios shipped)

### Phase 3 — The Wilderness ✓ (9 scenarios shipped)

### Phase 4 — Krezk & Deeper Wilderness

- [ ] **Dmitri Krezkov's Gate** — Krezk is sealed: what do you offer or demonstrate to earn entrance [T1] `scene.krezk-arrival`
- [ ] **Ireena at the Blessed Pool** — something happens that cannot be undone: let it complete or intervene [T2] `scene.krezk-arrival` · `story.krezk-and-the-abbey`
- [ ] **The Abbot's Bargain** — help find a wedding gown for Strahd's intended bride in exchange for Raise Dead [T1] `scene.abbot-bargain-concluded`
- [ ] **Kasimir's Sister** — help him resurrect Patrina at the Amber Temple, knowing why she was imprisoned there [T2] `scene.amber-temple-entry` · `story.amber-temple`
- [ ] **Vladimir Horngaard** — he won't let the Order's revenants fight Strahd: convince him or work around him entirely [T1] `scene.meeting-vladimir-horngaard`
- [ ] **The Midnight Riders** — Argynvostholt's revenants patrol the roads: engage, recruit, or avoid [T2] `scene.argynvostholt-arrival`
- [ ] **Argynvost's Beacon** — returning the dragon skull lights a beacon of hope, but requires entering Ravenloft before you're ready [T1] `scene.argynvost-beacon-lit` · `condition.dragon-skull-restored`
- [ ] **The Dark Powers — What You Choose to Understand** — engage with the Temple's history of how Strahd made his bargain, or treat it as a dungeon to loot [T3] `rule.dark-powers-corruption` · `story.amber-temple`

### Phase 5 — Collecting the Artifacts

- [ ] **The Sunsword — What It Costs to Claim It** — the blade belonged to Sergei, Strahd's murdered brother: claiming it is symbolic as much as tactical [T2] `scene.retrieving-sunsword`
- [ ] **The Tome of Strahd — the Man Before the Monster** — reading it reveals his full history: does this change how you fight him [T2] `scene.retrieving-tome-of-strahd`
- [ ] **An Artifact Lost to Strahd** — the graph's highest branch score (8): how do you recover it and what does the failure cost [T1] `scene.artifact-lost-to-strahd` · `condition.all-artifacts-held`
- [ ] **Premature Castle Assault** — attacking Ravenloft before all artifacts are held: what does arriving damaged look like [T1] `scene.premature-castle-assault`
- [ ] **Lycanthropy — a Party Member Bitten** — seek a cure (the Abbot can help, at a cost), accept it, or attempt to control it [T2] `rule.werewolf-curse-transmission` · `scene.abbot-bargain-concluded`
- [ ] **The Wereravens in the Endgame** — committing the Martikov network for the Ravenloft assault at civilian risk [T2] `story.wizard-of-wines` · `story.assault-on-castle-ravenloft`

### Phase 6 — Castle Ravenloft & Endgame

- [ ] **Strahd's Dinner** — if the party attends, they learn things; but Strahd also learns things: was it worth the access [T2] `condition.strahd-invitation-received`
- [ ] **Escher's Crisis** — Strahd's consort is miserable: is there enough of him left to turn against his master [T2] `character.escher`
- [ ] **Rahadin — the History the Chamberlain Carries** — he betrayed his own people to serve Strahd for centuries: use his dusk elf history against him or just fight [T2] `character.rahadin`
- [ ] **The Vampire Brides** — Anastrasya, Ludmilla, and Volenta have distinct agendas: fight all three, negotiate with one, or turn them on each other [T2] `character.anastrasya-karelova` · `character.ludmilla-vilisevic` · `character.volenta-popofsky`
- [ ] **Barovia After — the World You Built** — the Barovia that exists after Strahd is entirely shaped by the 49 choices before it [T3] `story.assault-on-castle-ravenloft`

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

## Phase 6 — Simulation & Graph Algorithms

Pure graph-computation tools — no AI cost, no server required. Each one runs a known algorithm over the NSM and produces a result no DM has ever been able to compute before. These tools are the most technically differentiated in the roadmap.

### Speedrun / Convention Route Calculator

Run BFS from campaign start to the final confrontation nodes and find the *minimum viable scene set* — the fewest scenes that satisfy all endgame entry conditions while maintaining narrative coherence. Also computes *mandatory scenes*: nodes that appear on every possible path to the endgame. Output: a curated ordered scene list for a 4-hour one-shot, a 3-session arc, or a 10-session compressed campaign. Primary audience: convention DMs and groups with limited time.

- [ ] Implement BFS/Dijkstra over the scene graph from start nodes to endgame entry conditions
- [ ] Compute mandatory scene set (intersection of all valid paths)
- [ ] Accept a target session count as input; output a ranked scene list fitting the constraint
- [ ] Export as a printable one-shot module outline

### Butterfly Effect Analyzer

Select any scene. Compute its full downstream blast radius: conditions it satisfies or violates, stories it permanently unblocks or closes, `foreshadows` edges it resolves, and total scenes that become newly available or permanently locked. Derive an *irreversibility score* — a measure of how many doors this scene closes forever vs. opens. The `violated_by` logic makes some CoS choices one-way doors; this tool surfaces them before the DM commits.

- [ ] `engine/blast-radius.ts` — given a scene id, compute full downstream effect set via graph traversal
- [ ] Compute irreversibility score: (permanently locked scenes) / (total affected scenes)
- [ ] Render as: radial blast diagram + bullet list of consequences grouped by type
- [ ] Surface from Scene Navigator: each scene has a "see consequences" affordance

### Counterfactual / "What If" Simulator

Fork the campaign state at any played scene and recompute the world without it. Shows the DM: which conditions would be in a different state, which stories would be blocked or open, which scenes would be permanently unavailable. Powerful for post-session debrief (*"what did that choice cost us?"*) and for DMs previewing consequences before committing. Operates entirely on the engine's state logic — no AI, no external calls.

- [ ] `engine/fork.ts` — clone a CampaignState, remove a scene, recompute derived state
- [ ] Diff function: compare two CampaignState objects, output changed conditions / stories / available scenes
- [ ] Render as side-by-side: actual state vs. counterfactual state with diff highlights
- [ ] Allow chaining: fork at multiple scenes to simulate multi-step alternate histories

### Narrative Diff Tool

Two groups both run CoS. Both export `campaign-state.json`. Load both into the diff tool and compare: where did the narratives diverge? Which conditions are in different states? Which scenes did one group play that the other skipped? Output a visual divergence map showing the fork point and downstream differences. Community sharing angle: post your campaign diff on Reddit to compare runs with other groups.

- [ ] Accept two `campaign-state.json` files as input
- [ ] Compute structural diff: diverging scenes, differing conditions, exclusive stories
- [ ] Render as annotated swimlane: shared path in grey, divergent branches in colour
- [ ] Shareable static HTML export

---

## Phase 7 — AI Agent Experiences

Claude-powered tools that go beyond session prep into interactive, conversational DM assistance. Each agent is constrained by the narrative graph — it cannot invent facts that the graph says aren't active. This is the key trust differentiator vs. generic LLM DM tools.

### The Strahd Oracle

An AI agent that *plays* Strahd — but constrained entirely by current campaign state. The agent knows only what Strahd would know given active conditions and played scenes in his domain. DM inputs a situation (*"The party just stole the Sunsword — what does Strahd do?"*) and receives an in-character response grounded strictly in canon facts that are currently true. No hallucinated motivations, no invented knowledge — if the condition isn't active, Strahd doesn't know about it.

- [ ] Define Strahd's knowledge model: which conditions and scenes are in his domain of awareness
- [ ] Build prompt: Strahd's character, goals, and current knowledge state → in-character response
- [ ] Constrain output: Claude may not reference conditions that are inactive or scenes that haven't played
- [ ] Expose as a chat interface panel in the tracker — DM types situation, Oracle responds

### Session Zero Connector

Before Session 1, the DM inputs party character backgrounds and personal hooks. The tool matches each character to their optimal graph entry points: which scenes, stories, and `foreshadows` edges are most thematically resonant with their backstory. Outputs a Session Zero guide — *"Character X should care about story.st-andral's-church because it satisfies these conditions that intersect their personal arc."* Plants hooks with guaranteed graph-backed payoff.

- [ ] Build character intake form: background, motivation, personal stakes (free text per character)
- [ ] Match inputs to relevant scene/story/condition nodes via Claude + graph context
- [ ] Output: per-character hook cards + DM guide for weaving them into Session 1
- [ ] Include `foreshadows` edges as planted seeds — show DM exactly when and where they pay off

### "How Did We Get Here" Explainer

Given current campaign state, trace backwards through `satisfied_by` and `follows_from` edges to reconstruct the full causal chain in plain language. Outputs a narrative summary — *"Barovia is in political crisis because the party played scene.festival-of-blazing-sun, which satisfied condition.vallaki-political-instability, which was the entry condition for..."* — as a living campaign bible generated automatically from state alone. Doubles as a session recap for returning players or a catch-up document for new ones.

- [ ] `engine/causal-chain.ts` — backwards graph traversal from active conditions to originating scenes
- [ ] Build prompt: causal chain data → narrative summary in plain language, no jargon
- [ ] Output: tiered summary (one paragraph for players, full causal detail for DM)
- [ ] Regenerate automatically each session as state changes

---

## Phase 8 — Platform & TAM Expansion

Tools that transcend Curse of Strahd and expand the addressable market to all TTRPG modules, homebrew campaigns, and narrative design broadly.

### Alternative Universe Generator

Use the CoS NSM structure as a *narrative grammar* — the node types, edge types, condition logic, and dramatic arc proportions — and use Claude to generate a completely different campaign with the same skeleton. Same number of arcs, same structural beats, same condition logic, entirely different IP: a haunted space station, a cursed arctic expedition, a Victorian ghost mystery. The CoS universe proves the grammar works; every generated universe is a new product. This is the transition from a CoS tool to a campaign design platform.

- [ ] Extract structural template from CoS graph: arc count, scene distribution, condition density, tension curve
- [ ] Build prompt: structural template + genre/setting input → full NSM-format universe outline
- [ ] Validate output loads cleanly through the Phase 2 engine loader
- [ ] Ship as a "New Universe" wizard in the tool — DM picks a genre and gets a playable campaign skeleton

### Cross-Module Transplant Tool

Some CoS story arcs are structurally isolated — clean entry conditions, minimal coupling to the rest of Barovia. Identify these arcs programmatically by computing their graph coupling score (how many external dependencies they have). Document how to extract and transplant them into a different campaign. *"The Amber Temple arc requires only these 2 entry conditions — here's how to drop it into a homebrew campaign as a standalone dungeon."* Extends CoS content value beyond its own module.

- [ ] `engine/coupling.ts` — compute coupling score per story arc (external precondition count vs. total)
- [ ] Identify low-coupling arcs suitable for transplant
- [ ] Generate transplant guide per arc: entry conditions, world effects, NPC dependencies, what to rename
- [ ] Output as standalone module PDF/HTML per arc

---

## Phase 11 — New DM Experience Layer

The single largest underserved segment in the CoS community. New DMs face a 256-page location-indexed book with no guidance on order, pacing, or recovery. The graph solves every one of these problems deterministically — this phase builds the tools that surface those solutions for someone running CoS for the first time.

### "First 3 Sessions" Starter Pack

A curated, opinionated guided start for brand-new CoS DMs. Pre-selects the 8–10 scenes with the best new-DM experience (clear setup, forgiving outcomes, strong hooks), presents them in recommended order, and explains *why* each works. Derived from the graph's `prominence` and `scene_type` data and locked in as a reusable template — not AI-generated, hand-curated once and shipped.

- [ ] Identify the 8–10 highest-value new-DM entry scenes using prominence, scene_type, and precondition count
- [ ] Write rationale for each scene selection
- [ ] Package as a guided starter mode in the tracker: first-time launch prompts "New DM?" and pre-populates recommended path

### The Spoiler Shield

New DMs are often players who just finished the campaign and are now running it. They know the ending. Spoiler Shield is a DM mode that hides all future scene outcomes, condition resolutions, and world effects until the party reaches them — protecting the DM's sense of discovery and preventing over-telegraphing at the table. Pure UI filter over the graph, no new data required.

- [ ] Add a `spoiler-shield` mode flag to the tracker
- [ ] In this mode: scene cards show only name and setup, not outcome or world_change
- [ ] Conditions show as `[LOCKED]` until their `satisfied_by` scene is played
- [ ] Toggle off at any time without data loss

### "What Just Happened?" Scene Mapper

The party does something unexpected and the new DM has no idea what it triggered. DM types a description; the tool maps it to the nearest scene node via semantic search over scene descriptions, computes the world state change, and explains in plain language: *"That sounds like scene.bones-of-st-andral-stolen. This activates condition.church-desecrated, which unlocks these 3 scenes and permanently locks this 1."* Damage control for new DMs.

- [ ] Build semantic scene matching: free-text input → nearest scene node via embeddings or keyword index
- [ ] On match: show world_change consequences, newly unlocked and permanently locked scenes
- [ ] Offer "Apply this scene to my campaign state" confirmation button
- [ ] Fallback: if no confident match, show top 3 candidates for DM to choose

### NPC Quick-Reference Cards (Print-and-Play)

Generate a print-ready deck of index-card-sized NPC reference cards — name, one-line goal, one-line secret, voice note, current disposition — for every character likely to appear in the next session. Formatted for A6 printing, cutting, and keeping at the table. Replaces the handwritten index cards the community is already making manually, derived from the session's Scene Navigator state.

- [ ] Define "likely to appear" logic: characters linked to ready-to-play scenes
- [ ] Build card layout: name, goal, secret, voice note, current disposition (from NPC Disposition Board)
- [ ] Print-ready HTML/CSS layout — fits 4 cards per A4/Letter page
- [ ] Include blank card template for homebrew NPCs

### The "Oh No" Recovery Guide

The party killed a key NPC, skipped a critical scene, or broke the narrative in some way. DM types what happened; the tool identifies which conditions are now unresolvable, which stories are permanently blocked, and — critically — proposes narrative recovery options using `contrasts_with` alternates and substitute scenes that restore the lost beats. Turns a session disaster into a recoverable situation.

- [ ] Accept free-text description of what went wrong → map to affected condition/scene nodes
- [ ] Compute: permanently blocked stories, unresolvable conditions, downstream locked scenes
- [ ] Identify recovery options: `contrasts_with` alternates, substitute scenes with equivalent world effects
- [ ] Output: triage card with severity assessment and ranked recovery suggestions

### Session Length Estimator

Given the scenes a DM plans to run next session, estimate the session length based on scene prominence, NPC count, scene_type, and combat/roleplay balance. Flag overloaded sessions (DM has planned 5 hours of content for a 3-hour slot) and underloaded ones. Grounded in graph metadata — not a generic time formula.

- [ ] Define time estimate per scene based on prominence, scene_type, and NPC involvement
- [ ] Sum estimates for a planned scene set; show breakdown by scene
- [ ] Warn when total exceeds user-set session length target
- [ ] Suggest scenes to cut or defer when over-planned, with reasoning

### Barovia Almanac (Living World Reference)

A DM-facing reference organised by *current world state* rather than by book chapter or location. Given active conditions, shows: which NPCs are currently in which locations, which factions are ascendant, what Strahd is likely doing, what the mood of Barovia is. Replaces constant mid-session book-diving — the most disruptive new-DM failure mode. Regenerates automatically as campaign state changes.

- [ ] Map active conditions → NPC locations, faction states, world atmosphere
- [ ] Build almanac view: one page per major location, populated from current state
- [ ] "Quick reference" sidebar for mid-session lookup: NPC name → current location and goal
- [ ] Print-ready export for DMs who prefer physical reference

### The "Three Hooks" Generator

At the end of every session, the DM needs to plant 2–3 hooks for next time. New DMs forget or plant hooks that lead nowhere. This tool inspects the current graph state, finds the 3 most dramatically ripe `foreshadows` edges ready to resolve, and writes concrete hook text the DM can deliver in the final 5 minutes of the session. Each hook is grounded in a real graph edge — it will pay off.

- [ ] Compute top `foreshadows` edges by: proximity to resolution, dramatic weight, current party position
- [ ] Generate hook text per edge: one sentence the DM can deliver as narration or NPC dialogue
- [ ] Surface as an end-of-session widget: "Session ending? Here are your three hooks."

### Session Zero Player Questionnaire

Before Session 1, each player fills out a short questionnaire: themes they enjoy, themes to avoid, character arc hopes. The tool maps responses to the NSM graph and outputs a DM guide: which arcs to emphasise, which to de-emphasise, how to frame the Tarokka reading to resonate with the specific group. Ensures the campaign is personalised before the first session.

- [ ] Build player questionnaire: theme preferences, content limits, desired arc type (redemption / tragedy / triumph)
- [ ] Map responses to relevant story arcs and condition nodes
- [ ] Output DM guide: arc emphasis recommendations + Tarokka framing suggestions per player
- [ ] Shareable link so players complete the questionnaire before Session Zero

---

## Phase 12 — Immersion & Atmosphere

Tools that enhance the *feel* of the campaign at the table — not tracked mechanics, but the emotional texture that separates a memorable run from a forgettable one.

### Mood Board Generator

Given the scenes planned for next session, generate a curated set of atmosphere prompts: music mood keywords, lighting descriptions, weather, ambient sounds, and visual tone references. Not AI prose — structured prompts the DM feeds to Spotify, their VTT, or a playlist tool. Grounded in scene location, `scene_type`, and active conditions (a scene during a political crisis has different atmosphere than the same location at peace).

- [ ] Map scene + active conditions → atmosphere parameters (tone, lighting, weather, sound)
- [ ] Output as a structured card: Spotify search keywords, VTT lighting preset name, ambient descriptor
- [ ] Support VTT-specific export formats (Roll20, Foundry) where possible

### Strahd's Diary

Strahd observes everything. After each session, generate a short in-character diary entry from Strahd's perspective: what he saw, what he thinks of the party, what he's planning next. Written by Claude, strictly constrained by campaign state — Strahd only comments on events in played scenes or conditions active in his domain. No invented intelligence. Shareable as a player handout or atmospheric Discord teaser between sessions.

- [ ] Define Strahd's knowledge model: conditions and scenes within his observational domain
- [ ] Build diary prompt: Strahd's voice + knowledge-constrained facts → first-person narrative entry
- [ ] Output: 150–250 word diary entry per session, formatted as an aged parchment HTML card
- [ ] Player-safe version: omit anything Strahd wouldn't reveal (some diary entries are DM-only)

### The Prophecy Tracker

The Tarokka reading is a prophecy. Track which elements have been fulfilled, which are in progress, and which are still to come — displayed as a visual prophecy scroll that the party can reference. Fulfillment is derived from world state automatically; no manual tracking. Player-facing artifact for groups that enjoy dramatic prophecy callbacks.

- [ ] Map each Tarokka card outcome to its corresponding condition or scene in the graph
- [ ] Build prophecy scroll UI: fulfilled (with session number), in progress, yet to come
- [ ] Player-visible version with flavour text; DM version shows the underlying condition/scene
- [ ] Update automatically as campaign state changes

### Encounter Difficulty Calibrator

Given the party's current level (derived from number and prominence of scenes played) and the scenes queued in the Scene Navigator, flag scenes where the party is likely under-levelled (too few prerequisite scenes, insufficient world knowledge) or over-levelled (challenge now trivial given their power and lore). Helps new DMs avoid accidental TPKs and anti-climactic fights.

- [ ] Define party power proxy: weighted sum of played scene prominence and story completions
- [ ] Map each scene to an expected party power level
- [ ] In Scene Navigator: show difficulty indicator (appropriate / too hard / too easy) per ready scene
- [ ] Suggest scenes to play first to reach the right level before a high-stakes encounter

---

## Phase 13 — Campaign Analytics, Reporting & Advanced Tools

Post-campaign analysis, longitudinal tracking, and high-ceiling power-DM tools. Most of these require a complete or near-complete campaign state to be most useful.

### Campaign Report Card

At campaign end, generate a full stat summary: scenes played vs. total, stories completed, conditions resolved, arcs explored, paths taken, and a distinctive campaign signature. Visualised as a shareable stat card — a Spotify Wrapped for your Curse of Strahd campaign. Drives community sharing and surfaces the tool to new audiences organically.

- [ ] Define campaign completion metrics: scene coverage %, story completion %, condition resolution %
- [ ] Design stat card layout: visually striking, shareable as an image
- [ ] Generate campaign "signature" — a short evocative description of the run's unique path
- [ ] Social share export: PNG card formatted for Twitter/Discord

### "Your Campaign vs. Canon" Comparison

Show the DM how their campaign compares to the structurally "canonical" path through the module — the path that satisfies the most conditions, completes the most stories, and hits every major dramatic beat. Which arcs did they run? Which did they skip? How does their path compare to the most-played routes? (Aggregated anonymised state data in Phase 9+.) Satisfies the meta-curiosity most veteran DMs have after a long campaign.

- [ ] Compute canonical path: BFS to maximise story completion and condition satisfaction
- [ ] Diff played state against canonical: missed scenes, skipped arcs, alternate resolutions
- [ ] Render as an annotated campaign map: shared path in grey, divergences highlighted
- [ ] Frame divergences positively — different, not wrong

### "The Road Not Taken" Post-Campaign Report

After the campaign ends, generate a full report of everything the party *didn't* do: scenes never played, conditions never satisfied, stories left incomplete, arcs they never discovered. Framed as replayability, not failure — *"You discovered 62% of Barovia. Here's what remains for your next run."* Drives replay and re-engagement with the tool.

- [ ] Compute inverse of played state: unplayed scenes, unsatisfied conditions, incomplete stories
- [ ] Group by arc and story for readability
- [ ] Frame each missed element with a one-line hook — why it would have been interesting
- [ ] Include "replay starting point" suggestion: which scene to start from to pick up missed threads

### The Living Strahd Timeline

Strahd doesn't wait for the party. Model a parallel timeline of what Strahd is doing between sessions based on conditions and scenes played. If the party takes too long, certain conditions resolve unfavorably on a timer. Visualise as a shadow timeline running alongside the campaign with countdown indicators. Adds urgency and consequence without the DM inventing off-screen events.

- [ ] Define time-pressure conditions: conditions that degrade or resolve without player intervention
- [ ] Build timeline model: session count → Strahd action triggers
- [ ] Visualise as a parallel track on the campaign timeline with upcoming trigger warnings
- [ ] DM toggle: enable/disable time pressure per condition (not all campaigns want the clock)

### Alternate Ending Calculator

CoS has multiple possible endings depending on artifact acquisition and ally choice. Given current state, show the DM which endings are still reachable, which are permanently locked out, and what the party needs to do to access each remaining ending. Helps DMs steer toward a dramatically satisfying conclusion without railroading — informed guidance, not scripted outcome.

- [ ] Map each ending to its prerequisite conditions and scenes
- [ ] Given current state: compute reachable endings, locked endings, and minimum scenes required per reachable ending
- [ ] Display as an "endings board" — reachable in green, locked in grey, in-progress in amber
- [ ] Show the delta: "2 more scenes to unlock Ending B"

### DM Notes Auto-Organiser

The DM writes free-form session notes after each session. This tool parses the notes, identifies which scenes were played (keyword matching + semantic search against scene descriptions), and automatically updates the campaign state. DMs who already write session notes don't need to do double data entry — the tracker stays current from the notes they were already writing.

- [ ] Build scene extraction pipeline: free-text notes → candidate scene matches with confidence scores
- [ ] DM confirms or corrects matches before state is updated
- [ ] Extract NPC mentions and disposition notes for the NPC Disposition Board
- [ ] Support common note formats: plain text, Markdown, Notion export

### "What Would a Good DM Do?" Coach

A coaching mode for new DMs. Given the current campaign state and a situation the DM describes, suggest 2–3 options for handling it — ranked by dramatic impact, player fairness, and narrative coherence — with reasoning grounded in the graph. Not a replacement for DM judgment: a thinking partner that helps new DMs build their instincts. Framed explicitly as coaching, not instruction. The DM always decides.

- [ ] Build coaching prompt: current state + DM situation description → 2–3 ranked options with reasoning
- [ ] Constrain output to graph-grounded options: suggested actions must map to real scene or condition nodes
- [ ] Include "why this works narratively" explanation per option — teaches, not just advises
- [ ] Add to tracker as an optional coaching panel, off by default (experienced DMs don't need it)

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

## Phase 9 — Monetisation & Distribution

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

## Phase 10 — Community Features

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
