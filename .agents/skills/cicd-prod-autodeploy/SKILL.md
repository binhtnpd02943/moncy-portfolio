---
name: cicd-prod-autodeploy
description: Assess, harden, or design CI/CD and auto-deploy for shared staging and production environments, including source-stack discovery, safe containerization, environment-specific compose assets, immutable artifact or image promotion, target-specific deploy workflows, deploy and rollback scripts, branch governance, and production deployment when `stg` is promoted into `main` or `master`.
---

# CI/CD Prod Autodeploy

Build the smallest safe delivery workflow first. Production automation is allowed only after environment collision risks, rollback, and merge governance are defined.

## Use This Skill When

- The user wants to set up or rebuild CI/CD for `stg`, `main`, `master`, or production branches.
- The deployment target is a shared server or shared environment where other systems are already running.
- The input is a cloned source repository and the user wants the skill to inspect the source before deciding the delivery model.
- The source may not have Docker configuration yet and the user wants AI to generate `Dockerfile` and `docker-compose` safely.
- The user wants AI to generate shell scripts for deployment, backup, rollback, health check, or service restart.
- The user needs branch protection rules such as "no direct push to main" or limited merge permissions.
- The repository already has CI/CD or deploy scripts and the user wants an objective review, hardening plan, or surgical fixes instead of a full rewrite.
- The team needs a safer release contract around image tags, artifact promotion, secrets, database migration handling, or rollback behavior.

## Read Only What You Need

- Read [references/discovery-checklist.md](references/discovery-checklist.md) first when the environment is unclear or shared.
- Read [references/workflow-blueprint.md](references/workflow-blueprint.md) when designing the end-to-end CI/CD flow.
- Read [references/review-rubric.md](references/review-rubric.md) when auditing existing CI/CD, deploy scripts, or release readiness.
- Read [references/platform-rules.md](references/platform-rules.md) when generating GitHub or GitLab settings and policies.
- Read [references/deploy-script-patterns.md](references/deploy-script-patterns.md) when writing shell scripts.
- Read [references/source-discovery-and-containerization.md](references/source-discovery-and-containerization.md) when the input is an existing source repository and the stack must be inferred.
- Read [references/runtime-matrix.md](references/runtime-matrix.md) when generating `Dockerfile` or `docker-compose` for backend or frontend stacks.
- Read [references/environment-split.md](references/environment-split.md) when deciding how to separate staging and production assets.
- Read [references/cloud-deploy-targets.md](references/cloud-deploy-targets.md) when generating provider-specific deployment workflows.
- Read [references/official-source-map.md](references/official-source-map.md) when provider, CI, Docker, or registry syntax might have changed and the answer must come from current official docs rather than memory.
- Read [references/deployment-decision-tree.md](references/deployment-decision-tree.md) when choosing between VM, Compose, AWS, Azure, GCP, or other delivery models.
- Read [references/trade-off-analysis.md](references/trade-off-analysis.md) when the user or leader needs rationale, comparison, or ADR-style decisions.
- Read [references/validation-matrix.md](references/validation-matrix.md) when validating generated YAML, Docker assets, or deploy scripts before calling the result complete.
- Read [references/anti-patterns.md](references/anti-patterns.md) before finalizing container or CI/CD output for production.
- Read [references/examples.md](references/examples.md) when you need maturity-level examples or reference output patterns.
- Read [examples/local-demo-input.md](examples/local-demo-input.md) and [examples/local-demo-expected-output.md](examples/local-demo-expected-output.md) when you need a local demonstration scenario or a reference output shape.
- Read [examples/review-mode-input.md](examples/review-mode-input.md) and [examples/review-mode-expected-output.md](examples/review-mode-expected-output.md) when testing audit or hardening mode against an existing production workflow.
- Read [examples/aws-ec2-input.md](examples/aws-ec2-input.md) and [examples/aws-ec2-expected-output.md](examples/aws-ec2-expected-output.md) when the target is fixed to AWS EC2 and the team wants a concrete source-driven prompt and expected result shape.

## Default Working Model

## Reference-First Rule

For mutable syntax and platform options, prefer current official docs, schemas, or validators over memory.

Apply this especially to:

- GitHub Actions workflow keys, expressions, permissions, and concurrency
- GitLab CI keywords, expressions, includes, and lint behavior
- Dockerfile instructions and Compose syntax
- provider-specific registry login, deploy commands, and runner identity patterns

If the user asks what options are available for a fixed target, derive the answer from current official docs for that target whenever feasible. If you cannot verify, label the uncertainty explicitly and choose the safest conservative pattern.

Assume this branch model unless the repository already uses another stable convention:

