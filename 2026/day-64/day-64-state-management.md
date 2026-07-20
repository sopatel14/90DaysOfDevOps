# Day 64: Terraform State Management and Remote Backends

## Overview

Today I learned how to manage Terraform state like a professional. State is the most important part of Terraform — it's the source of truth that maps between my `.tf` files and actual cloud resources. I learned about remote backends, state locking, importing resources, state surgery, and handling drift.

---

## Task 1: Inspect Current State

### What I Did

Explored my existing Terraform state from Day 63 to understand what Terraform tracks.

### Commands Used

```bash
terraform show
terraform state list
terraform state show aws_instance.main
terraform state show aws_vpc.main
cat terraform.tfstate | grep -A 5 '"serial"'
```

### Resources Tracked

- `aws_instance.main`
- `aws_internet_gateway.main`
- `aws_route_table.public`
- `aws_route_table_association.public`
- `aws_s3_bucket.imported`
- `aws_s3_bucket.lock_test`
- `aws_s3_bucket.logs`
- `aws_s3_bucket_ownership_controls.logs`
- `aws_security_group.main`
- `aws_subnet.public`
- `aws_vpc.main`
- `random_string.suffix`
- `data.aws_ami.amazon_linux`
- `data.aws_availability_zones.available`

**Total: 14 resources tracked**

### EC2 Instance Attributes in State

The state stores many more attributes than what I defined:

| Defined in Config | Stored in State |
|---|---|
| `instance_type` | Network: `public_ip`, `private_ip`, `public_dns`, `private_dns` |
| `ami` | Compute: `cpu_core_count`, `cpu_threads_per_core` |
| `tags` | Storage: `root_block_device` (`volume_id`, `size`, `type`) |
| `subnet_id` | Metadata: `metadata_options`, `enclave_options` |
| `vpc_security_group_ids` | State: `instance_state`, `disable_api_termination` |
| | Placement: `availability_zone`, `tenancy` |
| | Monitoring, hibernation, `user_data`, etc. |

### Serial Number

```json
"serial": 68
```

**What it represents:** The serial number increments each time the state file is updated. It's used for version tracking and conflict detection. When using remote state with locking, Terraform checks this serial to ensure you're working with the latest version of the state.

### Screenshots

1. `terraform state list` output showing all resources
2. `terraform show` showing all attributes
3. Serial number from `terraform.tfstate`

<img width="1120" height="402" alt="image" src="https://github.com/user-attachments/assets/47e5b545-7736-45b8-ad30-91bc92773c2e" />

<img width="1166" height="952" alt="image" src="https://github.com/user-attachments/assets/52c5ab96-115f-429b-89fd-9c72a5dd4832" />

<img width="1442" height="236" alt="image" src="https://github.com/user-attachments/assets/95205a51-c558-41b1-b563-7e134aa50607" />


---

## Task 2: Set Up S3 Remote Backend

### Why Remote Backend?

Storing state locally is dangerous because:

- One deleted file = infrastructure forgotten
- No team collaboration (only one person can run Terraform at a time)
- No state versioning or rollback
- No encryption by default

### Resources Created

**1. S3 Bucket (State Storage)**

```bash
aws s3api create-bucket \
  --bucket terraweek-state-souravpatel \
  --region us-east-1
```

- Bucket: `terraweek-state-souravpatel`
- Versioning enabled for recovery
- Encryption enabled for security

**2. DynamoDB Table (State Locking)**

```bash
aws dynamodb create-table \
  --table-name terraweek-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST \
  --region us-east-1
```

- Table: `terraweek-state-lock`
- Partition Key: `LockID` (String)
- Pay-per-request billing

### Backend Configuration (`backend.tf`)

```hcl
terraform {
  backend "s3" {
    bucket         = "terraweek-state-souravpatel"
    key            = "dev/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraweek-state-lock"
    encrypt        = true
  }
}
```

### Migration Process

1. Created S3 bucket with versioning enabled
2. Created DynamoDB table for locking
3. Added backend block to `backend.tf`
4. Ran `terraform init` and confirmed state migration
5. Verified local state file was cleared
6. Verified state file exists in S3 bucket

