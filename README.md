# CI/CD Node.js App — GitOps with Kubernetes, GitHub Actions & ArgoCD

A Node.js application built with Express, used as a reference implementation for a modern CI/CD pipeline based on **Docker**, **GitHub Actions**, and **GitOps (ArgoCD)** deployed to a local **Kubernetes (Kind)** cluster.

## Overview

On every push to `main`, GitHub Actions builds a Docker image, tags it with the commit SHA, and pushes it to DockerHub. The workflow then updates the Kubernetes deployment manifest with the new image tag and commits the change back to the repository. ArgoCD continuously watches the `k8s/` directory and automatically syncs the updated manifest to the cluster — no manual `kubectl apply` required.

## Features

- Dark-themed, responsive UI built with HTML/CSS/JS
- Lightweight Express backend serving static assets and video metadata
- Fully containerized with a `Dockerfile` and `docker-compose.yml`
- Automated CI (build & push) and CD (manifest update) via GitHub Actions
- GitOps-driven deployment to Kubernetes using ArgoCD
- Immutable, traceable image tags based on commit SHA

## Architecture

```
Developer
    │  git push (main)
    ▼
GitHub Repository
    │
    ▼
GitHub Actions
    ├── Build Docker image (tag: commit SHA)
    ├── Push image to DockerHub
    └── Update k8s/deployment.yaml with new tag
            │
            ▼
    Commit manifest change [skip ci]
            │
            ▼
         ArgoCD
    (watches k8s/ directory)
            │
            ▼
   Kubernetes Cluster (Kind)
            │
            ▼
     Node.js Application
```

## Technology Stack

| Layer | Technology |
|---|---|
| Runtime | Node.js |
| Backend framework | Express.js |
| Frontend | HTML, CSS, JavaScript |
| Containerization | Docker |
| Local orchestration | Docker Compose |
| Container orchestration | Kubernetes (Kind) |
| CI/CD | GitHub Actions |
| GitOps delivery | ArgoCD |
| Image registry | DockerHub |
| Version control | Git / GitHub |

## Project Structure

```
CICD-NodeJs-App/
├── .github/
│   └── workflows/
│       └── deploy.yml          # CI (build/push) + CD (manifest update)
├── k8s/
│   ├── deployment.yaml         # Kubernetes Deployment (image tag updated by CI)
│   └── service.yaml            # ClusterIP Service
├── public/
│   ├── index.html
│   ├── style.css
│   └── script.js
├── server.js                   # Express application entry point
├── package.json
├── package-lock.json
├── Dockerfile
├── docker-compose.yml
├── kind-config.yaml             # Kind cluster configuration
└── README.md
```

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) — required for Docker Compose and Kind
- [kubectl](https://kubernetes.io/docs/tasks/tools/) — for interacting with the cluster
- [Kind](https://kind.sigs.k8s.io/) — for running a local Kubernetes cluster
- [ArgoCD CLI](https://argo-cd.readthedocs.io/en/stable/cli_installation/) — optional, for managing ArgoCD from the terminal

## Running Locally

### Option 1 — Node.js

```bash
npm install
npm start
```

Available at `http://localhost:3000`.

### Option 2 — Docker Compose

```bash
docker compose up --build
```

Available at `http://localhost:3000`.

### Option 3 — Kubernetes (Kind + ArgoCD)

**1. Create the Kind cluster**

```bash
kind create cluster --config kind-config.yaml --name cicd-cluster
```

**2. Install ArgoCD**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**3. Connect ArgoCD to this repository**

Create an ArgoCD Application pointing to the `k8s/` directory of this repository. Once applied, ArgoCD will deploy `deployment.yaml` and `service.yaml`, and keep the cluster in sync with every future commit to that path.

**4. Access the application**

The service is exposed as `ClusterIP`, so access it via port-forwarding:

```bash
kubectl port-forward svc/cicd-app-service 3000:3000
```

Available at `http://localhost:3000`.

## CI/CD Pipeline (GitOps)

The pipeline is defined in `.github/workflows/deploy.yml` and triggers on every push to `main`.

**1. CI — Build & Push**
- Checks out the repository
- Authenticates to DockerHub using repository secrets
- Builds the Docker image, tagged with the commit SHA (`${{ github.sha }}`)
- Pushes the image to DockerHub

**2. CD — Update Manifest**
- Updates `k8s/deployment.yaml` in place with the new image tag using `sed`
- Commits the change back to `main` with a `[skip ci]` marker to prevent triggering a new workflow run

**3. GitOps — ArgoCD Sync**
- ArgoCD detects the new commit under `k8s/`
- Pulls the change and reconciles the cluster to match, rolling out the new image automatically

This design keeps deployment state declarative and version-controlled: the desired state of the cluster always matches what's committed to `k8s/`, and ArgoCD is solely responsible for applying it.

## Required GitHub Secrets

Configure the following under **Repository → Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | DockerHub account username |
| `DOCKERHUB_TOKEN` | DockerHub personal access token |

> EC2 and SSH-based secrets are not required in this setup, as deployment has moved from direct server access to a Kubernetes/ArgoCD GitOps model.
