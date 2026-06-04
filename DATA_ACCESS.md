# Data Access

Input data are obtained from two openly accessible sources. Neither dataset
is redistributed in this repository; instructions for downloading them are
given below.

---

## 1. Discharge Data — GRDC

Daily discharge records for all five stations were downloaded from the
**Global Runoff Data Centre (GRDC)**, Federal Institute of Hydrology,
Koblenz, Germany.

**URL:** https://www.bafg.de/GRDC

**Registration:** Free registration is required. Data are provided for
scientific research purposes under the GRDC terms of use.

**Stations and GRDC IDs:**

| Station | River | GRDC Station ID |
|---|---|---|
| Nowy Targ | Dunajec | 6340400 |
| Nowy Sącz | Dunajec | 6340500 |
| Strekowa Góra | Narew | 6370150 |
| Sieradz | Warta | 6340820 |
| Sandomierz | Vistula | 6371100 |

**Format:** Download as semicolon-separated `.txt` files with comment lines
starting with `#`. No manual editing is required before running the script.

**Period:** Select the full available record (script uses 1984–2025 only).

**File naming convention expected by the script:**

```
nowy_sacz__dunajec__grdc.txt
nowy_targ__dunajec__grdc.txt
strekowa_gora__narev__grdc.txt
sieradz__warta__grdc.txt
sandomierz__vistula__grdc.txt
```

---

## 2. Meteorological Data — NASA POWER

Daily meteorological data were downloaded from **NASA POWER** (Prediction Of
Worldwide Energy Resources).

**URL:** https://power.larc.nasa.gov/data-access-viewer/

**Access:** Free, no registration required. The POWER Data Access Viewer
supports both interactive downloads and a REST API.

**Variables downloaded (Daily, point location):**

```
T2M, T2M_MAX, T2M_MIN, T2MDEW,
ALLSKY_SFC_SW_DWN, ALLSKY_SFC_LW_DWN,
RH2M, PRECTOTCORR, WS2M, PS,
GWETTOP, GWETROOT, GWETPROF
```

**Coordinates used (catchment centroids, approximate):**

| Station | Latitude | Longitude |
|---|---|---|
| Nowy Targ | 49.58 | 20.03 |
| Nowy Sącz | 49.62 | 20.72 |
| Strekowa Góra | 52.95 | 22.10 |
| Sieradz | 51.59 | 18.73 |
| Sandomierz | 50.68 | 21.75 |

**Period:** 1984-01-01 to 2025-10-31

**Format:** Download as CSV. Select `CSV (Daily)` format. The file has 22
header rows followed by the data; the script handles this automatically.

**File naming convention expected by the script:**

```
nowy_sacz__dunajec__nasa.csv
nowy_targ__dunajec__nasa.csv
strekowa_gora__narev__nasa.csv
sieradz__warta__nasa.csv
sandomierz__vistula__nasa.csv
```

---

## Notes

- GRDC station IDs above are provided in good faith based on the stations
  used in the study. Confirm the correct ID on the GRDC portal before
  downloading.
- NASA POWER coordinates are approximate catchment centroids. Exact
  coordinates do not critically affect results given the 0.5° spatial
  resolution of the POWER product.
- Both datasets must be placed in the same directory as the script, or
  uploaded to the Colab `/content/` directory, before running.
