# Contributing to camunda-cockpit-sso

Thanks for your interest in improving this project! This guide takes you from a
fresh clone to a merged pull request.

## Prerequisites

- **Java 17** (see [`.java-version`](.java-version))
- **Docker / Podman** with Docker Compose (for the local SSO stack)

> Maven does not need to be installed — use the bundled
> [Maven Wrapper](https://maven.apache.org/wrapper/) (`./mvnw`), which downloads
> the pinned Maven version on first use. (`mvnw.cmd` on Windows.)

## Getting started

```bash
git clone https://github.com/Miragon/camunda-cockpit-sso.git
cd camunda-cockpit-sso

# Build all modules
./mvnw -B verify
```

### Running locally

The example service plus the local Keycloak stack let you test changes end-to-end.
First add the `keycloak` host alias described in the [README](README.md#quickstart-local-development), then:

```bash
# 1. Start Keycloak + PostgreSQL and seed the test realm/user
docker compose -f sso-stack/docker-compose.yml up -d

# 2. Load the matching SSO settings and run the example service
set -a && source sso-stack/local-docker.env && set +a
./mvnw -pl cockpit-sso-service -am spring-boot:run
```

Open <http://localhost:8082> and log in as `johndoe` / `test`.

## Inner loop

| Task | Command |
|---|---|
| Build & verify | `./mvnw -B verify` |
| Build a single module | `./mvnw -pl cockpit-sso-starter -am verify` |
| Run the example app | `./mvnw -pl cockpit-sso-service -am spring-boot:run` |

> **Note:** there is no automated test suite yet. Contributions that add tests are
> especially welcome. Until then, please verify changes manually against the local
> SSO stack and describe how you tested them in your PR.

## What CI enforces

Every pull request runs the [CI workflow](.github/workflows/ci.yml), which builds
all modules with `./mvnw -B verify`. Make sure it passes locally before opening a PR.

## Commit convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/).
Examples:

```
feat: gate REST API access behind the web-app role
fix: build https redirect URL behind a reverse proxy
docs: document the web-app-role property
refactor: extract token parsing into its own service
```

## Pull request process

1. Fork the repository and create a branch: `<type>/<short-description>`
   (e.g. `feat/multi-provider-login`).
2. Make your change and ensure `./mvnw -B verify` passes.
3. Open a pull request against `main` using the PR template. Link any related issue.
4. A code owner (see [`.github/CODEOWNERS`](.github/CODEOWNERS)) will review. Address
   feedback by pushing follow-up commits.

## Reporting issues

Please use the issue forms in the [issue tracker](https://github.com/Miragon/camunda-cockpit-sso/issues/new/choose):
a bug report, feature request, or refactor proposal. For security vulnerabilities,
**do not open a public issue** — see [SECURITY.md](SECURITY.md).

## Questions

Not sure about something? Open a feature/refactor issue to start a discussion, or
ask in your pull request.
