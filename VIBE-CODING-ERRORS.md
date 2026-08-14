# 50 Common Vibe-Coding Errors / 50 Yaygın Vibe-Coding Hatası

> **For humans** — quick-reference checklist when AI-built apps break.  
> **İnsanlar için** — yapay zeka ile yazılan uygulamalarda hızlı teşhis rehberi.

Not a playbook rule file. Pair with [VIBE-CODING-PITFALLS.md](VIBE-CODING-PITFALLS.md) (why mistakes happen) and [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) (how to prompt fixes).

**Agents:** [agents/04-errors.md](agents/04-errors.md) — fifty blocks as WHEN/WHERE/HOW/LOAD (not tables).

Playbook stacks: **Next.js UI** + **FastAPI API**. Fixes below assume that split unless noted.

**Root causes (predictable, not random):** context loss between files · deprecated APIs in training data · happy-path-only generation · no system-wide view. Studies report a large share of AI code gets reverted within weeks — usually because it never matched the **system**, not because every line was wrong.

---

## Error distribution / Hata dağılımı

| Category | Count | Share |
|----------|------:|------:|
| Build & compilation | 10 | 20% |
| Runtime & logic | 15 | 30% |
| API & data | 10 | 20% |
| Auth & authorization | 8 | 16% |
| Deployment | 7 | 14% |

Runtime dominates because agents skip edge cases ([VIBE-CODING-PITFALLS §9](VIBE-CODING-PITFALLS.md)).

---

## English

### How to use this page

1. Copy the **exact** error text from terminal, browser, or CI.
2. Find the number below (or search this file).
3. Apply the fix; `@` the playbook file in your next agent prompt.
4. **Do not** prompt “fix my app” without the error message ([HOW-TO-PROMPT.md](HOW-TO-PROMPT.md)).

**Agent prompt for any error:**

```text
Stack: [nextjs-frontend | python-fastapi-backend]
Error (verbatim): [paste]
File / line: [path:line]
Task: Fix only this error; minimal diff.
Playbook: [relevant numbered file from fix column]
Done when: error gone; tests pass; no unrelated refactor
```

---

### A · Build & compilation (1–10)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 1 | **Module not found** (npm/pip) | Install the package; lock the version in `package.json` / `pyproject.toml`. | — |
| 2 | **Cannot find module `./X`** | Verify path and **case** (Linux CI is case-sensitive). | [nextjs 03](nextjs-frontend/03-file-structure.md) |
| 3 | **Property does not exist** (TS) | Match the type/API; do not `as any` to silence. | [nextjs 02](nextjs-frontend/02-coding-principles.md) |
| 4 | **JSX must have one parent** | Wrap siblings in `<>...</>` or a single container. | [nextjs 11](nextjs-frontend/11-ui.md) |
| 5 | **Unexpected token** | Check line in message; look for unclosed `({[` above it. | — |
| 6 | **Duplicate identifier** | Remove or rename duplicate import/const/function. | [backend 01](python-fastapi-backend/01-coding-principles.md) |
| 7 | **X not assignable to Y** | Align argument type with function signature. | [nextjs 02](nextjs-frontend/02-coding-principles.md) |
| 8 | **import outside a module** | Align ESM/CJS: `"type": "module"` or consistent `require`. | — |
| 9 | **React must be in scope** | Use modern JSX transform or `import React`. | — |
| 10 | **Invalid hook call** | Hooks only at top level of client components — not in conditions. | [nextjs 05](nextjs-frontend/05-server-client.md) |

---

