# Anti-patterns

Read this before finalizing CI/CD, Docker, or deploy output.

## Container Anti-patterns

- single-stage Dockerfile that ships compilers, build caches, and test tools into production
- production image running as root without justification
- copying the entire repository into runtime when only built output is needed
- using `latest` as the only deploy reference
- bundling database state into the app image
- assuming SSR apps can be served as static assets

## Compose Anti-patterns

- one ambiguous `docker-compose.yml` for both staging and production when settings differ materially
- using `docker-compose.yml` as the production file by accident when it is meant for local development
- production compose with local bind mounts to source code
- production compose starting local Postgres or Redis by default when managed services are expected
- missing health checks for primary services
- no explicit image tag or digest in production

## Workflow Anti-patterns

- deployment directly from developer laptops
- no separation between CI validation and deploy steps
- production deploy on every branch push
- missing rollback path
- no branch policy check for `stg -> main`
- provider-specific workflow with missing secret source strategy
- confusing credential fields, such as mixing account ID with access key ID

## Governance Anti-patterns

- allowing `Write` role to bypass `main` protection broadly
- allowing direct push to `main`
- treating PO/circle-lead policy as prose without mapping to real users or teams

## Selection Anti-patterns

- defaulting to Kubernetes because it sounds advanced
- defaulting to cloud-native services without checking team capability
- generating startup commands without evidence from source
