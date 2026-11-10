# Summiva repair status

## Goal

Restore a reproducible build and verify the complete user journey across the Flutter client, FastAPI API, PostgreSQL, and NLP services. A production-ready claim is only made after the acceptance checks below pass.

## Current baseline findings

- The repository default branch is `master`; the previous workflow only listened to `main`.
- The previous workflow invoked deployment commands that are not present in the repository (`src/backend/deploy.py` and `flutter deploy`) and attempted to push Docker images without a verified release configuration.
- The Flutter entry point imports Provider and an AuthProvider, while the checked-in pubspec declares Riverpod and the repository tree does not contain that AuthProvider.
- 26 Flutter source files are zero-content/near-empty stubs (2 bytes in Git's tree metadata).
- Backend configuration and imports are inconsistent: settings require numerous environment variables, middleware imports modules not present in the tree, and auth modules reference database/model paths that do not line up.
- Several service entry points still return demonstration responses or contain TODO placeholders rather than completed NLP workflows.

## Changes in this repair branch

- Replaced the misleading CI/CD workflow with validation-only CI.
- CI now runs on pushes to `master` and `repair/**`, and on pull requests targeting `master`.
- Backend Python syntax is checked with `compileall`.
- Flutter dependency resolution, static analysis, tests, and release web build are explicit gates.
- Removed unverified Docker publishing and automatic staging/EKS deployment from CI. Deployment must not run until build, test, secrets, registry, and target-environment configuration are verified.

## Acceptance gates

1. Backend imports and starts from the documented command.
2. PostgreSQL migrations apply to an empty database and upgrade an existing database safely.
3. Registration, login, token refresh, logout, and protected-user routes pass integration tests.
4. Summarization, tagging, grouping, and search perform real work; no demo/TODO response is reachable in production.
5. Flutter dependency resolution, analysis, widget tests, and release builds pass.
6. Flutter-to-API journeys pass against a disposable test environment.
7. Security checks cover password storage, token lifecycle, authorization, CORS, secrets, upload validation, and rate limits.
8. Docker Compose starts the supported local stack and health/readiness checks reflect actual dependencies.
9. Production deployment is manual/approval-gated until a real target environment and required secrets are configured.

## Verification status

The CI workflow change is committed, but GitHub Actions has not yet executed it. No runtime, database, end-to-end, security, or production deployment tests have been claimed as passing. This branch is an in-progress repair, not a production release.
