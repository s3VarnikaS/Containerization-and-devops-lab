## Name: Varnika Singh

Sap-ID: 500120368,
School of Computer Science,
University of Petroleum and Energy Studies, Dehradun

## EXPERIMENT – 12
Kubernetes Deployment and Orchestration

## Aim
To understand and implement Kubernetes for container orchestration, deploy applications using YAML manifests, scale services dynamically, and manage production-grade container deployments across multiple nodes.

## Objectives

- To understand Kubernetes architecture and components
- To set up and configure Kubernetes cluster using kubeadm
- To create and deploy Kubernetes manifests (Deployments and Services)
- To manage pods and replicas
- To scale applications dynamically
- To understand service discovery and load balancing in Kubernetes
- To observe automatic failover and self-healing capabilities
- To compare Docker Swarm vs Kubernetes for production deployments

## Theory

### Kubernetes Overview

Kubernetes is an open-source container orchestration platform that automates many of the manual processes involved in deploying, managing, and scaling containerized applications.

### Key Kubernetes Concepts

**Pod**: Smallest deployable unit in Kubernetes, typically contains one container (can have multiple)

**Deployment**: Manages replicas of pods, handles rolling updates, and ensures desired state

**Service**: Exposes pods to network traffic, provides stable IP and DNS name

**Node**: Worker machine in Kubernetes cluster (VM or physical machine)

**Cluster**: Set of nodes managed by Kubernetes control plane

**Control Plane**: Manages cluster state and orchestration decisions

### Kubernetes vs Docker Swarm

| Feature | Docker Swarm | Kubernetes |
|---------|--------------|-----------|
| Learning Curve | Easy | Steep |
| Setup | Simple | Complex |
| Scalability | Medium | Enterprise |
| Networking | Built-in | Requires plugins (Calico, Flannel) |
| Storage | Limited | Advanced |
| Multi-cloud | Limited | Excellent |
| Production Ready | Small-Medium | Large-Scale |
| Community | Small | Very Large |

### Deployment Progression

```
Single Host    → Multi-Host Single Machine → Multi-Host Cluster
   (docker)           (Docker Compose)      (Docker Swarm / Kubernetes)
```

### Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Kubernetes Cluster                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │         Control Plane (Master Node)                     │    │
│  │  ┌──────────────────────────────────────────────────┐   │    │
│  │  │ API Server    │ etcd    │ Scheduler │ Controller │   │    │
│  │  └──────────────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────────────┘    │
│                           │                                     │
│         ┌─────────────────┼─────────────────┐                   │
│         ▼                 ▼                 ▼                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  Worker 1   │  │  Worker 2   │  │  Worker 3   │              │
│  │  ┌───────┐  │  │  ┌───────┐  │  │  ┌───────┐  │              │
│  │  │ Pod 1 │  │  │  │ Pod 2 │  │  │  │ Pod 3 │  │              │
│  │  └───────┘  │  │  └───────┘  │  │  └───────┘  │              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Software Requirements

- Linux OS with sudo access
- kubeadm, kubelet, kubectl installed
- Container runtime (Docker or containerd)
- 2GB+ RAM per machine
- Multiple machines for cluster (or VMs)

## Procedure / Steps to Perform the Experiment

### Step 1: Install Kubernetes on Ubuntu

Update system packages:

```bash
sudo apt update
```

Install required packages:

```bash
sudo apt install -y apt-transport-https ca-certificates curl
```

Add Kubernetes signing key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add Kubernetes repository:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Update package lists:

```bash
sudo apt update
```

Install kubeadm, kubelet, and kubectl:

```bash
sudo apt install -y kubeadm kubelet kubectl
```

Hold versions to prevent automatic updates:

```bash
sudo apt-mark hold kubeadm kubelet kubectl
```

### Step 2: Initialize Kubernetes Master Node

Initialize the control plane:

```bash
sudo kubeadm init
```

This command:
- Sets up the Kubernetes control plane
- Creates necessary certificates
- Initializes etcd database
- Generates join tokens for worker nodes
- Displays instructions for cluster setup

Create .kube directory:

```bash
mkdir -p $HOME/.kube
```

Copy admin configuration:

```bash
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

Fix permissions:

```bash
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 3: Install Container Network Plugin (Calico)

Deploy Calico for pod networking:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

This enables communication between pods across nodes.

### Step 4: Join Worker Nodes to Cluster

On master node, get join command:

```bash
kubeadm token create --print-join-command
```

On each worker node, run the join command:

```bash
kubeadm join 192.168.1.100:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:...
```

Replace with actual token and hash from your cluster.

### Step 5: Verify Cluster Status

List all nodes:

```bash
kubectl get nodes
```

**Output:**
```
NAME       STATUS   ROLES           AGE   VERSION
master     Ready    control-plane   5m    v1.29.0
worker1    Ready    <none>          2m    v1.29.0
worker2    Ready    <none>          2m    v1.29.0
```

### Step 6: Create WordPress Deployment YAML

Create `wordpress-deployment.yaml`:

```yaml
# wordpress-deployment.yaml
apiVersion: apps/v1                    # Which Kubernetes API to use
kind: Deployment                       # Type of resource
metadata:
  name: wordpress                      # Name of this deployment
spec:
  replicas: 2                          # Run 2 identical pods
  selector:
    matchLabels:
      app: wordpress                   # Pods with this label belong to this deployment
  template:                            # Template for the pods
    metadata:
      labels:
        app: wordpress                 # Label applied to each pod
    spec:
      containers:
      - name: wordpress
        image: wordpress:latest        # Docker image
        ports:
        - containerPort: 80            # Port inside the container
```

### Step 7: Deploy WordPress

Apply the deployment manifest:

```bash
kubectl apply -f wordpress-deployment.yaml
```

This creates a deployment with 2 replicas of WordPress pods.

### Step 8: Create WordPress Service YAML

Create `wordpress-service.yaml`:

```yaml
# wordpress-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-service
spec:
  type: NodePort                       # Exposes service on a port of each node (VM)
  selector:
    app: wordpress                     # Send traffic to pods with this label
  ports:
    - port: 80                         # Service port
      targetPort: 80                   # Pod port
      nodePort: 30007                  # External port (range: 30000–32767)
```

### Step 9: Deploy Service

Apply the service manifest:

```bash
kubectl apply -f wordpress-service.yaml
```

This creates a service that exposes WordPress pods on port 30007 of every node.

### Step 10: List Pods

View all running pods:

```bash
kubectl get pods
```

**Output:**
```
NAME                         READY   STATUS    RESTARTS   AGE
wordpress-xxxxx-yyyyy        1/1     Running   0          1m
wordpress-xxxxx-zzzzz        1/1     Running   0          1m
```

READY column shows (current running/total desired).

### Step 11: List Services

View all services:

```bash
kubectl get svc
```

**Output:**
```
NAME                 TYPE       CLUSTER-IP     PORT(S)        AGE
wordpress-service    NodePort   10.43.x.x      80:30007/TCP   1m
kubernetes           ClusterIP  10.43.0.1      443/TCP        10m
```

The NodePort shows mapping between internal and external ports.

### Step 12: Access Application

Find your node IP:

**On Minikube:**
```bash
minikube ip
```

**On Multi-node cluster:**
Use the IP of any worker node.

**On Local Machine:**
```
http://localhost:30007
```

Access WordPress installation page through the NodePort.

### Step 13: Scale Deployment

Increase replicas to 4:

```bash
kubectl scale deployment wordpress --replicas=4
```

Verify scaling:

```bash
kubectl get pods
```

All 4 WordPress pods should be running across available nodes.

### Step 14: Simulate Pod Failure

Get pod names:

```bash
kubectl get pods
```

Delete one pod:

```bash
kubectl delete pod <pod-name>
```

Example:
```bash
kubectl delete pod wordpress-xxxxx-yyyyy
```

### Step 15: Observe Self-Healing

Check pods again:

```bash
kubectl get pods
```

Kubernetes automatically creates a new pod to replace the deleted one, maintaining the desired replica count of 4.

## Kubernetes Manifest Structure

### Deployment Manifest Breakdown

