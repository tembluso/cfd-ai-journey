# Data

Observational data from the NASA InSight HP³ mission, as processed by Spohn et al. (2024). **These files are not included in the repository** — download them from the source below.

---

## Download

1. Go to the FigShare collection: [https://doi.org/10.6084/m9.figshare.25099754](https://doi.org/10.6084/m9.figshare.25099754)
2. Download the following `.xlsx` files and place them in this `data/` directory:

| Filename | Description |
|----------|-------------|
| `Mars_Soil_Temperatures_and_Thermal_Properties_from_Diurnal_Wave_data.xlsx` | RAD surface temperatures and TEM-A subsurface measurements over single sols |
| `Mars_Soil_Temperatures_and_Thermal_Properties_from_Annual_Wave_Data.xlsx` | Seasonal temperature variations and derived thermal properties |

---

## Instruments

- **RAD** — Radiometer on the lander deck; measures surface brightness temperature (boundary condition at x = 0)
- **TEM-A** — Temperature sensors on the HP³ mole; measures subsurface temperature at ~0.094 m depth (observational training data)

---

## Citation

> Spohn, T. (2024). Mars soil temperature and thermal diffusivity from InSight HP³ data workbook [Dataset, Software]. FigShare Collection. [https://doi.org/10.6084/m9.figshare.25099754](https://doi.org/10.6084/m9.figshare.25099754)

> Spohn, T., Müller, N., Knollenberg, J., Grott, M., Golombek, M. P., Plesa, A.‐C., et al. (2024). Mars soil temperature and thermal properties from InSight HP³ data. *Geophysical Research Letters*, 51, e2024GL108600. [https://doi.org/10.1029/2024GL108600](https://doi.org/10.1029/2024GL108600)

---
 
## Expected structure
 
```
resources/data/
├── README.md  (this file)
├── .gitkeep
├── Mars_Soil_Temperatures_and_Thermal_Properties_from_Diurnal_Wave_data.xlsx  (not tracked)
└── Mars_Soil_Temperatures_and_Thermal_Properties_from_Annual_Wave_Data.xlsx   (not tracked)
```