- `feature/*` -> merge into `stg`
- `stg` -> validates integration and pre-production checks
- `stg` -> merge request / pull request into `main`
- merge into `main` -> auto deploy production

Assume direct push to `main` is forbidden.

Assume only these roles can approve and merge into `main`:

- Product Owner of the project
- Circle lead / technical lead explicitly designated by the team

If the actual hosting stack is not stated, prefer a neutral Linux VM model with:

- source checkout or artifact fetch
- release directory strategy
- symlink switch
- systemd service restart or container compose restart
- health check and rollback

Assume the first responsibility is to inspect the repository and classify it into one of these delivery shapes:

- backend service
- frontend SPA
- frontend SSR app
- multi-service monorepo
- fullstack app with embedded frontend

Assume staging and production are independent deployment environments unless the repository or user explicitly says otherwise.
Assume `docker-compose.yml` is the default local development file when compose is needed for developer workflows.
Prefer promoting the same tested artifact or image digest from staging to production when the workflow and platform allow it. If production must rebuild on `main`, explain why artifact drift is acceptable and how dependencies remain pinned.

Deployment target priority rule:

- if the user or repository already specifies a deployment target such as AWS EC2, AWS ECS, Azure App Service, Azure VM, GCP Cloud Run, or VM Docker Compose, treat that target as fixed
- do not redesign the solution for another provider or platform unless the user explicitly asks for alternatives
- only compare multiple target shapes when the target is genuinely unknown

## Execution Process

First choose the operating mode:

- review and hardening mode: audit the existing pipeline, Docker assets, deploy scripts, permissions, and rollout behavior; keep working parts and patch the highest-risk gaps first
- greenfield mode: design the smallest safe workflow and generate only the missing assets

### 1. Survey the Existing Environment First

Never jump straight to pipeline YAML.

Identify at minimum:

- application runtime and build stack
- target host type: VM, container host, Kubernetes, PaaS
- target provider if already known: AWS, Azure, GCP, on-prem VM, private cloud
- current running systems on the same host
- occupied ports, domains, reverse proxy rules, shared volumes, service names
- deployment identity: SSH user, runner identity, secrets source
- rollback mechanism currently available, if any
- downtime tolerance, RTO/RPO expectations, and whether database rollback is realistically possible

If the user gives only policy requirements and no environment detail, continue by producing a safe default design and clearly label assumptions.

### 2. Inspect the Source Repository Before Choosing the Delivery Model

Never assume the stack from folder names alone.

Inspect at minimum:

- runtime markers such as `package.json`, `pom.xml`, `build.gradle`, `requirements.txt`, `pyproject.toml`, `go.mod`, `composer.json`
- frontend framework markers such as Next.js, Nuxt, React, Vue, Angular
- backend framework markers such as Spring Boot, FastAPI, Django, Express, NestJS, Gin, Laravel
- existing infra markers such as `Dockerfile`, `docker-compose.yml`, `docker-compose.stage.yml`, `docker-compose.prod.yml`, `.github/workflows`, `.gitlab-ci.yml`, `Procfile`, Helm charts
- data/service dependencies such as Redis, PostgreSQL, MySQL, MongoDB, RabbitMQ, Kafka, MinIO
- runtime ports, static asset output directories, health endpoint conventions, migration commands

Always determine:

- is the repo already containerized
- is the repo single-service or multi-service
- does it need build-time dependencies distinct from runtime dependencies
- is it safer to deploy as a single container, a compose stack, or a VM release directory
- which repository, module, or directory should own deploy artifacts such as pipeline YAML, `Dockerfile`, `docker-compose*.yml`, Helm charts, or host-side scripts
- whether the rollout should target one changed service only or a coordinated multi-service release

If Docker assets already exist, audit and refine them before generating new ones.
If Docker assets do not exist, generate them only after the stack and service dependencies are explicit.
If pipeline YAML or deploy scripts already exist, classify each artifact as keep, patch, or replace and explain why.

### 3. Define Asset Ownership And Deployment Scope

When the source contains multiple services, multiple repositories, or mixed app and infra folders, define ownership before generating files.

Decide explicitly:

- which repo or directory owns the application image build
- which repo or directory owns `docker-compose*.yml` or other runtime manifests
- which repo or directory owns deploy scripts
- which repo or directory owns environment-specific pipeline files
- whether a service can deploy independently or whether it must release together with other services

Default rule:

- generate shared infra assets only in the repo or directory that actually owns shared runtime behavior
- avoid duplicating the same deploy contract across multiple repos unless the platform truly requires it
- on shared hosts or Compose targets, update only the changed service unless a shared manifest, shared base image contract, migration contract, or tightly coupled release dependency requires a broader rollout

