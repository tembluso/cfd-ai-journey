# Notes that are on paper

## Learned ODEs

## Learned PDEs

## Learned the general Heat Equation

# Heat Equation & Fourier Series: Complete Summary

## 1. THE HEAT EQUATION FUNDAMENTALS

### The PDE (Partial Differential Equation)
$$\frac{\partial T}{\partial t} = \alpha \frac{\partial^2 T}{\partial x^2}$$

**Physical meaning:**
- Left side: How fast temperature changes in time
- Right side: Proportional to curvature (second derivative) in space
- α = thermal diffusivity (material property, always positive)

**Key insight:** Heat flows from hot to cold. Sharp temperature gradients (high curvature) cause rapid heat flow.

---

## 2. WHY SINE WAVES ARE SPECIAL

### Eigenfunctions of the Heat Operator

When T(x,0) = sin(x):
$$\frac{\partial T}{\partial t} = \alpha \cdot \frac{\partial^2 T}{\partial x^2} = \alpha \cdot (-\sin(x)) = -\alpha \sin(x)$$

**What this means:**
- The second derivative of sin(x) is -sin(x)
- So the rate of change is proportional to the function itself
- This gives exponential decay: T(x,t) = sin(x) · e^(-αt)

**Physical picture:**
- Peaks (hot spots) lose heat to surroundings → cool down
- Valleys (cold spots) gain heat → warm up
- The SHAPE stays the same, but AMPLITUDE decays

### Why It Decays

The negative sign in ∂²T/∂x² = -sin(x) is crucial:
- Where T is positive (hot) → ∂T/∂t is negative (cooling)
- Where T is negative (cold) → ∂T/∂t is positive (warming)
- Everything moves toward the average (zero)

---

## 3. EULER'S FORMULA & COMPLEX NUMBERS

### The Beautiful Connection

$$e^{i\theta} = \cos(\theta) + i\sin(\theta)$$

**Geometric interpretation:**
- e^(iθ) is a point on the unit circle
- θ is the angle (in radians)
- Real part = cos(θ), Imaginary part = sin(θ)

### Key Values

- e^(iπ/2) = i (quarter rotation)
- e^(iπ) = -1 (half rotation) ← **The most beautiful equation!**
- e^(2πi) = 1 (full rotation, back to start)
- e^(inπ) = (-1)^n (alternates for integer n)
- e^(2nπi) = 1 (always, for integer n)

### Converting Between Forms

**From exponentials to trig:**
$$\cos(\theta) = \frac{e^{i\theta} + e^{-i\theta}}{2}$$

$$\sin(\theta) = \frac{e^{i\theta} - e^{-i\theta}}{2i}$$

**Why complex exponentials are useful:**
- Easier to differentiate: d/dx[e^(inx)] = in·e^(inx)
- Easier to integrate: ∫e^(inx)dx = e^(inx)/(in)
- Easier to multiply: e^(inx)·e^(imx) = e^(i(n+m)x)

---

## 4. FOURIER SERIES: THE BIG IDEA

### The Revolutionary Insight

**ANY function can be written as a sum of sine and cosine waves!**

This is like decomposing a vector into components:
- 3D space: any vector = aî + bĵ + ck̂
- Function space: any function = sum of sines and cosines

### Two Equivalent Forms

#### Complex Form (What We Used in Challenges)
$$f(t) = \sum_{n=-\infty}^{\infty} c_n e^{in\omega t}$$

where ω = 2π/T (fundamental frequency)

**Find coefficients:**
$$c_n = \frac{1}{T}\int_0^T f(t) e^{-in\omega t} dt$$

**Properties:**
- Sum goes from -∞ to +∞
- Coefficients can be complex
- Very clean for periodic functions

#### Real Form (Traditional Physics)
$$f(x) = a_0 + \sum_{n=1}^{\infty} \left[a_n \cos\left(\frac{n\pi x}{L}\right) + b_n \sin\left(\frac{n\pi x}{L}\right)\right]$$

**Find coefficients:**
$$a_n = \frac{2}{L}\int_0^L f(x) \cos\left(\frac{n\pi x}{L}\right) dx$$

$$b_n = \frac{2}{L}\int_0^L f(x) \sin\left(\frac{n\pi x}{L}\right) dx$$

**Properties:**
- Sum goes from 1 to ∞
- All coefficients are real
- Natural for boundary value problems

### They're Equivalent!

Using Euler's formula, you can convert between them:
- ONE cosine term = TWO complex exponentials (n and -n)
- ONE sine term = TWO complex exponentials (n and -n)

