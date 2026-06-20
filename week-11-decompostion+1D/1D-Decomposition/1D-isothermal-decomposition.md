
# 1D Vertical Perturbation PINN — Design Summary

### Background

Previous attempts to recover κ from the InSight HP³ data:

- **1D point-sensor:** κ ≈ 4.19 × 10⁻⁸ (7% off Spohn), poor mole fit (RMSE 2.63 K)
- **1D spatial-average:** κ ≈ 6.3 × 10⁻⁸ (60% off Spohn), excellent mole fit (RMSE 0.22 K), analytically confirmed — discrepancy is the thermal fin effect
- **2D axisymmetric (forward):** works perfectly with fixed κ
- **2D axisymmetric (inverse):** fails structurally — network flattens radial gradients and pushes κ → 0. Root cause: no interior observation data in the 2D domain, only boundary data. Literature confirms inverse PINNs need interior data.

### Core idea

Go back to 1D but capture the key physics the previous 1D models missed: **the mole is isothermal across its entire depth extent.** The mole forces T_mole(t) at both z₀ = 1.7 cm and z₁ = 36.3 cm simultaneously. In the undisturbed soil, T_1D differs by ~12 K between these depths at solar noon. The mole erases this gradient — cooling the soil near the top, warming it near the bottom. This is the vertical expression of the fin effect.

Previous 1D models never enforced this isothermal constraint. The spatial average compared a depth-averaged T_1D to TEM-A but never demanded the same temperature at z₀ and z₁.

### Decomposition

$$T(z, t) = T_{1D}(z, t; \kappa) + T_{\text{pert}}(z, t)$$

**T_1D** — analytical Fourier series (25 harmonics from RAD data), not a neural network:

$$T_{1D}(z, t; \kappa) = \bar{T} + \sum_{n=1}^{25} A_n , e^{-z/\delta_n} \cos(n\omega t - z/\delta_n + \phi_n)$$

where δ_n = √(κP/(nπ)). Amplitudes and phases are fixed from RAD Fourier analysis. κ is learnable and gradients flow through δ_n via JAX autograd.

**T_pert** — a small neural network with inputs (z, t) modelling the mole's perturbation. Only active in the soil regions above and below the mole.

### Domain

Two disconnected soil regions (no PDE inside the mole — the mole is a known isothermal boundary):

- **Above mole:** z ∈ [0, z₀] where z₀ = 0.017 m (~1.7 cm of soil)
- **Below mole:** z ∈ [z₁, z_max] where z₁ = 0.363 m

One network handles both regions.

### PDE (for T_pert only)

$$\frac{\partial T_{\text{pert}}}{\partial t} = \kappa \frac{\partial^2 T_{\text{pert}}}{\partial z^2}$$

Enforced at collocation points in both soil regions. The same learnable κ as in T_1D — this is what ties them together.

### Boundary conditions for T_pert

| Location | Condition | Why |
| --- | --- | --- |
| z = 0 (surface) | T_pert = 0 | T_1D already matches RAD surface data |
| z = z₀ (mole top) | T_pert = T_mole(t) − T_1D(z₀, t; κ) | Mole forces isothermal temperature, differs from undisturbed soil. **Depends on κ.** |
| z = z₁ (mole bottom) | T_pert = T_mole(t) − T_1D(z₁, t; κ) | Same isothermal temperature at the bottom. **Depends on κ.** |
| z = z_max (deep) | T_pert = 0 | Deep temperature unaffected by mole |

The mole BCs are computed live during training — they change every time the optimizer updates κ.

### Why κ is identifiable

κ appears in **two places that constrain each other:**

1. **In T_1D:** κ controls the skin depth, which determines how T_1D varies with depth. This sets the mole BC targets (T_mole − T_1D) at z₀ and z₁.
2. **In the PDE:** κ controls how fast T_pert can diffuse in the thin soil layers between the mole BCs and the zero-valued surface/deep BCs.

Wrong κ → wrong T_1D → wrong mole BC targets → PDE with that same wrong κ can't reconcile those targets with T_pert = 0 at the edges.

Crucially, the mole BCs at z₀ and z₁ function as **interior data points** — exactly what the 2D model lacked. And the network has no radial dimension to escape into. It can only reshape T_pert along z in two thin layers, giving it far less freedom to cheat.

### Why this captures physics the previous 1D models missed

The spatial average model treated TEM-A as a depth-weighted average of T_1D. It never enforced that the mole imposes the **same temperature** at z₀ and z₁. This model does — and the discrepancy between T_mole(t) and T_1D(z, t; κ) at these two depths is the fingerprint of the fin effect projected onto the vertical axis.

At solar noon with Spohn's κ: T_1D(z₀) might be ~232 K, T_1D(z₁) might be ~220 K. The mole reads ~224 K. So T_pert = −8 K at the top (mole cools the soil) and T_pert = +4 K at the bottom (mole warms it). These values and their magnitudes are directly controlled by κ.

### Loss

$$\mathcal{L} = \lambda_{\text{edges}}(\mathcal{L}_{\text{surface}} + \mathcal{L}_{\text{deep}}) + \lambda_{\text{mole}}(\mathcal{L}_{\text{mole\_top}} + \mathcal{L}_{\text{mole\_bottom}}) + \lambda_{\text{phys}}\mathcal{L}_{\text{PDE}}$$

No separate data loss — all observations enter through BCs. κ is constrained by the tension between the κ-dependent mole BCs and the κ-dependent PDE.

### Network

- Inputs: (z, t), normalised to [0, 1]
- Architecture: [2, 32, 32, 32, 1] — small, the perturbation is smooth
- Activation: tanh
- Output: T_pert in Kelvin

### Learnable parameter

κ parameterised as exp(log_κ) to enforce positivity. Use separate learning rates (slow for κ, fast for network weights).

