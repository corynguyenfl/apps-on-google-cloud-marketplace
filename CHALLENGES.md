# CHALLENGES

## The Normal Marketplace Engagement Model (Reality)

### What Google Cloud Marketplace *is*

**Google Cloud Marketplace** is:

* a **procurement channel**
* a **deployment accelerator**
* a **billing conduit**

It is **not**:

* a system integrator
* a configuration service
* a field engineering team

---

## Who Does What (Clear Responsibility Split)

### Google Cloud / Marketplace

* Handles:

  * subscription purchase
  * billing
  * entitlement
  * deployment automation (Helm/Terraform)
* Ensures:

  * images can be pulled
  * deployment succeeds
  * security expectations are met

### **We (the vendor)**

* Handle:

  * OpenFMB circuit ingestion
  * CIM/OpenDSS/OpenFMB model mapping
  * Adapter configuration (DNP3/MODBUS)
  * NATS subject topology
  * Device onboarding
  * Validation & commissioning

This is **expected and normal**, especially for:

* grid control systems
* DERMS
* utility-facing software
* anything safety- or operations-critical

---

## How Customers Typically Engage After Install

The most common flow looks like this:

1. **Customer buys & installs** from Marketplace
   (often by a cloud or IT team)

2. **Customer contacts vendor**

   * “We’ve deployed the software; we’re ready to configure circuits and devices.”

3. **Vendor-led onboarding**

   * Remote sessions
   * Config templates
   * Guided ingestion
   * Validation steps

4. **Optional handoff**

   * Customer takes over day-to-day operations
   * Vendor provides support as needed

This is **completely standard**.

---

## How This Is Usually Documented (Important)

Marketplace reviewers want clarity, not “zero-touch magic”.

We should explicitly document:

> “Initial configuration and commissioning of circuits, OpenFMB models, and field adapters is performed in collaboration with the vendor.”

This reassures reviewers and customers that:

* onboarding complexity is acknowledged
* support exists
* expectations are clear

---

## Common Patterns Vendors Use

### Pattern A — Assisted Onboarding (Most Common)

* Marketplace install is self-service
* Configuration is vendor-assisted
* Often included in first-year subscription

### Pattern B — Professional Services Add-On

* Marketplace handles software subscription
* Configuration sold separately (PO / SOW)
* Very common in utilities

### Pattern C — Hybrid

* Basic example configs included
* Advanced/custom circuits require vendor help

For OpenFMB + adapters, **Pattern A or B is the norm**.

---

## What Customers *Do Not* Expect

Customers do **not** expect:

* Google to configure breakers, DERs, or circuits
* Marketplace install to auto-discover their grid
* Zero-touch OpenFMB ingestion for complex feeders

Trying to pretend otherwise usually causes:

* failed pilots
* unhappy customers
* support overload

---

## How to Phrase This Cleanly in Our Report

We can safely include language like:

> “While deployment is automated via Google Cloud Marketplace, initial circuit ingestion and OpenFMB adapter configuration require domain-specific knowledge and are typically performed in collaboration with the vendor. This assisted onboarding model is standard for grid control and DER management platforms.”

That wording is accurate, defensible, and reviewer-safe.

---

## How This Impacts Our Marketplace Strategy (Good News)

This actually **strengthens** our case:

* We’re not overselling “plug-and-play”
* We’re setting realistic expectations
* We reduce support risk
* We align with utility procurement norms

Marketplace is a **front door**, not the entire house.

---

## Final Answer (Plain English)

* ✅ Customers **can and should** reach out to us for configuration
* ✅ This is **normal and expected**
* ❌ Customers are **not limited to dealing only with GCP**
* ❌ Google does **not** provide OpenFMB or adapter configuration

