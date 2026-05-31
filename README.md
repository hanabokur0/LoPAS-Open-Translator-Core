# LoPAS Open Translator Core

> Translate raw open data into a common state space.

---

## What This Is

The world is full of open data.  
NOAA, CMEMS, FIRMS, GDELT, GBIF, Sentinel, AIS — all public, all free.  
But they speak different languages. Different formats, different update cycles, different structures.

This project is a **translation protocol**.  
Not an analysis tool. Not an AI. A schema.

```
Raw Data
↓
Translator
↓
Common State  →  SAFE / CAUTION / WARNING / CRITICAL
↓
LoPAS Indicators
↓
Action  →  HOLD / MONITOR / PREPARE / MOVE
```

Once translated, ocean data, wildfire data, ecosystem data, and geopolitical data  
can all be read in the same interface, compared on the same scale, and acted on with the same logic.

---

## Design Principles

### 1. Coordinate First
Every observation must be reducible to:
```yaml
lat:
lon:
timestamp:
```

### 2. Treat Async as Signal
Different data sources update at different rates.  
The gap between fast and slow data is not noise — it is information.

```yaml
lag_seconds:      # how stale is this data?
coverage_gap:     # what are we not seeing?
```

### 3. State Before Data
Generate state before surfacing raw values.  
`SAFE` is more actionable than `wave_height: 2.3m`.

### 4. No LLM Dependency in Core
Translators are deterministic.  
LLMs sit above as an interpretation layer, not inside the translation logic.  
The schema is the asset. APIs change. Schemas persist.

---

## State Model

All Translators output to this shared state space.

```yaml
state:
  status:   SAFE | CAUTION | WARNING | CRITICAL
  risk_score: 0.0–1.0
  trend:    IMPROVING | STABLE | DEGRADING | UNKNOWN
```

## Action Model

```yaml
action:
  recommendation: HOLD | MONITOR | PREPARE | MOVE | ESCALATE
  confidence: 0.0–1.0
```

---

## Core Schema

```yaml
meta:
  translator:       # e.g. ocean-translator
  version:          # e.g. 0.1.0
  timestamp:        # ISO 8601
  lag_seconds:      # seconds since oldest source update
  coverage_gap:     # true/false — missing observation zones
  source_count:     # number of active sources
  confidence:       # 0.0–1.0
  uncertainty:      # 0.0–1.0
  anomaly:          # true/false

coordinate:
  lat:
  lon:
  timestamp:

sources:
  - provider:       # e.g. CMEMS
    source_id:
    update_interval: # seconds
    confidence:

observation:
  # domain-specific raw fields

derived:
  # computed fields from observation

state:
  status:
  risk_score:
  trend:

lopas:
  # domain-specific LoPAS indicators
  # minimum required: RAR, SCI, CPTI

action:
  recommendation:
  confidence:
```

---

## LoPAS Indicators

This schema connects to [LoPAS](https://github.com/hanabokur0/LoPAS-Core) —  
a scale-invariant observation framework.

Core indicators used across all Translators:

| Indicator | Role in translation |
|-----------|-------------------|
| RAR | Reality Alignment Rate — how well predictions match reality |
| SCI | System Crisis Index — overall system stress |
| CPTI | Cognitive Phase Transition Index — early warning of state change |
| CDI | Concentration-Distribution Index — detecting dangerous centralization |
| OSI | Observer Slippage Index — detecting over-reliance on stale data |

---

## Extended Indicators (v0.2 candidates)

Three new indicators identified during schema design:

```yaml
LAG:  # Latency Alignment Gap
  definition: Measures misalignment between fast and slow data sources.
  signal: High LAG = something may be happening that slow data hasn't captured yet.

COV:  # Coverage Omission Value
  definition: Measures geographic blind spots in observation coverage.
  signal: High COV = decisions being made without seeing part of the picture.

SIL:  # Simultaneous Silence Index
  definition: Detects when multiple sources go quiet at the same time.
  signal: High SIL = anomaly. Silence is not absence of events.
```

---

## Translator Repositories

| Translator | Data Sources | Domain |
|-----------|-------------|--------|
| [Ocean-Translator](../Ocean-Translator) | CMEMS, NOAA GFS, Open-Meteo | Maritime safety, node control |
| [Fire-Translator](../Fire-Translator) | FIRMS, USGS, Open-Meteo | Wildfire spread prediction |
| [Eco-Translator](../Eco-Translator) | GBIF, Sentinel, EM-DAT | Ecosystem vulnerability |
| [Geo-Maritime-Translator](../Geo-Maritime-Translator) | GDELT, AIS, CMEMS | Geopolitical maritime risk |

---

## Philosophy

This is not a product. It is infrastructure.  
Anyone can build a Translator. No one owns the Core.

The schema is the commons.  
APIs are ephemeral. Structure persists.

---

*LoPAS Open Translator Core v0.1 — built from observation, not specification.*
