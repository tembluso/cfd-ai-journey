# CFD+AI Journey Progress Tracker

## Overview
- **Start Date:** Jan 27 2026
- **Goal:** Build original PINN contribution in 12 weeks
- **Public Accountability:** (https://x.com/tembluso/status/2016652932263862408?s=20)

## Weekly Progress

### Week 1: Foundations (Target: Jan 27 - Feb 2)
**Goals:**
- [Done] Set up infrastructure (GitHub, Claude Projects, NotebookLM)
- [Done] Understand what a PDE is
- [Done] Solve 1D heat equation with traditional method
- [Done] Read PINN paper (at least abstract + intro + Section 2)
- [Done] First X update posted

**What I Built:**
-Complete set up of claude prohjects, notebooks and habit tracker
-Turned a Pytorch PINN into a my own JAX PINN.

**What I Learned:**
-Basic differences between JAX and Pytorch and why JAX is faster.
-ODEs and PDEs
-Fourier series
-What are PINNs
-How to derive and solve the heat equation

---

### Week 2: First PINN (Target: Feb 3 - Feb 9)
**Goals:**
- [DONE] Implement PINN for 1D heat equation
- [DONE] Get it to converge
- [DONE] Compare to traditional solver
- [DONE] Blog post written
- [DONE] Learn linear and non-linear convection

**What I Built:**
-PINN for 1D heat euqation

**What I Learned:**
-How to create my first PINN from scratch for the heat equation.
-Learned convection equations both linear and non linear.

### Week 3: Nonlinear Convection
**Goals:**
- [DONE] Implement PINN for non linear convection
- [DONE] Get it to converge
- [DONE] Understand PINN limitations to model shocks
- [DONE] Blog post written
- [DONE] Learn basics of burger's equation

**What I Built:**
-PINN for nonlinear convection
-mega thread posting my results of the PINN with visuals
-Megathread explaining burgers equations with visuals (very good thread but no more theory threads for equations I prefer to do PINN directly.)

**What I Learned:**
-Learned about discontinuities in differential equations.
-Learned that PINNs are not able to model discontinuities.
-Learned on why steepening happends.
-Learned the basics of burgers equation

**Preview Next week**
-Model Burger's equation using a PINN
---

[... Continue for all 12 weeks ...]

## Key Concepts Mastered
- [YES] Partial derivatives
- [YES] Boundary conditions
- [YES] Fourier Series
- [YES] Automatic differentiation
- [YES] JAX basics (jit, vmap, grad)
- [YES] Neural network training
- [YES] Physics loss vs data loss
- [YES] Heat equation
- [YES] Linear convection
- [YES] Non linear convection
- [YES] Why PINNs struggle at discontinuities
- [YES] Theory of Burger's equation
- [ ] Navier-Stokes equations
- [ ] etc...

## Resources Used
- [3Blue1Brown Youtube DE Series](https://www.youtube.com/watch?v=r6sGWTCMz2k)
- [CFD Python - 12 Steps to Navier-Stokes](https://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/)
- [Harmonic Oscillator PINN Tutorial](https://benmoseley.blog/my-research/so-what-is-a-physics-informed-neural-network/)
- [Claude Projects](https://claude.ai)
- 

## People Connected With
-@Abderrazakasmi Connected through X after a post.
-@mathelirium Started following me after heat pinn.
-
```

**Update this every Sunday.** Then share summary on X.