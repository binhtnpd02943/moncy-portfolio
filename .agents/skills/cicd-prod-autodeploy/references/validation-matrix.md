# Validation Matrix

Use this file after generating or editing deploy artifacts.

Rule:

- validate as specifically as possible before broadening scope
- choose checks that match the artifacts actually generated
- do not claim completion only from visual inspection when a cheap validator exists

## GitHub Actions

Artifacts:

- `.github/workflows/*.yml`

Preferred checks:

- compare workflow keys and expressions against current GitHub docs
- use `actionlint` if available
- check `concurrency`, `permissions`, `needs`, and trigger branches

Minimum fallback:

- YAML parses
- references to jobs, outputs, and secrets are internally consistent

## GitLab CI

Artifacts:

- `.gitlab-ci.yml`

Preferred checks:

- GitLab CI Lint
- pipeline simulation when available
- compare syntax with current GitLab YAML reference

Minimum fallback:

- YAML parses
- `stages`, `needs`, `rules`, and includes are internally coherent

## Dockerfile

Artifacts:

- `Dockerfile`
- `.dockerignore`

Preferred checks:

- build the image when feasible
- compare instructions against Dockerfile reference
- confirm runtime command, port, and health check match source evidence

Minimum fallback:

- copy paths, entrypoint, and exposed port are evidence-backed
- multi-stage copy paths are internally consistent

## Docker Compose

Artifacts:

- `docker-compose.yml`
- `docker-compose.stage.yml`
- `docker-compose.prod.yml`

Preferred checks:

- `docker compose -f <file> config -q`
- merged config render with all intended files
- confirm service names, networks, and env interpolation

Minimum fallback:

- service graph is coherent
- no unintended local-only mounts or floating production image references

## Shell Scripts

Artifacts:

- `deploy.sh`
- `rollback.sh`
- `healthcheck.sh`
- `backup.sh`

Preferred checks:

- `bash -n`
- `shellcheck` if available
- dry-run or temp-path simulation when feasible

Minimum fallback:

- required env vars and binaries are validated
- health check and rollback paths are explicit
- script scope touches only the intended service or release set

## Production-Safety Checks

Always confirm:

- immutable artifact, tag, or digest exists
- health verification exists
- rollback path exists
- migration policy is explicit
- deploy serialization exists
- unrelated services are not restarted without a stated reason
- secret handling is externalized
