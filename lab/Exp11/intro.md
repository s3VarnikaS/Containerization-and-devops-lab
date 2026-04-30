## Name: Varnika Singh

Sap-ID: 500120368,
School of Computer Science,
University of Petroleum and Energy Studies, Dehradun

## EXPERIMENT – 11
Container Orchestration with Docker Stack

## Aim
To understand and implement Docker Stack for orchestrating multi-container applications, deploy services across a Docker Swarm cluster, and manage container lifecycle including scaling, failover, and service management.

## Objectives

- To understand the progression from `docker run` to Docker Compose to Docker Swarm to Kubernetes
- To initialize and configure Docker Swarm mode
- To deploy multi-container applications using Docker Stack
- To scale services dynamically
- To understand automatic failover and self-healing
- To manage and remove deployed stacks
- To compare different orchestration tools for various use cases

## Theory

### Evolution of Container Orchestration

```
docker run  →  Docker Compose  →  Docker Swarm  →  Kubernetes
   │               │                  │                │
Single container  Multi-container    Orchestration    Advanced
                 (single host)       (basic)         orchestration
```

**docker run**: Deploy a single container on a single host
**Docker Compose**: Deploy multi-container applications on a single host
**Docker Swarm**: Deploy multi-container applications across multiple hosts with orchestration
**Kubernetes**: Enterprise-grade orchestration with advanced features

### Docker Stack

Docker Stack is a feature of Docker Swarm that allows you to:
- Define multi-service applications using Docker Compose files
- Deploy services across a cluster of Docker nodes
- Automatically manage container placement, scaling, and health
- Provide built-in load balancing and networking

### Use Cases by Deployment Environment

```
Development ──────► Compose
Testing     ──────► Compose
Small Production ─► Swarm
Large Production ─► Kubernetes
```

**Docker Compose**: Used for local development and testing on a single machine
**Docker Swarm**: Ideal for small to medium production deployments
**Kubernetes**: Required for large-scale, complex production environments

### Key Differences: `docker stack deploy` vs `docker compose up`

| Feature | `docker compose up` | `docker stack deploy` |
|---------|-------------------|----------------------|
| Single Host | Yes | No (Swarm only) |
| Multiple Hosts | No | Yes |
| Orchestration | Basic | Full |
| Scaling | Manual | Automatic |
| Failover | Manual | Automatic |
| Networking | Bridge | Overlay |

## Software Requirements

- Docker Desktop or Docker Engine with Swarm support
- Ubuntu (WSL distribution) or Linux OS
- Docker Compose file from previous experiment

## Procedure / Steps to Perform the Experiment

### Step 1: Review Docker Compose File (from Experiment 6)

Use the docker-compose.yml from your previous WordPress experiment:

```yaml
# docker-compose.yml
version: '3.9'

services:
  db:
    image: mysql:5.7
    container_name: wordpress_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db_data:/var/lib/mysql

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    depends_on:
      - db
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html

volumes:
  db_data:
  wp_data:
```

### Step 2: Clean Up Previous Compose Setup

Stop any existing compose setup to avoid conflicts:

```bash
docker compose down -v
```

Verify no containers are running:

```bash
docker ps
```

![](Task%201.png)

### Step 3: Initialize Docker Swarm

Initialize Swarm mode on your machine:

```bash
docker swarm init
```

**Output:**
```
Swarm initialized: current node (xxxxx) is now a manager.
```

![](swarm%20initialization.png)

This command:
- Enables Swarm mode on the Docker daemon
- Makes the current node a manager
- Creates the Swarm cluster
- Generates tokens for joining worker nodes

### Step 4: Verify Swarm Status

List all nodes in the swarm:

```bash
docker node ls
```

**Output:**
```
ID                            HOSTNAME    STATUS    AVAILABILITY   MANAGER STATUS
xxxxxxxxxxxx                   your-pc     Ready     Active         Leader
```

![](list%20of%20nodes.png)

The current node appears as a Leader with Active availability.

### Step 5: Deploy Stack Using Docker Compose File

Deploy the WordPress application as a stack:

```bash
docker stack deploy -c docker-compose.yml wpstack
```

**Output:**
```
Creating network wpstack_default
Creating service wpstack_db
Creating service wpstack_wordpress
```

![](deploy%20group%20of%20service.png)

The `-c` flag specifies the compose file, and `wpstack` is the stack name.

### Step 6: List Deployed Services

View all services in the stack:

```bash
docker service ls
```

**Output:**
```
ID             NAME                MODE         REPLICAS   IMAGE
xxxxx          wpstack_db          replicated   1/1        mysql:5.7
xxxxx          wpstack_wordpress   replicated   1/1        wordpress:latest
```

![](list%20of%20services.png)

This shows:
- Service name
- Replication mode
- Current/desired replicas
- Image used

### Step 7: Check Service Tasks

View the individual tasks (containers) running for a service:

```bash
docker service ps wpstack_wordpress
```

**Output:**
```
ID             NAME                  IMAGE              NODE     DESIRED STATE   CURRENT STATE
xxxxx          wpstack_wordpress.1   wordpress:latest   your-pc   Running         Running
```

This shows which node is running each replica and the current state.

### Step 8: Verify Containers

List all running containers:

```bash
docker ps
```

**Output:**
```
CONTAINER ID        IMAGE               STATUS              NAMES
xxxxx               wordpress:latest    Up 2 minutes        wpstack_wordpress.1.xxxxx
yyyyy               mysql:5.7           Up 2 minutes        wpstack_db.1.xxxxx
```

### Step 9: Test Application

Access the WordPress application at:

```
http://localhost:8080
```