### B · Runtime & logic (11–25)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 11 | **Cannot read properties of undefined** | Optional chaining `?.`; guard before access; validate API shape. | [nextjs 07](nextjs-frontend/07-data.md) |
| 12 | **Hydration mismatch** (Next.js) | No `window`/`localStorage` during SSR; browser-only code in client leaf + `useEffect` if needed. | [nextjs 05](nextjs-frontend/05-server-client.md) |
| 13 | **Infinite re-render** | Fix `useEffect` deps; do not set state that re-triggers same effect. | [nextjs 13](nextjs-frontend/13-state.md) |
| 14 | **State not updating** | Functional `setState`; do not read stale state right after set. | [nextjs 13](nextjs-frontend/13-state.md) |
| 15 | **Memory leak in useEffect** | Return cleanup for subscriptions, timers, listeners. | [nextjs 05](nextjs-frontend/05-server-client.md) |
| 16 | **Maximum call stack** | Add recursion base case or replace with iteration. | [backend 01](python-fastapi-backend/01-coding-principles.md) |
| 17 | **Array index out of bounds** | Check `length` / use `.at()`; handle empty lists in UI (four states). | [nextjs 01](nextjs-frontend/01-design.md) |
| 18 | **Off-by-one pagination** | Align page index with API (`page` 0 vs 1); document in OpenAPI. | [backend 12](python-fastapi-backend/12-api.md) |
| 19 | **Race condition (async)** | `await` sequence; abort stale requests; idempotent writes on server. | [backend 06](python-fastapi-backend/06-database.md) |
| 20 | **Stale closure in handlers** | `useRef` for latest value or functional updates. | [nextjs 13](nextjs-frontend/13-state.md) |
| 21 | **Date / timezone wrong** | Store UTC in DB; convert for display only. | [backend 06](python-fastapi-backend/06-database.md) |
| 22 | **Float precision (money)** | Integer minor units (cents) in API/DB. | [backend 06](python-fastapi-backend/06-database.md) |
| 23 | **Encoding in URL/JSON** | `encodeURIComponent`; parameterized SQL — never concat user input. | [backend 15](python-fastapi-backend/15-security.md) |
| 24 | **CSS / Tailwind conflicts** | Tokens in [11-ui](nextjs-frontend/11-ui.md); avoid fighting library selectors. | [nextjs 01](nextjs-frontend/01-design.md) |
| 25 | **No loading / error UI** | Add loading, empty, error, success — mandatory four states. | [nextjs 01](nextjs-frontend/01-design.md) |

---

### C · API & data (26–35)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 26 | **CORS error** | Configure CORS on **FastAPI** for browser origin; UI calls API via [09-api-client](nextjs-frontend/09-api-client.md). | [backend 10](python-fastapi-backend/10-http.md) |
| 27 | **401 Unauthorized** | HttpOnly session cookie or Bearer from server; refresh on FastAPI — not ad-hoc headers in features. | [backend 13](python-fastapi-backend/13-identity-security.md), [nextjs 12](nextjs-frontend/12-auth.md) |
| 28 | **404 on API route** | Match mounted router path in `http/router.py` + module router prefix. | [backend 10](python-fastapi-backend/10-http.md), [09-modules](python-fastapi-backend/09-modules.md) |
| 29 | **429 Too Many Requests** | Backoff + cache; rate limits from `Settings`, not hardcoded. | [backend 03](python-fastapi-backend/03-config.md), [16](python-fastapi-backend/16-performance.md) |
| 30 | **JSON parse error** (HTML body) | Wrong URL or 502/404 page; check Network tab status + response type. | [backend 12](python-fastapi-backend/12-api.md) |
| 31 | **DB connection timeout** | Pool size/timeouts in `Settings`; close sessions per request/job. | [backend 03](python-fastapi-backend/03-config.md), [06](python-fastapi-backend/06-database.md) |
| 32 | **SQL injection** | SQLAlchemy/repository only — **no** string-built SQL from user input. | [backend 06](python-fastapi-backend/06-database.md), [15](python-fastapi-backend/15-security.md) |
| 33 | **Missing env var** | Add field to `Settings` / `lib/env.ts` + `.env.example`; verify CI/host env. | [backend 03](python-fastapi-backend/03-config.md), [nextjs 04](nextjs-frontend/04-config.md) |
| 34 | **Webhook signature fail** | Use correct secret per environment in `Settings`; raw body for verify. | [backend extra 09](python-fastapi-backend/extra/09-webhooks.md) |
| 35 | **JSON serialization error** | No `Date`/`Decimal`/NaN in response — use DTOs/schemas (Pydantic). | [backend 12](python-fastapi-backend/12-api.md) |

**API debug order:** (1) HTTP status → (2) browser Network tab → (3) FastAPI logs with `request_id` ([04-logging](python-fastapi-backend/04-logging.md)).

---