### 4. Decide the Containerization Strategy

Prefer the smallest correct packaging model.

Choose one:

- no containerization if the repo already has a stable VM-native release model and the user did not ask to containerize it
- one `Dockerfile` for a single backend or SPA service
- one application `Dockerfile` plus environment-specific compose files when the app depends on local services such as Redis or database containers for staging or prod-like testing
- multi-service compose only when the repository actually contains multiple cooperating services

Container generation rules:

- use multi-stage builds by default for backend services and SSR apps unless the stack is too trivial to justify it
- keep runtime images minimal
- do not bundle database data into the application image
- do not force Redis, database, or broker into compose if the target environment already provides managed services; support env-based external connections
- provide `.dockerignore` when a new `Dockerfile` is generated
- expose only the required port
- include a health check path or command when the stack supports it

Multi-stage rationale must be explained concretely, not with "smaller image" alone. Always mention the real engineering benefits that apply:

- build toolchain stays in the builder layer and does not ship to runtime
- fewer OS packages and compilers in runtime reduce attack surface
- dependency caching makes rebuilds faster and more stable
- runtime image becomes more deterministic and easier to scan
- builder and runtime responsibilities are separated, which is clearer for CI/CD

If the repository uses Python, Java, Node.js, Go, PHP, or SSR frontend, prefer explaining the builder/runtime split using the actual framework and dependency model of that stack.

### 5. Split Environment Assets Clearly

When the team wants independent staging and production environments, prefer explicit files over one overly generic compose file.

Default expectation:

- `docker-compose.yml` for local development
- `docker-compose.stage.yml`
- `docker-compose.prod.yml`

Use a shared base compose file only when it genuinely reduces duplication without making environment intent ambiguous.

Environment split rules:

- `docker-compose.yml` is for local coding, local integration, and developer-run testing
- staging may include debug-oriented settings, lower replica count, or sandbox dependencies
- production must avoid dev-only mounts, debug flags, and unsafe defaults
- if staging and production use different external databases, caches, domains, or image tags, reflect that explicitly
- if production uses managed cloud services, `docker-compose.prod.yml` should point to external endpoints instead of starting local data services by default
- if the app needs workers, schedulers, or queue consumers, decide whether they exist in both `stg` and `prod`

Evaluation rule:

- when testing this skill, the primary CI/CD assessment should focus on `stage` and `prod`
- `docker-compose.yml` is considered local developer support, not the main production automation artifact

Frontend-specific rules:

- React/Vue/Angular SPA: prefer static build served by `nginx` or another simple web server
- Next.js/Nuxt SSR: treat as application runtime, not static-only, unless the source is explicitly configured for static export

Backend-specific rules:

- Java: detect Maven vs Gradle, JAR vs WAR, and runtime version before writing the image
- Python: detect FastAPI, Django, Flask, or generic WSGI/ASGI shape before choosing the start command
- Node.js: detect Express, NestJS, Next.js, Nuxt, or custom server before selecting the runtime command
- Go: prefer compiled static binary flow
- PHP: detect Laravel or generic PHP app before choosing `php-fpm`, Apache, or artisan-related setup
- Unknown stack: report unsupported or ambiguous markers instead of inventing a startup command

### 6. Decide the Deployment Strategy

Prefer the simplest strategy that avoids impacting other systems on the same environment.

Use [references/deployment-decision-tree.md](references/deployment-decision-tree.md) when the correct target shape is not obvious.

Choose one:

- in-place service restart only if downtime is acceptable and rollback is trivial
- release directories + symlink switch for VM-based apps
- blue/green or canary only when downtime risk is unacceptable or multiple services must stay isolated
- Docker Compose service replacement when the host already standardizes on Compose

Explicitly explain why the chosen strategy is safer in a shared environment.
If the deploy target hosts multiple services, explain whether the rollout touches only one service or the full release set, and why.

### 7. Decide the Deployment Target Shape

Do not generate one generic deploy workflow for every project. Match the workflow to the actual target:

- VM with Docker Compose
- VM with systemd and release directories
- AWS ECR -> EC2 / ECS
- Azure Container Registry -> VM / App Service / Container Apps
- GCP Artifact Registry -> Compute Engine / Cloud Run
- private registry + SSH deploy

If the target is explicitly given by the user, repository, or team standard, treat this step as target refinement, not target reselection.
If the user gives no provider, generate a neutral VM/Compose flow and clearly label the provider section as assumption-based.
If multiple target shapes are plausible, compare them briefly and choose one using an ADR-style rationale.
Before finalizing provider-specific YAML or commands, check the current official references listed in [references/official-source-map.md](references/official-source-map.md) when the syntax or service behavior may have changed.

