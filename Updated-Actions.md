# Updated Recommendation

## ✅ Keep Keycloak-per-customer + NATS pinned public keys

This is still a good fit for **customer-hosted GKE Marketplace app**.

**Key implication:** you must define a safe, repeatable way to:

1. export Keycloak realm signing public key(s)
2. load them into NATS config
3. rotate keys without breaking clients

---

# Day-by-Day Execution Plan (Revised, 5 Days)

## **Day 1 — Scope + Offer Type + Success Criteria**

**Do**

* Confirm offer type: **Marketplace Kubernetes app (GKE/Helm)**
* Confirm deployment boundary: **single-tenant per customer**
* Define “Done” for spike: architecture, security, packaging, effort, risks

**Output**

* 1-page scope + chosen offer type + assumptions

---

## **Day 2 — Architecture & Deployment Packaging**

**Do**

* Draw target Marketplace architecture (components + namespaces)
* Identify minimum deployable set:

  * Keycloak
  * NATS
  * Adapters (DNP3/MODBUS → OpenFMB)
  * Core services (dispatch, ESS mgmt, health)
  * UI + API gateway/ingress
  * DBs (if any)
* Decide reference deployment:

  * single cluster
  * standard ingress (NGINX / GKE Ingress)
  * storage class assumptions

**Output**

* “Current vs Target” architecture diagram
* Component inventory + required K8s primitives (Deployments, StatefulSets, PVCs, Ingress, Secrets)

---

## **Day 3 — Auth Deep Dive: Keycloak → JWT → NATS (Pinned Key Model)**

This is your highest-value day.

### A) Define JWT Contract (based on your claims)

**Do**

* Document required claims:

  * `iss`, `aud`, `exp`, `iat`
  * `realm_access.roles`
  * `resource_access.${client_id}.roles`
* Define which client_id(s) you care about:

  * UI client
  * API client
  * Service clients (adapter, dispatch, health, etc.)

**Output**

* JWT contract doc + examples of expected roles

### B) Define NATS authorization model

**Do**

* Map roles → NATS permissions (subjects)
* Decide if you use:

  * coarse roles: viewer/operator/admin (recommended)
  * or many fine roles

**Output**

* Role → subject matrix (pub/sub patterns)

### C) Solve pinned public key distribution (key point)

Because NATS doesn’t do JWKS, you need a plan.

**Do**

* Decide where the Keycloak realm public key lives:

  * Option 1: Helm chart values (simple, but manual)
  * Option 2: init job extracts from Keycloak and writes to a secret (better)
  * Option 3: operator manages it (future)

**Output**

* Key distribution/rotation plan (with failure modes)

---

## **Day 4 — Security, Ops, and Rotation Strategy**

### A) Key rotation (must-have)

With pinned keys, rotation is the main risk.

**Do**

* Document a safe rotation runbook:

  1. Add new Keycloak key pair (keep old active)
  2. Export **both public keys**
  3. Update NATS pinned keys list (accept both)
  4. Roll clients
  5. Remove old key

**Output**

* Rotation runbook + operational checklist

### B) Secrets handling + bootstrap

**Do**

* Define how Keycloak is bootstrapped:

  * realm import
  * admin password creation
  * service clients creation
* Define secret storage:

  * K8s Secrets baseline
  * optional integration with GCP Secret Manager using Workload Identity

**Output**

* Bootstrap diagram + secrets plan

---

## **Day 5 — Marketplace Fit, Effort Estimate, Risks, and Next Steps**

**Do**

* Decide “Go / Conditional Go / No-Go”
* Effort estimate as Epic/Stories (deployment, auth, docs, billing)
* Identify blockers and mitigations

**Output**

* Final spike summary (decision-grade)
* Implementation roadmap (6–8 weeks typical)

---

# Questions You Should Prepare (and your best current answers)

## Auth / NATS

1. **How will NATS get Keycloak public keys?**
   **Answer:** NATS pins them; we must distribute via Helm values or generated secret.
2. **Which claims define permissions?**
   **Answer:** `realm_access.roles` and `resource_access.${client_id}.roles`.
3. **Do services use client credentials?**
   **Best practice answer:** Yes, each service is a Keycloak client with minimal roles.

## Operational

4. **How often do we rotate Keycloak signing keys?**
   **Target:** Rare (e.g., quarterly/annual), but we must support zero-downtime rotation.
5. **What happens if Keycloak changes keys unexpectedly?**
   **Mitigation:** Monitor, keep old key enabled until NATS updated.

## Marketplace reviewers

6. **Do we avoid default passwords?**
   **Target:** Yes; admin password generated once and stored as a secret.
7. **Is network access secured?**
   **Target:** TLS for UI/API/Keycloak; TLS for NATS connections recommended.

---

# Filled-In Answers Based on Your Architecture

## Would kube-based SaaS (customer-hosted GKE app) be a good fit?

✅ **Yes**

* Your system is container-native
* Single-tenant per customer is aligned with utility deployments
* Keycloak-per-customer aligns better with customer security posture than centralized SaaS

## Is externally hosted SaaS a good fit now?

⚠️ **Not as the first move**

* Device connectivity + operational control favors customer-hosted installs
* Multi-tenant SaaS would require major changes (tenant isolation, centralized ops, shared Keycloak or multi-tenant realms)

---

# Biggest Risk to Call Out in the Spike

**Pinned public keys in NATS** → rotation complexity.

But it’s manageable if you implement:

* dual-key acceptance window
* runbook + automation hooks
* clear operational ownership
