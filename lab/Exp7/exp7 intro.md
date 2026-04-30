# Experiment 7: CI/CD using Jenkins, GitHub and Docker Hub

## Aim

To design and implement a complete CI/CD pipeline using **Jenkins**, integrating source code from **GitHub**, and building & pushing Docker images to **Docker Hub**.

---

## Objectives

- Understand CI/CD workflow using Jenkins (GUI-based tool)
- Create a structured GitHub repository with application + Jenkinsfile
- Build Docker images from source code
- Securely store Docker Hub credentials in Jenkins
- Automate build & push process using webhook triggers
- Use same host (Docker) as Jenkins agent

---

## Theory

### What is Jenkins?

Jenkins is a **web-based GUI automation server** used to:

- Build applications
- Test code
- Deploy software

It provides:

- Dashboard (browser-based UI)
- Plugin ecosystem (GitHub, Docker, etc.)
- Pipeline as Code using `Jenkinsfile`

### What is CI/CD?

**Continuous Integration (CI):**
Code is automatically built and tested after each commit.

**Continuous Deployment (CD):**
Built artifacts (Docker images) are automatically delivered/deployed.

### Workflow Overview

```
Developer → GitHub → Webhook → Jenkins → Build → Docker Hub
```

---

## Prerequisites

- Docker & Docker Compose installed
- GitHub account
- Docker Hub account
- Basic Linux command knowledge

---

## Part A: GitHub Repository Setup (Source Code + Build Definition)

### 5.1 Create Repository

Create a repository on GitHub:

```
my-app
```

### 5.2 Project Structure

```
my-app/
├── app.py
├── requirements.txt
├── Dockerfile
└── Jenkinsfile
```

### 5.3 Application Code

**`app.py`**

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from CI/CD Pipeline!"
    #return "Hello from CI/CD Pipeline!, my sapid is 123456"

app.run(host="0.0.0.0", port=80)
```

**`requirements.txt`**

```
flask
```

### 5.4 Dockerfile (Build Process)

```dockerfile
FROM python:3.10-slim

WORKDIR /app
COPY . .

RUN pip install -r requirements.txt

EXPOSE 80
CMD ["python", "app.py"]
```

**Build Process Explanation:**

1. Source code pushed to GitHub
2. Jenkins pulls code
3. Dockerfile:
   - Creates environment
   - Installs dependencies
   - Packages app
4. Output → Docker Image

### 5.5 Jenkinsfile (Pipeline Definition in GitHub)

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "your-dockerhub-username/myapp"
    }

    stages {
        stage('Clone Source') {
            steps {
                git 'https://github.com/your-username/my-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
                    sh 'echo $DOCKER_TOKEN | docker login -u your-dockerhub-username --password-stdin'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'docker push $IMAGE_NAME:latest'
            }
        }
    }
}
```

---

## Part B: Jenkins Setup using Docker (Persistent Configuration)

### 6.1 Create Docker Compose File

```yaml
version: '3.8'

services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: always
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    user: root

volumes:
  jenkins_home:
```

### 6.2 Start Jenkins

```bash
docker-compose up -d
```

Access Jenkins at:

```
http://localhost:8080
```

### 6.3 Unlock Jenkins

```bash
docker exec -it jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### 6.4 Initial Setup

- Install suggested plugins
- Create admin user

---

## Part C: Jenkins Configuration

### 7.1 Add Docker Hub Credentials

Path:

```
Manage Jenkins → Credentials → Add Credentials
```

- Type: Secret Text
- ID: `dockerhub-token`
- Value: Docker Hub Access Token

### 7.2 Create Pipeline Job

1. New Item → Pipeline
2. Name: `ci-cd-pipeline`

Configure:

```
Pipeline script from SCM
```

- SCM: Git
- Repo URL: your GitHub repo
- Script Path: `Jenkinsfile`

---

## Part D: GitHub Webhook Integration

### 8.1 Configure Webhook

In GitHub:

```
Settings → Webhooks → Add Webhook
```

Payload URL:

```
http://<your-server-ip>:8080/github-webhook/
```

Events:

```
Push events
```

---

## Part E: Execution Flow (Stage-wise Explanation)

### Stage 1: Code Push

- Developer updates code in GitHub

### Stage 2: Webhook Trigger

- GitHub sends event to Jenkins

### Stage 3: Jenkins Pipeline Execution

**Stage: Clone**
- Pulls latest code from GitHub

**Stage: Build**
- Docker builds image using Dockerfile

**Stage: Auth**
- Jenkins logs into Docker Hub using stored token

**Stage: Push**
- Image pushed to Docker Hub

### Stage 4: Artifact Ready

- Docker image available globally

---

## Part F: Role of Same Host Agent

- Jenkins runs inside Docker
- Docker socket mounted:

```
/var/run/docker.sock
```

**Effect:** Jenkins container can run Docker commands on the host machine directly — no separate Docker agent needed.

---

## Part G: Understanding Jenkins Pipeline Syntax

Jenkins pipelines (written in a `Jenkinsfile`) follow a **clear structure of blocks and steps**.

### 1. Basic Pipeline Structure

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'echo Hello'
            }
        }
    }
}
```

