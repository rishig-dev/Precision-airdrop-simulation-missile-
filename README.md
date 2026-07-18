# 🎯 Precision Airdrop Simulator (JPADS-style)

A self-contained simulation of a **Joint Precision Airdrop System (JPADS)** concept:
a drone/aircraft plans a route to a drop zone, computes the **CARP (Computed Air
Release Point)** so wind drift carries the payload toward the target, and simulates
a guided parafoil correcting itself during descent to land on target.

No external dataset or API key is required — the notebook generates its own
synthetic terrain + wind-field dataset on first run and saves it as CSV files.

---

## What this simulates

1. **Terrain + wind dataset generation** — a synthetic 40×40 grid (10km × 10km)
   with elevation, wind speed/direction, and no-fly obstacle zones.
2. **A\* path planning** — the drone plans a route to the target, penalizing
   steep terrain and headwind, and avoiding no-fly zones.
3. **CARP calculation** — the core JPADS math:

   ```
   time_of_fall = altitude / descent_rate
   drift        = wind_vector * time_of_fall
   CARP         = target_position - drift
   ```

4. **Guided parafoil descent simulation** — the payload falls from the CARP,
   drifting with the wind while also steering itself toward the target (a
   simplified homing guidance law), producing a realistic miss distance.
5. **Visualization** — a static matplotlib mission plot, and an interactive
   Folium map anchored to real-world latitude/longitude.
6. **Mission summary metrics** — wind conditions, time of fall, drift, and
   final miss distance.

---

## Project structure

```
precision-airdrop-simulator/
├── README.md
├── LICENSE
├── requirements.txt
├── .gitignore
├── notebooks/
│   └── precision_airdrop_simulation.ipynb   # main simulation, run this
└── data/
    └── (generated CSVs land here when the notebook runs: terrain_grid.csv,
        wind_field.csv, obstacles.csv, mission_points.csv)
```

---

## Getting started

### Option A — Google Colab (recommended, zero setup)
1. Go to [colab.research.google.com](https://colab.research.google.com).
2. `File → Upload notebook` → select `notebooks/precision_airdrop_simulation.ipynb`.
3. `Runtime → Run all`.

### Option B — Run locally
```bash
git clone <your-fork-url>
cd precision-airdrop-simulator
pip install -r requirements.txt
jupyter notebook notebooks/precision_airdrop_simulation.ipynb
```

Total runtime is a few seconds; everything is generated/computed in-notebook.

---

## Key parameters (tweak these in the notebook)

| Parameter | Meaning | Default |
|---|---|---|
| `GRID_N` | Grid resolution (cells per side) | 40 |
| `CELL_SIZE_M` | Real-world size of one grid cell | 250 m |
| `DROP_ALTITUDE_M` | Release altitude AGL | 600 m |
| `DESCENT_RATE_MS` | Parafoil vertical descent rate | 5.5 m/s |
| `GLIDE_RATIO` | Guided parafoil forward glide ratio (L/D) | 3.0 |
| `start_cell` / `target_cell` | Mission launch point / drop zone | (2,2) / (32,30) |
| `CENTER_LAT` / `CENTER_LON` | Real-world map anchor for the Folium map | configurable |

---

## Known limitations

This is an educational/conceptual simulation, not a certified airdrop planning
tool. Current simplifications:

- Wind is sampled once at the target cell rather than varying with altitude
  (real systems must account for wind shear across the fall).
- Path planning targets the drop zone directly rather than planning toward
  the CARP itself.
- Terrain is procedurally generated, not real elevation (DEM) data.
- The descent guidance law is a simplified proportional homing controller,
  not a full parafoil flight-dynamics model.
- No Monte Carlo accuracy sweep (real JPADS accuracy figures like CEP are
  derived from many repeated trials under wind uncertainty).

## Extending to real data / hardware

Each synthetic piece maps to a real source:

- **Terrain** → SRTM/DEM data via `rasterio` or the Google Elevation API.
- **Wind field** → NOAA HRRR or Open-Meteo forecast APIs.
- **No-fly zones** → real restricted-airspace polygons (e.g. FAA UAS Facility Maps).
- **Drone telemetry** → MAVLink / ROS2 topics from a Pixhawk-based drone.
- **Guidance law** → a real parafoil autopilot's PID/L1 controller.

As long as replacements produce `terrain`, `wind_speed`, `wind_dir`, `obstacles`,
`start_cell`, and `target_cell` in the same shapes, the rest of the notebook
(A*, CARP math, visualization) works unchanged.

---

## License

MIT — see [LICENSE](LICENSE).
