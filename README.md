# ML Experiments Template — Reproducible ML Engineering

**Cookiecutter-style template for production-grade ML experiments**

**Author:** Mohammed Lazhari Krobba  
**Purpose:** Standardize ML experiment workflow with reproducibility, tracking, and CI/CD

---

## 🎯 Highlights

- **Zero-config reproducibility:** Fixed seeds, pinned deps, Docker, environment capture
- **Experiment tracking:** MLflow + DVC integration, automatic artifact versioning
- **CI/CD ready:** GitHub Actions for testing, evaluation, model validation
- **Multi-experiment management:** Config-driven, parallel runs, hyperparameter sweeps
- **Production-ready:** Model serving (SavedModel), monitoring hooks, drift detection hooks

---

## 🚀 Quick Start

```bash
# Use as template (GitHub)
gh repo create my-experiment --template your-username/ml-experiments-template

# Or clone and customize
git clone https://github.com/your-username/ml-experiments-template my-new-experiment
cd my-new-experiment

# Configure
cp config.yaml.example config.yaml
# Edit config.yaml with your experiment params

# Run
docker compose up --build
# OR
pip install -r requirements-lock.txt
python scripts/train.py --config config.yaml
```

---

## 🏗 Template Structure

```
ml-experiments-template/
├── {{project_name}}/
│   ├── config.yaml                 # Experiment configuration
│   ├── data/
│   │   ├── raw/                    # Immutable raw data
│   │   ├── processed/              # Preprocessed splits
│   │   └── README.md               # Data lineage
│   ├── models/
│   │   ├── model.py                # Model definition
│   │   ├── train.py                # Training loop
│   │   ├── evaluate.py             # Evaluation script
│   │   ├── saved_model/            # TensorFlow SavedModel / ONNX
│   │   ├── metrics.json            # Final metrics
│   │   ├── training_history.csv
│   │   └── config.yaml             # Model-specific config
│   ├── notebooks/
│   │   ├── 01_exploration.ipynb
│   │   ├── 02_preprocessing.ipynb
│   │   ├── 03_training.ipynb
│   │   └── 04_evaluation.ipynb
│   ├── experiments/
│   │   ├── configs/                # YAML configs for sweeps
│   │   ├── logs/                   # MLflow/TensorBoard logs
│   │   └── results/                # CSV predictions, metrics
│   ├── docker/
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   └── requirements.txt
│   ├── scripts/
│   │   ├── preprocess.py
│   │   ├── train.py
│   │   ├── evaluate.py
│   │   └── sweep.py                # Hyperparameter sweep
│   ├── tests/
│   │   ├── test_data.py
│   │   ├── test_model.py
│   │   └── test_reproducibility.py
│   ├── .github/workflows/
│   │   ├── test.yml
│   │   ├── evaluate.yml
│   │   └── sweep.yml
│   ├── requirements.txt
│   ├── requirements-lock.txt
│   ├── pyproject.toml
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── LICENSE
│   └── CITATION.cff
├── shared/                          # Shared utilities across projects
│   ├── preprocessing.py
│   ├── evaluation.py
│   ├── tracking.py                 # MLflow/DVC integration
│   ├── reproducibility.py          # Seed control, env capture
│   └── serving.py                  # Model serving utilities
└── .github/workflows/
    └── template-validation.yml
```

---

## 🔧 Configuration (config.yaml)

```yaml
# Experiment metadata
experiment:
  name: "my-experiment"
  version: "1.0.0"
  description: "Brief description"
  tags: ["domain", "architecture", "dataset"]

# Reproducibility
reproducibility:
  seed: 42
  python_hash_seed: 42
  numpy_seed: 42
  torch_seed: 42
  tf_seed: 42
  deterministic: true
  benchmark: false

# Data
data:
  source: "local"  # or "s3", "gcs", "zenodo"
  path: "data/raw/"
  splits:
    train: 0.7
    val: 0.15
    test: 0.15
  split_strategy: "chronological"  # or "random", "stratified"
  preprocessing:
    - name: "normalize"
      params: {method: "minmax", feature_range: [0, 1]}
    - name: "sequence_window"
      params: {window_size: 64, stride: 1}

# Model
model:
  name: "tcn"
  architecture:
    type: "TCN"
    params:
      num_channels: [64, 64, 64, 64]
      kernel_size: 3
      dropout: 0.2
      dilation_base: 2
  input_shape: [64, 1]
  output_shape: [1]

# Training
training:
  optimizer: "adam"
  learning_rate: 0.001
  lr_schedule: "cosine_annealing"
  batch_size: 32
  epochs: 100
  early_stopping:
    patience: 15
    metric: "val_loss"
    mode: "min"
  loss: "mse"
  metrics: ["mae", "mse"]
  mixed_precision: true
  gradient_clip: 1.0

# Evaluation
evaluation:
  metrics: ["mse", "mae", "rmse", "mape"]
  test_splits: ["test", "ood"]  # out-of-distribution
  statistical_tests:
    - "diebold_mariano"
    - "wilcoxon"
  prediction_horizons: [1, 5, 10, 24]

# Tracking
tracking:
  mlflow:
    enabled: true
    uri: "http://localhost:5000"
    experiment_name: "{{experiment.name}}"
  dvc:
    enabled: true
    remote: "myremote"
  artifacts:
    save_model: true
    save_predictions: true
    save_logs: true

# Compute
compute:
  device: "auto"  # "cpu", "cuda", "mps", "auto"
  mixed_precision: true
  num_workers: 4
  pin_memory: true

# Hyperparameter sweep (optional)
sweep:
  enabled: false
  method: "bayes"  # "grid", "random", "bayes"
  metric: "val_loss"
  mode: "min"
  max_runs: 50
  parameters:
    learning_rate:
      distribution: "log_uniform"
      min: 1e-5
      max: 1e-2
    batch_size:
      values: [16, 32, 64]
    dropout:
      distribution: "uniform"
      min: 0.1
      max: 0.5
```

