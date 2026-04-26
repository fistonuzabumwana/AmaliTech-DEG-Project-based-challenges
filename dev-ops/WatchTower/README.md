# WatchTower: Production-Grade Observability Stack

WatchTower is a comprehensive observability framework designed for microservices architectures. It leverages industry-standard tools to provide real-time monitoring, visualization, alerting, and structured log analysis, demonstrating modern DevOps best practices for maintaining service reliability and performance.

## 🏗️ Architecture Overview

The system follows a centralized observability pattern where metrics are autonomously scraped from microservices and visualized through a unified dashboard interface.

```text
┌──────────────────────────────────────────────────────────┐
│                  Visualization Layer                     │
│               ┌────────────────────────┐                 │
│               │  Grafana (Dashboards)  │                 │
│               └───────────┬────────────┘                 │
└───────────────────────────│──────────────────────────────┘
                            │ Queries
┌───────────────────────────▼──────────────────────────────┐
│                   Monitoring Layer                       │
│               ┌────────────────────────┐                 │
│               │    Prometheus (TSDB)    │                 │
│               └───────────┬────────────┘                 │
└───────────────────────────│──────────────────────────────┘
                            │ Scrapes /metrics
┌───────────────────────────▼──────────────────────────────┐
│                   Application Layer                      │
│  ┌──────────────┐  ┌──────────────────┐  ┌──────────────┐│
│  │ Order Service│  │ Tracking Service │  │ Notification ││
│  └──────┬───────┘  └────────┬─────────┘  └──────┬───────┘│
└─────────│───────────────────│───────────────────│────────┘
          │                   │                   │
          │         ┌─────────▼──────────┐        │
          └─────────► Structured JSON Logs ◄───────┘
                    └────────────────────┘
```


## 🛠️ Technology Stack

- **Orchestration:** Docker Compose
- **Metrics Collection:** Prometheus
- **Visualization:** Grafana
- **Logging:** Structured JSON (Winston/Pino standard)
- **Services:** Node.js-based microservices

## 📋 Service Inventory

| Service | Port | Description |
| :--- | :--- | :--- |
| **Order Service** | `3001` | Manages core order processing and lifecycle. |
| **Tracking Service** | `3002` | Provides real-time delivery status updates. |
| **Notification Service** | `3003` | Dispatches multi-channel system notifications. |
| **Prometheus** | `9090` | Time-series database and metrics aggregator. |
| **Grafana** | `3000` | Analytics and visualization platform. |

## 🚀 Getting Started

### Prerequisites

- Docker Engine & Docker Compose
- Node.js (for local development, optional)

### Quick Start

1.  **Initialize Environment**
    ```bash
    cp .env.example .env
    ```
2.  **Deploy Stack**
    ```bash
    docker compose up --build -d
    ```

### Accessing the Stack

| Component | URL |
| :--- | :--- |
| **Grafana** | [http://localhost:3000](http://localhost:3000) |
| **Prometheus** | [http://localhost:9090](http://localhost:9090) |
| **Order Health** | [http://localhost:3001/health](http://localhost:3001/health) |
| **Tracking Health** | [http://localhost:3002/health](http://localhost:3002/health) |

## 📊 Analytics & Dashboards

The system includes a pre-configured **WatchTower Observability Dashboard** accessible via:
`Dashboards → WatchTower → WatchTower Observability Dashboard` (Default Credentials: `admin / admin`).

### Key Performance Indicators (KPIs)

- **HTTP Request Throughput:** Monitors traffic volume and distribution across services.
- **5xx Error Propensity:** Tracks server-side exceptions to ensure system reliability.
- **Service Availability:** Real-time health signals derived from the Prometheus `up` metric.
- **Uptime Analytics:** A 24-hour availability percentage calculated via:
  `avg_over_time(up[24h]) * 100`

## 🚨 Alerting & Thresholds

Alerts are defined within `prometheus/alerts.yml` to ensure proactive response to system degradation.

| Alert Metric | Condition | Severity |
| :--- | :--- | :--- |
| **ServiceDown** | Instance unreachable for > 1 minute | Critical |
| **HighErrorRate** | > 5% 5xx errors over a 5-minute window | Warning |
| **MetricsStale** | Scrape failures exceeding 2 minutes | Information |

## 🪵 Logging Strategy

All microservices implement **Structured JSON Logging**. This enables machine-readability for log aggregation platforms and simplifies complex debugging.

**Log Format Example:**
```json
{"level":"info","service":"order-service","msg":"Processing payment","orderId":"12345"}
```

**Log Retrieval:**
```bash
docker logs order-service
```

## 🔐 Security & Network Isolation

- **Internal Networking:** Services communicate over an isolated Docker bridge network, minimizing public exposure.
- **Credential Management:** Sensitive data is handled via `.env` files and is explicitly excluded from version control.
- **Minimal Exposure:** Only the required dashboard and entry-point ports are exposed to the host.

## 📁 Project Structure

```text
WatchTower/
├── docker-compose.yml       # Stack orchestration
├── .env.example             # Environment template
├── prometheus/              # Monitoring configuration
│   ├── prometheus.yml       # Scrape configurations
│   └── alerts.yml           # Alert rules definitions
├── grafana/                 # Visualization provisioning
│   ├── dashboards/          # JSON dashboard definitions
│   └── provisioning/        # Automated datasource setup
└── app/                     # Microservice implementations
    ├── order-service/
    ├── tracking-service/
    └── notification-service/
```

## 📈 Future Roadmap

- [ ] **Notification Gateway:** Integrate Alertmanager for Slack and E-mail dispatching.
- [ ] **Traceability:** Implement distributed tracing using OpenTelemetry and Jaeger.
- [ ] **Log Aggregation:** Centralize logging with the ELK (Elasticsearch, Logstash, Kibana) stack.
- [ ] **Orchestration Migration:** Transition to Kubernetes (K8s) for high availability.

---
© 2026 WatchTower Project. Built with stability and observability in mind.
