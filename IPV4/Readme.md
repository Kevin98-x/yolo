# Kubernetes Deployment Project

This project demonstrates the deployment of a distributed application on Google Kubernetes Engine (GKE) using Kubernetes manifests.

## Prerequisites

- Docker
- Google Cloud SDK (`gcloud`)
- `kubectl`

## Steps to Deploy

1. **Set Up GKE Cluster**:
   ```bash
   gcloud container clusters create my-cluster \
     --num-nodes=2 \
     --zone=us-central1-a \
     --machine-type=e2-medium