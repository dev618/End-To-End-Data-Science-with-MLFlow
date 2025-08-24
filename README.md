# End-to-End Data Science with MLflow

> Production-ready MLOps template: experiment tracking with **MLflow**, modular pipelines, and **AWS** CI/CD (ECR, EC2, GitHub Actions self-hosted runner).

<p align="center">
  <img alt="python" src="https://img.shields.io/badge/Python-3.8+-blue"/>
  <img alt="mlflow" src="https://img.shields.io/badge/MLflow-enabled-0194E2"/>
  <img alt="docker" src="https://img.shields.io/badge/Docker-ready-2496ED"/>
  <img alt="aws" src="https://img.shields.io/badge/AWS-ECR%20%7C%20EC2%20%7C%20IAM-FF9900"/>
  <img alt="license" src="https://img.shields.io/badge/License-MIT-green"/>
  <img alt="status" src="https://img.shields.io/badge/Build-CI%2FCD-green"/>
</p>

## Tags  

**Tags:**  
[`mlops`](#) [`mlflow`](#) [`aws`](#) [`ecr`](#) [`ec2`](#) [`docker`](#) [`cicd`](#) [`sklearn`](#) [`python`](#) [`dagshub`](#) [`github-actions`](#) [`production-ml`](#) [`endtoendml`](#) [`machinelearning`](#) [`datascience`](#) [`modeldeployment`](#) [`featureengineering`](#) [`modelregistry`](#) [`trainingpipeline`](#) [`experimenttracking`](#)  

---
## 📌 Table of Contents
- [Project Overview](#project-overview) 
- [Architecture & Workflow](#architecture--workflow) 
- [Folder Structure](#folder-structure) 
- [Tech Stack](#tech-stack) 
- [Quick Start (Local)](#quick-start-local)
- [Experiment Tracking (MLflow)](#experiment-tracking-mlflow)
- [Remote Tracking on DagsHub](#remote-tracking-on-dagshub)
- [CI/CD on AWS](#cicd-on-aws)
- [Git Connectivity](#git-connectivity)
- [Configuration Files](#configuration-files)
- [Pipelines & Components](#pipelines--components)
- [Make It Yours](#make-it-yours)
- [FAQ](#faq)
---

## Project Overview
This repository demonstrates a **clean, modular, and reproducible** machine-learning pipeline:

- Config-first design (`config.yaml`, `params.yaml`, `schema.yaml`)
- Core pipeline stages: **Data Ingestion → Validation → Transformation → Model Training → Evaluation**
- **MLflow** for experiment tracking, params, metrics, and artifacts
- **Docker** packaging and **AWS** deployment via **ECR + EC2**
- Optional **GitHub Actions** (self-hosted runner) for end-to-end CI/CD

## 📦 Pipeline Stages  

- **📁 Folder structure creation (scaffold)**  
- **📥 Data Ingestion** (download/sync raw data)  
- **🧪 Data Validation** (schema checks, null handling, range validation)  
- **⚙️ Data Transformation** (feature engineering, encoding, splitting, scaling)  
- **🤖 Model Training** (baseline & hyperparameter tuned models)  
- **📊 Model Evaluation** (metrics, drift detection hooks)  
- **📌 Model Tracker: MLflow** (parameters, metrics, artifacts, versions)  
- **🐳 Model Packaging** (Docker containerization)  
- **🚀 Model Deployment** (AWS EC2 instance)  
- **🔄 CI/CD** (GitHub Actions → ECR → EC2 via self-hosted runner)

## Tech Stack

- Language: Python 3.8+
- Core: MLflow, scikit‑learn, pandas, numpy
- Serving: Flask/FastAPI (via app.py)
- MLOps: Docker, GitHub Actions, AWS (ECR, EC2, IAM)

## 🚀 Quick Start (Local)

**Clone**
git clone https://github.com/dev618/End-to-end-Machine-Learning-Project-with-MLflow
cd End-to-end-Machine-Learning-Project-with-MLflow

**Conda env**
conda create -n mlproj python=3.8 -y
conda activate mlproj

**Install deps**
pip install -r requirements.txt

**Run the web app**
**python app.py**

Open your browser at the printed localhost:PORT.

**Run the training pipeline**
python main.py

## 📊 Experiment Tracking (MLflow)
**Local UI**

mlflow ui
Open the printed URL to explore runs (params, metrics, artifacts).
Log from code: the pipeline components log_params, log_metrics, and log_artifacts are called for each stage.

**Remote Tracking on DagsHub**

Docs: https://dagshub.com/
Project page: https://dagshub.com/dev618/End-To-End-Data-Science-with-MLFlow.mlflow

**Set environment variables (PowerShell example):**
$env:MLFLOW_TRACKING_URI="https://dagshub.com/dev618/End-To-End-Data-Science-with-MLFlow.mlflow
"
$env:MLFLOW_TRACKING_USERNAME="<YOUR_DAGSHUB_USERNAME>"
$env:MLFLOW_TRACKING_PASSWORD="<YOUR_DAGSHUB_TOKEN>"

⚠️ Do not commit secrets. Prefer GitHub Encrypted Secrets or local env vars.


## ⚡ CI/CD on AWS

**1)AWS Console & IAM**
Create an IAM user with programmatic access and attach:
AmazonEC2ContainerRegistryFullAccess
AmazonEC2FullAccess

**2)Create an ECR Repository**
Example URI: 549328952286.dkr.ecr.us-east-1.amazonaws.com/mlproj

**3)Provision an EC2 (Ubuntu) host**
Install Docker:
sudo apt-get update -y && sudo apt-get upgrade -y
curl -fsSL https://get.docker.com
 -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu
newgrp docker

**4)Configure Self-Hosted Runner**
GitHub → Settings → Actions → Runners → New self-hosted runner (Linux) and run the displayed commands on EC2.

**5)GitHub Secrets**

- AWS_ACCESS_KEY_ID=<your-key>
- AWS_SECRET_ACCESS_KEY=<your-secret>
- AWS_REGION=us-east-1
- AWS_ECR_LOGIN_URI=549328952286.dkr.ecr.us-east-1.amazonaws.com
- ECR_REPOSITORY_NAME=mlproj

**6)Sample GitHub Actions (build & push)**

name: ci-cd
on: [push]
jobs:
build-and-push:
runs-on: self-hosted
steps:
- uses: actions/checkout@v4
- name: Configure AWS creds
uses: aws-actions/configure-aws-credentials@v4
with:
aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
aws-region: ${{ secrets.AWS_REGION }}
- name: Login to ECR
id: login-ecr
uses: aws-actions/amazon-ecr-login@v2
- name: Build, tag, and push image
run: |
IMAGE_URI=${{ secrets.AWS_ECR_LOGIN_URI }}/${{ secrets.ECR_REPOSITORY_NAME }}:$(git rev-parse --short HEAD)
docker build -t $IMAGE_URI .
docker push $IMAGE_URI
- name: Deploy on EC2
run: |
IMAGE_URI=${{ secrets.AWS_ECR_LOGIN_URI }}/${{ secrets.ECR_REPOSITORY_NAME }}:$(git rev-parse --short HEAD)
docker pull $IMAGE_URI
docker stop mlproj || true && docker rm mlproj || true
docker run -d --name mlproj -p 80:80 -e MLFLOW_TRACKING_URI -e MLFLOW_TRACKING_USERNAME -e MLFLOW_TRACKING_PASSWORD $IMAGE_URI


## 🔗 Git Connectivity

git init
git remote add origin https://github.com/dev618/End-to-end-Machine-Learning-Project-with-MLflow.git
git checkout -b main
git add . && git commit -m "init: project scaffold"
git push -u origin main

## ⚙️ Configuration Files

config.yaml – Paths, uris, data locations, artifact directories
schema.yaml – Data contracts used in validation (dtypes, ranges, required cols)
params.yaml – Hyperparameters (splits, model params, thresholds)

### Workflow to update:

Update config.yaml
schema.yaml
params.yaml
Update entities
Config manager (src/config)
Components
Pipeline
main.py
app.py

## 🛠 Pipelines & Components

### Entities & Config Manager
Strongly-typed dataclasses for each stage (inputs/outputs/paths)
Centralized loader to read YAMLs and expose stage configs
### Components

data_ingestion.py – fetch/copy raw data into artifacts/
data_validation.py – validate vs schema.yaml (required columns, dtypes, NA)
data_transformation.py – split train/test, encode, scale, feature build
model_trainer.py – train baseline + tuned models, save under artifacts/model/
model_evaluation.py – compute metrics; log plots & confusion matrices

### 📈 MLflow Tracking

Logs: params, metrics, artifacts for each component
Model registry (optional): promote staging → production

### 🚀 Deployment

app.py exposes prediction API/UI
Dockerfile containerizes the app (copy model artifacts + code)
GitHub Actions builds and ships the image to ECR; EC2 pulls & runs

  ## 📂 Folder Structure  

```bash
End-to-End-Data-Science-with-MLFlow/
│
├── config/                # YAML files for config, schema, params
├── src/                   # Core source code
│   ├── components/        # Data ingestion, validation, transformation, etc.
│   ├── pipeline/          # Training, evaluation pipelines
│   ├── config/            # Configuration manager
│   └── entity/            # Data entities
│
├── artifacts/             # Generated artifacts (data, models, logs)
├── main.py                # Orchestration script
├── app.py                 # Flask/FastAPI app for inference
├── requirements.txt       # Project dependencies
├── Dockerfile             # Docker build file
├── .github/workflows/     # GitHub Actions CI/CD pipelines
└── README.md              # Project documentation
---

## Architecture & Workflow
```text
          ┌────────────────────────────────────────────────────┐
          │                      GitHub                        │
          │  PRs / Commits  ─────────▶  Actions (Runner)       │
          └────────────────────────────────────────────────────┘
                           │  build & push image
                           ▼
┌───────────────────────────────────────────────────────────────┐
│                         AWS ECR (Registry)                    │
└───────────────────────────────────────────────────────────────┘
                           │  pull image
                           ▼
┌───────────────────────────────────────────────────────────────┐
│                      AWS EC2 (Compute)                        │
│   Docker run  ▶  start API (FastAPI/Flask) + MLflow logging   │
└───────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────┐
│                     MLflow Tracking Server                    │
│      local mlflow ui  or  remote (DagsHub / self-hosted)      │
└───────────────────────────────────────────────────────────────┘

