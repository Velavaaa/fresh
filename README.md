Tutorial: Setting Up Automated Versioning with GitHub Actions and Helm
================================================
Overview
This tutorial guides you through setting up automated versioning, building, and deployment for a Kubernetes application using GitHub Actions and Helm charts. We'll set up a system that automatically versions your containers based on Git commits and deploys them using Helm.
Prerequisites
Git installed
Docker installed and Docker Hub account
Access to a Kubernetes cluster (Minikube for local testing)
Basic understanding of YAML
GitHub account
Helm installed
Project Structure
```
mini-kube-test/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── backend/
│   ├── Dockerfile
│   ├── index.js
│   └── package.json
├── frontend/
│   ├── Dockerfile
│   ├── src/
│   └── package.json
└── myapp/              # Helm chart directory
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
        ├── backend-deployment.yaml
        ├── backend-service.yaml
        ├── frontend-deployment.yaml
        ├── frontend-service.yaml
        └── postgres-deployment.yaml
```

## Prerequisites

To follow this tutorial, you need the following tools installed:

1. **Minikube**: To create a local Kubernetes cluster. You can install Minikube using:
   - **macOS (Homebrew)**: `brew install minikube`
   - **Linux**: `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && sudo install minikube-linux-amd64 /usr/local/bin/minikube`
   - **Windows (Chocolatey)**: `choco install minikube`

2. **kubectl**: The command-line tool for managing Kubernetes clusters. Install using:
   - **macOS (Homebrew)**: `brew install kubectl`
   - **Linux**: `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && chmod +x kubectl && sudo mv kubectl /usr/local/bin/`
   - **Windows (Chocolatey)**: `choco install kubernetes-cli`

3. **Helm**: A package manager for Kubernetes that helps manage application deployments. Install using:
   - **macOS (Homebrew)**: `brew install helm`
   - **Linux/macOS**: `curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash`
   - **Windows (Chocolatey)**: `choco install kubernetes-helm`

