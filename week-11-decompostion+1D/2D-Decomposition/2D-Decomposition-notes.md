# 2D Axisymmetric PINN — Decomposition

---

### Background

The full 2D axisymmetric PINN (inputs r, z, t; output T) works perfectly as a forward solver (κ fixed), but the inverse problem fails structurally. The network flattens radial gradients and pushes κ → 0. The root cause: the network has too much interior freedom and κ only appears in the PDE, so the optimizer can always find a co-adapted (network weights, κ) pair that satisfies BCs without κ being physically correct.

### The fix: analytical + perturbation decomposition

Split the temperature field into two components:

$$T(r, z, t) = T_{1D}(z, t; \kappa) + T_{\text{pert}}(r, z, t)$$

**Component 1 — T_1D (analytical, not a neural network):**

The undisturbed soil temperature with no mole present. Implemented as a differentiable JAX function using the Fourier decomposition of the RAD surface data (25 harmonics already extracted):

$$T_{1D}(z, t; \kappa) = \bar{T} + \sum_{n=1}^{25} A_n , e^{-z/\delta_n} \cos(n\omega t - z/\delta_n + \phi_n)$$

where δ_n = √(κP/(nπ)). The amplitudes A_n and phases φ_n are fixed from the RAD Fourier analysis. κ is a learnable parameter that appears explicitly — gradients flow through δ_n via autograd.

**Component 2 — T_pert (small neural network):**

A network with inputs (r, z, t) and output T_pert that models only the mole's thermal perturbation of the surrounding soil. This should be a small network — the perturbation is a localised, smooth correction, not a complex global field.

**Total prediction:** T(r, z, t) = T_1D(z, t; κ) + T_pert(r, z, t)

### Governing PDE for T_pert

Since T_1D already satisfies the 1D heat equation and has no r-dependence, substituting the decomposition into the full cylindrical heat equation leaves:

$$\frac{\partial T_{\text{pert}}}{\partial t} = \kappa \left(\frac{\partial^2 T_{\text{pert}}}{\partial r^2} + \frac{1}{r}\frac{\partial T_{\text{pert}}}{\partial r} + \frac{\partial^2 T_{\text{pert}}}{\partial z^2}\right)$$

This is the physics loss — enforced at collocation points in the soil volume. Note the same learnable κ appears here as in T_1D.

### Boundary conditions for T_pert

- **r = r_max (far field):** T_pert = 0. No mole influence far away.
- **z = 0 (surface):** T_pert = 0. Surface temperature is the same with or without the mole at this radial distance.
- **z = z_max (deep):** T_pert = 0. Deep temperature unaffected by the mole.
- **r = R_mole (mole surface):** T_pert = T_mole(t) − T_1D(z, t; κ). The mole is isothermal at T_mole(t) from TEM-A, but T_1D varies with z, so the perturbation must make up the difference.

Three sides are zero. The mole side is nonzero and **depends on κ** through T_1D.

### Domain

- r: from R_mole (≈ 1.35 cm) to r_max (≈ 20–30 cm)
- z: from 0 to z_max (same as previous setups)
- t: one sol (diurnal cycle)

### Why κ is now identifiable

κ appears in two places that constrain each other:

1. **In T_1D:** κ controls the vertical decay and phase shift of the undisturbed temperature. This determines the z-profile of the mole BC (T_mole − T_1D), which varies with z because T_1D decays differently at different depths.
2. **In the PDE:** κ controls how fast T_pert diffuses radially from the mole surface (where it's nonzero) to the far field (where it's zero).

A wrong κ creates a mole BC with the wrong z-structure, and then the PDE with that same wrong κ cannot diffuse that wrong profile consistently to zero at the other three boundaries. The optimizer cannot resolve this tension without finding the correct κ.

Additionally, the perturbation network **cannot flatten radial gradients** because T_pert is forced to be nonzero at r = R_mole and zero at r_max. Real radial structure must exist — the network has no freedom to eliminate it.

### Loss structure

$$\mathcal{L} = \lambda_{\text{phys}} \mathcal{L}*{\text{PDE}} + \lambda*{\text{BC}} (\mathcal{L}*{\text{mole}} + \mathcal{L}*{\text{far}} + \mathcal{L}*{\text{surface}} + \mathcal{L}*{\text{deep}})$$

- **L_PDE:** cylindrical heat equation residual for T_pert at interior collocation points
- **L_mole:** |T_pert(R_mole, z, t) − [T_mole(t) − T_1D(z, t; κ)]|²
- **L_far:** |T_pert(r_max, z, t)|²
- **L_surface:** |T_pert(r, 0, t)|²
- **L_deep:** |T_pert(r, z_max, t)|²

No separate data loss — all observations enter as BCs.

### Learnable parameter

κ parameterised as log(κ), same as previous notebooks. Initialise from one of the independently-derived values (4.19 × 10⁻⁸ from point-sensor, 6.3 × 10⁻⁸ from spatial average, or 1 × 10⁻⁷ as stress test). Run all three to demonstrate convergence basin independence.

### Implementation notes

- The PDE residual scaling trick from the forward problem (multiplying by κΔt) should still be applied to balance term magnitudes.
- T_1D must be implemented as a pure JAX function (no numpy) so that gradients with respect to κ flow through automatically.
- The Fourier coefficients (A_n, φ_n, ω, T_mean) are fixed constants extracted from RAD data.
- The perturbation network should be small — the perturbation is a smooth, localised field. Start with 3 layers, 32 neurons each, and only increase if needed.
