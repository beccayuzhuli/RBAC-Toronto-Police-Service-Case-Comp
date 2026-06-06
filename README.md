# 🚇 Transit Safety Analysis
**RBAC × Toronto Police Service Case Competition | Winter 2026 | Final Round**

---

## 📌 Project Overview

Do large public events (concerts, sports games, festivals) cause a measurable increase in crime risk near Toronto transit hubs?

This project builds a **causal inference framework** to quantify event-driven crime uplift across the TTC network, and translates findings into an **operational patrol strategy** for TPS — with direct applications to the 2026 FIFA World Cup.

---

## 🗂️ Repository Structure

```
├── TTC Routes and Schedules Data/
│   ├── agency.txt                # TTC agency metadata (GTFS)
│   ├── calendar.txt              # Service calendar (GTFS)
│   ├── calendar_dates.txt        # Service calendar exceptions (GTFS)
│   ├── routes.txt                # Route definitions (GTFS)
│   ├── shapes.txt                # Route shape geometry (GTFS)
│   ├── stops.txt                 # Stop locations (GTFS)
│   ├── stop_times.txt            # Stop arrival/departure times (GTFS)
│   ├── trips.txt                 # Trip definitions (GTFS)
│   ├── Event_Dataset.xlsx        # Manually compiled large event records
│   └── Major_Crime_Indicators_Data.csv  # TPS MCI crime records (2014–Q3 2025)
├── TPS_Analysis25.ipynb          # Main analysis notebook
├── tps_analysis.py               # Core analysis script
└── README.md
```

---

## 📊 Datasets

| Dataset | Source | Description |
|---|---|---|
| TPS Major Crime Indicators | [TPS Public Safety Data Portal](https://data.torontopolice.on.ca/) | 452K+ crime records, 2014–Q3 2025; lat/lon, offence type, timestamp |
| TTC GTFS | [City of Toronto Open Data](https://open.toronto.ca/) | Full stop-level transit network — subway, streetcar, bus |
| Event Dataset | `TTC Routes and Schedules Data/Event_Dataset.xlsx` | ~10 years of events at BMO Field, Rogers Centre, Scotiabank Arena, RBC Amphitheatre; event time, type, and venue |
| TTC GTFS Files | `TTC Routes and Schedules Data/*.txt` | agency, calendar, routes, shapes, stops, stop_times, trips |

---

## 🔬 Methodology

### 1. Data Integration
- **Crime → TTC mapping:** Each crime incident assigned to its nearest TTC stop via haversine distance
- **Event → TTC mapping:** Events linked to *anchor stations* via the TTC network graph; influence propagates outward along transit lines (not straight-line distance)

### 2. Exposure Ring Construction
Each TTC stop is assigned to a ring based on network hops from event anchor stations:
- **Tier 1 (0–25%):** Immediate entrances, fare gates, bus bays
- **Tier 2 (25–50%):** Adjacent stops, nearby transfer points
- **Tier 3 (50–100%):** Dispersal corridors and walking links
- **Control:** Network-distant stops

### 3. Causal Inference — Bootstrap Difference-in-Differences
- **Panel unit:** event × relative hour × exposure ring
- **Outcome:** crime rate = crime count / # stops in ring (comparable across rings)
- **DiD estimand:** Δ_ring(t) = treated ring avg − control ring avg
- **Inference:** Event-level bootstrap resampling (preserves within-event correlation across hours and modes); 95% CIs from empirical percentile bounds
- Standard regression was ruled out due to sparse outcomes, high-dimensional fixed effects, and interpretability concerns

### 4. Key Findings
- Crime frequency is **elevated from T−1h through T+4h** near event venues
- **Spatial decay of 40–60%** from Tier 1 to Tier 3 — effect is highly localized
- Effect **dissipates sharply after T+4h**, consistent with crowd dispersal
- Bootstrap 95% CIs exclude zero during the critical window → **reject H₀**

---

## 🚔 Operational Recommendations

**Event Shock Scenario** — 3-phase patrol deployment:

| Phase | Window | Focus |
|---|---|---|
| Pre-Stage | T−24h to T−1h | Align with TTC/venue security; stage mobile spillover unit |
| Critical Window | T−1h to T+2h | Fixed coverage at Tier 1; roving at Tier 2; monitoring at Tier 3 |
| Extended Window | T+2h onward | Shift to egress; prioritize queue locations and delayed-service stops |

**Daily Routine Scenario** — Hub-and-spoke model with 15–25 min pulse patrols; mobile rovers across 3–5 station clusters; 14-day rolling trend for rotating Tier 2 coverage.

---

## ⚠️ Limitations
- Seasonal effects and venue capacity differences not explicitly modeled
- Concurrent events treated independently — compound effects not isolated

## 🔭 Future Directions
- Add seasonal fixed effects and attendance/capacity proxies
- Multi-event exposure modeling using overlapping treatment indicators

---

## 🛠️ Tools & Environment
- **Python:** pandas, geopandas, numpy, scipy, matplotlib, folium
- **References:** [TPS Public Safety Data Portal](https://data.torontopolice.on.ca/) · [City of Toronto Open Data](https://open.toronto.ca/)