---

## 5. CHALLENGE 1: STEP FUNCTION COEFFICIENTS

### Problem Setup

Step function on [0,1]:
$$\text{step}(t) = \begin{cases} 1 & \text{if } 0 \leq t < 0.5 \\ -1 & \text{if } 0.5 \leq t < 1 \end{cases}$$

### Finding c_n (Complex Fourier Coefficients)

$$c_n = \int_0^1 \text{step}(t) e^{-n \cdot 2\pi i t} dt$$

Split into two parts:
$$c_n = \int_0^{0.5} 1 \cdot e^{-n \cdot 2\pi i t} dt + \int_{0.5}^1 (-1) \cdot e^{-n \cdot 2\pi i t} dt$$

**Integration:**
$$\int e^{-n \cdot 2\pi i t} dt = \frac{e^{-n \cdot 2\pi i t}}{-n \cdot 2\pi i}$$

**Evaluate bounds and simplify:**

After substitution and using:
- e^(-nπi) = (-1)^n
- e^(-2nπi) = 1

We get:
$$c_n = \frac{1}{-n\pi i}((-1)^n - 1)$$

### The Result

**For even n:** (-1)^n = 1, so (-1)^n - 1 = 0 → **c_n = 0**

**For odd n:** (-1)^n = -1, so (-1)^n - 1 = -2 → **c_n = 2/(nπi)**

**Key takeaway:** The step function's symmetry automatically kills even harmonics!

---

## 6. CHALLENGE 2: CONVERTING TO REAL SINE SERIES

### Starting Point

$$\text{step}(t) = \sum_{n=-\infty}^{\infty} c_n e^{n \cdot 2\pi i t}$$

Only odd n survive (even c_n = 0), so:
$$\text{step}(t) = \sum_{n \text{ odd}} \frac{2}{n\pi i} e^{n \cdot 2\pi i t}$$

### The Pairing Trick

Group positive and negative n together. For n=1 and n=-1:

$$c_1 e^{2\pi i t} + c_{-1} e^{-2\pi i t} = \frac{2}{\pi i}e^{2\pi i t} + \frac{-2}{\pi i}e^{-2\pi i t}$$

Factor:
$$= \frac{2}{\pi i}(e^{2\pi i t} - e^{-2\pi i t})$$

**Use sine formula:** e^(ix) - e^(-ix) = 2i·sin(x)

$$= \frac{2}{\pi i} \cdot 2i\sin(2\pi t) = \frac{4}{\pi}\sin(2\pi t)$$

### General Pattern

For any odd n:
$$c_n e^{n \cdot 2\pi i t} + c_{-n} e^{-n \cdot 2\pi i t} = \frac{4}{n\pi}\sin(n \cdot 2\pi t)$$

### Final Result

$$\boxed{\text{step}(t) = \sum_{n=1,3,5,...} \frac{4}{n\pi} \sin(n \cdot 2\pi t)}$$

Or more explicitly:
$$\text{step}(t) = \frac{4}{\pi}\sin(2\pi t) + \frac{4}{3\pi}\sin(6\pi t) + \frac{4}{5\pi}\sin(10\pi t) + ...$$

**Physical meaning:**
- Low frequencies (n=1,3) have large amplitudes
- High frequencies (n=9,11,...) have small amplitudes (1/n decay)
- Need infinitely many terms to make sharp discontinuity!

---

## 7. CHALLENGE 3: CONVERTING TO COSINE SERIES

### The Key Idea: Coordinate Shift

Define new variable: τ = t - 0.25

Then:
$$\sin(2\pi t) = \sin(2\pi(\tau + 0.25)) = \sin(2\pi\tau + \pi/2)$$

**Use trig identity:** sin(x + π/2) = cos(x)

So:
$$\sin(2\pi t) = \cos(2\pi\tau)$$

### Pattern for Odd n

For general odd n:
$$\sin(n \cdot 2\pi t) = \sin(n \cdot 2\pi\tau + n\pi/2)$$

This alternates:
- n=1: sin(2πτ + π/2) = cos(2πτ)
- n=3: sin(6πτ + 3π/2) = -cos(6πτ)
- n=5: sin(10πτ + 5π/2) = cos(10πτ)
- n=7: sin(14πτ + 7π/2) = -cos(14πτ)

Pattern: **+, -, +, -, ...**

### The Sign Formula

The alternating sign is: **(-1)^((n-1)/2)** for odd n

Or reindex with m = 1,2,3,... and n = 2m-1: **(-1)^(m+1)**