![](local%20host.png)

You should see the WordPress installation page.

### Step 10: Scale the WordPress Service

Increase the number of WordPress replicas to 3:

```bash
docker service scale wpstack_wordpress=3
```

**Output:**
```
wpstack_wordpress scaled to 3
overall progress: 3 out of 3 tasks
```

![](service%20scale.png)

Check the updated service status:

```bash
docker service ls
```

**Output:**
```
ID             NAME                MODE         REPLICAS   IMAGE
xxxxx          wpstack_wordpress   replicated   3/3        wordpress:latest
```

### Step 11: Verify Scaled Services

View all tasks for the scaled service:

```bash
docker service ps wpstack_wordpress
```

Check running containers:

```bash
docker ps | grep wordpress
```

![](docker%20ps(after%20scailing).png)

All three WordPress containers should be running.

### Step 12: Test Load Balancing

Access the application multiple times:

```bash
localhost:8080
```

Docker Swarm automatically distributes requests across all three replicas.

### Step 13: Simulate Container Failure

Kill one of the running WordPress containers:

```bash
docker ps | grep wordpress
docker kill <container-id>
```

Example:
```bash
docker kill a1b2c3d4e5f6
```

![](docker%20kill.png)

### Step 14: Observe Automatic Failover

Check the service status:

```bash
docker service ps wpstack_wordpress
```

**Output:**
```
ID             NAME                      IMAGE              NODE     DESIRED STATE   CURRENT STATE
xxxxx          wpstack_wordpress.1       wordpress:latest   your-pc   Running         Running
yyyyy          wpstack_wordpress.2       wordpress:latest   your-pc   Running         Running
zzzzz          wpstack_wordpress.3       wordpress:latest   your-pc   Running         Running
aaaaa          \_ wpstack_wordpress.3    wordpress:latest   your-pc   Shutdown        Failed 5 seconds ago
```

![](docker%20grep(after%20kill).png)

The failed container is automatically replaced with a new one to maintain the desired state (3 replicas).

### Step 15: Verify Application Still Running

Check that WordPress is still accessible:

```bash
docker ps | grep wordpress
```

![](docker%20ps%20after%20kill.png)

All three containers are still running, demonstrating self-healing.

### Step 16: Remove the Stack

Remove all services and clean up:

```bash
docker stack rm wpstack
```

**Output:**
```
Removing service wpstack_db
Removing service wpstack_wordpress
Removing network wpstack_default
```

![](stack%20rm.png)

### Step 17: Verify Stack Removal

Confirm all services are removed:

```bash
docker service ls
```

**Output:**
```
ID                  NAME                MODE                REPLICAS            IMAGE
```

![](list%20of%20service(empty).png)

Check that no containers are running:

```bash
docker ps
```

### Step 18: Clean Up Volumes (Optional)

Remove orphaned volumes:

```bash
docker volume prune
```

This frees up disk space from volumes no longer in use.

### Step 19: Leave Swarm (Optional)

To disable Swarm mode:

```bash
docker swarm leave --force
```

The `--force` flag is needed for manager nodes.

## When to Use Each Tool

### Docker Compose
- **Use Case**: Local development and testing
- **Command**: `docker compose up -d`
- **Single Host**: Yes
- **Scaling**: Manual only

### Docker Stack (Swarm)
- **Use Case**: Small to medium production deployments
- **Command**: `docker stack deploy -c docker-compose.yml <stack-name>`
- **Multiple Hosts**: Yes
- **Scaling**: Dynamic with `docker service scale`
- **Failover**: Automatic

### Kubernetes
- **Use Case**: Large-scale, complex production environments
- **Multiple Hosts**: Yes
- **Scaling**: Dynamic and sophisticated
- **Failover**: Advanced with health checks

## Key Commands Quick Reference

```bash
# Initialize Swarm
docker swarm init

# Deploy stack
docker stack deploy -c docker-compose.yml <stack-name>

# List services
docker service ls

# Scale service
docker service scale <stack-name_service-name>=<replicas>

# See service tasks
docker service ps <service-name>

# Remove stack
docker stack rm <stack-name>

# Leave Swarm (if needed)
docker swarm leave --force

# Get join token for workers
docker swarm join-token worker

# Join worker to swarm
docker swarm join --token <token> <manager-ip>:2377

# List nodes
docker node ls
```

## Result

Docker Stack successfully deployed a multi-container WordPress application across a Docker Swarm cluster. The application demonstrated:
- Multi-service orchestration
- Automatic container placement
- Dynamic service scaling
- Automatic failover and self-healing
- Load balancing across replicas

All services maintained desired state and continued operation despite container failures.

## Conclusion

Docker Stack provides a simple yet powerful way to orchestrate multi-container applications in production. By using the same Docker Compose format for both development and production, teams can ensure consistency across environments. Swarm's automatic scaling, failover, and self-healing capabilities make it suitable for production deployments without the complexity of Kubernetes. The progression from `docker run` → Compose → Swarm → Kubernetes allows teams to scale their infrastructure as requirements grow.

## Viva-Voce Questions

1. What is the difference between `docker compose up` and `docker stack deploy`?
2. How does Docker Swarm automatically replace failed containers?
3. What is the difference between scaling and rolling updates?
4. Why is overlay networking used in Docker Swarm?
5. When would you choose Docker Swarm over Kubernetes?
6. How does the `--scale` flag work in docker service?
7. What happens when you kill a container in a scaled service?
8. How would you deploy a new version of a service without downtime?

## References

1. Docker Stack Official Documentation
2. Docker Swarm Mode Documentation
3. Docker Compose File Reference
4. Docker Service Scale Documentation
