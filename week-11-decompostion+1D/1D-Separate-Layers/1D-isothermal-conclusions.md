# 1D Isothermal Mole Constraint Conclusions


## Context

The HP³ mole on NASA's InSight lander got stuck at 36.3 cm depth on Mars. Its two TEM-A sensors read the same temperature, confirming the mole is isothermal — acting as a thermal fin conducting the diurnal surface wave deep into the soil. The goal was to use a physics-informed neural network (PINN) to identify the Martian soil thermal diffusivity κ by exploiting this isothermal constraint.

Two reference values exist for comparison:

- **Spohn et al.:** κ = 3.93 × 10⁻⁸ m²/s (from analytical wave-fitting)
- **Previous point-sensor PINN:** κ = 4.19 × 10⁻⁸ m²/s (from earlier 1D PINN treating the mole as a single point)

The hypothesis was that properly enforcing the isothermal mole constraint would yield κ **below** 4.19 × 10⁻⁸, potentially closer to Spohn's value, because the mole creates steeper gradients than undisturbed soil.

---

## Approach 1: Top Layer Only (z ∈ [0, 1.7 cm])

**Setup:** Single network, surface BC from RAD (z = 0), mole BC from TEM-A (z = 1.7 cm), PDE collocation throughout.

**Findings:**

- Both BCs were matched well.
- Temperature profiles were nearly linear at most times — expected, since 1.7 cm is only ~0.5 skin depths.
- An unphysical wiggle appeared at t = 8:00 LTST (sunrise transition), where the profile bulged outward near 0.75–1.0 cm. This violates monotonicity for a pure 1D diffusion problem between smoothly varying boundaries.
- The PDE residual showed structured patterns (not random noise), concentrated during the sunrise transition near the mole boundary. This indicates systematic tension between the BCs and the PDE at κ = 3.93 × 10⁻⁸.
- The fundamental issue: only 1.7 cm of soil with nearly linear profiles means ∂²T/∂z² ≈ 0 across most of the domain. The second spatial derivative — which is what κ identification depends on — carries very little signal.

**Conclusion:** Approach 1 is useful as a sanity check but unsuitable for reliable κ identification. The 1D assumption is also questionable here: the mole radius (~1.35 cm) is comparable to the layer thickness (1.7 cm), making the geometry fundamentally 2D.

---

## Approach 2: Both Layers (z ∈ [0, 1.7 cm] ∪ [36.3 cm, 50 cm])

**Setup:** Two separate networks (one per soil layer), sharing a single learnable κ. Four boundary conditions:

1. z = 0 → T_RAD(t) (surface)
2. z = 1.7 cm → T_TEM-A(t) (mole top)
3. z = 36.3 cm → T_TEM-A(t) (mole bottom, same data)
4. z = 50 cm → T_mean = 225.6 K (deep boundary)

Each network had its own z-normalisation (z → [0, 1] within its region) and correspondingly different effective κ in normalised space.

### Forward Problem

**Implementation details:**

- Top network: [2, 64, 64, 64, 1], 2500 collocation points
- Bottom network: [2, 64, 64, 64, 1], 5000 collocation points (more due to larger domain)
- Loss weights: w_edges = 20, w_data = 50, w_physics = 1

**Initial issues:**

- Phase lag and amplitude mismatch at mole bottom BC — resolved by increasing network capacity and collocation density in the bottom region, and increasing the mole BC weight.
- Deep boundary showed small drift (~±0.03 K around 225.6 K) — acceptable relative to the 7 K diurnal signal.

**Final forward results:**

- All four BCs matched well.
- Bottom layer residual showed smooth, physically structured patterns near z = 36.3 cm, decaying deeper. The magnitude (~±0.002) was about half of the top layer's.
- Top layer residual retained structured patterns similar to Approach 1, confirming the 2D geometry issue persists in the thin layer.

### Inverse Problem — Single Run

**Setup:** κ parameterised as exp(log_κ) inside the params dict. Separate learning rates using `optax.multi_transform`: networks at 1e-3 (with schedule decay), κ at 1e-4.

**Result:** Starting from κ_init = 3.93 × 10⁻⁸, the inverse converged to *κ ≈ 4.23 × 10⁻⁸ m²/s**.

This is ~7.6% above Spohn's value and ~1% above the point-sensor result. Convergence required a fresh optimizer restart after the initial schedule decayed the κ learning rate too aggressively. After restart with a steady LR of 1e-4, κ stabilised within ±0.003 × 10⁻⁸ over 50k iterations.

### Multi-Initialisation Robustness Test

Four initial values were tested: 3.93 × 10⁻⁸, 6.3 × 10⁻⁸, 1.0 × 10⁻⁷, and 1.0 × 10⁻⁹.

**Attempt 1 — Simultaneous training (networks + κ from scratch):**
Runs did not converge to the same value. Each κ got stuck near an inflated version of its starting point (ranging from 0.2 to 13.5 × 10⁻⁸). Cause: the networks are flexible enough to accommodate any κ by adjusting internal curvature.

