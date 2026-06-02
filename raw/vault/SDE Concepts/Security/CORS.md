# CORS

YT: 

[https://www.youtube.com/watch?v=E6jgEtj-UjI](https://www.youtube.com/watch?v=E6jgEtj-UjI)

CORS is a browser security mechanism that builds on the Same-Origin Policy to safely allow controlled cross-origin HTTP requests using specific headers on both simple and preflighted requests, and it must be configured correctly on the backend (or via proxies) rather than “fixed” purely from the frontend.

---

## Same-Origin Policy and why CORS exists

- Browsers enforce the Same-Origin Policy (SOP): a page can only read responses from the same origin, where origin = scheme + host + port (for example, https://app.comhttps://app.comhttps://app.com can freely call https://app.com/apihttps://app.com/apihttps://app.com/api, but not https://api.other.comhttps://api.other.comhttps://api.other.com).
- SOP exists to prevent attacks like a malicious tab silently calling your bank’s API with your cookies and reading the response, which would leak sensitive data without you knowing.
- Because many modern apps legitimately need cross-origin APIs (frontend on one domain, API on another), CORS (Cross-Origin Resource Sharing) was introduced as a controlled relaxation of SOP, letting servers explicitly say which origins can access them.

---

## What is a CORS request?

- Any request from one origin (frontend) to another origin (backend) that the browser enforces SOP on becomes a “CORS request”; this is still just normal HTTP, but with extra rules and headers checked by the browser.
- The browser automatically attaches an `Origin` header on cross-origin requests (for example, `Origin: https://myapp.com`), and the server decides whether that origin is allowed.
- If the response does not contain an appropriate `Access-Control-Allow-Origin` header that matches the requesting origin (or  when allowed), the browser blocks access to the response and shows a CORS error in the console.

---

## Core CORS response headers

- `Access-Control-Allow-Origin`: specifies which origin is allowed (exact origin like `https://myapp.com` or ); without this, the browser will block cross-origin access.
- Other key headers:
    - `Access-Control-Allow-Methods`: which HTTP methods are allowed for cross-origin requests (for example, `GET, POST, PUT`).
    - `Access-Control-Allow-Headers`: which custom request headers (for example, `Authorization`, `X-Api-Key`) are allowed.
    - `Access-Control-Allow-Credentials`: whether cookies/credentials are allowed (`true` or omitted); must not be used with  for `Access-Control-Allow-Origin` in secure setups.

---

## Simple vs preflight requests (your main conceptual doubt)

- A CORS request is treated as a **simple request** when all of these conditions hold:
    - HTTP method is `GET`, `HEAD`, or `POST`.
    - No non-standard/custom headers (for example, no `Authorization`, `X-*` headers).
    - `Content-Type` is one of a small safe list such as `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain` (not `application/json`).
- For **simple requests**, the browser sends the request directly with `Origin`; if the response includes a valid `Access-Control-Allow-Origin`, the browser lets your JS read it, otherwise it blocks with a CORS error.
- A request becomes a **preflighted request** when it is “potentially unsafe”, for example:
    - Uses methods like `PUT`, `DELETE`, `PATCH`, or other non-simple methods.
    - Uses custom headers like `Authorization`, `X-Api-Key`, or non-simple `Content-Type` such as `application/json`.
- In that case the browser first sends an automatic **preflight** request:
    - Method: `OPTIONS` to the same URL.
    - Includes `Origin`, `Access-Control-Request-Method`, and `Access-Control-Request-Headers` (which say what the *real* request intends to use).
    - The server must respond with `Access-Control-Allow-Origin` and matching `Access-Control-Allow-Methods` / `Access-Control-Allow-Headers`; only then does the browser send the actual request.

*(This simple vs preflight distinction is usually where confusion comes in: your doubt is mainly around why some requests work without an OPTIONS call while others trigger preflight—this is entirely determined by method + headers + content type and whether they stay within the “simple” constraints.)*

---

## Fixing CORS on the backend

- The recommended way to resolve CORS issues is by configuring the API server to send correct CORS headers, not by trying to “fix” it from the frontend alone.
- In Node.js (for example, Express with a CORS middleware), you can:
    - Enable CORS for all origins (useful for quick dev setups).
    - Or restrict to a whitelist of specific allowed origins and methods, and explicitly configure allowed headers and credentials.
- In Spring Boot (Java), CORS is typically configured via a global Web configuration class:
    - Map CORS rules to paths (for example, `/api/**`) and set allowed origins, methods, and headers.
    - Optionally enable credentials if cookies/auth are needed and ensure that origins are explicit, not , when credentials are allowed.

---

## Spring Security and CORS

- When Spring Security is enabled, CORS must also be wired into the security configuration; otherwise, security filters may block the request before CORS is evaluated correctly.
- The configuration typically enables CORS support in the security filter chain **before** authentication/authorization rules so that preflight and cross-origin checks succeed while still enforcing proper auth for protected endpoints.

---

## When you do not control the backend

- If you cannot change the backend (third‑party API or locked internal service), you can place a **proxy** between your frontend and the target API.
- The browser talks to your proxy on the same origin (no CORS), and the proxy communicates with the external API server‑side, then returns the response; to the browser, it looks like a same-origin call.
- For purely local development and debugging (not production), browser extensions like “CORS Everywhere” can temporarily disable or bypass CORS checks, letting you test APIs without configuring a backend or proxy, but this is only a dev convenience.

---

## CORS vs CSRF (important conceptual distinction)

- CORS is about **which origins are allowed to read responses from your API**; it is a browser-enforced access control policy for cross-origin reads.
- CSRF (Cross-Site Request Forgery) is a separate attack where a victim is tricked into sending an unintended request to a site where they are already authenticated (for example, a hidden transfer form auto-submitted to a bank).
- Proper CSRF defenses (CSRF tokens, same-site cookies, etc.) are still needed even when CORS is correctly configured, because CORS alone does not prevent CSRF attacks.

---

## Quick mental checklist for your future refresh

- Check if the frontend and backend **origins** are different; if yes, the browser will treat it as a CORS request and add `Origin`.
- Ask: is this a **simple** request (GET/POST/HEAD, no custom headers, safe content type) or will it trigger a **preflight** (PUT/DELETE/PATCH, auth headers, JSON, etc.)?
- For failing requests, confirm that the backend (or proxy) is sending:
    - `Access-Control-Allow-Origin` with the correct origin (or  for non-credentialed public APIs).
    - Matching `Access-Control-Allow-Methods` / `Access-Control-Allow-Headers` (especially for preflight).
- Remember: use proper backend/server config or a controlled proxy in production; use browser extensions only as a temporary local‑dev workaround; and treat CORS and CSRF as **complementary but different** concerns.
