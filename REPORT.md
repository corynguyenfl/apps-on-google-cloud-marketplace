# Evaluation Report - Offering OpenDSO Software on Google Cloud Marketplace

## 1. Summary

This report evaluates the feasibility, requirements, and recommended approach for offering our **containerized, OpenDSO software** on **Google Cloud Marketplace**.

**Recommenation:**
Offering the software as a **customer-hosted Kubernetes application (GKE / Helm-based)** on Google Cloud Marketplace is **technically feasible**, **well aligned with the existing architecture**.

An **externally hosted SaaS** offering is **not recommended as the initial approach** due to architectural, operational, and regulatory mismatches, but may be considered as a future extension.

## 2. Product Overview (Current State)

### Core Capabilities

* Fully containerized microservices architecture
* OpenFMB-based data and control plane
* OpenFMB adapters (DNP3 / MODBUS / OCPP1.6j → OpenFMB)
* NATS-based pub/sub event bus
* Web UI (one-line visualization, monitoring)
* Active device control:

  * Breaker / switch / recloser open-close
  * DER P/Q setpoints (ESS, PV, Generator)
* DER dispatch and optimization
* ESS management and automated testing
* Asset health monitoring, prediction, and maintenance recommendations
* Data viewer with embedded Grafana dashboard

### Key Architectural Characteristics

* Pub/Sub (NATS)
* Control-plane and data-plane separation
* Real-time control
* Single-tenant per deployment
* Designed for utility / critical-infrastructure environments

---

## 3. Marketplace Offer Model Evaluation

### Models Evaluated

| Model                                   | Fit       | Assessment                                                                                                                               |
| --------------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Externally Hosted SaaS                  | ❌ Poor    | Requires vendor-operated multi-tenant control plane; high operational burden; poor fit for field connectivity and regulatory constraints |
| VM-based Marketplace App                | ⚠️ Medium | Viable but less future-proof; weaker Kubernetes alignment                                                                                |
| **Kubernetes Application (GKE / Helm)** | ✅ Strong  | Best architectural fit; aligns with containerized, single-tenant design                                                                  |

### Recommneded Model

**Customer-hosted Kubernetes application deployed via Google Cloud Marketplace (GKE / Helm)**

**Rationale**

* Customers retain ownership of infrastructure, networking, and data
* Aligns with utility security, compliance, and operational expectations
* Avoids multi-tenant SaaS complexity
* Matches existing containerized deployment model

## 4. Target Marketplace Architecture

### Deployment Boundary

* One deployment per customer
* Single-tenant GKE cluster

### Core Components Deployed in Customer GKE

* Keycloak (per customer)
* NATS (with JWT-based authentication)
* OpenFMB adapters (DNP3 / MODBUS / OCPP1.6j)
* DER dispatch engine
* ESS management and testing services
* Asset health analytics pipeline
* API backend
* Topology services, Historian, Event Services
* UI frontend (Ingress-exposed)
* Supporting databases and stateful services
* Chart/Graph dashboard (data view)

## 5. Identity, Authentication, and Authorization

### Current Model (Retained)

* **One Keycloak instance per customer**
* Keycloak-issued JWTs used for:

  * UI authentication
  * API access
  * NATS user credentials

### NATS Integration

* NATS validates JWTs using **pinned public keys**
* No dynamic JWKS discovery
* Authorization driven by JWT claims:

  * `realm_access.roles`
  * `resource_access.${client_id}.roles`

### Authorization Model

* Coarse, role-based access:

  * Viewer
  * Operator
  * Admin
* Roles mapped to NATS subject permissions (publish/subscribe patterns)

### IAM Integration (Marketplace Expectation)

* **Google IAM is used for infrastructure and workloads**, not user auth:

  * GKE Workload Identity
  * Pod-level service accounts
  * Access to Secret Manager, Artifact Registry, and other GCP services
* No static cloud credentials baked into containers

**NOTE:**
This hybrid model (Keycloak for app/NATS auth, IAM for infrastructure) meets Google Cloud Marketplace security expectations without requiring disruptive re-architecture.

## 6. Security & Key Management Considerations

### Keycloak → NATS Public Key Pinning

Because NATS pins public keys:

* Key distribution must be explicit
* Key rotation must be operationally safe

