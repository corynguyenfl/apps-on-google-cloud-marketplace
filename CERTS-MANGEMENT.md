# CERTS MANGEMENT

## The Normal Responsibility Model (Marketplace-Expected)

### Summary Table

| Component                   | TLS Termination         | Certificate Owner                                      |
| --------------------------- | ----------------------- | ------------------------------------------------------ |
| UI (Ingress)                | Ingress / Load Balancer | **Customer**                                           |
| API (Ingress)               | Ingress / Load Balancer | **Customer**                                           |
| Keycloak (Ingress)          | Ingress / Load Balancer | **Customer**                                           |
| Internal service-to-service | In-cluster (optional)   | Vendor-provided defaults                               |
| NATS client connections     | NATS server             | Vendor provides mechanism, **customer provides certs** |

This split is **normal, defensible, and reviewer-safe**.

---

## 1️⃣ UI, API, Keycloak (External Access)

### How TLS is normally handled

* TLS terminates at:

  * GKE Ingress (NGINX, GKE Ingress, or Gateway API)
  * or a cloud load balancer
* Certificates are:

  * Customer-managed
  * Issued by:

    * Google-managed certs
    * Let’s Encrypt
    * Corporate CA

### Vendor responsibility

* Provide:

  * Ingress manifests
  * Clear documentation
  * Example annotations for managed certs
* Support standard Kubernetes TLS secrets

### Customer responsibility

* Own:

  * DNS
  * Certificate issuance
  * Renewal policies

📌 **Why this is correct**
Customers control DNS and trust roots. Vendors should not manage external certs in customer environments.

---

## 2️⃣ Internal Service-to-Service TLS (Optional but Recommended)

### Typical approaches

* Option A: mTLS inside the cluster (advanced)
* Option B: cluster-internal plaintext + perimeter TLS (common for v1)

### Vendor responsibility

* Decide default posture
* Provide optional mTLS support
* Generate internal CA if enabled

### Customer responsibility

* Choose whether to enable mTLS
* Approve trust model

📌 For v1 Marketplace:

> Plaintext inside the cluster + TLS at ingress is acceptable and common.

---

## 3️⃣ NATS TLS (Client ↔ Server)

### This is the one people get wrong — here’s the correct model

### Vendor provides

* NATS configuration that supports TLS
* Helm values for:

  * CA cert
  * server cert
  * client certs (if mTLS)
* Documentation for setup

### Customer provides

* Certificates (or approves generated ones)
* Trust root (especially in regulated environments)

### Common patterns

* Customer supplies:

  * TLS certs via Kubernetes Secrets
* Or vendor generates:

  * a self-signed CA for internal-only NATS traffic (acceptable for v1)

📌 **Key point:**
The vendor must *support* TLS, but should not *own* long-lived certs in customer infrastructure.

---

## 4️⃣ What Marketplace Reviewers Expect Us to Say

We should explicitly document something like this:

> “TLS is used to secure all external access to the UI, API, and Keycloak via Kubernetes Ingress. Certificate issuance and lifecycle management are customer-controlled, consistent with enterprise DNS and trust requirements.
>
> Internal communication can optionally be secured using TLS. NATS supports TLS for client connections, with certificates supplied by the customer or generated during deployment.”

That statement:

* clarifies responsibility
* avoids promising managed certs
* aligns with cloud-native norms

---

## 5️⃣ What We Should *Not* Do

❌ Vendor-managed public certificates in customer clusters
❌ Hard-coded certs in images
❌ Automatic Let’s Encrypt without customer opt-in
❌ Hiding cert lifecycle from the customer

Those all raise red flags in enterprise reviews.

---

## 6️⃣ Recommended Default for Our Marketplace v1

**Safe, low-friction defaults:**

* UI / API / Keycloak:

  * TLS via Ingress
  * Customer-managed certs
* NATS:

  * TLS supported
  * Self-signed CA by default (internal)
  * Customer can replace with their own
* Internal services:

  * Plaintext acceptable initially
  * Optional mTLS documented

This balances:

* security
* simplicity
* customer control

---

## 7️⃣ How to Phrase This in Documentation (Reusable)

> “The solution supports TLS for all external endpoints and messaging interfaces. Customers retain ownership of certificate issuance, trust roots, and renewal processes. The vendor provides configuration templates and deployment guidance to integrate customer-managed certificates.”

---

## Bottom Line

* **Customers own certificates**
* **Vendors support TLS and provide hooks**
* This is **expected**, **normal**, and **reviewer-approved**
* Avoid long-term cert ops risk while remaining secure
