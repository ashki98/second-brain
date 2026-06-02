# Trail: Security

**Objective:** Understand the most common web attack vectors and the defenses against them — from injection to auth flaws to real-world exploits.
**Prerequisites:** Trail 03 — Networking, Trail 04 — Backend Patterns (for JWT)
**Review time:** ~20 min
**Nodes:** 4

---

1. [[XSS Attacks]]
   > Start with the most common client-side attack: injecting malicious scripts through unsanitized user input. This note covers stored XSS (persisted on server), reflected XSS (in URL), and DOM-based XSS (client-side manipulation), with real-world examples (Samy Worm, PayPal 2019). Defenses: input sanitization, output escaping, CSP headers, DOMPurify.

2. [[CORS]]
   > The browser's same-origin policy is the first line of defence — CORS is the controlled relaxation of that policy. This note explains how preflight requests work, which headers control access, and where CORS misconfigurations create vulnerabilities. Read after XSS: both are browser-enforced security mechanisms.

3. [[JWT]]
   > Token-based auth from a security perspective. JWTs are only as secure as their implementation: algorithm confusion attacks (RS256 downgraded to none), long expiry, weak secrets, sensitive data in payload. This note covers the attack surface and how to use JWTs safely. Cross-reference with Trail 04 (Backend Patterns) where JWTs are introduced architecturally.

4. [[MongoBleed]]
   > A real-world vulnerability case study: what happens when DB authentication is bypassed. This note grounds the abstract security principles from notes 1–3 in an actual exploit. Case studies are the best way to retain security knowledge — abstract rules become vivid when you've seen what they prevent.
