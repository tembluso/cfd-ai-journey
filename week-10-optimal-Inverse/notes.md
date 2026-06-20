# Week 10: Improving the Inverse Problem

## Overview

Week 9 showed that the point-sensor inverse problem has structural issues — treating the mole as a single-depth measurement introduces systematic errors that prevent $\kappa$ from converging. This week explores two independent strategies to fix this:

1. **Spatial averaging** — model the mole as what it physically is: a 40 cm aluminum cylinder that reports a depth-averaged temperature, not a point measurement
2. **2D axisymmetric perturbance** — model the mole's thermal fin effect in cylindrical geometry, where the mole becomes a hard boundary condition rather than a soft data point

These are separate approaches that live in separate directories. Read either one independently.

---

## Approach 1: Spatial Average (`Spatial-Average/`)

**Read:** `Inverse_Spatial_Average.ipynb`

The key insight from Week 9 was that the TEM-A sensors don't measure temperature at a single depth — they measure the temperature of the mole, which is a nearly isothermal aluminum cylinder spanning $z = 1.7\ \text{cm}$ to $z = 36.3\ \text{cm}$. Aluminum conducts heat ~5000x better than Martian soil, so the mole equilibrates almost instantly. Its temperature is effectively a weighted spatial average of the surrounding soil temperature over its full length.

Instead of comparing the PINN prediction at one depth to the TEM-A reading, this notebook integrates the PINN prediction over the mole's depth range:

$$T_{\text{mole}}(t) = \frac{1}{z_1 - z_0} \int_{z_0}^{z_1} T_{\text{PINN}}(t, z)\, dz$$

This integral is approximated with 100 depth points and compared to the TEM-A data. The physics is unchanged (1D heat equation), and $\kappa$ is still learned via $\log(\kappa)$, but the data loss is now physically consistent with what the instrument actually measures.

**Result:** $\kappa$ converges to $6.27 \times 10^{-8}\ \text{m}^2/\text{s}$. Higher than Spohn's $3.93 \times 10^{-8}$, but stable and reproducible. The notebook also includes analytical verification and weighted-average experiments.

---

## Approach 2: 2D Axisymmetric Perturbance (`2D-perturbance/`)

This approach asks a different question: what if the problem is not how we represent the mole data, but that a 1D model is fundamentally insufficient? The mole is a physical object embedded in the soil — a metallic cylinder that conducts heat far better than its surroundings, acting as a "thermal fin" that pumps heat downward along its length. This creates radial temperature gradients that a 1D depth-only model cannot capture.

### Reading Order

There are four notebooks. Read them in this order:

**1. `AnalyticalSolutionRAD.ipynb`** — Preprocessing step. Decomposes the RAD surface temperature into a 25-harmonic Fourier series and saves the coefficients to `fourier_coefficients.pkl`. This provides a smooth, differentiable, noise-free surface boundary condition for the 2D model, avoiding the need to carry raw Excel data into the physics loop. Also derives the analytical 1D solution $T(z, t)$ for undisturbed soil (no mole), which serves as the far-field reference.

**2. `1D-Farfield.ipynb`** — Trains a dedicated 1D PINN to generate the far-field boundary condition at $r = r_{\max}$ (the outer edge of the 2D domain, far from the mole). This 1D model deliberately excludes TEM-A data because the far-field represents undisturbed soil. Uses depth-biased collocation sampling (more points near the surface where gradients are steep) and a periodicity loss to enforce $T(z, 0) = T(z, t_{\text{sol}})$.

**3. `Forward-2D-Perturbance.ipynb`** — The 2D forward problem. Solves the cylindrical heat equation:

$$\frac{\partial T}{\partial t} = \kappa \left(\frac{\partial^2 T}{\partial z^2} + \frac{\partial^2 T}{\partial r^2} + \frac{1}{r}\frac{\partial T}{\partial r}\right)$$

with three inputs $(t, z, r)$ and one output $T$. The $\frac{1}{r}$ geometric term is the key difference from a Cartesian 2D problem — it accounts for the cylindrical geometry where heat spreads over an increasing circumference as it moves outward from the mole.

Four boundary conditions:
- **Surface** ($z = 0$): RAD Fourier series from AnalyticalSolutionRAD
- **Bottom** ($z = 0.4\ \text{m}$): constant $T = 225.6\ \text{K}$
- **Mole surface** ($r = r_{\text{mole}}$): TEM-A data as a hard Dirichlet BC (promoted from soft data loss)
- **Far-field** ($r = r_{\max}$): Neumann $\partial T / \partial r = 0$ (the mole perturbation is negligible at 18x the mole radius)

This notebook validates the forward model with $\kappa$ fixed at Spohn's value. The key diagnostic is whether $T(r)$ decays monotonically from the mole outward — confirming the thermal fin effect is captured correctly.

**4. `Inverse-2D-Perturbance.ipynb`** — Attempts to learn $\kappa$ from the 2D model. This is where the approach fails — see conclusions below.

### 2D Perturbance Conclusions

The forward model works well. Depth profiles are physical, the thermal fin effect produces correct radial temperature decay, and the total loss reaches $3 \times 10^{-5}$.

The inverse problem fails for structural reasons. Three strategies were tried:

| Strategy | $\kappa$ Result | Diagnosis |
|---|---|---|
| Joint optimization | $1.69 \times 10^{-9}$ (0.04x true) | Network reshaped $T$ to make $\partial T/\partial t \approx 0$, making $\kappa \to 0$ the cheapest path |
| BC-only pretrain, then freeze network and optimize $\kappa$ | $\sim 1.6 \times 10^{-9}$ | Without physics during pretraining, the interior $T$ had wrong dynamics; time derivatives were too small |
| Warm-start from forward solution | N/A | Methodologically circular — forward weights were trained with the true $\kappa$ |

The core issue is **co-adaptation**: the network has enough freedom to reshape the interior temperature field to accommodate any $\kappa$, and the gradient-based optimizer exploits this every time. This is a structural identifiability problem, not a hyperparameter tuning issue.

The mole boundary condition is isothermal along $z$ — meaning the radial gradients near the mole are almost entirely determined by the difference between $T_{\text{mole}}(t)$ and the undisturbed 1D far-field $T(t, z)$. Whether that temperature difference contains enough information to uniquely determine $\kappa$ remains an open question.

---

## Key Takeaways

1. **Spatial averaging is physically correct** and produces a stable, convergent $\kappa$ — something the point-sensor model could not achieve.
2. **The 2D forward model works**, capturing the mole's thermal fin effect and producing physical temperature fields.
3. **The 2D inverse problem has a structural identifiability issue** — the network can co-adapt with $\kappa$ in ways that satisfy the loss without satisfying the physics.
4. **Promoting TEM-A from soft data to hard BC** changes the problem character: it constrains the mole surface but removes the data loss signal that was guiding $\kappa$ in the 1D case.

---

## Connection to Week 11

The 2D perturbance results motivate a decomposition approach: split the temperature field as $T = T_{1D} + T_{\text{pert}}$, where $T_{1D}$ is the undisturbed 1D solution (no mole) and $T_{\text{pert}}$ is the mole's perturbation. This separates the well-understood bulk behavior from the difficult-to-model mole effect, and may improve the inverse problem's identifiability.