### Verification

- ✅ `terraform plan` showed no changes
- ✅ Local `terraform.tfstate` is empty
- ✅ State file exists in S3 bucket

### Screenshots

5. S3 bucket creation
6. DynamoDB table creation
7. `terraform init` migration prompt
8. State file in S3 bucket
9. `terraform plan` showing no changes

<img width="1200" height="254" alt="image" src="https://github.com/user-attachments/assets/d5a4e68a-d2f3-4835-97b9-d88fe578477b" />

<img width="1518" height="840" alt="image" src="https://github.com/user-attachments/assets/c6045eeb-60db-442d-9209-bcccd35178a3" />

<img width="1224" height="608" alt="image" src="https://github.com/user-attachments/assets/0f268bbb-579d-4ded-b348-30e28bae0046" />

<img width="3360" height="906" alt="image" src="https://github.com/user-attachments/assets/fb8efbd8-df52-4e89-a9a8-3855ecc855a2" />

<img width="1822" height="796" alt="image" src="https://github.com/user-attachments/assets/6b2fa3af-9632-4b98-8637-aa9086239fb9" />


---

## Task 3: Test State Locking

### What is State Locking?

State locking prevents multiple users from modifying the infrastructure simultaneously. It uses DynamoDB as a distributed lock store.

### Test Process

1. Opened two terminals in the same project
2. Terminal 1: Started `terraform apply` and waited at "Enter a value:"
3. Terminal 2: Ran `terraform plan` while Terminal 1 was waiting
4. Terminal 2 showed lock error ✅

<img width="2236" height="950" alt="image" src="https://github.com/user-attachments/assets/5180a258-12ec-443b-b6b7-71d159f46449" />


### Lock Error Message

```text
Error: Error acquiring the state lock

Error message: ConditionalCheckFailedException: The conditional request failed
Lock Info:
  ID:        [long-id]
  Path:      terraweek-state-souravpatel/dev/terraform.tfstate
  Operation: OperationTypePlan
  Who:       souravpatel@Souravs-MacBook-Pro
  Version:   1.9.0
  Created:   2026-07-20 09:30:45.123456 +0000 UTC
```

### Why Locking is Critical for Team Environments

1. **Prevents State Corruption** — When two people run Terraform simultaneously, they both read the same state file. The person who writes last overwrites the other's changes, corrupting the state.
2. **Prevents Infrastructure Conflicts** — Person A might be adding a resource while Person B is modifying it. Locking ensures only one person can make changes at a time.
3. **Provides Audit Trail** — The lock shows *who* is currently running Terraform, *what* operation they're running, and *when* they started.
4. **Prevents Accidental Destroys** — If someone runs `terraform destroy` while another person is applying changes, it could be catastrophic. Locking prevents this.
5. **Race Condition Prevention** — Without locking, the last write wins — meaning one person's changes get silently overwritten and lost.

### How Locking Works

1. Terraform creates an entry in DynamoDB before any operation
2. The entry contains: `LockID`, `Who`, `Operation`, `Timestamp`
3. Other operations check for the lock before proceeding
4. When the operation completes, the lock is deleted

### Force Unlock

If Terraform crashes and leaves a stale lock:

```bash
terraform force-unlock <LOCK_ID>
```

> ⚠️ **Warning:** Only use `force-unlock` when you're 100% sure no operation is running!

### Screenshots

10. Two terminals open side by side
11. Terminal 1 waiting for "yes"
12. Lock error in Terminal 2

<img width="2236" height="950" alt="image" src="https://github.com/user-attachments/assets/5180a258-12ec-443b-b6b7-71d159f46449" />


---

## Task 4: Import an Existing Resource

### What is `terraform import`?

`terraform import` brings existing infrastructure under Terraform management without destroying or recreating it. It only adds the resource to the state file — it doesn't modify the actual resource in AWS.

### Import vs. Create from Scratch

