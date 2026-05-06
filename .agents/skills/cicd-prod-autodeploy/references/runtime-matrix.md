# Runtime Matrix

Use this file when generating Docker assets after the repository stack has been detected.

## Backend

### Java

Detect first:

- Maven vs Gradle
- Java version if declared
- JAR vs WAR
- Spring Boot vs generic Java

Preferred pattern:

- builder image with Maven or Gradle
- runtime image with JRE only
- `java -jar app.jar` for Spring Boot fat JARs

Why multi-stage matters here:

- Maven or Gradle cache and compilers stay out of runtime
- final image contains only the built artifact and JRE
- vulnerability surface is lower than shipping build tooling with the app

Avoid:

- copying the whole source tree into runtime image
- assuming artifact name without reading build config

### Python

Detect first:

- package manager: pip, Poetry, Pipenv
- framework: FastAPI, Django, Flask, generic
- server process: uvicorn, gunicorn, manage.py, custom module

Preferred pattern:

- slim Python image
- install only needed system packages
- freeze runtime command from actual app entrypoint

Why multi-stage matters here:

- build-only packages such as `gcc` and header libraries stay in builder
- runtime can keep only shared libs like `libpq5`
- wheel or virtualenv content is copied cleanly into runtime

Avoid:

- using Flask development server in production
- inventing `app:app` if the module path is not proven

### Node.js Backend

Detect first:

- package manager
- framework: Express, NestJS, Next.js custom server, Nuxt server
- build output directory if TypeScript is compiled

Preferred pattern:

- multi-stage image
- build in one stage
- run with production deps only

Why multi-stage matters here:

- build dependencies and transpilation tools do not ship to runtime
- runtime image can keep only `node_modules` needed in production
- frontend or TypeScript build output becomes explicit and reproducible

Avoid:

- `npm start` when scripts clearly require `pnpm` or `yarn`
- assuming `dist/main.js` without checking build output

### Go

Detect first:

- main package path
- CGO requirements

Preferred pattern:

- multi-stage build
- small runtime image or distroless/static if compatible

Why multi-stage matters here:

- Go builder toolchain is large and unnecessary at runtime
- final runtime can be minimal or even distroless

### PHP

Detect first:

- framework: Laravel or generic
- web stack expectation: `php-fpm` or Apache
- queue or scheduler requirements

Preferred pattern:

- app image based on `php-fpm` for nginx-backed setups
- separate notes for queue worker and scheduler if needed

Why multi-stage matters here:

- Composer install environment can be separated from runtime
- production image can exclude build helpers and cache clutter

Avoid:

- treating Laravel as one-process-only if queue and scheduler are required for real operation

## Frontend

### React / Vue / Angular SPA

Detect first:

- build tool: Vite, CRA, Vue CLI, Angular CLI
- output directory

Preferred pattern:

- build stage with Node
- runtime stage with nginx or another static server

Why multi-stage matters here:

- Node build toolchain is not needed once static assets are produced
- runtime image can serve only the final built files

### Next.js / Nuxt SSR

Detect first:

- SSR vs static export configuration
- standalone/server output support

Preferred pattern:

- SSR: application runtime container
- static export: build then serve static files

Avoid:

- forcing nginx static hosting for SSR apps

Why multi-stage matters here:

- build toolchain and framework compilation stay in builder
- runtime keeps only standalone output or required server files

## Compose Modeling

When generating compose files, model services separately:

- `app`
- `redis`
- `db`
- `worker`
- `scheduler`
- `nginx`

Use explicit environment variables for external service endpoints so local dev, stage, and prod files can stay aligned without becoming identical.

## Minimum Docker Deliverables

When new container files are created, include:

- `Dockerfile`
- `.dockerignore`
- `docker-compose.yml` for local development when justified
- `docker-compose.stage.yml` when staging is containerized
- `docker-compose.prod.yml` when production uses Compose
- short env var list
- startup and healthcheck logic aligned with the real stack