**NOTE:** Key rotation can be automated. One approach is automation-assisted, operator-approved rotation: a scheduled job exports the Keycloak realm signing public keys and updates the NATS trusted-key configuration, while operators control the cutover and retirement of old keys using a dual-key overlap window. This provides repeatable rotation with minimal operational risk and supports rapid rollback.

### Required Controls

* Documented JWT contract (issuer, claims, expiry)
* Explicit role → subject permission mapping
* Secure Keycloak bootstrap (no default credentials)
* TLS for UI, API, Keycloak, and NATS connections
* Key rotation runbook with dual-key overlap support

### Key Rotation Strategy

1. Introduce new Keycloak signing key (keep old active)
2. Export both public keys
3. Update NATS configuration to trust both keys
4. Roll workloads
5. Retire old key

### Certs Management - The Normal Responsibility Model

| Component                   | TLS Termination         | Certificate Owner                                      |
| --------------------------- | ----------------------- | ------------------------------------------------------ |
| UI (Ingress)                | Ingress / Load Balancer | **Customer**                                           |
| API (Ingress)               | Ingress / Load Balancer | **Customer**                                           |
| Keycloak (Ingress)          | Ingress / Load Balancer | **Customer**                                           |
| Internal service-to-service | In-cluster (optional)   | OES-provided defaults                                  |
| NATS client connections     | NATS server             | OES provides mechanism, **customer provides certs**    |

**NOTE:** TLS is used to secure all external access to the UI, API, and Keycloak via Kubernetes Ingress. Certificate issuance and lifecycle management are customer-controlled, consistent with enterprise DNS and trust requirements.

**NOTE:** Internal communication can optionally be secured using TLS. NATS supports TLS for client connections, with certificates supplied by the customer or generated during deployment.

## 7. Deployment & Packaging Requirements

### Required Marketplace-Ready Assets

* Docker images in Artifact Registry, not Docker hub
* Helm charts
* Automated, reproducible install
* Config externalization (values.yaml)
* Secure secret handling
* Clear uninstall/upgrade paths

### Kubernetes Primitives

* Deployments / StatefulSets
* Services
* Ingress
* PersistentVolumeClaims
* Secrets / ConfigMaps
* Jobs (for Keycloak bootstrap)

## 8. Billing & Licensing Model

### Initial Model

**Subscription-based billing**

* Per-cluster subscription
* Tiered by:

  * Asset count
  * DER count
  * ESS count
  * Per app subscription (core apps, DER dispatch, ESS Manager, ESS Tester, Asset Health...)

### Rationale

* Minimal integration complexity
* Aligns with enterprise procurement
* Avoids early fine-grained metering complexity

### Licensing Enforcement

* Marketplace entitlement check at startup
* Periodic entitlement validation
* Feature gating via configuration

## 9. Operational & Support Considerations

### Customer Responsibilities

* Infrastructure availability
* Network configuration
* Cluster lifecycle

### OES Responsibilities

* Software updates
* Security patches
* Helm chart maintenance
* Support and escalation

### Operational Readiness

* Upgrade strategy documented
* Monitoring and logging guidance provided (Adopt Google Cloud Logging as the default logging solution)
* Incident response expectations defined

## 10. Onboarding and Configuration

The solution is delivered via **Google Cloud Marketplace** with automated deployment into the customer’s Google Cloud project.
While infrastructure deployment is fully automated, **initial system configuration and commissioning require domain-specific knowledge** and are typically performed **in collaboration with the vendor**.

This onboarding model is standard for **OpenDSO platforms**, where accurate circuit modeling and device configuration are critical for safe and reliable operation.

### Deployment (Automated)

After purchasing a subscription, customers can deploy the solution directly from Google Cloud Marketplace. The deployment process:

* Provisions all required Kubernetes resources
* Pulls vendor-managed container images from Google Artifact Registry
* Applies default configuration values
* Brings all services to a healthy running state

At this stage, the platform is **operational but unconfigured** with respect to customer-specific circuits, devices, and field integrations.

### Configuration & Commissioning (Vendor-Assisted)

#### Configuration Scope

Initial configuration typically includes:

* Circuit and feeder ingestion (CIM, OpenDSS, or OpenFMB models)
* Topology validation
* Adapter configuration for field devices (e.g., DNP3, MODBUS)
* Mapping of physical devices to logical assets
* NATS subject and routing configuration
* Validation of telemetry and control paths
* Safety checks and commissioning verification
* Edge nodes provisioning

Due to the complexity and critical nature of these tasks, configuration is **not fully automated** and requires collaboration between the customer and OES.

### Customer Responsibilities

Customers are responsible for:

* Providing accurate circuit models and device metadata and point lists
* Granting necessary network access to field systems
* Designating technical contacts for onboarding
* Validating operational behavior prior to production use

### OES Responsibilities

OES provides:

* Configuration guidance and best practices
* Reference configuration templates
* Assisted setup of OpenFMB adapters and circuit models
* Validation of measurement and control operations
* Support during initial commissioning

### Edge Node Provisioning

Edge onboarding is vendor-managed and security-controlled; Marketplace only handles cloud-side deployment.  Marketplace is **not** responsible for edge provisioning, identity, or configuration.
That is normal and expected.

#### Edge Onboarding Patterns

Pre-Provisioned Edge: 

**Recommended default**

**How it works**

* Vendor prepares edge node:

  * OS image or container bundle
  * NATS client
  * OpenFMB adapters
  * Base config template
* Edge is shipped or installed onsite
* Customer connects it to network
* Edge establishes outbound connection to cloud

**Identity**

* Edge has:

  * unique client ID
  * Keycloak client credentials or cert
* JWTs issued by Keycloak
* NATS auth uses JWT + pinned public key

**Pros**

* Predictable
* Secure
* Works behind firewalls/NAT
* Familiar to utilities

**Cons**

* Requires vendor involvement

---

### Typical Onboarding Flow

1. **Marketplace Deployment**

   * Customer installs the software via Google Cloud Marketplace
2. **Onboarding Kickoff**

   * Customer contacts OES to initiate configuration
3. **Configuration & Validation**

   * OES assists with circuit ingestion and adapter setup
4. **Commissioning**

   * End-to-end validation of monitoring and control
5. **Operational Handoff**

   * System transitions to customer operations

### Support & Engagement Model

Ongoing support is available according to the customer’s subscription and support agreement.
Advanced configuration, custom integrations, and large-scale deployments may require additional professional services.

### Packaging Professional Services Alongside Marketplace Subscriptions

Google Cloud Marketplace supports **software procurement**, not deep system integration.
We **need** to define package professional services outside of Marketplace billing.

## 11. Scalability

GKE provides built-in mechanisms for horizontal and vertical scalability, but applications must be designed and configured to leverage them.

What GKE Does Not Automatically Do:

* Decide which of the services should scale
* Know whether scaling is safe for the app
* Scale stateful services safely by default
* Tune limits and requests

The solution is deployed on GKE and leverages Kubernetes-native autoscaling mechanisms, including Horizontal Pod Autoscaling and Cluster Autoscaler, for stateless services. Vertical scaling is supported for selected components where appropriate.

## 12. Effort Estimate

### Estimated Engineering Effort

| Area                                   | Effort          |
| -------------------------------------- | --------------  |
| CI/CD for Google Artifact Registry     | 1-2 weeks       |
| Google Cloud Logging                   | 0.5 weeks       |
| Helm packaging & deployment automation | 3–4 weeks       |
| Auth hardening & key rotation support  | 3–4 weeks       |
| Billing & entitlement integration      | 3–4 weeks       |
| Documentation & Marketplace review     | 1-2 weeks       |
| **Total**                              | **~12–16 weeks**|

## 13. Risks & Mitigations

| Risk                                  | Impact     | Mitigation                        |
| ------------------------------------- | ---------- | --------------------------------- |
| Key rotation complexity (pinned keys) | Medium     | Dual-key strategy + runbook       |
| Marketplace review delays             | Medium     | Early checklist and documentation |
| Customer networking variability       | Medium     | Reference architecture            |
| Operational support load              | Low–Medium | Clear support scope               |

## 14. Supply Chain Security

The solution follows a **defense-in-depth supply chain security model** designed to protect against compromised build systems, tampered artifacts, and vulnerable dependencies. Controls span **source code, build pipelines, container images, registries, and runtime deployment**, and align with cloud-native best practices for enterprise and critical-infrastructure software.

