# CI/CD Examples

Use these examples as pattern references, not as rigid templates.

## Example 1: Small Internal App

```yaml
Requirements:
  - 1 backend service
  - small team
  - Linux VM target
  - fast rollback more important than orchestration sophistication

Recommended Shape:
  Packaging: no forced containerization or 1 Dockerfile only
  Deploy: release directories + systemd
  Governance: stg -> main, no direct push main

Trade-offs Accepted:
  - less portability than full container platform
  - lower ops burden and simpler rollback
```

## Example 2: SaaS Backend With Redis/PostgreSQL

```yaml
Requirements:
  - Python/Java/Node backend
  - Redis and PostgreSQL dependencies
  - staging and production split required
  - team wants containerized deploy on VM

Recommended Shape:
  Packaging: multi-stage Dockerfile
  Compose: docker-compose.yml for local dev, docker-compose.stage.yml for staging, docker-compose.prod.yml for production
  Deploy: registry push + VM Docker Compose deploy

Trade-offs Accepted:
  - more Docker complexity than raw VM deploy
  - clearer environment parity and easier image-based rollout
```

## Example 3: Mid-size Cloud-Native Team

```yaml
Requirements:
  - multiple services
  - auto-scaling needed
  - moderate ops maturity
  - provider is explicit

Recommended Shape:
  AWS: ECR + ECS Fargate
  Azure: ACR + Container Apps
  GCP: Artifact Registry + Cloud Run

Trade-offs Accepted:
  - more provider lock-in
  - much less VM operational overhead
```

## Example 4: AWS ECR To EC2 Fixed Target

```yaml
Requirements:
  - provider already fixed by team: AWS
  - registry already fixed: ECR
  - runtime already fixed: EC2
  - production deploy uses Docker Compose on EC2
  - secrets pulled from AWS Secrets Manager

Recommended Shape:
  Packaging: multi-stage Dockerfile
  Compose: docker-compose.yml for local dev, docker-compose.stage.yml for staging, docker-compose.prod.yml for production
  Deploy: GitHub Actions -> build -> push ECR -> SSH to EC2 -> pull -> migrate -> compose up -d

Trade-offs Accepted:
  - more VM operations than ECS
  - preserves the team-standard target and avoids redesigning the runtime platform
```

## Example 5: Enterprise Platform Team

```yaml
Requirements:
  - many independent services
  - complex networking and policy needs
  - dedicated platform team

Recommended Shape:
  Kubernetes only if justified by actual platform needs

Trade-offs Accepted:
  - highest operational complexity
  - maximum control and extensibility
```
