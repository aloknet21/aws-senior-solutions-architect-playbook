# AWS Networking & DNS Architecture

## Overview
Design of highly available, secure network architecture.

## Components
- VPC (Multi-AZ)
- Subnets:
  - Public
  - Private
- NAT Gateway
- Internet Gateway
- Transit Gateway

## Hybrid Setup
- VPN / Direct Connect
- VPC Peering
- PrivateLink

## DNS Strategy
- Route53
- Failover Routing
- Latency-based routing

## Design Decisions
- Private workloads isolated in private subnets
- Controlled outbound internet via NAT

## Trade-offs
- NAT Gateway (managed, costly)
- NAT instance (cheap, less scalable)

## Diagram
Refer: ../diagrams/networking.png