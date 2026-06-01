# Changelog

## Unreleased

### Added

#### Butterfly Effect — Curated Scenario Backlog (Phase 1 & 2)

17 new scenarios added to `viz/butterfly-effect-v2.html`, organized into phase-grouped sections alongside the original 5 featured scenarios. The grid now renders three labeled sections: Featured, Phase 1 — Entering the Curse, and Phase 2 — Vallaki.

**Phase 1 — Entering the Curse (7 scenarios):**
- Rose and Thorn's Request — possession choice at Death House
- Accepting Barovia — fighting the mists vs. conserving resources
- Strahd's First Invitation — accept the carriage or refuse
- The Promise to Ireena — constant escort, strategic positioning, or nominal protection
- Ismark's Ask — does he travel to Vallaki or stay to guard the village
- The Fifth Card — which fated ally the Tarokka names (Ezmerelda, Van Richten, or Ireena)
- Strahd at Tser Pool — intervene, observe, or retreat when he visits Ireena

**Phase 2 — Vallaki (10 scenarios):**
- St. Andral's Bones — recover before the festival or lose the church sanctuary
- The Festival of the Blazing Sun — controlled outcome or chaos
- The Vallaki Stockade — quiet escape vs. burning the room
- Blinsky's Dolls — follow the commission to Izek or note and move on
- Izek Strazni's Obsession — confront or weaponize the fixation
- The Coffin Maker — public punishment or release with mercy
- Arrigal — the Spy in the Camp — expose to Luvash or run as a double agent
- Arabelle — What Happens After — return to Luvash or keep with the party
- Rictavio Unmasked — the Arabelle Fork — which crisis comes first
- The Wachterhaus Cult — expose, leverage, or ignore Lady Wachter's secret

**UI change:** `renderGrid()` now groups scenarios by phase with labeled section headers. CSS added for `.scenario-phase` and `.phase-label` to space and style the sections.
