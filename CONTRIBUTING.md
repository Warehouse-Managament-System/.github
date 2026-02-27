# Contributing

Thank you for your interest in contributing to the Warehouse Management System.

## Getting Started

1. Fork the repository you want to contribute to
2. Clone your fork locally
3. Create a feature branch from `main`

```bash
git checkout -b feature/your-feature-name
```

## Branch Naming

| Prefix | Purpose |
|:-------|:--------|
| `feature/` | New feature |
| `fix/` | Bug fix |
| `refactor/` | Code refactoring (no behavior change) |
| `docs/` | Documentation only |
| `chore/` | Build, CI, tooling changes |

## Commit Messages

Use clear, imperative-mood commit messages:

```
Add booking expiry batch job
Fix duplicate invoice generation on Kafka retry
Refactor delivery service Feign client error handling
```

## Pull Requests

1. Keep PRs focused — one feature or fix per PR
2. Update relevant documentation if your change affects APIs, events, or database schemas
3. Ensure all existing tests pass before submitting
4. Add tests for new functionality
5. Reference any related issues in the PR description

## Code Style

- Follow existing patterns in the codebase
- Use the project's Checkstyle/formatter configuration if present
- Keep service boundaries clean — no direct database access across services
- New Kafka events must follow the Transactional Outbox pattern
- New REST endpoints must be documented in the API docs

## Architecture Guidelines

- Each microservice owns its database — never access another service's DB directly
- Cross-service communication: Kafka for async events, OpenFeign for sync queries
- All Kafka events go through the `outbox_events` table (Transactional Outbox + Debezium CDC)
- Consumers must be idempotent
- Use Resilience4j for circuit breaking on sync calls

## Questions?

Open an issue if you have questions about the architecture or contribution process.
