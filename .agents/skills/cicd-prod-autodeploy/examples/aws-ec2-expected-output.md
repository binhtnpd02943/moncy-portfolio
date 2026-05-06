## Expected Output Shape

This is not the only valid answer, but a strong result should look close to this structure.

## 1. Repository Discovery

- detected stack and evidence files
- build command and runtime command
- existing Docker or CI assets if any
- whether the source is better suited to:
  - EC2 + systemd + release directories
  - ECR or registry -> EC2 + Docker Compose

## 2. Assumptions And Open Questions

- exact service name
- deploy user and SSH path
- health check URL
- app port
- secret source strategy
- migration safety expectations

## 3. Chosen EC2 Deployment Model

- explain why the chosen model is the smallest correct fit
- explain why more complex AWS runtimes are rejected for now
- state whether rollout touches one service only or a coordinated release group

## 4. ADR Summary

- context
- options considered
- chosen option
- trade-offs accepted

## 5. CI/CD Flow

- `feature/*` -> CI validation
- `stg` -> integration checks and optional staging deploy
- PR from `stg` -> `main`
- merge into `main` triggers production deploy
- production deploy uses immutable artifact or image identity
- production deploy is serialized

## 6. Branch Protection

- no direct push to `main`
- require pull request
- require passing checks
- restrict merge approvals to named owners

## 7. Pipeline YAML

Output should include one copyable workflow such as:

- build artifact or image
- publish artifact or image with exact release identity
- SSH or remote deploy to EC2
- health verification
- rollback path

## 8. Deploy Scripts

Output should include scripts or commands that:

- validate required variables
- target only the intended service or release set
- switch release or update exact image
- run health checks
- rollback deterministically on failure

## 9. Validation Checklist

- target stayed inside AWS EC2 lane
- release identity is immutable
- direct push to `main` is blocked
- production deploy is serialized
- health check exists
- rollback exists
- unrelated services are not restarted without justification