---

## 🐳 Docker Support

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# System deps
RUN apt-get update && apt-get install -y \
    git \
    && rm -rf /var/lib/apt/lists/*

# Python deps
COPY requirements-lock.txt .
RUN pip install --no-cache-dir -r requirements-lock.txt

# Code
COPY . .

# Non-root user
RUN useradd -m -u 1000 appuser && chown -R appuser:appuser /app
USER appuser

# Entrypoint
ENTRYPOINT ["python", "scripts/train.py"]
CMD ["--config", "config.yaml"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  train:
    build: .
    volumes:
      - ./data:/app/data
      - ./experiments:/app/experiments
      - ./models:/app/models
    environment:
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]

  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.8.0
    ports:
      - "5000:5000"
    volumes:
      - mlflow_data:/mlflow
    command: mlflow server --host 0.0.0.0 --port 5000

volumes:
  mlflow_data:
```

---

## 🔄 CI/CD Workflows

```yaml
# .github/workflows/test.yml
name: Test & Validate
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - name: Install deps
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements-lock.txt
      - name: Run tests
        run: |
          pytest tests/ -v --cov=shared --cov-report=xml
      - name: Check reproducibility
        run: |
          python scripts/train.py --config config.yaml --dry-run
          python scripts/train.py --config config.yaml --dry-run
          # Compare outputs for determinism
      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

```yaml
# .github/workflows/evaluate.yml
name: Evaluate Model
on:
  workflow_dispatch:
    inputs:
      config:
        description: 'Config file'
        required: true
        default: 'config.yaml'
jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.11"
      - name: Install deps
        run: pip install -r requirements-lock.txt
      - name: Run evaluation
        run: |
          python scripts/evaluate.py --config ${{ github.event.inputs.config }}
      - name: Upload metrics
        uses: actions/upload-artifact@v4
        with:
          name: evaluation-metrics
          path: experiments/results/
```

---

## 🧪 Reproducibility Guarantees

| Check | Implementation |
|-------|----------------|
| **Fixed seeds** | All seeds in config.yaml, set in `reproducibility.py` |
| **Pinned dependencies** | `requirements-lock.txt` generated by `pip freeze` |
| **Environment capture** | `reproducibility.py` saves `conda list`, `pip list`, GPU driver, CUDA version |
| **Data versioning** | DVC tracks `data/processed/`, `models/saved_model/` |
| **Code versioning** | Git commits tagged with experiment version |
| **Containerization** | Dockerfile + docker-compose for identical environments |
| **Deterministic ops** | `TF_DETERMINISTIC_OPS=1`, `CUBLAS_WORKSPACE_CONFIG=:4096:8` |

```python
# shared/reproducibility.py
import os
import random
import numpy as np
import torch
import tensorflow as tf

def set_deterministic(seed: int = 42):
    """Set all seeds for reproducibility."""
    os.environ['PYTHONHASHSEED'] = str(seed)
    os.environ['TF_DETERMINISTIC_OPS'] = '1'
    os.environ['TF_CUDNN_DETERMINISTIC'] = '1'
    os.environ['CUBLAS_WORKSPACE_CONFIG'] = ':4096:8'
    
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    torch.cuda.manual_seed_all(seed)
    tf.random.set_seed(seed)
    
    # Torch deterministic algorithms
    torch.use_deterministic_algorithms(True)
    torch.backends.cudnn.deterministic = True
    torch.backends.cudnn.benchmark = False
    
    # TF deterministic
    tf.config.experimental.enable_op_determinism()
```

---

## 📊 Experiment Tracking (MLflow + DVC)

```python
# shared/tracking.py
import mlflow
import dvc.api
from pathlib import Path

def init_tracking(config):
    """Initialize MLflow and DVC tracking."""
    mlflow.set_tracking_uri(config.tracking.mlflow.uri)
    mlflow.set_experiment(config.experiment.name)
    
    # Log config as params
    mlflow.log_params(flatten_dict(config.to_dict()))
    
    # DVC for data/model versioning
    if config.tracking.dvc.enabled:
        dvc_repo = dvc.api.Repo()
        return dvc_repo
    
    return mlflow.active_run()

def log_metrics(metrics: dict, step: int = None):
    mlflow.log_metrics(metrics, step=step)

def log_artifacts(local_dir: str, artifact_path: str = None):
    mlflow.log_artifacts(local_dir, artifact_path)

def save_model(model, path: str, signature=None):
    """Save model in multiple formats."""
    # TensorFlow SavedModel
    model.save(path + "/saved_model")
    
    # ONNX (optional)
    # import tf2onnx
    # onnx_model, _ = tf2onnx.convert.from_keras(model)
    # with open(path + "/model.onnx", "wb") as f:
    #     f.write(onnx_model.SerializeToString())
    
    # Log to MLflow
    mlflow.log_artifacts(path, "model")
```

---

## 📄 License

MIT License — see LICENSE file.