### Final Cosine Series

$$\boxed{\text{step}(\tau + 0.25) = \sum_{n=1,3,5,...} \frac{4}{n\pi} (-1)^{(n-1)/2} \cos(n \cdot 2\pi\tau)}$$

**Takeaway:** Sines vs cosines is just a coordinate shift (related by π/2 phase)!

---

## 8. SOLVING HEAT EQUATION WITH ARBITRARY INITIAL CONDITIONS

### The Master Plan

**Given:** Heat equation + initial condition T(x,0) = f(x)

**Solution Strategy:**

#### Step 1: Know the Eigenfunction Solutions

Each sine mode is a solution:
$$T_n(x,t) = \sin\left(\frac{n\pi x}{L}\right) \cdot e^{-\alpha(n\pi/L)^2 t}$$

**Key properties:**
- Spatial shape: sin(nπx/L) (never changes)
- Time evolution: e^(-α(nπ/L)²t) (exponential decay)
- Decay rate: proportional to n² (higher frequencies die faster!)

#### Step 2: Write General Solution (Superposition)

By linearity:
$$T(x,t) = \sum_{n=1}^{\infty} c_n \sin\left(\frac{n\pi x}{L}\right) \cdot e^{-\alpha(n\pi/L)^2 t}$$

This AUTOMATICALLY satisfies:
- The heat equation ✓
- Boundary conditions T(0,t)=0, T(L,t)=0 ✓
- Only need to match initial condition!

#### Step 3: Find Coefficients from Initial Condition

At t=0:
$$T(x,0) = \sum_{n=1}^{\infty} c_n \sin\left(\frac{n\pi x}{L}\right) = f(x)$$

**This is a Fourier sine series!**

Find c_n using orthogonality:
$$c_n = \frac{2}{L}\int_0^L f(x) \sin\left(\frac{n\pi x}{L}\right) dx$$

#### Step 4: Done!

Substitute c_n back into general solution. You've solved the PDE!

---

## 9. EXAMPLE: HEAT EQUATION WITH STEP INITIAL CONDITION

### Setup

**Rod:** 0 ≤ x ≤ L
**Boundary conditions:** T(0,t) = 0, T(L,t) = 0
**Initial condition:**
$$T(x,0) = \begin{cases} 1 & \text{if } 0 < x < L/2 \\ -1 & \text{if } L/2 < x < L \end{cases}$$

### Finding Coefficients

$$c_n = \frac{2}{L}\int_0^L T(x,0) \sin\left(\frac{n\pi x}{L}\right) dx$$

$$= \frac{2}{L}\left[\int_0^{L/2} 1 \cdot \sin\left(\frac{n\pi x}{L}\right) dx + \int_{L/2}^L (-1) \cdot \sin\left(\frac{n\pi x}{L}\right) dx\right]$$

**After integration and simplification:**
$$c_n = \begin{cases} \frac{4}{n\pi} & \text{if } n \text{ odd} \\ 0 & \text{if } n \text{ even} \end{cases}$$

### Complete Solution

$$\boxed{T(x,t) = \sum_{n=1,3,5,...} \frac{4}{n\pi} \sin\left(\frac{n\pi x}{L}\right) e^{-\alpha(n\pi/L)^2 t}}$$

### Physical Evolution

**t = 0:** Sharp step discontinuity

**Small t:**
- High modes (n=9,11,...) decay rapidly (e^(-81α(...)))
- Corners round off
- Still oscillatory

**Medium t:**
- Only low modes (n=1,3,5) remain
- Smooth wave pattern

**Large t:**
- Only n=1 survives significantly
- Nearly pure sine wave

**t → ∞:**
- All modes decay to zero
- Uniform temperature (boundary value)

---

## 10. WHY HIGH FREQUENCIES DECAY FASTER

### The Mathematical Reason

Decay rate: e^(-α(nπ/L)²t)

The exponent has **n²**, so:
- n=1: decay ∝ e^(-αt)
- n=2: decay ∝ e^(-4αt) (4× faster)
- n=3: decay ∝ e^(-9αt) (9× faster)
- n=10: decay ∝ e^(-100αt) (100× faster!)

### The Physical Reason

**High frequency = short wavelength = sharp features**

- Sharp temperature spikes have high curvature (large ∂²T/∂x²)
- High curvature drives fast heat flow
- Neighbors are very close, so diffusion is rapid
- Sharp features smooth out quickly

**Low frequency = long wavelength = gentle features**

