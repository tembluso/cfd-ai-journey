# Complete Learning Summary: Physics-Informed Neural Networks with JAX

## Session Overview
Built a complete PINN implementation in JAX to solve the 2D heat equation, progressing from understanding theory to creating animated visualizations of the learning process.

---

## 1. JAX Fundamentals

### Core Concepts
- **jax.numpy (jnp)**: Drop-in NumPy replacement with GPU support
- **Immutability**: No in-place operations, arrays are immutable
- **Functional programming**: Parameters stored separately from network functions
- **Explicit random keys**: `random.PRNGKey()` instead of global RNG state

### Key Operations

**Automatic Differentiation:**
```python
grad(func)  # Compute derivative
grad(func, argnums=0)  # Derivative w.r.t. first argument
grad(func, argnums=1)  # Derivative w.r.t. second argument
```

**Vectorization:**
```python
vmap(func, in_axes=(None, 0, 0, None))
# None = same value for all iterations
# 0 = different value for each iteration (along axis 0)
```

**JIT Compilation:**
```python
@jit
def function(...):
    # First call: ~2s compilation
    # Subsequent calls: 100-200x faster!
```

### Why JAX for PINNs?
- **Automatic differentiation** essential for computing PDE residuals
- **JIT compilation** provides 100x+ speedup
- Pure NumPy cannot compute gradients needed for physics loss

---

## 2. PINN Theory

### The Core Innovation
**Traditional Neural Networks:**
- Loss = MSE between predictions and training data
- Need input-output pairs everywhere

**Physics-Informed Neural Networks:**
- Loss = Data Loss + Physics Loss
- Only need boundary/initial data
- Physics equation enforced via automatic differentiation

### Why PINNs Are Powerful
- Work when you **don't have** the full solution
- Only need:
  - Initial conditions (IC)
  - Boundary conditions (BC)
  - The governing equation (PDE)
- Can solve PDEs in scenarios where analytical solutions don't exist

### When to Use Each Approach
**Standard NN:** When you have lots of training data everywhere  
**PINN:** When you only have boundary measurements but know the physics

---

## 3. Heat Equation Problem Setup

### The Physics
**Equation:** $\frac{\partial u}{\partial t} = \alpha \frac{\partial^2 u}{\partial x^2}$

**Domain:** 
- Space: $x \in [0, 1]$
- Time: $t \in [0, 1]$

**Initial Condition (IC):** $u(0, x) = \sin(\pi x)$

**Boundary Conditions (BC):** 
- $u(t, 0) = 0$ (left edge always zero)
- $u(t, 1) = 0$ (right edge always zero)

**Analytical Solution:** $u(t, x) = e^{-\alpha \pi^2 t} \sin(\pi x)$

### Training Data Structure

**Q&A: What data do we need?**
- ✅ IC data: Points at t=0 with various x values
- ✅ BC data: Points at x=0 and x=1 with various t values
- ✅ Physics collocation points: Interior points where PDE is enforced

**Our Setup:**
- 10 IC points at t=0
- 20 BC points (10 at each boundary)
- 900 physics points (30×30 grid)

---

## 4. Neural Network Architecture

### Network Design
**Harmonic Oscillator:** `[1, 32, 32, 32, 1]` (1D input: time)  
**Heat Equation:** `[2, 32, 32, 32, 1]` (2D input: time + space)

**Q&A: Does the forward pass function need modification?**
- ❌ **No!** Just changing layer_sizes is enough
- The network function is generic - matrix multiplication handles different input dimensions automatically

### JAX Functional Style
```python
# Parameters as list of dictionaries
params = [
    {'w': array(2,32), 'b': array(32)},   # First layer
    {'w': array(32,32), 'b': array(32)},  # Hidden
    {'w': array(32,1), 'b': array(1)}     # Output
]

# Network function
def network(params, x):
    for layer in params[:-1]:
        x = jnp.tanh(jnp.dot(x, layer['w']) + layer['b'])
    final_layer = params[-1]
    return jnp.dot(x, final_layer['w']) + final_layer['b']
```

---

## 5. Computing Physics Residuals

### The Challenge: Partial Derivatives

**Q&A: Why function inside a function?**

