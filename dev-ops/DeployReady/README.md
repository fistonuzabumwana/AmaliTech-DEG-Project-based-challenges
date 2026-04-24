# DeployReady: Automated DevOps Ecosystem

## Overview

**DeployReady** is a high-performance, automated DevOps implementation demonstrating a comprehensive production-grade workflow. The project integrates advanced containerization, a robust CI/CD pipeline, and secure cloud orchestration on AWS to deliver a zero-touch deployment experience.

By abstracting manual intervention through GitHub Actions and AWS Systems Manager (SSM), the system achieves superior operational security and reliability.

---

## 🛠 Architecture & Workflow

The system follows a streamlined path from development to production:

1.  **Version Control**: Code is pushed to GitHub.
2.  **CI/CD Orchestration**: GitHub Actions triggers an automated pipeline.
3.  **Artifact Management**: Optimized Docker images are built and published to the **GitHub Container Registry (GHCR)**.
4.  **Cloud Orchestration**: AWS Systems Manager (SSM) executes remote deployment commands.
5.  **Runtime**: The application serves traffic via Docker on an AWS EC2 instance.

---

## Key Features

- **Zero-Trust Deployment**: Secure remote execution via AWS SSM, eliminating the need for open SSH ports during deployment.
- **Automated Resilience**: Integrated health-check-driven rollback mechanism to handle failed releases.
- **Optimized Containerization**: Multi-stage, non-root Docker builds for enhanced security and performance.
- **Full Lifecycle Automation**: End-to-end pipeline covering testing, artifact generation, and environment promotion.

---

## API Reference

The application serves a lightweight Node.js API designed for high availability and monitoring.

| Method | Endpoint   | Description                                                       |
| :----- | :--------- | :---------------------------------------------------------------- |
| `GET`  | `/health`  | Returns real-time service availability status.                    |
| `GET`  | `/metrics` | Provides system insights including uptime and memory utilization. |
| `POST` | `/data`    | Echoes valid JSON payloads to verify end-to-end connectivity.     |

---

## Containerization Strategy

Our container strategy prioritizes security and efficiency:

- **Base Image**: Leverages `node:alpine` for a minimal attack surface and reduced image size.
- **Process Security**: The application runs as a dedicated **non-root user**, mitigating potential container breakout risks.
- **Optimized Build**: Layer caching is optimized by separating dependency installation from source code injection.
- **Dynamic Configuration**: Port settings and secrets are managed via environment variables.

### Local Development

To replicate the production environment locally:

```bash
# 1. Initialize environment configurations
cp .env.example .env

# 2. Launch orchestrated services
docker compose up --build
```

---

## CI/CD Pipeline Workflow

The automated pipeline executes on every commit to the `main` branch to maintain continuous delivery:

- **Validation**: Repository checkout and integrity verification.
- **Testing**: Execution of core unit and regression tests (`npm test`).
- **Containerization**: Generation of Docker images tagged with unique Git commit SHAs.
- **Publication**: Secure push to the GitHub Container Registry (GHCR).
- **Deployment**: Targeted AWS SSM command execution to refresh the production instance.

---

## Security & Infrastructure

Infrastructure is managed following the **Principle of Least Privilege**:

- **SSM-First Operations**: Standard deployments use SSM exclusively, allowing for a fully closed SSH perimeter.
- **Access Control**: SSH access is strictly limited to authorized administrative IP addresses for emergency use only.
- **Secret Management**: Sensitive credentials reside exclusively within GitHub Actions Secrets.
- **IAM Scoping**: IAM roles are granularly scoped to provide only the necessary permissions for SSM and EC2 management.

---

## Automated Rollback Mechanism

To ensure absolute service continuity, the deployment logic includes an automated safety net:

1.  **Snapshot**: The previous stable image reference is retained before updating.
2.  **Deployment**: The new container is instantiated and initialized.
3.  **Verification**: A post-deployment health probe targets the `/health` endpoint.
4.  **Recovery**: If the probe fails, the system immediately terminates the faulty container and restores the previous known-good instance.

---

## Repository Structure

```text
dev-ops/DeployReady/
├── app/               # Node.js Application Source
├── Dockerfile         # Production Container Definition
├── docker-compose.yml # Local Service Orchestration
├── .env.example       # Template for Environment Variables
├── DEPLOYMENT.md      # Detailed Infrastructure Documentation
└── README.md          # Comprehensive Project Documentation
```

---

## Deployment Status

The live production environment is accessible and monitored at:
**[http://51.20.96.51/health](http://51.20.96.51/health)**

---

## Documentation

For detailed infrastructure setups and step-by-step installation guides, please refer to:

- [DEPLOYMENT.md](file:///e:/bbb/DEPLOYMENT.md)
