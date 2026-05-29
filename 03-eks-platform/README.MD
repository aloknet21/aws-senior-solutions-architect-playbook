# Amazon EKS Enterprise Platform

## Overview
Design of scalable, production-grade Kubernetes platform on AWS.

## Architecture
- Multi-AZ EKS Cluster
- Managed Node Groups
- Spot + On-demand mix
- Auto Scaling (HPA / Karpenter)

## Core Components
- Ingress Controller (ALB)
- Service Mesh (Optional - Istio)
- Cluster Autoscaler
- Prometheus + Grafana

## Security
- IAM roles for service accounts (IRSA)
- Network policies
- Private cluster setup

## Design Decisions
- EKS vs ECS → Kubernetes flexibility required
- Fargate vs EC2 → Control vs simplicity

## Scaling Strategy
- Horizontal Pod Autoscaler
- Cluster Autoscaler

## Trade-offs
| Choice | Reason |
|--------|--------|
| EKS | Complex workloads |
| ECS | Simpler workloads |

## Diagram
Refer: ../diagrams/eks.png

Here's the full 3-tier EKS architecture sized for a 30–40 node workload. Here's how the node groups break down:


Ingress (2–3 nodes) — AWS ALB with NGINX ingress controllers handles TLS termination, WAF, and rate limiting at the edge.
Frontend (6–8 nodes, m6i.2xlarge) — SSR pods and BFF (GraphQL gateway) scale horizontally via HPA. Redis handles session caching, and static assets are served from S3 + CloudFront.


Backend (14–18 nodes, m6i.4xlarge) — The heaviest tier. Microservices (auth, orders, payments, notifications, search, inventory) communicate over Istio's service mesh with mTLS. KEDA scales workers based on Kafka queue depth, not just CPU.


Messaging layer — MSK Kafka sits between backend and data for event streaming, with SQS/SNS handling async notification fanout and dead-letter queues.
Data (8–10 nodes, r6i/i4i instances) — StatefulSets for Aurora PostgreSQL, Redis Cluster, OpenSearch, and TimescaleDB. EBS gp3 per node, EFS for shared volumes, Velero for backups.





Platform (4–6 dedicated nodes) — Prometheus + Grafana, Loki, Jaeger, Vault, ArgoCD (GitOps), and OPA for policy enforcement. These nodes carry a dedicated taint so workloads don't bleed onto them.


Karpenter handles burst scaling with spot/on-demand blended pools — up to 12 additional nodes during traffic spikes before scaling back.




***

```markdown
# 🚀 AWS EKS 3-Tier Complex Architecture

https://img.shields.io/badge/AWS-EKS-orange
https://img.shields.io/badge/Kubernetes-Production-blue
https://img.shields.io/badge/Scalable-Yes-brightgreen
https://img.shields.io/badge/Multi--AZ-HA-success

Production-grade **3-tier microservices architecture on AWS EKS**, designed for **30–45 worker nodes**, high availability, and large-scale workloads.

---

## 📐 Architecture Overview

This system follows a layered architecture:

```

Internet → Ingress → Frontend → Backend → Data + Messaging → Observability

