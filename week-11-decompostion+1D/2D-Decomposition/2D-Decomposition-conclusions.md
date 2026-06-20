# 2D Axisymmetric Decomposition — Full Conclusion


### What we built

A 2D PINN using the decomposition T(r,z,t) = T_1D(z,t;κ) + T_pert(r,z,t), where T_1D is an analytical Fourier series (25 harmonics) for undisturbed soil temperature, and T_pert is a neural network modelling only the mole's thermal perturbation in cylindrical coordinates. The PDE enforced is the cylindrical heat equation for T_pert with the 1/r term.

### Forward problem — success

The forward solver (κ fixed at 1e-7) converged well after iterating on architecture, weights, and training strategy:

**Architecture progression:**

- `[3, 32, 32, 32, 1]` — initial attempt, insufficient capacity. Final loss ~0.16, edge BC ~0.29
- `[3, 64, 64, 64, 1]` — final architecture. Loss dropped significantly

**Loss weight progression (64-wide network):**

- No weights: total loss 0.16, mole BC dominated at ~0.1
- 10x on mole BC only: mole BC improved to 0.007 but edges worsened to 0.29
- 10x on both edges and mole: all losses decreased together
- Final configuration: 10x edges + 10x mole + 1x physics

**Best forward result (100k iterations, with LR scheduler dropping at 40k):**

- Total loss: 0.023
- BC edges: 0.0005
- Physics: 0.011
- BC mole: 0.0007

**Diagnostics confirmed:**

- Mole BC matched perfectly — T_1D + T_pert sits exactly on TEM-A data at all depths (5cm, 15cm, 30cm)
- T_pert radial decay is physically correct — nonzero at r = R_mole (1.35cm), decaying smoothly to ~0 at r_max (25cm) at all three test times (6h, 12h, 18h)
- T_pert is zero at surface (z=0) and deep (z=z_max) boundaries
- Negative T_pert values near the mole at noon are physically correct (T_1D overpredicts vs T_mole at shallow depths during peak heating)

### Inverse problem — failure

Made κ learnable as `exp(log_kappa)` initialized at 1e-7 (true value ~3.93e-8). κ drifts downward in every configuration tried:

**Attempt 1 — Joint training, same LR (1e-3), original residual (κΔt multiplying spatial terms):**

- κ drifted to ~1.44e-8 by 60k iterations and kept falling
- Root cause: smaller κ shrinks `κΔt`, which directly reduces the physics residual. The optimizer exploits this — pushing κ→0 trivially satisfies the PDE

**Attempt 2 — Reformulated residual (dividing by κΔt instead):**

- Without warm-start, initial loss was 30+ (network trained under different residual scaling)
- κ drifted to 3.15e-9 by 60k iterations
- Reformulation changed the loss landscape but didn't solve the fundamental problem

**Attempt 3 — Warm-start + separate learning rates (network 1e-3, κ 1e-5):**

- Started from forward-trained weights at κ=1e-7
- Initial loss matched forward final (~0.02), confirming correct weight loading
- κ moved from 1e-7 toward 4.24e-8 by 100k iterations — promising, passed near Spohn's value (3.93e-8)
- But kept drifting: reached 3.04e-8 at 200k iterations, still decreasing
- The slow LR delayed the drift but didn't stop it

**Attempt 4 — Heavy physics weighting (50x on physics loss):**

- κ still drifted to 3.25e-9
- Confirmed that no amount of fixed weighting prevents the co-adaptation between network weights and κ

### Root cause analysis

Literature review confirmed the fundamental issue: **inverse PINNs require interior observation data** (a data loss term L_data with measured values at points inside the domain). Every successful inverse thermal diffusivity paper uses temperature measurements at interior spatial locations, not just boundary conditions.

Our 2D setup only has data on boundaries — the mole surface (r = R_mole), the far field (r = r_max), the surface (z = 0), and depth (z = z_max). There are no measured temperatures inside the soil volume between the mole and the far field. Without interior data to pin the temperature field, the optimizer can always co-adapt the network weights and κ: the network adjusts its spatial structure to accommodate whatever κ the optimizer tries, and the loss decreases regardless of whether κ is physically correct.

The decomposition architecture's theoretical identifiability argument (κ squeezed between T_1D and PDE) is correct in principle, but in practice the network has enough representational freedom to absorb the inconsistency at the loss values we achieve (~0.01-0.02).

### What we learned

1. The decomposition architecture (analytical + perturbation network) is a sound approach for forward problems
2. Cylindrical PDE with 1/r term works correctly in normalized coordinates
3. Loss weighting matters — the mole BC needs 10x to compete with zero-target BCs
4. Network capacity matters — 64 neurons per layer significantly outperformed 32
5. LR scheduling helps squeeze the last order of magnitude from the loss
6. For inverse problems, boundary data alone is insufficient — interior observations are essential
7. Residual formulation, loss weighting, separate learning rates, and warm-starting are all insufficient substitutes for actual interior data