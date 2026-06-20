# Mars PINN Forward Problem: Optimisation Journey

## Problem Setup

Modelling Martian subsurface temperatures using InSight HP³ mission data (Spohn et al. 2024). The PINN solves the 1D heat diffusion equation:

$$\frac{\partial u}{\partial t} = \kappa \frac{\partial^2 u}{\partial x^2}$$

- **Domain:** x ∈ [0, 0.4] m depth, t ∈ [0, 88775.244] s (one Martian sol)
- **Data:** RAD radiometer (surface, x = 0), TEM-A mole sensors (x = 0.094 m)
- **Fixed parameter:** κ = 3.93 × 10⁻⁸ m²/s
- **Bottom BC:** Fixed at mean temperature 225.6 K (x = 0.4 m)

---

## Baseline Configuration

| Setting | Value |
|---------|-------|
| Architecture | 3 layers × 64 neurons, tanh activation |
| Input dimension | 2 (normalised t, x) |
| Collocation points | 10,000 |
| Loss weights | BC × 1, Physics × 1, Data × 50 |
| Optimiser | Adam, constant lr = 1e-3 |
| Iterations | 80,000 |

**Baseline result:** Loss = 0.002070, RMSE = 0.33 K

---

## Approach 1: Fourier Feature Embeddings

**Hypothesis:** The physics loss plateau was caused by spectral bias — standard MLPs with tanh activations struggle to represent high-frequency components, and this compounds in second-order derivatives needed for the heat equation residual.

**Implementation:** Added sin(2πft) and cos(2πft) for integer frequencies f = 1, 2, ..., N to the time input only (depth has exponential decay character, not oscillatory). Raw t and x still included.

| Frequencies | Input dim | Loss | RMSE |
|-------------|-----------|------|------|
| 5 (f = 1–5) | 12 | 0.003556 | 0.36 K |
| 2 (f = 1–2) | 6 | 0.003666 | 0.39 K |
| 1 (f = 1) | 4 | 0.003488 | 0.38 K |

**Conclusion:** Fourier features **did not help**. The subsurface signal at 9.4 cm depth is already heavily damped by the thermal skin depth (~3.3 cm for diurnal). The temperature wave is smooth enough that tanh handles it without spectral assistance. The added input complexity made optimisation harder without providing useful representational benefit.

**Key lesson:** Fourier features help when the *target signal* contains high-frequency content the network can't capture. At depth, the soil acts as a low-pass filter — the signal is already smooth.

---

## Approach 2: Learning Rate Schedules

**Hypothesis:** The loss was still declining at 80k iterations; the optimizer needed either more time or a gentler step size to converge further.

### Piecewise constant schedules

| Schedule | Total iterations | Loss | RMSE |
|----------|-----------------|------|------|
| Constant 1e-3 | 80,000 | 0.002070 | 0.33 K |
| 1e-3 → 1e-4 at 60k | 80,000 | 0.002409 | 0.34 K |
| 1e-3 (80k) → 1e-4 (40k) | 120,000 | 0.001523 | 0.29 K |
| 1e-3 (80k) → 1e-4 (80k) | 160,000 | 0.001279 | 0.28 K |
| Constant 1e-3 | 160,000 | oscillates 0.0012–0.0022 | 0.31 K |
| 1e-3 (120k) → 1e-4 (80k) | 200,000 | 0.001081 | 0.26 K |

**Conclusion:** Learning rate schedules **helped meaningfully**. The constant 1e-3 rate finds good minima but overshoots them — visible as oscillating loss values. Decaying to 1e-4 allows the optimizer to settle. The best strategy was a long exploration phase at 1e-3 followed by a long settling phase at 1e-4.

**Key observation:** At 160k with constant lr, the loss *bounced* between 0.0012 and 0.0022 — proving the optimizer was finding but not staying in good minima. The decay to 1e-4 captured these.

**Key lesson:** Very low learning rates (1e-5) were unproductive at this loss scale. The landscape was still too coarse for such small steps.

---

## Approach 3: Increased Network Capacity

**Hypothesis:** The 3×64 network (~8,500 parameters) had insufficient capacity to represent the solution.

| Architecture | Parameters | Loss | RMSE | Depth profiles |
|-------------|------------|------|------|----------------|
| 3 × 64 | ~8,500 | 0.001081 | 0.26 K | Unphysical (swings ±100 K) |
| 4 × 128 | ~50,000 | 0.000026 | 0.05 K | Extremely unphysical (swings ±400 K) |
| 4 × 128 + scheduler | ~50,000 | 0.000034 | 0.006 K | Still unphysical |

**Conclusion:** The 4×128 network achieved spectacular data fit (0.006 K RMSE at mole depth) but **completely failed physically**. Temperature vs depth profiles showed swings to -400 K at 30 cm depth — physically impossible. The network memorised the measurement points while producing nonsense in data-sparse regions.

**Key lesson:** Low RMSE at measurement locations does not mean the model is correct. Depth profile validation is essential. A bigger network requires stronger physics constraints to prevent overfitting.

---

## Approach 4: Loss Weight Rebalancing

**Hypothesis:** The data loss (weight 50) was overpowering the physics loss, causing the network to fit data at the expense of PDE compliance.

