# Day 66: Provision an EKS Cluster with Terraform Modules

## 📌 Project Overview
This project automated the deployment of a production-style Amazon EKS (Elastic Kubernetes Service) cluster. Using official Terraform registry modules for both the networking tier (VPC) and the EKS control plane, this builds a highly available Kubernetes environment with a managed worker node group — deployable and destroyable with a single command.

---

## 📂 Project Directory Structure

<img width="308" height="359" alt="image" src="https://github.com/user-attachments/assets/5a79253f-d659-4eb4-8e73-17f72f487fad" />

---

## 📋 Configuration Files

### 1. `providers.tf`
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
  }
  required_version = ">= 1.0"
}

provider "aws" {
  region = var.region
}

provider "kubernetes" {
  host                   = module.eks.cluster_endpoint
  cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)

  exec {
    api_version = "client.authentication.k8s.io/v1beta1"
    command     = "aws"
    args        = ["eks", "get-token", "--cluster-name", module.eks.cluster_name]
  }
}
```

### 2. `variables.tf`
```hcl
variable "region" {
  description = "AWS region"
  type        = string
  default     = "us-east-1"
}

variable "cluster_name" {
  description = "EKS cluster name"
  type        = string
  default     = "terraweek-eks"
}

variable "cluster_version" {
  description = "Kubernetes version"
  type        = string
  default     = "1.31"
}

variable "node_instance_type" {
  description = "EC2 instance type for nodes"
  type        = string
  default     = "t3.medium"
}

variable "node_desired_count" {
  description = "Desired number of nodes"
  type        = number
  default     = 2
}

variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  default     = "10.0.0.0/16"
}
```

### 3. `terraform.tfvars`
```hcl
region             = "us-east-1"
cluster_name       = "terraweek-eks"
cluster_version    = "1.31"
node_instance_type = "t3.medium"
node_desired_count = 2
vpc_cidr           = "10.0.0.0/16"
```

### 4. `vpc.tf`
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "${var.cluster_name}-vpc"
  cidr = var.vpc_cidr

  azs             = ["${var.region}a", "${var.region}b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/role/internal-elb" = 1
  }

  tags = {
    Environment = "dev"
    Project     = "TerraWeek"
    ManagedBy   = "Terraform"
  }
}
```

### 5. `eks.tf`
```hcl
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 20.0"

  cluster_name    = var.cluster_name
  cluster_version = var.cluster_version

  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets

  cluster_endpoint_public_access           = true
  enable_cluster_creator_admin_permissions = true

  eks_managed_node_groups = {
    terraweek_nodes = {
      ami_type       = "AL2_x86_64"
      instance_types = [var.node_instance_type]

      min_size     = 1
      max_size     = 3
      desired_size = var.node_desired_count
    }
  }

  tags = {
    Environment = "dev"
    Project     = "TerraWeek"
    ManagedBy   = "Terraform"
  }
}
```

### 6. `outputs.tf`
```hcl
output "cluster_name" {
  value = module.eks.cluster_name
}

output "cluster_endpoint" {
  value = module.eks.cluster_endpoint
}

output "cluster_region" {
  value = var.region
}

output "cluster_security_group_id" {
  value = module.eks.cluster_security_group_id
}
```

### 7. `k8s/nginx-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-terraweek
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

---

## 🌐 Network Architecture

```text
                  +---------------------------------------------------+
                  |                  AWS Cloud Region                  |
                  |                                                     |
                  |     +---------------------------------------+       |
                  |     |               Target VPC               |      |
                  |     |                                         |     |
 Public Internet  |     |   +-------------------------------+     |     |
       |          |     |   |         Public Subnet          |     |     |
       v          |     |   |                                 |     |     |
 [User Browser] --+-----+---+-> AWS Elastic Load Balancer      |     |     |
                  |     |   +---------------|-----------------+     |     |
                  |     |                   | (NAT Gateway route)   |     |
                  |     |                   v                       |     |
                  |     |   +-------------------------------+       |     |
                  |     |   |         Private Subnet          |      |     |
                  |     |   |                                  |     |     |
                  |     |   |   EKS Worker Node (EC2)           |     |     |
                  |     |   |   Running Nginx Pod                |    |     |
                  |     |   +-------------------------------+       |     |
                  |     +---------------------------------------+       |
                  +---------------------------------------------------+
```

---

## 📈 Discussion Questions

### 1. Why does EKS require both public and private subnets?
- **Worker isolation**: EKS schedules worker node instances in private subnets, keeping them shielded from direct exposure to the internet.
- **Ingress path**: Public subnets host internet-facing Elastic Load Balancers (ELBs), which receive external traffic and forward it into the cluster over the NAT Gateway, without ever exposing the nodes themselves.

### 2. What do the subnet tags do?
- `"kubernetes.io/role/elb" = 1` — tells the AWS Cloud Controller Manager to place any external-facing `LoadBalancer` service in these public subnets.
- `"kubernetes.io/role/internal-elb" = 1` — tells the controller to place internal-only load balancers in these private subnets.

### 3. A note on module v20+
As of EKS module `v20.0`, `enable_cluster_creator_admin_permissions` defaults to `false`. Setting it to `true` grants the deploying IAM identity `system:masters` cluster access, which avoids `Unauthorized` errors when running `kubectl` right after `terraform apply`.

### 4. Reflection: Terraform-Provisioned EKS vs. Manual Kind/Minikube (Day 50)
Kind and Minikube are great for fast, local validation — everything runs inside a single machine's local resources, with no real cloud networking involved. Provisioning EKS through Terraform registry modules is a different exercise entirely: it stands up real multi-AZ VPC networking, IAM roles, managed node groups, and cloud load balancers — the same building blocks used in production. The payoff is that the entire environment becomes reproducible and disposable: one `terraform apply` builds it, one `terraform destroy` tears it down cleanly, with no manual cleanup steps to forget.

---

## 📸 Verification Checklist

| Item | Status |
|---|---|
| `terraform apply` completed successfully | ☐ |
| `kubectl get nodes` shows nodes in `Ready` state | ☐ |
| Nginx deployment running (3/3 pods) | ☐ |
| Nginx accessible via LoadBalancer external IP | ☐ |
| Total resources created by Terraform | ___ |



<img width="1194" height="950" alt="image" src="https://github.com/user-attachments/assets/05169c4a-c04d-4d45-97ae-3a19f721cc0a" />

<img width="1184" height="742" alt="image" src="https://github.com/user-attachments/assets/b3b34a54-5706-4dfc-ac67-803ad10f4a2e" />

<img width="2200" height="782" alt="image" src="https://github.com/user-attachments/assets/dadc2e17-2674-454b-8404-3ebd97cc181e" />


---

## 🧼 Cleanup Process

1. Removed the Kubernetes-managed LoadBalancer first, so its associated AWS ELB would be released before touching the VPC:
   ```bash
   kubectl delete -f k8s/nginx-deployment.yaml
   ```
2. Confirmed in the AWS Console (EC2 → Load Balancers) that the ELB was gone.
3. Ran the full teardown:
   ```bash
   terraform destroy
   ```
4. Verified in the AWS Console that everything was gone:
   - **EKS clusters**: none remaining
   - **EC2 instances**: no leftover node group instances
   - **VPC**: `terraweek-eks-vpc` deleted
   - **NAT Gateways**: deleted
   - **Elastic IPs**: released

Cost note: the single NAT Gateway used in this setup bills at roughly $0.045/hour plus data processing, so prompt teardown after the exercise avoids ongoing charges.
