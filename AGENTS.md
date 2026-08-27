# Reusable project instructions

## Purpose

Build and maintain a production-grade, multi-tier web application. This file contains rules shared by every layer; detailed rules live in the other specifications in this directory.

These specifications are technical only. Each project must supply a separate feature specification for product behavior, API operations, UI flows, entities, fields, validation rules, access patterns, seed data, and acceptance criteria.

# Specification Governance

## Repository Boundaries

* `/specs` contains normative requirements, technical constraints, derived requirements, and explicit assumptions.
* `/docs` contains architecture, design rationale, ADRs, and explanatory documentation.
* `/shared` contains canonical machine-readable contracts shared across system boundaries.
* `/src` contains implementation.
- Ignore /specs/ files that don't end in .md and file contents that are commented out.

## Specification Structure

```text
/specs/
├── functional/     # Human-authoritative functional requirements
├── technical/      # Human-authoritative technical constraints
├── derived/        # Inferred requirements traceable to authoritative specs
└── assumptions/    # Unvalidated assumptions required to proceed
```

## Authority Rules

1. `/specs/functional` and `/specs/technical` are authoritative. Never change these files.
2. AI MUST NOT add inferred requirements to authoritative directories.
3. Inferred requirements MUST go in `/specs/derived` and reference their source requirements.
4. Missing information required to proceed MUST be recorded in `/specs/assumptions`.
5. AI MUST surface conflicts with authoritative requirements rather than silently resolving them.

## Artifact Placement

* Requirements and constraints → `/specs`
* Architecture and design rationale → `/docs`
* API, schema, and event contracts → `/shared`
* Implementation → `/src` → `frontend/`, `backend/`, and `infrastructure/`.

AI MUST NOT duplicate an existing canonical artifact.

If a canonical API or schema exists under `/shared`, reference or update it instead of creating a duplicate specification under `/specs` or `/docs`.

## Core Rule

Maintain one canonical representation per concern, and keep AI-inferred requirements distinguishable from explicitly authoritative requirements.

# Engineering

## Technology and versions
- Deploys both locally and to AWS. Ignore infrastructure-Azure.md. 
<!-- - Deploys both locally and to Azure. infrastructure-AWS.md.  -->
- Use Python 3.13 or later for Python application and infrastructure code.
- Use Node.js 24 or later.
- Use Vue.js 3.5 or later.
- Use Boto3 1.42 or later.
- Start with the latest stable dependency versions allowed by the five-day supply-chain cooldown in `ci-cd.md`, then pin the selected versions and use them consistently in every layer and environment.
- Do not add dependencies with known vulnerabilities.


## Engineering rules

- Follow the specifications exactly and implement requested behavior end to end.
<!-- - Do not invent endpoints, components, fields, schemas, environment variables, authorization flows, deployment steps, or product behavior. -->
- When necessary, derrive and document endpoints, components, fields, schemas, environment variables, authorization flows, deployment steps, or product behavior.
- Use existing repository patterns before introducing new ones.
- Keep changes minimal, focused, reviewable, and backward compatible unless a breaking change is explicitly required.
- Do not introduce a framework, service, or pattern unless it materially improves the implementation and fits the architecture.
- Keep frontend, backend, data, infrastructure, contracts, tests, and documentation synchronized.
- Use idiomatic code, consistent cross-layer naming, explicit typing, deterministic outputs, and explicit error handling.
- Keep functions small, readable, and testable; avoid hidden state and unnecessary side effects.
- Use dependency injection or clear module boundaries where it improves testability.
- Keep imports at the top of source files.
- Do not hardcode environment-specific values in application code.
- Make incomplete-spec assumptions explicit in review notes and implement only the smallest safe interpretation.
- Use DRY implementations and reusable workflows.
- Update documentation and examples when public behavior, setup, deployment, secrets, or environment variables change.

## Security

- Validate untrusted input at system boundaries and fail closed.
- Apply least privilege and encrypt data in transit and at rest.
- Never log, commit, or expose secrets or sensitive data.
- Do not deploy code with known vulnerabilities or committed secrets.

## Delivery requirements

Before completion:

- Code compiles and infrastructure synthesizes without manual edits.
- Relevant formatting, linting, static analysis, unit, contract, integration, infrastructure, and end-to-end checks pass as applicable.
- Behavior changes have tests and documentation.
- Coverage is generated, published in CI, and the total is reflected in the project README.
- The README's project summary and local and provider deployment instructions remain current.
- Frontend, backend, data, infrastructure, and shared contracts remain aligned with the feature specification.


