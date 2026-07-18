# Day 63: Variables, Outputs, Data Sources, and Expressions


Today's engineering goal was to transform a static, hardcoded infrastructure configuration (Day 62) into a dynamic, highly reusable, and environment-aware pipeline. By leveraging Terraform Input Variables, Local Values, Dynamic Outputs, Data Sources, and Conditional Expressions, we decoupled the underlying infrastructure architecture from its configuration parameters.

---

## 🛠️ Core Architectural Concepts 

Before implementing the configuration files, it is vital to understand how Terraform manages data abstraction:

- **Input Variables (`variable`)**: Serve as function arguments or configurations exposed to external interfaces. They allow customization across deployment runs without altering core resource definitions.
- **Local Values (`local`)**: Serve as private, internal constants within a module. They are ideal for deduplicating repeated expressions, combining strings, or creating unified naming patterns.
- **Output Values (`output`)**: Act as return values for an infrastructure deployment. They surface critical resource telemetry (like IP addresses and IDs) directly to the terminal or pass them into remote execution states.
- **Data Sources (`data`)**: Provide read-only integration windows to query real-time configuration values directly from cloud APIs (e.g., dynamically locating active Availability Zones or finding the latest OS image).

---

## 📂 Production Code Registries

### 1. `variables.tf`

This file defines our input schemas. Note that `project_name` explicitly omits a default value to force user specification or environment variable passing at runtime.

```hcl
variable "region" {
  type        = string
  description = "The target AWS Region for deployment."
  default     = "us-east-1"
}

variable "vpc_cidr" {
  type        = string
  description = "The CIDR block network range for the VPC."
  default     = "10.0.0.0/16"
}

variable "subnet_cidr" {
  type        = string
  description = "The CIDR block range for the public subnet."
  default     = "10.0.1.0/24"
}

variable "instance_type" {
  type        = string
  description = "The compute sizing tier of the EC2 instance."
  default     = "t3.micro"
}

variable "project_name" {
  type        = string
  description = "The application or project identity moniker. No default assigned."
}

variable "environment" {
  type        = string
  description = "The specific tiering deployment environment."
  default     = "dev"
}

variable "allowed_ports" {
  type        = list(number)
  description = "List of network port numbers permitted for inbound security group rules."
  default     = [22, 80, 443]
}

variable "extra_tags" {
  type        = map(string)
  description = "Supplemental tag key-value pairs appended onto structural resources."
  default     = {}
}
```

### 2. `providers.tf`

The centralized provider interface. Notice how the region is mapped directly to our input variable.

```hcl
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region

  default_tags {
    tags = {
      EngineeredBy = "TerraWeek-Pipeline"
    }
  }
}
```

### 3. `main.tf`

Refactored to eliminate hardcoded configuration values, utilize centralized tagging structures via `locals`, dynamically search for AMIs, and process environment variables conditionally.

```hcl
# ----------------------------------------------------
# LOCALS BLOCK FOR DYNAMIC VALUE MANAGEMENT
# ----------------------------------------------------
locals {
  name_prefix = "${var.project_name}-${var.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

# ----------------------------------------------------
# DATA SOURCES FOR EXTERNAL APIS LOOKUPS
# ----------------------------------------------------
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

# ----------------------------------------------------
# COMPUTE AND NETWORK INFRASTRUCTURE RESOURCES
# ----------------------------------------------------
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = var.subnet_cidr
  availability_zone       = data.aws_availability_zones.available.names[0]
  map_public_ip_on_launch = true

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-subnet"
  })
}

resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-igw"
  })
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-rt"
  })
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

resource "aws_security_group" "main" {
  name        = "${local.name_prefix}-sg"
  description = "Allow SSH and HTTP traffic"
  vpc_id      = aws_vpc.main.id

  dynamic "ingress" {
    for_each = var.allowed_ports
    content {
      description = "Inbound connection configuration"
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    description = "All outbound traffic permitted"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-sg"
  })
}

resource "aws_instance" "main" {
  ami = data.aws_ami.amazon_linux.id

  # TASK 6 CONDITIONAL OPERATOR: Uses t3.small if prod, otherwise falls back to var input
  instance_type = var.environment == "prod" ? "t3.small" : var.instance_type

  subnet_id                   = aws_subnet.public.id
  vpc_security_group_ids      = [aws_security_group.main.id]
  associate_public_ip_address = true

  lifecycle {
    create_before_destroy = true
  }

  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
    systemctl enable httpd
    echo "<h1>Welcome to TerraWeek Server: ${local.name_prefix}</h1>" > /var/www/html/index.html
  EOF

  tags = merge(local.common_tags, var.extra_tags, {
    Name = "${local.name_prefix}-server"
  })
}

resource "aws_s3_bucket" "logs" {
  bucket     = "terraform-week-logs-${random_string.suffix.result}"
  depends_on = [aws_instance.main]

  tags = merge(local.common_tags, {
    Name    = "${local.name_prefix}-logs"
    Purpose = "Application logs"
  })
}

resource "aws_s3_bucket_ownership_controls" "logs" {
  bucket = aws_s3_bucket.logs.id
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
  depends_on = [aws_s3_bucket.logs]
}

resource "random_string" "suffix" {
  length  = 8
  special = false
  upper   = false
}
```

