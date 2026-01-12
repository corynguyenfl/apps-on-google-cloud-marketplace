Write a comprehensive evaluation report for offering a containerized software product on Google Cloud Marketplace. The report should be structured and detailed, but do not invent any technical, business, or product details—use only the information provided in the prompt or leave sections as placeholders for user input.

The report must include the following sections, each with clear headings and bullet points or tables where appropriate:

1. Executive Summary

   - Briefly summarize the purpose, scope, and main recommendation of the report.
2. Summary

   - State the feasibility, requirements, and recommended approach for offering the software on Google Cloud Marketplace.
   - Clearly indicate the preferred deployment model and rationale.

3. Product Overview (Current State)

   - List core capabilities and features.
   - Describe key architectural characteristics.
  
4. Marketplace Offer Model Evaluation

   - Present a table comparing different offer models (e.g., SaaS, VM-based, Kubernetes application) with fit and assessment.
   - Clearly state the recommended model and rationale.
  
5. Target Marketplace Architecture

   - Define deployment boundaries and core components.
   - List which components are deployed in the customer’s environment.

6. Identity, Authentication, and Authorization

   - Describe the current model and any integrations (e.g., Keycloak, NATS, Google IAM).
   - Explain the authorization model and role mapping.

7. Security & Key Management Considerations

   - Explain key management, rotation strategies, and required controls.
   - Include a table for certificate management responsibilities.
  
8. Deployment & Packaging Requirements

   - List required assets (e.g., Docker images, Helm charts).
   - Specify Kubernetes primitives used.
  
9. Billing & Licensing Model

   - Describe the initial billing model and rationale.
   - Explain licensing enforcement mechanisms.
  
10. Operational & Support Considerations

    - Define customer and vendor responsibilities.
    - Outline operational readiness and support models.
  
11. Onboarding and Configuration

    - Describe the deployment, configuration, and commissioning process.
    - List customer and vendor responsibilities during onboarding.
    - Explain edge node provisioning and onboarding flow.
  
12. Scalability

    - Discuss scalability mechanisms and what is not handled automatically.
  
13. Effort Estimate

    - Provide a table estimating engineering effort by area.
  
14. Risks & Mitigations

    - Present a table of key risks, impacts, and mitigations.
  
15. Supply Chain Security

    - Detail supply chain security controls, including source code, CI/CD, registry, vulnerability scanning, image signing, dependency management, deployment trust, and auditability.
  
16. Current State vs Marketplace-Ready State

    - Provide a gap analysis table comparing current state, requirements, and gaps.
    - For each gap, list what needs to be done, estimated effort, and deliverables.


**Instructions:**

- Do not invent or assume any technical, business, or product details not provided in the prompt.
- Use placeholders (e.g., [Insert product name], [Insert feature list]) where information is missing.
- Use clear, professional language and organize the report for readability.
- Use bullet points, tables, and section dividers as in the example.