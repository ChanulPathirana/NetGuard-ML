NetGuard ML

NetGuard ML is a machine-learning network intrusion detection system that analyzes authorized laptop network traffic, detects suspicious network flows, estimates a possible attack category, and displays warnings through a web dashboard.

The project combines network traffic analysis, machine learning, FastAPI, a monitoring dashboard, Docker, deployment, and CI/CD in one repository.

Current status: Phase 1 — project configuration, dataset inspection, and training/live feature compatibility.

What the project does

NetGuard processes network traffic using this workflow:

tcpdump captures packets from a network interface the user is authorized to monitor.

A pinned CICFlowMeter-compatible extractor groups the packets into network flows.

The extractor calculates flow statistics such as duration, packet counts, byte counts, packet rates, and TCP flag counts.

Model 1 predicts whether a completed flow is BENIGN or SUSPICIOUS.

If the flow is suspicious, Model 2 estimates its possible attack category.

FastAPI validates the input, runs inference, and stores recent results.

The dashboard displays network status, statistics, charts, and alerts.

NetGuard analyzes traffic metadata and behaviour. It does not need to inspect passwords, private messages, or encrypted HTTPS content.

Architecture

NetGuard uses a layered client-server architecture, a local edge collector, a FastAPI modular monolith, and a conditional two-stage ML pipeline.

flowchart TD
    A["Authorized laptop traffic"] --> B["tcpdump packet capture"]
    B --> C["Flow feature extraction"]
    C --> D["FastAPI validation"]
    D --> E["Model 1: attack detector"]
    E -->|"Benign"| G["Store result"]
    E -->|"Suspicious"| F["Model 2: attack classifier"]
    F --> G
    G --> H["Dashboard and alerts"]

The deployed server hosts the API, models, dashboard, and alert storage. Packet capture remains on the laptop because a remote server cannot directly observe the laptop's network interface.

Two-model pipeline

Model 1 — Binary attack detector

Every validated flow passes through Model 1.

BENIGN or SUSPICIOUS

The model is evaluated using suspicious-class precision, recall, F1-score, PR-AUC, false-positive rate, confusion matrix, and inference time. Accuracy alone is not sufficient for an imbalanced intrusion-detection dataset.

Model 2 — Attack-category classifier

Model 2 runs only when Model 1 reports suspicious traffic.

Possible normalized categories include:

Bot

BruteForce

DDoS

DoS

PortScan

WebAttack

Infiltration

Other or Unknown

The final supported categories will depend on the cleaned CICIDS2017 class counts and reliable test coverage.

Example prediction:

{
  "prediction": "SUSPICIOUS",
  "binary_confidence": 0.94,
  "possible_attack": "PortScan",
  "attack_confidence": 0.87
}

The returned values are model estimates, not proof that an attack occurred.

Planned features

Binary benign-versus-suspicious flow detection

Conditional attack-category classification

Single-flow and batch prediction APIs

Schema and feature-contract validation

Authorized local packet capture and flow extraction

SAFE, WARNING, and HIGH RISK dashboard states

Traffic totals and attack-category charts

Recent alert history

SQLite result storage

Unit and API tests

Docker packaging

Cloud deployment

GitHub Actions CI/CD

Technology stack

Area

Technology

Language

Python 3.11+

Data processing

pandas, NumPy

Machine learning

scikit-learn

Model persistence

joblib

Backend

FastAPI, Uvicorn, Pydantic

Dashboard

Jinja2, HTML, CSS, JavaScript, Chart.js

Storage

SQLite

Traffic capture

tcpdump

Flow extraction

Pinned CICFlowMeter-compatible implementation

Testing

pytest, FastAPI TestClient

Code quality

Ruff

Packaging

Docker, Docker Compose

Automation

GitHub Actions

Repository structure

netguard-ml/
├── .github/
│   └── workflows/             # CI/CD workflows
├── collector/                 # Local capture, extraction, and sender
├── dashboard/                 # Dashboard templates and static files
├── data/
│   ├── raw/                   # Original dataset files; not committed
│   ├── processed/             # Cleaned data; not committed
│   └── samples/               # Small safe test fixtures
├── docs/                      # Architecture and feature contract
├── models/                    # Saved model artifacts; not committed by default
├── notebooks/                 # EDA and model experiments
├── src/
│   └── netguard/
│       ├── api/               # FastAPI application and routes
│       ├── data/              # Cleaning and feature contract
│       ├── inference/         # Model loading and prediction
│       ├── monitoring/        # Collector integration
│       └── training/          # Model training and evaluation
├── tests/                     # Automated tests
├── .env.example
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
├── requirements.txt
└── README.md

The structure may grow as implementation progresses, but the complete MVP remains in one repository.

Phase 1 setup

1. Clone the repository

