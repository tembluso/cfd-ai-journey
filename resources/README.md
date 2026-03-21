# Resources

External papers, tutorials, and references used throughout this PINN learning journey. **No PDFs are stored in this repo** — all papers are linked to their original sources below.

---

## Papers

### Physics-Informed Neural Networks (PINNs)

> Raissi, M., Perdikaris, P., & Karniadakis, G. E. (2019). Physics-informed neural networks: A deep learning framework for solving forward and inverse problems involving nonlinear partial differential equations. *Journal of Computational Physics*, 378, 686–707.

- **DOI:** [https://doi.org/10.1016/j.jcp.2018.10.045](https://doi.org/10.1016/j.jcp.2018.10.045)
- **Used in:** Weeks 1–6 (theoretical foundation for all PINN implementations)

### Neural Operators

> Kovachki, N., Li, Z., Liu, B., Azizzadenesheli, K., Bhattacharya, K., Stuart, A., & Anandkumar, A. (2022). Neural Operator: Learning Maps Between Function Spaces With Applications to PDEs. *Journal of Machine Learning Research*, 23(1), 1–97.

- **DOI / arXiv:** [https://arxiv.org/abs/2108.08481](https://arxiv.org/abs/2108.08481)
- **Used in:** Background reading on operator learning approaches

### GraphCast

> Lam, R., Sanchez-Gonzalez, A., Willson, M., et al. (2023). Learning skillful medium-range global weather forecasting. *Science*, 382(6677), 1416–1421.

- **DOI:** [https://doi.org/10.1126/science.adi2336](https://doi.org/10.1126/science.adi2336)
- **Repo:** [https://github.com/google-deepmind/graphcast](https://github.com/google-deepmind/graphcast)
- **Used in:** Inspiration for ML-based physical modelling at scale

### Mars Soil Temperature (InSight HP³)

> Spohn, T., Müller, N., Knollenberg, J., Grott, M., Golombek, M. P., Plesa, A.‐C., et al. (2024). Mars soil temperature and thermal properties from InSight HP³ data. *Geophysical Research Letters*, 51, e2024GL108600.

- **DOI:** [https://doi.org/10.1029/2024GL108600](https://doi.org/10.1029/2024GL108600)
- **License:** CC BY-NC-ND 4.0
- **Used in:** Weeks 7–8 (Mars surface temperature PINN — data source and physical context)
- **Dataset:** See [`data/README.md`](data/README.md) for download instructions

---

## Tutorials & Blogs

### Harmonic Oscillator PINN — Ben Moseley

- **Blog post:** [So, what is a physics-informed neural network?](https://benmoseley.blog/my-research/so-what-is-a-physics-informed-neural-network/)
- **Repo:** [https://github.com/benmoseley/harmonic-oscillator-pinn](https://github.com/benmoseley/harmonic-oscillator-pinn)
- **Used in:** Early PINN intuition and theoretical foundation

---

## Video Series

### 3Blue1Brown — Differential Equations

- **Playlist:** [https://www.youtube.com/watch?v=r6sGWTCMz2k](https://www.youtube.com/watch?v=r6sGWTCMz2k)
- **Used in:** Building intuition for PDEs from the start

---

## Reference Repos

### CFD Python — 12 Steps to Navier-Stokes (Lorena Barba)

- **Webpage:** [https://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/](https://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/)
- **Repo:** [https://github.com/barbagroup/CFDPython](https://github.com/barbagroup/CFDPython)
- **Used in:** Steps 1–12 as reference for finite difference intuition before PINN implementations

---

## Mars Data (Raw Sources)

These are the original NASA PDS archives if you want to go deeper than Spohn's processed dataset:

- **HP³ overview:** [https://pds-geosciences.wustl.edu/missions/insight/hp3rad.htm](https://pds-geosciences.wustl.edu/missions/insight/hp3rad.htm)
- **TEM calibrated data:** [https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_hp3_tem/data_tem_calibrated/](https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_hp3_tem/data_tem_calibrated/)
- **TEM raw data:** [https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_hp3_tem/data_tem_raw/](https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_hp3_tem/data_tem_raw/)
- **RAD derived data (surface temperature):** [https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_rad/data_derived/](https://pds-geosciences.wustl.edu/insight/urn-nasa-pds-insight_rad/data_derived/)
