# AWS provider specification

This file supplements `infrastructure.md` for AWS. 

- Use aws-cdk@latest
- Use aws-cdk-lib@latest
- Use Boto3 1.42 or later

## Target services

- Host Vue build artifacts in a private Amazon S3 bucket behind Amazon CloudFront.
- Expose API routes through Amazon API Gateway with explicit AWS Lambda integrations.
- Implement Lambda functions in Python and package code from `backend/` as versioned zip assets in S3.
- Use Amazon DynamoDB for the application datastore.
- Define and deploy all AWS resources with AWS CDK v2 in Python. 
- Use the latest stable AWS CDK v2 release selected at project initialization.
- UI & API authentication and authorization shold use Amazon Cognito User Pool → Cognito Essentials → Managed Login → this web app/API. 

## CDK conventions

- Use one CDK app with separate, focused frontend, API, and data stacks.
- Use L1 or L2 constructs; avoid L3 constructs so security and configuration remain explicit.
- Use idiomatic Python and the versions declared in `AGENTS.md`.
- Separate reusable constructs from environment-specific stacks.
- Name stacks and resources with the project, deployment instance, environment, and resource type. Include account-derived or generated uniqueness where global names require it.
- Deploy changed stacks together as the affected portion of one release.
- Use CDK context or environment configuration for account, region, network IDs, and environment differences.
- Use CDK-managed tags on all taggable resources. Tag keys are uppercase and include `PRODUCT`, `ENVIRONMENT`, and `VERSION`.
- Set `ENVIRONMENT` to `int`, `uat`, or `prod`; set `PRODUCT` to `shared` unless a project specification overrides it; set `VERSION` to the deployed git ref or release identifier.
- Apply `cdk-nag` and synthesize cleanly without manual template edits.

## Region and networking

- Deploy in `us-east-1`.
- Use an approved existing VPC and subnets only for services that require VPC connectivity; do not create a VPC by default.
- Request required VPC and subnet IDs per deployment account and pass them through CDK context or environment configuration.
- Use two Availability Zones where applicable and keep related VPC workloads on a consistent network placement.

## Frontend hosting

- Block all public access to the S3 origin, enable S3-managed encryption, enforce TLS, and define a retention policy.
- Use CloudFront Origin Access Control, or Origin Access Identity only for an established legacy implementation.
- Redirect viewers to HTTPS.
- Use a non-caching or zero-TTL policy for the SPA shell and an optimized long-lived policy for hashed assets.
- Map frontend 403/404 responses to the SPA shell where client-side routing requires it, without applying that behavior to API paths.
- Select and document an appropriate CloudFront price class.

## API Gateway and Lambda

- Use explicit API Gateway routes and Lambda proxy integrations.
- Define API stage names by environment.
- Configure contract-appropriate request validation, throttling, CORS, authorization, and gateway error behavior.
- Document any intentional public API exposure and restrict direct access with authorization, AWS WAF, or resource policies where the feature permits.
- Keep functions small and group endpoints only when they share a cohesive responsibility.
- Use the supported Python runtime selected in `AGENTS.md`.
- Begin with 256 MB memory and a 30-second timeout when the feature has no measured sizing requirement, then tune from evidence.
- Pass values such as table names, allowed origins, and log levels through Lambda environment variables defined by CDK.
- Set an explicit CloudWatch Logs retention policy.

## DynamoDB

- Define keys and indexes from the access patterns in the feature specification.
- Use on-demand billing when workload capacity is unknown or variable unless evidence supports provisioned capacity.
- Enable encryption and point-in-time recovery.
- Use a `RETAIN` removal policy for production data unless the project explicitly defines another recovery strategy.
- Grant each Lambda only the required table and index actions.

## Security

- Use least-privilege IAM policies for Lambda, API Gateway, and deployment identities.
- Permit public access only through CloudFront and explicitly documented API Gateway entry points.
- Enable encryption at rest and in transit for every supported resource.
- Do not embed credentials in CDK, application code, assets, or synthesized templates.

## AWS deployment from CI/CD

- Deploy application code and infrastructure together through CDK.
- Authenticate GitHub Actions with OIDC temporary credentials using a verified release of `aws-actions/configure-aws-credentials`.
- Store environment-specific role ARNs as GitHub Actions environment secrets
- Configure `AWS_REGION` as `us-east-1`.
- Give deployment roles only the permissions required to synthesize, publish assets, and deploy the target stacks.
- Use CDK environment-specific configuration for resource names, scaling, and other tier differences.