```

- **Microservices-based**
- **Event-driven architecture**
- **Multi-AZ deployment**
- **Autoscaling at every layer**

---

## 🌐 1. Internet / Client Layer

**Purpose:** External entry point for all users and clients.

### Components
- **Route 53** → DNS resolution
- **CloudFront CDN** → global edge caching
- **HTTPS traffic only**

### Responsibilities
- Domain routing
- Global latency optimization
- Traffic forwarding to ingress layer

---

## 🚪 2. Ingress Tier  
**2–3 nodes · m6i.xlarge**

**Purpose:** Secure traffic entry and routing inside the cluster.

### Components
- AWS ALB (via AWS Load Balancer Controller)
- NGINX Ingress Controller (3 pods)
- Cert Manager (TLS automation)
- ExternalDNS (Route53 sync)
- AWS WAF + Shield (security)

### Responsibilities
- TLS termination
- Path-based routing
- Rate limiting
- Web security & DDoS protection

---

## 🖥️ 3. Frontend Tier  
**6–8 nodes · m6i.2xlarge · HPA: 20 → 60 pods**

**Purpose:** Client-facing layer for UI rendering and aggregation.

### Components
- **SSR Pods (~20)** → Next.js / React (server-side rendering)
- **BFF Pods (~10)** → GraphQL gateway
- **Redis (×3)** → session & cache
- **S3 + CloudFront** → static content
- **LaunchDarkly** → feature flags

### Responsibilities
- UI rendering (SSR)
- Backend aggregation (BFF)
- Session management
- CDN offloading

---

## ⚙️ 4. Backend / Application Tier  
**14–18 nodes · m6i.4xlarge · HPA + KEDA**

**Purpose:** Core business logic implemented using microservices.

### Architecture
- **Istio Service Mesh (mTLS enabled)**
- **gRPC + REST APIs**
- **Event-driven workers**

### Services
- Auth Service → JWT / OAuth2 / OIDC
- Order Service → business logic
- Payment Service → PCI-compliant zone
- Notification Service → Email/SMS/Push
- Search Service → OpenSearch
- Inventory Service → SKU management
- Scheduler → cron/batch jobs
- Kafka Workers → async consumers
- gRPC Gateway → internal routing
- API Gateway → Kong / Envoy

### Features
- Circuit breaking & retries
- Service discovery
- Traffic routing policies
- Observability hooks

---

## 🧵 5. Messaging & Streaming Layer

### Kafka (AWS MSK)
- 3 brokers (r6i.2xlarge)
- Multi-AZ replication

**Topics:**
- Orders
- Payments
- Events
- Audit logs

### SQS / SNS
- Async message fanout

**Queues:**
- Email
- SMS
- Push
- Dead Letter Queue (DLQ)

### Purpose
- Decoupling services
- Async processing
- Fault tolerance

---

## 💾 6. Data Tier  
**8–10 nodes · r6i.4xlarge / i4i.4xlarge**

**Purpose:** Persistent state and databases.

### Components
- Aurora PostgreSQL (primary + read replicas)
- Redis Cluster (cache + pub/sub)
- OpenSearch (full-text search)
- TimescaleDB (time-series metrics)
- S3 Data Lake (archive storage)

### Storage
- StatefulSets + PVCs
- EBS gp3 (node storage)
- EFS (shared storage)

### Resilience
- Multi-AZ replication
- Cross-region replication
- Velero backups
- RDS snapshots

---

## 📊 7. Platform / Observability Layer  
**4–6 nodes · m6i.2xlarge**

**Purpose:** Monitoring, deployment, and governance.

### Components
- Prometheus + Grafana → metrics
- Loki → logs
- Jaeger → tracing
- Vault → secrets management
- ArgoCD → GitOps deployment
- OPA / Kyverno → policy enforcement
- Karpenter → autoscaling

### Capabilities
- Full observability
- Continuous delivery (GitOps)
- Security enforcement
- Cluster autoscaling

---

## 📦 8. Node Group Summary

| Tier        | Instance Type     | Nodes   | Scaling Strategy              |
|-------------|------------------|--------|------------------------------|
| Ingress     | m6i.xlarge       | 2–3    | Fixed                        |
| Frontend    | m6i.2xlarge      | 6–8    | HPA (20–60 pods)             |
| Backend     | m6i.4xlarge      | 14–18  | HPA + KEDA                   |
| Data        | r6i / i4i        | 8–10   | Cluster autoscaler           |
| Platform    | m6i.2xlarge      | 4–6    | Fixed                        |
| Burst (Spot)| Mixed instances   | 0–12   | Karpenter (on-demand + spot) |

**Total Nodes:** `~34–45`

---

## 🌍 9. AWS VPC & Network Design

### Availability
- 3 Availability Zones (Multi-AZ HA)

### Subnets
- Public → ALB, NAT Gateway, Bastion
- Private → EKS nodes, databases

### Networking Features
- VPC Endpoints → S3, ECR, CloudWatch
- Security Groups → least privilege
- IRSA → pod-level IAM roles
- Calico Network Policy → zero-trust networking

---

## ✅ Key Highlights

- ✅ Fully **production-ready architecture**
- ✅ **Microservices + Event-driven design**
- ✅ **Autoscaling at all layers (HPA, KEDA, Karpenter)**
- ✅ **Multi-AZ high availability**
- ✅ **Strong observability stack**
- ✅ **Secure by design (mTLS, WAF, IRSA, Vault)**

---

## 📌 Use Cases

- Large-scale eCommerce platforms
- FinTech systems (PCI workloads supported)
- SaaS platforms with high traffic
- Real-time/event-driven applications

---

## 📷 Reference Diagram

> Add your architecture diagram here

```

docs/eks-architecture.png

```

---

## 🤝 Contributing

Feel free to fork and extend this architecture for your use cases.

---

## 📄 License

MIT License
```



***

# 🧠 System Design Answer: Scalable EKS-based 3-Tier Architecture

## 🧩 Problem Statement

Design a **highly scalable, fault-tolerant web platform** (e.g., eCommerce / SaaS) that:

* Handles **millions of requests/day**
* Supports **real-time + async workflows**
* Is **highly available (Multi-AZ)**
* Is **secure (TLS, WAF, mTLS)**
* Scales **dynamically (traffic + events)**

***

## 🏗️ High-Level Architecture

```
Clients → CDN → Ingress → Frontend → Backend → Data + Messaging
                                   ↓
                         Observability / Platform
```

***

## 🌐 1. Client & Edge Layer

**Goal:** Reduce latency and protect backend systems.

### Design

* **Route53** → DNS routing
* **CloudFront** → CDN caching
* HTTPS enforced

### Why?

* Offloads traffic via caching
* Improves global performance
* Prevents direct exposure of backend

***

## 🚪 2. Ingress Layer