git clone <YOUR_REPOSITORY_URL>
cd NetGuard-ML

2. Create and activate a virtual environment

python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip

If the repository is stored on an NTFS, exFAT, or other mounted data partition and environment creation is unusually slow, keep the virtual environment on the Linux filesystem:

mkdir -p /home/chanul/.venvs
python3 -m venv /home/chanul/.venvs/netguard-ml
source /home/chanul/.venvs/netguard-ml/bin/activate
python -m pip install --upgrade pip

Verify the active interpreter:

which python
python --version
pip --version

3. Install the initial data and ML dependencies

pip install pandas numpy scikit-learn jupyter matplotlib seaborn joblib
pip freeze > requirements.txt

FastAPI, testing, dashboard, and deployment dependencies will be added when those phases begin.

4. Add the dataset

Download the CICIDS2017 labelled flow CSV files and place them in:

data/raw/

Do not commit the complete dataset to Git.

5. Verify feature compatibility

Before training the final models:

Pin one exact CICFlowMeter-compatible extractor and version.

Convert a small authorized PCAP capture into a flow CSV.

Normalize the extractor and CICIDS2017 column names.

Compare their feature definitions, types, order, and units.

Keep only trustworthy features available in both sources.

Record the approved ordered list in docs/feature-contract.md.

This step prevents training a model that works on the dataset but cannot process real traffic.

Dataset preparation rules

The training pipeline will:

Normalize column names and attack labels.

Convert selected inputs to numeric values.

Replace positive and negative infinity with missing values.

Impute missing values using parameters learned only from training data.

Remove duplicate, constant, unusable, and leakage-prone columns.

Preserve the approved feature order.

Prefer a capture-day or file-based holdout where possible.

Keep the final test partition untouched during model selection.

The following fields are normally excluded from model input:

Flow ID

Source IP address

Destination IP address

Timestamp

Binary target label

Attack-category target label

Planned API

GET  /health
POST /api/v1/predict
POST /api/v1/predict/batch
GET  /api/v1/status
GET  /api/v1/alerts
GET  /api/v1/statistics

The prediction endpoints will require the expected schema version and all features from the saved feature contract.

Running the completed application

These commands will become available after the FastAPI and dashboard phase is implemented.

Development server:

uvicorn netguard.api.main:app --reload

Tests and quality checks:

pytest
ruff check .

Docker:

docker compose up --build

Project phases

Phase

Work

Target

1

Repository, environment, dataset inspection, extractor compatibility, and feature contract

Day 1 morning

2

Data cleaning, binary model, attack classifier, evaluation, and saved artifacts

Day 1 afternoon–Day 2

3

FastAPI backend, SQLite storage, and simple dashboard

Day 3

4

Authorized laptop traffic capture and end-to-end integration

Day 4 morning

5

Tests, Docker, deployment, documentation, and CI/CD

Day 4 afternoon

The target is a portfolio-ready MVP in three to four full working days. Complex UI work, automatic firewall blocking, extensive hyperparameter tuning, and 24/7 monitoring remain outside the first release.

CI/CD plan

Pull requests will run:

Install dependencies
→ lint
→ test
→ verify model metadata
→ build Docker image

Pushes to main will run all CI checks, deploy the application, and verify the deployed /health endpoint. Model retraining will remain a separate controlled workflow rather than running during every deployment.

Security and privacy

Capture only traffic you own or are explicitly authorized to monitor.

Do not commit PCAP captures because they can contain sensitive metadata or unencrypted content.

Keep secrets and API keys in environment variables.

Protect public prediction endpoints before deployment.

Run the deployed web application without packet-capture privileges.

Keep the privileged local collector separate from the public API service.

Limitations

The MVP will not:

Automatically block network traffic or modify firewall rules.

Remove malware or replace an antivirus product.

Guarantee that every alert is a real attack.

Detect every new or unseen attack technique.

Inspect encrypted HTTPS application content.

Monitor every device on a Wi-Fi network without gateway or mirrored-port access.

Treat predictions on unlabeled live traffic as formal accuracy measurements.

Formal model performance will be reported using untouched labelled test data. Real laptop traffic demonstrates that the end-to-end pipeline works, but it does not provide ground-truth attack labels.

Documentation

Detailed project documents:

docs/what-netguard-does.md

docs/full-project-implementation.md

docs/project-phases.md

docs/two-models-details.md

docs/feature-contract.md — created during Phase 1

Project goal

The final NetGuard ML MVP will demonstrate an end-to-end engineering workflow:

CICIDS2017 dataset
→ feature-compatible preprocessing
→ binary and multiclass model training
→ saved inference pipelines
→ authorized laptop traffic capture
→ flow extraction and validation
→ FastAPI predictions
→ dashboard status and alerts
→ Docker deployment and CI/CD