### 2. Key Terms Explained

**1. `pipeline {}`**
- Root block
- Everything is written inside this

**2. `agent`**
```groovy
agent any
```
- Defines **where the pipeline runs**
- `any` → run on any available node (in your case, same Docker host)

**3. `stages {}`**
- Groups all phases of pipeline
- Logical separation (Clone, Build, Test, Deploy)

**4. `stage('Name')`**
```groovy
stage('Build')
```
- A **single step/phase** in pipeline
- Visible in Jenkins GUI as blocks

**5. `steps {}`**
- Contains actual commands to execute

### 3. Common Steps

**`git`**
```groovy
git 'https://github.com/user/repo.git'
```
- Clones source code from **GitHub**

**`sh` (Shell Command)**
```groovy
sh 'echo Hello'
```
- Runs Linux commands inside agent
- Equivalent to terminal command

**`echo`**
```groovy
echo "Build started"
```
- Prints message in Jenkins console log

### 4. Full Readable Example

```groovy
pipeline {
    agent any

    stages {
        stage('Clone') {
            steps {
                git 'https://github.com/user/repo.git'
            }
        }

        stage('Build') {
            steps {
                echo "Building..."
                sh 'docker build -t myapp .'
            }
        }

        stage('Deploy') {
            steps {
                sh 'docker run -d myapp'
            }
        }
    }
}
```

---

## Part H: The Challenging Part — `withCredentials`

### Original Code

```groovy
withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
    sh 'echo $DOCKER_TOKEN | docker login -u your-username --password-stdin'
}
```

### What Problem Does This Solve?

You need to login to **Docker Hub**, but you should **NOT** write the password directly in code. Jenkins stores it securely.

### Step-by-Step Meaning

**Step 1: Store Secret in Jenkins**

You saved token with ID:
```
dockerhub-token
```

**Step 2: `withCredentials` Block**
```groovy
withCredentials([...]) {
    // use secret here
}
```
- Temporarily **injects secret into environment variable**

**Step 3: Inside the Bracket**
```groovy
string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')
```

| Part            | Meaning                  |
|-----------------|--------------------------|
| `string`        | Type of secret (text token) |
| `credentialsId` | Name used in Jenkins     |
| `variable`      | Temporary variable name  |

**Step 4: What Happens Internally**
```bash
DOCKER_TOKEN=your_actual_secret_value
```

**Step 5: Using It in `sh`**
```bash
sh 'echo $DOCKER_TOKEN | docker login -u username --password-stdin'
```
- `$DOCKER_TOKEN` → replaced with actual token
- `--password-stdin` → secure login (no plain password)

### Simplified Analogy

```
Locker (Jenkins Credentials)
    ↓
You ask by ID (dockerhub-token)
    ↓
Jenkins gives temporary key (DOCKER_TOKEN)
    ↓
You use it → then it disappears
```

### Common Mistakes

Writing password directly:
```bash
sh 'docker login -u user -p mypassword'  # ❌ Wrong
```

Wrong credentialsId:
```groovy
credentialsId: 'wrong-name'  # ❌ Wrong
```

Using variable outside block:
```groovy
echo $DOCKER_TOKEN  // won't work outside  # ❌ Wrong
```

### Clean Final Example (Student Friendly)

```groovy
stage('Push to Docker Hub') {
    steps {
        withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKER_TOKEN')]) {
            echo "Logging into Docker Hub"

            sh '''
            echo $DOCKER_TOKEN | docker login -u your-username --password-stdin
            docker push your-username/myapp:latest
            '''
        }
    }
}
```

---

## Key Takeaways

- `pipeline → stages → stage → steps` = structure
- `sh` = run commands
- `git` = fetch code
- `withCredentials` = securely use secrets
- Secrets are **temporary and protected**