| Aspect | `terraform import` | Creating from scratch |
|---|---|---|
| Resource exists in AWS? | Yes (created manually) | No (Terraform creates it) |
| What Terraform does | Adds existing resource to state only | Creates new resource AND adds to state |
| Risk | Low — resource already exists | Medium — may fail or cost money |
| Use case | Migrating existing infrastructure | Building new infrastructure |
| Configuration | Must match reality | Can be customized |
| Destroy behavior | Will destroy if you run destroy | Will destroy if you run destroy |

### Import Process

1. Created bucket manually in AWS Console: `terraweek-import-test-souravpatel`

2. Wrote resource block with just the bucket name:

   ```hcl
   resource "aws_s3_bucket" "imported" {
     bucket = "terraweek-import-test-souravpatel"
   }
   ```

3. Ran import command:

   ```bash
   terraform import aws_s3_bucket.imported terraweek-import-test-souravpatel
   ```

4. Ran `terraform plan` — showed changes needed (tags were missing)

5. Updated config to match reality:

   ```hcl
   resource "aws_s3_bucket" "imported" {
     bucket = "terraweek-import-test-souravpatel"
     tags = {
       Environment = "TerraWeek"
       ManagedBy   = "Terraform"
     }
   }
   ```

6. Ran `terraform plan` again — showed "No changes" ✅

7. Verified with `terraform state list`

### What I Learned

- Import only adds to state, doesn't modify infrastructure
- Configuration must match reality (or plan shows changes)
- After import, Terraform fully manages the resource
- Import is great for migrating existing infrastructure to Terraform

### Screenshots

15. Bucket in AWS Console
16. Resource block in config
17. Import success
18. State list showing imported bucket

<img width="3342" height="900" alt="image" src="https://github.com/user-attachments/assets/716af840-f408-4732-9b37-8d36011a46d7" />

<img width="1914" height="758" alt="image" src="https://github.com/user-attachments/assets/1e334b36-5eb3-48a3-834a-dc1acbf66fd9" />

<img width="1118" height="498" alt="image" src="https://github.com/user-attachments/assets/6324f4d9-f8c4-4eb4-8ed4-957ef5f7d68c" />

---

## Task 5: State Surgery — `mv` and `rm`

### What is `terraform state mv`?

The `mv` command renames a resource in the state file without changing the actual infrastructure.

**When to use `state mv`:**

- Renaming a resource in your Terraform configuration
- Moving resources between modules
- Reorganizing your state structure
- When you want to change resource names without destroying/recreating

### What is `terraform state rm`?

The `rm` command removes a resource from the state file **without** destroying it in AWS.

**When to use `state rm`:**

- Stopping Terraform from managing a resource
- Removing corrupted state entries
- Migrating resources to a different state file
- When you want to delete a resource from state but keep it in AWS

### State Surgery Practice

**1. Renamed Resource (`mv`)**

```bash
# Before
terraform state list | grep imported
aws_s3_bucket.imported

# After
terraform state mv aws_s3_bucket.imported aws_s3_bucket.logs_bucket

# Verify
terraform state list | grep logs_bucket
aws_s3_bucket.logs_bucket
```

**Result:** Resource renamed in state only, no infrastructure changes

**2. Removed from State (`rm`)**

```bash
terraform state rm aws_s3_bucket.logs_bucket
```

**Result:** Bucket removed from state but still exists in AWS

### Important Notes

- `state mv` only changes state, not infrastructure
- `state rm` only removes from state, doesn't destroy resources
- Always verify with `terraform plan` after state operations
- The resource still exists in AWS after `state rm`

### Screenshots


22. `mv` command success
23. `terraform state rm aws_s3_bucket.logs_bucket`
24. `terraform plan` one change added
25. import the bucket again
    
<img width="1818" height="360" alt="image" src="https://github.com/user-attachments/assets/5ea804de-ef94-4bb1-aff9-32404fbff3cf" />

<img width="1496" height="346" alt="image" src="https://github.com/user-attachments/assets/b4f0550b-1306-48c0-9344-929672a6f83e" />

