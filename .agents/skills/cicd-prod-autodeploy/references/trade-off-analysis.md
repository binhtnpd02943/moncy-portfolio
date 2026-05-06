# CI/CD Trade-off Analysis And ADR

Use this file when a major deployment or container decision needs explanation.

## Decision Framework

For each major choice, document:

```markdown
## Decision Record

### Context
- Problem:
- Constraints:
- Team capability:
- Environment constraints:

### Options Considered

| Option | Pros | Cons | Operational Overhead |
|--------|------|------|----------------------|
| Option A | ... | ... | Low/Med/High |
| Option B | ... | ... | Low/Med/High |

### Decision
Chosen: ...

### Rationale
1. ...
2. ...

### Trade-offs Accepted
- What we give up:
- Why it is acceptable:
```

## Typical Decisions To Explain

- release directories + systemd vs Docker Compose
- Docker Compose vs AWS ECS / Azure Container Apps / Cloud Run
- ECS Fargate vs EC2
- App Service vs VM
- multi-stage Dockerfile vs single-stage Dockerfile
- one shared compose file vs split `stg` and `prod` compose files
- managed database/cache vs local containerized data services

## Required Mindset

Do not explain a choice only by convenience.
Always include:

- team skill and ops bandwidth
- runtime control needs
- rollback simplicity
- security posture
- cost implications
- future migration path