### Source Code & Build Integrity

* All source code is maintained in version-controlled repositories with controlled access.
* Changes are reviewed prior to merge using standard code review processes.
* Build pipelines are fully automated and run in isolated CI environments.
* Build credentials are short-lived and scoped to the minimum required permissions.

### CI/CD Pipeline Security

* CI/CD pipelines authenticate to cloud services using **OIDC-based workload identity** rather than long-lived static credentials.
* Pipelines are restricted to publishing artifacts only to vendor-owned registries.
* Production images are built from **pinned base images** to ensure reproducibility.
* Immutable version tags are used for all production images; mutable tags (e.g., `latest`) are not relied upon for deployments.

### Container Image Registry & Integrity

* All container images are stored in **Google Artifact Registry** under a vendor-controlled Google Cloud project.
* Artifact Registry access is enforced using **Google IAM**, ensuring that only authorized build systems can push images.
* Customer deployments pull images through Marketplace-managed permissions; images are not publicly accessible.

### Vulnerability Scanning & Remediation

* Container images stored in Artifact Registry are **automatically scanned for known vulnerabilities** upon push.
* Vulnerability findings include:

  * CVE identifiers
  * Severity ratings (Low, Medium, High, Critical)
  * Affected packages and versions
* High and critical vulnerabilities are tracked and remediated through:

  * Base image updates
  * Dependency upgrades
  * Image rebuilds and redeployments
* Zero known vulnerabilities are not guaranteed at all times; instead, we will focus is on **timely remediation** and risk-based prioritization.

### Image Signing & Build Verification

* Container images are **cryptographically signed at build time** using
  **Sigstore cosign** with **keyless identity-based signing**.
* Image signatures are generated using GitHub Actions’ **OIDC identity**
  and are published **alongside container images** in the container registry.
* Each image signature is bound to:
  * The originating GitHub repository
  * The exact CI workflow file
  * The source branch used for the build
* Image signatures are **verified as part of the CI pipeline** to ensure
  signing correctness prior to release.
* Customers may independently verify image signatures using standard
  `cosign verify` workflows.
* Runtime enforcement of signed images is **customer-configurable** and
  may be enabled using Kubernetes admission controls (e.g., policy engines),
  if required.

### Dependency Management

* OS-level and language-level dependencies are explicitly defined and version-controlled.
* Dependency updates are incorporated through regular maintenance releases.
* Third-party libraries are monitored for security advisories as part of ongoing maintenance.

### Deployment & Runtime Trust

* Customer deployments pull images directly from Artifact Registry using IAM-controlled access.
* Images are referenced by **immutable digests or versioned tags** in Helm charts.
* No images are downloaded from anonymous or public registries during production deployment.
* Customers retain full control over runtime policies, including network restrictions and admission controls.

### Auditability & Transparency

* Artifact Registry provides audit logs for:

  * image uploads
  * access attempts
  * vulnerability scan results
* CI/CD pipelines produce auditable build logs.
* The vendor maintains documentation describing:

  * build and release processes
  * vulnerability response procedures
  * image signing and verification options

OpenDSO’s supply chain security model:

* Uses **trusted, cloud-native tooling**
* Minimizes reliance on public registries
* Provides **visibility, auditability, and control** over artifacts
* Supports enterprise and utility security expectations without introducing unnecessary operational complexity

This approach aligns with Google Cloud Marketplace requirements.

## 15. Current State vs Marketplace-Ready State

| Area                   | Current State                                 | Marketplace Requirement             | Gap             |
| ---------------------- | --------------------------------------------- | ----------------------------------- | --------------- |
| CI/CD & Images         | Docker Hub primary target                     | Google Artifact Registry            | **Medium**      |
| Logging                | EFK-based (Elasticsearch, Fluent Bit, Kibana) | Google Cloud Logging                | **Low–Medium**  |
| Authentication         | Keycloak per customer, JWT-based              | Hardened, documented, auditable     | **Medium**      |
| Key Rotation           | Implicit / manual                             | Explicit, documented, zero-downtime | **Medium–High** |
| Helm Packaging         | Partial / internal                            | Marketplace-ready Helm charts       | **High**        |
| Kubernetes Deployment  | Some POC done                                 | Deterministic, self-service         | **High**        |
| Billing & Entitlement  | -                                             | Enforced across all services        | **High**        |
| Documentation & Review | Internal docs                                 | Marketplace-grade                   | **High**        |

