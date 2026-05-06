# CI/CD Review Rubric

Use this file when reviewing an existing CI/CD setup, deploy workflow, Docker assets, or release scripts.

## Review Modes

Choose one mode before proposing changes:

- audit only: identify strengths, gaps, and risks without editing files
- audit plus patch plan: identify risks and produce a prioritized change plan
- audit plus direct edits: patch the highest-value issues while preserving working components

Default rule:

- prefer patching the current system over replacing it wholesale unless the current design is fundamentally unsafe or incoherent

## Evaluation Pillars

### 1. Governance And Change Control

Check:

- direct push to `main` or production branch is blocked
- production deploy is tied to a controlled merge path
- approvals map to real users, teams, or groups
- CI status checks are required before merge

High-risk failures:

- anyone with broad write access can deploy to production
- production deploy runs on arbitrary branch pushes

### 2. Build Integrity And Artifact Promotion

Check:

- build output is deterministic enough to reproduce
- image tag or artifact version is tied to commit SHA, release ID, or digest
- production uses an exact artifact or image reference
- staging and production preferably use the same tested artifact when the platform allows it

High-risk failures:

- production deploy depends on `latest`
- staging validates one build while production deploys another build without explanation

### 3. Deployment Safety And Rollback

Check:

- deploy target is explicit
- deploy steps are idempotent enough for retries
- rollback path exists and is realistic
- production deploys are serialized by concurrency controls or locking
- shared-host collision checks exist when relevant
- rollout scope is explicit and does not restart unrelated services without reason

High-risk failures:

- no rollback path
- multiple production deploys can race each other
- deploy script can restart or overwrite the wrong service
- shared host deploy rebuilds the whole stack when only one service changed without justification

### 4. Secrets And Identity

Check:

- secrets are externalized
- CI authenticates with the target using the smallest realistic privilege set
- short-lived identity such as OIDC or runner-attached identity is preferred when supported
- deploy scripts do not print or persist secrets accidentally

High-risk failures:

- secrets committed to repository files
- long-lived personal or root credentials are the default deployment identity

### 5. Stateful Change Management

Check:

- migration behavior is explicit
- automatic migrations are limited to safe, backward-compatible changes
- destructive migrations or backfills are gated
- the team distinguishes app rollback from data rollback

High-risk failures:

- destructive schema changes run automatically during deploy
- rollback assumes database state magically reverts

### 6. Runtime Verification And Observability

Check:

- health check or smoke test exists
- timeout and failure behavior are explicit
- logs, metrics, or error signals can be inspected after deploy
- production success is not inferred only from `docker compose up -d` or service restart exit code
- generated YAML, Docker, or script artifacts are validated with the best available artifact-specific check

High-risk failures:

- no post-deploy verification
- deploy declared successful before application readiness is confirmed

### 7. Platform Fit And Complexity

Check:

- deployment target matches team capability and actual scale
- solution is not over-engineered relative to the environment
- cloud-specific workflow matches the fixed provider and runtime

High-risk failures:

- Kubernetes chosen without operational justification
- workflow targets a different runtime than the real production environment

## Severity Model

Use this severity model when reporting findings:

- `Critical`: production safety is materially compromised; fix before enabling or keeping auto-deploy
- `High`: major operational or security risk; fix in the next hardening pass
- `Medium`: correctness, maintainability, or resilience issue; schedule soon
- `Low`: improvement opportunity, naming cleanup, or non-blocking ergonomics

## Minimum Bar For Production Auto-Deploy

Do not call a workflow production-ready unless all of these are true:

- protected production branch and controlled merge path exist
- exact artifact or image reference is deployed
- secrets are externalized
- health check exists
- rollback path exists
- destructive migrations are not part of the default automated path
- production deploys are serialized
- target provider or host model is explicit

## Recommended Review Output

Structure the response in this order:

1. current shape and evidence
2. what is already acceptable
3. critical and high risks
4. recommended target state
5. smallest patch plan
6. optional later improvements
