# Capstone Project: 3-Tier Kubernetes App Deployment on Azure

This project demonstrates the use of **Terraform** to provision infrastructure and **Azure DevOps Pipelines** to automate deployment of a 3-tier web application on **Azure Kubernetes Service (AKS)**.

---

## 📦 Project Overview

### Infrastructure Components (via Terraform):

* **Azure Kubernetes Service (AKS)**
* **Azure SQL Database**
* **Azure Storage Account** (for Terraform state)
* **Azure Virtual Network (VNet)** and Subnets
* **Azure App Monitoring Tools**: Prometheus & Grafana

### Application Layers:

* **Frontend**: Next.js (served via Nginx)
* **Backend**: Node.js REST API
* **Database**: Azure SQL

---

## 📁 Repository Structure

```
root/
│
├── terraform/                   # Infrastructure as code
│   ├── Azurerm/                 # Modularized resources
│   └── solution/                # Root terraform configuration
│
├── k8s_solution/               # K8s manifests for backend/frontend
│
├── frontend/                   # Frontend application code
└── backend/                    # Backend application code
```

---

## 🌐 Terraform Remote State (Azure Blob)

### Why Remote State?

Using a shared backend allows the team to:

* Keep state centralized and consistent
* Prevent concurrent modification issues
* Enable collaboration without conflict

### Backend Configuration

```hcl
terraform {
  backend "azurerm" {
    resource_group_name   = "DevOps1-tfstate-rg"
    storage_account_name  = "devopstfstatechamp"
    container_name        = "tfstate"
    key                   = "terraform.tfstate"
  }
}
```

### Storage Setup (One-Time Step)

```bash
az group create --name DevOps1-tfstate-rg --location "East US"
az storage account create --name devopstfstatechamp --resource-group DevOps1-tfstate-rg --sku Standard_LRS
az storage container create --name tfstate --account-name devopstfstatechamp --auth-mode login
```

---

## ⚙️ Terraform Deployment Process

### Prerequisites:

* Azure CLI
* Terraform CLI
* Contributor & Storage Blob Data Contributor roles
* Git

### Steps:

```bash
cd terraform/solution
az login
terraform init
terraform plan
terraform apply
```

---

## 💡 AKS Pipeline with Azure DevOps

### Pipeline 1: AKS Infrastructure + Monitoring

**Stage 1 – Terraform Deployment**

* Install Terraform
* Authenticate with Azure using service principal
* Run `terraform init`, `plan`, `apply`

**Stage 2 – Monitoring Setup**

* Install Helm
* Install Prometheus and Grafana via Helm
* Expose both services as LoadBalancers
* Use a `for` loop to wait up to 5 minutes for external IPs to be assigned
* Print the external IPs + Grafana credentials (admin/admin123)

```bash
for i in {1..10}; do
  kubectl get svc -n monitoring | grep -q 'pending' && sleep 30 || break
done
```

---

## 🧪 Frontend and Backend Pipelines

### Backend Pipeline (build & deploy):

1. **Build Docker image**

   * Push to Docker Hub

2. **Deploy to AKS**

   * Use `kubectl apply` to deploy:

     * Deployment
     * ClusterIP service
     * Ingress
   * Ingress managed via Helm-installed nginx controller

### Frontend Pipeline:

* Same structure as backend pipeline
* Deploys Next.js app via Nginx
* Uses config map for environment settings
* Displays external IP via pipeline logs

---

## 🔐 Secure Azure SQL Database

* The backend connects to Azure SQL via an environment variable (set in K8s ConfigMap)
* Database is provisioned via Terraform
* Virtual network rules restrict access to AKS subnet

---

## 📈 Access Monitoring Tools

After pipeline run:

* Visit Prometheus External IP (`/graph`)
* Visit Grafana External IP and login (`admin/admin123`)
* Use port 9090 for Prometheus and 3000 for Grafana

---

## 👥 Team Collaboration Notes

* Terraform state is shared via Azure Blob
* Avoid committing `.terraform/` or `*.tfstate` to Git
* All team members run the same commands:

```bash
terraform init
terraform plan
terraform apply
```gi