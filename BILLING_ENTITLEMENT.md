## 1) The two “standard” integration paths

### A) Customer-hosted Kubernetes app (GKE / Helm)

**Common approach:** **Marketplace handles billing; your app enforces entitlements locally**.

* Customer buys a plan in Marketplace.
* Customer deploys the app into **their** GKE cluster.
* Your app checks entitlement (or plan info) and enables features accordingly.
* For subscription-only listings, there’s typically **no usage reporting** required.

This is aligned with how commercial Kubernetes apps are deployed and billed in Marketplace. ([Google Cloud Documentation][1])

**What you implement**

* A small **license/entitlement gate** inside your app (often in the API service):

  * “Is this installation entitled to run?”
  * “Which plan/tier?”
  * “What limits apply?”
* Feature gating / limits:

  * max DER count, max sites, advanced analytics enabled, etc.

**Why it’s common**

* Customers expect “buy in Marketplace → deploy via Marketplace”
* Vendors avoid complex metering at first
* Easier to review and support

---

### B) Externally hosted SaaS (vendor-hosted)

**Common approach:** **Procurement/Entitlement API + webhook/PubSub notifications to provision accounts**, optionally **usage reporting** if metered pricing.

Google’s documented model is:

* Integrate backend with **Cloud Commerce Partner Procurement API** to manage accounts + entitlements. ([Google Cloud Documentation][2])
* When a customer purchases, changes plan, or cancels, Google notifies you (commonly via Pub/Sub), and your backend provisions/updates the customer. ([Google Cloud][3])
* If you use usage-based pricing, report usage to Google (Service Control API). ([Google Cloud Documentation][4])

**Why it’s common**

* SaaS runs on your infra; Marketplace needs a way to control access and billing lifecycle

---

## 2) What “entitlement integration” usually looks like in practice

Across both models, vendors usually implement these components:

### 1) Entitlement source of truth

* “Customer X is entitled to Product Y at Plan Z”
* For SaaS: retrieved/managed via Procurement API. ([Google Cloud Documentation][5])
* For K8s apps: often enforced through the Marketplace deployment/billing linkage + your internal gating. ([Google Cloud Documentation][1])

### 2) Provisioning / activation

* SaaS: create tenant, admin user, workspace, etc., on purchase events. ([Google for Developers][6])
* K8s app: “activation” is typically the deployment itself; your app enables features based on plan.

### 3) Enforcement

* At startup and periodically:

  * verify entitlement is active
  * enforce plan limits
  * handle suspension/cancellation gracefully

### 4) Usage metering (only if you choose usage-based pricing)

* Aggregate usage (hourly/daily) and submit usage reports. ([Google Cloud Documentation][4])

---

## 3) What’s most common for a new listing (recommended pattern)

**Most vendors start with:**

* **Subscription billing** (tiered plans)
* **Entitlement checks + feature gating**
* **No usage metering** initially (unless the business model requires it)

This matches Google’s own guidance that usage reporting is required when you choose usage-based pricing, but subscription-only is simpler. ([Google Cloud Documentation][4])

---

## 4) What I’d recommend for your product specifically

Given your system:

* customer-hosted GKE is your best fit
* Keycloak-per-customer + NATS JWT already provides strong internal authorization
* you likely want simple procurement

### Recommended “common approach” for you (Phase 1)

1. **Offer subscription tiers** (Basic / Operations / Advanced Analytics)
2. Implement an **Entitlement Controller** in your API service that:

   * reads plan/tier config (from deployment values or an entitlement record)
   * enforces:

     * max DER count / adapters
     * enabling DER dispatch
     * enabling predictive maintenance features
3. Provide an **Admin UI page** showing:

   * current plan
   * limits
   * usage counters (even if not reported to Google yet)

### Phase 2 (optional)

If you later want “pay by DER-hours” or “pay by metered messages/telemetry,” then add usage reporting.

---

## 5) Common pitfalls reviewers see (and how to avoid them)

* **No clear enforcement point** → reviewers want to see *how the plan gates features*
* **No cancellation behavior** → define what happens when entitlement ends
* **Overly granular metering early** → increases complexity and failure modes
* **Secrets/keys in images** → use GKE Workload Identity for GCP services (even if Keycloak is your IdP)

---

## Bottom line

* **Kubernetes app**: common approach is **subscription tiers + internal entitlement gating**, minimal external API integration to start. ([Google Cloud Documentation][1])
* **Externally hosted SaaS**: common approach is **Procurement API + entitlement lifecycle + optional usage reporting**. ([Google Cloud Documentation][2])


[1]: https://docs.cloud.google.com/marketplace/docs/kubernetes-apps?utm_source=chatgpt.com "Deploying Kubernetes apps | Google Cloud Marketplace"
[2]: https://docs.cloud.google.com/marketplace/docs/partners/commerce-procurement-api/reference?utm_source=chatgpt.com "Overview | Google Cloud Marketplace Partners"
[3]: https://cloud.google.com/blog/topics/partners/google-cloud-ai-agent-marketplace?utm_source=chatgpt.com "Google Cloud AI Agent Marketplace"
[4]: https://docs.cloud.google.com/marketplace/docs/partners/integrated-saas/configure-usage-reports?utm_source=chatgpt.com "Configure usage reporting to Google"
[5]: https://docs.cloud.google.com/marketplace/docs/partners/integrated-saas/manage-entitlements?utm_source=chatgpt.com "Manage customer entitlements for your SaaS product"
[6]: https://developers.google.com/codelabs/gcp-marketplace-saas?utm_source=chatgpt.com "Integrate your SaaS solution with the Google Cloud ..."


...

That’s a **very good choice**, and it aligns with what **most successful first-time Google Cloud Marketplace listings do**.

