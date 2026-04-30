# Experiment 12: Kubernetes Deployment and Orchestration

**Name:** Varnika Singh  
**SAP ID:** 500120368  
**School:** School of Computer Science  
**University:** University of Petroleum and Energy Studies, Dehradun  

---

## Objective

Learn why Kubernetes is used, its basic concepts, and how to deploy, scale, and manage applications using Kubernetes commands.

---

## Why Kubernetes over Docker Swarm?

| Reason | Explanation |
|--------|-------------|
| Industry standard | Most companies use Kubernetes |
| Powerful scheduling | Automatically decides where to run your app |
| Large ecosystem | Many tools and plugins available |
| Cloud-native support | Works on AWS, Google Cloud, Azure |

---

## Core Kubernetes Concepts (Simple Explanation)

| Docker Concept | Kubernetes Equivalent | What it means |
|---------------|----------------------|----------------|
| Container | Pod | A pod is a group of one or more containers. Smallest unit in Kubernetes |
| Compose service | Deployment | Describes how your app should run |
| Load balancing | Service | Exposes your app to the outside world |
| Scaling | ReplicaSet | Ensures required number of pod copies are running |

---

## Hands-On Lab (Using Minikube / k3d)

> Make sure `kubectl` and a cluster are already installed.

---

## PART A – Core Tasks

### Task 1: Create a Deployment

A deployment defines:
- Container image  
- Number of replicas  
- Labels for pods  

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
spec:
  replicas: 2
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest
        ports:
        - containerPort: 80

**Apply:**

```bash
kubectl apply -f wordpress-deployment.yaml

### Task 2: Expose Deployment as Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
spec:
  type: NodePort
  selector:
    app: wordpress
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007

## Apply WordPress Service

Run the following command:

```bash
kubectl apply -f wordpress-service.yaml

Task 3: Verify Everything
kubectl get pods
kubectl get svc

## Access Application

Open the app in your browser:

http://localhost:30007

## Task 4: Scale Deployment

Run the following commands:

```bash
kubectl scale deployment wordpress --replicas=4
kubectl get pods

Now 4 pods will be running.

## Task 5: Self-Healing

Run the following commands:

```bash
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods

Deleted pod is automatically recreated.

## PART B – Cluster Info

Run the following commands:

```bash
kubectl get nodes
kubectl cluster-info
