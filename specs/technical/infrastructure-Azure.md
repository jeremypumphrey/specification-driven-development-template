# Azure provider specification

This file supplements `infrastructure.md` for Azure. 

## Target services

- For the POC, Azure Front Door Standard with WAF will be the only public entry point. API requests will be routed through API Management, which will validate the X-Azure-FDID header to ensure calls came through Front Door. Azure Functions will not be exposed directly; they will be callable only through API Management using function-level authorization and, where practical, Function App access restrictions. Direct authentication with Entra ID/JWT can be added later if required.


<!-- - Implement Lambda functions in Python and package code from `backend/` as versioned zip assets in S3.
 Use idiomatic Python and the versions declared in `AGENTS.md`.
- Use CDK-managed tags on all taggable resources. Tag keys are uppercase and include `PRODUCT`, `ENVIRONMENT`, and `VERSION`.
- Set `ENVIRONMENT` to `int`, `uat`, or `prod`; set `PRODUCT` to `shared` unless a project specification overrides it; set `VERSION` to the deployed git ref or release identifier. -->

## Region and networking

- Deploy in `eastus`.
<!-- - Use an approved existing VPC and subnets only for services that require VPC connectivity; do not create a VPC by default.
- Request required VPC and subnet IDs per deployment account and pass them through CDK context or environment configuration.
- Use two Availability Zones where applicable and keep related VPC workloads on a consistent network placement. -->

- Authenticate GitHub Actions with OIDC temporary credentials to Azure and deploy using Terraform.
- Look for environment-specific GitHub environment secrets:
  - `AZURE_CLIENT_ID`
  - `AZURE_TENANT_ID`
  - `AZURE_SUBSCRIPTION_ID`
  - `TF_STATE_RESOURCE_GROUP`
  - `TF_STATE_STORAGE_ACCOUNT`
  - `TF_STATE_CONTAINER`
  - `TF_STATE_KEY`
- Configure `AZURE_REGION` as `eastus`.
- Give deployment roles only the permissions required to synthesize, publish assets, and deploy the target stacks.

## Deployment units
- Compare each release with the latest successful deployment in the target
  environment. Do not run a mutating deployment step when its component has no
  content or infrastructure dependency change.
- Always plan infrastructure and verify the environment even for a no-op
  release. Apply Terraform only when the saved plan contains changes, and
  preserve carried-forward component revisions in deployment evidence.

## Target architecture

* Build an Azure serverless web stack with:

  * **Azure Front Door Standard + WAF** as the only public entry point.
  * **Azure Storage static website** for the Vue.js SPA.
  * **Azure API Management** for all `/api/*` traffic.
  * **Azure Functions** for backend APIs.
  * **Cosmos DB or Azure SQL Database serverless** for persistence.
  <!-- * **Power BI Embedded** exposed from a page inside the SPA. -->
  * **Key Vault** for secrets.
  * **Application Insights / Log Analytics** for observability.

## Public routing requirements

* Configure one public hostname, for example:
  * `https://app.example.gov`
* Configure Azure Front Door routes:
  * `/*` routes to the Azure Storage static website origin.
  * `/api/*` routes to Azure API Management.
* Do not require the SPA to call a separate API hostname.
* All browser API calls must use:
  * `https://app.example.gov/api/...`

## Front Door security requirements
* Use **Azure Front Door Standard**.
* Enable **WAF** on Front Door.
* Configure WAF in prevention mode for deployed environments unless POC testing requires detection mode first.
* Configure Front Door to forward the built-in `X-Azure-FDID` header.
* Treat Front Door as the only supported public ingress point.

## API Management requirements
* Deploy Azure API Management behind Front Door.
* Expose backend APIs only under:
  * `/api/*`
* Add an inbound APIM policy that rejects any request where `X-Azure-FDID` does not match the expected Front Door ID.
* Return HTTP `403 Forbidden` for requests that fail the Front Door ID check.
* Add basic APIM controls:
  * request size limits
  * rate limiting
  * response header hardening
  * correlation ID propagation
  * structured logging to Application Insights or Log Analytics
* Do not require Entra ID or JWT validation for the initial POC.
* Design APIM policies so JWT validation can be added later without changing the public API shape.

