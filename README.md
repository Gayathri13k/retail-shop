Retail Shop (E-commerce Microservices) – Documentation & Terraform Pack
This pack includes a production‑ready README.md, supporting docs, a refactored Terraform module for EKS, Kubernetes manifests (non‑Helm), and a CI/CD workflow for GitHub Actions.

---

📁 Repository Structure (proposed)
.
├─ README.md
├─ terraform/
│ └─ main.tf
├─ k8s/
│ ├─ catalog-deployment.yaml
│ ├─ catalog-service.yaml
│ ├─ ui-deployment.yaml
│ ├─ ui-service.yaml
└─ .github/
└─ workflows/
└─ docker-image.yml

If your current repo already has some of these, merge or replace as needed. Secrets used by CI/CD must be created in the repo’s Settings → Secrets and variables → Actions.

---

1. README.md

# Retail Shop – Microservices E‑commerce (DevOps Assignment)

This repository contains a containerized microservice (e.g., `product-service`) deployed on **Amazon EKS** using **Kubernetes manifests** (no Helm for app deployment), with **CI/CD via GitHub Actions**, and **monitoring using Prometheus & Grafana**. Infrastructure is provisioned using **Terraform**.

## ⚙️ Tech Stack

- **Containerization:** Docker → Docker Hub registry
- **IaC:** Terraform → Amazon EKS + VPC + managed node group
- **Orchestration:** Kubernetes (manifests)
- **CI/CD:** GitHub Actions (build, test, push image, deploy to EKS)
- **Observability:** Prometheus + Grafana (dashboards for uptime & request metrics)

## 🧭 Architecture Overview

Dev → GitHub → GitHub Actions → Docker Hub → EKS (K8s) → Users ↑ ↕ Terraform (EKS) Prometheus/Grafana

### Main Components

- **EKS Cluster** with two public subnets and a managed node group.
- **product-service** Kubernetes Deployment with resource limits, liveness/readiness probes, and **HPA**.
- **Service** exposed via **Ingress** (NGINX) or **LoadBalancer**.
- **Prometheus/Grafana** used to monitor uptime and request metrics.

## 🚀 Quick Start

### A) Infrastructure (Terraform)

