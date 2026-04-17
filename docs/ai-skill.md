---
name: sg-replay-analysis
description: Use only when the current working directory is inside a folder named `sg` and the task is to analyze SolidGames / sg.zone Arma replay links, OCAP JSON files, attack-defense routes, NPP objective markers, unit approach timing, losses, vehicle roles, or to prepare a tactical review or plan image for SG missions.
---

# SG Replay Analysis

Use this skill only if `pwd` is inside a project folder named `sg`. If not, stop and say this skill is restricted to `sg`.

This skill is for quick analysis of `sg.zone/replays/...` links and downloaded OCAP JSON from SolidGames Arma replays.

## What this skill covers

- fetch replay pages and OCAP JSON
- extract mission metadata and objective markers
- compare attack vs defense behavior
- measure approach to objective by distance rings
- review red attack routes, losses, and pressure timing
- review vehicle role at a positional level
- prepare a short tactical brief

## Quick workflow

1. Confirm you are inside `sg`.
2. Save replay inputs and raw JSON under `analysis/raw/`.
3. Use `scripts/fetch_replays.py` first when the user gives replay URLs.
4. Use `scripts/summarize_replays.py` for a fast multi-replay summary.
5. Use `scripts/render_attack_plan.py` when the user wants a plan image.
6. Read `references/heuristics.md` before drawing conclusions.
7. Build the answer around:
   - objective and victory condition
   - approach sectors
   - entries into `1000m`, `500m`, `250m`
   - losses before `1500m`
   - vehicle timing and survivability
8. Keep conclusions concrete. Prefer “what worked / what failed / what to change”.

## Inputs to collect

- replay URLs or local OCAP JSON files
- which side is attack
- objective marker or screenshot if objective is not obvious from markers
- whether the user wants:
  - summary
  - tactical review
  - vehicle review
  - image / plan diagram

## Fetching

When given replay URLs, run:

```bash
python3 .codex/skills/sg-replay-analysis/scripts/fetch_replays.py --output analysis/raw <url1> <url2> ...
```

This creates:

- `analysis/raw/replays.json`
- one HTML file per replay
- one OCAP JSON file per replay when available

## Fast summary

When replay JSON files are already present, run:

```bash
python3 .codex/skills/sg-replay-analysis/scripts/summarize_replays.py --output analysis/reports/summary.json analysis/raw/*.json
```

This writes:

- machine summary JSON
- human-readable Markdown next to it

## Plan image

When the user wants a plan image on a real Sahrani map, run:

```bash
python3 .codex/skills/sg-replay-analysis/scripts/render_attack_plan.py \
  --output analysis/reports/attack-plan \
  --objective 6552.12 5600.56 \
  --label "План атаки НПЗ"
```

Optional overlays:

- `--ring 250 --ring 500 --ring 1000 --ring 1500 --ring 2000`
- `--arrow x1 y1 x2 y2 color label`
- `--box x y w h title body`

This writes:

- `attack-plan.svg`
- `attack-plan.png`
- downloaded map tiles under a sibling folder

## Analysis rules

- Treat replay HTML as metadata only; the useful data is in the OCAP JSON.
- Prefer `EditorMarkers` for NPP zone and key objects such as `mrk_oil`, `npz1`, `npz2`, `npz3`.
- Use `events` for kills and mission result.
- Use unit `positions` to measure distance-to-objective over time.
- Separate route logic from kill counts. A route is good only if it helps enough players reach the last `500m` / `250m`.
- Vehicle kill counts are often incomplete at the vehicle-entity level. Treat vehicle analysis primarily as positional and timing analysis unless crew linkage is reconstructed.

## Output shape

Default answer shape:

1. short verdict
2. strongest pattern
3. biggest failure mode
4. 3-5 practical recommendations

If the user wants a written artifact, save it under `analysis/reports/`.

## References

- For conclusions and thresholds: read [references/heuristics.md](references/heuristics.md)
- For a compact report structure: read [references/report-structure.md](references/report-structure.md)
