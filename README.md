# CFD + AI Learning Journey

A 13-week hands-on exploration of Physics-Informed Neural Networks (PINNs) applied to computational fluid dynamics, culminating in a research application: recovering Martian soil thermal diffusivity from NASA InSight HP3 mission data using JAX. The final research paper is also included (Sanchez_Cheble_PINNs_Mars_Thermal_Diffusivity_2026)

## What This Repo Covers

**Weeks 1-6** build foundational PINN skills on increasingly complex PDEs:

| Week | Topic | PDE | Key Concept |
|------|-------|-----|-------------|
| 1 | [Foundations](week-01-foundations/) | Harmonic oscillator | What is a PINN, JAX basics |
| 2 | [First PINN](week-02-first-pinn/) | 1D heat equation | Physics loss vs data loss, boundary conditions |
| 3 | [Nonlinear Convection](week-03-nonlinear-convection/) | Inviscid Burgers | Shock formation, PINN limitations at discontinuities |
| 4 | [Burgers Equation](week-04-Burgers/) | Viscous Burgers | Convection + diffusion interaction, wave collisions |
| 5 | [2D Problems](week-05-2D/) | 2D diffusion, 2D Burgers | Extending PINNs to higher dimensions (3 inputs) |
| 6 | [Poisson & Navier-Stokes](week-06-Poisson-Navier-Stokes/) | Poisson, incompressible NS | Steady-state PDEs, incompressibility constraint, Fourier features |

**Weeks 7-13** apply PINNs to real Mars thermal data:

| Week | Topic | Focus |
|------|-------|-------|
| 7 | [Mars Data](week-07-data-on-Mars/) | InSight HP3 data exploration, first Mars PINN |
| 8 | [Forward Optimization](week-08-better-Mars-PINN/) | Systematic optimization of the forward problem |
| 9 | [Inverse Problem](week-09-Inverse-problem/) | Learning thermal diffusivity (kappa) from data |
| 10 | [Optimal Inverse](week-10-optimal-Inverse/) | Spatial averaging, 2D axisymmetric perturbance model |
| 11 | [Decomposition](week-11-decompostion+1D/) | T = T_1D + T_pert decomposition, layer-wise models |
| 12 | [Verification](week-12-verify/) | Statistical verification across multiple initializations |
| 13 | [Noise Robustness](week-13-noisy-spatial-inverse/) | Sensitivity to measurement noise (3 noise seeds) |

## The Mars Problem

The HP3 instrument on NASA's InSight lander measured subsurface temperatures on Mars using the TEM-A sensors on a penetrating probe (the "mole"). The mole got stuck at 36.3 cm depth, but its temperature readings, combined with the RAD surface radiometer, provide a unique dataset for studying Martian soil thermal properties.

The core question: **can a PINN recover the soil thermal diffusivity (kappa) from temperature measurements alone?**

The journey progresses through:
1. **Forward problem** (Weeks 7-8): Given a known kappa, predict T(x,t). Required extensive optimization — the key breakthrough was dividing the PDE residual by kappa to balance term magnitudes.
2. **Inverse problem** (Weeks 9-10): Learn kappa from data. Single-kappa recovery independently validated Spohn et al.'s published value (3.93 x 10^-8 m2/s). Multi-kappa and 2D approaches revealed structural identifiability challenges.
3. **Decomposition** (Week 11): Splitting the solution as T = T_1D + T_pert to isolate the mole's thermal perturbation from undisturbed soil temperature.
4. **Verification** (Weeks 12-13): Statistical validation across multiple initializations and noise robustness testing.

Final paper pdf document: (Sanchez_Cheble_PINNs_Mars_Thermal_Diffusivity_2026)

## Prerequisites

- Python 3.10+
- JAX (GPU recommended but CPU works for weeks 1-6)
- NumPy, Matplotlib, SciPy
- Jupyter

```bash
pip install jax jaxlib numpy matplotlib scipy jupyter optax
```

## Data

Weeks 7-13 use InSight HP3 data from [Spohn et al. (2024)](https://doi.org/10.1029/2023GL105588). Download the Excel files from [FigShare](https://doi.org/10.6084/m9.figshare.25099754) and place them in `resources/data/`:

- `Mars Soil Temperatures and Thermal Properties from Annual Wave Data.xlsx`
- `Mars Soil Temperatures and Thermal Properties from Diurnal Wave data.xlsx`

## References

- Raissi, M., Perdikaris, P., & Karniadakis, G. E. (2019). [Physics-informed neural networks](https://www.sciencedirect.com/science/article/abs/pii/S0021999118307125). *Journal of Computational Physics*.
- Spohn, T., et al. (2024). Mars Soil Temperatures and Thermal Properties from InSight HP3. *Geophysical Research Letters*.
- Barba, L. A. [CFD Python: 12 Steps to Navier-Stokes](https://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/).
- Moseley, B. [So What Is a Physics-Informed Neural Network?](https://benmoseley.blog/my-research/so-what-is-a-physics-informed-neural-network/)
