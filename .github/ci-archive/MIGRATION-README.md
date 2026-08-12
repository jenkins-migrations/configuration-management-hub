# Jenkins to GitHub Actions migration report

## Source inventory

| Jenkins source | Replacement workflow |
| --- | --- |
| `Jenkinsfile` | `.github/workflows/node-docker-deploy.yml` |
| `dockercomposedeploy/Jenkinsfile` | `.github/workflows/compose-production.yml` |
| `whenconditions/Jenkinsfile` | `.github/workflows/branch-conditions.yml` |

The original Jenkinsfiles are retained in this directory for audit history and removed from their former locations.

## Migration summary

- The Node.js pipeline retains dependency installation, testing, build output, Docker image creation, registry push, Kubernetes deployment, and cleanup. Test and build artifacts replace Jenkins result publishing and artifact archival.
- The Compose pipeline retains the `swarm-ci` and `swarm-prod` runner requirements, test commands, release-version computation, annotated tag, image publish, Swarm deployment, and Slack notifications. GitHub Actions checks replace Bitbucket status notifications.
- `waitUntilServicesReady`, whose shared-library implementation was not included in the repository, is expanded as an inline 60-second poll that requires every Compose service to reach the running state. Configure Compose health checks if readiness requires stronger guarantees.
- The branch demonstration preserves the `master` and non-`master` conditions. Its Jenkins expression always returned false, so no equivalent executable Actions step is emitted.
- The version tool is pinned to `softonic/ci-version@sha256:c42462a455293199629426dc87bf78c01ea348d4b1642a0f3e919ba3b1cf52b3`. All Actions are pinned to immutable commit SHAs.

## Required repository configuration

### Runners

- A hardened self-hosted runner labeled `node` with Node.js, Docker, and `kubectl`.
- A hardened self-hosted runner labeled `swarm-ci` with Docker Engine and Docker Compose v2.
- A protected self-hosted Swarm manager labeled `swarm-prod` with Docker Engine, Docker Compose v2, and production-cluster access.

Do not use these privileged runners for untrusted pull-request code.

### Secrets

| GitHub secret | Replaces | Use |
| --- | --- | --- |
| `REGISTRY_USERNAME` | `docker-registry` username | Docker registry authentication |
| `REGISTRY_PASSWORD` | `docker-registry` password | Docker registry authentication |
| `KUBECONFIG_B64` | Jenkins runner Kubernetes context | Base64-encoded Kubernetes config for the Node deployment |
| `SLACK_WEBHOOK_URL` | `slack_token` | Deployment notifications |

Create a protected `production` environment, restrict these secrets to it, require reviewers, and configure the deployment job to use that environment before enabling production deployments. Prefer OIDC-based registry and Kubernetes authentication where the platform supports it.

### Permissions and behavior

- Normal jobs use `contents: read`; the release job alone uses `contents: write` to create tags, replacing the Jenkins SSH credential.
- Registry pushes and Kubernetes deployment run only for pushes to `master`. The privileged workflows do not run pull-request code; validate pull requests with a separate unprivileged workflow if required.
- GitHub artifact uploads retain JUnit, Clover, and `dist` outputs. GitHub Actions does not provide an equivalent to Jenkins artifact fingerprinting.

## Validation

The workflows were checked with `actionlint`. Functional deployment validation requires the configured self-hosted runners, repository secrets, registry, Kubernetes cluster, and Swarm environment.