**Attempt 2 — Two-phase training (data-only → physics + κ):**
Phase 1: 50k iterations data-only (κ frozen) to establish the temperature field.
Phase 2: 150k iterations with physics on, networks at 1e-5, κ at 1e-4.
Result: Improved spread (1.4 to 5.5 × 10⁻⁸) but still not converging to a single point. Even at 1e-5, the networks drifted enough during Phase 2 to accommodate different κ values.

**Attempt 3 — Three-phase training (data-only → physics + κ → networks frozen):**
Added Phase 3: networks completely frozen, only κ learns.
Result: Similar spread. By the time Phase 3 started, the four runs had already developed different temperature fields in Phase 2.

**Attempt 4 — Two-phase: data-only → κ-only (skip Phase 2):**
Phase 1: data-only (identical across all runs due to same random seed).
Phase 3: networks frozen, only κ optimises.
Result: κ diverged to infinity for all runs. Reason: the data-only network had no physics constraint, so ∂²T/∂z² ≈ 0 everywhere. The residual du_dt/κ − d²u/dz² reduces to du_dt/κ, which is minimised as κ → ∞.

---

## Key Conclusions

### 1. κ cannot be identified from boundary data alone

The curvature (∂²T/∂z²) that encodes κ only emerges when the PDE is enforced during training. A data-only network produces smooth interpolations with no physically meaningful second derivative. This is fundamental, not a tuning problem.

### 2. Simultaneous learning creates a degeneracy

When networks and κ learn together, the network can absorb any κ by adjusting its internal curvature. This produces multiple "solutions" at different κ values, all with similar total loss. The inverse problem has a unique physical solution, but the PINN optimisation landscape has many local minima.

### 3. The single warm-started result (κ ≈ 4.23 × 10⁻⁸) is the most trustworthy

Starting from a physics-consistent forward solution (κ = 3.93 × 10⁻⁸), the inverse moved *away* from the starting value to 4.23 × 10⁻⁸. This movement against the bias is meaningful — the data is pulling κ upward. However, without multi-init confirmation, the robustness of this specific value is uncertain.

### 4. The result contradicts the original hypothesis

The hypothesis predicted κ should decrease below 4.19 × 10⁻⁸ when the isothermal constraint is enforced. Instead it increased slightly. Possible explanations:

- The 1D assumption in the top layer introduces systematic error (2D effects around the mole geometry).
- The bottom layer may genuinely indicate a slightly higher κ at depth.
- The RAD Sol 511 surface data is a proxy for Sol 1202 and may introduce timing/amplitude errors.

### 5. The bottom layer provides the cleanest κ signal

Physically, the bottom layer (36.3–50 cm) is the strongest identifier: the natural diurnal wave is dead at that depth, but the mole forces a signal that can only dissipate through diffusion. The forward residual confirms this — it's smoother and lower magnitude than the top layer.

---

## Technical Lessons (JAX/PINN)

- **Argument order bugs are silent killers in JAX.** When all inputs are arrays, mismatched function arguments produce no error but completely wrong training. Always verify argument order when calling jitted functions with many parameters.
- **Separate z-normalisation per region is correct.** With two networks spanning different physical length scales, each needs its own z → [0, 1] mapping and correspondingly different effective κ in normalised space.
- **Collocation density should scale with domain size and signal complexity.** 2500 points for 1.7 cm was adequate; the same for 13.7 cm was not. The bottom layer needed 5000–10000 points.
- **Learning rate schedules that decay too aggressively can stall κ convergence.** A better approach: train to rough convergence, then reset the optimizer with a fresh moderate LR rather than relying on pre-set decay boundaries.
- **`optax.multi_transform` is essential for inverse problems.** Network weights and physical parameters have completely different gradient magnitudes and need separate learning rates.
- **JIT closure capture of optimizers:** when redefining an optimizer inside a loop, the jitted function must also be redefined or it will use the stale optimizer from the first iteration.

---

## Future Work

1. **Train separate forwards at κ = 3.93, 6.3, 10 × 10⁻⁸, then run inverse from each.** This is the proper robustness test — different physics-consistent starting fields, checking whether all converge to the same κ.
2. **Grid independence test on z_max.** Run at z_max = 50, 70, 100 cm and verify κ is stable. This rules out the deep boundary influencing the result.
3. **Bottom-layer-only inverse.** Isolate the bottom network's contribution to κ identification by running an inverse that only uses the bottom layer's physics loss. This avoids contamination from the 2D-geometry issues in the top layer.
4. **2D axisymmetric model.** The fundamental limitation of this entire study is the 1D assumption. The mole has finite radius (~1.35 cm), and heat flows around it radially. A 2D axisymmetric PINN (r, z, t) would resolve the geometry properly and likely produce a more reliable κ.
5. **Multi-sol training.** Using data from multiple sols simultaneously would provide more constraint on κ and reduce sensitivity to measurement noise on any single sol.
6. **Constrained optimisation or penalty methods** to prevent the network degeneracy (networks absorbing κ), such as weight regularisation, explicit curvature penalties, or alternating optimisation with hard freezing cycles.