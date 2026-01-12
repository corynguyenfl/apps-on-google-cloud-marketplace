# Executive Summary

**Evaluation of Offering OpenFMB-Based Software on Google Cloud Marketplace**

## Objective

Evaluate the technical, operational, and security readiness of the OpenFMB-based, containerized software platform for distribution via **Google Cloud Marketplace**, and identify risks and mitigation strategies.

---

## Overall Assessment

**The software is a strong candidate for Google Cloud Marketplace**, particularly as a **customer-hosted GKE application with subscription-only pricing**.
The existing architecture (containerized services, Kubernetes-native deployment, Keycloak-based identity, NATS messaging) aligns well with Marketplace expectations and requires **no fundamental redesign**.

The largest effort is not infrastructure compatibility, but **operational hardening and consistency** across CI/CD, security, entitlement enforcement, and documentation.

---

## Key Evaluation Findings

### Architecture & Deployment

* Fully containerized, Kubernetes-native design fits Marketplace’s preferred deployment model.
* Helm-based packaging is appropriate and expected.
* Terraform is optional and best positioned as a reference for infrastructure provisioning, not a requirement.

### Security & Identity

* Existing Keycloak-issued JWT model is compatible with Marketplace and enterprise security expectations.
* NATS authentication using pinned public keys is acceptable and secure.
* TLS is supported for UI, API, Keycloak, and NATS, with certificate lifecycle correctly owned by the customer.
* Key rotation is feasible using a broker-side, dual-key overlap strategy; automation can assist without removing operator control.

### Observability

* Migration from EFK (Elasticsearch, Fluent Bit, Kibana) to **Google Cloud Logging** significantly reduces operational complexity.
* No application code changes are required for basic logging.
* This aligns with cloud-native best practices and Marketplace reviewer preferences.

### Billing & Entitlement

* Subscription-only pricing is the correct initial model.
* Entitlement enforcement must be implemented consistently across services written in multiple languages (C++, Rust, Python, TypeScript).
* Helm-driven tier configuration with server-side enforcement is sufficient for Marketplace v1.

### Onboarding & Customer Engagement

* Automated deployment via Marketplace is appropriate.
* Circuit ingestion and OpenFMB adapter configuration require **vendor-assisted onboarding**, which is normal and expected for grid control software.
* Professional services can be packaged alongside Marketplace subscriptions without friction.

**NOTE**: Edge nodes are onboarded through a vendor-managed process. Edge devices authenticate using unique identities and establish outbound, TLS-secured connections to the cloud-hosted control plane. Marketplace deployment applies to cloud components only.

---

## Key Risks and Mitigations

### 1. **Entitlement Enforcement Consistency**

**Risk:** Inconsistent enforcement across services could lead to unauthorized feature use or customer disputes.
**Mitigation:**

* Define a single entitlement specification.
* Enforce server-side in all services.
* Add tests demonstrating enforcement behavior.

### 2. **Key Rotation Operational Errors**

**Risk:** Incorrect key rotation could cause authentication outages.
**Mitigation:**

* Use broker-side dual-key overlap.
* Keep rotation operator-approved.
* Provide a documented runbook and rollback procedure.

### 3. **Helm Chart Complexity**

**Risk:** Overly flexible charts increase install failures and support burden.
**Mitigation:**

* Start with opinionated defaults.
* Minimize required values.
* Validate install/upgrade/uninstall paths early.

### 4. **Marketplace Review Delays**

**Risk:** Review cycles may introduce schedule uncertainty.
**Mitigation:**

* Avoid over-claiming automation or “zero-touch” onboarding.
* Clearly document assisted configuration and support boundaries.
* Use private listings for early validation.

### 5. **Customer Onboarding Friction**

**Risk:** Customers may underestimate configuration complexity.
**Mitigation:**

* Explicitly document vendor-assisted onboarding.
* Include limited onboarding assistance or offer professional services packages.

---

## Estimated Delivery Timeline

With parallel execution across teams:

* **Foundation (CI/CD, Logging, Registry):** 1–2 weeks
* **Security & Identity Hardening:** 2–3 weeks
* **Helm/Kubernetes Packaging:** 2–3 weeks
* **Billing & Entitlement Integration:** 2–4 weeks
* **Documentation & Review:** 1–2 weeks

**Total (parallelized): ~6–8 weeks**

---

## Final Recommendation

Proceed with Google Cloud Marketplace onboarding using a **customer-hosted GKE, subscription-only model**.
Focus engineering effort on **entitlement consistency, key rotation robustness, and clear documentation**, rather than additional infrastructure automation.

This approach minimizes risk, aligns with Marketplace norms, and positions the platform for future expansion (including AWS Marketplace) with minimal rework.

---
