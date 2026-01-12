# Security & Identity

## Overview

The solution is deployed as a **customer-hosted Kubernetes application** within the customer’s Google Cloud project.
Security controls are designed to ensure **strong tenant isolation, least-privilege access, secure identity management, and auditable operations**, while aligning with enterprise and utility operational requirements.

The platform follows a **defense-in-depth** approach across identity, access control, network security, secrets management, and operational processes.

---

## Identity & Authentication

### User Authentication

* User authentication is handled by a **dedicated Keycloak instance deployed per customer**.
* Each customer deployment is **single-tenant**, ensuring strict isolation of identities and credentials.
* Keycloak supports federation with external identity providers (e.g., Google Workspace, Azure AD, Okta) using standard OIDC/SAML protocols.
* User authentication tokens are **JWTs issued by Keycloak**.

### Service Authentication

* All internal services authenticate using **Keycloak-issued JWTs**.
* Backend services use **client credentials** (non-user identities) with narrowly scoped roles.
* No shared or hard-coded credentials are used between services.

---

## Authorization Model

### Application-Level Authorization

* Authorization is **role-based**, using claims embedded in JWTs:

  * `realm_access.roles`
  * `resource_access.<client_id>.roles`
* Roles are coarse-grained to reduce complexity and risk:

  * Viewer (read-only)
  * Operator (control and dispatch)
  * Administrator (configuration and management)

### Messaging Authorization (NATS)

* NATS uses JWTs as user credentials.
* Authorization is enforced through **subject-level publish/subscribe permissions** derived from JWT role claims.
* This ensures that control operations (e.g., breaker operations, DER setpoints) are restricted to authorized roles only.

---

## JWT Validation & Trust Model

* NATS validates JWTs using **pinned public signing keys** from the customer’s Keycloak realm.
* Dynamic key discovery (JWKS) is intentionally not used to ensure deterministic behavior and offline validation.
* JWT validation includes:

  * Issuer verification
  * Audience validation
  * Expiry enforcement
  * Signature verification

### Key Rotation

* A documented, zero-downtime key rotation procedure is supported:

  1. Introduce a new signing key in Keycloak while keeping the existing key active.
  2. Export both public keys.
  3. Update NATS configuration to trust both keys.
  4. Roll dependent workloads.
  5. Retire the old key.
* This approach ensures uninterrupted service during key rotation events.

---

## Infrastructure Identity & Access (Google Cloud IAM)

### Workload Identity

* All Kubernetes workloads use **GKE Workload Identity** to authenticate to Google Cloud services.
* No static Google Cloud service account keys are stored in containers or configuration files.

### Cloud Resource Access

* Access to Google Cloud resources (e.g., Secret Manager, Artifact Registry, Cloud Storage) is controlled using:

  * Dedicated service accounts
  * Least-privilege IAM roles
* IAM policies are scoped at the project or resource level as appropriate.

---

## Secrets Management

* Sensitive values (e.g., database credentials, Keycloak admin credentials, TLS keys) are not hard-coded.
* Secrets are stored using:

  * Kubernetes Secrets (baseline)
  * Optionally integrated with Google Secret Manager using Workload Identity
* Keycloak administrative credentials are:

  * Generated at deployment time
  * Stored securely
  * Not exposed in logs or container images

---

## Network Security

* All external access to the UI, APIs, and Keycloak is secured using **TLS (HTTPS)**.
* Internal service communication is restricted using Kubernetes networking controls.
* NATS client connections are configured to use **TLS** where supported.
* Customers retain full control over network topology, firewall rules, and private connectivity.

---

## Tenant Isolation

* Each customer deployment is isolated at the infrastructure level:

  * Separate GKE cluster or namespace
  * Separate Keycloak instance
  * Separate messaging, storage, and configuration
* No customer data or credentials are shared across deployments.

---

## Auditability & Logging

* Authentication and authorization decisions are logged by:

  * Keycloak (login events, token issuance)
  * Application services (access decisions, control actions)
* Logs are compatible with customer-managed logging and SIEM systems.
* Customers retain ownership of all operational logs within their project.

---

## Secure Development & Operations

* No default or shared passwords are used.
* Images are built from minimal base images and scanned for vulnerabilities.
* Upgrades and patches are delivered via versioned container images and Helm charts.
* Security updates can be applied independently of functional releases.

---

## Compliance Alignment

While no specific compliance certifications are required to deploy the solution, the architecture aligns with common enterprise and utility security expectations, including:

* Least privilege access
* Strong tenant isolation
* Secure credential handling
* Auditable access and control actions

---

## Summary

The solution employs a **hybrid identity model**:

* **Keycloak** for application and messaging authentication and authorization
* **Google Cloud IAM** for infrastructure and workload identity

This approach provides strong security guarantees, operational flexibility, and compatibility with Google Cloud Marketplace requirements, while minimizing architectural disruption and operational risk.
