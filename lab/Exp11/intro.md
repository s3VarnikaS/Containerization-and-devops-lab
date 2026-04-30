# Experiment 11: Docker Swarm — Orchestration and Container Management

## Name: Varnika Singh

Sap-ID: 500120368
School of Computer Science,
University of Petroleum and Energy Studies, Dehradun

## EXPERIMENT – 11
Docker Swarm — Container Orchestration, Clustering, and Service Management

## Aim
To install, configure, and manage Docker Swarm, deploy containerized services across multiple nodes, and understand container orchestration, load balancing, and scaling in a clustered environment.

## Objectives

- To understand the concepts of container orchestration and clustering
- To initialize and configure a Docker Swarm cluster
- To join nodes to the swarm
- To deploy services across multiple nodes
- To perform service scaling and management
- To understand load balancing in Docker Swarm
- To manage and troubleshoot swarm services

## Theory

### What is Docker Swarm?

Docker Swarm is a native orchestration tool for Docker that allows you to manage multiple Docker containers across multiple machines. It turns a group of Docker engines into a single virtual Docker engine.

**Key Characteristics:**

- **Container Orchestration**: Automatically schedules and deploys containers across nodes
- **Clustering**: Combines multiple Docker hosts into a single cluster
- **Service Deployment**: Abstracts away individual containers and works with services
- **Automatic Scheduling**: Places containers on available nodes based on resource requirements
- **Load Balancing**: Distributes traffic across replicated services
- **Scaling**: Easily scale services up or down
- **High Availability**: Ensures services are always running, replacing failed containers

### Docker Swarm vs Kubernetes

| Feature | Docker Swarm | Kubernetes |
|---------|--------------|-----------|
| Learning Curve | Easy | Steep |
| Setup Complexity | Simple | Complex |
| Scalability | Medium | Enterprise-grade |
| Orchestration | Basic | Advanced |
| Best For | Small to Medium teams | Large, complex deployments |

### Core Concepts

**Swarm**: A cluster of Docker daemons running in swarm mode
**Node**: A Docker instance participating in the swarm (Manager or Worker)
**Manager Node**: Controls the swarm, maintains cluster state, assigns tasks
**Worker Node**: Executes tasks assigned by managers
**Service**: An abstraction that defines how to run a Docker image in the swarm
**Task**: An individual container running as part of a service
**Stack**: A group of services that make up an application

### Swarm Architecture

```
┌─────────────────────────────────────────────┐
│         Docker Swarm Cluster                │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────────────┐  ┌──────────────────┐ │
│  │  Manager Node 1  │  │  Manager Node 2  │ │
│  │  (Raft State)    │◄─►(Replicated)     │ │
│  └──────────────────┘  └──────────────────┘ │
│         ▲                       ▲            │
│         │                       │            │
│   ┌─────┴───────┬───────────────┴─────┐     │
│   │             │                     │     │
│   ▼             ▼                     ▼     │
│┌──────────┐ ┌──────────┐ ┌──────────┐      │
││Worker 1  │ │Worker 2  │ │Worker 3  │      │
│└──────────┘ └──────────┘ └──────────┘      │
│                                             │
└─────────────────────────────────────────────┘
```

## Software Requirements

- Docker Desktop or Docker Engine with Swarm support
- Multiple Docker nodes (can be on same machine for testing)
- Linux OS or WSL (Windows Subsystem for Linux) with Docker

## Procedure / Steps to Perform the Experiment

### Step 1: Initialize Docker Swarm

Initialize the first node as a manager:

```bash
docker swarm init --advertise-addr <MANAGER_IP>
```

Example:
```bash
docker swarm init --advertise-addr 192.168.1.100
```

![](swarm%20initialization.png)

This command:
- Initializes the swarm
- Designates this node as a manager
- Outputs a join token for worker nodes
- Sets up the cluster state management

### Step 2: Get Join Tokens

To add worker nodes to the swarm, you need the join token:

**For Worker Nodes:**
```bash
docker swarm join-token worker
```

**For Additional Manager Nodes:**
```bash
docker swarm join-token manager
```

These commands display the exact command to run on other nodes to join the swarm.

### Step 3: Join Worker Nodes to Swarm

On other machines/nodes, run the join command:

```bash
docker swarm join --token SWMTKN-1-xxxxx <MANAGER_IP>:2377
```

Example:
```bash
docker swarm join --token SWMTKN-1-5di8g6iou2e6q8 192.168.1.100:2377
```

### Step 4: List Nodes in the Swarm

From the manager node, view all connected nodes:

```bash
docker node ls
```

![](list%20of%20nodes.png)

This displays:
- Node ID
- Hostname
- Status (Ready/Down)
- Availability (Active/Drain)
- Manager Status (Leader/Reachable for managers)

### Step 5: Create a Docker Compose File for Multi-Service Deployment

