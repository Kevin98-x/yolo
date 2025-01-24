# Kubernetes Deployment Project

This project demonstrates the deployment of a distributed application on Google Kubernetes Engine (GKE) using Kubernetes manifests.

## Live Application

The application is accessible at: [http://34.28.65.168](http://34.28.65.168)

## Service Details

Below are the details of the services running in the cluster:

| Name       | Type           | Cluster-IP      | External-IP   | Port(s)        | Age     |
|------------|----------------|-----------------|---------------|----------------|---------|
| backend    | ClusterIP      | 34.118.233.96   | <none>        | 5000/TCP       | 7m11s   |
| frontend   | LoadBalancer   | 34.118.227.233  | 34.28.65.168  | 80:31394/TCP   | 7m10s   |
| kubernetes | ClusterIP      | 34.118.224.1    | <none>        | 443/TCP        | 13m     |
| mongo      | ClusterIP      | 34.118.237.221  | <none>        | 27017/TCP      | 7m10s   |

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