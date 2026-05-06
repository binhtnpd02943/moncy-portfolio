# Source Discovery And Containerization

Use this file when the input is a cloned source repository and the user expects the skill to infer the stack before generating CI/CD or container files.

## Discovery Order

1. Identify monorepo vs single app
2. Identify runtime and framework markers
3. Identify build command
4. Identify startup command
5. Identify service dependencies
6. Check for existing container and CI/CD assets
7. Choose packaging model

Do not skip the startup-command step. Many incorrect Dockerfiles come from guessing how the app starts.

## Stack Marker Map

### Node.js / JavaScript / TypeScript

- `package.json` means Node-based project
- `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json` indicate package manager
- Next.js markers: `next` dependency, `next.config.*`, `app/` or `pages/`
- Nuxt markers: `nuxt` dependency, `nuxt.config.*`
- React SPA markers: `react` + Vite or CRA style scripts
- Vue SPA markers: `vue` + Vite or Vue CLI markers
- Angular markers: `angular.json`
- NestJS markers: `@nestjs/core`
- Express markers: `express`

### Java

- `pom.xml` means Maven
- `build.gradle` or `gradle.properties` means Gradle
- Spring Boot markers: `spring-boot-starter-*`, `@SpringBootApplication`
- confirm artifact location before writing copy commands

### Python

- `requirements.txt`, `pyproject.toml`, `poetry.lock`, `Pipfile`
- FastAPI markers: `fastapi`, `uvicorn`
- Django markers: `manage.py`, `django`
- Flask markers: `flask`, `gunicorn`
- generic ASGI/WSGI should remain explicit if startup command is unclear

### Go

- `go.mod`
- detect main package and binary output path

### PHP

- `composer.json`
- Laravel markers: `artisan`, `config/app.php`, `public/index.php`
- generic PHP app requires explicit runtime decision

## Dependency Discovery

Look for evidence of infrastructure dependencies in:

- `.env.example`
- `application.yml`, `application.properties`
- `config/*.php`
- `docker-compose.yml`
- `docker-compose.stage.yml`
- `docker-compose.prod.yml`
- `package.json` scripts
- source constants or config modules

Typical services to detect:

- Redis
- PostgreSQL
- MySQL
- MongoDB
- RabbitMQ
- Kafka
- Elasticsearch
- MinIO / S3 compatible storage

Only add a service to compose if one of these is true:

- the repository clearly needs it to boot locally
- the user asked for a prod-like local sandbox
- no managed external endpoint is available for the intended environment

## Packaging Decision Guide

### Generate only Dockerfile

Use when:

- single app
- external managed database or cache expected
- production target likely runs one app container or one image per service

### Generate Dockerfile + docker-compose

Use when:

- app plus at least one required side service for sandbox or shared-host deployment
- monorepo with multiple services that must start together
- the target environment already standardizes on Docker Compose

### Do Not Generate New Container Files

Use when:

- repository already has correct and current Docker assets
- user only asked for CI/CD review or deploy workflow refinement

## Required Output From Discovery

Every good result should name:

- detected stack
- evidence files
- chosen build command
- chosen runtime command
- detected service dependencies
- whether container files existed before
