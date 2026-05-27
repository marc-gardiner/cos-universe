# Contributing

## What this project accepts contributions for

This repo is the **product layer** — the tools, visualizations, and community-facing content built on top of the Barovia narrative graph. Contributions here fall into:

- New tools that consume `data/cos.serialize.json` or `data/cos.context.json`
- Improvements to the visualization (`viz/`)
- Documentation and DM-facing content
- Bug fixes and usability improvements to existing tools

## What it does not accept

Changes to the underlying narrative graph (characters, scenes, stories, conditions, rules) belong in [Story Complete](https://github.com/marc-a-gardiner/story-complete), not here. The `data/` files are generated outputs — do not edit them by hand.

## How the data gets here

The `data/` files are regenerated from Story Complete and committed to this repo when the universe has meaningful changes. If you notice the graph is stale or missing something, open an issue describing what's wrong — it will be addressed upstream.

## Fan Content Policy

All contributions must comply with the [Wizards of the Coast Fan Content Policy](https://company.wizards.com/en/legal/fancontentpolicy). This project is non-commercial. Do not submit contributions that would move it outside that boundary.

## Getting started

1. Fork the repo
2. Open `viz/cos.viz.html` in a browser to orient yourself in the narrative graph
3. Read [`docs/DATA-MODEL.md`](DATA-MODEL.md) before writing any code that consumes the graph
4. Open an issue before starting significant work — alignment up front saves rework
