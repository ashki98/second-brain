# Trail: WellnessLiving Work

**Objective:** Document the full architecture of your WL work — Give, Ripples, the WL platform — so you can onboard teammates, prepare for reviews, and reconnect quickly after time away.
**Prerequisites:** Trail 04 — Backend Patterns, Trail 11 — AI & Modern Stack (for Ripples)
**Review time:** ~30 min
**Nodes:** 10

---

1. [[Give Intro]]
   > Start here: what Give is. A charitable giving and Donor Advised Fund (DAF) management platform under TIFIN/WL. This note establishes the business context — what a DAF is, who the users are, and where Give sits in the WL ecosystem. Everything else in this trail is a layer of detail on top of this.

2. [[Give Overview]]
   > The full architecture. Charity Service (FastAPI + React Admin, Elasticsearch for charity discovery). Louise Backend (Django + GraphQL, multi-tenant, funds/portfolios/giving/bank/CRM). Louise Frontend (React, Redux, Okta, Apollo). Elevate (Python microservices, DORM, django-tenants). Integrations: Okta, Plaid, GuideStar, DocuSign, AWS S3. This is your reference when you need to re-orient in the codebase.

3. [[Why graphql]]
   > Louise uses GraphQL rather than REST. This note explains the reasoning: flexible queries, reduced over-fetching, typed schema. With the Give architecture in mind, you can evaluate this trade-off concretely — Louise's multi-tenant, multi-module structure benefits specifically from GraphQL's flexibility.

4. [[Give Optimisations]]
   > What you've actually improved: performance work on the Give platform. This note connects the optimization concepts from Trail 06 to your specific codebase. Bridge between theory and the code you've shipped.

5. [[GIVE SAML]]
   > Enterprise SSO for Give: SAML 2.0 integration so corporate clients can log in with their identity provider. This note explains the SAML flow (IdP, SP, assertions, the redirect dance) in the Give context. Cross-reference with JWT (Trail 10) for the difference between SAML and token-based auth.

6. [[GIVE Account Statement Generation]]
   > The account statement feature: generating financial reports for DAF holders. This note covers the data model and generation pipeline — a self-contained feature that touches DB queries, PDF generation, and async job processing.

7. [[WL Wrapper (PHP)]]
   > The WellnessLiving platform's PHP API wrapper. This note covers how Give integrates with the parent WL platform via this wrapper — the authentication, the endpoints, the quirks. Useful when debugging anything that crosses the Give/WL boundary.

8. [[Ripples Infra and Setup]]
   > WL's AI-powered product: Ripples. This note covers the infrastructure setup — how Ripples is deployed, what services it depends on, and how it integrates with the WL ecosystem. The LangGraph agent architecture (Trail 11, note 5) runs here.

9. [[Privileged Keys]]
   > Key and secret management across WL services. This note covers what privileged keys are, how they're stored and rotated, and the access patterns. Boring but critical: this is what you check when a service mysteriously stops working.

10. [[Langgraph Flow]]
    > The AI brain of Ripples: LangGraph-based agent system. This note is the same as Trail 11, node 5 — it appears here because Ripples is where LangGraph is deployed. Read it in Trail 11 for theory; re-read it here for production context.
