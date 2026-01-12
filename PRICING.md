1. the **market reality** (what customers expect)
2. **pricing anchors** that reviewers and buyers won’t push back on
3. a **concrete starting price proposal** tailored to *your* software
4. how to **structure tiers** so you don’t underprice yourself

---

## 1️⃣ Market Reality (Customer-Hosted GKE Apps)

For **customer-hosted, mission-critical infrastructure software** (utilities, DERMS, control systems):

* Customers expect **annual subscriptions**
* Pricing is usually **per deployment / per environment**
* Buyers care more about:

  * predictability
  * procurement simplicity
  * auditability
    than “pay per message” precision

Typical buyers:

* utilities
* system integrators
* energy operators
* regulated enterprises

👉 These buyers are **not price-sensitive at the $1k–$10k/month level** if the software controls real assets.

---

## 2️⃣ Common Pricing Bands (What the Market Accepts)

Here are **realistic, commonly accepted ranges** for software comparable in *criticality* (not feature-for-feature):

### Monthly price bands (per deployment)

| Band                   | Typical Use                             |
| ---------------------- | --------------------------------------- |
| $500–$1,500 / month    | Lightweight monitoring tools            |
| $2,000–$5,000 / month  | Operational control platforms           |
| $6,000–$15,000 / month | DERMS, grid control, analytics          |
| $20,000+ / month       | Full enterprise platforms / fleet scale |

Your software **clearly does not belong in the lowest band**.

---

## 3️⃣ A Strong, Defensible Starting Price (Recommended)

### 🎯 **Recommended v1 Pricing (Subscription-Only)**

> **$3,000 – $8,000 per month per deployment**
> *(billed monthly or annually via Marketplace)*

Why this works:

* Anchors you as **operational software**, not a dashboard
* Fits utility procurement thresholds
* Avoids “too cheap to be trusted”
* Leaves room for upsell and negotiation

---

## 4️⃣ Suggested Tier Structure (Concrete Proposal)

This is a **very Marketplace-friendly** structure.

### **Tier 1 — Monitoring**

**$2,000 / month**

Includes:

* Telemetry ingestion (OpenFMB)
* One-line visualization
* Read-only dashboards
* Asset state & alarms

Limits:

* DER count cap (e.g., 50–100)
* No control actions

---

### **Tier 2 — Operations (Recommended Default)**

**$5,000 / month**

Includes:

* All Monitoring features
* Device control (breakers, switches)
* DER dispatch
* ESS management
* Operator UI

Limits:

* Higher DER count (e.g., 250–500)

This will likely be your **most common sale**.

---

### **Tier 3 — Advanced Analytics**

**$8,000–$12,000 / month**

Includes:

* All Operations features
* Asset health scoring
* Predictive maintenance
* ESS automated testing
* Advanced reporting

Limits:

* Highest DER cap
* Optional premium support

---

## 5️⃣ Why This Pricing Is Justifiable (Key Talking Points)

If someone asks “why does this cost $X?”:

* This software:

  * issues **real control commands**
  * manages **grid-connected assets**
  * affects **safety, reliability, and uptime**
* One prevented outage or avoided failure:

  * pays for **months or years** of subscription
* Pricing reflects **risk reduction**, not CPU usage

This framing resonates strongly with utility buyers.

---

## 6️⃣ Marketplace Reviewer Perspective (Important)

Reviewers **do not approve/reject based on price**, but they *do* look for:

* Clear tier definitions
* No hidden usage charges
* Enforceable limits
* Reasonable, non-predatory pricing

Your subscription-only tiers meet all of these.

---

## 7️⃣ What *Not* to Do (Early Mistakes)

❌ $99/month pricing → signals “toy”
❌ Usage-based pricing v1 → unnecessary complexity
❌ Per-message or per-event pricing → procurement friction
❌ Too many tiers → confusion

---

## 8️⃣ My Strong Recommendation

Start with:

* **3 tiers**
* **$2k / $5k / $8–12k per month**
* **Annual option with 10–15% discount**

You can *always*:

* add usage-based pricing later
* add enterprise tiers
* negotiate custom contracts outside Marketplace

But it’s **very hard** to raise prices once anchored low.
