# CI/CD technical specification

## Platform and supply chain

- Use GitHub Actions with pipeline configuration versioned in the repository.
- Use Actions only from GitHub or verified creators, use the latest version but no newer than 5 days, and pin them to immutable revisions.
- Use the latest versions of libraries, tools, and dependencies, but Do not select releases newer than 5 days.
- Pin dependency versions and integrity hashes where the ecosystem supports them.
- For Python, use the repository's `uv` lock workflow with an equivalent of `--exclude-newer "5 days"`.
- For npm, use a five-day update cooldown, such as `ncu --cooldown 5 -u`, and commit the lockfile.
- Configure Dependabot for every repository ecosystem.

## CI/CD specifications
- Use GitHub Flow workflow model.
- Protect Main from direct commits, all changes shall merge through reviewed pull requests with required CI checks.
- Every successful merge to `main` shall automatically build immutable application artifacts and deploy to INT.
- CI Tests, Run fail-fast stages in this order: format, lint, SAST, unit tests. These must pass for merge to main.
- CD, The same immutable application artifacts shall be used to deploy to INT, UAT, & Production. Build once, deploy copies.
- A release shall be eligible for promotion only after the preceding environment deploys successfully and passes required tests.
- UAT deployment shall be manually triggered by tagging an INT-tested release with a version number, and shall require GitHub Environment approval.
- Production shall deploy only a UAT-approved release, triggered from an immutable semantic-version tag or GitHub Release, with required approval and no self-approval.

## Secrets and environments

- Use GitHub Actions Environment Secrets under Environments called "INT", "UAT", and "Production".
- Pass only required secrets to reusable workflows.
- Provide a project setup script that seeds required GitHub secrets and variables, and document its use. In github Actions Secrets and Environment Secretes it should create any Secrets or Variables names needed for all activites, and populate with blank values as placeholders. Specify in the readme file any secrets and variable that need to have values populated before deployments will work as planned.
- Keep README pipeline, manual deployment, secret, and environment-variable documentation synchronized with the workflows.


## Test reporting

- Follow `testing.md`.
- Publish validation, coverage, integration, end-to-end, and deployment results in GitHub Actions and in the README.
- Treat successful environment tests as a prerequisite for promotion.

<!-- ## Archived for future use

## Azure CI/CD specifications

1. GitHub shall host source code, specifications, Terraform, and GitHub Actions workflows.

2. `main` shall be protected; all changes shall merge through reviewed pull requests with required CI checks.

3. Pull requests shall run frontend lint/build validation, backend tests, Terraform format/validate/plan, and security scans; PR workflows shall not deploy.

4. Every successful merge to `main` shall automatically build immutable application artifacts and deploy to INT.

5. Application artifacts shall be built once, identified by Git SHA/version/digest, and promoted unchanged through INT, UAT, and Production.

6. Each environment deployment shall validate the immutable artifacts, run a
   Terraform plan, publish deployment evidence, and run smoke tests. Mutating
   steps shall execute only when their deployed content or dependencies differ
   from the latest successful deployment in that environment:

   1. apply the saved Terraform plan only when it contains changes;
   2. deploy the Function App only when the backend artifact changes or changed
      infrastructure requires the package to be restored;
   3. upload the SPA to `$web` only when the frontend artifact changes or its
      hosting infrastructure changes;
   4. purge Front Door only after an SPA upload or a relevant Front Door change;
   5. seed deployment data only on first deployment or when the seed input or
      its storage changes.

   A first deployment shall treat every component as changed. A release with no
   changes shall still produce a successful, traceable no-op deployment record
   after planning and read-only smoke tests; no apply, package upload, SPA
   upload, seed, secret synchronization, or cache purge shall run.

7. A release shall be eligible for promotion only after the preceding environment deploys successfully and passes required tests.

8. UAT shall be manually triggered by selecting an INT-tested release and shall require GitHub Environment approval.

9. Production shall deploy only a UAT-approved release, triggered from an immutable semantic-version tag or GitHub Release, with required approval and no self-approval.

10. INT, UAT, and Production shall use separate GitHub environments, Terraform state, configuration, Azure identities, and RBAC scopes.

