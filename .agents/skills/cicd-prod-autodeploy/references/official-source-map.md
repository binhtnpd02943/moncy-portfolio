# Official Source Map

Use this file when the request depends on mutable platform syntax, vendor-specific configuration, or current service behavior.

Rule:

- prefer official docs, schemas, or official validation tools over memory when generating or reviewing provider-specific CI/CD assets
- if the target is fixed, fetch or consult only the sources relevant to that fixed target
- if verification is not possible, say so explicitly and use the safest conservative syntax

## GitHub Actions

Use when generating or reviewing `.github/workflows/*.yml`.

- Workflow syntax: https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax
- Workflows and actions reference: https://docs.github.com/en/actions/reference/workflows-and-actions
- Concurrency: https://docs.github.com/en/actions/concepts/workflows-and-actions/concurrency

Check especially:

- trigger syntax
- `permissions`
- `concurrency`
- expressions and contexts
- reusable workflow shape

## GitLab CI

Use when generating or reviewing `.gitlab-ci.yml`.

- YAML syntax reference: https://docs.gitlab.com/ee/ci/yaml/
- CI/CD expressions: https://docs.gitlab.com/ci/yaml/expressions/
- CI Lint: https://docs.gitlab.com/ci/yaml/lint/

Check especially:

- `rules`
- `needs`
- `stages`
- `include`
- expression syntax
- lint or pipeline simulation availability

## Dockerfile

Use when generating or reviewing `Dockerfile`.

- Dockerfile reference: https://docs.docker.com/reference/builder

Check especially:

- instruction correctness
- multi-stage behavior
- `HEALTHCHECK`
- `ENTRYPOINT` and `CMD`
- build cache and secret handling

## Docker Compose

Use when generating or reviewing `docker-compose*.yml`.

- Docker Compose CLI reference: https://docs.docker.com/reference/cli/docker/compose/
- `docker compose config`: https://docs.docker.com/reference/cli/docker/compose/config/

Check especially:

- merged config behavior
- environment interpolation
- project naming
- profiles
- service, network, and volume resolution

## AWS ECR And EC2

Use when the target is fixed to AWS ECR or EC2.

- Push to ECR private repo: https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html
- ECR push overview and permissions context: https://docs.aws.amazon.com/en_us/AmazonECR/latest/userguide/image-push.html

Check especially:

- registry authentication flow
- exact image tag or digest handling
- least-privilege push permissions
- whether the runtime remains EC2 and should not drift into ECS or EKS

## How To Apply This Map

When asked for platform-specific options:

1. identify the fixed target
2. consult only the relevant official references
3. derive the answer from those references
4. validate generated artifacts with the best available tool for that artifact type
