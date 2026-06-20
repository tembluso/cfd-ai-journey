# Point Sensor Verification — Notes

## What Was Done

Three initializations of the point-sensor inverse PINN (Model B) at $\kappa_0 = 10^{-9}$, $10^{-8}$, and $3.93 \times 10^{-8}\ \text{m}^2/\text{s}$. Each run was trained for 1.7–2.0 million iterations via sequential warm restarts (save final params, reload, continue with fresh Adam optimizer). The model places TEM-A at $z = 9.4\ \text{cm}$ (Spohn's diurnal representative depth).

## Results

All three trajectories drift into the $1.5$–$1.8 \times 10^{-8}\ \text{m}^2/\text{s}$ region but never plateau. Final values: 1.71, 1.65, and $1.73 \times 10^{-8}$, all still drifting downward at termination (~$-0.1$%/10k iters).

The loss has no minimum — it decreases monotonically as $\kappa$ decreases. Both physics and data loss decrease in tandem, so nothing acts as a restoring force.

## Why It Fails

Diagnosed analytically: the amplitude ratio and phase lag at $z = 9.4\ \text{cm}$ demand different $\kappa$ values ($3.93 \times 10^{-8}$ vs $2.12 \times 10^{-8}$, a 440% inconsistency). No single $\kappa$ can satisfy both. This is because TEM-A measures a spatial average over the full mole length, not a point value — the amplitude and phase of that average don't correspond to any single depth in the analytical solution.

## Role

Serves as a negative control for Model A. The same PINN architecture and training procedure, applied to an inconsistent forward model, produces drift instead of convergence. See `point_sensor_conclusions.pdf` for full figures and analysis.
