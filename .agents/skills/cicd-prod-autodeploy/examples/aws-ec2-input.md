## Demo Goal

Use this prompt when the target provider and runtime are already fixed to AWS EC2 and the skill must stay inside that lane.

## Input

Use this repository as the source of truth and design a production-safe CI/CD flow for AWS EC2.

Fixed target:
- provider: AWS
- runtime: EC2
- deploy path must stay inside the EC2 model
- do not redesign to ECS, EKS, App Runner, or Kubernetes

Delivery rules:
- inspect the full source tree before deciding the packaging model
- if the repo is a simple Java `.jar`, Python app, or Node service that fits VM-native deployment, do not force Docker
- if the repo is already containerized and Compose on EC2 is justified, keep the design inside the ECR -> EC2 or registry -> EC2 lane
- production must use immutable release identity, health checks, rollback, and deploy serialization
- direct push to `main` must be blocked
- only merge from `stg` to `main` triggers production deploy

Please return:
1. repository discovery and evidence
2. assumptions and open questions
3. chosen EC2 deployment model
4. ADR-style rationale
5. CI/CD flow for `feature/*`, `stg`, `main`
6. branch protection rules
7. concrete pipeline YAML
8. deploy scripts
9. rollback and validation checklist

## Expected Behavior

When the skill works correctly, it should:

- stay inside the AWS EC2 target lane
- choose the smallest correct deployment shape from source evidence
- prefer release directories + systemd for simple VM-native apps
- prefer ECR/registry -> EC2 + Compose only when the source and runtime justify it
- include immutable artifact or image identity
- include deploy serialization and health verification
- avoid restarting unrelated services on a shared EC2 host
