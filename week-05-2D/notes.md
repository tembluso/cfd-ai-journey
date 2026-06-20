# 2D Problems: Theory and PINN Implementation

## Overview
This week makes the leap from 1D to 2D. Instead of $u(t, x)$, the network now predicts $u(t, x, y)$ — adding a spatial dimension changes the network input from 2 to 3, the collocation from lines to volumes, and introduces the 2D Laplacian. Two notebooks build up in complexity: 2D diffusion (one output) then 2D Burgers (two coupled outputs).

**Recommended reading order:** 2D-Diffusion first (simpler, one output), then 2D-Burgers (coupled system, two outputs).

---

## 1. What Changes in 2D

### Network Architecture

| Aspect | 1D (Weeks 2-4) | 2D (This week) |
|--------|----------------|-----------------|
| Input | $(t, x)$ — 2 values | $(t, x, y)$ — 3 values |
| Output | $u$ — 1 value | $u$ (diffusion) or $(u, v)$ (Burgers) |
| Input layer | 2 neurons | 3 neurons |
| Collocation | 2D grid ($t \times x$) | 3D grid ($t \times x \times y$) |
| Points needed | ~900 (30 x 30) | ~8000 (20 x 20 x 20) |

The number of collocation points scales as $N^d$ where $d$ is the number of dimensions. Going from 2D to 3D input space means the collocation grid grows rapidly — this is the **curse of dimensionality** that makes higher-dimensional PINNs expensive.

### The 2D Laplacian

In 1D, the diffusion operator is $\frac{\partial^2 u}{\partial x^2}$.

In 2D, it becomes the **Laplacian**: $\nabla^2 u = \frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2}$

Physically, the Laplacian measures how much a point's value differs from the average of its neighbors in all directions. If you're hotter than your surroundings, $\nabla^2 u < 0$ and heat flows out.

### Boundary Conditions in 2D

In 1D, boundaries are two points (left and right edges). In 2D, boundaries are four walls (left, right, bottom, top) — each is a line, not a point. This requires many more BC points: 1000 total (250 per edge) instead of 20.

---

## 2. Notebook 1: 2D Diffusion

### The Physics

$$\frac{\partial u}{\partial t} = \alpha \left(\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2}\right)$$

This is the 2D generalization of Week 2's heat equation. A Gaussian bump placed at the center of a square domain spreads outward in all directions and is absorbed at the zero-temperature walls.

**Setup:**
- Domain: $x, y \in [0, 1]$, $t \in [0, 2]$, $\alpha = 0.01$
- IC: Gaussian bump centered at $(0.5, 0.5)$ with $\sigma = 0.1$
- BC: $u = 0$ on all four walls
- Architecture: `[3, 32, 32, 32, 1]` — 3 inputs, 1 output
- Points: 2000 IC, 1000 BC, 8000 physics (20 x 20 x 20)
- Iterations: 40,000

### Key Implementation Detail

The physics residual adds a third `argnums`:
```python
d2u_dx2 = grad(grad(u, argnums=1), argnums=1)(t, x, y)
d2u_dy2 = grad(grad(u, argnums=2), argnums=2)(t, x, y)
residual = du_dt - alpha * (d2u_dx2 + d2u_dy2)
```

### New Feature: Component Loss Tracking

This is the first notebook that tracks IC, BC, and physics losses separately using `has_aux=True` in JAX's `value_and_grad`. This is essential for debugging — if one loss dominates, the optimizer may neglect the others.

### Validation

The peak temperature decay is compared against the analytical infinite-domain solution $u_{\max}(t) = \sigma^2 / (\sigma^2 + 2\alpha t)$. The PINN matches well at early times but diverges slightly later due to boundary effects (the analytical solution assumes infinite domain).

---

## 3. Notebook 2: 2D Burgers

### The Physics

$$\frac{\partial u}{\partial t} + u\frac{\partial u}{\partial x} + v\frac{\partial u}{\partial y} = \nu \left(\frac{\partial^2 u}{\partial x^2} + \frac{\partial^2 u}{\partial y^2}\right)$$

$$\frac{\partial v}{\partial t} + u\frac{\partial v}{\partial x} + v\frac{\partial v}{\partial y} = \nu \left(\frac{\partial^2 v}{\partial x^2} + \frac{\partial^2 v}{\partial y^2}\right)$$

This is the first **coupled system** — two PDEs that must be satisfied simultaneously. The $x$-velocity $u$ is advected by $v$, and vice versa. The network outputs two values at every point.

**Setup:**
- Domain: $x, y \in [0, 1]$, $t \in [0, 2]$, $\nu = 0.01$
- Exact solution: Fletcher (1983) analytical solution
- Architecture: `[3, 32, 32, 32, 2]` — 3 inputs, **2 outputs**
- Points: 2000 IC, 1000 BC, 8000 physics
- Iterations: 40,000

### Key Implementation Details

**Two outputs, two residuals:**
```python
def u(t, x, y):
    return network(params, [[t, x, y]])[0, 0]  # First output

def v(t, x, y):
    return network(params, [[t, x, y]])[0, 1]  # Second output

residual_u = du_dt + u*du_dx + v*du_dy - nu*(d2u_dx2 + d2u_dy2)
residual_v = dv_dt + u*dv_dx + v*dv_dy - nu*(d2v_dx2 + d2v_dy2)
```

**Combined physics loss:**
```python
physics_loss = mean(residual_u**2 + residual_v**2)
```

### Fletcher Analytical Solution

The exact solution has a notable property: $u + v = 3/2$ everywhere, always. The two fields are mirror images around $3/4$, connected by a shared exponential that governs a diagonal wavefront propagating across the domain. The IC and BC are all sampled from this exact solution, making quantitative error analysis possible.

### Validation: Relative L2 Error

The notebook computes L2 error at multiple time slices to check whether the PINN degrades at later times. This is important because PINNs can accumulate error as they extrapolate forward in time.

---

## 4. Key Takeaways

1. **The dimensional leap is mainly about collocation:** The network architecture barely changes (input size 2 to 3), but the number of collocation points jumps from ~900 to ~8000 to adequately sample the interior.
2. **Coupled systems output multiple values:** The network's last layer changes from 1 to 2 outputs. Each output gets its own PDE residual, and the total physics loss sums them.
3. **Component loss tracking is essential:** With IC, BC, and physics competing, you need to see which loss dominates. This becomes standard practice from here on.
4. **Exact solutions enable real validation:** Both notebooks use analytical solutions (Gaussian decay, Fletcher) to compute pointwise errors — not just loss values.

---

## 5. Connection to Next Week

Week 6 introduces two major new concepts:
- **Poisson equation** — a steady-state problem with no time dependence at all. The network input drops back to $(x, y)$ only, and there are no IC points — only BC and physics. This is conceptually different from everything so far.
- **Incompressible Navier-Stokes** — the full momentum equations with a pressure field and an incompressibility constraint. The network outputs three quantities $(u, v, p)$ and must satisfy three PDEs simultaneously, including $\nabla \cdot \mathbf{u} = 0$.
