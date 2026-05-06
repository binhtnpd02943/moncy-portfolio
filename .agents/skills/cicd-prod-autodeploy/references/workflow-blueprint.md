# Workflow Blueprint

## Release Integrity Principles

Prefer these defaults unless the project already uses another safe contract:

- build one immutable artifact or image per release candidate
- tag artifacts with commit SHA, release ID, or exact image digest
- validate that same artifact in staging and promote it to production when feasible
- serialize production deploys with workflow concurrency or environment locking
- prefer short-lived CI identity such as OIDC or runner-attached identity over long-lived credentials
- separate application rollout from destructive data changes

## Recommended Default Flow

1. Developers merge work into `stg`.
2. CI runs test, lint, build, and packaging on `stg`, producing one immutable artifact or image reference.
3. If staging is auto-deployed, staging deploy uses that exact artifact with staging-specific assets such as `docker-compose.stage.yml`.
4. Release candidate is reviewed in staging.
5. A PR/MR from `stg` to `main` is created with the release identity kept explicit.
6. Only PO or circle lead can approve and merge.
7. Merge into `main` triggers the production deploy pipeline.
8. Pipeline serializes production deploy execution and uses provider-specific or host-specific deploy logic.
9. Production deploy promotes the same tested artifact or image reference whenever the platform allows it.
10. Production deploy uses production-specific assets such as `docker-compose.prod.yml` or release directories.
11. Deploy script performs backup, approved backward-compatible migrations if any, release switch, restart, smoke test, and health check.
12. If health check fails, rollback script runs.

## Shared-Host Deployment Patterns

### Pattern A: Release Directories + Symlink Switch

Use when:

- app runs on VM
- build artifact is a folder, JAR, or package
- fast rollback is needed

Layout:

```text
/opt/app/
  current -> /opt/app/releases/20260504-153000
  releases/
  shared/
  backups/
```

Flow:

- upload new release to a timestamped directory
- reuse shared config and persistent directories
- switch `current` symlink atomically
- restart service
- smoke or health check with timeout
- switch back on failure

### Pattern B: Docker Compose Service Update

Use when:

- host already uses Docker Compose
- service boundaries are already containerized
- avoiding port collision is critical

Flow:

- pull or load the exact image tag or digest
- validate compose file and env file
- serialize deployment so only one production rollout runs at a time
- update only the target service
- run smoke and health checks against the service endpoint

### Pattern C: In-Place Restart

Use when:

- small internal app
- downtime accepted
- rollback is simple

Avoid this by default on crowded shared hosts.

## Output Template

Every solution should declare:

- source branch and target branch
- staging deployment path, if any
- pipeline trigger
- provider or host target
- asset ownership for pipeline, runtime manifests, and deploy scripts when the repo spans multiple modules or repos
- artifact or image strategy
- concurrency or environment lock strategy
- deploy scope: changed service only, release group, or full stack
- target service name
- target directory
- migration policy
- restart command
- health check command
- rollback command