| Weights (BC, Physics, Data) | Architecture | Loss | RMSE | Depth profiles |
|----------------------------|-------------|------|------|----------------|
| 1, 1, 50 | 4 × 128 | 0.000026 | 0.05 K | Catastrophic |
| 1, 1, 10 | 4 × 128 | 0.000034 | 0.15 K | Still bad |
| 1, 10, 10 | 3 × 64 | — | 2.07 K | Slightly better, still wild |

**Conclusion:** Rebalancing alone **did not fix depth profiles**. Even with high physics weights, the depth behaviour remained unphysical. This pointed toward a more fundamental issue in how the physics was being enforced.

---

## Approach 5: Increased Collocation Points

**Hypothesis:** 10,000 physics points were insufficient to constrain a large network across the full domain.

| Collocation points | Architecture | Weights | RMSE | Depth profiles |
|-------------------|-------------|---------|------|----------------|
| 10,000 | 4 × 128 | 1, 10, 10 | 0.88 K | Catastrophic (worse) |
| 50,000 | 4 × 128 | 1, 10, 10 | — | Even worse |

**Conclusion:** More collocation points **made things worse**. The additional computational cost (50 min vs 10 min) produced no improvement. The problem was not sampling density.

---

## Approach 6: PDE Residual Rescaling (Breakthrough)

**Diagnosis:** The normalised diffusivity κ_eff ≈ 0.022 was causing a fundamental imbalance in the physics residual:

$$r = \frac{\partial u}{\partial \hat{t}} - 0.022 \cdot \frac{\partial^2 u}{\partial \hat{x}^2}$$

The second spatial derivative was being multiplied by 0.022 before entering the loss, meaning the network could have massive spatial curvature (d²u/dx² = 100) and the physics loss barely noticed (contribution of only 2.2). The physics loss was effectively learning "du/dt ≈ 0" while **ignoring spatial smoothness**.

**Fix:** Divide the PDE by κ_eff so both terms contribute equally:

$$\frac{1}{\kappa_{\text{eff}}} \frac{\partial u}{\partial \hat{t}} - \frac{\partial^2 u}{\partial \hat{x}^2} = 0$$

Code change: `return du_dt / kappa_eff - d2u_dx2` instead of `return du_dt - kappa_eff * d2u_dx2`

| Config | RMSE | Depth profiles |
|--------|------|----------------|
| Before rescaling (any config) | various | All unphysical, ±100–400 K swings |
| After rescaling, 3×64, data×200 | 1.47 K | Physical: 200–260 K range, converging to 225.6 K |
| After rescaling, 4×64, data×200 | 1.44 K | Similar, slightly better |

**Conclusion:** PDE rescaling was the **single most important fix** for physical correctness. All configurations before rescaling produced unphysical depth profiles. After rescaling, the temperature field behaved physically at all depths.

**Key lesson:** When normalising inputs in PINNs, check whether the effective PDE coefficients create imbalanced residuals. A small coefficient on d²u/dx² means the network can create large spatial oscillations at negligible physics loss cost. Rescaling the PDE to balance terms is essential.

---

## Approach 7: Additional Network Depth (Post-Rescaling)

| Architecture | RMSE | Depth profiles | Dominant loss |
|-------------|------|----------------|---------------|
| 3 × 64 | 1.47 K | Physical | BC: 0.025 |
| 4 × 64 | 1.44 K | Physical | BC: 0.025 |

**Conclusion:** Adding a fourth layer provided minimal improvement. The bottleneck is now the BC loss (surface temperature matching), not network capacity.

---

## Final State and Diagnosis

**Best physically-valid result:** RMSE ≈ 1.44 K with rescaled PDE, 4×64 network, data weight 200.

**Best data-fit result (unphysical):** RMSE ≈ 0.006 K with 4×128 network, original PDE, but catastrophically wrong depth profiles.

The remaining error in the physically-valid model shows a **clean sinusoidal pattern** — consistent phase lag and amplitude mismatch at the mole depth. This signature indicates the fixed thermal diffusivity κ = 3.93 × 10⁻⁸ m²/s is not exactly correct for this depth and timescale. No amount of network tuning can fix this because the network is now correctly solving the heat equation — just with the wrong κ.

**Untried approaches:**
- L-BFGS as a second-stage optimizer (standard in PINN literature, could help 10–15%)
- Residual-adaptive resampling of collocation points
- Sinusoidal activation functions (SIREN)

**Natural next step:** Inverse problem — let the PINN learn κ from the data rather than fixing it, which would directly address the diagnosed phase/amplitude mismatch.

---

## Key Takeaways

1. **Always validate physics, not just data fit.** The 0.006 K RMSE result was meaningless because the depth profiles were catastrophically wrong. Depth profile plots should be the primary diagnostic.

2. **PDE coefficient scaling is critical.** A small κ_eff made the physics loss blind to spatial oscillations. Rescaling the PDE to balance terms was the most impactful single change.

3. **Network capacity is a double-edged sword.** Bigger networks fit data better but require proportionally stronger physics constraints to prevent memorisation in data-sparse regions.

4. **Fourier features aren't always useful.** When the target signal is already smooth (as with subsurface temperatures filtered by the thermal skin depth), Fourier features add complexity without benefit.

5. **Change one variable at a time.** Multiple simultaneous changes made diagnosis impossible. The most progress came from isolated experiments.

6. **Structured residual errors have physical meaning.** A sinusoidal error pattern at the mole depth indicates a wrong diffusivity parameter, not a network limitation — pointing naturally toward the inverse problem.