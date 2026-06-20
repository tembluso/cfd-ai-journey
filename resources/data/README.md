# Data

Observational data from the NASA InSight HP3 mission, as processed by Spohn et al. (2024). Used in Weeks 7-12 of this project.

---

## Background

NASA's InSight lander operated on Mars from 2018 to 2022. Its HP3 (Heat Flow and Physical Properties Package) instrument included a self-hammering penetrator called the "mole", designed to burrow up to 5 m into the Martian regolith to measure the planet's heat flow. The mole got stuck at ~37 cm depth due to unexpectedly low soil friction, but its temperature sensors still recorded valuable subsurface thermal data.

Two instruments provide the data used in this project:

- **RAD** — A radiometer on the lander deck that measures surface brightness temperature. Used as the top boundary condition ($x = 0$) in all PINN models.
- **TEM-A** — Thermistors mounted along the mole's support structure (tether), measuring subsurface temperature at ~0.094 m depth. Used as interior observational data to constrain the PINN solution.

---

## Download

1. Go to the FigShare collection: [https://doi.org/10.6084/m9.figshare.25099754](https://doi.org/10.6084/m9.figshare.25099754)
2. Download the following `.xlsx` files and place them in this `data/` directory:

| Filename | Description | Used in |
|----------|-------------|---------|
| `Mars Soil Temperatures and Thermal Properties from Diurnal Wave data.xlsx` | RAD surface temperatures (Sol 511) and TEM-A subsurface measurements (Sol 1202) over single sols | Weeks 7-12 |
| `Mars Soil Temperatures and Thermal Properties from Annual Wave Data.xlsx` | Seasonal temperature variations over ~600 sols and derived thermal properties | Week 7 (visualization) |

### Key sheets used by the notebooks

**Diurnal file:**
- `TEM-A Sol 1202` — Mole temperature over one sol. Column 6 = LTST (hours), Column 7 = average of sensors T15, T16 (Kelvin).
- `RAD SOL 511 D` — Surface temperature from the radiometer. Column 4 = LTST (hours), Column 5 = FOV 2 temperature (Kelvin). Spohn et al. use this sol as the surface proxy for the warmest sampled period.

**Annual file:**
- `2nd year Temp (Fig. 2)` — Daily-averaged temperatures from multiple sensors over the second Martian year of the mission.

---

## Other files

| File | Description |
|------|-------------|
| `fourier_coefficients.pkl` | Pre-computed Fourier series coefficients fitted to the RAD surface temperature data. Used in Weeks 10-12 to provide an analytical surface boundary condition instead of raw discrete measurements. |

---

## Citation

> Spohn, T. (2024). Mars soil temperature and thermal diffusivity from InSight HP3 data workbook [Dataset, Software]. FigShare Collection. [https://doi.org/10.6084/m9.figshare.25099754](https://doi.org/10.6084/m9.figshare.25099754)

> Spohn, T., Muller, N., Knollenberg, J., Grott, M., Golombek, M. P., Plesa, A.-C., et al. (2024). Mars soil temperature and thermal properties from InSight HP3 data. *Geophysical Research Letters*, 51, e2024GL108600. [https://doi.org/10.1029/2024GL108600](https://doi.org/10.1029/2024GL108600)

---

## Expected structure

```
resources/data/
├── README.md                                                                   (this file)
├── fourier_coefficients.pkl                                                    (generated)
├── Mars Soil Temperatures and Thermal Properties from Diurnal Wave data.xlsx   (download)
└── Mars Soil Temperatures and Thermal Properties from Annual Wave Data.xlsx    (download)
```
