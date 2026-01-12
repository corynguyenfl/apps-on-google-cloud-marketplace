# Supply Chain Security

## Overview

The solution follows a **defense-in-depth supply chain security model** designed to protect against compromised build systems, tampered artifacts, and vulnerable dependencies. Controls span **source code, build pipelines, container images, registries, and runtime deployment**, and align with cloud-native best practices for enterprise and critical-infrastructure software.

---

## Source Code & Build Integrity

* All source code is maintained in version-controlled repositories with controlled access.
* Changes are reviewed prior to merge using standard code review processes.
* Build pipelines are fully automated and run in isolated CI environments.
* Build credentials are short-lived and scoped to the minimum required permissions.

---

## CI/CD Pipeline Security

* CI/CD pipelines authenticate to cloud services using **OIDC-based workload identity** rather than long-lived static credentials.
* Pipelines are restricted to publishing artifacts only to vendor-owned registries.
* Production images are built from **pinned base images** to ensure reproducibility.
* Immutable version tags are used for all production images; mutable tags (e.g., `latest`) are not relied upon for deployments.

---

## Container Image Registry & Integrity

* All container images are stored in **Google Artifact Registry** under a vendor-controlled Google Cloud project.
* Artifact Registry access is enforced using **Google IAM**, ensuring that only authorized build systems can push images.
* Customer deployments pull images through Marketplace-managed permissions; images are not publicly accessible.

---

## Vulnerability Scanning & Remediation

* Container images stored in Artifact Registry are **automatically scanned for known vulnerabilities** upon push.
* Vulnerability findings include:

  * CVE identifiers
  * Severity ratings (Low, Medium, High, Critical)
  * Affected packages and versions
* High and critical vulnerabilities are tracked and remediated through:

  * Base image updates
  * Dependency upgrades
  * Image rebuilds and redeployments
* Zero known vulnerabilities are not guaranteed at all times; instead, the focus is on **timely remediation** and risk-based prioritization.

---

## Image Signing & Provenance

* The build process supports **cryptographic image signing** using industry-standard tooling.
* Image signatures are stored alongside images in Artifact Registry.
* Customers may optionally verify image signatures prior to deployment.
* Runtime enforcement of signed images can be enabled by customers using Kubernetes admission controls if required.

---

## Dependency Management

* OS-level and language-level dependencies are explicitly defined and version-controlled.
* Dependency updates are incorporated through regular maintenance releases.
* Third-party libraries are monitored for security advisories as part of ongoing maintenance.

---

## Deployment & Runtime Trust

* Customer deployments pull images directly from Artifact Registry using IAM-controlled access.
* Images are referenced by **immutable digests or versioned tags** in Helm charts.
* No images are downloaded from anonymous or public registries during production deployment.
* Customers retain full control over runtime policies, including network restrictions and admission controls.

---

## Auditability & Transparency

* Artifact Registry provides audit logs for:

  * image uploads
  * access attempts
  * vulnerability scan results
* CI/CD pipelines produce auditable build logs.
* The vendor maintains documentation describing:

  * build and release processes
  * vulnerability response procedures
  * image signing and verification options

---

## Shared Responsibility Model

* **Vendor responsibilities**

  * Secure build pipelines
  * Signed, scanned, and versioned images
  * Timely remediation of critical vulnerabilities
  * Clear documentation of supply chain controls

* **Customer responsibilities**

  * Runtime security policies
  * Cluster hardening
  * Optional enforcement of signed images
  * Network and access controls

---

## Summary

The solution’s supply chain security model:

* Uses **trusted, cloud-native tooling**
* Minimizes reliance on public registries
* Provides **visibility, auditability, and control** over artifacts
* Supports enterprise and utility security expectations without introducing unnecessary operational complexity

This approach aligns with Google Cloud Marketplace requirements and provides a strong foundation for future enhancements such as stricter enforcement policies or additional provenance attestations.
