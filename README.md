# Voting App DevOps Project

## Overview

This project is based on the Docker Example Voting App, a multi-service application where users can vote between two options and view the current voting results.

I used the existing application source code and focused on building the DevOps infrastructure around it. I containerized the application services, orchestrated the stack with Docker Compose, deployed it to a local Kubernetes cluster using Minikube, and later packaged the Kubernetes manifests into a Helm chart.

The project also includes a GitHub Actions CI pipeline that validates the Helm chart, builds the application container images, and publishes them to GitHub Container Registry (GHCR).

## Architecture

```text
                ┌──────────────┐
                │     User     │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │     Vote     │
                │   Frontend   │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │    Redis     │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │    Worker    │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │  PostgreSQL  │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │    Result    │
                │   Frontend   │
                └──────────────┘
```

The Vote service receives a user's vote and stores it in Redis. The Worker service consumes the vote from Redis and writes it to PostgreSQL. The Result service reads the stored voting data from PostgreSQL and displays the results to users.

## Tech Stack

- **Docker** — Application containerization
- **Docker Compose** — Local multi-container orchestration
- **Kubernetes** — Container orchestration
- **Minikube** — Local Kubernetes cluster
- **Helm** — Kubernetes packaging and configuration
- **GitHub Actions** — Continuous Integration
- **GitHub Container Registry (GHCR)** — Container image registry
- **Redis** — Temporary vote storage
- **PostgreSQL** — Persistent voting data

## Running with Docker Compose

Create the local environment file:

```bash
cp .env.example .env
```

Build and start the application:

```bash
docker compose up -d --build
```

Access the services:

- Vote: `http://localhost:8080`
- Result: `http://localhost:4000`

Stop the stack:

```bash
docker compose down
```

## Kubernetes Deployment

Raw Kubernetes manifests are available in the `k8s/` directory and were used to deploy the application to a local Minikube cluster.

Start Minikube and enable the NGINX Ingress Controller:

```bash
minikube start
minikube addons enable ingress
```

The raw Kubernetes deployment uses a local `k8s/secret.yaml` file for the database password. This file is excluded from Git and must be created locally before deployment.

Apply the manifests:

```bash
kubectl apply -f k8s/
```

The Kubernetes deployment includes Deployments, Services, ConfigMap, Secret, PersistentVolumeClaim, and Ingress resources.

PostgreSQL uses persistent storage through a PersistentVolumeClaim, while non-sensitive application configuration is provided through a ConfigMap.

## Helm Deployment

After validating the raw Kubernetes manifests, the resources were packaged into a Helm chart located in:

```text
helm/voting-app/
```

The chart uses `values.yaml` to configure image repositories and tags, replica counts, service ports, storage, Ingress hosts, and application configuration.

Create the namespace and database Secret:

```bash
kubectl create namespace voting-app

kubectl create secret generic voting-app-secret \
  -n voting-app \
  --from-literal=DB_PASSWORD="your-password"
```

Validate the chart:

```bash
helm lint helm/voting-app
```

Install the application:

```bash
helm install voting-app helm/voting-app -n voting-app
```

Apply future chart changes with:

```bash
helm upgrade voting-app helm/voting-app -n voting-app
```

The database password is not stored in `values.yaml` or managed by the Helm release. The chart references the existing Kubernetes Secret instead.

## CI Pipeline

The GitHub Actions workflow is defined in:

```text
.github/workflows/ci.yml
```

The pipeline is triggered on every push to the `main` branch.

```text
Push to main
      │
      ▼
Checkout repository
      │
      ▼
Helm lint
      │
      ▼
Login to GHCR
      │
      ▼
Build Docker images
      │
      ▼
Push :latest images to GHCR
```

The pipeline builds and publishes the Vote, Result, and Worker images using the `latest` tag.

The CI pipeline does not automatically deploy the application. Deployment to the local Minikube cluster is managed separately with Helm.

## Project Structure

```text
voting-app-devops/
├── .github/
│   └── workflows/
│       └── ci.yml
├── helm/
│   └── voting-app/
├── k8s/
├── vote/
│   └── Dockerfile
├── result/
│   └── Dockerfile
├── worker/
│   └── Dockerfile
├── .env.example
├── compose.yml
├── LICENSE
└── README.md
```

Sensitive local files such as `.env` and `k8s/secret.yaml` are excluded from version control.

## Credits

The application source code is based on Docker's official `dockersamples/example-voting-app` project.

The existing application was used as the foundation for this project, while the containerization, Docker Compose configuration, Kubernetes manifests, Helm chart, GHCR integration, and GitHub Actions CI workflow were implemented as part of this DevOps project.

The original application is licensed under the Apache License 2.0.