# NetGuard ML

NetGuard ML is a planned machine-learning network intrusion detection system. It
will turn locally observed network-flow metadata into predictions that help an
operator identify suspicious traffic. The project is currently in **Phase 1:
repository and Python environment setup**; it does not yet collect traffic,
train models, serve predictions, or provide a dashboard.

## Planned system

The detection pipeline will use two stages:

1. A binary model will classify each network flow as benign or potentially
   malicious.
2. A multiclass model will categorize flows flagged as malicious into an attack
   type, subject to the classes supported by the selected training data.

A planned local collector will derive flow-level features from network traffic
and submit them to a separately deployed application. That application is
planned to expose a FastAPI backend for inference and a simple dashboard for
reviewing results. The models, FastAPI backend, dashboard, collector, Docker
configuration, deployment, and CI/CD automation are **planned and are not
currently implemented**.

Only flow metadata needed by the eventual feature pipeline should be retained.
Raw payload inspection and automated traffic blocking are outside the current
scope.

## Repository structure

```text
app/                 Planned deployed application
  api/               Planned FastAPI routes
  ml/                Planned inference and model-loading code
  services/          Planned application services
collector/           Planned local network-flow collector
training/            Planned data preparation and model training
dashboard/           Planned operator dashboard
data/
  raw/               Local raw datasets (ignored by Git)
  processed/         Local processed datasets (ignored by Git)
  samples/           Small, safe sample data when appropriate
models/              Local trained model artifacts (ignored by Git)
notebooks/           Planned exploration notebooks
tests/               Planned automated tests
docs/                Planned design and operational documentation
scripts/             Planned development and maintenance scripts
.github/workflows/    Planned CI/CD workflows
```

Reserved directories contain `.gitkeep` or Python package markers so the
initial layout can be versioned before implementation begins.

## Phase 1 environment setup

Python 3.13 is used by the current local environment. From the repository root,
create and activate a virtual environment, then install the pinned Phase 1
dependencies:

```bash
python3.13 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m pip check
```

On Windows PowerShell, activate the environment with:

```powershell
.venv\Scripts\Activate.ps1
```

Phase 1 provides the libraries needed for data analysis, visualization,
notebooks, and initial scikit-learn experimentation. Application and deployment
dependencies will be added only when those phases begin.

## Planned 3-4 day development schedule

- **Day 1:** finalize the repository and environment, inspect candidate network
  intrusion datasets, define flow features, and establish preprocessing.
- **Day 2:** train and evaluate the binary detector and attack-category model;
  save reproducible preprocessing and model artifacts locally.
- **Day 3:** implement the FastAPI inference service, simple dashboard, and
  local collector integration, with focused tests.
- **Day 4 (if available):** add Docker packaging, deployment configuration,
  CI/CD checks, documentation, and end-to-end validation.

This is an intentionally aggressive prototype schedule. Work may move between
days as data quality, model evaluation, and integration findings require.

## Security and limitations

- Never commit packet captures, raw or processed datasets, trained models,
  virtual environments, credentials, or local `.env` files.
- Network capture can expose sensitive metadata and may require elevated
  privileges. Use the future collector only on networks and interfaces where
  capture is explicitly authorized.
- ML predictions can contain false positives and false negatives. NetGuard ML
  is planned as an investigative aid, not a replacement for layered security
  controls or expert review.
- Model performance will depend on training-data relevance, label quality,
  traffic visibility, concept drift, and consistency between training and live
  flow features.
- The current repository contains setup scaffolding only. There are no trained
  models, live inference endpoints, traffic collection, automated response, or
  production-security guarantees yet.


