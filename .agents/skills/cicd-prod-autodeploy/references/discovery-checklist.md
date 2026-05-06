# Discovery Checklist

Use this before proposing a deployment design for a new environment that already hosts other systems.

## 1. Host and Runtime

- OS and version
- CPU, RAM, disk headroom
- runtime stack: Java, Node.js, Python, PHP, Docker, Compose, systemd, Kubernetes
- package manager and shell availability

## 2. Existing Systems on the Same Environment

- running services and service names
- open ports
- domains, subdomains, and reverse proxy routes
- shared directories
- scheduled jobs and background workers
- existing monitoring or alerting

## 3. Delivery Mechanics

- how code or artifact reaches the server
- whether the server can build locally or must receive a prebuilt artifact
- whether a CI runner already exists
- whether SSH access is allowed from CI
- whether the deployment target is already fixed by the project or customer
- which registry is expected: ECR, ACR, Artifact Registry, Docker Hub, private registry
- which cloud account / subscription / project / region is in scope

## 4. Safety Controls

- current backup method
- current rollback method
- acceptable downtime
- database migration policy
- health check endpoint or smoke test command

## 5. Secrets and Access

- where secrets live: GitHub Secrets, GitLab CI Variables, Vault, SSM, `.env` on host
- who owns merge permission to production
- who owns server access
- how CI authenticates to the target provider
- whether runtime secrets are pulled on-host or injected at deploy time

## Minimum Assumption Set When Details Are Missing

If the user cannot provide environment details, state these assumptions explicitly:

- Linux VM target
- dedicated deploy user
- systemd or Docker Compose service control
- SSH-based deployment from CI
- release directories under `/opt/<app>/releases`
- reverse proxy already exists and must not be modified automatically
