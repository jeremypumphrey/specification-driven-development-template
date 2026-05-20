# AGENTS.md

## Mission
Build and maintain a production-grade web application with:
- A Vue.js web UI
- Python AWS Lambda API endpoints
- DynamoDB as the backend datastore
- AWS infrastructure defined in AWS CDK using Python

## Versions
- Begin with the latest stable version of all dependencies.
- Throughout the entire project use the chosen versions consistently.
- Use Python >= 3.14
- Use Node.js >= 24
- Use Vue.js >= 3.5
- Use aws-cdk@latest
- Use Boto3 >= 1.42

## Core rules
- Follow repository guidance files first:
  - `AGENTS.md`
  - `specs/instructions/*.instructions.md`
  - `specs/*.md`
- Do not hallucinate endpoints, components, table schemas, environment variables, or deployment steps.
- Keep diffs minimal, reviewable, and focused on the requested task.
- Use existing patterns in the repository before introducing new ones.
- Do not invent APIs or behavior not stated in the spec.
- When a spec is ambiguous, surface the ambiguity before coding.
- If a requirement conflicts with current code, follow the spec and call out the conflict.
- README is a documentation output, not a source of specifications.
- Follow DRY principles and reusable workflows. 

## Output standards
- Prefer code that is idiomatic for the target stack.
- Keep naming consistent across frontend, backend, and infrastructure.
- Add tests for behavior changes.
- Update docs when public behavior changes.

## Implementation discipline
- If a spec is incomplete, implement the smallest safe interpretation and clearly mark the assumption in code review notes.
- Never bypass the spec to save time.
- Treat infrastructure, backend, and frontend as one system; changes must remain consistent across all three layers.

## Operating principles
- Follow the repository spec exactly.
- When a requirement conflicts with existing code, follow the spec and call out the conflict in the PR summary or changelog note.
- Do not invent product behavior, API fields, UI flows, endpoints, UI behavior, data fields, table attributes, auth flows, or deployment steps.
- Prefer the smallest correct, reviewable change.
- Preserve existing conventions unless the spec explicitly requires a change.
- Implement the requested behavior end-to-end.
- Keep implementation, tests, and infrastructure aligned.
- Treat frontend, backend, and infrastructure as one system. Changes in one layer must remain consistent with the others.
- Do not introduce new frameworks, services, or patterns unless they materially improve the implementation and fit the existing architecture.
- Update related documentation and examples.
- Any new imported library, action, or depency should use the latest stable version available at that time. Never start using an older version that will already need to be udpated. 
- Do not import libraries, actions, or dependencies with know vulnerabilities. 

## Folder structure
- Use unique folders for frontend/, backend/, and infrastructure/ deployable units
- Use a shared/ folder for a single source of truth for api contracts

## Work style
- Ask for clarification only when the requirement is truly ambiguous and cannot be resolved from repo context.
- When requirements conflict, follow the most specific repo guidance file and the feature spec.
- Make assumptions explicit in code review notes or commit messages.
- Do not silently broaden scope.
- Preserve backward compatibility unless the change request explicitly requires a breaking change.

## Quality bar
- Keep functions small and testable.
- Write code that is readable by humans first and agents second.
- Prefer deterministic outputs and explicit error handling.
- Avoid hidden state and side effects.

## Testing
- Add or update unit tests for new business logic.
- Write tests for every user-visible behavior.
- Every change should include tests when behavior changes.
- Add integration or contract tests when request/response behavior changes.
- Do not leave TODOs in place of required tests.
- Generate test coverage (for example "pytest + pytest-cov" or "vitest --coverage") and post results in CI Actions and also update README.md with total test coverage %.

## Implementation rules
- Use explicit contracts between layers.
- Keep API schemas stable and versioned when practical.
- Move shared constants, types, and schemas into shared locations.
- Use explicit typing such that tools like mypy won't find issues.
- Do not hardcode environment-specific values in application code.
- Use dependency injection or clear module boundaries where it improves testability.
- Design for muli-tier (Integration, UAT, Production) deployment of code with expected environmental differences.
- Design with unique stack and resource names so that multiple instances can be run in the same AWS account without conflict.
- Library imports should be at the top of files, not in modules. 

## Delivery checklist
Before marking a task complete:
- Code compiles or synthesizes cleanly.
- Relevant tests pass.
- New behavior is documented.
- README updated with key summary, local deployment, and AWS deployment information
- Infrastructure changes are reflected in deployment code.
- Frontend, backend, and infra stay aligned with the same spec.
