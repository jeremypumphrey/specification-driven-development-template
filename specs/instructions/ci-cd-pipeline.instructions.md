# CI/CD Pipeline instructions

## Pipeline design
- Use GitHub Actions for CI/CD pipelines.
- Only use github workflow extensions from Verified Creators or GitHub itself.
- Use a single unified deployment workflow that deploys all components (infrastructure, api, data layer, and frontend) together per tier.
- Update and deploy any workflows with changes, keep them in sync with the code and infrastructure changes.
- Use environment-specific deployment workflows (e.g. Integration, UAT, Production).
- All components — infrastructure, api, data layer, and frontend — must be deployed together as a single atomic unit to each tier. Frontend and backend must never be deployed to or promoted through a tier independently. All changes must deploy to Integration together, then UAT together, then Production together, using a single unified deployment workflow with a single promotion gate per tier. All tests must pass before promoting to the next tier.
- Infrastructure uses seperate stacks for different layers (Frontend UI, api, data layer), but only deploy stack updates if changes happened in that stack. For example, if only the frontend and api code changed, only deploy the frontend and api stacks, but not the data layer stack. However, all stacks that have changed should be deployed together to each tier, and promoted together to the next tier, to ensure that all components stay in sync across tiers.
- Initial build and test, then deployment and test of each stage should all pass before promoting to the next tier. 
- Deployment to AWS accounts (Integration, UAT, Production) should be kept in sync with github tags or branches to ensure traceability between code changes and deployed versions.
- Keep pipeline configuration in code and versioned in the repository.
- Use CDK in the pipeline to deploy infrastructure and application code.

## Workflow architecture
- Factor all shared CI logic (formatting, linting, static analysis, unit testing) into reusable workflows (GitHub Actions workflow_call) stored under .github/workflows/.
- The CI workflow (triggered on push to any branch and pull requests) and the deploy workflow (triggered on push to main, version tags, and workflow_dispatch) must both call the same reusable workflows for their shared stages. No step definitions may be duplicated between workflows.
- Reusable workflow files should be named clearly (e.g., _lint.yml, _sast.yml, _test.yml) and prefixed with _ to distinguish them from top-level trigger workflows.
- Each reusable workflow should accept inputs/secrets needed to parameterize behavior (e.g., checkout ref, environment name) so both callers can customize without duplicating steps.
- No duplication across workflow files. If two or more workflows share the same job steps (e.g., setup, lint, test), those steps must be extracted into a reusable workflow that is called by both. Copy-pasting job definitions between workflows is prohibited.
- Maintain exactly two top-level trigger workflows:
  - CI workflow (ci.yml) — triggered on push to any branch and pull requests. Calls reusable workflows for: format → lint → SAST → unit tests. No deployments.
  - Deploy workflow (deploy.yml) — triggered on push to main, version tags (v*), and workflow_dispatch. Calls the same reusable workflows for format → lint → SAST → unit tests, then proceeds to deploy to Integration → UAT → Production with gates.
- Both workflows must share all pre-deployment validation logic through reusable workflow calls. The deploy workflow adds deployment and gate jobs on top of the shared validation.
- Common environment setup steps (Python, uv, Node.js, dependency installation) should be extracted into composite actions (under .github/actions/) to eliminate per-job setup duplication within and across workflows.


## Gates
- Implement a pipeline model that deploys to Integration, then has manual approval gates to deploy to UAT, and Production AWS accounts.
- Manual Gate approvals should interact with Slack clinicalbiomed.slack.com channel #jp-gha-test for notifications and approval requests.
- Manual Gate approvals in Slack should include details about the deployment, such as the version, the branch or tag being deployed, the environment it is being deployed to, and any relevant test results or deployment notes to inform the approvers before they approve the deployment to the next tier.
- Nothing should ever promote to Production without first being deployed and tested in UAT environment, and receive manual approval from Slack.
- Nothing should ever promote to UAT without first being deployed and tested in Integration environment, and receive manual approval from Slack.

## Protect Main Branch
- Pre-checkin branch protection rules on main branch to ensure security, specifically no new vulnerable libraries, and no secrets have been committed. Security checks must pass before allowing merges to main branch.

## CI/CD Flow and Validation
- CI/CD Flow Steps:
  - Run Formatters → Linters → Static Analysis → Tests first, and in that order. This allows the pipeline to "fail fast" on simple issues.
  - Formatters, Linters, SAST, should always run first and all pass before any unit tests are attempted. 
  - Unit tests should always run and pass before any deployments are attempted. 
  - Deploy to Integration if unit tests pass, then run integration tests against the deployed environment. 
  - If Integration integration tests pass, send Slack notifications for manual approval to deploy to UAT.
  - Deploy to UAT if manual approval is granted, then run integration tests against the deployed environment. 
  - If UAT integration tests pass, send Slack notifications for manual approval to deploy to Production.
  - Deploy to Production if manual approval is granted, then run read only integration tests against the deployed environment. 
  - If either manual approval via Slack is not granted within 7 days, the deployment should be automatically cancelled to prevent stale deployments.
