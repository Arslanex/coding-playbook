# 15 · Security and access

WHEN: any PR, new route, form, HTML, env, or "the frontend will check."
LOAD: this file **before** merge. Auth placement: 12. Design of focus/labels: 01 and this file's access section.
RELATED: 04 (`NEXT_PUBLIC_`) · 09 (error bodies) — open only if the task is also that topic.
SCOPE: browser and Next headers. Skipping a layer because "the API checks it" is still a defect **for XSS and secrets**. Skipping authz on the API because "the button is hidden" is a defect on the **backend**.

A hidden button is not a control. Curl is.

---

## Layers (do not skip)

1. **Secrets** — none in `NEXT_PUBLIC_`, none in client bundles, none in repo (04).
2. **Transport** — HTTPS in production. Cookie Secure + HttpOnly + SameSite (12).
3. **CSRF** — SameSite on the session cookie, or a CSRF token the API already requires. MUST NOT: a cookie a script can read and replay blindly if SameSite is None without CSRF.
4. **XSS** — React text interpolation by default. MUST NOT: `dangerouslySetInnerHTML` unless the HTML is sanitised and the product must render it. MUST NOT: `eval`.
5. **Open redirect** — login `?next=` only same-origin paths.
6. **Click / UI** — disable is not authz. The API still 401/404 (12).
7. **Headers** — `next.config` security headers (CSP as soon as the app is real; `X-Content-Type-Options`, frame deny unless a product embed exists).
8. **Errors** — no stack on the glass (09, 01).

MUST NOT: `target="_blank"` without `rel="noopener noreferrer"`.
MUST NOT: put PII in the URL that the product can keep in the body.

---

## Access (a11y)

MUST: visible label (not placeholder-only). MUST: visible focus (01). MUST: keyboard reaches every action.
MUST: `alt` on meaningful images; decorative `alt=""`.
MUST: status not colour-only (01).
MUST: `lang` on `<html>`.
MUST NOT: `outline: none` without a replacement ring.
MUST NOT: icon-only control without an accessible name.

---

## Agent pass (PR)

- [ ] No secret in `NEXT_PUBLIC_` or the bundle
- [ ] No `dangerouslySetInnerHTML` without a sanitiser
- [ ] `next` redirect is same-origin
- [ ] Cookie still HttpOnly; no JWT in storage
- [ ] Protected UI still fails closed if the API 401s
- [ ] New control has a name and a focus ring
- [ ] Test for 401 / error banner this route introduced (14)

---

## Done

- [ ] Layers above still present
- [ ] Access section true for new controls
- [ ] Agent pass is all yes
