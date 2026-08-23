# CLAUDE.md — LoPAS-Open-Translator-Core

Loaded automatically by Claude Code at session start. This repo is **a schema, not an analysis tool or an AI**. Its job is translating heterogeneous open data (NOAA, CMEMS, FIRMS, GDELT, GBIF, Sentinel, AIS) into one common state space.

## One-line summary

```
Raw Data → Translator → Common State (SAFE/CAUTION/WARNING/CRITICAL) → LoPAS Indicators → Action (HOLD/MONITOR/PREPARE/MOVE)
```

Once translated, ocean/wildfire/ecosystem/geopolitical data can be read on the same scale and acted on with the same logic.

## Directory map

| Path | Role |
|---|---|
| `Core Schema.yaml` | The shared schema every Translator must output to (`meta`, `coordinate`, `sources`, `observation`, `derived`, `state`, `lopas`, `action`) |
| `Ocean Translator`, `Fire Translator`, `Eco Translator`, `Geo-Maritime Translator` | Domain-specific translators, each mapping its own raw sources into Core Schema |

## Design principles (do not violate these when adding a Translator)

1. **Coordinate first** — every observation must reduce to `lat`/`lon`/`timestamp`.
2. **Treat async as signal** — `lag_seconds` and `coverage_gap` are first-class fields, not noise to hide. The gap between fast and slow sources is information.
3. **State before data** — surface `SAFE`/`CAUTION`/`WARNING`/`CRITICAL` before raw values. `SAFE` is more actionable than `wave_height: 2.3m`.
4. **No LLM dependency in core** — Translators must be deterministic. An LLM may sit above as an interpretation layer, never inside the translation logic itself. The schema is the durable asset; APIs churn, schemas persist.

## LoPAS indicators used here — naming note

This repo already uses **`SCI` = System Crisis Index**, consistent with the LoPAS-SEED v1.19+ registry rename (not the older "Structural Collapse Index" wording). Minimum required indicators per Translator: `RAR`, `SCI`, `CPTI`. If you pull in `DoQ`, `RDI`, or `BCDI` from SEED, disambiguate as `-S` (semantic) vs `-R` (router) per the Master Indicator Registry — this repo does not yet specify which variant it expects, so ask before assuming.

Extended indicators (v0.2 candidates, not yet stabilized): `LAG` (Latency Alignment Gap), `COV` (Coverage Omission Value), `SIL` (Simultaneous Silence Index).

## Quick task recipes

**"Add a new Translator for [domain]"** → start from `Core Schema.yaml`, keep `lat/lon/timestamp` at the root, populate `sources[]` with `provider`/`update_interval`/`confidence` per feed, and compute `state.status` deterministically — no LLM call inside this path.

**"Why is this state CRITICAL?"** → trace `lopas.RAR`, `lopas.SCI`, `lopas.CPTI` plus `meta.confidence`/`meta.anomaly`/`meta.coverage_gap`. A CRITICAL state driven by high `coverage_gap` means "we can't see enough," not "we're certain it's bad" — say which one it is.

**"Compare across domains"** → this is the whole point of the Core Schema: ocean/fire/eco/geo states are comparable once translated. Don't compare raw domain-specific fields directly.

## Relationship to other repos in this ecosystem

Part of a wider set: `information-compost`, `lopas-protocol-foundry`, `classification-simulation-pack`, `Verifiable-Capability-Exchange`, `LoPAS-LCA`. This repo's Translators feed `state`/`action` into LoPAS-Core; it does not itself implement Protocol Foundry-style candidate promotion or Capability Exchange routing. If a task needs those, say so rather than assuming this repo covers it.
