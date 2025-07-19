# Azure DevOps: Managing Containers with Docker and Azure Container Instances

In this project, we explore containerization, understand Docker, and automate the deployment of a React application using Azure DevOps CI/CD pipelines into Azure Container Instances (ACI).

---

## 📋 Agenda

- What is a Container?
- Containers vs Virtual Machines
- Challenges with Traditional Deployments
- Docker Architecture Explained
- Dockerizing a React To-Do App (Multi-Stage Dockerfile)
- Azure Container Instance (ACI) Overview
- CI/CD with Azure DevOps for Container Deployment

---

## 🚢 What is a Container?

A container packages your application and its dependencies into a single unit that runs reliably across environments.

- Lightweight alternative to VMs  
- Shares host OS kernel  
- Fast startup, minimal resource usage  

---

## ⚠️ Challenges Without Containers

> “It works on my machine!”

- Differences in infrastructure across Dev, Test, and Prod  
- Manual builds and deployments  
- Environment-specific bugs and inconsistencies  

---

## 🧱 Docker Architecture (Simplified)

**Key Components:**
- **Docker Client:** CLI or GUI (e.g., Docker Desktop)  
- **Docker Host & Daemon:** Executes commands, manages images/containers  
- **Docker Registry:** Stores images (Docker Hub, Azure Container Registry)  

**Workflow:**
1. Write a Dockerfile  
2. `docker build` to create image  
3. `docker push` to registry  
4. `docker pull` in production  
5. `docker run` to start container  

---

## 📦 Dockerizing a React To-Do App (Multi-Stage Build)

**Why Multi-Stage?**  
Avoids shipping unnecessary build tools and dependencies in the final image.

```dockerfile
# Stage 1: Build
FROM node:18-alpine as installer
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Deploy with Nginx
FROM nginx:latest as deployer
COPY --from=installer /app/build /usr/share/nginx/html
```

---

## 🛠️ Creating Azure Container Registry (ACR)

1. Go to Azure Portal → Search "Container Registry"
2. Create a new ACR (Basic tier is fine for testing)
3. Enable Admin User to get:
    - Login server
    - Username
    - Password

---

## 🔁 CI/CD Pipeline in Azure DevOps

**Pipeline Steps:**
- Import GitHub repo
- Add Dockerfile
- Create Azure DevOps pipeline using Docker template
- Authenticate with ACR
- Build & Push Docker Image
- Create Azure Container Instance (ACI)

**Sample YAML Snippet:**

```yaml
trigger:
- main

resources:
- repo: self

variables:
  dockerRegistryServiceConnection: 'ID'
  imageRepository: 'containers'
  containerRegistry: 'rudraksh.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/DockerFile'
  tag: '$(Build.BuildId)'
  name: "self"

stages:
- stage: Build
  displayName: Build and push stage
  jobs:
  - job: Build
     displayName: Build
     pool:
        name: "self"
     steps:
     - task: AzureCLI@2
        inputs:
          azureSubscription: 'ID'
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: 'az acr login --name=$(containerRegistry)'

     - task: Docker@2
        displayName: Build and push an image to container registry
        inputs:
          command: buildAndPush
          repository: $(imageRepository)
          dockerfile: $(dockerfilePath)
          containerRegistry: $(dockerRegistryServiceConnection)
          tags: |
             $(tag)

     - task: AzureCLI@2
        inputs:
          azureSubscription: 'ID'
          scriptType: 'bash'
          scriptLocation: 'inlineScript'
          inlineScript: |
             az container create \
             --name day10app \
             --resource-group container_rg \
             --image $(containerRegistry)/$(imageRepository):$(tag) \
             --registry-login-server $(containerRegistry) \
             --registry-username rudraksh \
             --registry-password CHECK_ON_AZURE \
             --dns-name-label aci-demo-rudraksh \
             --os-type Linux \
             --cpu 1 \
             --memory 1.5
```

> **Note:** Store sensitive data like credentials in Azure Key Vault.

---

## ☁️ What is Azure Container Instance?

Azure Container Instances (ACI) allow you to run containers without managing virtual machines.

- Fully managed PaaS
- No infrastructure overhead
- Ideal for dev/test workloads and quick deployments

---

## ✅ Final Thoughts

Docker + Azure DevOps = simplified, consistent, and automated container deployments.  
No more “it works on my machine.” ACI takes care of the infrastructure so you can focus on code.
