# InfraBlueprint: Production-Grade Cloud Infrastructure Orchestration

## Executive Overview
**InfraBlueprint** is a sophisticated Infrastructure as Code (IaC) implementation designed to deploy a secure, high-availability environment on **Amazon Web Services (AWS)** using **Terraform**. This project serves as a cornerstone for modern DevOps practices, emphasizing automated provisioning, network isolation, and granular identity management.

### Principal Objectives:
- **Infrastructure as Code (IaC)**: Comprehensive, version-controlled environment lifecycle management via Terraform.
- **Hierarchical Network Design**: Logical separation between public ingress and private data layers within a custom-defined VPC.
- **Service Isolation**: Ensuring zero-trust communication paths for mission-critical data stores.
- **IAM-Driven Security**: Utilizing temporary, role-based service credentials over static administrative access keys.

---

## 🧱 Comprehensive Architecture Topology
The following diagram illustrates the logical high-level flow and network partitioning within the InfraBlueprint ecosystem.

```text
                   Internet
                      |
              ┌───────────────┐
              │  Internet GW  │
              └───────┬───────┘
                      |
              ┌───────▼───────┐
              │      VPC      │
              │  10.0.0.0/16  │
              └───────┬───────┘
                      |
           ┌──────────┴──────────┐
           │                     │
    ┌──────▼──────┐       ┌──────▼──────┐
    │Public Subnet│       │Public Subnet│
    │ 10.0.1.0/24 │       │ 10.0.2.0/24 │
    └──────┬──────┘       └──────┬──────┘
           │                     │
           └──────────┬──────────┘
                      │
                ┌─────▼─────┐
                │    EC2    │ <--- Web Server (Public)
                └─────┬─────┘
                      │
              (Allowed Traffic)
                      ▼
              ┌───────────────┐
              │ Security Group│
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │Private Subnet │
              │  10.0.3.0/24  │
              └───────┬───────┘
                      │
                      ▼
              ┌───────────────┐
              │    RDS DB     │ <--- Private (No public access)
              └───────────────┘

         S3 Bucket (Private, IAM-controlled)
```

> [!IMPORTANT]
> The **Private Subnet** is designed to be air-gapped from the public internet. This configuration ensures that data assets hosted on the RDS instance are only accessible through authorized compute resources within the VPC.

---

## ⚙️ Orchestration Workflow & Setup

### 1. Environmental Prerequisites
Ensure the following components are installed and globally accessible:
- **Terraform CLI**: The core engine for infrastructure state management.
- **AWS CLI**: For secure authentication and communication with the AWS API.

### 2. Authentication & Identity Configuration
Initialize your local AWS session with appropriate administrative permissions:
```bash
aws configure
```
*Input requirements: IAM Access Key ID, Secret Access Key, and the default Deployment Region (e.g., `eu-north-1`).*

### 3. Variable Initialization (tfvars)
Custom environment settings are managed via the `example.tfvars` file. Edit this file to define your specific deployment parameters.

**Variable Reference Mapping:**
| Variable | Type | Description |
| :--- | :--- | :--- |
| `aws_region` | `string` | The target AWS region for resource localization. |
| `vpc_cidr` | `string` | The CIDR block defining the primary network space (e.g., `10.0.0.0/16`). |
| `allowed_ssh_cidr` | `string` | Your specific authorized IP address for restricted SSH access. |
| `db_username` | `string` | Master administrative username for the RDS tier. |
| `db_password` | `string` | Secure master password for the RDS database instance. |
| `s3_bucket_name` | `string` | A globally unique identifier for the S3 object store. |

---

## 🚀 Infrastructure Lifecycle Management

Execute the following commands in sequence to manage the end-to-end infrastructure state:

| Phase | Command | Purpose |
| :--- | :--- | :--- |
| **Initialize** | `terraform init` | Downloads provider plugins and initializes the state backend. |
| **Plan** | `terraform plan -var-file="example.tfvars"` | Performs a dry-run to preview infrastructure modifications. |
| **Apply** | `terraform apply -var-file="example.tfvars"` | Executes the changes and provisions resources on AWS. |
| **Destroy** | `terraform destroy -var-file="example.tfvars"` | Dismantles the environment to eliminate unnecessary cloud costs. |

---

## 🧠 Strategic Design Decisions

### I. Logical Subnet Isolation
The **Relational Database Service (RDS)** is deployed exclusively within private subnets. This architectural decision prevents direct internet exposure and forces all traffic to pass through security group filters, effectively minimizing the project's attack surface.

### II. Role-Based Identity (IAM Instance Profiles)
Rather than utilizing static, long-term access keys on the compute hosts, the infrastructure assigns an **IAM Instance Profile** to the EC2 instances. This enables:
- Secure, token-based interaction with S3 and other AWS services.
- Elimination of credential storage on the instance disk.
- Adherence to AWS security best practices (Least Privilege).

### III. Hardened Storage Policy (S3)
The S3 bucket is instantiated with **Public Access Block** configurations enabled. This ensures that object access is managed strictly through IAM-controlled policies, preventing accidental data exposure.

---

## ⭐ Bonus Implementation: Automated RDS Snapshot Protection

### Feature Overview
The Terraform configuration has been extended to ensure that a **final snapshot of the RDS database is automatically generated** before the instance is destroyed. This protection mechanism is critical for production-grade data governance and recovery.

**Implementation Details:**
```hcl
skip_final_snapshot       = false
final_snapshot_identifier = "vela-db-final-snapshot"
```

### Rationale & Strategic Value
In production environments, deleting a database without a final backup can lead to catastrophic data loss. This implementation prioritizes data integrity and disaster recovery (DR) preparedness:

*   **Proactive Data Protection:** Ensures a recovery point exists even during planned infrastructure teardowns or accidental `terraform destroy` executions.
*   **Production Standards:** Aligns the infrastructure with real-world backup policies and compliance requirements.
*   **Resilient Lifecycle:** Demonstrates an advanced awareness of cloud reliability concerns beyond basic resource provisioning.
*   **Seamless Restoration:** Provides a deterministic identifier (`vela-db-final-snapshot`) that can be used to quickly restore the environment from the final state.


---

## 🔐 Security Governance
- **Perimeter Control**: Administrative SSH ingress is strictly restricted to a specific authorized IP (/32).
- **Network Segmentation**: No public routes exist for the private database or internal service tiers.
- **Credential Integrity**: No sensitive values (passwords, private keys) are committed to version control.

---

## 📦 Operational Outputs
Upon successful deployment, execute `terraform output` to retrieve critical environment identifiers:
- **EC2 Public IP Address**: Primary ingress for web traffic.
- **RDS Regional Endpoint**: Secure connection string for the database tier.
- **S3 Bucket Name**: The unique path for object storage management.

---

## 📁 Repository Structure
```text
infra/
├── main.tf            # Core Infrastructure Orchestration Logic
├── variables.tf       # Parameter Definition & Type Management
├── outputs.tf         # Exported Infrastructure Data Identifiers
├── example.tfvars     # Configuration Template for Variable Inputs
└── README.md          # Comprehensive Project Documentation
```
