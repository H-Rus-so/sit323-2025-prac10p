# SIT323 Task 10.1P – Monitoring and Visibility

## Overview

This task involved deploying a Node.js microservice to a Google Kubernetes Engine (GKE) cluster. The app connects to a MongoDB database running inside the same cluster. 
Monitoring and logging were set up using Google Cloud Monitoring and Logs Explorer to track resource usage and view application logs.

## Features

- Dockerized Node.js microservice with Express
- MongoDB integration using a StatefulSet and persistent volume
- Kubernetes deployment and LoadBalancer service
- Application logs visible in Logs Explorer
- Monitoring dashboard with CPU, memory, and restart count metrics

## Deployment Instructions

### Step 1 – Build and Push Docker Image to GCR

Make sure you're authenticated to GCP in your terminal.

```bash
docker build -t gcr.io/<your-project-id>/monitoring-app .
docker push gcr.io/<your-project-id>/monitoring-app
```
Step 2 – Connect to Your GKE Cluster
```bash
gcloud container clusters get-credentials monitoring-cluster \
  --zone australia-southeast1-a \
  --project <your-project-id>
```

Step 3 – Apply Kubernetes Files
Apply all resource manifests to the cluster:

```bash
kubectl apply -f k8s/mongo-secret.yaml
kubectl apply -f k8s/mongo-pv-pvc.yaml
kubectl apply -f k8s/mongo-service.yaml
kubectl apply -f k8s/mongo-statefulset.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```
Step 4 – Get the App’s External IP
```bash
kubectl get svc
```
When the EXTERNAL-IP appears for monitoring-app-service, open the app in a browser:

```cpp
http://<external-ip>
```
You should see the app homepage confirming Task 10.1P, with MongoDB and GKE mentions.

Monitoring and Logging Setup
Logs Explorer (Cloud Logging)
Go to:
(https://console.cloud.google.com/logs/query)

Use this LQL filter to view your app’s container logs:
```sql
resource.type="k8s_container"
resource.labels.cluster_name="monitoring-cluster"
resource.labels.namespace_name="default"
resource.labels.container_name="monitoring-app"
```

Look for:

```arduino
✅ Connected to MongoDB
Express App running at http://127.0.0.1:3000/
```
Monitoring Dashboard
Go to: https://console.cloud.google.com/monitoring/dashboards


### Create a new dashboard

Add widgets for:

CPU usage (per container)

Memory usage (per container)

Restart count (per pod)


### Common Issues and Fixes
Pods not starting?
Check logs with kubectl logs <pod-name> and ensure secrets like mongo-secret are applied correctly.

No external IP?
LoadBalancer services can take 1–3 mins to take effect. Use kubectl get svc to refresh.

Auth plugin error?
If GKE auth plugin fails, run:

```bash
gcloud auth application-default login
```
App crashes after deploying to GKE?
Rebuild and push Docker image again, then restart the deployment:

```bash
kubectl rollout restart deployment monitoring-app-deployment
```


##### Author
Created by Hope Russo
Deakin University – SIT323 Cloud Native Application Development
