# Key Rotation Strategy

*(Keycloak → JWT → NATS with Pinned Public Keys)*

## 1. Scope and Threat Model

### What keys are rotated

* **Keycloak realm signing keys** used to sign JWTs
* These keys are trusted by:

  * application services
  * NATS (via pinned public keys)

### Why rotation matters

* Limits blast radius if a key is compromised
* Meets enterprise and utility security expectations
* Required for long-lived deployments

### Constraints

* NATS does **not** support dynamic JWKS discovery
* Public keys must be explicitly configured
* Rotation must be **zero-downtime**

## 2. Design Principles

The rotation design follows these principles:

1. **No downtime**
2. **No token invalidation during rotation**
3. **Explicit trust management**
4. **Reversible until final cutover**
5. **Auditable and repeatable**

This is achieved using **dual-key overlap**.

## 3. High-Level Rotation Flow

```text
Time ───────────────────────────────────────────────>

Keycloak:
  [ OLD KEY ]──────────────┐
                            ├── overlap window
  [ NEW KEY ]──────────────┘

NATS trust:
  [ OLD PUBKEY ]────────────┐
                              ├── overlap window
  [ NEW PUBKEY ]────────────┘

Clients:
  Old JWTs continue working
  New JWTs accepted immediately
```

## 4. Step-by-Step Rotation Procedure

### Step 1 — Generate a New Signing Key in Keycloak

In the customer’s Keycloak realm:

* Generate a **new active signing key**
* Keep the **old signing key enabled**
* Do **not** delete or disable the old key yet

Keycloak now:

* Can issue JWTs signed with either key
* Will gradually start using the new key (depending on Keycloak config)

✅ **No service disruption at this step**

### Step 2 — Export Public Keys

From Keycloak:

* Export the **public key for the old signing key**
* Export the **public key for the new signing key**

These are:

* public material only
* safe to store in Kubernetes Secrets

### Step 3 — Update NATS Trust Configuration (Dual-Key Mode)

Update NATS configuration to trust **both** public keys:

* Add the new public key
* Keep the old public key
* Restart or reload NATS (depending on configuration)

At this point, NATS:

* Accepts JWTs signed by **either key**
* Continues to function normally

✅ **Zero downtime**

### Step 4 — Roll Dependent Services (Optional but Recommended)

Optionally:

* Restart application pods
* Ensure all services fetch fresh tokens from Keycloak

This step:

* Accelerates adoption of the new signing key
* Is not strictly required for correctness

### Step 5 — Observe the Overlap Period

Allow an overlap window (e.g. hours to days):

* Existing JWTs signed with the old key continue to work
* Newly issued JWTs use the new key
* Monitoring should show:

  * successful authentication
  * no NATS auth failures

This window provides a **safe rollback opportunity**.

### Step 6 — Retire the Old Key

Once confident:

1. Remove the **old public key** from NATS
2. Disable or delete the **old signing key** in Keycloak

At this point:

* Only the new key is trusted
* Rotation is complete

## 5. Rollback Strategy (If Something Goes Wrong)

If authentication issues are detected:

* Re-enable the old signing key in Keycloak
* Re-add the old public key to NATS
* Restart affected services

Because rotation uses overlap:

> **Rollback is immediate and safe**

## 6. Automation vs Manual Execution

### Manual (Acceptable for v1)

* Key rotation performed by operator
* Public keys updated via Helm values or Kubernetes Secrets
* Documented runbook followed

### Automated (Optional, Later)

* Job or script to:

  * export Keycloak public keys
  * update Kubernetes Secret
  * trigger NATS reload
* Can be added later without architectural change

Marketplace reviewers **do not require automation**, only a **documented and safe process**.

## 7. Operational Guidelines

### Rotation Frequency

* Recommended:

  * annually
  * or upon security event
* No need for aggressive rotation unless policy requires it

### Monitoring

* Monitor:

  * NATS auth failures
  * token validation errors
  * Keycloak signing activity

### Access Control

* Only privileged operators can:

  * generate signing keys
  * update trusted public keys
* Actions should be auditable

## 8. Reviewer-Friendly Summary (Reusable)

> “JWT signing keys are rotated using a dual-key overlap strategy. New signing keys are introduced while existing keys remain trusted, and NATS is configured to accept both public keys during the transition. This allows zero-downtime rotation and safe rollback before retiring the old key.”

This statement directly answers:

* *How do you rotate keys?*
* *Is there downtime?*
* *Is it safe?*

## 9. Why This Is the Correct Approach

* Works with pinned public keys
* No dependency on JWKS
* Common in regulated environments
* Used by many production NATS + OIDC deployments
* Aligns with Marketplace and enterprise security expectations

## Bottom Line

* **Key rotation is vendor-supported, customer-executed**
* Uses **dual-key overlap**
* Causes **no downtime**
* Is **auditable and reversible**
* Does **not require architectural changes**

Key rotation can be automated. The recommended approach is automation-assisted, operator-approved rotation: a scheduled job exports the Keycloak realm signing public keys and updates the NATS trusted-key configuration, while operators control the cutover and retirement of old keys using a dual-key overlap window. This provides repeatable rotation with minimal operational risk and supports rapid rollback.
