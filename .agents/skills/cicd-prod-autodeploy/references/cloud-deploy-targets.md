# Cloud Deploy Targets

Use this file when generating deployment workflow sections. The workflow must match the real target.

## Priority Rule

If the project, team, or user already specifies the target provider and runtime, do not switch to another target.
Examples:

- `AWS ECR -> EC2` means stay inside the AWS ECR to EC2 deployment model
- `Azure ACR -> App Service` means optimize that path, not replace it with Container Apps unless asked
- `VM Docker Compose` means stay VM-based unless the user requests provider alternatives

## Decision Order

1. Identify registry target
2. Identify runtime target
3. Identify secret source
4. Identify remote execution model
5. Generate provider-specific steps only after the first four are clear

## AWS

Typical shape:

- build image
- push to ECR
- deploy to EC2, ECS, or another AWS runtime
- fetch secrets from AWS Secrets Manager or SSM Parameter Store

If the target is specifically `ECR -> EC2`, compare only close variants inside that lane, such as:

- ECR -> EC2 with Docker Compose
- ECR -> EC2 with systemd-managed docker run

Do not jump to ECS, App Runner, or EKS unless the user asks for alternatives.

Good signals:

- ECR, EC2, ECS, Secrets Manager, SSM, ALB

Do not confuse:

- `AWS_ACCESS_KEY_ID` with account ID
- image push flow with runtime deployment flow

## Azure

Typical shape:

- build image
- push to Azure Container Registry
- deploy to VM, App Service, AKS, or Container Apps
- fetch secrets from Key Vault or environment configuration

Good signals:

- ACR, App Service, AKS, Container Apps, Key Vault

## GCP

Typical shape:

- build image
- push to Artifact Registry
- deploy to Cloud Run, GKE, or Compute Engine
- fetch secrets from Secret Manager

Good signals:

- Artifact Registry, Cloud Run, Compute Engine, Secret Manager

## Generic VM With Docker Compose

Typical shape:

- build image
- push to registry or transfer image artifact
- copy `docker-compose.prod.yml`
- update env file or secret material on host
- `docker compose pull`
- optional migration step
- `docker compose up -d`
- health check

Use this when the provider is unknown or the project is hosted on a plain Linux VM.

## Generic VM With Systemd

Typical shape:

- build artifact
- upload release
- switch symlink
- restart service
- health check

Use this when the project is not containerized in production.

## Output Requirement

Every generated workflow should state:

- target provider or host model
- registry choice
- secret source
- deploy command path
- rollback strategy