- Gentle temperature variations have low curvature
- Low curvature means slow heat flow
- Neighbors are far apart
- Gentle features persist longer

**Real example:**
- Ice cube on hot pan
  - Initial: sharp edge (all frequencies)
  - 1 second: rounded edge (high freq gone)
  - 10 seconds: gentle hill (only low freq)
  - 1 minute: nearly flat (approaching equilibrium)

---

## 11. TWO METHODS: WHEN TO USE WHICH?

### Complex Exponential Method

**Form:**
$$f(t) = \sum_{n=-\infty}^{\infty} c_n e^{in\omega t}$$

**Best for:**
- Periodic functions on [0,T]
- Time-domain problems
- When you want cleaner algebra
- Signal processing applications

**Coefficient formula:**
$$c_n = \frac{1}{T}\int_0^T f(t) e^{-in\omega t} dt$$

### Real Sine/Cosine Method

**Form:**
$$f(x) = \sum_{n=1}^{\infty} c_n \sin\left(\frac{n\pi x}{L}\right)$$

**Best for:**
- Boundary value problems (fixed endpoints)
- Spatial problems
- When boundary conditions naturally pick sine or cosine
- When you want real-valued coefficients

**Coefficient formula:**
$$c_n = \frac{2}{L}\int_0^L f(x) \sin\left(\frac{n\pi x}{L}\right) dx$$

### They're Mathematically Equivalent!

You can convert between them using:
$$\sin(x) = \frac{e^{ix} - e^{-ix}}{2i}, \quad \cos(x) = \frac{e^{ix} + e^{-ix}}{2}$$

---

## 12. KEY INSIGHTS & TAKEAWAYS

### Mathematical Insights

