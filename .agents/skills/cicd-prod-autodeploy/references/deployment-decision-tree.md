# Deployment Decision Tree

Use this file when the repository can plausibly be deployed in multiple ways and the skill must choose the smallest correct target.

Priority rule:

- if the user, repository, or team standard already fixes the target platform, use this tree only to refine choices inside that target
- do not use this tree to jump from a fixed target like `AWS ECR -> EC2` to another platform such as ECS, App Runner, or Kubernetes unless the user asks for alternatives

## Top-Level Decision Tree

```text
START: What is the actual production runtime target?

┌─ The repository already runs well as a VM-native app without containers?
│  ├─ YES
│  │  → Validate: Did the user ask to containerize?
│  │     ├─ NO  → Release directories + systemd
│  │     └─ YES → Evaluate whether containerization adds real value
│  └─ NO
│     → Go to container path
│
├─ The app is a single service or simple web app?
│  ├─ YES
│  │  → Validate: Is cloud PaaS acceptable?
│  │     ├─ YES → Consider App Service, App Runner, Cloud Run, Container Apps
│  │     └─ NO  → VM + Docker Compose or plain VM deploy
│  └─ NO
│     → Go to multi-service path
│
├─ The app has a few cooperating services and team wants low ops overhead?
│  ├─ YES → Docker Compose host, ECS Fargate, Azure Container Apps, or Cloud Run split services
│  └─ NO  → Evaluate Kubernetes only if scale, policy, or platform needs justify it
│
└─ Requires deep orchestration, service mesh, custom controllers, or very large-scale microservices?
   ├─ YES → EKS / AKS / GKE
   └─ NO  → Prefer the simpler non-Kubernetes option
```

## Practical Selection Rules

### Choose Release Directories + Systemd

When:

- repository is not containerized
- service is monolithic or simple
- team is comfortable with VM management
- rollback through symlink switching is enough

### Choose Docker Compose On VM

When:

- service is already containerized or should be containerized
- there are a few cooperating containers
- team wants low operational overhead
- production is still VM-based

### Choose Cloud PaaS / Managed Container Runtime

When:

- team wants minimal ops burden
- app is service-oriented but not Kubernetes-complex
- provider-specific services are acceptable

Examples:

- AWS App Runner or ECS Fargate
- Azure App Service or Container Apps
- GCP Cloud Run

### Choose Kubernetes

Only when:

- many independently deployed services
- platform engineering maturity exists
- strict multi-tenant isolation, service mesh, or custom controllers are needed
- team accepts the higher operational load

## Output Requirement

Every result should explicitly say:

- which path was chosen
- which simpler alternatives were considered
- why the chosen path fits team size, scale, and risk better
