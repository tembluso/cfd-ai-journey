# Poisson Equation & Incompressible Navier-Stokes: Theory and PINN Implementation

## Overview
Week 6 covers the capstone of the foundational PINN journey. Two new types of problem appear: a **steady-state** PDE (Poisson) with no time dependence, and the full **incompressible Navier-Stokes** equations with pressure coupling and a divergence-free constraint. A third notebook experiments with Fourier feature embeddings to address spectral bias.

**Recommended reading order:** Poisson first (simpler, no time), then Navier-Stokes (standard), then Fourier-features (same problem, different input representation).

---

## 1. Notebook 1: Poisson Equation

### The Physics

$$\nabla^2 p = \frac{\partial^2 p}{\partial x^2} + \frac{\partial^2 p}{\partial y^2} = f(x, y)$$

### What's New: No Time

This is the first PDE with **no time dependence**. The Poisson equation asks: given a source distribution $f(x,y)$ and boundary values, what is the equilibrium field $p(x,y)$?

Think of it as the steady-state limit of the heat equation — $\partial u / \partial t = 0$ — where generation and conduction perfectly balance.

**Consequences for the PINN:**
- Network input: $(x, y)$ only — 2 inputs, not 3
- **No initial condition points** — there is no $t = 0$
- Loss = BC loss + physics loss (two terms, not three)
- The collocation grid is 2D: $50 \times 50 = 2500$ points

**Setup:**
- Domain: $x, y \in [0, 1]$
- Source: $f(x,y) = -2\pi^2 \sin(\pi x)\sin(\pi y)$
- Exact solution: $p(x,y) = \sin(\pi x)\sin(\pi y)$
- BC: $p = 0$ on all four walls
- Architecture: `[2, 64, 64, 64, 1]`
- Optimizer: Adam with cosine decay schedule (new — lr goes from 1e-3 to 1e-5)
- Loss weights: BC weighted 10x over physics
- Iterations: 40,000

### Key Implementation Detail

The residual enforces $\nabla^2 p - f = 0$:
```python
residual = d2p_dx2 + d2p_dy2 + 2*pi^2*sin(pi*x)*sin(pi*y)
```

Note the source term $f$ appears explicitly in the residual — this is new. In all previous problems, the PDE was homogeneous (no external forcing).

---

## 2. Notebook 2: Incompressible Navier-Stokes

### The Physics

$$\frac{\partial u}{\partial t} + u\frac{\partial u}{\partial x} + v\frac{\partial u}{\partial y} = -\frac{\partial p}{\partial x} + \nu \nabla^2 u$$

$$\frac{\partial v}{\partial t} + u\frac{\partial v}{\partial x} + v\frac{\partial v}{\partial y} = -\frac{\partial p}{\partial y} + \nu \nabla^2 v$$

$$\frac{\partial u}{\partial x} + \frac{\partial v}{\partial y} = 0$$

### What's New: Three Outputs and an Incompressibility Constraint

Compared to 2D Burgers (Week 5):

| | 2D Burgers | Navier-Stokes |
|---|---|---|
| Outputs | $u, v$ (2) | $u, v, p$ (3) |
| PDEs enforced | 2 momentum | 2 momentum + 1 incompressibility |
| Pressure | Not present | Learned by the network |
| Architecture | `[3, 32, 32, 32, 2]` | `[3, 128, 128, 128, 3]` |

The pressure gradient terms $-\partial p / \partial x$ and $-\partial p / \partial y$ steer the fluid from high to low pressure. The incompressibility constraint $\nabla \cdot \mathbf{u} = 0$ prevents the fluid from compressing or expanding — it acts as a constraint that pressure must adjust to satisfy at every instant.

The network learns $p$ simultaneously with $u$ and $v$. There is no separate pressure equation — pressure emerges from the constraint.

### Test Problem: Taylor-Green Vortex

One of the few exact solutions to the Navier-Stokes equations. A periodic array of counter-rotating vortices decays smoothly due to viscosity:

$$u = -\cos(x)\sin(y)\,e^{-2\nu t}, \quad v = \sin(x)\cos(y)\,e^{-2\nu t}$$

$$p = -\frac{1}{4}(\cos 2x + \cos 2y)\,e^{-4\nu t}$$

