# SIT323 Week 6P – Kubernetes Deployment for Containerized Node.js Application

## Task Overview

This task demonstrates how to deploy a simple Node.js application to a local Kubernetes cluster using Docker Desktop. The app is containerized using Docker and then deployed using Kubernetes Deployment and Service resources. This submission includes all configuration files and step-by-step instructions.


## Tools and Technologies Used

- Node.js  
- Express.js  
- Docker & Docker Hub  
- Kubernetes via Docker Desktop  
- kubectl (Kubernetes CLI)  
- Visual Studio Code  
- Git & GitHub


### 1️ Install Required Tools

Ensure the following are installed:

- Node.js and npm  
- Docker Desktop  
- Git  
- Visual Studio Code

To verify installation:

node -v
npm -v
docker --version
kubectl version --client


### 2️ Enable Kubernetes in Docker Desktop

- Open Docker Desktop  
- Go to **Settings > Kubernetes**  
- Check **"Enable Kubernetes"**  
- Click **Apply & Restart**  
- Wait until Kubernetes is running


### 3️ Create Node.js App

mkdir k8s-node-app
cd k8s-node-app
npm init -y
npm install express

Create `app.js`:

const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send('Hello from Kubernetes!');
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});


### 4️ Create Dockerfile

Create a file named `Dockerfile`:

FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]


### 5️ Build and Push Docker Image

Replace `yourdockerhubusername` with your actual Docker Hub username:

docker build -t yourdockerhubusername/k8s-node-app .
docker login
docker push yourdockerhubusername/k8s-node-app


### 6️ Create Kubernetes Deployment

Create `deployment.yaml`:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-node-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: k8s-node
  template:
    metadata:
      labels:
        app: k8s-node
    spec:
      containers:
        - name: node-container
          image: yourdockerhubusername/k8s-node-app
          ports:
            - containerPort: 3000


Apply the deployment:

kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods


### 7️ Create Kubernetes Service

Create `service.yaml`:

apiVersion: v1
kind: Service
metadata:
  name: k8s-node-service
spec:
  type: NodePort
  selector:
    app: k8s-node
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30036


Apply the service:

kubectl apply -f service.yaml
kubectl get services


Access the app in your browser:  
**http://localhost:30036**


### 8️ Setup Kubernetes Dashboard

Deploy the dashboard:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml


Create `dashboard-adminuser.yaml`:

apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard


kubectl apply -f dashboard-adminuser.yaml

Create `cluster-role-binding.yaml`:

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard


kubectl apply -f cluster-role-binding.yaml


Generate token and start dashboard:

kubectl -n kubernetes-dashboard create token admin-user
kubectl proxy


Login at:

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/


## Summary

This project shows the full workflow of containerizing a Node.js app, deploying it on Kubernetes via Docker Desktop, and making it accessible locally. It covers both CLI-based deployment and an optional visual dashboard for management.
