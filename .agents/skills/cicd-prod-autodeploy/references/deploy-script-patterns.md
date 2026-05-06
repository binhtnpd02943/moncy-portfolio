# Deploy Script Patterns

## Baseline Script Rules

All generated shell scripts should prefer:

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
```

Use helper functions:

- `log` for timestamps
- `require_env` for mandatory env vars
- `require_bin` for required binaries
- `with_lock` or CI-side concurrency controls when deploys can race
- `rollback` trap only when the deploy flow truly supports automatic revert

Release identity rules:

- use a deterministic `RELEASE_ID`, `IMAGE_TAG`, or `IMAGE_DIGEST`
- never rely on floating production references such as `latest`
- keep previous release identity available for rollback

## Release Directory Pattern

Recommended variables:

```bash
APP_NAME
APP_ROOT
RELEASES_DIR
SHARED_DIR
CURRENT_LINK
SERVICE_NAME
ARTIFACT_PATH
HEALTHCHECK_URL
RELEASE_ID
```

Recommended phases:

1. validate variables and binaries
2. create release directory
3. unpack artifact
4. link shared config
5. run migrations if explicitly allowed
6. switch symlink
7. restart service
8. run smoke or health check with timeout
9. clean old releases

## Compose Deploy Pattern

Recommended variables:

```bash
COMPOSE_FILE
ENV_FILE
DOCKER_IMAGE
SERVICE_NAME
HEALTHCHECK_URL
IMAGE_TAG
IMAGE_DIGEST
```

Recommended phases:

1. validate compose file and env file
2. login to registry if needed
3. pull exact image tag or digest
4. run migrations only if explicitly required and safe for automatic rollout
5. `docker compose -f "$COMPOSE_FILE" up -d`
6. verify target service health
7. rollback to previous image tag if health check fails

## Health Check Rules

Prefer one of:

- HTTP endpoint such as `/health`
- local command returning zero exit code
- container health status

Do not consider deployment successful only because the restart command exited successfully.

Add a bounded timeout and a clear failure exit path so the pipeline can stop instead of hanging indefinitely.

## Rollback Rules

Rollback should:

- identify previous release deterministically
- restore symlink or previous image tag
- restart service
- rerun health check

For compose-based rollback, keep a deterministic previous image reference rather than relying only on `latest`.

Application rollback does not imply schema or data rollback. State rollback needs a separate plan.

## Migration Safety Rules

- automatic production migrations should be backward compatible by default
- destructive DDL, data rewrites, and long-running backfills should be gated outside the default auto-deploy path
- if rollback depends on database restore, say that explicitly and require backup readiness
- when migrations are optional, make the toggle explicit rather than silently running them every deploy

## Shared Environment Guardrails

Before restarting anything, validate:

- expected service name exists
- expected target directory belongs to the app
- expected port is the app's configured port
- expected reverse proxy target is unchanged unless the task explicitly modifies it