Velocity decays as $e^{-2\nu t}$, pressure as $e^{-4\nu t}$ (twice as fast — a consequence of the quadratic relationship between pressure and velocity).

**Setup:**
- Domain: $x, y \in [0, 2\pi]$, $t \in [0, 2]$, $\nu = 0.1$
- Architecture: `[3, 128, 128, 128, 3]` — wider than previous weeks
- Loss weights: BC weighted 10x ($w_{bc} = 10$)
- Optimizer: Adam with cosine decay schedule
- Points: 1000 IC, 1000 BC, 8000 physics
- Iterations: 40,000

### Key Implementation: Efficient Jacobian Computation

Instead of computing each derivative separately with `grad`, this notebook uses `jax.jacfwd` to compute the full Jacobian in one call:

```python
J = jacfwd(u_v_p, argnums=(0, 1, 2))(t, x, y)
# J[0] = [du/dt, dv/dt, dp/dt]
# J[1] = [du/dx, dv/dx, dp/dx]
# J[2] = [du/dy, dv/dy, dp/dy]
```

This is more efficient than 9 separate `grad` calls — one forward-mode pass gives all first derivatives of all three outputs with respect to all three inputs.

### Validation

The notebook includes three levels of validation:
- **Vorticity** ($\omega = \partial v/\partial x - \partial u/\partial y$): shows where the fluid spins, more physically informative than raw velocity
- **Temporal probes**: u, v, p tracked over time at fixed spatial points to catch drift
- **Line probes**: cuts through the domain at fixed $y$ to reveal phase and amplitude errors

---

## 3. Notebook 3: Fourier Feature Embeddings

### The Problem: Spectral Bias

Standard MLPs with tanh activation learn low-frequency functions easily but struggle with higher frequencies. The TGV solution contains spatial frequencies $k = 1$ (in $u$, $v$) and $k = 2$ (in $p$). Without help, the network must discover this periodic structure from scratch.

### The Fix

Instead of passing raw $(t, x, y)$, pre-encode the spatial inputs into their natural trigonometric basis:

$$\phi(t, x, y) = [t, \sin x, \cos x, \sin y, \cos y, \sin 2x, \cos 2x, \sin 2y, \cos 2y]$$

This transforms 3 inputs into 9 features. Time is left raw because the decay $e^{-2\nu t}$ is a smooth monotone function that tanh can learn without help.

**Architecture:** `[9, 128, 128, 128, 3]` — only the input layer changes.

Everything else — the loss function, training loop, optimizer, collocation points — is identical. This isolation makes it a clean controlled experiment.

### Results

The Fourier-features PINN converges to lower final loss (0.000014 vs 0.000038) and lower L2 errors across all fields. The improvement is most visible in the pressure field, which oscillates at $k = 2$ — the frequency that standard MLPs struggle with most.

---

## 4. Key Takeaways

1. **Steady-state PDEs have no IC:** Poisson removes time entirely. The loss has only BC and physics terms. This is conceptually distinct from every problem before it.
2. **Pressure is a constraint, not an equation:** In incompressible NS, pressure has no evolution equation. It adjusts instantaneously to keep $\nabla \cdot \mathbf{u} = 0$. The PINN learns this implicitly.
3. **Jacobian computation is more efficient:** `jacfwd` gives all first derivatives in one pass, replacing multiple `grad` calls. Essential when you have 3 outputs and 3 inputs.
4. **Fourier features help when you know the frequency content:** Pre-encoding spatial inputs in their natural basis overcomes spectral bias. But this requires prior knowledge — you must know what frequencies matter.
5. **Loss weighting matters more for complex systems:** With 3-4 loss terms, weighting BC 10x helps the network anchor at the boundaries before shaping the interior.

---

## 5. Connection to the Mars Problem

Weeks 1-6 built the complete PINN toolkit. Starting in Week 7, these techniques are applied to real Mars thermal data from the InSight lander:
- The heat equation (Weeks 2, 5) becomes the governing PDE for Martian subsurface temperatures
- Loss weighting strategies (all weeks) become critical when balancing physics loss against real measurement data
- The concept of the forward problem (given parameters, predict the field) transitions to the **inverse problem** (given data, learn the parameters)