### D · Auth & authorization (36–43)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 36 | **Redirect loop on login** | Exclude public routes (login, callback) from auth middleware. | [nextjs 06](nextjs-frontend/06-routing.md), [12-auth](nextjs-frontend/12-auth.md) |
| 37 | **Session not persisting** | Cookie `HttpOnly`, `Secure` (HTTPS), `SameSite` matching deployment. | [nextjs 12](nextjs-frontend/12-auth.md), [backend 13](python-fastapi-backend/13-identity-security.md) |
| 38 | **OAuth callback mismatch** | IdP redirect URI must **exactly** match production URL. | [extra SSO](python-fastapi-backend/extra/07-sso.md), [nextjs extra 04](nextjs-frontend/extra/04-sso.md) |
| 39 | **RLS / tenant blocks all rows** | Explicit tenant filter in repository + tests — do not rely on “magic” defaults. | [extra multi-tenant](python-fastapi-backend/extra/01-multi-tenant.md) |
| 40 | **Token in localStorage** | **MUST NOT** — session in HttpOnly cookie; FastAPI owns authz. | [nextjs 12](nextjs-frontend/12-auth.md) |
| 41 | **Missing authorization check** | Every protected route: identity + permission in **service**, not UI-only. | [backend 13](python-fastapi-backend/13-identity-security.md) |
| 42 | **Password reset token never expires** | Short TTL in `Settings`; one-time use in DB. | [backend 13](python-fastapi-backend/13-identity-security.md) |
| 43 | **CSRF on mutations** | SameSite cookies + server-side writes via FastAPI; no secret in `NEXT_PUBLIC_`. | [nextjs 15](nextjs-frontend/15-security.md) |

---

### E · Deployment (44–50)

| # | Error | Quick fix | Playbook |
|---|--------|-----------|----------|
| 44 | **Build OK locally, fails in CI** | Pin Node/Python versions; set all env vars; fix path case. | [nextjs 04](nextjs-frontend/04-config.md), [backend 03](python-fastapi-backend/03-config.md) |
| 45 | **Serverless timeout** | Move heavy work to **worker** queue — not HTTP request. | [backend 11](python-fastapi-backend/11-workers.md) |
| 46 | **OOM during build** | Optimize images; trim deps; split apps if needed. | [nextjs 16](nextjs-frontend/16-performance.md) |
| 47 | **Mixed content (HTTP on HTTPS)** | All asset/API URLs HTTPS in env. | [nextjs 04](nextjs-frontend/04-config.md) |
| 48 | **Static gen error for dynamic page** | Server fetch + dynamic route; no stale static data for user-specific pages. | [nextjs 06](nextjs-frontend/06-routing.md), [07](nextjs-frontend/07-data.md) |
| 49 | **SSL / DNS not ready** | Wait for propagation; verify cert on host before cutting traffic. | — |
| 50 | **Cold start latency** | Long jobs off HTTP; cache warm paths; pools in `Settings`. | [backend 16](python-fastapi-backend/16-performance.md) |

---

### Where to start (by situation)

| You are… | Focus first |
|----------|-------------|
| New to vibe coding | #1–10, #11–15, #26–28 (~80% of first project pain) |
| Backend-heavy | #26–35, #36–41, [backend 15-security](python-fastapi-backend/15-security.md) |
| Frontend-heavy | #11–13, #25, [nextjs 01-design](nextjs-frontend/01-design.md), [07-data](nextjs-frontend/07-data.md) |
| Production incident | #33, #27, #31, #44 + git revert checkpoint ([VIBE-CODING-PITFALLS §3](VIBE-CODING-PITFALLS.md)) |

---

## Türkçe

### Bu sayfayı nasıl kullanırsınız

1. Terminal, tarayıcı veya CI'dan **aynı** hata metnini kopyalayın.
2. Aşağıdaki numarayı bulun (veya dosyada arayın).
3. Düzeltmeyi uygulayın; sonraki agent prompt'unda playbook dosyasını `@` ile verin.
4. Hata mesajı olmadan “uygulamayı düzelt” **yazmayın** ([HOW-TO-PROMPT.md](HOW-TO-PROMPT.md)).

---

