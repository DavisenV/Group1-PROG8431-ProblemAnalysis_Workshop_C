# PROG8431 Problem Analysis Workshop C

**Transformations and linear regression on a predictive maintenance dataset**

Group 1: Davisen Veerasamy, Carlos Gutierrez, Nnamdi Ikengah

## Goal

Estimate the expected **torque** of a milling machine from its other readings, and check whether torque far from the expected value is a warning sign of failure.

All the work is in [`Workshop_C.ipynb`](Workshop_C.ipynb).

## Setup

Requires **Python 3.14 or later**.

```bash
git clone https://github.com/DavisenV/Group1-PROG8431-ProblemAnalysis_Workshop_C.git
cd Group1-PROG8431-ProblemAnalysis_Workshop_C
```

With [uv](https://docs.astral.sh/uv/):

```bash
uv sync
```

Or with pip:

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
```

Open the notebook, select the `.venv` kernel, then restart the kernel and run all cells. Run it from the repository root, because the data paths are relative. The data files are included, so no download is needed.

## Data

| Source | Used for | File |
|---|---|---|
| [AI4I 2020 Predictive Maintenance Dataset](https://archive.ics.uci.edu/dataset/601/ai4i+2020+predictive+maintenance+dataset) (UCI), 10,000 runs | Machine readings and failure flags | `data/predictive_maintenance.csv` |
| AI4I dataset documentation | Product type specifications and failure rules | `data/product_type_specs.csv` |
| Environment and Climate Change Canada, Kitchener/Waterloo station | Hourly weather | `data/external/weather_kitchener_waterloo_hourly.csv` |
| IESO | Hourly Ontario energy price | `data/external/ieso_hoep_hourly.csv` |
| IESO | Hourly Ontario demand | `data/external/ieso_ontario_demand_hourly.csv` |

The notebook writes the combined study file, with 45 variables plus a timestamp, to `data/study_file.csv`.

**Assumption:** the AI4I data has no date or time column. To join the weather and electricity data, we assume one run per hour in the original order, starting 2024-01-01 00:00 EST.

## Method

1. Clean the data, add the external sources and create new variables, including Tukey's ladder transformations.
2. Split into training (80%) and test (20%) sets and compare them with Kolmogorov-Smirnov and chi-square tests.
3. Compare two simple linear regressions (rotational speed and tool wear), then refine the better one.
4. Fit a multiple linear regression with all legitimate predictors and reduce it with backward selection.
5. Check the final model and use its errors to flag unusual runs.

Variables calculated from or defined by torque, and failure outcomes known only after a run, are left out of the models to avoid data leakage.

## Results

**Final model:** torque = −142.78 + 0.0282 × speed + 212.05 × (1000 / speed), with speed in rpm and torque in Nm.

| Model | R² (training) | Test RMSE (Nm) |
|---|---|---|
| Simple: tool wear | 0.000 | 9.98 |
| Simple: rotational speed | 0.765 | 4.79 |
| Simple: 1000 / rotational speed | 0.837 | 4.01 |
| Full multiple regression (26 terms) | 0.848 | 3.89 |
| **Final model (2 terms)** | **0.848** | **3.88** |

- The weather and electricity variables did not help predict torque.
- Test runs with torque far **above** the expected value failed 30.0% of the time, against 2.7% for normal runs. This catches only 18 of 72 failures, mostly power and overstrain.

## Limitations

- Above 2,700 rpm the final model over-predicts torque.
- The residual spread is not constant.
- The dataset is simulated, and the join with the external data rests on an assumed timeline.

## Repository layout

```
Workshop_C.ipynb    Analysis and discussion
data/               Input data and the generated study file
docs/               Assignment brief and feedback form
pyproject.toml      Dependencies (locked in uv.lock)
requirements.txt    Exported from uv.lock for pip users
```
