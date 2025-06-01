
# DevOps Technical Task: Simulated EKS-Style Secure Deployment Using Minikube

Overview:

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

Video for reference:
<div style="position: relative; padding-bottom: 56.19146722164412%; height: 0;"><iframe src="https://www.loom.com/embed/5c9d37b5438149049fecd13535f7362e?sid=d582c83b-e86d-4561-885d-0d02c8e9ad32" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe></div>

## Setup Instructions

Install the following tools:
- minikube
- kubectl
- helm
- docker
- kyverno

## Minikube Cluster Initialization
### Start Minikube with appropriate add-ons

```bash
minikube start --driver=virtualbox --cpus=4 --memory=8192 --addons=ingress,metrics-server --cni calico --no-vtx-check
```
![Project setup](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/1.%20minikube%20start.png)

#### Cluster Verification
```bash
kubectl get nodes
```

![kubectl get nodes](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/2.%20kubectl%20get%20all.png)

Why:
- We mimic EKS behavior locally, including:
    - Ingress for external routing
    - Metrics Server for pod-level metrics
    - Calico CNI for enforcing Network Policies

How:
- Allocate enough CPU/RAM for observability and app services.
- Ensure Calico enables network isolation via NetworkPolicies.

## Clone the repository
```bash
git clone https://github.com/fzhussain/billeasy-eks-style-minikube-deployment.git
cd billeasy-eks-style-minikube-deployment/
```
![kubectl get nodes](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/3.%20Clone%20and%20project%20structure.png)

### Project structure:
```
.
├── README.md
└── kustomize
    ├── base                             
    │   ├── kustomization.yaml            
    │   ├── microservices                
    │   │   ├── auth-service
    │   │   │   ├── deployment.yaml       
    │   │   │   ├── service.yaml          
    │   │   │   └── kustomization.yaml    
    │   │   ├── data-service
    │   │   │   ├── deployment.yaml      
    │   │   │   ├── service.yaml         
    │   │   │   └── kustomization.yaml    
    │   │   └── gateway
    │   │       ├── deployment.yaml      
    │   │       ├── service.yaml         
    │   │       ├── ingress.yaml          
    │   │       └── kustomization.yaml
    |   |
    │   ├── infra                         
    │   │   └── minio                     
    │   │       ├── minio-deployment.yaml       
    │   │       └── minio-service.yaml          
    │   │                  
    │   ├── network-policies                      
    │   │   ├── auth-to-data-only.yaml             
    │   │   └── minio-access-policy.yaml
    |   |
    │   ├── cluster-policies                                   
    │   │   └── prevent-auth-header-logging.yaml
    |   |
    │   ├── namespaces                  
    │   │   ├── application-namespace.yaml       
    │   │   ├── minio-namespace.yaml     
    │   │   └── system-namespace.yaml
    |   |
    │   ├── rbac                       
    │   │   ├── minio-secret-reader-role.yaml      
    │   │   └── minio-secret-reader-rolebinding.yaml  
    │   │   
    │   ├── secrets                       
    │   │   └── minio-credentials.yaml    
    │   │    
    │   └── serviceaccounts               
    │       ├── auth-service-sa.yaml     
    │       └── data-service-sa.yaml       
    |
    └── overlays
        ├── dev
        │   ├── auth-service
        │   │   ├── replica-patch.yaml
        │   │   └── service-type-patch.yaml
        |   |
        │   ├── data-service
        │   │   ├── replica-patch.yaml
        │   │   └── service-type-patch.yaml
        |   |
        │   ├── gateway
        │   │   └── replica-patch.yaml
        |   |
        │   └── kustomization.yaml
        └── prod
            ├── auth-service
            │   ├── replica-patch.yaml
            │   ├── service-account-patch.yaml
            │   ├── logger-patch.yaml
            │   └── debug-sidecar-patch.yaml
            |    
            ├── data-service
            │   ├── replica-patch.yaml
            │   └── debug-sidecar-patch.yaml
            |
            ├── gateway
            │   └── replica-patch.yaml
            |
            └── kustomization.yaml
```

## Apply the kustomize overlay to create resources 

### Apply secure production deployment
```bash
kubectl apply -k kustomize/overlays/prod/
```
![kubectl apply -k](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/4.%20kubectl%20apply%20-k.png)

- Why:
    - The prod overlay applies:
        - Secure namespace segmentation
        -  Resource constraints
        -  Sidecar debug tools for observability
        -  Patch files for service accounts & ingress routing

### Verify your resource creations

#### How to Verify:
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
#### Ensure /etc/hosts maps your Minikube IP to faraz.billeasy.com.
Note: In my case it is: **192.168.59.112**

![sudo vim /etc/hosts](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/6.%20Edit%20the%20etc%20hosts.png)


## Part 1: Microservice Stack - Testing

### 1. Gateway via Ingress
What: Public entrypoint to the services
Why: Mimics ALB/Ingress on EKS
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
- Readiness/Liveness probes ensure healthy containers.
- ClusterIP exposure restricts internal-only access.
- Ingress + RBAC + Namespace isolation reflects real-world cloud environments.