Below is the **standard, reviewer-approved approach for subscription-only billing & entitlement**, written specifically for a **customer-hosted Kubernetes (GKE) application** like yours.

---

# Common Approach for Subscription-Only Billing & Entitlement

*(Customer-Hosted Kubernetes App on Google Cloud Marketplace)*

## 1. High-Level Pattern (What Google Expects)

For **subscription-only** Marketplace offers:

> **Google Cloud Marketplace handles billing and procurement.
> Your software enforces entitlements and feature limits locally.**

There is **no per-event usage reporting** and **no real-time billing API calls required**.

This is the **lowest-friction, lowest-risk** integration path and is widely accepted by Marketplace reviewers.

---

## 2. What “Entitlement” Means in Practice (Subscription Model)

An **entitlement** answers three questions:

1. **Is this deployment licensed?**
2. **Which plan/tier is active?**
3. **What limits/features apply?**

For subscription-only:

* Entitlement is **binary + tiered**
* Enforcement happens **inside your app**
* Billing lifecycle (start, change, cancel) is handled by Marketplace

---

## 3. Reference Architecture (Subscription-Only)

```
Google Cloud Marketplace
    |
    |  (Subscription purchase)
    v
Customer GCP Project
    |
    |  (Marketplace deploy)
    v
Customer GKE Cluster
    |
    +-- Entitlement Controller (your app)
           |
           +-- Reads plan/tier
           +-- Enforces limits
           +-- Enables/disables features
```

No outbound billing calls required at runtime.

---

## 4. What You Need to Implement (Minimal & Sufficient)

### A. Define Subscription Tiers

Keep tiers **coarse and understandable**.

Example (illustrative):

| Tier       | Enabled Features                                          |
| ---------- | --------------------------------------------------------- |
| Basic      | Monitoring, telemetry, one-line UI                        |
| Operations | Device control, DER dispatch                              |
| Advanced   | Asset health analytics, prediction, ESS automated testing |

Limits can include:

* Max DER / ESS count
* Max adapters
* Feature flags (on/off)

---

### B. Implement an **Entitlement Controller**

This is typically:

* A small module in your **API backend**
* Or a dedicated internal service

**Responsibilities**

* Determine active plan/tier
* Enforce limits and feature gates
* Expose entitlement state to UI

**Common enforcement points**

* API endpoints (block control calls if not entitled)
* Background services (disable analytics jobs)
* UI feature visibility

---

### C. Where Entitlement State Comes From

For subscription-only Kubernetes apps, common options:

#### Option 1 (Most Common, Reviewer-Friendly)

* Plan/tier is passed in at deployment time (Helm values)
* Marketplace deployment creates a plan-specific release

Pros:

* Simple
* Deterministic
* Easy to audit

Cons:

* Plan change requires redeploy/upgrade

#### Option 2 (More Dynamic, Optional Later)

* App reads entitlement metadata from Marketplace-provided context
* Or from a small entitlement config object updated on upgrade

Pros:

* Supports upgrades without redeploy
  Cons:
* Slightly more logic

**Recommendation for v1:**
👉 **Option 1** (Helm-based tier selection)

---

## 5. How Reviewers Expect Enforcement to Work

Marketplace reviewers look for **clear answers** to these questions:

### “What happens if the subscription expires or is canceled?”

**Expected behavior**

* Non-destructive
* Read-only mode or feature disable
* No data loss
* Clear admin notification

### “What prevents customers from bypassing limits?”

**Expected behavior**

* Server-side enforcement
* Not UI-only
* Limits enforced in APIs and background services

### “Is billing tied to usage?”

**Expected answer**

* No (subscription-only)
* Usage metrics may be shown to the customer, but not billed

---

## 6. How This Fits Your Existing Architecture

Your stack already has all the right pieces:

| Component   | Role in Entitlement                        |
| ----------- | ------------------------------------------ |
| Keycloak    | User/service authentication                |
| NATS        | Messaging authorization (already enforced) |
| API backend | **Primary entitlement enforcement point**  |
| UI          | Displays plan + limits                     |
| Helm charts | Plan/tier selection                        |

You **do not** need to:

* Change Keycloak
* Change NATS auth
* Add metering pipelines
* Integrate real-time billing APIs

---

## 7. What You Explicitly Do *Not* Need (Subscription-Only)

❌ Usage reporting APIs
❌ Per-event metering
❌ Marketplace Procurement API integration
❌ Billing webhooks
❌ Cloud Service Control integration

These are only required for **usage-based** or **externally hosted SaaS** offers.

---

## 8. Minimal Documentation Reviewers Expect

You should document:

1. **Plan definitions**
2. **What each plan enables**
3. **How enforcement works**
4. **What happens on cancellation**
5. **How customers can upgrade/downgrade**

Short, clear, and operational — not marketing language.

---

## 9. Recommended “v1” Implementation Checklist

### Engineering

* [ ] Define 2–4 subscription tiers
* [ ] Add Entitlement Controller in API
* [ ] Enforce feature gates server-side
* [ ] Add plan info to UI (read-only)
* [ ] Ensure safe behavior on entitlement loss

### Packaging

* [ ] Helm values for tier selection
* [ ] Separate Marketplace listings or parameters per tier

### Documentation

* [ ] Billing & entitlement section
* [ ] Upgrade/downgrade guidance

---

## 10. Bottom Line

For your product and architecture:

> **Subscription-only billing with internal entitlement enforcement is the correct, standard, and reviewer-approved approach for Google Cloud Marketplace v1.**

It:

* Minimizes risk
* Minimizes integration effort
* Fits utility/enterprise buying behavior
* Leaves room for usage-based pricing later
