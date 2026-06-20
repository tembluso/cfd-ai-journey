# Week 12: Statistical Verification of Inverse Results

## Overview

Weeks 7–11 produced two measurement models for estimating Martian soil thermal diffusivity $\kappa$ from InSight HP³ data:

- **Model A (Spatial Average):** Treats TEM-A as measuring a depth-weighted average of $T_{1D}$ over the mole extent.
- **Model B (Point Sensor):** Treats TEM-A as measuring temperature at a single depth ($z = 9.4\ \text{cm}$).

Both were previously tested with a single initialization each. This week runs each from **three different starting values** spanning two orders of magnitude, with sequential warm restarts (save final params, reload, continue with fresh optimizer). The goal: determine which model has a well-defined convergence basin and which does not.

---

## Reading Order

### 1. `Spatial_Average_Verification/`

- `spatial_average_conclusions.md` — summary of results
- `spatial_average_convergence_conclusions.pdf` — full analysis with figures
- `verify-inverse-spatial-average.ipynb` — training and analysis notebook

**Result:** Grand mean $\kappa = 6.278 \times 10^{-8}\ \text{m}^2/\text{s}$, cross-run spread 0.49%. Three initializations bracket from opposite directions. Analytically confirmed (PINN/analytical = 0.99).

### 2. `Point_Sensor_Verification/`

- `point_sensor_conclusions.md` — summary of results
- `point_sensor_conclusions.pdf` — full analysis with figures
- `verify-inverse-point-sensor.ipynb` — training and analysis notebook

**Result:** All three trajectories drift into 1.5–1.8 $\times 10^{-8}$ but never plateau after ~2M iterations. No loss minimum. 440% analytical inconsistency between amplitude and phase observables. Serves as negative control.

---

## Summary

| | Model A (Spatial Average) | Model B (Point Sensor) |
|---|---|---|
| Grand mean $\kappa$ ($\times 10^{-8}$) | 6.278 | ~1.7 (still drifting) |
| Cross-run spread | 0.49% | Not converged |
| Analytical confirmation | Yes (ratio 0.99) | No (440% inconsistency) |
| Loss minimum | Yes | No (monotonic decrease) |

---

## Data Files

Each subdirectory contains `.zip` archives with `.pkl` checkpoint files from cloud GPU runs. These are ignored by `.gitignore`. The notebooks load these checkpoints to produce the analysis figures.

---

## Connection to the Paper

Model A's convergence and analytical consistency establish $\kappa \approx 6.3 \times 10^{-8}$ as a robust PINN estimate under the spatial-average forward model. Model B's failure serves as a negative control — the same method applied to an inconsistent forward model does not converge. The 59.7% gap between Model A and Spohn's value is explained by the thermal fin effect.
