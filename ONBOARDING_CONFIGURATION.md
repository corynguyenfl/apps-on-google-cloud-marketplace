# Onboarding & Configuration

## Overview

The solution is delivered via **Google Cloud Marketplace** with automated deployment into the customer’s Google Cloud project.
While infrastructure deployment is fully automated, **initial system configuration and commissioning require domain-specific knowledge** and are typically performed **in collaboration with the vendor**.

This onboarding model is standard for **grid control, DER management, and OpenFMB-based platforms**, where accurate circuit modeling and device configuration are critical for safe and reliable operation.

---

## Deployment (Automated)

After purchasing a subscription, customers can deploy the solution directly from Google Cloud Marketplace. The deployment process:

* Provisions all required Kubernetes resources
* Pulls vendor-managed container images from Google Artifact Registry
* Applies default configuration values
* Brings all services to a healthy running state

At this stage, the platform is **operational but unconfigured** with respect to customer-specific circuits, devices, and field integrations.

---

## Configuration & Commissioning (Vendor-Assisted)

### Configuration Scope

Initial configuration typically includes:

* Circuit and feeder ingestion (CIM, OpenDSS, or OpenFMB models)
* OpenFMB topology validation
* Adapter configuration for field protocols (e.g., DNP3, MODBUS)
* Mapping of physical devices to logical assets
* NATS subject and routing configuration
* Validation of telemetry and control paths
* Safety checks and commissioning verification

Due to the complexity and critical nature of these tasks, configuration is **not fully automated** and requires collaboration between the customer and the vendor.

---

## Customer Responsibilities

Customers are responsible for:

* Providing accurate circuit models and device metadata
* Granting necessary network access to field systems
* Designating technical contacts for onboarding
* Validating operational behavior prior to production use

---

## Vendor Responsibilities

The vendor provides:

* Configuration guidance and best practices
* Reference configuration templates
* Assisted setup of OpenFMB adapters and circuit models
* Validation of telemetry and control operations
* Support during initial commissioning

---

## Typical Onboarding Flow

1. **Marketplace Deployment**

   * Customer installs the software via Google Cloud Marketplace
2. **Onboarding Kickoff**

   * Customer contacts vendor to initiate configuration
3. **Configuration & Validation**

   * Vendor assists with circuit ingestion and adapter setup
4. **Commissioning**

   * End-to-end validation of monitoring and control
5. **Operational Handoff**

   * System transitions to customer operations

---

## Support & Engagement Model

Ongoing support is available according to the customer’s subscription and support agreement.
Advanced configuration, custom integrations, and large-scale deployments may require additional professional services.

---

# Packaging Professional Services Alongside Marketplace Subscriptions

Google Cloud Marketplace supports **software procurement**, not deep system integration.
It is **normal and accepted** to package professional services **outside of Marketplace billing**.

Below are **reviewer-safe, customer-friendly approaches**.

---

## Option 1 — Included Assisted Onboarding (Recommended for v1)

### Model

* Marketplace subscription includes:

  * Limited onboarding assistance (e.g., fixed hours or scope)
* Advanced or custom work is scoped separately

### Example

> “Subscription includes up to 10 hours of assisted onboarding for initial configuration.”

### Pros

* Low friction for customers
* Strong early success rate
* Simple messaging

### Cons

* Requires clear scope definition

---

## Option 2 — Professional Services Add-On (Very Common)

### Model

* Marketplace handles software subscription only
* Configuration and commissioning sold via:

  * Statement of Work (SOW)
  * Purchase Order (PO)
  * Services agreement

### Example Services

* Circuit ingestion and validation
* Adapter configuration
* Custom device mappings
* On-site or remote commissioning
* Operator training

### Pros

* Clean separation of concerns
* Scales well for complex deployments
* Familiar to utilities and enterprises

### Cons

* Slightly longer procurement path for services

---

## Option 3 — Tiered Service Packages

### Model

Offer pre-defined onboarding packages:

| Package    | Scope                                |
| ---------- | ------------------------------------ |
| Basic      | Single feeder, limited devices       |
| Standard   | Multiple feeders, full adapter setup |
| Enterprise | Large-scale, custom integrations     |

### Pros

* Predictable pricing
* Easy procurement
* Clear expectations

### Cons

* Less flexibility for edge cases

---

## Recommended Approach

For initial Marketplace launch:

1. **Include a small amount of assisted onboarding** in the subscription
2. **Offer professional services separately** for advanced or custom needs
3. Clearly document:

   * What is included
   * What requires additional services

This approach:

* Aligns with Marketplace norms
* Sets realistic expectations
* Avoids overselling “plug-and-play”
* Protects engineering and support capacity

---

## Reviewer-Friendly Statement (You Can Reuse This)

> “While deployment is automated via Google Cloud Marketplace, initial configuration and commissioning are typically performed in collaboration with the vendor. Professional services are available to assist with circuit ingestion, OpenFMB adapter configuration, and system validation.”

---

## Summary

* Marketplace is the **entry point**, not the full implementation
* Vendor-assisted onboarding is **expected and normal**
* Professional services complement Marketplace subscriptions cleanly
* This model aligns with utility procurement, security, and operational practices
