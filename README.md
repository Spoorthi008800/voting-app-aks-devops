# 🗳️ End-to-End DevOps Pipeline on Azure Kubernetes Service (AKS)

**Terraform · Azure DevOps · Argo CD · Prometheus · Grafana · Docker · Kubernetes**

---

## 🔍 What This Project Demonstrates

A fully automated, production-grade DevOps pipeline built from scratch on Microsoft Azure — covering infrastructure provisioning, CI/CD automation, GitOps delivery, container orchestration, and live monitoring across **12 running pods** and **5 application services**.

This is not a deployment demo. It includes real infrastructure decisions, multi-layer debugging, and documented failure recovery — the kind of work that happens in production environments.

**Base application:** [Example Voting App](https://github.com/dockersamples/example-voting-app) by Docker Samples (open-source microservices demo used as the application layer).

---

## 🏗️ Architecture Overview

![Project Workflow](docs/screenshots/Project-workflow.PNG)

| Layer | Technology | Purpose |
|---|---|---|
| Infrastructure | Terraform + Azure | Provision AKS, ACR, VNet, Resource Group |
| CI/CD | Azure DevOps Pipelines | Build, tag, push Docker images to ACR |
| GitOps | Argo CD | Sync manifests from GitHub → AKS automatically |
| Orchestration | Kubernetes (AKS) | Run and manage all 12 pods across 5 services |
| Monitoring | Prometheus + Grafana | System supervision, dashboards, metrics visualisation |
| Security | RBAC, Service Principals, Registry Secrets | Secure access across all layers |

---

## ⚙️ Infrastructure as Code — Terraform

Terraform provisions the entire Azure environment from scratch:

- Azure Resource Group
- Azure Kubernetes Service (AKS) cluster
- Azure Container Registry (ACR)
- Virtual Network (VNet) and subnets
- Remote state stored securely in Azure Storage Account with state locking

![Terraform Plan](docs/screenshots/Terraform-plan.png)
![Terraform Plan 2](docs/screenshots/Terraform-plan2.png)
![Azure Resources](docs/screenshots/Azureresources.png)
![Azure Network Resources](docs/screenshots/Azurenetworkresources.png)

---

## 🔁 CI/CD Pipeline — Azure DevOps

Three independent YAML pipelines — one per microservice (Vote, Result, Worker):

1. Triggered on code commit to GitHub
2. Builds Docker image for the microservice
3. Pushes tagged image to Azure Container Registry (ACR)
4. Updates Kubernetes manifests with new image tag

![Pipelines](docs/screenshots/Pipelines.PNG)
![Azure Repo](docs/screenshots/Azure-repo.png)
![Azure Repo Terraform](docs/screenshots/Azure-repo-tf.png)
![ACR Vote App](docs/screenshots/ACR-voteapp.png)

---

## 🚀 GitOps Delivery — Argo CD

Argo CD continuously monitors the GitHub repository and syncs any manifest changes directly to the AKS cluster — no manual deployments.

- Declarative, Git-based deployment model
- Drift detection: Argo CD shows `OutOfSync` status with a full diff between desired (Git) and live (cluster) state
- Rollback: executed via Argo CD history with immediate health verification

---

## 🐳 Kubernetes — Running State

All 12 pods healthy and running across 5 application services + Argo CD components:

```
NAME                                          READY   STATUS    RESTARTS
argocd-application-controller-0               1/1     Running   0
argocd-applicationset-controller-xxxx         1/1     Running   0
argocd-dex-server-xxxx                        1/1     Running   2
argocd-notifications-controller-xxxx          1/1     Running   0
argocd-redis-xxxx                             1/1     Running   0
argocd-repo-server-xxxx                       1/1     Running   0
argocd-server-xxxx                            1/1     Running   0
db-xxxx                                       1/1     Running   0
metrics-server-xxxx                           1/1     Running   0
redis-xxxx                                    1/1     Running   0
result-xxxx                                   1/1     Running   0
vote-xxxx                                     1/1     Running   0
worker-xxxx                                   1/1     Running   0
```

![Kubernetes Pods](docs/screenshots/K8s-pods.png)
![Kubernetes Services Output](docs/screenshots/K8s-svc-output.png)
![Vote Deployment](docs/screenshots/K8s-votedeployment.png)
![Load Balancers](docs/screenshots/Az-LoadBalancers.png)
![Inbound Rules](docs/screenshots/Az-inboundrules.png)

---

## 📊 System Supervision — Prometheus & Grafana

Prometheus and Grafana deployed on AKS for live system supervision across all running workloads.

### ✅ What's Working

**Prometheus targets UP** — `metrics-server` and `postgres-exporter` ServiceMonitors successfully scraping at 28–30ms:

![Prometheus Targets Up](docs/screenshots/prometheuspodsup.png)

**Grafana live dashboard** — `process_cpu_seconds_total` visualised in real time from running containers:

![Grafana CPU Dashboard](docs/screenshots/grafanavisual.png)

### 🔍 Troubleshooting in Progress

**Prometheus targets DOWN** — `vote-service-monitor` returning `connection refused` on ports 8080/8081. Root cause traced to ServiceMonitor label selector mismatch and missing RBAC permissions for the vote app's metrics endpoint. Fix documented in Future Enhancements:

![Prometheus Targets Down](docs/screenshots/prometheuspodsdown.png)

This is real operational debugging — identifying exactly which targets are failing, why, and what needs to change. The fact that some targets are UP confirms the Prometheus stack itself is healthy; the issue is isolated to application-level ServiceMonitor configuration.

---

## 🔐 Security & Access Control

- Azure Service Principals scoped to CI/CD pipeline operations
- Docker registry secrets for AKS → ACR image pull authentication
- Kubernetes RBAC policies enforcing least-privilege access
- Terraform remote state secured in Azure Storage with restricted access

---

## 🐛 Real Troubleshooting — What Actually Happened

This project involved real infrastructure debugging, not just guided steps:

**Azure Public IP Exhaustion**
Result page failed to expose publicly after multiple deployments. Root cause: orphaned public IPs from deleted resources were consuming the subscription allocation. Fix: audited all public IPs in the resource group, deleted orphaned allocations, then traced a secondary issue to health probe misconfiguration and frontend IP settings in the load balancer — corrected both to restore service exposure.

**Kubernetes Pod Failures**
Diagnosed ImagePull errors (ACR authentication), crashlooping containers, and YAML indentation errors in pipeline configs using `kubectl logs` and `kubectl describe` — developed a systematic multi-layer troubleshooting approach.

**Prometheus Scraping Issue**
Node and pod metrics operational. Application-level metrics blocked by `connection refused` — traced to ServiceMonitor configuration and RBAC permissions. Documented for resolution in next iteration.

---

## 🔄 End-to-End Workflow

Code commit → GitHub
       ↓
Azure DevOps Pipeline triggers
       ↓
Docker image built + pushed to ACR
       ↓
Kubernetes manifests updated with new image tag
       ↓
Argo CD detects change → syncs to AKS
       ↓
Prometheus + Grafana monitor running workloads


![Update Shell Script](docs/screenshots/Update-shell-script.png)

---

## 🛠️ Tools & Technologies

| Category | Tool |
|---|---|
| Cloud | Microsoft Azure (AKS, ACR, VNet, Load Balancers) |
| IaC | Terraform |
| CI/CD | Azure DevOps YAML Pipelines |
| GitOps | Argo CD |
| Containers | Docker, Kubernetes, Helm |
| Monitoring | Prometheus, Grafana |
| Security | RBAC, Service Principals, Registry Secrets, Key Vault |
| SCM | GitHub |

---

## ❓ FAQ

**Q: How do you detect configuration drift?**
Argo CD shows `OutOfSync` status with a full diff between the desired state in Git and the live cluster state.

**Q: How do you handle a failed deployment?**
Investigate using Argo CD health and diff views, check deployment history, execute rollback to last stable version, verify healthy pod status.

**Q: How are images tagged and promoted?**
CI pipeline builds and tags each image with the pipeline run ID, pushes to ACR, and updates the Kubernetes manifest with the new tag — Argo CD picks up the change automatically.

**Q: How is Terraform state secured?**
Remote backend in Azure Storage Account with restricted IAM access and state locking to prevent concurrent modifications.

**Q: Why is some monitoring partial?**
Node and pod metrics are fully operational. Application-level metrics scraping is pending a ServiceMonitor + RBAC fix — documented in Future Enhancements.

---

## 🔮 Future Enhancements

- [ ] Fix Prometheus `connection refused` for application-level metrics (ServiceMonitor + RBAC)
- [ ] Implement Prometheus Alertmanager for automated incident alerting
- [ ] Extend Grafana dashboards with application-level performance metrics
- [ ] Add Helm chart packaging for simplified deployment
- [ ] Implement network policies for pod-level traffic isolation

---

## 📚 References

- Base application: [dockersamples/example-voting-app](https://github.com/dockersamples/example-voting-app)
- DevOps workflow concepts: [Abhishek Veeramalla – DevOps Projects Series](https://www.youtube.com/@AbhishekVeeramalla) (YouTube)

> All infrastructure, automation, pipelines, Kubernetes configuration, monitoring setup, and troubleshooting in this repository are custom-built and original work.