## Azure Functions exposure requirements
* Do **not** expose Azure Functions as a public backend for browser clients.
* Browser clients must never call `*.azurewebsites.net` directly.
* APIM must be the only intended caller of Azure Functions.
* Configure HTTP-triggered Functions with function-level authorization.
* Store Function keys in APIM named values backed by Key Vault where practical.
* Where feasible for the POC, configure Function App access restrictions to allow APIM and deny other public callers.
* Do not place Function keys, backend URLs, or secrets in the SPA.

## Static website requirements
* Host the Vue.js build artifacts in Azure Storage static website hosting.
* Upload built files to the `$web` container.
* Configure:
  * `index.html` as the index document.
  * SPA fallback behavior so client-side routes resolve to `index.html`.
* Do not embed secrets in frontend configuration.
* Frontend configuration may include only public values such as:
  * API base path: `/api`
  <!-- * Power BI report keys or friendly identifiers, not secrets. -->

<!-- ## Power BI Embedded requirements
* Add a SPA route such as:
  * `/reports`
  * `/analytics`
* The SPA must request report embed configuration from:
  * `GET /api/powerbi/embed-config`
* The Azure Function behind this endpoint must:
  * validate the requested report key
  * map the request to an approved workspace/report
  * call the Power BI REST API
  * return only the embed URL, report ID, embed token, expiration, and safe client settings
* The SPA must render the report using the Power BI JavaScript client.
* Power BI service principal credentials must be stored in Key Vault or equivalent secure configuration.
* Do not store Power BI secrets, workspace admin credentials, or embed token generation logic in the SPA. -->

## Data layer requirements
* Use **Cosmos DB** if the application needs document/key-value style access.
* Use **Azure SQL Database serverless** if the application needs relational tables, joins, constraints, or SQL reporting.
* Backend Functions must access data services using managed identity where supported.
* Avoid storage account keys, database account keys, and connection strings unless no managed identity option is available.
* Store unavoidable secrets in Key Vault.

## Infrastructure as code requirements
* Define all Azure resources using Terraform.
* Organize infrastructure into modules:
  * Front Door
  * WAF policy
  * Storage static website
  * API Management
  * Azure Functions
  * Key Vault
  * Cosmos DB or Azure SQL
  * Application Insights / Log Analytics
  * Power BI configuration placeholders
* Parameterize:
  * environment name
  * region
  * domain name
  * Front Door ID expected by APIM
  * SKU selections
  * database choice
  * tags
* Include standard tags:
  * `Application`
  * `Environment`
  * `Owner`
  * `CostCenter`
  * `DataClassification`
  * `ManagedBy`

## CI/CD requirements
- Create a cleanup workflow that runs terraform destroy to clean the Azure footprint

## Runtime security requirements
* Public users can access only:
  * `https://app.example.gov/*`
  * `https://app.example.gov/api/*`
* Public users must not directly access:
  * Azure Functions default hostname
  * APIM default hostname
  * storage account management endpoints
  * database endpoints
  * Key Vault
* APIM must reject direct calls that do not contain the valid `X-Azure-FDID`.
* Functions must reject calls that do not include the expected function authorization.
* All secrets must live in Key Vault or secure platform configuration.
* All traffic must use HTTPS.

## Observability requirements
* Enable Application Insights for Azure Functions.
* Enable APIM diagnostic logging.
* Enable Front Door access and WAF logs.
* Send logs to Log Analytics.
* Track:
  * API request count
  * API latency
  * APIM 4xx/5xx responses
  * Function failures
  * Power BI embed-token failures
  * WAF blocks
  * Front Door origin errors
* Create basic alerts for:
  * high 5xx rate
  * Function failures
  * APIM backend errors
  * Power BI token generation failures
  * abnormal WAF blocks

## Cost-control requirements
* Pull all resources in a common Resource Group based on the application prefix
* Use Front Door Standard for the POC.
* Use APIM Consumption if feature requirements allow.
* Use Functions Consumption or Flex Consumption for low traffic.
* Schedule Power BI Embedded capacity for business-hours usage only, such as 8 AM–5 PM.
* Schedule any Linux VM to stop/deallocate outside business hours.
* Add Azure budgets and alerts for the subscription.
* Add lifecycle rules for storage if stored data grows beyond POC assumptions.