**Problem:** Need to differentiate network output w.r.t. inputs (t, x), not parameters.

**Solution:** Create a closure that fixes params:
```python
def physics_residual_single(params, t_point, x_point, alpha):
    def u(t_val, x_val):
        tx_in = jnp.array([[t_val, x_val]])
        return network(params, tx_in)[0, 0]
    
    # First derivative w.r.t. time (argument 0)
    du_dt = grad(u, argnums=0)(t_point, x_point)
    
    # Second derivative w.r.t. space (argument 1)
    du_dx = grad(u, argnums=1)(t_point, x_point)
    d2u_dx2 = grad(grad(u, argnums=1), argnums=1)(t_point, x_point)
    
    # PDE residual: should be ≈ 0
    return du_dt - alpha * d2u_dx2
```

**Why the closure pattern:**
- `params` is "locked in" from outer scope
- `u(t, x)` is now only a function of t and x
- JAX can clearly differentiate w.r.t. t or x
- Without closure: "differentiate network w.r.t. what?" → ambiguous
- With closure: "differentiate u w.r.t. t or x" → clear!

### Vectorization with vmap

**Q&A: What does `in_axes=(None, 0, 0, None)` mean?**

```python
physics_residual_batch = vmap(physics_residual_single, 
                               in_axes=(None, 0, 0, None))
```

- `params` (None): **Same** for all 900 points
- `t_point` (0): **Different** for each point (from t_physics array)
- `x_point` (0): **Different** for each point (from x_physics array)
- `alpha` (None): **Same** constant for all points

**What vmap does:** Automatically loops over arrays, calling function 900 times efficiently.

---

## 6. Loss Function Components

### Three Loss Terms

```python
def loss_fn(params):
    # 1. Initial Condition Loss
    tx_ic = jnp.concatenate([t_ic, x_ic], axis=1)
    u_pred_ic = network(params, tx_ic)
    ic_loss = jnp.mean((u_pred_ic - u_ic)**2)
    
    # 2. Boundary Condition Loss
    tx_bc = jnp.concatenate([t_bc, x_bc], axis=1)
    u_pred_bc = network(params, tx_bc)
    bc_loss = jnp.mean((u_pred_bc - u_bc)**2)
    
    # 3. Physics Loss
    physics_res = physics_residual_batch(params, t_physics, x_physics, alpha)
    physics_loss = WEIGHT * jnp.mean(physics_res**2)
    
    return ic_loss + bc_loss + physics_loss
```

**Q&A: Why three separate losses?**
- **IC Loss:** Enforce known initial condition
- **BC Loss:** Enforce known boundary conditions
- **Physics Loss:** Enforce PDE at interior points where we DON'T have data
- **No "normal" MSE loss** - we don't know the solution at interior points!

**Q&A: Why multiply physics loss by a weight?**
- Balance different components
- Without weight: 900 physics points overwhelm 30 data points
- Too small weight: Network ignores physics, just interpolates boundaries
- Too large weight: Physics dominates, might not fit IC/BC
- **Sweet spot needed!**

### Critical Debugging Moment

**Problem Encountered:**
- Loss = 0.0003 (looks great!)
- But PINN prediction didn't match exact solution
- Temperature decay was wrong

**Root Cause:** Physics weight = 1e-4 was too small
- Network learned IC/BC perfectly
- Ignored physics in interior
- Low total loss misleading!

**Solution:** Increased weight to 1e-2
- Now physics matters
- Final error improved 100x
- Temperature decay correct

**Key Lesson:** Low loss ≠ correct solution! Must verify physics is actually enforced.

---

## 7. Array Shapes and Concatenation

### Stack vs Concatenate

**Q&A: Why stack instead of concatenate?**

**Stack:** Creates NEW dimension
```python
a = jnp.array([1, 2, 3])  # shape (3,)
b = jnp.array([4, 5, 6])  # shape (3,)
jnp.stack([a, b], axis=1)  # shape (3, 2) ← new dimension
```

**Concatenate:** Joins along EXISTING dimension
```python
a = jnp.array([[1], [2], [3]])  # shape (3, 1)
b = jnp.array([[4], [5], [6]])  # shape (3, 1)
jnp.concatenate([a, b], axis=1)  # shape (3, 2) ← existing dimension
```

