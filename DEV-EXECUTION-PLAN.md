# Gap Analysis & Execution Plan

**Google Cloud Marketplace Enablement**

---

## 1. Gap Analysis (Current State vs Marketplace-Ready State)

### Summary Table

| Area                   | Current State                                 | Marketplace Requirement             | Gap             |
| ---------------------- | --------------------------------------------- | ----------------------------------- | --------------- |
| CI/CD & Images         | Docker Hub primary target                     | Google Artifact Registry            | **Medium**      |
| Logging                | EFK-based (Elasticsearch, Fluent Bit, Kibana) | Google Cloud Logging                | **Low–Medium**  |
| Authentication         | Keycloak per customer, JWT-based              | Hardened, documented, auditable     | **Medium**      |
| Key Rotation           | Implicit / manual                             | Explicit, documented, zero-downtime | **Medium–High** |
| Helm Packaging         | Partial / internal                            | Marketplace-ready Helm charts       | **High**        |
| Kubernetes Deployment  | Some POC done                                 | Deterministic, self-service         | **High**        |
| Billing & Entitlement  | Not unified                                   | Enforced across all services        | **High**        |
| Documentation & Review | Internal docs                                 | Marketplace-grade                   | **High**        |

## 2. Execution Plan (By Workstream)

Each section includes:

* **What needs to be done**
* **Key decisions**
* **Estimated effort**
* **Deliverables**

## 2.1 CI/CD → Google Artifact Registry

### What Needs to Be Done

* Update all build pipelines to:

  * authenticate to GCP
  * push images to Artifact Registry
* Update Helm charts to reference Artifact Registry images
* Ensure immutable tagging (no `latest` for production)

### Key Decisions

* Use **Workload Identity Federation** for CI (recommended)
* Keep Docker Hub only for dev (optional)

### Estimated Effort

* **1–2 days per repo**
* Parallelizable

### Deliverables

* CI templates updated
* Artifact Registry repo structure defined
* Verified multi-arch pushes

## 2.2 Logging: EFK → Google Cloud Logging

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

## 2.3 Authentication Hardening (Keycloak + NATS)

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

## 2.4 Key Rotation (Keycloak → NATS)

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

## 2.5 Helm Packaging (Marketplace-Grade)

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

* **2–3 weeks**

### Deliverables

* Marketplace-ready Helm chart
* Installation/upgrade/uninstall paths
* Values reference documentation

## 2.6 Kubernetes Deployment Hardening

### What Needs to Be Done

* Validate:

  * readiness/liveness probes
  * resource requests/limits
  * node affinity (esp. stateful components)
* Ensure clean install/uninstall
* Verify upgrades don’t break state

### Estimated Effort

* **1–2 weeks**
* Overlaps with Helm work

### Deliverables

* Production-ready deployment profiles
* Reference architecture diagram
* Capacity guidance

## 2.7 Billing & Entitlement Integration (Multi-Language)

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
* Helm-driven tier selection (v1)
* Optional signed entitlement token (v1.5)

### Estimated Effort

* **2–4 weeks** (this is the heaviest lift)

### Deliverables

* Entitlement spec
* Shared enforcement guidelines
* Tests proving limits are enforced
* `/entitlement` endpoint for UI

## 2.8 Documentation & Marketplace Review

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

## 3. Integrated Timeline (Realistic)

| Phase                    | Duration      |
| ------------------------ | ------------- |
| CI/CD + Logging          | 1–2 weeks     |
| Auth + Key Rotation      | 2–3 weeks     |
| Helm + K8s Hardening     | 2–3 weeks     |
| Billing & Entitlement    | 2–4 weeks     |
| Docs + Review            | 1–2 weeks     |
| **Total (parallelized)** | **6–8 weeks** |

## 4. Risk Assessment

| Risk                         | Impact | Mitigation                   |
| ---------------------------- | ------ | ---------------------------- |
| Key rotation failure         | High   | Dual-key strategy + runbook  |
| Entitlement inconsistencies  | High   | Shared spec + tests          |
| Helm complexity              | Medium | Start opinionated            |
| Marketplace review delays    | Medium | Clear docs, realistic claims |
| Customer onboarding friction | Medium | Assisted onboarding model    |

## 5. Recommended Execution Strategy

### Phase 1 (Foundation)

* Artifact Registry
* Cloud Logging
* Helm skeleton
* Auth documentation

### Phase 2 (Control & Monetization)

* Entitlement enforcement
* Key rotation
* Tier gating

### Phase 3 (Marketplace Readiness)

* Testing
* Docs
* Review
* Private listing testing

## 6. Final Assessment

**This is achievable without architectural rewrites.**
The largest risks are **entitlement consistency** and **key rotation**, not Kubernetes or GCP itself.
