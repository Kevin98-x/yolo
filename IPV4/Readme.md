# Kubernetes Deployment Project

This project demonstrates the deployment of a distributed application on Google Kubernetes Engine (GKE) using Kubernetes manifests.

## Prerequisites

- Docker
- Google Cloud SDK (`gcloud`)
- `kubectl`

## Steps to Deploy

1. **Set Up GKE Cluster**:
   ```bash
   gcloud container clusters create kevin-yolo-cluster \
     --num-nodes=2 \
     --zone=us-central1-a \
     --machine-type=e2-medium

## Live Application

The application is accessible at: [http://34.28.65.168](http://34.28.65.168)