### A · Derleme hataları (1–10)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 1 | **Module not found** | Paketi kurun; sürümü `package.json` / `pyproject.toml`'da sabitleyin. | — |
| 2 | **`./X` modülü bulunamadı** | Yol ve **büyük/küçük harf** (Linux CI duyarlı). | [nextjs 03](nextjs-frontend/03-file-structure.md) |
| 3 | **Property does not exist** | Tip/API ile eşleştirin; `as any` ile susturmayın. | [nextjs 02](nextjs-frontend/02-coding-principles.md) |
| 4 | **JSX tek parent ister** | Kardeşleri `<>...</>` veya tek kapsayıcıda toplayın. | [nextjs 11](nextjs-frontend/11-ui.md) |
| 5 | **Unexpected token** | Mesajdaki satır; üstte kapanmamış `({[` arayın. | — |
| 6 | **Duplicate identifier** | Yinelenen import/const/fonksiyonu kaldırın veya yeniden adlandırın. | [backend 01](python-fastapi-backend/01-coding-principles.md) |
| 7 | **X, Y'ye atanamaz** | Argüman tipini imzayla hizalayın. | [nextjs 02](nextjs-frontend/02-coding-principles.md) |
| 8 | **Modül dışında import** | ESM/CJS tutarlılığı. | — |
| 9 | **React scope'ta olmalı** | Modern JSX transform veya `import React`. | — |
| 10 | **Invalid hook call** | Hook'lar yalnızca client component'in üst seviyesinde. | [nextjs 05](nextjs-frontend/05-server-client.md) |

---

### B · Çalışma zamanı ve mantık (11–25)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 11 | **undefined property okuma** | `?.`; erişim öncesi guard; API şeklini doğrulayın. | [nextjs 07](nextjs-frontend/07-data.md) |
| 12 | **Hydration mismatch** | SSR'da `window`/`localStorage` yok; tarayıcı kodu client leaf'te. | [nextjs 05](nextjs-frontend/05-server-client.md) |
| 13 | **Sonsuz re-render** | `useEffect` bağımlılıkları; effect'i tetikleyen state güncellemesini düzeltin. | [nextjs 13](nextjs-frontend/13-state.md) |
| 14 | **State güncellenmiyor** | Fonksiyonel `setState`; set hemen sonrası eski state okumayın. | [nextjs 13](nextjs-frontend/13-state.md) |
| 15 | **useEffect bellek sızıntısı** | Abonelik/timer/listener için cleanup return edin. | [nextjs 05](nextjs-frontend/05-server-client.md) |
| 16 | **Maximum call stack** | Recursion taban durumu veya iterasyon. | [backend 01](python-fastapi-backend/01-coding-principles.md) |
| 17 | **Dizi sınır dışı** | `length` kontrolü; boş liste UI (dört durum). | [nextjs 01](nextjs-frontend/01-design.md) |
| 18 | **Sayfalama off-by-one** | API ile sayfa indeksini hizalayın (0 vs 1). | [backend 12](python-fastapi-backend/12-api.md) |
| 19 | **Async race condition** | `await` sırası; eski istekleri iptal; sunucuda idempotent yazma. | [backend 06](python-fastapi-backend/06-database.md) |
| 20 | **Stale closure** | `useRef` veya fonksiyonel güncelleme. | [nextjs 13](nextjs-frontend/13-state.md) |
| 21 | **Tarih / timezone** | DB'de UTC; gösterimde yerel. | [backend 06](python-fastapi-backend/06-database.md) |
| 22 | **Para float hatası** | API/DB'de tam sayı minor birim (kuruş). | [backend 06](python-fastapi-backend/06-database.md) |
| 23 | **URL/JSON encoding** | `encodeURIComponent`; parametreli SQL. | [backend 15](python-fastapi-backend/15-security.md) |
| 24 | **CSS / Tailwind çakışması** | Token'lar [11-ui](nextjs-frontend/11-ui.md)'da. | [nextjs 01](nextjs-frontend/01-design.md) |
| 25 | **Loading / error UI yok** | Dört durum: loading, empty, error, success. | [nextjs 01](nextjs-frontend/01-design.md) |

---

### C · API ve veri (26–35)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 26 | **CORS** | CORS **FastAPI**'de; UI [09-api-client](nextjs-frontend/09-api-client.md) ile çağırır. | [backend 10](python-fastapi-backend/10-http.md) |
| 27 | **401** | HttpOnly cookie veya sunucu tarafı Bearer; refresh FastAPI'de. | [backend 13](python-fastapi-backend/13-identity-security.md), [nextjs 12](nextjs-frontend/12-auth.md) |
| 28 | **404 route** | `http/router.py` mount + modül prefix eşleşmesi. | [backend 10](python-fastapi-backend/10-http.md) |
| 29 | **429** | Backoff + cache; limitler `Settings`'te. | [backend 03](python-fastapi-backend/03-config.md) |
| 30 | **JSON parse (HTML döndü)** | Yanlış URL veya hata sayfası; Network sekmesi. | [backend 12](python-fastapi-backend/12-api.md) |
| 31 | **DB timeout** | Pool/timeout `Settings`'te; oturum kapatma. | [backend 03](python-fastapi-backend/03-config.md) |
| 32 | **SQL injection** | Repository/SQLAlchemy — kullanıcı girdisiyle string SQL yok. | [backend 06](python-fastapi-backend/06-database.md) |
| 33 | **Eksik env** | `Settings` / `lib/env.ts` + `.env.example`; CI/host kontrolü. | [backend 03](python-fastapi-backend/03-config.md), [nextjs 04](nextjs-frontend/04-config.md) |
| 34 | **Webhook imza hatası** | Ortama göre secret `Settings`'te. | [extra 09](python-fastapi-backend/extra/09-webhooks.md) |
| 35 | **JSON serileştirme** | Pydantic DTO; ham `Date`/NaN yok. | [backend 12](python-fastapi-backend/12-api.md) |

