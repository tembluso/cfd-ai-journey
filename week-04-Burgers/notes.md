# Burgers' Equation: Theory and PINN Implementation

## Overview
Burgers' equation combines the nonlinear convection from Week 3 with the diffusion from Week 2. This is the first PDE where two competing physical mechanisms — steepening and smoothing — coexist, making it a foundational model in fluid dynamics and a direct stepping stone toward Navier-Stokes.

---

## 1. The Physics: Burgers' Equation

### Governing Equation

$$\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x} = \nu \frac{\partial^2 u}{\partial x^2}$$

| Term | Mechanism | Effect |
|------|-----------|--------|
| $u \frac{\partial u}{\partial x}$ | Nonlinear convection (Week 3) | Steepens the wave |
| $\nu \frac{\partial^2 u}{\partial x^2}$ | Diffusion (Week 2) | Smooths the wave |

The viscosity $\nu$ controls which force wins. Small $\nu$ means convection dominates and steep fronts form. Large $\nu$ means diffusion dominates and the wave spreads out gently.

### Why Burgers' Matters

Burgers' equation is the simplest nonlinear PDE that captures the essence of fluid dynamics — the competition between inertia (convection) and viscosity (diffusion). It is used as a testbed for numerical methods because:
- It has the same nonlinear convection structure as Navier-Stokes
- Unlike pure convection (Week 3), the diffusion term prevents true discontinuities, so the solution stays smooth and PINNs can represent it
- For small $\nu$, the solution develops very steep but continuous fronts — a challenging test for any solver

---

## 2. Two Notebooks

This week has two notebooks that should be read in order:

### Notebook 1: `burgers-equation/burgers-pinn.ipynb` — Single Traveling Wave

A sine pulse in the first third of the domain steepens and travels rightward, with diffusion preventing shock formation.

**Setup:**
- Domain: $x \in [0, 3]$, $t \in [0, 2]$
- $\nu = 0.01/\pi \approx 0.00318$
- IC: $u = \sin(\pi x)$ for $x \leq 1$, $u = 0$ for $x > 1$
- BC: Dirichlet, $u = 0$ at both walls
- Architecture: `[2, 64, 64, 64, 64, 1]`
- Collocation: 2500 physics points (50 x 50 grid)
- Iterations: 20,000

**Key difference from Week 3:** The boundary conditions switched back to Dirichlet (from periodic). Since diffusion dissipates the wave energy, the solution naturally decays toward zero at the boundaries — no artificial reflections.

### Notebook 2: `burgers-double-bump/colliding-waves.ipynb` — Wave Collision

Two Gaussian bumps of opposite sign — one positive (traveling right), one negative (traveling left) — collide head-on.

**Setup:**
- Domain: $x \in [0, 2]$, $t \in [0, 2]$
- $\nu = 0.01/\pi$
- IC: $u = 1.2\, e^{-100(x-0.7)^2} - 1.2\, e^{-100(x-1.3)^2}$
- BC: Dirichlet, $u = 0$
- Same architecture and training as Notebook 1

**Why this matters:** The collision creates steep gradients in both space and time simultaneously, testing the PINN's ability to handle complex dynamics. The PINN captures the pre-collision steepening well but struggles with the sharp compression zone during and after collision. This reveals a practical limitation: even though diffusion prevents a true shock, the gradients during collision are steep enough to challenge the network's smooth function space.

---

## 3. Key Differences from Previous Weeks

### Physics Residual

The residual now combines both terms:
```python
residual = du_dt + u * du_dx - v * d2u_dx2
```

This is the first residual that requires computing both first and second spatial derivatives in the same expression.

### Collocation Density

Increased from 900 (30 x 30) to 2500 (50 x 50) physics points. The steeper gradients in Burgers' equation require denser sampling to resolve the physics accurately.

---

## 4. Key Takeaways

1. **Diffusion rescues PINNs:** Pure convection (Week 3) formed shocks that PINNs couldn't represent. Adding even small diffusion keeps the solution smooth and PINN-solvable.
2. **Burgers' = convection + diffusion:** Understanding this equation as the sum of Week 2 and Week 3 physics is the key conceptual insight.
3. **Collisions are hard:** The double-bump experiment shows PINNs struggling with multiple interacting steep gradients, even when the solution is technically smooth.
4. **Collocation density matters:** Steeper solutions need more physics points. This foreshadows the scaling challenges of harder problems.

---

## 5. Connection to Next Week

Week 5 extends to **2D**, where diffusion and convection happen in two spatial dimensions simultaneously. The 2D Burgers equation introduces a second velocity component $v(t, x, y)$ and couples the two — each velocity field advects the other. The network must output two quantities instead of one, and the physics residual enforces two coupled PDEs.