## Part 2: Simulate IAM with MinIO
### Use MinIO to mimic S3-based IAM behaviors
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

- MinIO allows us to simulate cloud IAM + bucket policies.
- RBAC ensures only the data-service can access secrets.

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

- auth-service fails the check
- data-service passes (and can access MinIO)

#### Simulated Attack:
- Deliberately expose credentials to auth-service.
- Observe failure to connect due to NetworkPolicy blocking.
- Proves defense in depth: even if secrets leak, network barriers save us.

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

- Simulate logging of Authorization headers:
    - This tests if confidential headers get logged accidentally.

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

### Policy Enforcement using Kyverno
- Install Kyverno:
```bash
helm repo add kyverno https://kyverno.github.io/kyverno/

helm repo update

helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```
![Install kyverno](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/26.%20Install%20kyverno.png)

- Verify install:
  
![verify kyverno install](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/27.%20Verify%20install.png)

- Enforce that Authorization headers cannot be logged:
    - Prevent developers from pushing insecure code
    - Act as a compliance-as-code layer

```bash
kubectl apply -f kustomize/base/clusterpolicies/prevent-auth-header-logging.yaml

kubectl get clusterpolicy -o wide
```

```bash
kubectl apply -k kustomize/overlays/prod/
```

![cluster policy applied and tested](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/28.%20Auth-leak-kyverno-policy.png)
- Rejected if deployment logs %({Authorization}i)s
#### Hence, the kyverno policy which we created successfully blocks the creation of deployment if the deployment contains %({Authorization}i)s

- Reapplying after fixing:

![Removed leaks](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/30.%20Testing%20after%20fixing%20auth-leak.png)

### We sucessfully demonstated security leaks and fixed the incident.


## Part 4: Observability with Prometheus & Grafana

- Add and update the Prometheus Helm chart repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring --create-namespace 
```

![add prom grafana](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/31.%20helm%20add%20prom.png)

![install prom grafana](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/31.%20helm%20install%20prom.png)

- Verfiy your install:
    - Check all resources deployed in the monitoring namespace:
       
```bash
kubectl get all  -n monitoring
```
![verify install](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/32.%20Verify%20monitoring%20install.png)

- Port forwarding to access Prometheus and Grafana via UI"
For Prometheus:
```bash
kubectl port-forward service/prometheus-operated 9090 -n monitoring
```
Open in browser: [http://localhost:9090/query](http://localhost:9090/query)

![open prom on web](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/33.%20Port%20forward%20and%20open%20prom%20on%20web.png)

For Grafana:
```bash
kubectl port-forward service/monitoring-grafana -n monitoring 3000:80
```

- Enter Default credentials:
    - username: admin
    - password: prom-operator

Open in browser: [http://localhost:3000/](http://localhost:3000/)
![open grafana on web](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/34.%20Port%20forward%20and%20open%20grafana%20on%20web.png)

- Verify your Grafana is connected to your data source (Prometheus)

![verify grafana to prom](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/35.%20Verify%20prom%20connection%20with%20grafana.png)


#### Creating Panels and dashboard:
- Use Prometheus queries to create insightful panels:
  
##### 1. POD CPU Usage:
![CPU Usage](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/36.%20POD%20CPU%20usage.png)

##### 2. POD memory Usage:
![Memory Usage](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/37.%20POD%20memory%20usage.png)

##### 3. Pod Restarts:
- Simulating Restart:
![Restart Usage](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/39.%20Simulating%20restart.png)
- Panel:
![Restart Panel Demo 1](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/40.%20Corresponding%20restart%20dashboard.png)

![Restart Panel Demo 2](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/41.%20Corresponding%20restart.png)

##### Complete Dashboard Sample:
![Dashboard Usage](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/38.%20Dashboard%20CPU%20Memory%20Restarts.png)

- Future Enhancements
    - Monitor HTTP request rates and error counts
    - Integrate Prometheus Alertmanager for:
        - Custom alert rules
        - Email or Slack notifications  

## Part 5: Failure Simulation

#### Simulate a partial failure:
- When a pod is deleted, the ReplicaSet ensures the pod is automatically recreated, demonstrating Kubernetes' self-healing capability.

![Auto healing](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/42.%20Auth%20heal%20simulation.png)

##### As you can see, even when the pod was deleted, the pod is recovered via ReplicaSet

##### A visible spike in CPU and memory usage, can be observed in the Grafana dashboard after pod deletion.
![Grafana spike](https://github.com/fzhussain/billeasy-eks-style-minikube-deployment/blob/main/Screenshots%20for%20Readme.md/43.%20Spike%20on%20graph%20after%20deletion.png)



---

**Closing Note:**  
I’ve put this guide together based on my current understanding of **Kubernetes**. It’s a work in progress, and I’ll keep refining it as I learn more.😇