1. **Eigenfunction principle:** Sines and cosines are eigenfunctions of ∂²/∂x²
2. **Linearity is power:** Sum of solutions is a solution
3. **Fourier's genius:** ANY function decomposes into these eigenfunctions
4. **Orthogonality:** Different modes are independent (don't interfere)

### Physical Insights

1. **Heat flows from hot to cold** (second derivative determines flow rate)
2. **Sharp features smooth out fast** (high curvature = fast diffusion)
3. **Gentle features persist** (low curvature = slow diffusion)
4. **Each frequency evolves independently** (for linear PDEs)

### Problem-Solving Strategy

1. Identify eigenfunction solutions (sine/cosine basis)
2. Write general solution as superposition
3. Match initial conditions using Fourier series
4. Attach time evolution to each mode
5. Physical interpretation of solution

### Connections

- **A-level mechanics:** Normal modes of vibrating string (same math!)
- **A-level electronics:** Frequency analysis of signals
- **Further maths:** Eigenvectors and eigenvalues
- **Complex numbers:** Rotations on unit circle
- **Differential equations:** Separation of variables

---

## 13. WHAT'S NEXT: PATH TO PINNs

### Where Classical Fourier Methods Fail

**Problem 1: Complex Boundary Conditions**
- Not just T=0 at edges
- Irregular geometries (not simple rod/rectangle)
- Time-dependent boundaries

**Problem 2: Nonlinear PDEs**
- Navier-Stokes: u·∇u term (nonlinear!)
- Can't use superposition anymore
- Fourier series don't work directly

**Problem 3: Sparse/Noisy Data**
- Don't know full initial condition
- Only have measurements at scattered points
- Need to interpolate and solve simultaneously

**Problem 4: Inverse Problems**
- Unknown parameters (α, boundary conditions)
- Need to infer from observations
- Classical methods struggle

### Enter Physics-Informed Neural Networks (PINNs)

PINNs can handle all these cases by:
- Using neural networks as solution approximators (flexible!)
- Encoding physics (PDE) as loss function constraints
- Training on sparse data while respecting physical laws
- Solving forward and inverse problems in same framework

**Coming up:** How PINNs work and why they're revolutionary!

---

## QUICK REFERENCE FORMULAS

### Heat Equation
$$\frac{\partial T}{\partial t} = \alpha \frac{\partial^2 T}{\partial x^2}$$

### Euler's Formula
$$e^{i\theta} = \cos(\theta) + i\sin(\theta)$$

### Complex Fourier Series
$$f(t) = \sum_{n=-\infty}^{\infty} c_n e^{in\omega t}, \quad c_n = \frac{1}{T}\int_0^T f(t) e^{-in\omega t} dt$$

### Real Fourier Series (Sine)
$$f(x) = \sum_{n=1}^{\infty} c_n \sin\left(\frac{n\pi x}{L}\right), \quad c_n = \frac{2}{L}\int_0^L f(x) \sin\left(\frac{n\pi x}{L}\right) dx$$

### Heat Equation Solution
$$T(x,t) = \sum_{n=1}^{\infty} c_n \sin\left(\frac{n\pi x}{L}\right) e^{-\alpha(n\pi/L)^2 t}$$

### Step Function Fourier Series
$$\text{step}(t) = \sum_{n=1,3,5,...} \frac{4}{n\pi} \sin(n \cdot 2\pi t)$$

---

**END OF NOTES**

These notes cover everything from physical intuition to mathematical machinery. Review them before moving forward with PINNs!

# Here is the breakdown of where the equations fit in and how PINNs differ from normal NNs.

### 1. The Core Difference: The Loss Function
In a **normal Neural Network**, as you correctly noted, the training process focuses on minimizing the error between the network's output and the known data (expected output).
*   **Normal Loss:** $Loss = (Output - Data)^2$

In a **PINN**, the neural network architecture (layers, neurons, activation functions) is usually the same as a normal network. However, the loss function has **two** parts:
*   **PINN Loss:** $Loss = \underbrace{(Output - Data)^2}_{\text{Data Loss}} + \underbrace{(Differential \ Equation \ Residual)^2}_{\text{Physics Loss}}$

The "Physics Loss" is where the equation lives. The network is penalized not just for missing the data points, but also for producing a curve that violates the laws of physics.

### 2. How the "Equation" is put into the Network
You asked where the equations are "somewhere in there." They are not hard-coded into the architecture (like the connections between neurons); rather, they are calculated **during the forward pass** to check if the network's current guess obeys physics.

Here is the step-by-step mechanism using the **Harmonic Oscillator** (a spring system) as an example from your sources:

**The Equation:**
The physics states that mass times acceleration plus friction plus spring force equals zero:
$$m \ddot{x} + \mu \dot{x} + kx = 0$$

**The Process:**
1.  **Input:** You pass a time $t$ into the network.
2.  **Output:** The network guesses the position $x$.
3.  **Automatic Differentiation (The Key):** This is the missing piece of the puzzle. Because neural networks are differentiable graphs, we can use "autograd" (automatic differentiation) to calculate the derivatives of the output $x$ with respect to the input $t$.
    *   The network calculates the slope (velocity, $\dot{x}$).
    *   The network calculates the curvature (acceleration, $\ddot{x}$).
4.  **Calculating the Physics Residual:** The network takes those calculated values and plugs them into the physics equation:
    *   $Residual = m(\text{network\_acceleration}) + \mu(\text{network\_velocity}) + k(\text{network\_position})$
5.  **The Goal:** If the network perfectly understands the physics, this $Residual$ should be **0**. If it is not 0, the value is added to the Loss Function, and the optimizer adjusts the weights to force this residual closer to zero.

### 3. Training Data vs. Collocation Points
This leads to a major operational difference in how you feed data to the network:

*   **Normal NN (Data points):** You can only train the network where you have "labels" (actual known measurements). If you have measurement data at $t=0$ and $t=1$, the network learns those points but might draw a crazy squiggly line in between them because it has no constraints in the empty space.
*   **PINN (Collocation points):** You train on the known data points (Data Loss), **BUT** you also feed the network thousands of random input points called "collocation points" where you *don't* know the answer.
    *   At these random points, the network doesn't check against "expected outputs" (because you don't have them).
    *   Instead, it checks against the **Physics Equation**. It asks: "I don't know if my position prediction is right, but do my predicted acceleration and velocity balance out to zero according to the formula?".

### Summary Table

| Feature | Normal Neural Network | Physics-Informed Neural Network (PINN) |
| :--- | :--- | :--- |
| **Input** | $t$ (Time/Space) | $t$ (Time/Space) |
| **Output** | $u$ (Solution) | $u$ (Solution) |
| **Differentiation** | Used only for backpropagation (updating weights). | Used for **both** backpropagation **and** calculating physics derivatives ($\frac{du}{dt}, \frac{d^2u}{dt^2}$). |
| **Loss Function** | "Did I hit the training data points?" | "Did I hit the training data points **AND** does my curve obey the differential equation?". |
| **Where is the equation?** | Nowhere (it relies purely on data patterns). | In the **Loss Function**, calculated using derivatives of the output. |
| **Result** | Good at interpolation (connecting dots); bad at extrapolation (guessing outside data). | Good at extrapolation; the physics "guides" the solution in areas with no data. |