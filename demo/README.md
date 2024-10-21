# Setting Up Local WSL Environment

This README provides instructions for setting up this CI/CD pipeline locally on a Windows Subsystem for Linux (WSL) environment. Currently, my PostgreSQL database is still set up locally and I connect to it from WSL.

## Prerequisites

1. **Windows 10**
2. **Windows Features**:
   - Ensure the following Windows features are enabled:
     - Windows Subsystem for Linux
     - Virtual Machine Platform

## Installation Steps

### 1. Enable WSL and Virtual Machine Platform

Open PowerShell as Administrator and run the following command:

```powershell
wsl --install

# Verify with
wsl --version
```

Issues I had:
1. Slow start to WSL on VSCode
2. Connecting to local PostgreSQL DB

Solution:
1. Went into Task Manager and killed WSL services, then Restarted "LxssManager" related services in services.msc. (Hit Windows + R and type services.msc to access)
2. Ensure application.properties is pointing to WindowsIP. Allow connections from WSLIP.

### 2. Install Docker on WSL Ubuntu
Download Docker Desktop for Windows. Navigate to Settings > Resouces > WSL Integration. Ubuntu distro should automatically be fetched, enable the integration. Alternatively:
1. Update your package repository:
   ```bash
   sudo apt-get update
   ```
2. Install Docker:
   ```bash
   sudo apt install docker.io
   ```
3. Add your user to the Docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```
4. Restart your terminal or run:
   ```bash
   newgrp docker
   ```

#### Verify Docker Installation
Run:
```bash
docker --version
```

You should see the Docker version. Now Docker is ready to use.

### 2.5 Dockerize the Application
Create a `Dockerfile` in the root of your project to containerize your Spring Boot application. Build the Docker image with the following command:
```bash
docker build -t your-docker-image-name .
```

### 2.75 Set up JFrog Artifactory
Follow the steps to create and set up JFrog Artifactory. Ensure you are authenticated and logged in with the correct credentials. Push the image.
1. **Authenticate with JFrog**: Log in to your JFrog Artifactory.
   ```bash
   docker login myname.jfrog.io
   ```
2. **Tag the Image**: Tag your image appropriately.
   ```bash
   docker tag your-docker-image-name myname.jfrog.io/artifactory/springapp-docker/springapp:v1
   ```
3. **Push the Image**: Push your Docker image to JFrog Artifactory.
   ```bash
   docker push myname.jfrog.io/artifactory/springapp-docker/springapp:v1
   ```


### 3. Install MicroK8s on WSL Ubuntu
Run:
```bash
snap install microk8s --classic --channel=1.28/stable

# HOST to lower case
sudo hostnamectl set-hostname desktop-gpci6s3

sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
newgrp microk8s

# Enable add-ons
microk8s enable dns
microk8s enable storage
microk8s enable ingress # Failed for me for unknown reason
microk8s enable dashboard

# Alias kubectl: MicroK8s provides its own kubectl command. Alias it for easier usage.
alias kubectl='microk8s kubectl'

# Create namespace for project
kubectl create namespace springapp-namespace
```

### 3.5 Deploy the Application
Create a `deployment.yaml` file in your project directory for Kubernetes deployment. Apply the configuration with:
```bash
microk8s kubectl apply -f deployment.yaml
```

### 4. Install ArgoCD on WSL Ubuntu
Followed https://medium.com/@SambathKumarJ/argo-cd-declarative-gitops-of-cd-for-kubernetes-430c4801e80b 
Run:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Install ArgoCD CLI
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# Access ArgoCD API Server
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Port Forwarding
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Navigate to 8080 and login as admin, retrieve pass using
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

### 4.5 Configure Spring Booot App in ArgoCD for Your GitHub Repository
1. Create a new application in ArgoCD.
2. Set the **Repo URL** to your GitHub repository containing the Kubernetes manifests.
3. Specify the **target revision** (e.g., `WSL-edits` branch).
4. Set the **path** to your manifests (e.g., `./demo/k8s`).
5. Choose the appropriate **cluster** and **namespace** where the application will be deployed.

Decide on the sync policy (manual or automatic) based on your deployment strategy. Sync the application to deploy your Spring Boot app to the Kubernetes cluster.
