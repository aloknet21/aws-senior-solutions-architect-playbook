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