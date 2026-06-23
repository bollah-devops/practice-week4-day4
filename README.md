# Week 4 Day 4 - AWS EKS Deployment with Helm

## What this project does
Deploys a Flask application to a real AWS EKS cluster using Helm charts,
with staging and production environments isolated in separate Kubernetes namespaces.
Each environment gets its own ConfigMap, Secret, Deployment, and AWS Load Balancer.

## Tech stack
- AWS EKS (Elastic Kubernetes Service)
- eksctl for cluster provisioning
- Helm for Kubernetes package management
- kubectl for cluster interaction
- Docker image: tawa123/k8s-practice:latest

## How to deploy

Step 1 - Create the EKS cluster

    eksctl create cluster \
      --name week4-eks \
      --region us-east-1 \
      --nodegroup-name standard-workers \
      --node-type t3.medium \
      --nodes 2 \
      --nodes-min 1 \
      --nodes-max 3 \
      --managed

Step 2 - Create namespaces

    kubectl create namespace staging
    kubectl create namespace production

Step 3 - Deploy with Helm

    helm install staging helm/flask-chart --values helm/staging-values.yaml
    helm install production helm/flask-chart --values helm/production-values.yaml

Step 4 - Verify

    kubectl get pods -n staging
    kubectl get pods -n production
    kubectl get services -n staging
    kubectl get services -n production

## Environment isolation

| Setting    | Staging                | Production                |
|------------|------------------------|---------------------------|
| ENV_NAME   | staging                | production                |
| VERSION    | 1.1.0                  | 1.0.0                     |
| SECRET_KEY | staging-secret-key-123 | production-secret-key-456 |
| Replicas   | 3                      | 3                         |
| Service    | LoadBalancer           | LoadBalancer              |

## IMPORTANT - always delete the cluster after each session

    eksctl delete cluster --name week4-eks --region us-east-1

EKS bills approximately $0.10 per hour for the control plane regardless of
whether you are actively using it. Always delete after finishing a session.

## Key lessons from this session
- Helm charts built for minikube work unchanged on real EKS clusters
- NodePort services on minikube become LoadBalancer services on EKS
- AWS automatically provisions a real Load Balancer with a DNS hostname
- Namespace isolation ensures staging and production never share secrets or config