Create a `docker-compose.yml` file to define services:

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    deploy:
      replicas: 3
      placement:
        constraints: [node.role == worker]
    networks:
      - webnet

  database:
    image: postgres:13
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: admin123
    deploy:
      replicas: 1
      placement:
        constraints: [node.role == manager]
    networks:
      - webnet

networks:
  webnet:
    driver: overlay
```

![](.yml%20file.png)

### Step 6: Deploy the Stack

Deploy the services to the swarm:

```bash
docker stack deploy -c docker-compose.yml mystack
```

This creates:
- A network connecting all services
- Service replicas across available nodes
- Load balancing across replicas

![](deploy%20group%20of%20service.png)

### Step 7: List Services in the Stack

View all running services:

```bash
docker service ls
```

![](list%20of%20services.png)

This shows:
- Service name
- Replicas (current/desired)
- Image name
- Port mappings

### Step 8: Check Service Details

Get detailed information about a specific service:

```bash
docker service ps mystack_web
```

This displays:
- Task ID
- Node where task is running
- Current state
- Error messages (if any)

### Step 9: Scale a Service

Increase or decrease the number of replicas:

```bash
docker service scale mystack_web=5
```

![](service%20scale.png)

This changes the desired replica count to 5. Swarm automatically:
- Schedules new containers on available nodes
- Maintains the desired state
- Removes containers if scaling down

![](docker%20ps(after%20scailing).png)

### Step 10: Test Load Balancing

Access the service through the load balancer:

```bash
curl http://localhost:8080
```

![](local%20host.png)

Docker Swarm automatically:
- Routes requests to any available replica
- Balances load across replicas
- Handles failover if a node goes down

### Step 11: Stop and Remove Services

Stop a running service container:

```bash
docker kill <CONTAINER_ID>
```

![](docker%20kill.png)

Check containers after kill:

```bash
docker ps -a | grep mystack_web
```

![](docker%20ps%20after%20kill.png)

The scheduler automatically restarts the service to maintain the desired replica count.

### Step 12: Remove Services

Remove a service from the swarm:

```bash
docker service rm mystack_web
```

This removes all containers running the service across all nodes.

### Step 13: Remove Stack

Remove all services in a stack:

```bash
docker stack rm mystack
```

![](stack%20rm.png)

Check that services are removed:

```bash
docker service ls
```

![](list%20of%20service(empty).png)

### Step 14: Leave the Swarm

To remove a node from the swarm:

**On Manager Node:**
```bash
docker swarm leave --force
```

**On Worker Node:**
```bash
docker swarm leave
```

## Common Swarm Commands

| Command | Purpose |
|---------|---------|
| `docker swarm init` | Initialize swarm mode |
| `docker swarm join-token worker` | Get worker join token |
| `docker swarm join-token manager` | Get manager join token |
| `docker node ls` | List all nodes |
| `docker node promote <node-id>` | Promote worker to manager |
| `docker node demote <node-id>` | Demote manager to worker |
| `docker service create` | Create a service |
| `docker service ls` | List all services |
| `docker service ps <service>` | List tasks of a service |
| `docker service scale <service>=<replicas>` | Scale a service |
| `docker service update` | Update service configuration |
| `docker service rm <service>` | Remove a service |
| `docker stack deploy` | Deploy a stack |
| `docker stack ls` | List stacks |
| `docker stack rm <stack>` | Remove a stack |

## Troubleshooting

### Node Not Joining
- Check firewall allows port 2377 (manager), 7946 (node comm), 4789 (overlay)
- Verify IP connectivity between nodes
- Check join token is correct and not expired

### Service Not Starting
- Check node has sufficient resources
- Verify image is available or can be pulled
- Check service constraints and placement rules

### High Latency
- Ensure low-latency network between manager and workers
- Reduce number of manager nodes if experiencing consensus issues
- Use placement constraints to keep related services on same node

## Result

Docker Swarm was successfully configured with multiple nodes, services were deployed and scaled across the cluster, and load balancing was verified. Swarm automatically manages container placement, scaling, and failover, providing a complete orchestration solution for multi-container applications.

## Conclusion

Docker Swarm provides a lightweight, integrated container orchestration solution that is easy to set up and manage. While simpler than Kubernetes, it is ideal for teams looking for quick deployment of clustered applications without the overhead of additional tools. Swarm's built-in load balancing, automatic scaling, and high availability features make it suitable for production deployments at small to medium scale.

## Viva-Voce Questions

1. What is the difference between Docker Swarm and Kubernetes?
2. How does Docker Swarm elect a leader among manager nodes?
3. What is the purpose of overlay networks in Docker Swarm?
4. How does load balancing work in Docker Swarm?
5. What happens when a worker node goes offline?
6. How do you ensure a service always runs on manager nodes only?
7. What is the difference between `docker service scale` and replica limits?
8. How would you update a service without downtime?

## References

1. Docker Swarm Official Documentation
2. Docker Service Documentation
3. Docker Stack Compose File Reference
