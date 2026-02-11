# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Java test library providing pre-configured Keycloak integration test infrastructure using Testcontainers. Consumers add this as a test dependency and extend `KeycloakITBase` to get a running Keycloak container with a pre-imported test realm, TLS enabled, and helper methods for OIDC token operations.

- **Group/Artifact**: `de.cuioss.test:cui-test-keycloak-integration`
- **Java module**: `de.cuioss.test.keycloakit`
- **Parent POM**: `de.cuioss:cui-java-parent` (CUI organization parent)
- **Package**: `de.cuioss.test.keycloakit` (note: `keycloakit`, not `keycloak`)

## Build Commands

```bash
# Full build with tests (requires Docker running for Testcontainers)
./mvnw clean verify

# Build without tests
./mvnw clean install -DskipTests

# Run a single test class
./mvnw test -Dtest=KeycloakITBaseTest

# Run a single test method
./mvnw test -Dtest=KeycloakITBaseTest#shouldStartKeycloak

# Generate site/javadoc (note: legacyMode and generateReports workarounds in pom.xml)
./mvnw site
```

CI builds against Java 21 and 25 (configured in `.github/project.yml`).

## Architecture

This is a small, focused library with only two production classes:

- **`TestRealm`** — `@UtilityClass` holding all test realm constants: realm name, admin credentials, OIDC client ID/secret, test user credentials, and TLS keystore paths. These match the realm JSON at `src/main/resources/oauth_integration_tests-realm.json`.

- **`KeycloakITBase`** — Abstract base class annotated with `@Testcontainers`. Declares a `KeycloakContainer` with the test realm imported and TLS enabled. Provides helper methods: `getRealmUrl()`, `getWellKnownUrl()`, `getOIDCUrl()`, `getTokenUrl()`, `getJWKSUrl()`, `getIssuer()`, and `parameterForScopedToken()`.

Consumers extend `KeycloakITBase` in their own test classes. See `KeycloakITBaseTest` for usage patterns including RestAssured TLS setup with the provided keystore.

## Key Dependencies

- **testcontainers-keycloak** (`com.github.dasniko`): Keycloak-specific Testcontainer — scoped as `compile` (not `test`) since this library provides a base class
- **testcontainers** + **junit-jupiter** (Testcontainers): Also `compile` scope for the same reason
- **rest-assured**: Used in tests for HTTP/OIDC endpoint verification
- **Lombok**: Used with `@UtilityClass`; `lombok.config` enables `addLombokGeneratedAnnotation` for coverage exclusion
- **cui-java-tools**: CUI core utilities

## Test Realm Configuration

The realm JSON (`oauth_integration_tests-realm.json`) is in `src/main/resources/` (not test resources) because it ships with the library for consumers to use.

- Realm: `oauth_integration_tests`
- Test user: `testUser` / `drowssap`
- OIDC client: `test_client` / `yTKslWLtf4giJcWCaoVJ20H8sy6STexM`
- Admin: `admin` / `adminPass`
- TLS keystore: provided by testcontainers-keycloak at `/tls.jks` (password: `changeit`)

## Conventions

- Tests require a running Docker daemon (Testcontainers starts Keycloak in Docker)
- SonarCloud integration: project key `cuioss_cui-test-keycloak-integration`
- `TestRealm.java` is excluded from Sonar coverage via `sonar.coverage.exclusions` (constants-only class)
- EditorConfig: 4-space indentation, LF line endings, UTF-8
- Apache License 2.0 headers on all source files

## Git Workflow

All cuioss repositories have branch protection on `main`. Direct pushes to `main` are never allowed. Always use this workflow:

1. Create a feature branch: `git checkout -b <branch-name>`
2. Commit changes: `git add <files> && git commit -m "<message>"`
3. Push the branch: `git push -u origin <branch-name>`
4. Create a PR: `gh pr create --repo cuioss/cui-test-keycloak-integration --head <branch-name> --base main --title "<title>" --body "<body>"`
5. Wait for CI + Gemini review (waits until checks complete): `gh pr checks --watch`
6. **Handle Gemini review comments** — fetch with `gh api repos/cuioss/cui-test-keycloak-integration/pulls/<pr-number>/comments` and for each:
   - If clearly valid and fixable: fix it, commit, push, then reply explaining the fix and resolve the comment
   - If disagree or out of scope: reply explaining why, then resolve the comment
   - If uncertain (not 100% confident): **ask the user** before acting
   - Every comment MUST get a reply (reason for fix or reason for not fixing) and MUST be resolved
7. Do **NOT** enable auto-merge unless explicitly instructed. Wait for user approval.
8. Return to main: `git checkout main && git pull`
