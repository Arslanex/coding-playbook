# Extra 04 · SSO (browser)

WHEN: users start login at an IdP (Google, Okta, Entra, SAML) from this UI.
LOAD: this file **and** 08, 12, 15. Backend: [python-fastapi-backend/extra/07-sso.md](../../python-fastapi-backend/extra/07-sso.md).
SCOPE: buttons and redirects. This app does not verify the IdP token.

---

## Flow

`features/auth`: "Continue with {provider}" → `GET` the FastAPI start URL (09). Callback hits **FastAPI**, which Set-Cookies **our** session (12). Next never holds the IdP access token.

MUST NOT: implicit flow in the browser. MUST NOT: put client secret in `NEXT_PUBLIC_`. MUST NOT: `jwt-decode` the IdP token for the sidebar.

After callback: redirect same-origin `next` only (15).

---

## Done

- [ ] Start URL from the API; cookie from the API
- [ ] No IdP token in JS storage
