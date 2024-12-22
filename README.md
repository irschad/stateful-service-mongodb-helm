# Deploy MongoDB Stateful Service on Kubernetes Using Helm

This project demonstrates how to deploy a replicated MongoDB StatefulSet on Amazon Elastic Kubernetes Service (EKS) using Helm, with data persistence backed by AWS cloud storage. It also includes the deployment of Mongo Express UI for interacting with MongoDB and configuring an NGINX ingress to expose the UI via a web browser.

## Project Overview

- **Managed Kubernetes Cluster (EKS)**: Set up a managed Kubernetes cluster on AWS using Elastic Kubernetes Service (EKS).
- **Replicated MongoDB Deployment**: Deploy a MongoDB service with a replica set using Helm for high availability.
- **Data Persistence**: Configure persistent storage for MongoDB using AWS Elastic Block Store (EBS) volumes.
- **Mongo Express UI**: Deploy Mongo Express UI for database interaction via the browser.
- **NGINX Ingress Controller**: Set up NGINX ingress to expose the Mongo Express UI.

## Files Used

- `helm-mongodb.yaml`: Helm chart configuration for deploying MongoDB with persistence and authentication.
- `sc.yaml`: Defines an AWS EBS-backed StorageClass for persistent volumes.
- `mongo-express.yaml`: Defines the deployment and service for Mongo Express UI.
- `helm-ingress.yaml`: Configures NGINX ingress to expose the Mongo Express UI.

## Prerequisites

- AWS account and EKS CLI tools installed
- Helm CLI installed and configured with the EKS cluster
- Kubernetes `kubectl` configured to access the EKS cluster

## Steps to Deploy

1. **Create an EKS Cluster**:
   - Use the `eksctl` command to create a managed EKS cluster:
     ```bash
     eksctl create cluster --name mongo-cluster --region <region> --nodegroup-name mongo-nodes --nodes 3 --nodes-min 1 --nodes-max 3 --managed
     ```

2. **Install AWS EBS CSI Driver**:
   - Add the AWS EBS CSI driver repository:
     ```bash
     helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
     helm repo update
     helm install aws-ebs-csi-driver aws-ebs-csi-driver/aws-ebs-csi-driver --namespace kube-system --set enableVolumeScheduling=true --set enableVolumeResizing=true --set enableVolumeSnapshot=true
     ```

3. **Create Storage Using StorageClass**:
   - Before deploying MongoDB, apply the `sc.yaml` file to create the AWS EBS-backed StorageClass for persistent volumes:
     ```bash
     kubectl apply -f sc.yaml
     ```

4. **Deploy MongoDB with Helm**:
   - Install the MongoDB Helm chart with:
     ```bash
     helm install mongodb -f helm-mongodb.yaml stable/mongodb
     ```

5. **Deploy Mongo Express**:
   - Apply the Mongo Express deployment using:
     ```bash
     kubectl apply -f mongo-express.yaml
     ```

6. **Install NGINX Ingress Controller**:
   - Add the NGINX ingress repository and install the controller:
     ```bash
     helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
     helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.publishService.enabled=true --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-scheme"=internet-facing
     kubectl get pods
     ```

   - If needed, upgrade the NGINX ingress controller to use an internet-facing AWS load balancer:
     ```bash
     helm upgrade nginx-ingress ingress-nginx/ingress-nginx --set controller.service.annotations."service\.beta\.kubernetes\.io/aws-load-balancer-scheme"=internet-facing
     ```

7. **Configure NGINX Ingress**:
   - Apply the ingress configuration using:
     ```bash
     kubectl apply -f helm-ingress.yaml
     ```

## Accessing the Mongo Express UI

Once the NGINX ingress is configured, access the Mongo Express UI via the external URL provided by the NGINX ingress.

## Conclusion

This project demonstrates how to deploy MongoDB with persistent storage on EKS, and how to set up a user-friendly Mongo Express UI, all exposed via an NGINX ingress for easy access from a browser.