### 4. `outputs.tf`

Surfaces the system parameters directly to the console runtime output stack upon deployment.

```hcl
output "vpc_id" {
  description = "The ID of the created VPC."
  value       = aws_vpc.main.id
}

output "subnet_id" {
  description = "The ID of the public subnet."
  value       = aws_subnet.public.id
}

output "instance_id" {
  description = "The ID of the EC2 instance."
  value       = aws_instance.main.id
}

output "instance_public_ip" {
  description = "The public IP address of the EC2 instance."
  value       = aws_instance.main.public_ip
}

output "instance_public_dns" {
  description = "The public DNS name assigned to the EC2 instance."
  value       = aws_instance.main.public_dns
}

output "security_group_id" {
  description = "The ID of the security group."
  value       = aws_security_group.main.id
}
```

---

## 🌍 Environment Variables Files (`.tfvars`)

### `terraform.tfvars` (Development Standard — Auto-Loaded)

```hcl
project_name  = "terraweek"
environment   = "dev"
instance_type = "t3.micro"
```

### `prod.tfvars` (Production Parameters)

```hcl
project_name  = "terraweek"
environment   = "prod"
instance_type = "t3.small"
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
```

---

## 📈 Key Engineering Theory Breakdowns

### 1. The 5 Core Data Variable Types

| Type | Description | Example |
|------|-------------|---------|
| `string` | Plain textual parameters delimited by quotes | `"us-east-1"` |
| `number` | Integer or floating-point numerical metrics | `80`, `22` |
| `bool` | Explicit true/false state evaluations | `true`, `false` |
| `list` | Ordered, position-indexed sequences of matching parameters | `[22, 80]` |
| `map` | Unordered lookups mapping keys directly to configuration string layers | `{ Environment = "dev" }` |

### 2. Variable Precedence Order Hierarchy

When conflicting values are passed to the same input variable, Terraform resolves them using a strict, lowest-to-highest evaluation priority framework:

1. **Environment Variables** (System variables matching the pattern `TF_VAR_name`) — *Lowest Priority*
2. The `terraform.tfvars` file (Parsed automatically if present in the root folder)
3. The `terraform.tfvars.json` file
4. Any `*.auto.tfvars` or `*.auto.tfvars.json` files (Processed in alphabetical order)
5. Command-line file arguments (Values provided via the `-var-file` runtime flag)
6. Inline command-line assignments (Values provided via individual `-var` flags) — *Highest Priority*

### 3. Structural Difference: `resource` vs `data`

- **Managed Resource (`resource`)**: A declarative blueprint instructing Terraform to mutate infrastructure state directly via creation, modification, or deletion operations. Terraform actively tracks and owns lifecycles for these instances.
- **Data Source (`data`)**: A read-only query engine designed to import active external data points or query cloud endpoints outside Terraform's direct tracking scope. It never modifies or destroys cloud target architectures.

### 4. Top Five Core Built-In Functions Used

1. **`merge()`** — Combines multiple maps into a unified single lookup map. Indispensable for joining global tags with localized resource definitions.
2. **`cidrsubnet()`** — Dynamically divides a base network prefix into a calculated subnet layout without needing manual binary network maps.
3. **`lookup()`** — Retrieves values from a map key entry safely, providing a fallback option to prevent configuration build errors.
4. **`join()`** — Assembles independent string fragments into unified naming conventions using custom separator delimiters.
5. **`upper()`** — Forces text input mutations to uppercase formatting configurations, standardizing organizational cloud naming policies.

---

## 📸 Verification & Screenshot Registries

### 📷 Screenshot 1: Variable Input Prompt Enforcement

<img width="1156" height="168" alt="image" src="https://github.com/user-attachments/assets/0d8ae1d0-a226-41ef-bca6-80244d72991c" />


### 📷 Screenshot 2: Environment Precedence Execution Testing

<img width="1488" height="1648" alt="image" src="https://github.com/user-attachments/assets/d772c685-6315-43c5-b76f-51ad3c10df61" />


### 📷 Screenshot 3: Interactive Console Functions Testing

<img width="1138" height="296" alt="image" src="https://github.com/user-attachments/assets/d9043dcd-a403-4a06-8d4a-a6d3afac18f1" />

### 📷 Screenshot 4: Precedence Execution Testing

<img width="1488" height="1648" alt="image" src="https://github.com/user-attachments/assets/01f141dd-add1-4c98-bbe8-3280b7cab32d" />


### 📷 Screenshot 5: Infrastructure Verification!

<img width="3350" height="520" alt="image" src="https://github.com/user-attachments/assets/af6c8324-f8c9-4fe1-9078-1adf8d27acba" />


### 📷 Screenshot 6: Final Infrastructure Outputs Display

<img width="574" height="120" alt="image" src="https://github.com/user-attachments/assets/f4cb440c-d2d2-46b4-84c1-f78808212868" />
