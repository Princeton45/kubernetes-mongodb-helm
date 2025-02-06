# Deploying MongoDB with Mongo Express on Linode Kubernetes Engine

## Project Overview
I successfully deployed a stateful MongoDB service with a web-based UI (Mongo Express) on a Linode Kubernetes Engine (LKE) cluster, utilizing Helm for package management and Nginx Ingress for external access.

![diagram](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/diagram.jpg)


## Infrastructure Setup
- Created a Kubernetes cluster with 2 worker nodes using Linode's Kubernetes Engine (LKE)

![setup](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/setup.png)

- Downloaded the `test-kubeconfig.yaml` file (which contains all the certificate information for the cluster) and added it to my User `KUBECONFIG` environment variable so I can talk to the cluster with `kubectl`

![variable](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/variable.png)

## Technologies Used
- Kubernetes (LKE)
- Helm
- MongoDB
- Mongo Express
- Nginx Ingress
- Linode Cloud Storage

## MongoDB Deployment
- Deployed MongoDB using Helm chart with replication enabled & Linode persistent cloud storage

First, I needed to add the bitnami crepository that contains the MongoDB Helm chart

`helm repo add bitnami https://charts.bitnami.com/bitnami`

Then I created `helm-mongodb.yaml` values file to override default configurations for the MongoDB Helm chart to match specifications for the project

```yaml
architecture: replicaset
replicaCount: 3
persistence:
  storageClass: "linode-block-storage"
auth:
  rootPassword: secret-root-pwd
```
(including `architecture: replicaset` automatically makes the helm chart deployment run as a statefulset)

Then I ran the command below to install the chart

`helm install mongodb --values .\helm-mongodb.yaml bitnami/mongodb`

![pods-up](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/pods-up.png)

There is now 3 PVCs in Linode now, which was created from the Helm Chart

![pvc-mongodb](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/pvc-mongodb.png)


## Mongo Express Setup
- Deployed Mongo Express as a web-based client interface with `helm-mongo-express.yaml` file

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-express
  labels:
    app: mongo-express
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-express
  template:
    metadata:
      labels:
        app: mongo-express
    spec:
      containers:
      - name: mongo-express
        image: mongo-express
        ports:
        - containerPort: 8081
        env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          value: root
        - name: ME_CONFIG_MONGODB_SERVER
          value: mongodb-0.mongodb-headless
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD 
          valueFrom:
            secretKeyRef:
              name: mongodb
              key: mongodb-root-password
---
apiVersion: v1
kind: Service
metadata:
  name: mongo-express-service
spec:
  selector:
    app: mongo-express
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
```

- Connected Mongo Express to the MongoDB service

This section from the `helm-mongo-express.yaml` file connects the Mongo Express pod app to the MongoDB Databases

```yaml
 env:
        - name: ME_CONFIG_MONGODB_ADMINUSERNAME
          value: root
        - name: ME_CONFIG_MONGODB_SERVER
          value: mongodb-0.mongodb-headless
        - name: ME_CONFIG_MONGODB_ADMINPASSWORD 
          valueFrom:
            secretKeyRef:
              name: mongodb
              key: mongodb-root-password
```

![mongo-express](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/express.png)


## Ingress Configuration
- Installed and configured Nginx Ingress Controller

`helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`
`helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true`

This `--set controller.publishService.enabled=true` makes sures that there is a public IP allocated for the ingress address to use with NGINX

NGINX Ingress controller is now deployed

![nginx](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/nginx-deployment.png)

This NodeBalancer in Linode was automatically created when I deployed the Ingress controller with Helm. This NodeBalancer will now become the entrypoint into the cluster for external web traffic with its public IP of `45.79.245.223`

![nodebalancer](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/nodebalancer.png)

- Set up ingress rules for Mongo Express access

`helm-ingress.yaml`
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: mongo-express
spec:
  rules:
  - host: 45-79-245-223.ip.linodeusercontent.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: mongo-express-service
            port:
              number: 8081
```

The `host` comes from the Host Name of the NodeBalancer

![host](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/host.png)

Now going to `45-79-245-223.ip.linodeusercontent.com` takes me to the Mongo Express UI which is connected to the MongoDB Databases

![success](https://github.com/Princeton45/kubernetes-mongodb-helm/blob/main/images/success.png)