### 15.1 CI/CD → Google Artifact Registry

### What Needs to Be Done

* Update all build pipelines to:

  * authenticate to GCP
  * push images to Artifact Registry
* Update Helm charts to reference Artifact Registry images
* Ensure immutable tagging (no `latest` for production)

### Estimated Effort

* **1–2 days per repo**
* Parallelizable

### Deliverables

* CI templates updated
* Artifact Registry repo structure defined
* Verified multi-arch pushes

## 15.2 Logging: EFK → Google Cloud Logging

### What Needs to Be Done

* Remove Elasticsearch/Kibana
* Rely on GKE’s native logging agent
* Update documentation to reference Cloud Logging

### Code Impact

* **None required**
* Optional: structured JSON logging later

### Estimated Effort

* **2–3 days**

  * deployment cleanup
  * doc updates
  * retention policy definition

### Deliverables

* Cloud Logging queries documented
* Retention/exclusion guidance

---

## 15.3 Authentication Hardening (Keycloak + NATS)

### What Needs to Be Done

* Document JWT contract:

  * issuer
  * claims (`realm_access.roles`, `resource_access.*`)
* Ensure all services use:

  * client credentials (not user tokens)
* Enforce TLS everywhere
* Standardize role → permission mapping

### Key Decisions

* Keep Keycloak per customer
* Keep JWT-based NATS auth with pinned public keys

### Estimated Effort

* **1–2 weeks**

### Deliverables

* Auth architecture doc
* Role/permission matrix
* Security review checklist

## 15.4 Key Rotation (Keycloak → NATS)

### What Needs to Be Done

* Define public key distribution mechanism
* Support dual-key trust in NATS
* Document zero-downtime rotation process
* Optional automation for key export → secret update

### Why This Is High-Risk

Pinned keys mean **rotation must be explicit**.

### Estimated Effort

* **1–2 weeks**

### Deliverables

* Key rotation runbook
* Helm/K8s support for multiple public keys
* Operational checklist

## 15.5 Helm Packaging (Marketplace-Grade)

### What Needs to Be Done

* Create a top-level Helm chart with subcharts:

  * Keycloak
  * NATS
  * Adapters
  * Core services
  * UI
* Externalize all config via `values.yaml`
* Support tier-based configuration (billing)

### Key Decisions

* Single-tenant per install
* Opinionated defaults, overrideable values

### Estimated Effort

* **3–4 weeks**

### Deliverables

* Marketplace-ready Helm chart
* Installation/upgrade/uninstall paths
* Values reference documentation

## 15.6 Kubernetes Deployment Hardening

### What Needs to Be Done

* Validate:

  * readiness/liveness probes
  * resource requests/limits
  * node affinity (esp. stateful components) (what node to run on based on labels)
* Ensure clean install/uninstall
* Verify upgrades don’t break state

### Estimated Effort

* **1–2 weeks**
* Overlaps with Helm work

### Deliverables

* Production-ready deployment profiles
* Reference architecture diagram
* Capacity guidance

## 15.7 Billing & Entitlement Integration (Multi-Language)

### What Needs to Be Done

* Define **single entitlement model**:

  * tier
  * limits
  * enabled features
* Implement enforcement in:

  * Rust services
  * C++ services
  * Python workers
  * TypeScript UI
* Ensure server-side enforcement (not UI-only)
* Define cancellation behavior

### Key Decisions

* Subscription-only
* Helm-driven tier selection

### Estimated Effort

* **2–4 weeks** (this is the heaviest lift)

### Deliverables

* Entitlement spec
* Shared enforcement guidelines
* Tests proving limits are enforced
* `/entitlement` endpoint for UI

## 15.8 Documentation & Marketplace Review

### What Needs to Be Done

* Write Marketplace-facing docs:

  * Security & Identity
  * Billing & Entitlement
  * Onboarding & Configuration
  * Support model
* Prepare reviewer test plan
* Respond to review feedback

### Estimated Effort

* **1–2 weeks** (often overlaps with review cycles)

### Deliverables

* Complete Marketplace submission package
* Reviewer Q&A responses
* Final approval
