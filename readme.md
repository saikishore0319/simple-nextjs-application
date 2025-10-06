# Containerize and Deploy a Next.js Application using Docker, GitHub Actions, and Minikube

## Table of Contents

| Section | Description |
|---------|-------------|
| [Setup Instructions](#setup-instructions) | How to set up environment and tools |
| [Deployment Steps for Minikube](#deployment-steps-for-minikube) | Steps to deploy the app on Minikube |
| [Local Run Commands](#local-run-commands) | Commands to run locally |
| [How to Access the Deployed Application](#how-to-access-the-deployed-application) | Accessing the app via browser |

---

## Setup Instructions

### Overview
This section provides instructions to set up GitHub Actions, Minikube, and Kubernetes secrets for deploying the Next.js application.

---

### GitHub Actions Setup

**Workflow Location:**  
`.github/workflows/docker-build.yml`

**Workflow Functionality:**  
- Builds the Docker image whenever changes are pushed to the `main` branch  
- Logs in to **GitHub Container Registry (GHCR)**  
- Pushes the image to GHCR with two tags:  
  - `latest`  
  - `v<run_number>` (example: `v3`)

**Workflow Example Values (populated by GitHub Actions):**
```yaml
username: ${{ github.actor }}
password: ${{ secrets.GITHUB_TOKEN }}

ghcr.io/${{ github.repository_owner }}/nextjs-k8s-demo:latest
ghcr.io/${{ github.repository_owner }}/nextjs-k8s-demo:v${{ github.run_number }}
```

**Workflow Path:**  
![GitHub Actions Workflow](./assets/github%20action%20.PNG)  
**Workflow Stages:**  
![Workflow Stages](./assets/Screenshot%202025-10-07%20002748.png)  
**Pushed Image at GHCR:**  
![Pushed Image at GHCR](./assets/Screenshot%202025-10-06%20201925.png)

---

### Minikube Setup

#### 1. Install Minikube

**Linux:**
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube version
```

**Start Minikube Cluster:**
```bash
minikube start --driver=docker
```
- `--driver=docker` runs Minikube inside Docker  

**Verify Cluster:**
```bash
minikube status
kubectl get nodes
```

#### 2. Create Namespace
```bash
kubectl create namespace nextjs-demo
```

#### 3. Authenticate Minikube to GHCR
Since the image is in GHCR, Kubernetes needs a secret to pull it.

**Step 1: Generate a Personal Access Token (PAT)**
- GitHub → Settings → Developer Settings → Personal Access Token → Token (classic)  
![Generate PAT](./assets/Screenshot%202025-10-06%20201840.png)

**Step 2: Create Docker Registry Secret**
```bash
kubectl create secret docker-registry ghcr-secret \
    --namespace=nextjs-demo \
    --docker-server=ghcr.io \
    --docker-username=saikishore0319 \
    --docker-password=$PAT \
    --docker-email=$EMAIL
```

**Reference in Deployment:**
```yaml
spec:
  imagePullSecrets:
    - name: ghcr-secret
```

---

## Deployment Steps for Minikube

### Step 1: Deploy Application
```bash
kubectl apply -f k8s/deployment.yaml
```
- Fetches the container from GHCR and deploys inside pods

### Step 2: Create Service
```bash
kubectl apply -f k8s/service.yaml
```
- Exposes the application via service

---

## Local Run Commands

### 1. Git
```bash
git clone <repo-url>       # Clone the repository
git add .                  # Stage changes for commit
git commit -m "message"    # Commit staged changes
git push origin main       # Push commits to main branch
```

### 2. Minikube
```bash
minikube start --driver=docker       # Start Minikube cluster locally
minikube status                       # Check cluster status
minikube dashboard                   # Open Kubernetes dashboard in browser
```

### 3. Kubernetes / kubectl
```bash
kubectl apply -f k8s/deployment.yaml   # Deploy the app
kubectl apply -f k8s/service.yaml      # Expose the app via service
kubectl get pods -n nextjs-demo        # List pods
kubectl get svc -n nextjs-demo         # List services
kubectl port-forward svc/nextjs-service 3000:3000 -n nextjs-demo  # Forward port
kubectl describe pod nextjs -n nextjs-demo  # Inspect pod details
kubectl logs nextjs -n nextjs-demo         # View pod logs
```

---

## How to Access the Deployed Application

Use the following command to access the application at localhost:
```bash
kubectl port-forward svc/nextjs-service 3000:3000 -n nextjs-demo
```
- Access the app at: [http://localhost:3000](http://localhost:3000)  
![App Screenshot](./assets/Screenshot%202025-10-07%20004414.png)


