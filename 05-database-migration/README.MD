# Database Migration Strategy

## Overview
Approach for migrating on-prem databases to AWS.

## Migration Types
- Rehost
- Replatform
- Refactor

## Tools
- AWS DMS
- Native tools (Datapump, Dump/Restore)

## Decision Matrix
| Approach | Downtime | Complexity |
|----------|----------|------------|
| DMS | Low | Medium |
| Export/Import | High | Low |

## Strategy
- Use CDC for minimal downtime
- Perform validation before cutover

## Challenges
- Data consistency
- Large database size
- Schema compatibility

## Diagram
Refer: ../diagrams/database.png
