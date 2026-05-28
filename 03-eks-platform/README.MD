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
