# SCALABLITY

## Short, Correct Answer

> **GKE provides built-in mechanisms for horizontal and vertical scalability, but applications must be designed and configured to leverage them.**

You can rely on the *platform*, **not automatic scaling of your app** unless you explicitly enable it.

---

## What GKE Gives You Out of the Box

### 1) Horizontal Scalability (Yes, built-in)

GKE supports **Horizontal Pod Autoscaling (HPA)**:

* Scales pod replicas based on:

  * CPU / memory
  * custom metrics (via Prometheus / Cloud Monitoring)
* Works for:

  * stateless services (API, UI)
* Native, production-grade, well-understood

But:

* You must **define HPA objects**
* Your pods must:

  * expose metrics
  * have resource requests set

> HPA exists, but it is **opt-in**.

---

### 2) Vertical Scalability (Yes, built-in)

GKE supports **Vertical Pod Autoscaler (VPA)**:

* Automatically recommends or applies:

  * CPU/memory request/limit changes
* Best for:

  * stateful or singleton workloads
  * NATS, Keycloak, controllers

Caveat:

* VPA often requires **pod restarts**
* Typically used in:

  * “recommendation” mode
  * controlled environments

---

### 3) Node Scaling (Also built-in)

GKE provides **Cluster Autoscaler**:

* Automatically adds/removes nodes
* Works with:

  * HPA/VPA
  * multiple node pools
* Essential for true elasticity

Again:

* Must be enabled and configured
* Requires sane resource requests

---

## What GKE Does *Not* Automatically Do

❌ Decide which of your services should scale
❌ Know whether scaling is safe for your app
❌ Scale stateful services safely by default
❌ Tune limits and requests for you

That logic belongs to **your application design**.

---

## What Reviewers Expect You to Say (Exact Tone)

You should **not** claim:

> “The platform scales automatically.”

Instead, say:

> “The solution is deployed on GKE and leverages Kubernetes-native autoscaling mechanisms, including Horizontal Pod Autoscaling and Cluster Autoscaler, for stateless services. Vertical scaling is supported for selected components where appropriate.”

That phrasing shows competence and avoids overpromising.

---

## How This Applies to *Your* Architecture

### Safe to scale horizontally

* API services
* UI backend
* OpenFMB adapters (if stateless)
* Analytics workers (if idempotent)

### Scale carefully / vertically

* NATS
* Keycloak
* Datastores
* Control-plane components

### Typical pattern

* Separate node pools:

  * stateless (HPA-enabled)
  * stateful (fixed size or VPA)

---

## Marketplace-Ready Assumptions (Acceptable)

You may assume:

* Customers can enable HPA/VPA
* GKE cluster autoscaler is available
* Customers can scale node pools

You must document:

* Which components are safe to scale horizontally
* Which require care
* Recommended defaults

---

## Bottom Line (What to Put in the Report)

> **Yes, GKE provides built-in horizontal and vertical scalability primitives.
> The application is designed to leverage these mechanisms where appropriate, with clear guidance on which components support horizontal scaling and which require controlled scaling.**
