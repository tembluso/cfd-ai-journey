# 2D Perturbance Conclusions 


**Built:** a 2D axisymmetric PINN solving the heat equation around the HP³ mole on Mars. Three inputs (t, z, r), one output (T), with boundary data from TEM-A (mole), RAD (surface), constant bottom, and a far-field condition.

**Problem 1 — Residual magnitude imbalance.** The time term coefficient was O(10⁵), spatial terms were O(10). The network learned du_dt ≈ 0 and ignored spatial diffusion entirely. T(r) was non-physical — increasing away from the mole. **Fix:** multiplied the PDE by κΔt so all terms became O(0.01–1). Physics loss dropped from 1.5e-2 to 4.7e-5.

**Problem 2 — Analytical far-field BC conflicted with bottom BC.** The Fourier series converged to T_mean = 228.7 K at depth, but the bottom BC enforced 225.6 K. The 3 K corner conflict at (r_max, z_max) dominated the loss. Restricting the far-field to the upper 60% of the domain didn't help enough — the function itself was still wrong at depth. **Fix:** replaced the analytical Dirichlet BC with a Neumann condition (∂T/∂r = 0 at r_max). Physically cleaner — the mole perturbation is negligible at 18× the mole radius. Far-field loss dropped from 7.7e-3 to effectively zero.

**Problem 3 — Forward solution validated.** With both fixes, the forward PINN produced physically correct results: T(r) decays monotonically from mole to far-field, heatmaps show diurnal penetration to ~0.05 m (consistent with skin depth), T(z) profiles separate near the surface and converge at depth, time series tracks TEM-A data. Total loss reached 3e-5.

**Problem 4 — Inverse problem (learning κ) failed.** Three approaches tried, all failed for structural reasons:

- **Joint optimization** — κ collapsed to 1.69e-9 (0.04× true value). The network reshaped T to make du_dt ≈ 0, making κ → 0 the cheapest path. This is a known co-adaptation failure.
- **BC-only pretrain → freeze network → optimize κ** — κ again went to ~1.6e-9. Without physics during pretraining, the interior T field had wrong dynamics. The time derivatives were too small, so the optimizer again pushed κ toward zero. Physics loss stayed at 0.6 — the BC-only field doesn't satisfy the PDE for any κ.
- **Warm-starting from forward solution** — methodologically circular since the forward weights were trained with the true κ.

**Core conclusion so far:** the forward PINN works well. The inverse problem has a structural identifiability issue — the network has too much freedom to reshape the interior T field to accommodate any κ. The gradient-based optimizer exploits this every time.


**Considerations:** the mole BC is isothermal along z — meaning T at the mole surface depends only on t, not z. This means radial gradients near the mole are almost entirely determined by the difference between T_mole(t) and the 1D far-field T(t,z). Whether that temperature difference contains enough information to uniquely determine κ is a physics question, not a PINN question. The sweep will answer it empirically.