4. **Docker**: Minikube will use Docker as the container runtime. Make sure Docker is installed and running:
   - **macOS/Windows**: Install Docker Desktop from [Docker's official website](https://www.docker.com/products/docker-desktop).
   - **Linux**: Install Docker using:
     ```sh
     sudo apt-get update
     sudo apt-get install -y docker.io
     ```

## Deployment and Access Instructions

### 1. Create a Secret for PostgreSQL

To store the PostgreSQL credentials securely, create a Kubernetes secret:

```sh
kubectl create secret generic myapp-database-secret \
    --from-literal=POSTGRES_PASSWORD=password \
    --from-literal=PGPASSWORD=password
```

### 2. Start Up Minikube

Ensure that Minikube is installed on your local machine. Start Minikube with sufficient resources:

```sh
minikube start --cpus=4 --memory=8192mb
```

### 3. Check If `myapp` is Already Running

Verify if `myapp` is already deployed using Helm:

```sh
helm list
```

If `myapp` is already running, delete it using the following command:

```sh
helm uninstall myapp
```

### 4. Install the Application Again

To deploy the application using Helm, run:

```sh
helm install myapp .
```

### 5. Port-Forward Backend and Frontend Services

To access the backend and frontend services locally, use the following commands:

- **Port-forward Backend Service**:
  ```sh
  kubectl port-forward service/myapp-backend-service 3000:3000
  ```

- **Port-forward Frontend Service**:
  ```sh
  kubectl port-forward service/myapp-frontend-service 8080:80
  ```

### 6. Access the Application

- **Access the Frontend UI**: Open the following URL in your browser:
  ```
  http://localhost:8080/
  ```

- **Access the Backend Endpoint**: You can access the backend API at:
  ```
  http://localhost:3000/person
  ```

------------------------------------------------------------------------------------------------------------------

kubectl create secret generic myapp-database-secret \
     --from-literal=POSTGRES_PASSWORD=password \
     --from-literal=PGPASSWORD=password
     
`cd myapp`

Set Up Minikube
Ensure Minikube is installed on your local machine.
Start Minikube with sufficient resources:
`minikube start --cpus=4 --memory=8192mb`


 Check if the myapp-release is already running
`helm list`

 If myapp-release is already running, then delete by using the following command
`helm uninstall myapp`

 Install the app again
`helm install myapp .`

 Port-forward backend service 
`kubectl port-forward service/myapp-backend-service 3000:3000`

 Port-forward frontend service
`kubectl port-forward service/myapp-frontend-service 8080:80`

Access the UI
`http://localhost:8080/`

Access the BE
`http://localhost:3000/person`

# Troubleshooting PostgreSQL Connection in Kubernetes

The steps to troubleshoot a PostgreSQL connection issue in a Kubernetes environment. We encountered a scenario where our backend service was unable to connect to the PostgreSQL database, and we will explain how to investigate and resolve this issue. This guide can help identify root causes and fix common issues involving database connectivity.


## How We Troubleshot the PostgreSQL Connection Issue

In this section, we will walk through the steps taken to troubleshoot and resolve the PostgreSQL connection issue in our Kubernetes environment. The backend service (`myapp-backend`) was unable to connect to the PostgreSQL database (`myapp-postgres`). Below are the detailed steps we followed to identify and solve the issue.

### Step 1: Verify the PostgreSQL Pod is Running

First, we needed to confirm that the PostgreSQL pod was up and running without any errors.

To do this, we ran the following command to check the status of all pods:

```sh
kubectl get pods
```

The output indicated that the PostgreSQL pod (`myapp-postgres-0`) was in a `Running` state, which meant that the pod was healthy and ready to accept connections.

If the pod had not been running, we would have investigated further by checking the logs with:

```sh
kubectl logs myapp-postgres-0
```

### Step 2: Check if PostgreSQL is Listening on Port 5432

We needed to ensure that PostgreSQL was properly listening on port 5432. We verified this by executing a command inside the PostgreSQL pod.

First, we accessed the PostgreSQL pod by running:

```sh
kubectl exec -it myapp-postgres-0 -- bash
```

Once inside, we updated the package index and installed `net-tools` to use `netstat` to check for listening ports:

```sh
apt-get update
apt-get install -y net-tools
netstat -tuln | grep 5432
```

The output confirmed that PostgreSQL was listening on all interfaces (IPv4 and IPv6) on port 5432, indicating that the database was set up correctly to accept connections.

### Step 3: Verify Service Endpoints

Next, we verified whether the PostgreSQL service (`myapp-postgres-service`) was correctly exposing the PostgreSQL pod. We checked the service endpoints using the following command:

```sh
kubectl get endpoints myapp-postgres-service
```

The output showed the correct IP address and port of the PostgreSQL pod (`10.244.0.27:5432`). If no endpoints had been listed, it would have indicated that the service was not properly linked to the pod.

### Step 4: Test Connectivity from Backend Pod

To ensure that the backend pod could reach the PostgreSQL service, we performed a manual connectivity test.

First, we started a shell in the backend pod:

```sh
kubectl exec -it myapp-backend-5c78d5c65b-qngh9 -- bash
```

Then, we installed the PostgreSQL client to test the connection:

```sh
apt-get update && apt-get install -y postgresql-client
```

We attempted to connect to the PostgreSQL database using the following command:

```sh
psql -h myapp-postgres-service -U postgres -d people -p 5432
```

If the connection had failed, it would have indicated a networking or service configuration issue. However, in our case, the connection was successful, meaning the backend could reach the database.

### Step 6: Check Network Policies and Firewalls

Lastly, we verified if there were any `NetworkPolicy` objects that might be blocking traffic between the backend and PostgreSQL pods.

We listed all network policies with:

```sh
kubectl get networkpolicies
```

We ensured that there were no policies that restricted ingress traffic from the backend to the PostgreSQL pod on port 5432.

## Summary

- **Verify PostgreSQL Pod**: Ensure it is running and ready.
- **Check Listening Ports**: Confirm that PostgreSQL is listening on port 5432.
- **Verify Service Endpoints**: Ensure the service is properly exposing the pod.
- **Remove Unrelated Pods**: Delete any pods not related to `myapp` to avoid conflicts.
- **Test Backend Connectivity**: Manually connect to PostgreSQL from the backend pod.
- **Check Network Policies**: Ensure no policies are blocking communication.

By following these steps, we successfully troubleshot and resolved the PostgreSQL connection issue in our Kubernetes environment.



**Step-by-Step Instructions
Step 1: Repository Setup**
Create a new directory for your project:

```mkdir mini-kube-test
cd mini-kube-test
git init```
Create the GitHub Actions directory structure:
```mkdir -p .github/workflows```

**Step 2: Create CI Workflow**
Create .github/workflows/ci.yml:
touch .github/workflows/ci.yml

Add the following content to ci.yml:
# Name of the workflow
name: CI

# When to run this workflow
on:
  push:
    branches: [ main ]  # Runs on pushes to main branch
  pull_request:
    branches: [ main ]  # Runs on PRs targeting main branch

# Environment variables used across jobs
env:
  BACKEND_IMAGE: sivaaira/backend-image    # Change to your Docker Hub username
  FRONTEND_IMAGE: sivaaira/frontend-image   # Change to your Docker Hub username

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      # Step 1: Check out the code
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Generate version number from git commit
      - name: Generate Version
        id: version
        run: |
          echo "VERSION=v$(git rev-parse --short HEAD)" >> $GITHUB_ENV
          echo "version=${{ env.VERSION }}" >> $GITHUB_OUTPUT

      # Step 3: Build backend image
      - name: Build Backend
        run: |
          docker build -t ${{ env.BACKEND_IMAGE }}:${{ env.VERSION }} ./backend
          docker tag ${{ env.BACKEND_IMAGE }}:${{ env.VERSION }} ${{ env.BACKEND_IMAGE }}:latest

      # Step 4: Build frontend image
      - name: Build Frontend
        run: |
          docker build -t ${{ env.FRONTEND_IMAGE }}:${{ env.VERSION }} ./frontend
          docker tag ${{ env.FRONTEND_IMAGE }}:${{ env.VERSION }} ${{ env.FRONTEND_IMAGE }}:latest

      # Step 5: Login to Docker Hub
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      # Step 6: Push images to Docker Hub
      - name: Push Images
        run: |
          docker push ${{ env.BACKEND_IMAGE }}:${{ env.VERSION }}
          docker push ${{ env.BACKEND_IMAGE }}:latest
          docker push ${{ env.FRONTEND_IMAGE }}:${{ env.VERSION }}
          docker push ${{ env.FRONTEND_IMAGE }}:latest

**Step 3: Create CD Workflow**
	Create .github/workflows/cd.yml:
	touch .github/workflows/cd.yml
	Add the following content:
	# Name of the deployment workflow
	name: CD
	
	# Trigger CD after CI workflow completes
	on:
	  workflow_run:
	    workflows: ["CI"]
	    types:
	      - completed
	
	jobs:
	  deploy:
	    runs-on: ubuntu-latest
	    if: ${{ github.event.workflow_run.conclusion == 'success' }}
	    steps:
	      # Step 1: Check out the code
	      - name: Checkout code
	        uses: actions/checkout@v4
	
	      # Step 2: Get version from git commit
	      - name: Get Version
	        id: version
	        run: |
	          echo "VERSION=v$(git rev-parse --short HEAD)" >> $GITHUB_ENV
	
	      # Step 3: Update Helm values with new image versions
	      - name: Update Helm Values
	        run: |
	          cd myapp
	          yq eval ".backend.image.tag = \"${{ env.VERSION }}\"" -i values.yaml
	          yq eval ".frontend.image.tag = \"${{ env.VERSION }}\"" -i values.yaml
	
	      # Step 4: Deploy using Helm
	      - name: Deploy to Kubernetes
	        run: |
	          helm upgrade --install myapp ./myapp

**Step 4: Create Helm Chart**
Create Helm chart structure:

helm create myapp
cd myapp
Update values.yaml:
# Backend configuration
backend:
  image:
    repository: sivaaira/backend-image  # Change to your Docker Hub username
    tag: latest  # This will be automatically updated by CD pipeline
  replicaCount: 1
  service:
    type: ClusterIP
    port: 3000
  env:
    PGUSER: "postgres"
    PGPASSWORD: "password"
    PGDATABASE: "people"
    PGHOST: "postgres-service"
    PGPORT: "5432"

# Frontend configuration
frontend:
  image:
    repository: sivaaira/frontend-image  # Change to your Docker Hub username
    tag: latest  # This will be automatically updated by CD pipeline
  replicaCount: 1
  service:
    type: NodePort
    port: 80
    nodePort: 30001

# PostgreSQL configuration
postgres:
  image:
    repository: postgres
    tag: "13"
  service:
    type: ClusterIP
    port: 5432
  env:
    POSTGRES_USER: "postgres"
    POSTGRES_PASSWORD: "password"
    POSTGRES_DB: "people"


**Step 5: Set Up GitHub Secrets**
Go to your GitHub repository settings
Navigate to "Secrets and variables" → "Actions"
Add the following secrets:
**DOCKERHUB_USERNAME**: Your Docker Hub username
**DOCKERHUB_TOKEN**: Your Docker Hub access token

**Step 6: Local Testing**	
Test the version generation:
VERSION="v$(git rev-parse --short HEAD)"
echo $VERSION

Build images locally:

docker build -t sivaaira/backend-image:${VERSION} ./backend
docker build -t sivaaira/frontend-image:${VERSION} ./frontend

Test Helm chart:

helm lint ./myapp
helm template ./myapp

**Step 7: Deploy**
Push your code to GitHub:
git add .
git commit -m "Initial setup"
git push origin main
Monitor GitHub Actions:
Go to your repository on GitHub
Click "Actions" tab
Watch the CI/CD pipelines run
Verify deployment:

kubectl get pods
kubectl get services

Automated Versioning and Deployment Strategy - Conceptual Overview
**1. Core Concepts**
	A. Versioning Strategy
		We use Git commit SHA as our version number
		Example: If commit is abc123456, version becomes v-abc1234
		Ensures unique version for every code change
		Provides traceability between code and deployment
		Makes rollbacks straightforward
	B. Key Components
		Source Code
		Backend (Node.js application)
		Frontend (Vue application)
		Database (PostgreSQL)
		Container Images
		Each component packaged as Docker image
		Images tagged with Git commit version
		Always maintain a 'latest' tag
		Helm Charts
		Single chart managing all components
		Centralized configuration in values.yaml
		Version-aware deployments
**2. Automation Flow**
	A. Development Phase
		Developer writes code
		Commits changes to Git
		Pushes to GitHub repository
	B. Build Phase (CI)
		GitHub Actions triggers on push
		Generates version from Git commit
		Builds Docker images
		Tags with specific version
		Tags with 'latest'
		Pushes images to Docker Hub
	C. Deployment Phase (CD)
		Updates Helm chart values
		Deploys to Kubernetes
		Updates application version

3. Benefits
	A. Traceability
		Every deployment tied to specific code version
		Easy to identify which code is running
		Clear audit trail of changes
	B. Reliability
		Automated process reduces human error
		Consistent versioning across all components
		Reproducible builds and deployments
	C. Maintainability
		Easy rollbacks to previous versions
		Simple version tracking
		Centralized configuration
4. Process Breakdown
	A. When Developer Commits Code:
		Code Change → Git Commit → GitHub → CI/CD Pipeline
	
	B. During CI (Continuous Integration):
		Code → Version Generation → Docker Build → Image Push
	
	C. During CD (Continuous Deployment):
		Image → Helm Update → Kubernetes Deployment

5. Version Flow Example
	Let's follow a single change:
	Developer makes code change
	Commits with SHA: abc123456
	CI creates images:
	backend:vabc1234
	frontend:vabc1234
	CD updates Helm values with new version
	Deploys to Kubernetes with these versions
6. Recovery Process
	If something goes wrong:
	Identify last working version
	Use Helm rollback
	System returns to previous state
	All components sync to same version

**Step 7: creating kubernetes secret for password**
    
     $ kubectl create secret generic myapp-database-secret \
     --from-literal=POSTGRES_PASSWORD=password \
     --from-literal=PGPASSWORD=password

     secret/myapp-database-secret created 

    1.change the values in deployment file(postgres/backend)
     add this in env
    - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: myapp-database-secret
                  key: POSTGRES_PASSWORD
      - name: POSTGRES_PASSWORD
              valueFrom:
               secretKeyRef:
                name: myapp-database-secret
                key: POSTGRES_PASSWORD

  
    2.secret is stored in myapp-database-secret
     helm upgrade myapp ./myapp
     