**Goal:** Secure entry point and intelligent routing.

### Design

* **AWS ALB + NGINX Ingress**
* TLS termination using **Cert Manager**
* **WAF + Shield** for protection

### Key Decisions

* Use ALB for AWS-native integration
* NGINX for flexible routing (path, host)

### Scalability

* Few nodes (2–3) since mostly routing

***

## 🖥️ 3. Frontend Tier (Presentation + Aggregation)

**Goal:** Serve UI and reduce backend complexity.

### Design

* **SSR (Next.js / React)** → SEO + performance
* **BFF (GraphQL gateway)** → aggregate APIs
* **Redis** → session/cache
* **S3 + CloudFront** → static assets

### Why BFF?

* Prevents multiple client → service calls
* Tailors APIs per frontend

### Scaling

* HPA-driven (20 → 60 pods)

***

## ⚙️ 4. Backend Tier (Microservices)

**Goal:** Decouple business logic into scalable services.

### Design

* Microservices (Auth, Orders, Payments, etc.)
* **Istio service mesh (mTLS)**
* **gRPC (internal) + REST (external)**

### Patterns Used

* **Event-driven architecture**
* **Service mesh for resilience**
* **API Gateway for external access**

### Scalability

* HPA → CPU/memory scaling
* KEDA → event/queue-based scaling

***

## 🧵 5. Messaging & Event Layer

**Goal:** Enable loose coupling and async processing.

### Design

* **Kafka (MSK)** → event streaming
* **SQS / SNS** → queue + fanout

### Why Two Systems?

* Kafka → high-throughput streaming
* SQS/SNS → simple async workflows

### Benefits

* Reliability via retries + DLQ
* Decoupling services
* Smooth traffic spikes

***

## 💾 6. Data Layer

**Goal:** Reliable, scalable storage.

### Design

* **Aurora PostgreSQL** → transactional DB
* **Redis Cluster** → caching
* **OpenSearch** → search
* **TimescaleDB** → time-series
* **S3** → data lake

### Key Decisions

* Use polyglot persistence (right DB for right use case)
* Multi-AZ replication

### Reliability

* Backups (Velero, snapshots)
* Cross-region replication

***

## 📊 7. Observability & Platform Layer

**Goal:** Operate system reliably at scale.

### Design

* Metrics → **Prometheus + Grafana**
* Logs → **Loki**
* Traces → **Jaeger**
* Secrets → **Vault**
* Deployments → **ArgoCD (GitOps)**
* Policies → **OPA/Kyverno**
* Autoscaling → **Karpenter**

### Why?

* Full visibility (metrics + logs + traces)
* GitOps ensures reproducible deployments
* Policy enforcement improves security

***

## 🌍 8. Infrastructure & Network Design

### Design

* **3 Availability Zones**
* Public + Private subnets
* **VPC Endpoints (no public egress)**

### Security

* Security Groups (least privilege)
* **IRSA** (pod-level IAM)
* **Network policies (Calico)**

***

## ⚖️ Key Trade-offs

| Decision      | Benefit                 | Trade-off           |
| ------------- | ----------------------- | ------------------- |
| Microservices | Scalability, isolation  | Complexity          |
| Kafka + SQS   | Flexibility             | Ops overhead        |
| Istio         | Observability, security | Resource cost       |
| SSR           | SEO & UX                | Higher compute cost |

***

## 🚀 Scaling Strategy

* **Horizontal scaling everywhere**
* HPA → CPU/memory
* KEDA → event queues
* Karpenter → node autoscaling
* CDN → traffic offload

***

# 🔧 Tools Overview (Short Explanation)

## Networking & Edge

* **Route53** → DNS routing
* **CloudFront** → CDN caching globally
* **AWS ALB** → Layer 7 load balancing
* **NGINX Ingress** → Kubernetes routing layer

## Security

* **AWS WAF + Shield** → DDoS + web attack protection
* **Cert Manager** → TLS automation
* **Vault** → secure secrets storage
* **OPA/Kyverno** → policy enforcement

## Compute & Orchestration

* **EKS (Kubernetes)** → container orchestration
* **Karpenter** → dynamic node provisioning
* **HPA** → pod autoscaling
* **KEDA** → event-driven scaling

## Application Layer

* **React / Next.js** → frontend
* **GraphQL (BFF)** → API aggregation
* **gRPC / REST** → service communication
* **Istio** → service mesh with mTLS

## Messaging

* **Kafka (MSK)** → high-throughput event streaming
* **SQS** → reliable queues
* **SNS** → pub/sub fanout

## Data

* **Aurora PostgreSQL** → relational DB
* **Redis** → caching
* **OpenSearch** → search engine
* **TimescaleDB** → time-series data
* **S3** → object storage / data lake

## Observability

* **Prometheus** → metrics collection
* **Grafana** → dashboards
* **Loki** → logs
* **Jaeger** → distributed tracing

## DevOps

* **ArgoCD** → GitOps deployments
* **Velero** → backup & restore

***




