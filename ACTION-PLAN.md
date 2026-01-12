# ✅ Executive Answer (Up Front)

**Yes — a Kubernetes-based SaaS offer is a *very good* fit for our software**, and likely the **best Marketplace entry path**, with **SaaS (externally hosted) as a second-phase option**.

Why:

* Fully containerized
* Event-driven (NATS)
* Control-plane + data-plane separation is natural
* UI + analytics + control favors centralized SaaS
* Utilities and enterprises are comfortable deploying GKE workloads via Marketplace

---

## 📅 Day-by-Day Execution Plan (SPIKE)

### **Day 1 — Product & Marketplace Model Alignment**

#### Goals

* Lock scope
* Select viable Marketplace model(s)
* Kill bad options early

#### Tasks

1. **Document current product boundaries**

   * Control plane (UI, orchestration, analytics)
   * Data plane (adapters, NATS, edge connectivity)
2. Map product to Marketplace offer types:

   * VM-based ❌
   * GKE / Helm-based ✅
   * Externally hosted SaaS ⚠️ (future)

#### Outputs

* **Chosen primary model:**
  👉 *Google Cloud Marketplace – Kubernetes Application (GKE / Helm)*

#### Filled-in Answers (Based on our Architecture)

| Question                                    | Answer                              |
| ------------------------------------------- | ----------------------------------- |
| Where does software run?                    | Containers on Kubernetes            |
| Is customer infra required?                 | Yes (GKE in customer project)       |
| Is software control-plane heavy?            | Yes                                 |
| Does it require low-latency device control? | Yes (NATS + adapters)               |
| Is SaaS-only realistic today?               | Not ideal due to field connectivity |

---

### **Day 2 — Architecture Gap Analysis (Current vs Marketplace)**

#### Goals

* Identify what must change to be Marketplace-ready
* Confirm Kubernetes design viability

#### Tasks

1. Draw **current architecture**
2. Draw **Marketplace target architecture**
3. Identify gaps:

   * Install automation
   * Identity
   * Multi-tenant assumptions
   * Networking

#### Target Marketplace Architecture (Filled In)

**Customer GKE Cluster**

* OpenFMB Adapters (DNP3, MODBUS)
* NATS (clustered or leaf-node)
* DER dispatch engine
* ESS manager
* ESS tester
* Asset health pipeline
* API backend
* UI frontend (Ingress)
* Keycloak

**Optional Vendor-Hosted**

* Model training
* Fleet analytics
* Licensing / entitlement
* Update service

#### Gaps Identified

| Area       | Gap                             |
| ---------- | ------------------------------- |
| Deployment | Need Helm charts / Terraform    |
| Identity   | Need GCP IAM + app auth mapping |
| Ingress    | Standardize on HTTPS / certs    |
| Config     | Externalize all configs         |
| Tenancy    | Single-tenant per cluster (OK)  |

---

### **Day 3 — Billing, Licensing & Control Plane Impact**

#### Goals

* Understand how Google bills *us*
* Understand how *we* enforce licensing

#### Tasks

1. Evaluate Marketplace billing models
2. Map to our product value units
3. Identify required runtime changes

#### Billing Model Fit

**Best fit for us initially:**

* **Subscription-based**

  * Per cluster
  * Per DER count tier
  * Per ESS count tier

**Avoid initially:**

* Fine-grained metering (high effort)

#### Filled-In Billing Answers

| Question                                      | Answer                             |
| --------------------------------------------- | ---------------------------------- |
| Does software already meter usage?            | Partially (asset counts, sessions) |
| Can usage be coarse-grained?                  | Yes                                |
| Is runtime license enforcement feasible?      | Yes                                |
| Can system operate if billing API is delayed? | Yes                                |

#### Required Work

* Add Marketplace entitlement check at startup
* Periodic license validation
* Feature gating (scale limits)

---

### **Day 4 — Security, Compliance & Ops Readiness**

#### Goals

* Ensure we won’t fail Marketplace review
* Identify documentation gaps

#### Tasks

1. Review Marketplace security expectations
2. Compare against current posture
3. Identify documentation needed

#### Security Fit (Good News)

our architecture already aligns well:

| Requirement             | Status                            |
| ----------------------- | --------------------------------- |
| Containerized workloads | ✅                                 |
| IAM-based access        | ⚠️ (needs mapping)                |
| Secrets handling        | ⚠️ (Secret Manager / K8s Secrets) |
| Network isolation       | ✅                                 |
| Customer data isolation | ✅ (per cluster)                   |

#### Required Docs

* Security overview
* Data flow diagram
* Upgrade strategy
* Support escalation process

---

### **Day 5 — Effort Estimate, Risks & Recommendation**

#### Goals

* Close the SPIKE with decision-quality output

### Engineering Effort Estimate

| Area                | Effort         |
| ------------------- | -------------- |
| Helm + deployment   | 2–3 weeks      |
| Billing integration | 1–2 weeks      |
| Auth/IAM mapping    | 1 week         |
| Docs & review       | 1 week         |
| **Total**           | **~5–7 weeks** |

#### Risks

| Risk                      | Mitigation                  |
| ------------------------- | --------------------------- |
| Marketplace review delays | Early checklist             |
| Billing complexity        | Start subscription-only     |
| Customer networking       | Reference architecture      |
| Operational load          | Limit initial support scope |

---

## ❓ Key Questions + Pre-Filled Answers

### **Is Kubernetes-based SaaS a good fit?**

✅ **Yes — best initial Marketplace model**

### **Should UI be included?**

✅ Yes — Marketplace customers expect a full solution

### **Should NATS be bundled?**

✅ Yes — core to architecture

### **Should adapters run in-cluster?**

✅ Yes (primary)
⚠️ Edge mode can be phase 2

### **Multi-tenant or single-tenant?**

👉 **Single-tenant per customer cluster** (best for utilities)

---

## 🧭 Final Recommendation

### **Recommended Path**

1. **Phase 1:**
   Kubernetes Application via Google Cloud Marketplace

   * Helm-based
   * Subscription pricing
   * Single-tenant per customer

2. **Phase 2:**
   Hybrid SaaS control plane (optional)

   * Central analytics
   * Model training
   * Fleet insights

### **Go / No-Go**

✅ **GO** — strong technical fit, manageable effort, high enterprise credibility