### Meshgrid for 2D Grids

**Q&A: How to create physics collocation points?**

Need all combinations of 30 time values × 30 space values = 900 points

```python
t_vals = jnp.linspace(0, 1, 30)  # 30 time points
x_vals = jnp.linspace(0, 1, 30)  # 30 space points

# Create 2D grids
T_grid, X_grid = jnp.meshgrid(t_vals, x_vals)  # Both shape (30, 30)

# Flatten to column vectors
t_physics = T_grid.reshape(-1, 1)  # shape (900, 1)
x_physics = X_grid.reshape(-1, 1)  # shape (900, 1)
```

**What meshgrid does:** Creates every (t, x) combination like a chessboard grid.

### Shape Conventions
- Column vectors: `(n, 1)` ← use `.reshape(-1, 1)`
- Network input: `(n, 2)` for n points with 2 coordinates each
- Always check shapes with `.shape` to debug!

---

## 8. Python Decorators

### The @ Symbol

**Q&A: What does @ mean?**

A **decorator** wraps a function:

```python
# These are equivalent:
@jit
def update_step(...):
    ...

# Same as:
def update_step(...):
    ...
update_step = jit(update_step)
```

**What @jit does:**
1. First call: Traces function, compiles to optimized XLA code (~2 seconds)
2. Subsequent calls: Runs compiled version (100-200x faster!)

**Other common decorators:**
- `@property` - Makes method act like attribute
- `@staticmethod` - Method without self
- `@jit` - Compile for speed

---

## 9. Training Performance

### Why PINNs Are Slower Than Regular NNs

**Regular NN per iteration:**
- 1 forward pass on training data
- 1 backward pass
- Total: ~3 operations

**PINN per iteration:**
- Forward pass on IC data (10 points)
- Forward pass on BC data (20 points)
- For each of 900 physics points:
  - Forward pass
  - Compute ∂u/∂t (via automatic differentiation)
  - Compute ∂²u/∂x² (nested differentiation)
- Backward pass through everything
- Total: ~900+ operations (300× more work!)

### Performance Results

**Without JIT:** ~0.2s per iteration (67 minutes total)  
**With JIT:** ~0.01s per iteration after compilation (5 minutes total)

**Speedup: ~10-15x from JIT alone!**

---

## 10. Effect of Parameters

### Alpha (Thermal Diffusivity)

**α = 0.01:** 
- At t=1: 91% of initial temperature remains (9% decay)
- Subtle, hard to see visually

**α = 0.1:**
- At t=1: 37% of initial temperature remains (63% decay)
- Dramatic cooling, much more visible

**Formula:** $u(t, x) = e^{-\alpha \pi^2 t} \sin(\pi x)$

Larger α → faster heat diffusion → more decay

### Learning Rate & Training

**Setup:**
- Learning rate: 1e-3 (Adam optimizer)
- Iterations: 20,000
- Time: ~90 seconds total

**Observation:** Model learns structure quickly (~3,000 iterations)
- Early: Random predictions
- Mid: Rough shape emerges
- Late: Fine-tuning details

---

## 11. Visualization Strategies

### 2D Heatmap (Spatiotemporal Plot)
- x-axis: time, y-axis: space, color: temperature
- Shows entire solution at once
- Best for: Publications, overview

### Time Slices (u vs x at fixed times)
- Shows spatial profile at t=0.25, 0.5, 0.75, 1.0
- Best for: Detailed error analysis

### Space Slices (u vs t at fixed positions)
- Shows temporal evolution at x=0.25, 0.4, 0.6, 0.75
- Best for: Understanding decay dynamics

### 3D Surface Plot
- Full u(t, x) as 3D surface
- Best for: Presentations, demos, "wow factor"
- Less precise than 2D slices

**Key Insight:** 
- **3D visualizations:** Impress non-experts
- **2D slices:** Convey precise information to experts

---

## 12. Common Pitfalls & Debugging

### Issue 1: Analytical Solution Available

**Q&A: Why not use analytical solution for training?**

We HAVE the solution: $u(t, x) = e^{-\alpha \pi^2 t} \sin(\pi x)$

