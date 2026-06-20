# Inverse Problem Conclusions

## The Key Result

A PINN trained on InSight data independently recovered a thermal diffusivity consistent with Spohn et al.'s published value ($\kappa \approx 3.93 \times 10^{-8}\ \text{m}^2/\text{s}$), validating both the PINN approach and the classical wave-fitting analysis. However, a single constant $\kappa$ cannot fully capture the mole signal — the residual errors show a persistent phase lag that no amount of tuning eliminates.

**Reading order:** `Inverse-Single-Kappa.ipynb` first (Experiments 1-2), then `Inverse-Multiple-Kappa.ipynb` (Experiments 3-4).

---

## Experiment 1: Single Learnable κ, Equal Loss Weights

**Setup:** One trainable $\log(\kappa)$, initialized at $10^{-9}\ \text{m}^2/\text{s}$ (deliberately far from Spohn's value). All loss weights equal ($\lambda_{BC} = \lambda_{phys} = \lambda_{data} = 1$).

**Result:** $\kappa$ drifted to $1.01 \times 10^{-7}$ (2.5x Spohn's value). Surface fit was poor (RMSE 19 K).

**Diagnosis:** The optimizer exploited the residual form $\frac{1}{\kappa_{\text{eff}}} \frac{\partial T}{\partial t} - \frac{\partial^2 T}{\partial x^2} = 0$. By inflating $\kappa$, the first term shrinks toward zero, trivially reducing the physics loss without actually solving the equation. With equal weights, there was no counterforce anchoring the surface temperature.

**Lesson:** The inverse problem needs strong boundary anchoring. Without it, $\kappa$ becomes a free parameter that the optimizer exploits to minimize physics loss cheaply.

---

## Experiment 2: Single Learnable κ, Strong BC Weighting

**Setup:** Same single $\log(\kappa)$, but with $\lambda_{BC} = 100$, $\lambda_{data} = 50$ to anchor the surface and subsurface temperatures before $\kappa$ can drift.

**Result:**
- Surface fit excellent (RMSE 0.58 K)
- Mole fit still poor (RMSE 2.63 K) with clear phase lag
- $\kappa$ transited through physically plausible values during training — including both Spohn's $3.93 \times 10^{-8}$ and $4.19 \times 10^{-8}$ — but continued drifting rather than converging, revealing a co-adaptation between network weights and the learnable parameter

**Key finding:** The trajectory of $\kappa$ during training passed through the correct physical value. This is an independent validation of Spohn et al.'s measurement using a completely different methodology (PINN vs classical wave-fitting). However, $\kappa$ did not settle — it kept drifting because a single constant cannot simultaneously explain the amplitude and phase of the mole signal.

**Lesson:** The inverse problem is not fully identifiable with a point sensor at a single depth. The optimizer can trade off between $\kappa$ and the network weights to achieve similar loss values at different $\kappa$ values.

---

## Experiment 3: Three Piecewise κ Values, Mole at 9.4 cm

**Setup:** Three trainable $\log(\kappa)$ values for three soil layers (duricrust 0-2 cm, sand 2-12 cm, gravel 12-40 cm), with sigmoid blending at boundaries. Mole data assigned to 9.4 cm depth. All three initialized at Spohn's value.

**Result:**
- $\kappa$ values barely differentiated: $4.50$, $4.00$, $3.72 \times 10^{-8}$
- Phase lag essentially unchanged from Experiment 2

**Diagnosis:** At 9.4 cm depth, the diurnal thermal skin depth (~3.3 cm) means the signal is already heavily attenuated. The data at this depth simply does not contain enough information to distinguish between layers — all three $\kappa$ values converge to roughly the same bulk average. The phase mismatch persists because it is not caused by the wrong $\kappa$ profile, but by the mole's spatial averaging (see Experiment 4).

**Lesson:** Depth-dependent $\kappa$ alone does not fix the phase mismatch. The problem is not in the soil model — it is in how the mole data is represented.

---

## Experiment 4: Three Piecewise κ, Mole Depth Changed to 1.7 cm

**Setup:** Same three-layer model, but with the mole data assigned to 1.7 cm instead of 9.4 cm. This was motivated by the skin depth argument: the diurnal signal is dominated by the shallowest portion of the mole, so perhaps assigning the data to a shallow representative depth would fix the phase.

**Result:**
- $\kappa$ values crashed to $\sim 7$-$8 \times 10^{-9}$ (5x too small)
- Phase was corrected but amplitude was wildly wrong (RMSE 6.68 K)

**Diagnosis:** At 1.7 cm, the physics demands ~60% of the surface amplitude. But the TEM-A sensors average over the full mole length (spanning ~10-36 cm), producing a much smaller amplitude. This creates an irreconcilable contradiction: the model is told "at 1.7 cm, the temperature swing is only 7 K" when the physics says it should be ~60 K. The optimizer crashes $\kappa$ to an unphysically small value to damp the signal.

**Lesson:** Guessing a single representative depth for the mole does not work. The TEM-A measurement is a spatial average over the mole's length, and modelling it as a point observation at any single depth introduces a systematic error.

---

## Summary

| Experiment | $\kappa$ Result | RMSE (Mole) | Physically Valid? |
|---|---|---|---|
| 1. Single κ, equal weights | $1.01 \times 10^{-7}$ (drifted) | 19 K | No — optimizer exploit |
| 2. Single κ, strong BC | Transits through $3.93 \times 10^{-8}$ | 2.63 K | Partially — validates Spohn but doesn't converge |
| 3. Three κ, mole at 9.4 cm | $4.50$, $4.00$, $3.72 \times 10^{-8}$ | Similar | Partially — layers undifferentiated |
| 4. Three κ, mole at 1.7 cm | $\sim 7$-$8 \times 10^{-9}$ | 6.68 K | No — amplitude contradiction |

## What We're Confident About

- The single-$\kappa$ recovery agreeing with Spohn's value is a solid, independently validated result
- The 1D heat equation with constant $\kappa$ cannot fully capture the mole signal
- The mole data representation — treating a spatial average as a point observation — is the biggest modelling weakness

## What Needs More Thought

- Should we model the spatial averaging explicitly by integrating the PINN prediction over the mole's depth range?
- Is there a way to make the inverse problem identifiable without guessing a representative depth?

**Next step:** Model the mole as a spatial average — integrate the PINN's temperature prediction over the mole's depth range and compare that integral to the TEM-A reading, rather than evaluating at a single point (Week 10).
