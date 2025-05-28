
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
Note: In my case it is: **192.168.59.112**

![sudo vim /etc/hosts](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/6.%20Edit%20the%20etc%20hosts.png)


## Part 1: Microservice Stack - Testing

### 1. Accessing Gateway via Ingress:
Open in browser: [http://faraz.billeasy.com/](http://faraz.billeasy.com/)
![gateway ingress](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/7.%20Testing%20ingress-gateway.png)

### 2. Accessing Auth service via Port forwarding:
Since, the service is in Cluster IP mode, we will access via port forwarding:
```bash
kubectl port-forward svc/prod-faraz-auth-service-svc 8081:80 -n system
```
Open in browser: [http://localhost:8081/](http://localhost:8081/)
![auth service](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/8.%20Testing%20auth-service.png)

### 3. Accessing Data service via Port forwarding:
Since, the service is in Cluster IP mode, we will access via port forwarding:
```bash
kubectl port-forward svc/prod-faraz-data-service-svc 8082:5678 -n system
```
Open in browser: [http://localhost:8082/](http://localhost:8082/)
![auth service](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/9.%20Testing%20data-service.png)

### Hence we demonstated the following:
- ### Health Monitoring
    - Liveness probes configured to detect and restart unhealthy containers
    - Readiness probes implemented to ensure traffic only reaches ready pods

- ### Resource Management
    - Proper resource requests and limits defined for all containers
    - CPU and memory requirements specified to ensure stable operation

- ### Networking Configuration
    - Gateway service exposed publicly via:
        - Ingress controller with appropriate routing rules
        - External access properly configured

    - Internal-only services:
        - auth-service restricted to cluster-internal access
        - data-service restricted to cluster-internal access

## Part 2: Simulate IAM with MinIO

We already have our minio deployment and service created when we applied the kustomize/overlay/prod 
- Verify once again:
```bash
kubectl get all -n minio-storage
```
![kubectl get all -n minio-storage](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/10.kubectl%20get%20all%20-n%20minio-storage.png)

- For Minio UI:
```bash
kubectl port-forward service/prod-faraz-minio -n minio-storage 9001:9001
```
Open in browser: [http://localhost:9001/](http://localhost:9001/)
![minio UI](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/11.%20Minio%20UI%20localhost.9001.png)

- For Minio CLI:
```bash
kubectl port-forward service/prod-faraz-minio -n minio-storage 9000:9000
```

### Verify RBAC
```bash
kubectl get secrets -n system
kubectl get sa -n system
```
Test for data service service account:
```bash
kubectl auth can-i get secrets/prod-faraz-minio-credentials --as=system:serviceaccount:system:prod-faraz-data-service-sa -n system
```
Test for auth service service account:
```bash
kubectl auth can-i get secrets/prod-faraz-minio-credentials --as=system:serviceaccount:system:prod-faraz-auth-service-sa -n system
```

![verify RBAC](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/13.%20Verifying%20RBAC.png)

### Creating bucket from UI and testing access 
- Create bucket
![create bucket](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/12.%20Create%20a%20bucket%20from%20UI.png)

- Now, we will simulate a scenario where maliciously auth service via a patch is able to mount environment variables:
```bash
kubectl describe pod -n system prod-faraz-auth-service-deploy-5dd5c7fcb7-6qnxm
```

![auth service credentials mount](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/14.%20auth-service%20mounted%20minio%20credentials.png)

- Data service has it's minio credentials mounted:
```bash
kubectl describe pod -n system prod-faraz-data-service-deploy-65b58dcfb-p8jfq
```

![data service credentials mount](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/15.%20data-service%20mounted%20minio%20credentials.png)

- Temporarily disable the egress, so auth service can fetch curl and minio-client:
```bash
kubectl get netpol -A
kubectl delete netpol prod-allow-auth-to-data-only -n system
```
![auth service egress disable](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/16.%20Temporarily%20disabling%20egress%20for%20auth-service.png)


- Installing minio client on auth and data service pods:
```bash
kubectl exec -it prod-faraz-auth-service-deploy-5dd5c7fcb7-6qnxm -n system -c debug-tools -- sh -c "apk add curl && wget https://dl.min.io/client/mc/release/linux-amd64/mc && chmod +x mc"
```
```bash
kubectl exec -it prod-faraz-data-service-deploy-65b58dcfb-p8jfq -n system -c debug-tools -- sh -c "apk add curl && wget https://dl.min.io/client/mc/release/linux-amd64/mc && chmod +x mc"
```

![Install minio client on both the pods](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/17.%20Installing%20mc%20client%20and%20curl%20to%20auth%20and%20data%20service%20pods.png)


- As you can see that we have deliberatly mounted the MINIO credentials:
```bash
kubectl exec -it prod-faraz-auth-service-deploy-5dd5c7fcb7-6qnxm -n system -c debug-tools -- sh -c "env | grep MINIO_"

kubectl exec -it prod-faraz-data-service-deploy-65b58dcfb-p8jfq -n system -c debug-tools -- sh -c "env | grep MINIO_"
```

![minio cred mounts on pods](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/18.Minio%20credentials%20access%20to%20auth%20and%20data%20service.png)

- Testing if auth-service and data-service is able to access minio and list the buckets:

    - If you try to access Minio from data-service, we will be successfull:

```bash
kubectl exec -it prod-faraz-data-service-deploy-65b58dcfb-p8jfq -n system -c debug-tools -- sh -c "./mc alias set faraz-minio http://prod-faraz-minio.minio-storage.svc.cluster.local:9000 \$MINIO_ACCESS_KEY \$MINIO_SECRET_KEY"

kubectl exec -it prod-faraz-data-service-deploy-65b58dcfb-p8jfq -n system -c debug-tools -- sh -c "./mc ls faraz-minio"

```

-  But now if you try to access via auth-service, it will fail:

```bash
 kubectl exec -it prod-faraz-auth-service-deploy-5dd5c7fcb7-6qnxm -n system -c debug-tools -- sh -c "./mc alias set faraz-minio http://prod-faraz-minio.minio-storage.svc.cluster.local:9000 \$MINIO_ACCESS_KEY \$MINIO_SECRET_KEY"
```

![minio cred mounts on pods](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/19.%20Network%20policy%20blocked%20auth-service%20to%20access%20minio.png)

This is because the network policy (prod/dev)allow-only-data-service allows only data service to communicate to minio pods.

#### Hence, proved that even if Auth-service is misconfigured, it is unable to communicate to Minio and only data-service can access minio and list the buckets.

NOTE for future improvements: 
- We can improve security by creating users (non-admin/root) and access via user/service account.
- We should implement bucket policies. (Current Requirement)
- Implement only data-service to mount credentials.


## Part 3. Security Incident Simulation

<Story of P1>

- Listing the pods with their labels:

```bash
kubectl get pods -n system -o wide --show-labels
```

![Listing pods with labels](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/20.%20system%20ns%20pods%20show%20labels.png)

- Simulating auth leaks:

```bash
curl -H "Authorization: Bearer faraz-secret-token-123" http://localhost:8081/headers

curl -H "Authorization: Bearer faraz-token-098" http://localhost:8081/get
```
```bash
kubectl logs -l app=faraz-auth-service -n system | grep Bearer
```

![Auth leak demo](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/21.%20Auth%20header%20leak%20in%20logs.png)

    - To solve this, we can manually remove the Auth headers from deployment:
![Auth leak manual solved](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/23.%20Manually%20solve%20auth%20header%20leaks.png)

    - And after re-applying the changes, this auth leak was solved:
![Auth leak manual solved test](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/24.%20Manually%20redacting%20auth%20header%20leaks.png)


- Also, we noticed that auth-service can communicate to External services

```bash
kubectl exec -it prod-faraz-auth-service-deploy-669977d44d-5srg8 -c debug-tools -n system -- sh -c "wget -qO- http://prod-faraz-data-service-svc.system.svc.cluster.local:5678"

kubectl exec -it prod-faraz-auth-service-deploy-669977d44d-5srg8 -c debug-tools -n system -- sh -c "wget -qO- https://www.google.com" 
```
![No egress configured](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/22.%20Auth%20service%20egress%20not%20configured.png)

Hence, we enforce network policy to only allow auth-service communication to data service:

```bash
kubectl get netpol -A
```
If not active then:
```bash
kubectl apply -k kustomize/overlays/prod/
```
![egress configured](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/25.%20After%20configuring%20egress%20for%20auth-service.png)

Now the external access to auth-service is blocked using Network Policies and auth-service should be able to communicate to data-service only.

### Using Kyverno to block/detect if deployment logs Authorization headers
- Install Kyverno:
```bash
helm repo add kyverno https://kyverno.github.io/kyverno/

helm repo update

helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```
![Install kyverno](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/26.%20Install%20kyverno.png)

- Verify install:
![verify kyverno install](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/27.%20Verify%20install.png)

- Create and apply the cluster policy:

```bash
kubectl apply -f kustomize/base/clusterpolicies/prevent-auth-header-logging.yaml

kubectl get clusterpolicy -o wide
```

```bash
kubectl apply -k kustomize/overlays/prod/
```

![cluster policy applied and tested](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/28.%20Auth-leak-kyverno-policy.png)

#### Hence, the kyverno policy which we created successfully blocks the creation of deployment if the deployment contains %({Authorization}i)s

- Reapplying after fixing:

![Removed leaks](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/30.%20Testing%20after%20fixing%20auth-leak.png)

### We sucessfully demonstated security leaks and fixed the incident.