## Acceptance criteria
* `https://app.example.gov/` loads the SPA through Front Door.
* SPA client-side routes work after browser refresh.
* `https://app.example.gov/api/health` succeeds through Front Door and APIM.
* Direct call to APIM default hostname returns `403 Forbidden` because `X-Azure-FDID` is missing or invalid.
* Direct browser call to Function App default hostname fails or is unauthorized.
* SPA contains no backend secrets.
* Power BI report page loads by calling `/api/powerbi/embed-config`.
* Function App logs appear in Application Insights.
* APIM and Front Door logs appear in Log Analytics.
* CI/CD can deploy infrastructure, backend, and frontend from source control.
* Monthly POC cost controls are documented and scheduled shutdown is enabled for expensive resources.


## Azure specifications

### AZ-01 — Explicit deployment scope

**Requirement:** Every environment MUST explicitly declare the Azure tenant ID, subscription ID, environment name, primary region, resource-group names, and Azure cloud environment. Resources MUST NOT inherit an implicit Azure CLI subscription or location.

**Acceptance:** The workflow runs `az account show` before Terraform and fails unless the authenticated tenant and subscription match the environment configuration. The AzureRM provider receives an explicit subscription ID. ([Terraform Registry][1])

---

### AZ-02 — Deterministic, Azure-compliant naming

**Requirement:** All resource names MUST be generated from a centralized naming specification that accounts for each resource type’s length, character, casing, and uniqueness rules. Globally unique services MUST use a stable suffix derived from immutable inputs such as organization, application, environment, and subscription ID.

**Acceptance:** Names are validated before `terraform plan`. Re-running the workflow with identical inputs produces identical names; random values generated during deployment are prohibited. Azure applies different naming and uniqueness rules to different resource types, including globally unique App Service names. ([Microsoft Learn][2])

---

### AZ-03 — Region, SKU, feature, and quota preflight

**Requirement:** Before Terraform planning, the pipeline MUST verify that every requested Azure service, SKU, operating system, zone configuration, and feature is available for the target subscription and region, and that sufficient quota exists.

**Acceptance:** The preflight stage fails with the unavailable SKU, region, feature, or quota and lists approved alternatives. For App Service, it checks the requested SKU with `az appservice list-locations --sku`; compute workloads check regional and SKU quotas. Terraform MUST NOT silently substitute a different SKU or region. ([Microsoft Learn][3])

---

### AZ-04 — Resource-provider registration contract

**Requirement:** The specification MUST list every required Azure resource-provider namespace, such as `Microsoft.Web`, `Microsoft.Storage`, `Microsoft.KeyVault`, and `Microsoft.Network`, and identify whether registration is performed by a bootstrap process or Terraform.

**Acceptance:** A preflight step verifies every namespace is in the `Registered` state before application infrastructure is planned. The AzureRM provider MUST explicitly set its resource-provider registration behavior rather than relying on provider-version defaults. ([Terraform Registry][1])

---

### AZ-05 — Azure Policy and governance compatibility

**Requirement:** Each environment specification MUST document inherited Azure Policy constraints, including allowed regions, allowed SKUs, required tags, encryption settings, private-network requirements, diagnostic settings, and any approved exemptions.

**Acceptance:** CI queries applicable policy assignments before Terraform and fails with the policy assignment and definition identifiers when the proposed configuration would be denied. Azure Policy with a `deny` effect prevents noncompliant create or update requests. ([Microsoft Learn][4])

---

### AZ-06 — GitHub OIDC and permission matrix

**Requirement:** GitHub Actions MUST authenticate to Azure through OpenID Connect federation. Long-lived Azure client secrets MUST NOT be used. The specification MUST define the exact control-plane and data-plane roles required at each scope.

**Acceptance:** The workflow has `id-token: write`, uses a federated Azure identity, and verifies access before Terraform. Role-assignment creation is placed in a separately authorized bootstrap stack unless the deployment identity has `Microsoft.Authorization/roleAssignments/write`. ([GitHub Docs][5])

Example permission categories that must be considered separately:

* Azure resource creation and modification.
* Terraform state-container data access.
* Key Vault secret or certificate data access.
* Container registry push or pull access.
* Network subnet join and delegation.
* Creation of managed-identity role assignments.

