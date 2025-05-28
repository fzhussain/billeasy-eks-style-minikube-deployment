
# DevOps Technical Task: Simulated EKS-Style Secure Deployment Using Minikube

Overview
This project simulates a production-style microservices deployment on a local Kubernetes cluster using Minikube. The goal is to reflect AWS EKS patterns with a strong emphasis on infrastructure security, observability, IAM controls, and incident response.

The solution demonstrates:
- Secure deployment of containerized services
- IAM simulation using MinIO
- Network segmentation and RBAC
- Observability using Prometheus & Grafana
- Incident simulation and response
- Policy enforcement using Kyverno

Objectives
- Deploy a secure microservice architecture
- Simulate IAM controls
- Respond to a security incident involving log leaks and unauthorized network access
- Visualize system health and traffic using observability tools
- Recover gracefully from simulated failures

## Setup Instructions

Install the following tools:
- minikube
- kubectl
- helm
- docker
- kyverno

## Minikube Cluster Initialization

```bash
minikube start --driver=virtualbox --cpus=4 --memory=8192 --addons=ingress,metrics-server --cni calico --no-vtx-check
```
![Project setup](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/1.%20minikube%20start.png)

#### Cluster Verification
```bash
kubectl get nodes
```

![kubectl get nodes](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/2.%20kubectl%20get%20all.png)


## Clone the repository
```bash
git clone https://github.com/fzhussain/billeasy-eks-style-minikube-deployment.git
```
![kubectl get nodes](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/3.%20Clone%20and%20project%20structure.png)

```
.
├── README.md
├── kustomize
│   ├── kustomization.yaml
│   ├── clusterpolicies
│   │   └── prevent-auth-header-logging.yaml
│   ├── infra
│   │   └── minio
│   │       ├── minio-deployment.yaml
│   │       └── minio-service.yaml
│   ├── microservice
│   │   ├── auth-service
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   └── kustomization.yaml
│   │   └── data-service
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── kustomization.yaml
│   ├── policies
│   │   ├── networkpolicy.yaml
│   │   ├── kyverno-policy.yaml
│   │   └── kustomization.yaml
│   ├── base
│   │   ├── namespaces
│   │   │   ├── application-namespace.yaml
│   │   │   ├── minio-namespace.yaml
│   │   │   └── system-namespace.yaml
│   │   ├── network-policies
│   │   │   ├── auth-to-data-only.yaml
│   │   │   ├── default-deny-all.yaml
│   │   │   └── minio-access-policy.yaml
│   │   ├── rbac
│   │   │   ├── minio-secret-reader-role.yaml
│   │   │   └── minio-secret-reader-role-binding.yaml
│   │   ├── secrets
│   │   │   └── minio-credentials-system-ns.yaml
│   │   └── serviceaccounts
│   │       ├── auth-service-service-account.yaml
│   │       └── data-service-service-account.yaml
├── overlays
│   ├── dev
│   │   ├── auth-service
│   │   │   ├── replica-patch.yaml
│   │   │   └── service-type-patch.yaml
│   │   ├── data-service
│   │   │   ├── replica-patch.yaml
│   │   │   └── service-type-patch.yaml
│   │   ├── gateway
│   │   │   └── replica-patch.yaml
│   │   └── kustomization.yaml
│   └── prod
│       ├── auth-service
│       │   ├── replica-patch.yaml
│       │   ├── service-account-patch.yaml
│       │   ├── logger-patch.yaml
│       │   └── debug-sidecar-patch.yaml
│       ├── data-service
│       │   ├── replica-patch.yaml
│       │   └── debug-sidecar-patch.yaml
│       ├── gateway
│       │   └── replica-patch.yaml
│       └── kustomization.yaml
```

## Apply the kustomize overlay to create resources 
```bash
kubectl apply -k kustomize/overlays/prod/
```
![kubectl apply -k](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/4.%20kubectl%20apply%20-k.png)

### Verify your resource creations

```bash
kubectl get all -n application
kubectl get all -n system
kubectl get ingress -n application
```
![kubectl get all -n application](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/4.%20kubectl%20get%20all%20-n%20system%2C%20application.png)
![kubectl get ingress -n application](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/5.%20kubectl%20get%20ingress.png)

```bash
sudo vim /etc/hosts
```
#### Make sure you add proper minikube IP to /etc/hosts
![sudo vim /etc/hosts](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/6.%20Edit%20the%20etc%20hosts.png)
