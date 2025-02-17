# stord-challenge
# SRE Technical Challenge Helm Chart

This Helm chart deploys a Phoenix application for the SRE Technical Challenge. It manages database migrations and configures a web server with health probes and environment variables. The application image is available at `ghcr.io/stordco/sre-technical-challenge`.

## Overview

- **Database Migrations:** A Kubernetes Job (using Helm hooks) runs the `migrate` command to apply database migrations before the web server starts.
- **Web Server:** A Deployment starts the Phoenix web server using the `server` command.
- **Environment Variables:** All environment variables defined in `values.yaml` under `env` are passed through to both the migration job and the application container.
- **Health Probes:** Liveness and readiness probes are set up to check the `/_health` endpoint.
- **Service Exposure:** The application is exposed via a ClusterIP service on port 80 that routes to the container’s port 4000.

## Prerequisites

- **Kubernetes Cluster:** An accessible cluster with a valid `kubeconfig`.
- **Helm v3:** Ensure you have Helm v3 installed.
- **PostgreSQL Database:** This chart expects an existing PostgreSQL database. The Bitnami PostgreSQL chart is used for this challenge.

## Installation

### Step 1: Deploy the PostgreSQL Database

Use the Bitnami PostgreSQL chart to install the database:

```bash
NAMESPACE=stord

helm install sre-db oci://registry-1.docker.io/bitnamicharts/postgresql \
  --create-namespace \
  --namespace $NAMESPACE \
  --set auth.database=sre-technical-challenge \
  --set auth.postgresPassword=password
```

### Step 2: Deploy the helm chart for sre-app (with debug mode)

```bash
helm install sre-app . --debug --namespace $NAMESPACE --values values.yaml
```

### Step 3: Validate

Run port forwarder for the postgresql instance

```bash
kubectl port-forward -n $NAMESPACE svc/sre-db-postgresql 55432:5432
psql postgresql://postgres:password@127.0.0.1:55432/sre-technical-challenge
```

Run port forwarder for the app
```bash
kubectl port-forward -n stord svc/sre-app-sre-technical-challenge 8080:80
```

From a browser, access:
<http://localhost:8080/todos>

confirm that health checks work correctly, from term session:
```bash
curl -v http://localhost:8080/_health
```