11. Terraform shall generate a separate environment-specific plan, but all environments shall use the same reviewed Terraform source and reusable deployment workflow.

12. Azure authentication shall use GitHub OIDC/workload identity federation with no stored Azure credentials; runtime secrets shall use Azure Key Vault or managed identity where possible.

13. Deployments shall be serialized per environment, and Production deployments shall never cancel an active deployment.

14. Each deployment shall record the Git SHA, artifact digests, Terraform plan summary, environment, approvals, test results, and outcome.

    Deployment evidence shall also record the baseline deployment, carried or
    updated component revisions, component change decisions, and whether the
    deployment performed mutations or was a no-op. Unchanged components retain
    their previously deployed revision; release identity does not force
    infrastructure or application changes.

15. Cleanup shall use a separate manually triggered workflow requiring environment selection, typed confirmation, approval, and a reviewed Terraform destroy plan; Production destruction shall require a separate break-glass process.

16. In github Actions Secrets, create any Secrets or Variables names needed for all activites, and populate with blank values as placeholders. Specify in the readme file any secrets and variable that need to have values populated before deployments will work as planned.  -->


<!-- ## Workflow architecture

- Maintain exactly two top-level workflows:
  - `ci.yml`: runs on pull requests and pushes to any branch; performs validation only.
  - `deploy.yml`: runs on pushes to the main branch, `v*` tags, and manual dispatch; performs the same validation and then deployment.
- Put shared validation in clearly named reusable `workflow_call` workflows under `.github/workflows/`, prefixed with `_`.
- Both top-level workflows must call the same reusable formatting, linting, SAST, and unit-test workflows.
- Do not duplicate shared jobs or steps across workflows.
- Put repeated Python, uv, Node.js, and dependency setup in composite actions under `.github/actions/`.
- Parameterize reusable workflows with the required ref, environment, configuration, and explicitly passed secrets.
- Update workflows with application and infrastructure changes so pipeline behavior remains synchronized with the release.

## Validation order and tools

Run fail-fast stages in this order: format, lint, SAST, unit tests.

- Frontend formatting: Prettier with write mode.
- Python backend and infrastructure formatting: Black.
- Do not auto-format synthesized provider templates.
- Frontend linting: ESLint with Vue, security, and import plugins, using auto-fix where possible.
- Python backend linting: Ruff with fixes, then mypy.
- Python infrastructure linting: Ruff with fixes, then mypy with service-specific Boto3 stubs.
- Synthesized AWS template linting: `cfn-lint`.
- Frontend SAST: Semgrep and `npm audit`.
- Backend SAST: Semgrep and `pip-audit`.
- AWS infrastructure SAST: `cdk-nag`, `pip-audit`, and Checkov on synthesized templates. Checkov may report with `--soft-fail`, but known vulnerabilities still block deployment.
- Use GitHub Code Scanning default setup for CodeQL; add an advanced workflow only when default setup is disabled.
- Scan for committed secrets.
- Block deployment when validation fails, a known vulnerability remains, or a secret is committed.
- Where repository permissions safely allow it, commit formatter and linter auto-fixes back to the same branch and pull request.
- Protect the main branch and require vulnerability and secret checks to pass before merge.

## Release and promotion

- Build and validate one release containing all affected frontend, API, data, and infrastructure components.
- Deploy only stacks changed by the release, but deploy and promote all affected stacks together.
- Never deploy or promote frontend and backend changes from the same release independently.
- Deploy to Integration, test it, request approval, deploy the same release to UAT, test it, request approval, then deploy it to Production.
- Require all pre-deployment checks and environment tests to pass before the next stage.
- Run read-only integration or smoke tests in Production.
- Use one promotion gate per tier transition.
- Cancel an approval request and deployment if approval is not granted within seven days.
- Keep deployed versions traceable to immutable Git tags or commits.
- Define and document rollback behavior for failed environment deployments.

## Approval notifications

- Send UAT and Production approval requests and deployment notifications through Slack.
- Configure the Slack workspace and channel rather than hardcoding organization-specific values.
- Include release version, branch or tag, destination environment, test results, and relevant deployment notes.
- Store `SLACK_BOT_TOKEN` as a GitHub secret and `SLACK_CHANNEL_ID` as a secret or protected environment variable.
 -->