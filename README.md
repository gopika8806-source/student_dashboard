# Early At-Risk Student Prediction — Lightweight LSTM + Dashboard

Working implementation of the framework described in *"A Resource-Efficient
Deep Learning Model for Early At-Risk Student Prediction."*

## What's here

```
edm-project/
├── data/
│   └── generate_data.py      # builds the hybrid dataset (see note below)
├── model/
│   ├── train_model.py        # trains the lightweight LSTM, CPU-only
│   └── artifacts/            # saved model, scaler, metrics.json (after training)
├── app/
│   └── app.py                # Streamlit faculty-facing dashboard
└── requirements.txt
```

## ⚠️ About the dataset

The paper's base dataset (Realinho et al., 2021) lives on UCI/Kaggle, which
this build environment cannot reach. `generate_data.py` therefore
**synthesizes** a dataset matching the same structure and class balance
(4,424 students, ~32% Dropout / 18% Enrolled / 50% Graduate) with simulated
weekly attendance/LMS engagement correlated to outcome, exactly as the paper
describes for the augmentation step.

**Before you rely on this for anything real:** download the actual dataset
from https://doi.org/10.24432/C5MC89, save it as
`data/raw_academic_success.csv`, set `USE_REAL_BASE = True` in
`generate_data.py`, and re-run the pipeline. The synthetic engagement
patterns here are also cleanly separable by design, so the trained model
hits ~100% accuracy — on real institutional data, expect it to be
meaningfully lower and noisier.

## Run it locally

```bash
pip install -r requirements.txt

python data/generate_data.py     # 1. build the dataset
python model/train_model.py      # 2. train the LSTM, saves model + metrics
streamlit run app/app.py         # 3. launch the dashboard
```

The dashboard opens at `http://localhost:8501` with three tabs: single-student
manual entry, batch CSV scoring, and model/deployability metrics.

## Deploy it for real (Streamlit Community Cloud)

I can build and test everything above, but I can't create accounts or push
to services on your behalf. To get a public URL:

1. Create a GitHub repo and push this whole `edm-project/` folder to it.
2. Make sure `model/artifacts/` (the trained `.keras` model + `scaler.joblib`
   + `metrics.json`) is committed too — Streamlit Cloud only sees what's in
   the repo, it won't run your training script for you.
3. Go to **share.streamlit.io**, sign in with GitHub, click **New app**,
   point it at your repo and set the main file path to `app/app.py`.
4. Streamlit Cloud installs `requirements.txt` automatically and deploys.
   Free tier is enough for this app's size.

That's the same deployment path described in Section 3.7 of the paper.

## Measured deployability (synthetic data, this machine)

| Metric | Value |
|---|---|
| Model size | ~0.05 MB |
| CPU inference time | ~58 ms / prediction |
| Total training time (CPU) | ~8 sec |
| Trainable parameters | 1,489 |

Re-run `model/train_model.py` to regenerate `model/artifacts/metrics.json`
with numbers you can drop directly into the paper's Section 4 `[X]` placeholders.