```yaml
apiVersion: apps/v1                    # API version
kind: Deployment                       # Resource type
metadata:
  name: wordpress                      # Deployment name
spec:
  replicas: 2                          # Number of pod copies
  selector:
    matchLabels:
      app: wordpress                   # Label selector
  template:
    metadata:
      labels:
        app: wordpress                 # Pod labels
    spec:
      containers:                      # Container specs
      - name: wordpress
        image: wordpress:latest        # Docker image
        ports:
        - containerPort: 80            # Port exposed by container
```

### Service Manifest Breakdown

```yaml
apiVersion: v1
kind: Service                          # Service resource
metadata:
  name: wordpress-service              # Service name
spec:
  type: NodePort                       # Service type
  selector:
    app: wordpress                     # Pods to expose
  ports:
    - port: 80                         # Service port
      targetPort: 80                   # Pod port
      nodePort: 30007                  # External node port
```

## Common Kubernetes Commands

| Command | Purpose |
|---------|---------|
| `kubectl apply -f file.yaml` | Apply/create resource from manifest |
| `kubectl get pods` | List all pods |
| `kubectl get svc` | List all services |
| `kubectl get nodes` | List all cluster nodes |
| `kubectl get deployment` | List all deployments |
| `kubectl describe pod <name>` | Show detailed pod info |
| `kubectl scale deployment <name> --replicas=N` | Scale deployment |
| `kubectl delete pod <name>` | Delete a pod |
| `kubectl delete deployment <name>` | Delete entire deployment |
| `kubectl logs <pod-name>` | View pod logs |
| `kubectl exec -it <pod-name> -- /bin/bash` | Execute command in pod |
| `kubectl port-forward <pod-name> 8080:80` | Forward local port to pod |

## Troubleshooting

### Pods Not Running
- Check logs: `kubectl logs <pod-name>`
- Describe pod: `kubectl describe pod <pod-name>`
- Verify image is available
- Check node resources

### Service Not Accessible
- Verify service type and port: `kubectl get svc`
- Check pod labels match selector: `kubectl get pods --show-labels`
- Test connectivity: `kubectl port-forward service/wordpress-service 8080:80`

### Nodes Not Ready
- Check node status: `kubectl get nodes`
- Verify network plugin is installed: `kubectl get pods -n kube-system`
- Check kubelet logs on node

## Result

Kubernetes cluster successfully deployed with multiple nodes. WordPress application deployed with 4 replicas across nodes with automatic load balancing. Self-healing demonstrated by automatic pod recreation when pods failed. Service discovery and NodePort access verified.

## Comparison: Docker Swarm vs Kubernetes

### Docker Swarm
- **Setup**: Minutes
- **Learning**: Easy
- **Scale**: Up to thousands of nodes
- **Persistence**: Requires external storage
- **Networking**: Built-in

### Kubernetes
- **Setup**: Hours/Days for production
- **Learning**: Steep curve
- **Scale**: Hundreds of thousands of nodes
- **Persistence**: Advanced persistent volumes
- **Networking**: Plugin-based (Calico, Flannel, etc.)

### When to Use Each

**Docker Swarm**: Small teams, simple deployments, rapid development

**Kubernetes**: Large organizations, complex applications, multi-cloud, enterprise requirements

## Conclusion

Kubernetes provides enterprise-grade container orchestration with advanced features like automatic scaling, self-healing, rolling updates, and multi-cloud support. While more complex than Docker Swarm, Kubernetes is the industry standard for production container deployments at scale. The declarative YAML manifest approach enables version control and GitOps workflows, making infrastructure as code a best practice.

## Viva-Voce Questions

1. What is the difference between a Deployment and a Pod?
2. How does Kubernetes schedule pods across nodes?
3. What is a NodePort and when would you use it?
4. How does Kubernetes ensure self-healing of failed pods?
5. What is the role of etcd in Kubernetes?
6. How does the Scheduler decide which node to place a pod on?
7. What happens when you scale a deployment?
8. How do you provide persistent storage in Kubernetes?
9. What is the difference between ClusterIP, NodePort, and LoadBalancer services?
10. How would you perform a rolling update of a deployment?

## References

1. Kubernetes Official Documentation
2. kubeadm Installation Guide
3. Calico Network Plugin Documentation
4. Kubernetes Service Types Documentation
5. Kubernetes Deployment Documentation
