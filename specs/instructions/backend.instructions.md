# Backend instructions

## Runtime
- Implement Lambda endpoints in Python.
- Use the Python runtime defined by this repository's infrastructure code.
- Keep runtime assumptions explicit in code and deployment config.

## API design
- Keep handlers thin.
- Put business logic in separate service modules.
- Normalize all request and response schemas.
- Return consistent error payloads and HTTP status codes.

## DynamoDB usage
- Design around access patterns first.
- Put DynamoDB access in repository modules.
- Use a partition-key and sort-key model deliberately.
- Avoid scans unless the spec explicitly allows them.
- Prefer single-table or well-justified multi-table design based on the use case.
- Keep items small and predictable.
- Use conditional writes where correctness matters.
- Do not store large blobs in DynamoDB when S3 is more appropriate.

## Reliability
- Make handlers idempotent where practical.
- Validate all input at the boundary.
- Log meaningful context, but never secrets or sensitive data.
- Fail closed on malformed requests.

## Testing
- Unit test core logic without AWS dependencies.
- Mock AWS services at the boundary.
- Add contract tests for API shape and DynamoDB access patterns when needed.