<img width="2158" height="934" alt="image" src="https://github.com/user-attachments/assets/00f1a0a4-92ef-4522-bc6b-f649d41b0538" />

<img width="1926" height="862" alt="image" src="https://github.com/user-attachments/assets/59a99773-11d4-4587-8c46-05375bf4da48" />

---

## Task 6: Simulate and Fix State Drift

### What is State Drift?

State drift occurs when infrastructure changes outside of Terraform (through AWS Console, CLI, or another tool). Terraform's state no longer matches reality.

### Common Causes of Drift

- **Manual changes** — Someone modified resources in AWS Console
- **Other tools** — Resources modified by other IaC tools
- **Auto-scaling** — Resources that change automatically
- **Human error** — Accidental modifications

### My Drift Example

**What Changed:**
Manually changed the `Name` tag of the EC2 instance from `terraweek-dev-server` to `ManuallyChanged` in AWS Console.

**How Detected:**

```bash
terraform plan
```

Terraform detected the drift and showed it would update the tags.

**How Fixed (Option A — Reconcile):**

```bash
terraform apply -auto-approve
```

- Applied Terraform to restore tags to original value
- Infrastructure matches config again ✅

**Result:**

- `terraform plan` shows "No changes"
- Tag restored in AWS Console

### How Teams Prevent Drift in Production

1. **Restrict Console Access** — Limit who can make changes in AWS Console
2. **Use CI/CD** — All changes go through Terraform in CI/CD pipelines
3. **Enable State Locking** — Prevents concurrent modifications
4. **Regular Drift Detection** — Run `terraform plan` in CI/CD pipelines
5. **Automated Remediation** — Auto-apply to fix drift
6. **Audit Trails** — Track who made changes and when
7. **Tagging Strategy** — Tag resources to identify what's managed by Terraform
8. **Infrastructure as Code Only** — Enforce rule: no manual changes

### Two Options to Handle Drift

**Option A: Reconcile (Force Reality to Config)**
- Run `terraform apply` to restore config
- Good for: Fixing unauthorized changes

**Option B: Accept Drift (Update Config to Reality)**
- Update `.tf` file to match manual change
- Good for: Intended changes

### Screenshots


27. Manual tag change in AWS Console
28. Drift detected in `terraform plan`
29. Apply completed — tags restored
30. Tag restored in AWS Console

<img width="2822" height="630" alt="image" src="https://github.com/user-attachments/assets/d15fa743-7d23-4090-95c9-fe089dba8683" />

<img width="1482" height="780" alt="image" src="https://github.com/user-attachments/assets/6b67b32d-eda7-473a-899e-d56f861d0198" />

<img width="713" height="455" alt="image" src="https://github.com/user-attachments/assets/b13a0918-6d0b-414b-aca6-f4acaa828f2a" />

<img width="3342" height="730" alt="image" src="https://github.com/user-attachments/assets/03d26e52-e589-490b-acf5-80b992805675" />


---

## Summary of Key Commands

| Command | Purpose |
|---|---|
| `terraform show` | Full state in human-readable format |
| `terraform state list` | List all resources tracked |
| `terraform state show <resource>` | Show specific resource attributes |
| `terraform init` | Initialize backend and providers |
| `terraform plan` | Check for drift and changes |
| `terraform apply` | Apply changes to infrastructure |
| `terraform import <address> <id>` | Import existing resource into state |
| `terraform state mv <src> <dst>` | Rename resource in state |
| `terraform state rm <address>` | Remove resource from state |
| `terraform force-unlock <lock-id>` | Remove stale state lock |
| `terraform apply -refresh-only` | Update state to match reality |

---

## Key Takeaways

1. **State is critical** — It's the source of truth. Never lose it!
2. **Always use remote backends** — S3 with DynamoDB locking for production
3. **Locking prevents corruption** — Critical for team environments
4. **Import existing resources** — Don't recreate, import them
5. **State surgery is powerful** — `mv` and `rm` can save you from recreating resources
6. **Drift happens** — Regular `terraform plan` catches it
7. **Prevent drift** — Restrict console access, use CI/CD
