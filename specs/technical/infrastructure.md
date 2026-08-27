# Cloud-agnostic infrastructure specification

## Architecture

- Model the system as independently maintainable frontend, API, and data layers with explicit interfaces.
- Use managed services where possible to reduce operational overhead.
- Use serverless and pay-per-use services wherever possible, avoiding 24/7 billable services when possible.
- Keep the architecture no more complex than the requirements demand.
- Prefer small services, clear module boundaries, testability, maintainability, and predictable deployments.
- Keep UI code unaware of infrastructure details, application functions unaware of deployment mechanics, and infrastructure code free of business logic.
- Keep infrastructure, application code, data models, and shared contracts synchronized.
- Any knowable infrastructure size or naming restrictions should be enforced by the application rather than failing from the infrastructure at runtime. 

## Infrastructure as code

- Define all infrastructure in version-controlled code; do not depend on manually created console resources.
- Use one infrastructure application with small, composable stacks or modules for frontend, API, and data layers.
- Separate reusable, environment-agnostic constructs from provider- and environment-specific configuration.
- Keep names deterministic and include project, deployment instance, environment, and resource type; add provider-required uniqueness without losing traceability.
- Allow multiple application instances to coexist in one provider account or subscription.
- Make infrastructure synthesize or compile without manual edits.

## Environments and configuration

- Support Integration, UAT, and Production with explicit environmental differences.
- Supply environment-specific values through deployment configuration or a secret manager, never application-code constants.
- Use the same release identity across all components in an environment.
- Document intentional public entry points; deny direct public access to internal services and storage.

## Security and operations

- Apply least privilege to identities, workloads, and pipelines.
- Encrypt resources in transit and at rest.
- Define retention, backup, recovery, logging, and observability policies explicitly.
- Tag or label resources with at least application, environment, and version metadata for ownership, inventory, and cost tracking.
- Keep deployment steps documented and builds reproducible with pinned dependencies.

## Network

- Reuse approved existing networks and subnets when a workload requires private networking; do not create a network solely by default.
- Obtain network identifiers from deployment configuration rather than hardcoding them.
- Place related private workloads consistently to reduce unnecessary cross-zone traffic and latency.
- Use multiple availability zones where the selected services and resilience requirements make this applicable.

## Hosting and APIs

- Host static frontend artifacts in private object storage behind a public content-delivery entry point.
- Use secure origin access rather than public object-storage access.
- Cache immutable hashed assets and avoid stale caching of the application shell.
- Define SPA fallback behavior without masking API errors.
- Route the public API entry point through explicit function integrations.
- Keep authentication, authorization, validation, throttling, CORS, and error handling explicit.
- Pass runtime configuration from infrastructure code.

## Deployment units

- Treat frontend, API, data, and infrastructure changes as one versioned release.
- Deploy only stacks affected by a release, but deploy, test, and promote all affected stacks together as one atomic release.
- Never promote frontend and backend portions of the same release independently.
