# ☸️ Clahan Academy V2 - Deployment & Orchestration Repository Documentation

This repository contains the deployment manifests, Helm charts, and GitOps configurations used to manage, run, and scale **Clahan Academy V2** in Kubernetes environments (specifically Azure Kubernetes Service).

---

## 📂 Directory Structure

```
clahan-academy-helm/
├── argocd/                     # GitOps Application & Project specifications
│   └── dev/
│       ├── application.yaml    # ArgoCD Application manifest
│       └── project.yaml        # ArgoCD AppProject definition
├── clahan-academy/             # Production-grade Helm chart
│   ├── Chart.yaml              # Chart metadata (v1.0.0)
│   ├── values.yaml             # Default configuration values
│   ├── values-dev.yaml         # Dev environment overrides
│   ├── values-prod.yaml        # Production environment overrides
│   └── templates/              # Kubernetes templates
│       ├── deployments/        # Microservice deployments
│       ├── services/           # Service endpoints
│       ├── hpa/                # Horizontal Pod Autoscalers
│       ├── pdb/                # Pod Disruption Budgets
│       ├── networkpolicies/    # Service egress/ingress rules
│       ├── secretproviders/    # Azure Key Vault Secrets CSI driver provider
│       ├── serviceaccounts/    # Pod service accounts with Workload Identity annotations
│       ├── storage/            # PVCs & persistent storage mappings
│       ├── gateway.yaml        # Ingress Gateway (kgateway)
│       └── httproute.yaml      # HTTP Routes for service routing
└── kubernetes/                 # Standalone manifests for local/manual deployments
```

---

## ⚙️ Helm Chart Architecture

The Helm chart dynamically generates resource definitions based on values defined in `values.yaml`.

### Key Features:
1.  **Workload Identity Integration**: The chart generates `ServiceAccounts` annotated with `azure.workload.identity/client-id`. This binds Kubernetes workloads to Azure Managed Identities, enabling passwordless secret access.
2.  **Secret Provider Class**: Integrates with the Secrets Store CSI Driver. Secrets such as SMTP configurations, storage keys, and database connections are synchronized directly from Azure Key Vault into Kubernetes `Secret` resources.
3.  **Horizontal Pod Autoscaling (HPA)**: Configurations for resource-heavy services (`auth`, `exam`, `proctoring`) to scale out automatically when CPU utilization exceeds 70%.
4.  **Network Isolation**: Implements strict `NetworkPolicies` ensuring that only required internal communication channels are open. For example, `ai-service` and database ports are not exposed to the public internet.
5.  **Dedicated Node Pools**: Specific workloads like `ai-service` and `ollama` are configured with `nodeSelector` and `tolerations` mapping to specialized GPU pools (`pool: ai`), separating them from standard application node pools.

---

## 🔄 ArgoCD & GitOps Workflow

Continuous Deployment is managed declaratively by ArgoCD.

*   **AppProject (`argocd/dev/project.yaml`)**: Defines the target namespaces, repositories, and allowed resources.
*   **Application (`argocd/dev/application.yaml`)**: Points ArgoCD to the `clahan-academy` Helm chart in this repo and synchronizes state to the AKS cluster in real-time. When changes are pushed to this repository, ArgoCD detects the diff and updates the cluster automatically.

---

## 🚀 Manual Deployment

### Option A: Using the Helm Chart
To deploy manually using Helm (substituting variables):
```bash
helm upgrade --install clahan-academy ./clahan-academy \
  --namespace clahan-academy \
  --create-namespace \
  -f ./clahan-academy/values-dev.yaml
```

### Option B: Using Standalone Manifests (Quick Demo)
To quickly deploy the microservices stack without requiring Helm or Workload Identity integration:
```bash
kubectl apply -f kubernetes/ --recursive
```