**API teşhis sırası:** status kodu → Network → FastAPI log (`request_id`).

---

### D · Kimlik doğrulama ve yetkilendirme (36–43)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 36 | **Login redirect döngüsü** | Public route'ları middleware'den hariç tutun. | [nextjs 06](nextjs-frontend/06-routing.md), [12](nextjs-frontend/12-auth.md) |
| 37 | **Oturum kalıcı değil** | `HttpOnly`, `Secure`, `SameSite` deployment'a uygun. | [nextjs 12](nextjs-frontend/12-auth.md) |
| 38 | **OAuth callback uyuşmazlığı** | IdP redirect URI production ile **birebir** aynı. | [extra SSO](python-fastapi-backend/extra/07-sso.md) |
| 39 | **Tenant/RLS her şeyi bloklar** | Repository'de açık tenant filtresi + test. | [extra 01](python-fastapi-backend/extra/01-multi-tenant.md) |
| 40 | **Token localStorage'da** | **Yasak** — HttpOnly cookie; yetki FastAPI'de. | [nextjs 12](nextjs-frontend/12-auth.md) |
| 41 | **Yetki kontrolü yok** | Korumalı route: kimlik + izin **service** katmanında. | [backend 13](python-fastapi-backend/13-identity-security.md) |
| 42 | **Reset token süresiz** | Kısa TTL `Settings`'te; tek kullanımlık. | [backend 13](python-fastapi-backend/13-identity-security.md) |
| 43 | **CSRF** | SameSite + mutation FastAPI'de; `NEXT_PUBLIC_` secret yok. | [nextjs 15](nextjs-frontend/15-security.md) |

---

### E · Deploy (44–50)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 44 | **Lokal OK, CI fail** | Node/Python sürümü; tüm env; path case. | [nextjs 04](nextjs-frontend/04-config.md), [backend 03](python-fastapi-backend/03-config.md) |
| 45 | **Serverless timeout** | Ağır iş **worker** kuyruğuna — HTTP'de değil. | [backend 11](python-fastapi-backend/11-workers.md) |
| 46 | **Build OOM** | Görsel/dependency optimizasyonu. | [nextjs 16](nextjs-frontend/16-performance.md) |
| 47 | **Mixed content** | Tüm URL'ler HTTPS. | [nextjs 04](nextjs-frontend/04-config.md) |
| 48 | **Statik sayfa dinamik veri** | Sunucu fetch; kullanıcıya özel sayfada statik üretim yok. | [nextjs 06](nextjs-frontend/06-routing.md), [07](nextjs-frontend/07-data.md) |
| 49 | **SSL / DNS** | Yayılım; sertifika doğrulama. | — |
| 50 | **Cold start gecikmesi** | Uzun iş HTTP dışı; pool `Settings`'te. | [backend 16](python-fastapi-backend/16-performance.md) |

---

### Nereden başlanır

| Durum | Önce odaklan |
|-------|----------------|
| Vibe coding'e yeni | #1–10, #11–15, #26–28 |
| Backend ağırlıklı | #26–35, #36–41, [15-security](python-fastapi-backend/15-security.md) |
| Frontend ağırlıklı | #11–13, #25, [01-design](nextjs-frontend/01-design.md) |
| Production olayı | #33, #27, #31, #44 + git revert |

---

## Related / İlgili

- [VIBE-CODING-PITFALLS.md](VIBE-CODING-PITFALLS.md) — behavioral mistakes
- [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) — prompt for fixes
- [USER-GUIDE.md](USER-GUIDE.md) — repo setup