- Formatter tools to use:
  - Frontend JavaScript: prettier --write
  - Backend Python: black
  - Infrastructure Python CDK: black
  - Infrastructure CloudFormation (post-synthesis): none (do not auto-format CloudFormation templates to avoid unintended changes)
  - Changes from formatters should be automatically committed back to the branch and included in the same pull request to keep the code clean and consistent without requiring manual intervention.
- Linter tools to use:
  - Frontend JavaScript: eslint --fix and eslint-plugin-vue and eslint-plugin-security and eslint-plugin-import
  - Backend Python: ruff --fix and mypy
  - Infrastructure Python CDK: ruff --fix and mypy with boto3-stubs for aws resources in use.
  - Infrastructure CloudFormation (post-synthesis): cfn-lint 
- SAST tools to use:
  - Frontend JavaScript: semgrep and npm audit 
  - Backend Python: semgrep and pip-audit 
  - Infrastructure Python CDK: cdk-nag and pip-audit 
  - Infrastructure CloudFormation (post-synthesis): checkov --soft-fail
- Do not deploy if there are any known vulnerabilities in the codebase
- Do not deploy if there are any secrets committed in the codebase.
- Run formatters and linters with auto-fix options enabled when possible to automatically fix simple issues.  Changes from formatters and linters should be automatically committed back to the branch and included in the same pull request. 


## Documentation
- Update README.md with instructions for running the pipeline and deploying manually.
- Update README.md with secrets and environment variable requirements for the pipeline build and deployment steps. Keep in sync with any new or removed secrets or environment variables required for the pipeline.
- Create a script for new projects that seed all the secrets and environment variables required for the pipeline to run, and document how to use it in the README.md. Keep in sync with any new or removed secrets or environment variables required for the pipeline.

## Security
- Use GitHub Code Scanning Default setup for CodeQL for static code analysis in the pipeline to identify and remediate security vulnerabilities in the codebase. Do not add an advanced CodeQL workflow unless Default setup is disabled.
- Customize Dependabot for maximum effectiveness specific to this project's codebase and dependencies, and ensure that it is properly configured to scan all relevant code and dependencies in the repository.
- Do not promote code with known vulnerabilities to UAT or Production environments. Address any identified vulnerabilities before allowing promotion to higher environments.
- No library dependencies should be pulled that are newer than 5 days old. Ensure all dependencies are pinned to specific versions with sha256 hashes to prevent supply chain attacks and ensure reproducible builds. For example python should use 'uv lock --upgrade --exclude-newer "5 days"''', and npm should use 'ncu --cooldown 5 -u'. 

## CDK deployment in pipeline
- Use AWS CDK for infrastructure deployment, and ensure the pipeline has permissions to deploy CDK stacks to the respective AWS accounts.
- Use AWS CDK to deploy application code and infrastructure together, ensuring that any changes to the infrastructure are tested in the same pipeline as the application code changes.
- Ensure that the pipeline has appropriate permissions to deploy to the AWS accounts, and that any secrets or credentials are securely stored and accessed in the pipeline configuration.
- Use AWS CDK's built-in support for environment-specific configurations to manage differences between Integration, UAT, and Production environments, such as different resource names, configurations, or scaling settings.

## Testing in pipeline
- Implement automated testing in the pipeline to validate that the deployed application and infrastructure are functioning correctly in each environment before promoting to the next tier. This can include unit tests, integration tests, and end-to-end tests that run after deployment to ensure that the application is working as expected in the target environment.
- Testing reports and results should be clearly communicated in the pipeline.

## Deployment details
- Use OIDC temporary credentials for AWS access such as aws-actions/configure-aws-credentials

## Secret management
- Use GitHub Secrets to securely store and manage any secrets or credentials required for the pipeline, following this template:
- GH_REPO                   This GitHub repository in OWNER/REPO format
- AWS_REGION                AWS region to deploy to "us-east-1"
- SLACK_BOT_TOKEN           Slack Bot OAuth token (xoxb-…)
- SLACK_CHANNEL_ID          Slack channel ID for gate approvals and deployment notifications
- AWS_ROLE_ARN_INTEGRATION  IAM role ARN for the Integration AWS account
- AWS_ROLE_ARN_UAT          IAM role ARN for the UAT AWS account
- AWS_ROLE_ARN_PRODUCTION   IAM role ARN for the Production AWS account