```bash
cd infra/terraform
terraform init
terraform plan -out tfplan
terraform apply tfplan
Outputs include VPC ID, EKS cluster endpoint/name, node group, and public subnets. After apply, update kubeconfig:
aws eks update-kubeconfig --region <region> --name <cluster_name>
B) Build & Push Image (local)
docker build -t ${{ secrets.DOCKER_HUB_USERNAME }}/ui:${{ github.sha }} ./ui docker push ${{ secrets.DOCKER_HUB_USERNAME }}/ui:${{ github.sha }}
C) Deploy to Kubernetes (manifests)
kubectl apply -f k8s/catalog-deployment.yaml
kubectl apply -f k8s/catalog-service.yaml
kubectl apply -f k8s/ui-deployment.yaml
kubectl apply -f k8s/ui-service.yaml
(or)
kubectl apply -f .
D) CI/CD (GitHub Actions)
On push to main, the workflow will build & push the Docker image and apply manifests to EKS. Configure GitHub Action secrets (see docs/CI-CD.md).
E) Monitoring
•	Install Prometheus & Grafana (see docs/MONITORING.md).
•	Service is annotated for scraping basic HTTP metrics (examples provided). Import the provided dashboard JSON or build panels using PromQL queries.
🔐 Secrets & Config
•	Put non‑secret runtime config into k8s/configmap.yaml.
•	Store secrets in GitHub Actions Secrets (CI/CD) and Kubernetes Secrets (runtime) if needed.
✅ Security, Scalability, Fault Tolerance
•	Limited blast radius via VPC networking; managed node groups.
•	Resource requests/limits + HPA for elasticity.
•	Readiness/liveness probes for self‑healing.
•	Network access controlled by Security Groups; use private subnets + NAT for production.
📄 Useful Paths
•	Terraform: terraform/
•	Kubernetes Manifests: k8s/
•	CI/CD Workflow: .github/workflows/ docker-image.yml
•	Docs: docs/
🧹 Cleanup
cd infra/terraform
terraform destroy

## ✅ Security, Scalability, Fault Tolerance

- **Networking isolation:** Dedicated VPC, subnets, and security groups for the EKS cluster.
- **Scalability:** Horizontal Pod Autoscaler (HPA) defined; scales pods based on CPU utilization (requires metrics-server).
- **Self-healing:** Liveness and readiness probes ensure unhealthy pods are restarted and traffic is routed only to healthy pods.
- **Access control:** Security Group rules restrict access; currently broad for demo purposes. In a production setup, private subnets with a NAT gateway would be recommended.

________________________________________
2) docs/ARCHITECTURE.md
Architecture
High‑level
•	Microservice (e.g., product-service) containerized with Docker, pushed to Docker Hub.
•	EKS cluster hosts the app. Traffic → Ingress → Service → Pods.
•	Prometheus scrapes targets and Grafana visualizes metrics.
Kubernetes Objects
•	Namespace → logical isolation.
•	Deployment → declarative rollout, replicas.
•	Service (ClusterIP) → stable access to pods.
•	Ingress (NGINX) → external HTTP(S) routing.
•	HPA → scale pods based on CPU (or custom metrics if available).
Reliability & Scaling
•	Rolling updates with maxUnavailable/maxSurge.
•	Probes for health checks.
•	Requests/limits tuned for HPA.
________________________________________
3) docs/ASSIGNMENT_MAPPING.md
Assignment → Implementation Mapping
•	Containerization → Dockerfile and CI build pushing to Docker Hub.
•	IaC (Terraform) → EKS cluster, VPC, subnets, security groups, IAM roles/policies.
•	Kubernetes Deployment → Plain manifests (no Helm) for Deployment
•	CI/CD → GitHub Actions builds, tests, pushes image, then applies manifests on main.
•	Monitoring & Logging → Prometheus & Grafana; dashboards for uptime & request metrics.
Deliverables are organized under repo root (README.md), infra/terraform, k8s, .github/workflows, and docs/.
________________________________________
4) docs/CI-CD.md
CI/CD – GitHub Actions
Secrets to add (Repository → Settings → Secrets and variables → Actions)
•	DOCKERHUB_USERNAME
•	DOCKERHUB_TOKEN
•	AWS_ACCESS_KEY_ID
•	AWS_SECRET_ACCESS_KEY
•	KUBECONFIG_CONTENT

Workflow Summary
1.	Checkout
2.	Log in to Docker Hub
3.	Build & push Docker image
4.	Configure AWS creds
5.	Update kubeconfig for EKS
6.	Apply Kubernetes manifests
The workflow file is in .github/workflows/ci-cd.yaml.
________________________________________
5) docs/MONITORING.md
Monitoring – Prometheus & Grafana
Option A (recommended): kube‑prometheus‑stack (Helm)
•	Installs Prometheus Operator, CRDs, Alertmanager, Grafana, and exporters.
•	After install, expose Grafana via Service or Ingress.
Option B: Minimal Prometheus + Grafana (manifests)
•	Use standalone Deployments/Services for Prometheus and Grafana.
•	Annotate application Service for HTTP metrics scraping.
Scrape Annotations (example)
Your k8s/service.yaml is annotated to scrape /metrics on port 8080. If your app exposes metrics elsewhere, adjust annotations.
Key PromQL
•	Uptime (per pod):
 	max by (pod) (time() - process_start_time_seconds{job="product-service"})
•	Request rate (HTTP):
 	sum(rate(http_requests_total{app="product-service"}[5m]))
•	Error rate (5xx):
 	sum(rate(http_requests_total{app="product-service",status=~"5.."}[5m]))
Grafana
•	Import a simple dashboard with 3 panels: Uptime, Request Rate, Error Rate.
•	Add Prometheus as a data source: URL http://prometheus-server.prometheus.svc.cluster.local:80 (adjust namespace/service name to your install).
________________________________________

```