### 8. Define the CI/CD Contract

The generated solution should usually include:

- CI checks on feature and `stg`
- one immutable artifact or image reference tied to commit SHA, release ID, or digest
- promotion of the same tested artifact or image digest from staging to production when feasible
- protected merge path into `main`
- build or verify generated container assets when containerization is part of the solution
- a distinct staging deploy path if `stg` is intended to auto-deploy to staging
- production deploy job triggered only by merge into `main`
- one active production deploy at a time through pipeline concurrency control or environment locking
- manual approval gate only if the user still wants human confirmation before prod
- post-deploy verification
- rollback command or rollback job
- service-specific deploy scope when the repository or platform allows partial rollout

Always specify:

- trigger branch
- runner or execution location
- artifact path
- secret sources
- deploy steps
- health checks
- rollback steps
- release identity such as image tag, digest, or release directory ID
- whether credentials are short-lived runner identity, OIDC, or static secrets and why
- asset ownership map for pipeline, runtime manifests, and deploy scripts
- deploy scope: changed service only, release group, or full stack

When the source repo is the input, the CI/CD contract should also state:

- which files were detected as stack markers
- which runtime command was inferred
- whether `Dockerfile` already existed or was newly generated
- whether `docker-compose.stage.yml` and `docker-compose.prod.yml` are both required
- whether local developer support also needs `docker-compose.yml`
- which cloud or host target the deploy workflow is generated for

For significant choices, include a compact decision record:

- context
- options considered
- chosen option
- trade-offs accepted

### 9. Generate Delivery Assets

Produce the smallest set of concrete assets needed by the detected stack:

- `Dockerfile`
- `.dockerignore`
- `docker-compose.yml` when local developer workflows need compose support
- `docker-compose.stage.yml` when staging is containerized
- `docker-compose.prod.yml` when production uses Compose
- optional shared `docker-compose.base.yml` only when it reduces duplication cleanly
- `.github/workflows/*.yml` or `.gitlab-ci.yml`
- branch policy workflow when `main` must only accept PRs from `stg`
- `deploy.sh`, `rollback.sh`, `healthcheck.sh`, `backup.sh` as needed

Do not generate compose files that silently include infrastructure the app does not actually use.
Do not force a single compose file to represent both staging and production if that makes secrets, domains, ports, service counts, or dependencies ambiguous.

### 10. Generate Deploy Shell Scripts

When writing shell scripts:

- use `bash` with `set -Eeuo pipefail`
- make scripts idempotent where possible
- fail fast on missing env vars, paths, or binaries
- separate concerns into scripts such as `deploy.sh`, `rollback.sh`, `healthcheck.sh`, `backup.sh`
- avoid hardcoded secrets
- log each step with short clear messages
- use deterministic release identifiers, image tags, or image digests
- guard against concurrent deploy execution with pipeline concurrency or host-side locking when the platform can race
- protect shared-host resources by validating service name, port, and target directory before modifying anything
- restart, replace, or rebuild only the intended service unless the workflow explicitly justifies a broader coordinated release

If the repository already contains deploy scripts, refine them instead of replacing them blindly.

### 11. Handle Migrations And Stateful Changes Safely

State changes are a separate risk from application rollout.

Default rules:

- run automatic database migrations only when they are explicitly approved and backward compatible
- prefer expand and contract migration strategy for zero or low-downtime production deploys
- treat destructive schema changes, backfills, and irreversible data rewrites as gated operations, not as the default auto-deploy step
- distinguish application rollback from data rollback; if schema rollback is not realistic, say so and require backup or restore planning
- if the app depends on Redis, database, queue, or object storage state, declare which parts are external dependencies and which parts the deploy may mutate

### 12. Enforce Governance Rules

When the user asks for branch restrictions, produce concrete repo rules, not generic advice.

Minimum governance for `main`:

- disable direct push
- require PR/MR from `stg`
- require at least one approval
- restrict merge permission to PO and circle lead
- require passing CI status checks before merge
- optionally require signed commits or linear history if the team already uses them

If the platform does not support role names like "PO" directly, map them to concrete usernames, groups, code owners, or maintainers.

### 13. Output Structure

Prefer this order in the response:

1. assumptions and discovered constraints
2. source discovery result and inferred stack
3. environment split decision for `stg` and `prod`
4. containerization decision
5. deployment target choice
6. short ADR or trade-off rationale
7. asset ownership and deploy scope
8. proposed CI/CD flow
9. branch protection and approval rules
10. pipeline config and release identity strategy
11. Docker and compose assets
12. deploy shell scripts and migration policy
13. rollback and validation checklist

