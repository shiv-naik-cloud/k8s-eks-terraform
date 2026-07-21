# Provisioning Amazon EKS with Terraform

Infrastructure-as-Code project that provisions a complete, production-style
**Amazon EKS (Elastic Kubernetes Service)** cluster on AWS using **Terraform** —
including its own VPC, networking, IAM roles, and a managed node group. A
two-tier application is then deployed onto the cluster with `kubectl`.

## 🏗️ What This Builds

Running `terraform apply` provisions **53 AWS resources**, including:

- A dedicated **VPC** (`10.0.0.0/16`) spanning two Availability Zones
- **Public & private subnets**, an Internet Gateway, and a NAT Gateway
- An **EKS cluster** (Kubernetes v1.30) with public API access
- A **managed node group** of 2× `t3.small` EC2 worker nodes
- All required **IAM roles, policies, and security groups**
- A **KMS key** for cluster secret encryption

```
                          AWS Cloud (us-east-1)
   ┌──────────────────────────────────────────────────────────┐
   │  VPC  10.0.0.0/16                                         │
   │                                                          │
   │   Public Subnets            Private Subnets              │
   │   ┌──────────────┐          ┌──────────────────────┐    │
   │   │ NAT Gateway  │───────▶  │  EKS Managed Nodes    │    │
   │   │ Internet GW  │          │  2 × t3.small (EC2)   │    │
   │   └──────────────┘          │  → app Pods run here  │    │
   │                             └──────────┬───────────┘    │
   │                                        │                 │
   │                     ┌──────────────────▼──────────────┐  │
   │                     │   EKS Control Plane (managed)    │  │
   │                     └──────────────────────────────────┘  │
   └──────────────────────────────────────────────────────────┘
```

## 🧰 Tech Stack

- **Terraform** (with official `terraform-aws-modules/vpc` and `.../eks` modules)
- **Amazon EKS** — managed Kubernetes
- **AWS** — VPC, EC2, IAM, KMS, NAT Gateway
- **kubectl** — application deployment

## 📁 Project Structure

```
k8s-eks-terraform/
├── main.tf            # Provider, VPC module, EKS module
├── .gitignore         # Excludes state files & secrets
├── README.md
└── screenshots/       # Proof of a working cluster
```

## 🚀 How to Deploy

**Prerequisites:** AWS account, AWS CLI configured (`aws configure`), Terraform,
and kubectl installed.

```bash
# 1. Initialize (downloads modules & providers)
terraform init

# 2. Preview what will be created
terraform plan

# 3. Create the cluster (~15-20 min)
terraform apply

# 4. Connect kubectl to the new EKS cluster
aws eks update-kubeconfig --region us-east-1 --name my-eks-cluster

# 5. Verify the worker nodes are ready
kubectl get nodes

# 6. Deploy the two-tier app (namespace must be created first)
kubectl apply -f ../k8s-2tier-app/manifests/
kubectl get all -n twotier-app
```

## 💰 Cost Management (Important)

EKS is **not free** — the control plane, EC2 nodes, and NAT Gateway all incur
hourly charges. This project was run in a short, controlled window and then torn
down:

```bash
terraform destroy      # removes all 53 resources
terraform state list   # confirm empty = nothing left running
```

Always destroy the cluster after testing to avoid ongoing charges.

## 📸 Screenshots

| # | What it shows |
|---|---------------|
| 1 | `terraform apply` complete — 53 resources created |
| 2 | `kubectl get nodes` — 2 EKS worker nodes `Ready` |
| 3 | `kubectl get all -n twotier-app` — app running on EKS |
| 4 | AWS Console — EKS cluster `Active` |
| 5 | AWS Console — 2 EC2 instances running |

*(Screenshots are in the `screenshots/` folder.)*

## 🔑 Key Concepts Demonstrated

- **Infrastructure as Code** — an entire Kubernetes platform defined in ~40 lines
  of Terraform using reusable modules
- **VPC design** — public/private subnet separation across multiple AZs
- **EKS managed node groups** — worker nodes provisioned and joined automatically
- **IAM & RBAC** — understanding the difference between AWS IAM permissions and
  in-cluster Kubernetes access (`enable_cluster_creator_admin_permissions`)
- **Cost-conscious operations** — provision, verify, and tear down responsibly

## 📝 A Note on the Access Fix

On first apply, `kubectl` returned a "must be logged in" error. This is because
creating a cluster in AWS (IAM) is separate from having admin access *inside* the
cluster (Kubernetes RBAC). Adding
`enable_cluster_creator_admin_permissions = true` to the EKS module and
re-applying granted the necessary in-cluster access — a common real-world EKS
gotcha.

---

*Built as a hands-on learning project to provision real cloud infrastructure with
Terraform and run Kubernetes workloads on Amazon EKS.*
