# Security & Governance in AWS

## Overview
Implements enterprise-grade security controls and compliance enforcement.

## Key Components
- IAM (Least Privilege)
- SCP Policies
- AWS Config
- AWS Security Hub
- CloudTrail

## Security Strategy
- Zero Trust Model
- No implicit trust between services
- Identity-driven access

## Controls
- Encryption: KMS
- Network segmentation (SG, NACL)
- Logging enabled for all accounts

## Decision Points
- SCP vs IAM permissions
- Central vs decentralized governance

## Best Practices
- Use IAM roles over users
- Enable MFA
- Audit logs in central account

## Diagram
Refer: ../diagrams/security.png