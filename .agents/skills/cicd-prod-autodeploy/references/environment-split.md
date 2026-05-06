# Environment Split

Use this file when the team wants clear separation between staging and production.

## Default Recommendation

Prefer explicit environment assets:

- `docker-compose.yml` for local development
- `docker-compose.stage.yml`
- `docker-compose.prod.yml`

Use one shared base file only when the overlap is large and the resulting structure remains obvious.

## Why Separate Staging And Production

Clear separation helps because:

- domains, ports, replicas, secrets, and external service endpoints are usually different
- local developer needs are different from deployment environment needs
- staging may allow lower resources or mock integrations
- production should not inherit debug flags, bind mounts, or dev-only helpers
- reviewers can see environment intent directly from filenames

## When To Use A Shared Base File

Consider:

- `docker-compose.yml`
- `docker-compose.base.yml`
- `docker-compose.stage.yml`
- `docker-compose.prod.yml`

Only if:

- service topology is almost identical
- differences are mostly env vars, image tags, or small overrides

Avoid a base file if it makes the final deploy harder to read.

## Staging Defaults

Staging may reasonably include:

- smaller worker count
- sandbox domains
- non-production image tags
- optional debug logging
- disposable database or cache containers when the target is not using managed services

## Production Defaults

Production should prefer:

- explicit image tags or digests
- externalized secrets
- managed external data services when the environment provides them
- no source bind mounts
- strict restart policy
- health checks and rollback-ready deploy logic

## Compose File Scope

### `docker-compose.yml`

Good for:

- local coding
- local integration tests
- developer-run sandbox dependencies

It may include developer conveniences that should never appear in production, such as bind mounts or debug settings.

### `docker-compose.stage.yml`

Good for:

- staging deployment
- sandbox validation
- previewing container interactions before prod

### `docker-compose.prod.yml`

Good for:

- real production deploy on VM/Compose hosts
- pulling image from registry
- referencing external production secrets and endpoints

It should not contain developer conveniences such as local bind mounts or ad hoc debug commands.
