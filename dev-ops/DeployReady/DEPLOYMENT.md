# Deployment Strategy and Infrastructure Documentation

## Executive Overview
This document outlines the deployment architecture and automated procedures for the **DeployReady** application. Our strategy leverages modern DevOps principles, focusing on containerization, CI/CD automation, and secure cloud orchestration on AWS.

The primary objective is a **zero-touch deployment** model where code changes are automatically validated and promoted to production without manual SSH intervention, utilizing AWS Systems Manager (SSM) for secure command execution.

---

## 1. Infrastructure Architecture

### Cloud Provider Configuration
The application is hosted on Amazon Web Services (AWS) using an optimized EC2 instance configured for reliability and cost-efficiency.

| Component | Specification |
| :--- | :--- |
| **Instance Type** | `t2.micro` (Free Tier Eligible) |
| **Operating System** | `Amazon Linux 2023` |
| **Region** | `eu-north-1` (Stockholm) |
| **Orchestration** | AWS Systems Manager (SSM) |

### Network Security (Security Group)
The ingress/egress rules follow the principle of least privilege, minimizing the attack surface.

| Port | Protocol | Source | Purpose |
| :--- | :--- | :--- | :--- |
| `80` | TCP (HTTP) | `0.0.0.0/0` | Public API & Application Traffic |
| `22` | TCP (SSH) | Admin IP | Restricted Administrative Access |

> [!IMPORTANT]
> While SSH is configured for emergency debugging, it is strictly forbidden for standard deployment operations. All deployments are performed via SSM.

---

## 2. Environment Initialization

### Host Configuration
The EC2 instances are prepared with Docker to ensure environment parity. The following initialization sequence is executed:

```bash
# Update system packages
sudo dnf update -y

# Install and enable Docker Engine
sudo dnf install docker -y
sudo systemctl enable docker
sudo systemctl start docker

# Configure user permissions
sudo usermod -aG docker ec2-user
```

---

## 3. Identity and Access Management (IAM)

Security is enforced through specific IAM roles and users tailored for the deployment lifecycle.

### EC2 Service Role
The instance is attached to an IAM role containing the `AmazonSSMManagedInstanceCore` policy. This allows the instance to register with SSM and execute commands securely.

### CI/CD Deployment User
A dedicated IAM user handles the GitHub Actions integration with the following permissions:
- `AmazonSSMFullAccess`: To trigger remote commands.
- `AmazonEC2ReadOnlyAccess`: To verify instance status.

*Sensitive credentials (ACCESS_KEY_ID, SECRET_ACCESS_KEY) are securely stored in GitHub Secrets.*

---

## 4. Continuous Integration & Deployment (CI/CD)

The deployment pipeline is triggered automatically upon every merge to the `main` branch.

### Pipeline Workflow
1.  **Code Validation**: Checkout repository and verify integrity.
2.  **Dependency Resolution**: Install necessary modules (`npm install`).
3.  **Automated Testing**: Execute regression tests (`npm test`).
4.  **Artifact Containerization**: 
    - Build Docker image.
    - Tag with unique Git commit SHA for traceability.
5.  **Registry Publication**: Push image to GitHub Container Registry (GHCR).
6.  **Remote Execution**: Trigger AWS SSM to pull and deploy the new image.

---

## 5. Deployment Mechanics

Deployment is handled via the **AWS SSM Run Command** to maintain high security. The execution flow includes:

1.  **Registry Authentication**: Log in to GHCR on the target machine.
2.  **Image Procurement**: Pull the latest container image.
3.  **Lifecycle Management**: 
    - Stop and remove the existing container instance.
    - Instantiate the new container with production configurations.

```bash
docker run -d \
  --name deployready-app \
  --restart unless-stopped \
  -e PORT=3000 \
  -p 80:3000 \
  ${IMAGE_URL}:${GITHUB_SHA}
```

---

## ⭐ Bonus Implementation: Automatic Rollback on Failed Deployment

### Feature Workflow
The CI/CD pipeline has been enhanced with an autonomous rollback mechanism that ensures system stability by automatically reverting to the **previous stable Docker image** if a new deployment fails health validation.

**Automated Deployment Sequence:**
1.  **State Preservation:** The currently running image identifier is cached as the "Last Known Good" version.
2.  **Application Update:** The new container image is pulled and instantiated.
3.  **Health Validation:** A critical health probe is executed against the endpoint:
    ```bash
    curl http://localhost/health
    ```
4.  **Autonomous Recovery:** If the probe fails to return a success signal:
    *   The unstable container is immediately terminated and removed.
    *   The previous stable image is redeployed to restore service parity.

---

### Rationale & Strategic Value
In production-grade CI/CD environments, deployment failures (due to bugs, misconfigurations, or dependency conflicts) must be mitigated without manual human intervention:

*   **High Availability (HA):** Significantly reduces Mean Time to Recovery (MTTR) by automating the rollback process.
*   **Zero-Touch Recovery:** Eliminates the need for emergency manual SSH access or manual pipeline triggers during failures.
*   **Production-Grade DevOps:** Aligns the project with industry standards for resilient, self-healing infrastructure.
*   **Continuous Reliability:** Guarantees that the application remains accessible to end-users even if a faulty version is pushed to the `main` branch.


---

## 7. Monitoring and Observability

Post-deployment verification can be performed using standard Docker diagnostics:

| Action | Command |
| :--- | :--- |
| **Verify Container Status** | `docker ps` |
| **Review Application Logs** | `docker logs deployready-app` |
| **External Health Audit** | `curl http://<EC2_PUBLIC_IP>/health` |

Expected response for a healthy system:
```json
{ "status": "ok" }
```

---

## 8. Security Best Practices
- **Credential Sanitization**: No secrets are committed to version control.
- **Environment Isolation**: `.env` is ignored; `.env.example` provides the schema.
- **Minimal Exposure**: SSM eliminates the need for open SSH ports in standard workflows.

---

## 9. Final Deployment Status
The application is currently live and accessible at:
[http://51.20.96.51/health](http://51.20.96.51/health)
