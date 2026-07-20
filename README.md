# camunda-cockpit-sso

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/Miragon/camunda-cockpit-sso/actions/workflows/ci.yml/badge.svg)](https://github.com/Miragon/camunda-cockpit-sso/actions/workflows/ci.yml)
[![Java 17](https://img.shields.io/badge/Java-17-blue.svg)](https://adoptium.net/)
[![Camunda 7.20](https://img.shields.io/badge/Camunda-7.20-orange.svg)](https://docs.camunda.org/manual/7.20/)
[![Spring Boot 3.1](https://img.shields.io/badge/Spring%20Boot-3.1-green.svg)](https://spring.io/projects/spring-boot)

A Spring Boot starter that adds **Single Sign-On (OAuth2 / OIDC)** to the
**Camunda 7 web applications** — Cockpit, Tasklist and Admin.

Out of the box, the Camunda web apps use a form-based login against the engine's
own user database. This starter replaces that with a standard OAuth2/OIDC login
flow against an external identity provider such as **Keycloak**, so your Camunda
web apps participate in your organisation's SSO like any other application.

## Why use it

- **No custom security code.** Drop in the starter, point it at your identity
  provider via standard Spring `spring.security.oauth2.*` properties, and the web
  apps are protected by OIDC.
- **Role-gated access.** Only users carrying a configurable client role
  (`application.web-app-role`) may reach the web apps.
- **Engine integration.** Bridges the OAuth2 principal into Camunda's
  `ContainerBasedAuthenticationFilter` and ships a read-only identity provider, so
  the engine sees the authenticated user without maintaining a separate user store.
- **Reverse-proxy aware.** Honours `X-Forwarded-*` headers so OAuth2 redirect URLs
  are built correctly behind HTTPS termination (e.g. an OpenShift/Ingress route).

## How it works

The `cockpit-sso-starter` configures a Spring Security filter chain that:

1. Redirects unauthenticated web-app requests to the identity provider
   (`oauth2Login`) and validates bearer tokens on the REST API
   (`oauth2ResourceServer`).
2. Requires the role configured in `application.web-app-role` for `/app/**`,
   `/api/**`, `/assets/**` and `/lib/**`.
3. Maps the token's roles into Spring `GrantedAuthorities` and hands the
   authenticated user to Camunda through a container-based authentication provider.

See [`CamundaWebAppsSecurityConfiguration`](cockpit-sso-starter/src/main/java/io/miragon/camunda/sso/config/CamundaWebAppsSecurityConfiguration.java)
for the full filter chain.

## Requirements

- Java 17
- An OAuth2/OIDC identity provider (Keycloak in the examples)
- Docker / Podman + Docker Compose (for the local development stack)

> Maven itself does **not** need to be installed — the repository ships the
> [Maven Wrapper](https://maven.apache.org/wrapper/) (`./mvnw`), which downloads
> the pinned Maven version on first use.

## Modules

| Module | Description |
|---|---|
| [`cockpit-sso-starter`](cockpit-sso-starter) | The reusable Spring Boot starter — all the security/identity glue. Add this to your own Camunda web-app application. |
| [`cockpit-sso-service`](cockpit-sso-service) | A runnable example application that uses the starter. Handy as a reference and for local testing. |
| [`sso-stack`](sso-stack) | A local Keycloak + PostgreSQL development environment (dev only, **not** for production). |

## Quickstart (local development)

The example service (`cockpit-sso-service`, port `8082`) together with the local
`sso-stack` gives you a working SSO-protected Camunda in a few steps.

> **Host alias required.** The stack references Keycloak by the hostname
> `keycloak`. Add it to your hosts file so the same issuer URL resolves both
> inside Docker and on your machine:
> ```
> 127.0.0.1 localhost keycloak
> ```
> (`/etc/hosts` on macOS/Linux, `C:\Windows\System32\Drivers\etc\hosts` on Windows.)

1. **Start the identity provider** (Keycloak, PostgreSQL, and a migration job that
   seeds the `testrealm`, an `engine` client and a `johndoe` test user):

   ```bash
   docker compose -f sso-stack/docker-compose.yml up -d
   ```

2. **Run the example service** with the matching SSO settings:

   ```bash
   set -a && source sso-stack/local-docker.env && set +a
   ./mvnw -pl cockpit-sso-service -am spring-boot:run
   ```

3. **Open the web apps** at <http://localhost:8082> and log in through Keycloak:

   | Username | Password |
   |---|---|
   | `johndoe` | `test` |

## Using the starter in your application

> **Status:** the current version is `1.0.0-SNAPSHOT` and is **not yet published to
> Maven Central**. Build and install it locally with `./mvnw install` until a release
> is available.

Add the starter to your Camunda Spring Boot web-app project:

```xml
<dependency>
    <groupId>io.miragon.camunda</groupId>
    <artifactId>cockpit-sso-starter</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>
```

Then configure your identity provider with the standard Spring Security OAuth2
properties plus the two starter-specific properties:

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${SSO_BASE_URL}/realms/${SSO_REALM}
      client:
        provider:
          keycloak:
            issuer-uri: ${SSO_BASE_URL}/realms/${SSO_REALM}
            user-name-attribute: sub
        registration:
          keycloak:
            provider: keycloak
            client-id: ${SSO_ENGINE_CLIENT_ID}
            client-secret: ${SSO_ENGINE_CLIENT_SECRET}
            redirect-uri: "{baseUrl}/app/{action}/oauth2/code/{registrationId}"

application:
  # <client-id>:<role> that a user must hold to access the web apps
  web-app-role: "${SSO_ENGINE_CLIENT_ID}:${WEBAPP_REQUIRED_ROLE}"
  # id of the spring.security.oauth2.client.registration entry used for login
  registration: keycloak
```

### Configuration reference

| Property | Description | Example |
|---|---|---|
| `application.web-app-role` | Client role required to access the web apps, as `<client-id>:<role>`. | `engine:webapp-user` |
| `application.registration` | The OAuth2 client registration id used for the login redirect. | `keycloak` |
| `spring.security.oauth2.client.*` | Standard Spring Security OAuth2 client configuration (login). | — |
| `spring.security.oauth2.resourceserver.jwt.issuer-uri` | Issuer used to validate bearer tokens on the REST API. | — |

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for the
development setup, build/run commands and the pull-request process.

## License

Distributed under the MIT License. See [LICENSE](LICENSE) for details.
