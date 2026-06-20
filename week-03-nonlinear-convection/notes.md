# Nonlinear Convection: Theory and PINN Implementation

## Overview
This week moves from the **diffusion-dominated** heat equation (Week 2) to **convection-dominated** physics. The nonlinear convection equation introduces wave steepening and shock formation — phenomena where PINNs hit fundamental limitations.

---

## 1. The Physics: Nonlinear Convection

### Governing Equation

$$\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x} = 0$$

**Compare to Week 2's heat equation:**

| | Heat Equation | Nonlinear Convection |
|---|---|---|
| Equation | $\frac{\partial u}{\partial t} = \alpha \frac{\partial^2 u}{\partial x^2}$ | $\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x} = 0$ |
| Mechanism | Diffusion (smooths out) | Convection (steepens) |
| Derivatives | Second-order in space | First-order in space |
| Behavior | Decays over time | Steepens over time |
| Discontinuities | Never forms them | Forms shocks |

### Why Does the Wave Steepen?

The term $u \frac{\partial u}{\partial x}$ means **the wave speed depends on the wave amplitude**:
- Where $u$ is large, the wave travels fast
- Where $u$ is small, the wave travels slowly
- The tall part "catches up" to the short part ahead of it
- This causes the wavefront to steepen until it becomes vertical — a **shock**

### Shock Formation Time

For an initial condition $u(0, x) = \sin(\pi x)$ on $[0, 1]$, the maximum initial slope is $\pi$ (at $x = 0$). The shock forms when the wavefront becomes vertical:

$$t_{\text{shock}} = \frac{1}{\max(u_0'(x))} = \frac{1}{\pi} \approx 0.318$$

After this time, the classical solution breaks down — the equation develops a discontinuity.

---

## 2. Why PINNs Struggle with Shocks

PINNs approximate solutions with neural networks, which are **smooth functions** (especially with tanh activation). A shock is a discontinuity — a sharp jump from one value to another. The network physically cannot represent it.

What happens in practice:
- Before the shock ($t < 1/\pi$): PINN captures the steepening well
- At and after the shock ($t \geq 1/\pi$): PINN smears the discontinuity into a smooth transition
- The approximation is qualitatively right (wave moves, amplitude stays bounded) but quantitatively wrong at the shock front

This is a **fundamental limitation**, not a training issue. No amount of iterations or architecture tuning fixes it — the function space of smooth neural networks cannot contain discontinuous solutions.

---

## 3. Key Differences from Week 2

### Physics Residual

Week 2 (heat equation):
```python
residual = du_dt - alpha * d2u_dx2  # Second derivative in space
```

Week 3 (nonlinear convection):
```python
residual = du_dt + u * du_dx  # First derivative, multiplied by u itself
```

The nonlinear term $u \cdot \frac{\partial u}{\partial x}$ is what makes this equation harder — the coefficient of the spatial derivative is the solution itself, not a constant.

### Boundary Conditions: Periodic Instead of Dirichlet

Week 2 used **Dirichlet** BCs (fixed values at boundaries: $u = 0$ at both edges).

This week uses **periodic** BCs: $u(t, 0) = u(t, L)$. The loss enforces that the network predictions at the left and right boundaries match:

```python
bc_loss = mean((u_pred_left - u_pred_right)^2)
```

Why periodic? The wave propagates to the right. With Dirichlet BCs ($u = 0$ at the right edge), the wave would be forced to zero as it reaches the boundary, creating artificial reflections. Periodic BCs let the wave pass through one side and re-enter the other, avoiding boundary contamination.

### Network Architecture

| | Week 2 | Week 3 |
|---|---|---|
| Layers | `[2, 32, 32, 32, 1]` | `[2, 64, 64, 64, 64, 1]` |
| Hidden layers | 3 | 4 |
| Neurons per layer | 32 | 64 |
| Total parameters | ~2,200 | ~13,000 |

The deeper, wider network is needed because nonlinear convection is harder to approximate — the solution develops steep gradients that require more expressive capacity.

### Domain

| | Week 2 | Week 3 |
|---|---|---|
| Space | $x \in [0, 1]$ | $x \in [0, 3]$ |
| Time | $t \in [0, 1]$ | $t \in [0, 2]$ |

Larger domain to let the wave propagate and steepen over a longer period.

---

## 4. Initial Condition

$$u(0, x) = \begin{cases} \sin(\pi x) & \text{if } x \leq 1 \\ 0 & \text{if } x > 1 \end{cases}$$

This creates a sine bump in the first third of the domain and zero elsewhere. The bump will travel to the right, steepening as it goes.

50 IC points are sampled (up from 10 in Week 2), because the piecewise nature of the IC needs denser sampling to capture the transition at $x = 1$.

---

## 5. Training Setup

| Setting | Value |
|---------|-------|
| IC points | 50 |
| BC points | 20 (10 left, 10 right) |
| Physics collocation | 900 (30 x 30 grid) |
| Physics loss weight | 1e-2 |
| Optimizer | Adam, lr = 1e-3 |
| Iterations | 20,000 |
| Final loss | 0.000147 |

---

## 6. Key Takeaways

1. **Convection vs diffusion:** Diffusion smooths, convection steepens. This is the fundamental contrast in fluid dynamics.
2. **PINNs can't represent shocks:** Neural networks are smooth functions. They approximate shocks as steep but continuous transitions. This is a hard limitation of the function space, not a training problem.
3. **Nonlinear terms are harder:** When the PDE coefficient depends on the solution itself ($u \cdot \partial u / \partial x$), the optimization landscape becomes more complex.
4. **Periodic BCs are useful:** When the physics involves wave propagation, periodic boundaries prevent artificial reflections.
5. **Deeper networks for harder problems:** Going from 3x32 to 4x64 was necessary to capture the steeper gradients.
6. **Shock formation time is predictable:** For simple ICs, $t_{\text{shock}} = 1 / \max(u_0')$ gives the exact breakdown time.

---

## 7. Connection to Next Week

Week 4 tackles **Burgers' equation**, which combines nonlinear convection (this week) with diffusion (Week 2):

$$\frac{\partial u}{\partial t} + u \frac{\partial u}{\partial x} = \nu \frac{\partial^2 u}{\partial x^2}$$

The diffusion term $\nu \frac{\partial^2 u}{\partial x^2}$ prevents true shocks from forming — instead of a discontinuity, the solution develops steep but smooth transitions. This makes Burgers' equation solvable by PINNs where pure convection is not, and is a better model of real viscous fluid behavior.
