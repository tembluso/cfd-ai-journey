# Week 13: Noise Robustness Test

## Overview

Week 12 verified that the spatial-average inverse PINN converges from three different $\kappa$ initializations. This week tests a different axis of robustness: **sensitivity to measurement noise**.

The same spatial-average model is run three times with identical $\kappa_{\text{init}} = 3.93 \times 10^{-8}$ and identical network initialization (`PRNGKey(123)`), but with different Gaussian noise added to the TEM-A data ($\sigma = 0.5\ \text{K}$, seeds 1, 2, 3).

## Results

| Noise seed | Iterations | Final $\kappa$ ($\times 10^{-8}$) | Diff from Spohn |
|---|---|---|---|
| 1 | 900k | 6.23 | +58.5% |
| 2 | 1M | 6.30 | +60.3% |
| 3 | 1M | 6.22 | +58.4% |

All three converge to the same region as the noise-free Week 12 result ($6.278 \times 10^{-8}$). The spread across noise realizations is ~1.3%, confirming the estimate is not sensitive to measurement noise at the $\pm 0.5\ \text{K}$ level.

## Notebook

`noisy_spatial_average.ipynb` — contains all three runs sequentially (seed 1, seed 2, seed 3), each with its own training loop and $\kappa$ convergence plot. Run on Google Colab with cloud GPU.
