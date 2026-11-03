# How many days ahead can a river forecast really be trusted?

This repository contains the code behind the manuscript *"When Does a Streamflow Forecast Stop Being a Forecast? A SHAP-Based Framework for Detecting Predictability Horizon Collapse Across Contrasting Polish Catchments"* by Tevfik Denizhan Müftüoğlu (under review).

## The problem in plain words

Machine-learning models can forecast how much water a river will carry tomorrow, in three days or in two weeks. Such forecasts are used for flood warning, reservoir operation and water supply.

The further ahead the forecast, the less the model knows about the actual situation in the river. At some point it stops forecasting and simply returns what is *usual for that time of year*. For example, it says "rivers are high in spring" whatever the real conditions are. The forecast still looks like a forecast, but it has become a calendar.

The usual way to find this point is to compare forecasts with what actually happened. That needs years of observations and does not show *why* the forecast fails.

## What this code does

It looks inside the model and checks what the model bases each forecast on. It uses **SHAP**, a standard method that measures how much each input (yesterday's river flow, rainfall, soil wetness, the date, and so on) contributes to a forecast.

The code then finds the first forecast day at which **the date becomes one of the three most important inputs**. From that day on, the model relies mainly on the season instead of the current state of the river. This day is called the **SHAP-based predictability horizon (SHAP-H)**.

In short, the code tells you *how many days ahead the model is still forecasting the river, and from which day it is only repeating the seasonal pattern*.

## Example result

The method was tested on five rivers in Poland:

| Station | River type | Model relies on the season from | Forecast accuracy acceptable up to |
|---|---|---|---|
| Nowy Targ | small, fast mountain river | day 4 | day 1 |
| Nowy Sącz | mountain river | day 4 | day 2 |
| Sandomierz | large river (Vistula) | day 6 | day 5 |
| Sieradz | slow lowland river | not within 14 days | day 7 |
| Strękowa Góra | slow lowland river | not within 14 days | day 14 or more |

Fast mountain rivers lose their predictability within a few days. Slow lowland rivers keep it for two weeks or longer.

## How to run it

**You need:**
- daily river flow for each station from the [Global Runoff Data Centre (GRDC)](https://grdc.bafg.de/data/data_portal/), free for research;
- daily weather data for the same locations from [NASA POWER](https://power.larc.nasa.gov/data-access-viewer/), free.

The data are not included here because GRDC does not allow redistribution.

**Steps (Google Colab, no installation needed):**
1. Open `streamflow_shapH.ipynb` in [Google Colab](https://colab.research.google.com).
2. Run the cell. A **Choose Files** button appears; select the 10 data files (one flow file and one weather file per station).
3. Wait about two hours. A zip file with all tables and figures is downloaded at the end.

The script lists the exact file names it expects at the top. To use your own river, prepare the two files for it in the same format and add a line for it to the `BASINS` list in the script.

## Files

| File | Content |
|---|---|
| `streamflow_shapH.ipynb` | The analysis, ready for Google Colab |
| `requirements.txt` | Package versions used for the published results |
| `shapH_outputs.zip` | Results of the published run: tables, figures and run log |

## Data note

Do not upload GRDC data files to public repositories.

## Author

Tevfik Denizhan Müftüoğlu
