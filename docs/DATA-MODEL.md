# Data Model Reference

The narrative graph is produced by [Story Complete](https://github.com/marc-a-gardiner/story-complete) using the Narrative State Machine (NSM) architecture. Two files are published here:

| File | Purpose | Size |
|---|---|---|
| `data/cos.serialize.json` | Full graph — all nodes, all edges, all fields | ~1.4 MB |
| `data/cos.context.json` | LLM-optimised authoring context — structured for prompt injection | ~181 KB |

---

## Node Types

Every node has a `node_type` field. The types present in this universe:

| Type | What it represents | Count |
|---|---|---|
| `character` | An NPC or party-level actor | 31 |
| `location` | A place where interactions occur | 25 |
| `object` | A capability, item, or mechanism the narrative turns on | 15 |
| `scene` | An atomic interaction — inputs, outputs, adversarial variants | 75 |
| `story` | A user journey — ordered sequence of scenes with an emotional arc | 19 |
| `arc` | A thematic grouping of stories (epic-level) | 6 |
| `rule` | A world constraint (NFR) that governs scene outcomes | 7 |
| `condition` | A named world-state fact — true or false at any point in the campaign | 12 |

---

## Key Fields

### Scene nodes

```json
{
  "id": "scene.tarokka-reading",
  "node_type": "scene",
  "scene_type": "revelation",
  "primary_character": "character.madam-eva",
  "location": "location.tser-pool-encampment",
  "object": "object.tarokka-deck",
  "preconditions": [
    { "description": "Party has reached Tser Pool", "source": "scene.finding-madam-eva" }
  ],
  "world_change": [
    { "state": "holy-symbol-location-known", "value": true },
    { "state": "sunsword-location-known", "value": true },
    { "state": "tome-location-known", "value": true },
    { "state": "key-ally-identified", "value": true }
  ],
  "connections": {
    "follows_from": ["scene.finding-madam-eva"],
    "leads_to": ["scene.interpreting-the-reading"]
  }
}
```

`world_change` entries are the key field for the campaign state tracker — they declare what becomes true in the world when a scene is played.

### Condition nodes

```json
{
  "id": "condition.sunsword-location-known",
  "node_type": "condition",
  "connections": {
    "satisfied_by": ["scene.tarokka-reading"],
    "violated_by": []
  }
}
```

Conditions are the world-state predicates. A condition is **active** when at least one of its `satisfied_by` scenes has been played and none of its `violated_by` scenes have been played since.

### Story nodes

```json
{
  "id": "story.collect-the-artifacts",
  "node_type": "story",
  "scenes": [
    "scene.locating-holy-symbol",
    "scene.retrieving-sunsword",
    "scene.retrieving-tome-of-strahd",
    "scene.artifacts-assembled"
  ],
  "entry_conditions": ["condition.holy-symbol-location-known"],
  "world_effects": ["condition.all-artifacts-held"]
}
```

`entry_conditions` tells you what must be true before a story can begin. `world_effects` tells you what becomes true when the story is complete.

---

## Connection Types

Edges between nodes carry typed semantics:

| Type | Meaning |
|---|---|
| `follows_from` | Causal dependency — this node cannot occur without the source having occurred |
| `leads_to` | Informational inverse — what this node enables |
| `rules_in_effect` | Which world rules govern this scene |
| `satisfied_by` | Which scenes produce this condition |
| `violated_by` | Which scenes un-satisfy this condition |
| `contrasts_with` | Same character + location + object, different outcome |
| `foreshadows` | Planned but not yet authored — target may not exist |

---

## Working With the Data

### Find all scenes with satisfied preconditions given a played set

```python
import json

with open("data/cos.serialize.json") as f:
    graph = json.load(f)

played = {"scene.finding-madam-eva", "scene.village-of-barovia-arrival"}
scenes = [n for n in graph["nodes"] if n["node_type"] == "scene"]

def preconditions_met(scene, played):
    sources = [p.get("source") for p in scene.get("preconditions", []) if p.get("source")]
    return all(s in played for s in sources)

ready = [s["id"] for s in scenes if preconditions_met(s, played) and s["id"] not in played]
print(ready)
```

### Find all active conditions given a played set

```python
conditions = [n for n in graph["nodes"] if n["node_type"] == "condition"]

def is_active(condition, played):
    satisfied_by = condition.get("connections", {}).get("satisfied_by", [])
    violated_by = condition.get("connections", {}).get("violated_by", [])
    ever_satisfied = any(s in played for s in satisfied_by)
    currently_violated = any(s in played for s in violated_by)
    return ever_satisfied and not currently_violated

active_conditions = [c["id"] for c in conditions if is_active(c, played)]
print(active_conditions)
```

---

## Regenerating

See the [README](../README.md#regenerating-the-data) for the commands to regenerate both files from Story Complete.
