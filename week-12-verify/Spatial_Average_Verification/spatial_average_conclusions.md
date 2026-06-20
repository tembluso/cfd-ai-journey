# Spatial Average Verification — Notes

## What Was Done

Three initializations of the spatial-average inverse PINN (Model A) at $\kappa_0 = 10^{-8}$, $3.93 \times 10^{-8}$, and $10^{-7}\ \text{m}^2/\text{s}$. Each run was trained via sequential warm restarts (save final params, reload, continue with fresh Adam optimizer) for 350k–900k total iterations.

## Results

All three converge to the same basin:

| $\kappa_{\text{init}}$ | Total iters | Last-10% mean $\kappa$ ($\times 10^{-8}$) |
|---|---|---|
| $1 \times 10^{-8}$ | 350k | 6.277 |
| $3.93 \times 10^{-8}$ | 900k | 6.263 |
| $1 \times 10^{-7}$ | 800k | 6.294 |

**Grand mean:** $\kappa = 6.278 \times 10^{-8}\ \text{m}^2/\text{s}$, cross-run spread 0.49%.

The three initializations bracket the answer from opposite directions ($10^{-8}$ rises from below, $10^{-7}$ drops from above), confirming a genuine basin rather than an optimization artifact. Analytically confirmed: PINN/analytical ratio = 0.99.

## The 59.7% Gap with Spohn

The gap with Spohn's $3.93 \times 10^{-8}$ is the thermal fin effect — the mole conducts heat vertically, making the soil appear more diffusive. This is a property of the spatial-average forward model, not an optimization failure. The analytical solution independently produces $\kappa = 6.35 \times 10^{-8}$.

## Caveats

- Physics loss still decaying after $\kappa$ stabilized (network weights not fully settled, but $\kappa$ is fixed).
- The $10^{-7}$ run is the weakest — largest variability and most aggressive physics loss decay.

See `spatial_average_convergence_conclusions.pdf` for full figures and analysis.
