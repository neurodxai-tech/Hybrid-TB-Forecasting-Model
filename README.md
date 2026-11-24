# TB notifications in Nepal: hybrid SARIMA–CNNAR forecasting (analysis code)

Code for the manuscript *“Data-Driven Hybrid Model of SARIMA-CNNAR For Tuberculosis Incidence Time Series Analysis in Nepal”* (Public Health Challenges, manuscript 3049755).

## Files

| File | Purpose |
|---|---|
| `TB_SARIMA_CNNAR_Analysis.ipynb` | Clean notebook (no outputs). Run this one. |
| `TB_SARIMA_CNNAR_Analysis_executed.ipynb` | The same notebook, executed end-to-end, with all outputs (the run behind the manuscript numbers). |
| `tb_forecasting.py` | The notebook as a plain Python script, in "percent" cell format (`# %%`). Easy to diff and review, and opens as a notebook in VS Code or with Jupytext. |
| `requirements.txt` | Exact package versions used for the published results. |

## Quick start

**1. One-time setup.** Use a dedicated environment; Anaconda's *base* environment often has incompatible packages (for example TensorFlow built for NumPy 1.x next to NumPy 2, or a broken Pillow). In Terminal:

```bash
cd "<folder containing these files>"
bash setup_environment.sh
```

The script finds conda, creates the environment `tb_forecast` (Python 3.12), installs compatible versions of all packages (pip chooses the right TensorFlow/NumPy pair for Apple-silicon and Intel Macs), registers the Jupyter kernel "Python (tb_forecast)" and checks that everything imports. Your base environment is not changed.

**2. Run.** Open `TB_SARIMA_CNNAR_Analysis.ipynb` in Jupyter, choose **Kernel → Change kernel → Python (tb_forecast)**, then **Kernel → Restart Kernel and Run All Cells**. Cell 1.0 checks the environment first; it should end with `Environment OK.`

## Input and output locations

Set in **Section 1** of the notebook (already set to the project folders):

* `DATA_FILE` = `…/Data Driven Hybrid Model of SARIMACNNAR For Tuberculosis Incidence Time Series Analysis in Nepal/TB_Incidence.csv`
* `OUTPUT_DIR` = `…/Final after revision Submitted files/Outputs of Model`
* `FIG_DPI = 600`, `FIG_FORMAT = "tif"`: every figure is saved as a 600 dpi, RGB, LZW-compressed TIFF on a white background.

The CSV can be wide (a month-name column plus one column per year, 2015–2024 or 2018–2024) or long (a date column plus a value column). Before any analysis, it is checked against the published dataset; see `06_Logs_and_Reproducibility/Data_check_input_vs_published.csv`. If 2015–2017 are missing from the file, they are back-cast with the month-specific CAGR method.

## Running on demand

All settings are in **Section 1** of the notebook:

* `RUN = dict(...)` switches stages on or off (EDA, grid search, cross-validation, extended analyses, sensitivity, 2025 forecasts, figures).
* Expensive stages are cached in `Outputs of Model/_cache/` in a sub-folder tied to the run mode and to the input data, so changing the CSV automatically triggers a recomputation. After the first run, re-running the notebook takes about a minute. `USE_CACHE = False`, or deleting the cache folder, recomputes everything.
* `QUICK_TEST = True` is a smoke test that finishes in about 15 minutes. Its numbers are **not** the manuscript numbers.

A first full run takes about 2 hours on 2 CPU cores, mostly neural-network training: 8 models × 5 folds × 5 seeds, plus the extended and sensitivity analyses.

## Outputs

```
Outputs of Model/
├── 01_Manuscript_Tables/          Table01 … Table18 (CSV) + Manuscript_Tables.xlsx (one sheet per table)
├── 02_Supplementary_Tables/       S1 … S9 (CSV + XLSX)
├── 03_Model_Outputs/              2025 forecasts, cross-validation forecasts, final SARIMA/SARIMAX summaries, parameters, fitted values
├── 04_Exploratory_and_Tests/      descriptive statistics and statistical tests
├── 05_Figures_600dpi_TIFF/        Fig01 … Fig22 (.tif, 600 dpi)
├── 06_Logs_and_Reproducibility/   environment, input-data check, SARIMA fit log, reproducibility check
├── Outputs_index.csv              list of every output file
└── _cache/                        saved results of the expensive stages (safe to delete; specific to the dataset)
```

## Reproducibility

