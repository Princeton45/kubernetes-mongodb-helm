# Deploying MongoDB with Mongo Express on Linode Kubernetes Engine

## Project Overview
I successfully deployed a stateful MongoDB service with a web-based UI (Mongo Express) on a Linode Kubernetes Engine (LKE) cluster, utilizing Helm for package management and Nginx Ingress for external access.

![diagram](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/diagram.jpg)


## Infrastructure Setup
- Created a Kubernetes cluster using Linode's Kubernetes Engine (LKE)
- Configured kubectl for cluster management
- Installed Helm package manager

[Screenshot suggestion: Add LKE cluster dashboard showing the active nodes]

## MongoDB Deployment
- Deployed MongoDB using Helm chart with replication enabled
- Configured persistent storage using Linode's Cloud Storage
- Verified MongoDB replication status and health

[Screenshot suggestion: Add terminal output showing MongoDB pods running]

## Mongo Express Setup
- Deployed Mongo Express as a web-based client interface
- Connected Mongo Express to the MongoDB service
- Configured necessary environment variables and secrets

[Screenshot suggestion: Add Mongo Express UI dashboard]

## Ingress Configuration
- Installed and configured Nginx Ingress Controller
- Set up ingress rules for Mongo Express access
- Secured the connection using TLS

[Screenshot suggestion: Add browser showing successful connection to Mongo Express through Ingress]

## Technologies Used
- Kubernetes (LKE)
- Helm
- MongoDB
- Mongo Express
- Nginx Ingress
- Linode Cloud Storage

## Architecture
The deployment consists of:
- LKE Kubernetes cluster
- Replicated MongoDB StatefulSet
- Persistent volumes for data storage
- Mongo Express deployment
- Nginx Ingress Controller

[Screenshot suggestion: Add diagram showing the components and their connections]

## Access Information
- MongoDB Service: `mongodb-service:27017`
- Mongo Express UI: `https://[your-domain]/mongo-express`

## Current Status
✅ MongoDB running with replication
✅ Persistent storage configured
✅ Mongo Express accessible via Ingress
✅ All services healthy and operational

[Screenshot suggestion: Add terminal output showing all pods running]