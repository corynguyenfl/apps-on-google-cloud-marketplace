## 1️⃣ Establish the Scope & Assumptions (Day 0)

**What to do**

* Clearly define **what product** you are evaluating for Marketplace
* Decide what you are *not* evaluating (important for time-boxing)

**Decisions to record**

* Is this:

  * A **SaaS control plane** (vendor-hosted)?
  * A **customer-deployed application** (VM/GKE)?
  * Both (future)?

**Artifacts**

* 1–2 paragraph scope statement
* Explicit assumptions (e.g., “single-tenant SaaS”, “Kubernetes-first”)

---

## 2️⃣ Identify Applicable Google Cloud Marketplace Model(s)

**What to do**

* Map your product to Marketplace offer types:

  * SaaS (externally hosted)
  * GKE / Helm application
  * VM image
* Eliminate non-viable models early

**Key evaluation points**

* Where does the software actually run?
* Who owns the infrastructure?
* Who is responsible for upgrades?

**Artifacts**

* Comparison table:

  * Offer type
  * Pros / cons
  * Fit for your product
* **Recommended offer type**

✅ *This step drives everything else*

---

## 3️⃣ Architecture Gap Analysis

**What to do**

* Compare **current architecture** vs **Marketplace-required architecture**

**Evaluate**

* Deployment automation requirements

  * Terraform
  * Helm charts
  * GKE manifests
* Tenant isolation model
* Identity model (Google IAM vs internal auth)
* Networking (private VPC, ingress, DNS)

**Questions to answer**

* Can this be deployed **without manual steps**?
* Is onboarding reproducible and auditable?
* Can customers deploy without your involvement?

**Artifacts**

* Current vs Required architecture diagram (even rough)
* List of required changes / refactors

---

## 4️⃣ Billing & Licensing Investigation (Critical)

**What to do**

* Identify **how Google expects you to bill**
* Map that to your current pricing/licensing model

**Evaluate**

* Subscription vs usage-based billing
* Required integration points:

  * Marketplace Metering API
  * License verification flow
* Revenue share and payout mechanics

**Questions to answer**

* Do you need to emit usage metrics?
* Do you need license enforcement at runtime?
* Can your system tolerate delayed billing signals?

**Artifacts**

* Billing flow diagram
* List of required code changes
* Open questions / risks

⚠️ *Billing is often the biggest hidden cost*

---

## 5️⃣ Security, Compliance & Legal Readiness

**What to do**

* Identify **minimum security bar** to pass Marketplace review

**Evaluate**

* IAM usage (service accounts, least privilege)
* Secrets handling
* Network security posture
* Data residency & ownership
* Customer data isolation

**Compliance reality check**

* Are SOC2 / ISO / security docs required *now* or later?
* Is there a public security overview required?

**Artifacts**

* Security checklist with pass/fail
* List of gaps (docs, controls, audits)

---

## 6️⃣ Operational & Support Expectations

**What to do**

* Understand what Google expects once you are listed

**Evaluate**

* Upgrade strategy
* Backward compatibility
* Monitoring & alerting
* Customer support SLAs
* Incident response expectations

**Key question**

> “If 10 customers deploy this tomorrow, can we support it?”

**Artifacts**

* Operational readiness checklist
* Support responsibility matrix (Customer vs Vendor)

---

## 7️⃣ Marketplace Onboarding Process Mapping

**What to do**

* Walk through Marketplace onboarding end-to-end

**Evaluate**

* Vendor registration steps
* Review checkpoints
* Testing & validation requirements
* Time to publish (realistic timeline)

**Artifacts**

* Step-by-step onboarding flow
* Estimated timeline (best / worst case)

---

## 8️⃣ Effort, Risk & Cost Estimation

**What to do**

* Convert findings into **decision-grade inputs**

**Estimate**

* Engineering effort (weeks by role)
* Ongoing operational cost
* Compliance/documentation effort

**Identify**

* Hard blockers
* Soft risks
* Unknowns requiring follow-up spikes

**Artifacts**

* Effort estimate table
* Risk register

---

## 9️⃣ Final Recommendation & Next Steps

**What to do**

* Synthesize everything into a clear recommendation

**Recommendation format**

* ✅ Go / ⚠️ Conditional Go / ❌ No-Go
* Why
* What would need to change to revisit later

**Artifacts**

* 1–2 page summary
* High-level roadmap if Go

---

## ✅ How You Know the SPIKE Is “Done”

You should be able to answer, confidently:

* Can we offer this on Google Cloud Marketplace **today**?
* If not, **what exactly blocks us**?
* How much effort would it take?
* Is the ROI worth it *for this product*?
