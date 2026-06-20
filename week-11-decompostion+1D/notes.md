# Week 11: Decomposition and 1D Isothermal Models

## Overview

Week 10 showed that the 2D inverse problem fails because the network can co-adapt with $\kappa$ when there is no interior observation data — only boundary conditions. This week explores three strategies to break that degeneracy, progressively simplifying the geometry while sharpening the physics constraints.

The key physical insight driving all three approaches: **the mole is isothermal**. TEM-A reads the same temperature at the mole top ($z_0 = 1.7\ \text{cm}$) and bottom ($z_1 = 36.3\ \text{cm}$). In undisturbed soil, the temperature differs by ~12 K between these depths at solar noon. The mole erases this gradient — this is the vertical fingerprint of the thermal fin effect, and previous 1D models never enforced it.

---

## Reading Order

The three subdirectories represent three approaches tried roughly in chronological order. Each has its own design notes and conclusions:

### 1. `2D-Decomposition/` — First attempt

**Idea:** Keep the 2D cylindrical geometry but decompose $T = T_{1D} + T_{\text{pert}}$ where $T_{1D}$ is an analytical Fourier series (not a network) and $T_{\text{pert}}$ is a small perturbation network. This puts $\kappa$ into the analytical component where the network can't hide it, while the perturbation network only models the mole's localized effect.

**Read:**
- `2D-Decomposition-notes.md` — design rationale and identifiability argument
- `2D-Decomposition-Forward.ipynb` — forward validation ($\kappa$ fixed)
- `2D-Decomposition-Inverse.ipynb` — inverse attempts
- `2D-Decomposition-conclusions.md` — full results

**Result:** Forward works well (loss 0.023, all BCs matched, physically correct radial decay). Inverse still fails — $\kappa$ drifts downward in every configuration. The decomposition's theoretical identifiability argument is correct in principle, but the perturbation network has enough freedom to absorb the inconsistency at achievable loss levels. Literature confirms: inverse PINNs require interior observation data, not just boundary data.

### 2. `1D-Decomposition/` — Reduce to 1D with decomposition

**Idea:** Drop the radial dimension entirely and apply the same $T = T_{1D} + T_{\text{pert}}$ decomposition in 1D. The mole top and bottom become boundary conditions for $T_{\text{pert}}$ in two thin soil layers (above and below the mole). Since the mole BCs depend on $\kappa$ through $T_{1D}$, and the PDE also depends on $\kappa$, the two constraints should pin $\kappa$ without interior data.

**Read:**
- `1D-isothermal-decomposition.md` — design rationale
- `1D-Forward-decomposition.ipynb` — forward validation
- `1D-Inverse-decomposition.ipynb` — inverse with multi-initialization tests

**Result:** The forward shows that the top layer (1.7 cm) is too thin — temperature profiles are nearly linear, so $\partial^2 T / \partial z^2 \approx 0$ and there's little $\kappa$ signal. Unphysical wiggles appear at sunrise. The mole radius (~1.35 cm) is comparable to the layer thickness, making the 1D assumption questionable. Single warm-started inverse yields $\kappa \approx 4.23 \times 10^{-8}$ but multi-initialization tests show the network can accommodate a wide range of $\kappa$ values.

### 3. `1D-Separate-Layers/` — Direct 1D with separate networks

**Idea:** Skip the decomposition entirely. Use two separate networks (one per soil region), each predicting $T(z, t)$ directly, sharing a single learnable $\kappa$. Four boundary conditions: RAD at the surface, TEM-A at the mole top and bottom (same data — isothermal), and constant mean temperature at depth.

**Read:**
- `1D-isothermal-normal.md` — design for both approaches (top-only and both layers)
- `top-layer.ipynb` — forward validation of the top layer alone
- `2-layers-forward.ipynb` — forward validation with both layers
- `2-layers-inverse.ipynb` — inverse with multi-initialization tests
- `1D-isothermal-conclusions.md` — comprehensive results and technical lessons

**Result:** The strongest result of the week. Forward solver works well in both layers. The bottom layer (36.3–50 cm) provides the cleanest $\kappa$ signal because the natural diurnal wave is completely dead at that depth — any temperature variation there is forced entirely by the mole and dissipated purely through diffusion. Single warm-started inverse: $\kappa \approx 4.23 \times 10^{-8}\ \text{m}^2/\text{s}$ ($\pm 0.003$), stable over 50k fine-tuning iterations. Multi-initialization tests reveal a degeneracy problem — different starting values converge to different $\kappa$ — but the warm-started result moved *against* its initialization bias (started at 3.93, ended at 4.23), which gives it credibility.

---

## Summary of Results

| Approach | Forward | Inverse $\kappa$ | Status |
|---|---|---|---|
| 2D Decomposition | Works (loss 0.023) | Fails — $\kappa$ drifts to ~$10^{-9}$ | Boundary data insufficient |
| 1D Decomposition | Top layer too thin | $4.23 \times 10^{-8}$ (single run) | Geometry issues in top layer |
| 1D Separate Layers | Works in both layers | $4.23 \times 10^{-8}$ (single run, stable) | Best result, but multi-init shows degeneracy |

All three approaches that produce a $\kappa$ estimate converge to the same value ($4.23 \times 10^{-8}$), which is 7.6% above Spohn's published $3.93 \times 10^{-8}$ and 1% above the Week 9 point-sensor result. This contradicts the original hypothesis that the isothermal constraint would push $\kappa$ downward.

---

## Key Conclusions

1. **Decomposition works well for forward problems** — separating the bulk 1D solution from the mole perturbation produces clean, physical temperature fields in both 1D and 2D.

2. **The 2D inverse problem needs interior data.** Boundary conditions alone are insufficient regardless of architecture, loss weighting, or training strategy. This is confirmed by the PINN inverse problem literature.

3. **The 1D isothermal constraint provides a usable $\kappa$ signal**, particularly from the bottom layer where the mole forces a diurnal signal into otherwise thermally dead soil.

4. **Multi-initialization reveals a degeneracy** — the network can accommodate a range of $\kappa$ values by adjusting its internal curvature. The warm-started result is the most trustworthy because it moved against its initialization.

5. **The bottom layer is the cleanest identifier** — no interference from the surface signal, no 2D geometry issues, and physically clean residuals.

---

## Connection to Week 12

The single warm-started $\kappa \approx 4.23 \times 10^{-8}$ and the spatial-average $\kappa \approx 6.27 \times 10^{-8}$ from Week 10 bracket Spohn's value from above. Week 12 performs statistical verification — running the spatial-average inverse from multiple initializations and across multiple $\kappa$ targets to establish convergence basins and confidence intervals.
