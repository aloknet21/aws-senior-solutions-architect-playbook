# Disaster Recovery Strategy

## Overview
Ensures high availability and business continuity.

## Models
- Backup & Restore
- Pilot Light
- Warm Standby
- Active-Active

## Strategy
- Multi-AZ by default
- Multi-region for critical apps

## Metrics
- RTO (Recovery Time Objective)
- RPO (Recovery Point Objective)

## Failover
- Route53 DNS failover
- Automated recovery

## Trade-offs
| Model | Cost | Recovery Time |
|-------|------|--------------|
| Backup | Low | High |
| Active-Active | High | Low |

## Diagram
Refer: ../diagrams/dr.png