---

### AZ-07 — Complete network and DNS contract

**Requirement:** Every Azure service MUST declare its inbound-access model, outbound-access model, public-network setting, firewall rules, subnet, subnet delegation, route requirements, private endpoints, private DNS zones, and virtual-network links.

**Acceptance:** For App Service, the specification distinguishes outbound VNet integration from inbound private-endpoint access. Private endpoints are not complete until their recommended private DNS zones, DNS records, and VNet links exist and name resolution has been tested from the application network. ([Microsoft Learn][6])

---

### AZ-08 — Identity and RBAC propagation handling

**Requirement:** Managed identities and their role assignments MUST be provisioned before resources or application startup operations that depend on those permissions. The deployment MUST account for Azure RBAC’s eventual consistency.

**Acceptance:** Dependent deployment stages execute a bounded access verification with retry and backoff. They proceed only after the identity can perform the required operation, rather than assuming that successful creation of a role assignment means immediate authorization. Azure documents that role assignments can take several minutes—and for some services longer—to propagate. ([Microsoft Learn][7])

---

### AZ-09 — Deletion, replacement, and recovery behavior

**Requirement:** Every stateful or security-sensitive resource MUST define its deletion and recovery semantics: retain, destroy, recreate, soft-delete, purge protection, resource lock, backup, and acceptable replacement downtime.

**Acceptance:** Production destruction or replacement of databases, storage, DNS zones, Key Vaults, or shared networking requires an approved deployment environment. Production Key Vaults enable soft delete and purge protection unless a documented exception exists. ([Microsoft Learn][8])

---

### AZ-10 — Azure deployment acceptance tests

**Requirement:** A successful `terraform apply` MUST NOT by itself constitute a successful deployment. The deployment is complete only after Azure and application-level acceptance checks pass.

**Acceptance:** The workflow verifies, as applicable:

1. Azure resources report successful provisioning states.
2. The website health endpoint returns the expected HTTP response.
3. Public or private DNS resolves to the expected address.
4. TLS and custom-domain bindings are valid.
5. The application starts without configuration errors.
6. Managed identity can access Key Vault, storage, databases, or other dependencies.
7. Application logs and metrics are available.
8. The deployed application version matches the Git commit.

Azure distinguishes syntax validation, preflight validation, deployment failures, and runtime troubleshooting; your pipeline should do the same. ([Microsoft Learn][9])

---

## Terraform specifications

### TF-01 — Version and dependency reproducibility

**Requirement:** The root module MUST constrain the Terraform CLI version and every provider version. Reusable modules MUST also have version constraints. `.terraform.lock.hcl` MUST be committed to source control.

**Acceptance:** CI fails when the runner uses an unsupported Terraform version or when dependency initialization modifies the lock file unexpectedly. Provider upgrades occur only in dedicated pull requests with a reviewed plan. HashiCorp recommends provider constraints and committing the dependency lock file. ([HashiCorp Developer][10])

---

### TF-02 — Bootstrapped remote state

**Requirement:** Terraform state MUST use an Azure Blob Storage backend created by a separate bootstrap stack. Each application and environment MUST have a unique state key. The state backend MUST NOT be created by the same state it stores.

**Acceptance:** The backend uses Microsoft Entra authentication through the GitHub federated identity, supports state locking, and denies broad public access. Backend credentials MUST be supplied through environment-based authentication rather than committed backend configuration. ([HashiCorp Developer][11])

---

### TF-03 — Immutable plan-to-apply workflow

**Requirement:** GitHub Actions MUST create a saved Terraform plan from an exact Git commit, lock file, variable set, backend, and workspace. The approved deployment MUST apply that saved plan without recalculating it.

**Acceptance:** The workflow runs conceptually as:

```text
terraform plan -out=<environment>.tfplan
review and approve
terraform apply <environment>.tfplan
```

The plan artifact includes a checksum and commit SHA. The apply job rejects a mismatched commit, variable set, provider lock file, or stale plan. Applying a saved plan ensures Terraform performs only the reviewed changes. ([HashiCorp Developer][12])

---

### TF-04 — Mandatory validation pipeline