Could generate input-output pairs everywhere (like standard NN).

**But:** The point of PINNs is to work WITHOUT full solution!
- In real problems, you often don't have analytical solution
- Only have boundary measurements + governing equation
- PINNs let you solve anyway!

**Use analytical solution for:**
- ✅ Validation/comparison
- ✅ Plotting alongside prediction
- ✅ Computing error metrics
- ❌ NOT for training

### Issue 2: Low Loss But Wrong Solution

**Symptoms:**
- Training loss = 0.0003 (great!)
- But predictions don't match exact solution

**Diagnosis:** Physics loss weight too small
- Network fits IC/BC perfectly (low loss from those terms)
- Ignores physics in interior (weighted too low to matter)
- Total loss misleading!

**Fix:** Increase physics weight from 1e-4 to 1e-2+

### Issue 3: Shape Mismatches

**Common errors:**
- Expecting (900, 2) but need separate (900, 1) arrays
- Forgetting `.reshape(-1, 1)` for column vectors
- Concatenating wrong axis

**Solution:** Always print shapes during debugging!

---

## 13. Key Takeaways

### Conceptual
1. **PINNs encode physics as a loss term**, not just data fitting
2. **Automatic differentiation** is essential - can't do PINNs without it
3. **Partial derivatives** via `argnums` parameter in JAX
4. **Closure pattern** critical for isolating variables to differentiate
5. **Loss balancing** is a crucial hyperparameter (physics weight)

### Practical
1. **Always use @jit** in JAX for production code (100x+ speedup)
2. **Check intermediate results** - low loss doesn't guarantee correctness
3. **Visualize from multiple angles** - 2D slices + 3D surfaces
4. **Model learns structure quickly** - first few thousand iterations most important
5. **Shape debugging essential** - print shapes liberally

### JAX-Specific
1. **Functional programming** - parameters separate from network
2. **Immutable arrays** - no in-place operations
3. **grad with argnums** - specify which argument to differentiate
4. **vmap for vectorization** - specify which dimensions vary
5. **JIT compilation** - first call slow, subsequent calls fast

---

## 14. Complete Code Pattern

### Minimal PINN Template

```python
import jax
import jax.numpy as jnp
from jax import grad, jit, vmap
import optax

# 1. Define network
def network(params, x):
    for layer in params[:-1]:
        x = jnp.tanh(jnp.dot(x, layer['w']) + layer['b'])
    return jnp.dot(x, params[-1]['w']) + params[-1]['b']

# 2. Physics residual (single point)
def physics_residual_single(params, t, x, alpha):
    def u(t_val, x_val):
        return network(params, jnp.array([[t_val, x_val]]))[0, 0]
    
    du_dt = grad(u, argnums=0)(t, x)
    d2u_dx2 = grad(grad(u, argnums=1), argnums=1)(t, x)
    
    return du_dt - alpha * d2u_dx2

# 3. Vectorize
physics_residual_batch = vmap(physics_residual_single, 
                               in_axes=(None, 0, 0, None))

# 4. Loss function
@jit
def update_step(params, opt_state, ...):
    def loss_fn(p):
        ic_loss = ...  # Fit initial condition
        bc_loss = ...  # Fit boundary conditions
        physics_loss = ...  # Enforce PDE
        return ic_loss + bc_loss + physics_loss
    
    loss, grads = jax.value_and_grad(loss_fn)(params)
    updates, opt_state = optimizer.update(grads, opt_state, params)
    params = optax.apply_updates(params, updates)
    return params, opt_state, loss

# 5. Training loop
for i in range(num_iterations):
    params, opt_state, loss = update_step(params, opt_state, ...)
```

---

## 15. Final Results

**Heat Equation with α=0.1:**
- Training time: ~90 seconds (20,000 iterations)
- Final loss: 0.000001
- Relative L2 error: ~0.003
- Temperature decay: 100% → 37% (63% cooling)
- Model learns structure in first 3,000 iterations

**Success metrics:**
- ✅ IC perfectly matched (t=0)
- ✅ BC perfectly maintained (x=0, x=1)
- ✅ Physics equation satisfied in interior
- ✅ Predictions match analytical solution

---
