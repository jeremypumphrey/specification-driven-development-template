# Backend and API technical specification

## Runtime and boundaries

- Implement serverless API handlers in Python at the version declared in `AGENTS.md`.
- Use the runtime selected by infrastructure code and keep runtime assumptions explicit in code and deployment configuration.
- Keep handlers thin and focused on transport concerns.
- Put business logic in service modules and datastore access in repository modules.
- Keep application code independent of deployment implementation details.

## API design

- Define routes, methods, status codes, authorization decisions, and request and response shapes in the feature specification and version-controlled contract under `shared/`.
- Treat the shared contract as the single source of truth across backend, frontend, tests, and infrastructure.
- Normalize requests and responses.
- Return consistent error payloads and HTTP status codes.
- Keep schemas stable and version them when practical.
- Do not add endpoints, fields, or behavior that the feature specification does not define.

## Validation and reliability

- Validate and normalize all input at the API boundary, including rejecting unknown fields when the contract is closed.
- Fail closed on malformed requests.
- Make mutation handlers idempotent where practical.
- Use explicit error handling and deterministic response formatting.
- Log meaningful operational context without secrets or sensitive data.
- Pass configuration through environment-specific deployment settings rather than hardcoding it.

## Design quality

- Keep functions small and responsibilities clear.
- Use explicit Python typing compatible with static type checking.
- Isolate provider SDK calls at module boundaries so core logic can be tested without cloud dependencies.
- Apply database correctness controls, such as conditional writes, when required by the access pattern.