* Seeds are fixed, and TensorFlow deterministic operations are on, so a rerun on the same platform gives identical results.
* Statistical models and tests are deterministic on any platform. Neural networks and XGBoost can differ slightly with other hardware or library versions.
* The last section of the notebook compares key results with the published values and reports `PASS` / `CHECK` / `FAIL`.

## Method notes

* The data (2018–2024 observed; 2015–2017 back-cast by month-specific CAGR) are embedded in Section 2 and checked against S1 Table.
* Scalers are fitted on training data only. Break dates are re-estimated inside each fold. The lockdown indicator is set to 0 over every forecast horizon, so no future information is used.
* SARIMA fits whose unconstrained likelihood is degenerate are refitted with stationarity and invertibility enforced. All fits are logged in `06_Logs_and_Reproducibility/SARIMA_fit_log.csv`.

## Troubleshooting

| Message | Cause | Fix |
|---|---|---|
| `ImportError: dlopen(... PIL/_imaging ...) Symbol not found: __ZNKSt3__1...` | Pillow's compiled library does not match the C++ runtime of an Anaconda environment (common on macOS) | Best: use the dedicated environment above. Quick fix in the current environment: `pip install --force-reinstall --no-cache-dir pillow` (or `conda install -c conda-forge pillow libcxx`), then restart the kernel |
| `NameError: name 'save_table' is not defined` (or any other `NameError`) | An earlier cell failed, usually the import cell, so its functions were never defined | Fix the first error shown (cell 1.0 names it), restart the kernel and run all cells again |
| `ModuleNotFoundError: No module named ...` | Package not installed in the kernel's environment | `pip install -r requirements.txt` in the environment of the selected kernel; check the kernel name at the top right of Jupyter |
| `A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x` (from TensorFlow) | TensorFlow and NumPy versions do not match, typically in the Anaconda base environment | Run `bash setup_environment.sh` and use the kernel "Python (tb_forecast)". In an existing environment: `pip install -U tensorflow` (Apple silicon) or `pip install "numpy<2"` (Intel Mac) |
| `conda-libmamba-solver ... Symbol not found ... libc++.1.0.dylib` / `CondaValueError: ... libmamba ... not recognized` | conda's default solver is broken in this Anaconda install | `CONDA_SOLVER=classic bash setup_environment.sh` (the current script already uses the classic solver) |
| `XGBoost Library (libxgboost.dylib) could not be loaded ... libomp.dylib` | OpenMP runtime missing on macOS | `CONDA_SOLVER=classic conda install -n tb_forecast -c conda-forge llvm-openmp -y` (or `brew install libomp`), then restart the kernel |
| Cell 1.0 prints “this kernel runs in the Anaconda *base* environment” | Wrong kernel selected | Kernel → Change kernel → Python (tb_forecast) |
| TensorFlow will not install | The platform or Python version has no TensorFlow build (e.g. Intel Macs support TensorFlow only up to 2.16) | Use Python 3.11 or 3.12; on an Intel Mac, `pip install "tensorflow==2.16.*"` |
| A neural-network result is flagged `CHECK` in Section 16 | Different hardware or library versions | Expected; statistical results must still `PASS` |

## Running from this repository
```bash
conda create -n tb_forecast python=3.12 -y && conda activate tb_forecast
pip install -r requirements-lock.txt      # exact versions used for the manuscript
export TB_DATA_FILE="$PWD/data/TB_Incidence.csv"
export TB_OUTPUT_DIR="$PWD/Outputs"
jupyter nbconvert --to notebook --execute --ExecutePreprocessor.timeout=-1 \
  TB_SARIMA_CNNAR_Analysis.ipynb --output TB_SARIMA_CNNAR_Analysis_executed.ipynb
cat Outputs/06_Logs_and_Reproducibility/reproducibility_check.csv
```
Neural networks are retrained from fixed seeds (no stored weights). Deterministic results must PASS exactly;
neural/XGBoost/Prophet results may show CHECK on other hardware or library versions (tolerance 0.5 MAPE points).
`results/reproducibility_check.csv` is the check from the manuscript run.

**macOS:** XGBoost needs the OpenMP runtime, which pip does not install. After `pip install`, run
`conda install -c conda-forge llvm-openmp -y` (or `brew install libomp`).
Register the environment as a kernel and execute with it, so the notebook does not run in another environment:
`python -m ipykernel install --user --name tb_forecast` and add `--ExecutePreprocessor.kernel_name=tb_forecast` to the `nbconvert` command.