## Output Requirements

When implementing or proposing a solution, include these concrete artifacts whenever feasible:

- one source-stack summary with evidence from the repository
- one gap analysis with risk severity when the repository already contains CI/CD or deploy logic
- one asset ownership map when the source spans multiple modules or repos
- one branch flow description
- one pipeline file such as `.github/workflows/prod-deploy.yml` or `.gitlab-ci.yml`
- one `Dockerfile` when the repo is not already containerized and containerization is justified
- one `.dockerignore` alongside a new `Dockerfile`
- one `docker-compose.yml` for local development when compose improves local coding or local integration testing
- one `docker-compose.stage.yml` when staging is part of the requested automation
- one `docker-compose.prod.yml` when production uses Compose
- one provider-specific deploy workflow or deploy section, not just a generic local-only pipeline
- one branch-policy workflow or equivalent rule when `main` must only accept PRs from `stg`
- one short decision rationale for the chosen deploy target
- one immutable artifact, image tag, or image digest strategy
- one or more shell scripts under `scripts/` or `deploy/`
- one deploy concurrency rule or environment lock strategy for production
- one deploy scope rule stating when only the changed service is updated and when a broader rollout is required
- one migration policy when persistent data may be changed during deploy
- one short environment variable inventory
- one rollback path

Avoid vague answers like "set up CI/CD using best practices". Produce copyable configuration.
Include a validation plan tied to the generated artifacts and the tooling realistically available in the target environment.

## Constraints

- Do not propose direct deployment from a developer laptop to production as the main flow.
- Do not allow direct pushes to `main` unless the user explicitly overrides the rule.
- Do not overwrite shared nginx, shared ports, or shared service names without first checking collision risks.
- Do not embed secrets in repository files.
- Do not rely on `latest` or other floating image references in production when an exact tag or digest can be used.
- Do not choose Kubernetes by default when a single VM or Compose host is enough.
- Do not invent framework-specific startup commands without evidence from the source tree.
- Do not put databases or brokers into production compose by default if the environment likely uses managed external services.
- Do not treat SSR frameworks like static SPAs unless the repository explicitly supports static export.
- Do not collapse staging and production into one ambiguous compose file when the team wants independent environments.
- Do not generate AWS/Azure/GCP deploy steps unless the target provider is explicit or clearly labeled as an assumption.
- Do not choose a more complex platform like EKS/AKS/Kubernetes unless the scale, team, or platform constraints justify it.
- Do not auto-run destructive or irreversible database migrations in the default production deploy path.
- Do not use long-lived cloud root or personal credentials when workload identity, OIDC, or runner-attached identity is available.
- Do not allow concurrent production deploy jobs to race unless the workflow explicitly serializes them.
- Do not rebuild or restart unrelated services on shared hosts unless a shared manifest or coordinated release requirement makes it necessary.
- Do not duplicate ownership of shared runtime manifests across multiple repos without an explicit reason.
- Do not skip trade-off explanation for major decisions like Compose vs VM, EC2 vs ECS, App Service vs VM, or static SPA vs SSR runtime.

## Validation Checklist

- [ ] Shared-environment risks were identified or explicit assumptions were stated
- [ ] The repository stack was inferred from real source markers, not guesses
- [ ] Existing Docker assets were audited before generating new ones
- [ ] Asset ownership for pipeline, runtime manifests, and deploy scripts is explicit
- [ ] Generated Docker and compose files match the detected stack and dependencies
- [ ] Provider-specific YAML or deploy commands were checked against current official references when the syntax is mutable
- [ ] `stg` and `prod` environment assets are separated clearly when requested
- [ ] Multi-stage Dockerfile usage is justified by security, reproducibility, and runtime isolation, not only image size
- [ ] The deploy workflow matches the actual cloud or host target
- [ ] The release uses an immutable artifact, image tag, or image digest
- [ ] Production deploy serialization or concurrency control exists
- [ ] The rollout touches only the intended service set and does not restart unrelated workloads
- [ ] Major deployment choices have a clear decision rationale
- [ ] Known CI/CD and container anti-patterns were avoided
- [ ] Generated artifacts were validated with artifact-specific checks where tooling was available
- [ ] `main` direct push is blocked
- [ ] only authorized roles can merge to `main`
- [ ] merge from `stg` to `main` triggers production deployment
- [ ] deploy scripts are idempotent and fail fast
- [ ] migration behavior is explicit and destructive data changes are gated
- [ ] health check exists
- [ ] rollback path exists
- [ ] secret handling is externalized