**Requirement:** Every pull request MUST pass Terraform formatting, initialization, validation, module tests, security or policy checks, and an environment-specific plan before merging.

**Acceptance:** At minimum, CI performs:

```text
terraform fmt -check -recursive
terraform init -backend=false
terraform validate
terraform test
terraform init
terraform plan
```

`terraform validate` MUST NOT be treated as Azure validation because it checks configuration consistency but does not query remote provider APIs. Apply-based Terraform tests run only in an isolated test subscription with cleanup controls. ([HashiCorp Developer][13])

---

### TF-05 — Strong module and input contracts

**Requirement:** Every input MUST have an explicit type, description, nullability decision, sensitivity classification, and validation rules. Modules MUST use preconditions and postconditions for assumptions that Terraform can verify.

**Acceptance:** Planning fails with a specific error for invalid regions, SKUs, CIDR ranges, names, resource IDs, environment combinations, or production settings. Preconditions are used for blocking requirements; nonblocking `check` blocks are not used where failure must stop the deployment. ([HashiCorp Developer][14])

---

### TF-06 — Explicit infrastructure ownership and import

**Requirement:** Every Azure resource referenced by Terraform MUST be classified as one of:

* Created and owned by this state.
* Imported and owned by this state.
* Read-only external dependency.
* Owned by another Terraform state.

Terraform MUST NOT attempt to create a resource that already exists outside its state.

**Acceptance:** Existing infrastructure uses declarative `import` blocks; read-only infrastructure uses data sources. Resource or module renames use `moved` blocks so refactoring does not cause unintended destroy-and-create operations. ([HashiCorp Developer][15])

---

### TF-07 — Dependency and stack-boundary design

**Requirement:** Terraform MUST express normal dependencies through direct resource references. `depends_on` MUST be used only for hidden behavioral dependencies Terraform cannot infer.

**Acceptance:** Foundation concerns—state storage, provider registration, deployment identities, shared networking, DNS, and organization-level policy—are separated from application infrastructure when they have different permissions or deployment lifecycles. Catch-all module-level `depends_on` declarations are prohibited without written justification. ([HashiCorp Developer][16])

---

### TF-08 — Controlled lifecycle and replacements

**Requirement:** Every resource that could be replaced MUST have its replacement behavior and outage impact documented. Critical persistent resources SHOULD use `prevent_destroy`; `create_before_destroy` MAY be used only where Azure naming, quotas, networking, and service behavior permit both resources to coexist.

**Acceptance:** The plan is rejected when it unexpectedly destroys or replaces a protected resource. `ignore_changes` is limited to explicitly named attributes owned by another system; `ignore_changes = all` is prohibited. Terraform lifecycle rules can prevent destruction, alter replacement ordering, or ignore selected external changes, but each has operational limitations. ([HashiCorp Developer][17])

---

### TF-09 — Timeouts and transient-failure policy

**Requirement:** Long-running Azure resources MUST define supported Terraform `timeouts` appropriate to the service. Retries MUST be bounded and restricted to documented transient conditions such as throttling, temporary conflicts, or authorization propagation.

**Acceptance:** The implementation MUST NOT use unlimited retries, blanket retry-on-error behavior, or arbitrary sleeps as a substitute for dependency modeling. For permission propagation, the pipeline performs an actual authorization test before proceeding. Terraform resources may expose configurable create, read, update, and delete timeouts, depending on provider support. ([HashiCorp Developer][18])

---

### TF-10 — Serialized apply, approvals, and failure evidence

**Requirement:** Only one Terraform apply may run against a given environment and state at a time. Production applies MUST use a protected GitHub environment with approved deployment branches and reviewers.

**Acceptance:** GitHub Actions uses an environment-specific concurrency key, such as:

```yaml
concurrency:
  group: terraform-${{ inputs.environment }}
  cancel-in-progress: false
```

On failure, the workflow preserves a sanitized plan, plan JSON, Terraform output, provider errors, Azure correlation or request IDs, and relevant Azure activity-log details. A retry generates a new plan; it MUST NOT apply a stale plan after infrastructure or state may have changed. GitHub environments provide protection rules, and concurrency groups can prevent simultaneous deployments to the same environment. ([GitHub Docs][19])
