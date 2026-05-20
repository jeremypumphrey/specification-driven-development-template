# Infrastructure instructions

## Target architecture
- Frontend:
  - Vue.js static site
  - S3 bucket for the Vue.js build artifacts
  - CloudFront public distribution in front of the bucket
- API layer:
  - API Gateway public interface routing to api endpoints
  - Lambda functions for API handlers
  - Lambda functions in Python, deployed from zip files in S3
- Data layer:
  - DynamoDB backend table(s)
- Infrastructure:
  - AWS CDK v2 in Python for all infrastructure as code

## Source of truth
- Feature requirements live in `specs/`
- Repo behavior lives in `AGENTS.md`
- Shared API shapes and contracts should live in a shared schema location

## Design principles
- Use AWS managed services where possible to reduce operational overhead.
- Keep the architecture simple and focused on the requirements.
- Keep the API contract explicit and versionable.
- Keep frontend, backend, and infrastructure changes synchronized.
- Prefer small services and clear module boundaries.
- Optimize for testability, predictable deployments, and maintainability.

## Implementation conventions
- UI code should not know infrastructure details.
- Lambda code should not know deployment specifics.
- CDK should not embed business logic.
- DynamoDB schema should be driven by read/write access patterns.

## Tagging
- Use AWS CDK's support for tagging resources to ensure that all deployed resources are tagged with appropriate metadata, such as the environment, application name, and version, to facilitate tracking and management of resources across different environments and deployments, including cost tracking.
- All tag keys should be uppercase. 
- Tag PRODUCT should be "shared"
- Tag ENVIRONMENT should be one of "int" for Integration, "uat" for UAT, or "prod" for Production.

## Security
- Use least-privilege IAM policies.
- Do not allow any public access other than through CloudFront.
- Restrict direct API access where feasible (for example, with auth, WAF rules, and resource policies), and document any intentional public API exposure.
- Encryption at rest and in transit should be enabled for all resources.

## CDK conventions
- Use AWS CDK v2 in Python for infrastructure.
- Avoid CDK Level 3 constructs. Use L1 or L2 constructs for greater control and future customization as needed.
- Define infrastructure as code only; do not rely on console-created resources.
- Keep stacks small, composable, and focused.
- Use a single CDK app with seperate stacks for different layers (Frontend UI, api, data layer).
- Use Python CDK constructs and idiomatic Python naming.
- Separate environment-agnostic constructs from environment-specific stacks where practical.
- AWS resource names should be consistent and include the project name, environment, and resource type for clarity and to avoid naming conflicts. For example, an S3 bucket for the frontend in the Integration environment could be named "bookstore-int-frontend-bucket".
- Keep stack names and resource names deterministic, but append unique identifiers where necessary to avoid conflicts. More than one instance of the application stack should be able to coexist in one account, so names should be clear and consistent, but unique. 
- Grant only the permissions required by the Lambda functions, API Gateway, and deployment pipeline.

## Network
- Use existing VPC and subnets, do not create new ones.
- Per deployment account, ask for vpc and subnet IDs to use for any service that requires them, and use those values in CDK context or environment variables rather than hardcoding them in code.
- Use only us-east-1 region
- Single region deployment is sufficient, but 2 Availability Zones should be used whenever possible. 
- Use the same VPC, AZs, and subnets for the entire stack to avoid cross-AZ data transfer costs and latency.

## Frontend hosting
- Configure the S3 bucket for private access.
- The S3 bucket for UI hosting should be named with the pattern "www.project.environment", for example "www.bookstore.int" for the frontend bucket in the Integration environment.
- Serve the UI through CloudFront.
- Use OAC/OAI-style secure access patterns rather than public bucket access.
- Enable cache-control behavior appropriate for static assets and app shell assets.

## API Gateway and Lambda
- Route API Gateway to Lambda through explicit integrations.
- Define routes clearly and keep authentication/authorization decisions explicit.
- Pass environment variables from CDK rather than hardcoding values in code.
- Keep Lambda functions small and focused on a single responsibility.
- Use separate Lambda functions for different API endpoints or groups of related endpoints.
- Use API Gateway features for request validation, throttling, and error handling where appropriate.

## DynamoDB
- Define table keys to match the application access patterns.
- Add indexes only when required by a concrete query pattern.
- Enable encryption at rest and use least-privilege access.

## Deployment
- The infra code must synthesize cleanly without manual edits.
- Keep deployment steps documented.
- Prefer reproducible builds and pinned dependencies.