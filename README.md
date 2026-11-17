# Landslide Data Imputation and Runout Distance Prediction

The six imputation notebooks use variational autoencoders (VAEs) and ensemble models to fill missing values in landslide predictors. `Runout distance.ipynb` trains a generative adversarial network (GAN) ensemble to predict runout distance `L(m)` from the completed predictors.

## Setup

From this folder in PowerShell, create a Python 3.12 environment and start JupyterLab.

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install jupyterlab tensorflow==2.21.0 scikit-learn numpy pandas scipy torch matplotlib openpyxl
.\.venv\Scripts\python.exe -m jupyterlab
```

## Inputs and run order

Place the input CSVs in the notebook directory. `L(m)` is the runout-distance column in metres. The imputation notebook excludes it from predictor training and restores it by row in the second cell. Leave missing predictor values blank.

If using the supplied Excel workbook, run this preparation code once from the notebook directory. It exports the required columns with the headers expected by the notebooks, preserving row order and leaving existing CSVs in place.

```python
from pathlib import Path

import pandas as pd

workbook = Path("Mixed") / "Landslide runout distancce dataset.xlsx"
sheets = {
    "Earthquake": ("Earthquake", ["H", "At", "Wa"]),
    "Mixed": ("Mixed landslides", ["H", "V"]),
    "Others": ("Others", ["H", "V"]),
    "Rainfall": ("Rainfall", ["As", "H", "V", "Wsm", "θ"]),
    "Rock": ("Rock landslides", ["H", "As", "V", "Wsm", "θ"]),
    "Soil": ("Soil landslides", ["H", "As", "At", "V", "Wsa"]),
}
headers = {
    "L": "L(m)", "H": "H(m)", "At": "At(m²）", "As": "As(m²）",
    "Wa": "Wa(m)", "Wsa": "Wsa(m)", "Wsm": "Wsm(m)", "V": "V(m³)", "θ": "θ(°)",
}
with pd.ExcelFile(workbook, engine="openpyxl") as excel:
    for dataset, (sheet, predictors) in sheets.items():
        output = Path(f"{dataset}.csv")
        if output.exists():
            continue
        columns = predictors + ["L"]
        frame = pd.read_excel(excel, sheet_name=sheet, usecols=columns)[columns]
        frame.rename(columns=headers).to_csv(output, index=False)
```

| Imputation notebook prefix | Input CSV | File passed to the runout notebook |
| --- | --- | --- |
| `Earthquake-` | `Earthquake.csv` | `imputed_earthquake_with_runout.csv` |
| `Mixed-` | `Mixed.csv` | `imputed_mixed_with_runout.csv` |
| `Others-` | `Others.csv` | `imputed_others_with_runout.csv` |
| `Rainfall-` | `Rainfall.csv` | `imputed_rainfall_with_runout.csv` |
| `Rock-` | `Rock.csv` | `imputed_rock_with_runout.csv` |
| `Soil-` | `Soil.csv` | `imputed_soil_with_runout.csv` |

1. Open the imputation notebook for your dataset and run both cells from top to bottom. The first cell writes `imputed_<dataset>.csv` and `evaluation_<dataset>.csv`. The second cell writes the corresponding `imputed_<dataset>_with_runout.csv`.
2. Open `Runout distance.ipynb` and run the cell for the same dataset. The six cells are ordered Earthquake, Mixed, Others, Rainfall, Rock, and Soil.
3. The runout cell writes the training and validation predictions to `train_val_predictions.csv` and the diagnostics plot to `gan_outlier_removal_results.png`. Each runout cell uses these filenames, so move the files out of the working directory before running another dataset if you want to keep both results.

The imputation notebooks also export observed-versus-predicted comparisons. Earthquake writes a separate CSV per feature in the working directory, Soil writes them in `observed_vs_predicted_soil/`, and the other four notebooks write `observed_vs_predicted_<dataset>.csv`. Rock also writes `result_hash_rock.txt`.

## Validation

All six imputation-to-prediction workflows ran on CPU under Python 3.12 with TensorFlow 2.21.0, scikit-learn 1.9.1, pandas 3.0.1, and PyTorch 2.14.1. The check used all 284 Mixed and 579 Others records, and 240 records each from Earthquake, Rainfall, Rock, and Soil. It verified the CSV exports, the restored `L(m)` values, and the PNG diagnostics. Neural-network training in the check used two epochs. Full-length training and model-accuracy assessment remain pending.
