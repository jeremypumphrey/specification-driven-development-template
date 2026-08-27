# Testing standards

## Required coverage

- Unit test coverage should be 95% or higher
- Add or update unit tests for new and changed business logic.
- Test every user-visible behavior and applicable loading, empty, success, error, retry, and submission state.
- Add contract tests whenever request or response behavior changes.
- Add integration tests for datastore access patterns and deployed API behavior.
- Add infrastructure assertions for resources, permissions, configuration, security controls, names, and tags.
- Add end-to-end or smoke tests for the deployed application when a behavior crosses layers.
- Do not leave TODOs in place of required tests.

## Isolation

- Unit-test core backend logic without cloud dependencies.
- Mock provider services only at provider boundaries.
- Mock frontend network access at the API-client boundary.
- Keep tests deterministic and independent of execution order.

## Contract testing

- Validate frontend models, backend responses, and deployed API responses against the version-controlled contract in `shared/`.
- The feature specification must identify the contract format, ownership, and required operation-specific cases.
- Verify success responses, validation failures, malformed requests, missing resources, and authorization behavior where applicable.

## Deployment testing

- Run unit and contract tests before deployment.
- After each environment deployment, run integration and reachability tests before promotion.
- Run only non-mutating production smoke tests unless explicitly approved otherwise.
- Clearly publish test reports and results in CI.

## Coverage reporting

- Generate coverage with the repository's established tools, such as pytest/pytest-cov and Vitest coverage.
- Publish coverage artifacts and totals in GitHub Actions.
- Keep the total coverage percentage in README current.
- A project-specific minimum threshold may be defined by the feature or repository specification; absence of a threshold does not waive coverage reporting.
