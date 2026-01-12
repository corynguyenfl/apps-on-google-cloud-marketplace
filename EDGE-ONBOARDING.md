# Recommended Ways to Onboard Edge Nodes

## Core Principle (Very Important)

> **Edge onboarding is vendor-managed and security-controlled; Marketplace only handles cloud-side deployment.**

Marketplace is **not** responsible for edge provisioning, identity, or configuration.
That is normal and expected.

---

## The 3 Common Edge Onboarding Patterns

### **Pattern 1 — Pre-Provisioned Edge (Most Common for Utilities)**

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

📌 **This is the most defensible approach for critical infrastructure.**

---

### **Pattern 2 — Bootstrap + Registration (Self-Service Assisted)**

**Good for scaling later**

**How it works**

1. Edge boots with:

   * minimal agent
   * bootstrap token (short-lived)
2. Edge calls cloud API:

   * “register me”
3. Cloud:

   * creates edge identity
   * returns credentials/config
4. Edge switches to normal operation

**Identity**

* Bootstrap token (one-time or time-limited)
* Permanent Keycloak identity issued after registration

**Pros**

* Scales better
* Less manual handling

**Cons**

* More engineering
* Requires careful security controls

📌 Often implemented after v1.

---

### **Pattern 3 — Manual Credential Injection**

**Least recommended, but sometimes unavoidable**

**How it works**

* Operator manually installs certs / JWTs
* Config files copied onto edge

**Pros**

* Simple
* No automation needed

**Cons**

* Error-prone
* Doesn’t scale
* Hard to audit

📌 Acceptable for pilots, not ideal long-term.

---

## Connectivity Model (Critical for Edge)

### Recommended

* **Outbound-only connections from edge**
* Edge connects to:

  * NATS (TLS)
  * optional control plane API

Why:

* No inbound firewall holes
* Works in substations
* Easier security approval

---

## Identity & Trust Model (Best Practice)

### Edge identity

Each edge node should have:

* unique identity
* scoped permissions (site/circuit-based)
* revocable credentials

### Authentication

* Keycloak issues JWTs
* JWT used for:

  * NATS connection
  * API calls
* NATS broker validates JWT using pinned public keys

### Revocation

* Disable edge client in Keycloak
* Token expiry + broker-side trust enforcement

---

## Configuration Delivery

Recommended mechanisms:

* Secure config pulled from cloud at startup
* Versioned configuration
* Read-only local config where possible

Avoid:

* hardcoded credentials
* long-lived secrets baked into images

---

## Recommended v1 Onboarding Model (For Your Platform)

### Cloud (Marketplace-installed)

* GKE cluster
* Keycloak
* NATS
* Core services

### Edge

* Vendor-provided container bundle
* Pre-registered Keycloak client
* Outbound TLS connection to NATS
* OpenFMB adapters configured with templates

### Onboarding flow

1. Customer installs cloud stack via Marketplace
2. Customer requests edge onboarding
3. Vendor provisions edge identity + config
4. Edge connects and is validated
5. Circuits and devices activated

---

## What to Document for Marketplace (Reviewer-Safe)

You should explicitly state:

> “Edge nodes are onboarded through a vendor-managed process. Edge devices authenticate using unique identities and establish outbound, TLS-secured connections to the cloud-hosted control plane. Marketplace deployment applies to cloud components only.”

This sets correct expectations and avoids review issues.

---

## Key Risks & Mitigations

| Risk                 | Mitigation                    |
| -------------------- | ----------------------------- |
| Credential leakage   | Short-lived JWTs, TLS         |
| Edge compromise      | Per-edge identity, revocation |
| Firewall constraints | Outbound-only model           |
| Scaling onboarding   | Move to bootstrap flow later  |

---

## Bottom Line

* **Vendor-managed edge onboarding is normal**
* **Pre-provisioned edge nodes are the safest v1**
* Marketplace does **not** handle edge devices
* Your Keycloak + NATS model is well-suited
* You can evolve toward self-service later
