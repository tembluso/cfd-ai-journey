# 1D isothermal No split

# 1D PINN with Isothermal Mole Constraint — Two Approaches

## Core Idea (shared by both)

The mole is isothermal: TEM-A reads the same temperature at the mole top (z₀ = 1.7 cm) and mole bottom (z₁ = 36.3 cm). Previous 1D PINNs never enforced this — the point-sensor model pinned TEM-A at one depth and let the network invent any gradient elsewhere. The isothermal constraint is the vertical fingerprint of the fin effect.

**Architecture (both approaches):**

- Network inputs: (z, t) → output: T(z, t) directly (no decomposition, no analytical pieces)
- Learnable κ parameterised as exp(log_κ) to enforce positivity
- PDE: 1D heat equation ∂T/∂t = κ ∂²T/∂z² enforced only in soil regions (not inside the mole)
- κ is identifiable because only one value of κ makes the heat equation hold between the pinned boundary temperatures

**Physical prediction:** κ should converge **below** 4.19 × 10⁻⁸ m²/s (the point-sensor result), potentially toward Spohn's 3.93 × 10⁻⁸. Reasoning: the isothermal mole creates steeper gradients (larger ∂²T/∂z²) than undisturbed soil, so κ must decrease to keep ∂T/∂t = κ ∂²T/∂z² balanced.

---

## Approach 1: Top Layer Only

**Domain:** z ∈ [0, z₀] where z₀ = 0.017 m (1.7 cm of soil)

**Boundary conditions:**

| Location | Condition | Source |
| --- | --- | --- |
| z = 0 (surface) | T = T_RAD(t) | RAD surface data |
| z = z₀ (mole top) | T = T_TEM-A(t) | TEM-A sensor data |

**PDE collocation:** throughout [0, z₀]

**Loss:**
$$\mathcal{L} = \lambda_{\text{surf}} \mathcal{L}*{\text{RAD}} + \lambda*{\text{mole}} \mathcal{L}*{\text{TEM-A}} + \lambda*{\text{phys}} \mathcal{L}_{\text{PDE}}$$

**Why it's sound:**

- The layer is ~0.5 skin depths thick, so the diurnal wave amplitude decays by ~40% across it — there IS curvature (not a linear profile)
- Two strong data boundaries pin both ends; the PDE must hold in between with a single κ
- Simplest possible setup: two BCs, one PDE, one unknown

**Concerns:**

- Only 1.7 cm of soil — limited curvature may weaken κ identifiability, making the optimizer sensitive to noise or loss weighting
- 1D assumption is questionable here: mole radius (~1.35 cm) is comparable to layer thickness (1.7 cm), so aspect ratio is ~1:1. In reality, surface heat flows around the mole radially as well as straight down — a 2D effect this model ignores

---

## Approach 2: Both Layers (4 Boundary Conditions)

**Domain:** z ∈ [0, z₀] ∪ [z₁, z_max] — two disconnected soil regions, nothing modelled inside the mole

**Boundary conditions:**

| Location | Condition | Source |
| --- | --- | --- |
| z = 0 (surface) | T = T_RAD(t) | RAD surface data |
| z = z₀ = 0.017 m (mole top) | T = T_TEM-A(t) | TEM-A sensor data |
| z = z₁ = 0.363 m (mole bottom) | T = T_TEM-A(t) | Same TEM-A data (isothermal mole) |
| z = z_max (deep) | T = T_mean | Constant mean temperature |

**PDE collocation:** in [0, z₀] and [z₁, z_max], not inside the mole

**Loss:**
$$\mathcal{L} = \lambda_{\text{surf}} \mathcal{L}*{\text{RAD}} + \lambda*{\text{mole_top}} \mathcal{L}*{\text{TEM-A}(z_0)} + \lambda*{\text{mole_bot}} \mathcal{L}*{\text{TEM-A}(z_1)} + \lambda*{\text{deep}} \mathcal{L}*{\text{T*{mean}}} + \lambda_{\text{phys}} \mathcal{L}_{\text{PDE}}$$

**Why it's sound:**

- The bottom layer has a clean κ signal: the natural diurnal wave is completely dead at 36.3 cm (~10 skin depths), but the mole forces a diurnal perturbation at that depth. The rate at which the soil absorbs/dissipates this forced wave is governed purely by κ with no interference from the surface signal
- Two independent soil layers constrain the same κ simultaneously — stronger identifiability
- The isothermal constraint is directly enforced: same TEM-A data at z₀ and z₁

**Concerns:**

- Choosing z_max: too shallow biases κ, too deep wastes collocation points. The mole forces a diurnal perturbation at z₁ = 36.3 cm, which decays with skin depth δ ≈ 3.4 cm — so the perturbation is dead by ~50 cm. In principle z_max anywhere beyond ~50 cm should give the same κ
- **Resolution: run at z_max = 50 cm, 70 cm, 1 m and verify κ is stable (grid independence test)**
- 1D assumption concern in the top layer still applies (but the bottom layer is safer — the physics there is a downward-diffusing perturbation from the mole bottom)
- One network spans two disconnected regions — may need care with input normalisation

---
