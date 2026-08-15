# 50 yaygın vibe-coding hatası

> **İnsanlar için** — yapay zeka ile yazılan bir uygulama bozulduğunda hızlı teşhis listesi.

Playbook yığınları: **Next.js arayüz** + **FastAPI API**. Aşağıdaki çözümler aksi belirtilmedikçe bu ayrımı varsayar.

**Kök nedenler rastgele değil, öngörülebilir:** dosyalar arasında kaybolan bağlam · eğitim verisindeki eskimiş API'ler · yalnızca mutlu yolu üretme · sistemin bütününü görmeme. Yapay zeka kodunun büyük bir kısmı haftalar içinde geri alınıyor — genelde her satırı yanlış olduğu için değil, **sisteme** hiç uymadığı için.

| Kategori | Adet | Pay |
|----------|-----:|----:|
| Derleme | 10 | %20 |
| Çalışma zamanı ve mantık | 15 | %30 |
| API ve veri | 10 | %20 |
| Kimlik ve yetki | 8 | %16 |
| Deploy | 7 | %14 |

Çalışma zamanı baskın, çünkü ajanlar edge case'leri atlıyor ([04-pitfalls §9](04-pitfalls.md)).

**English:** [05-errors.md](../en/05-errors.md) · **Dizin:** [for-humans/](../README.md) · **Ajanlar şunu okur:** [agents/README.md](../../agents/README.md)

---

## Bu sayfayı nasıl kullanırsınız

1. Terminal, tarayıcı veya CI'dan **aynı** hata metnini kopyalayın.
2. Aşağıdaki numarayı bulun (veya dosyada arayın).
3. Düzeltmeyi uygulayın; sonraki agent prompt'unda playbook dosyasını `@` ile verin.
4. Hata mesajı olmadan “uygulamayı düzelt” **yazmayın** ([02 Prompt yazımı](02-how-to-prompt.md)).

---

## A · Derleme hataları (1–10)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 1 | **Module not found** | Paketi kurun; sürümü `package.json` / `pyproject.toml`'da sabitleyin. | — |
| 2 | **`./X` modülü bulunamadı** | Yol ve **büyük/küçük harf** (Linux CI duyarlı). | [nextjs 03](../../nextjs-frontend/03-file-structure.md) |
| 3 | **Property does not exist** | Tip/API ile eşleştirin; `as any` ile susturmayın. | [nextjs 02](../../nextjs-frontend/02-coding-principles.md) |
| 4 | **JSX tek parent ister** | Kardeşleri `<>...</>` veya tek kapsayıcıda toplayın. | [nextjs 11](../../nextjs-frontend/11-ui.md) |
| 5 | **Unexpected token** | Mesajdaki satır; üstte kapanmamış `({[` arayın. | — |
| 6 | **Duplicate identifier** | Yinelenen import/const/fonksiyonu kaldırın veya yeniden adlandırın. | [backend 01](../../python-fastapi-backend/01-coding-principles.md) |
| 7 | **X, Y'ye atanamaz** | Argüman tipini imzayla hizalayın. | [nextjs 02](../../nextjs-frontend/02-coding-principles.md) |
| 8 | **Modül dışında import** | ESM/CJS tutarlılığı. | — |
| 9 | **React scope'ta olmalı** | Modern JSX transform veya `import React`. | — |
| 10 | **Invalid hook call** | Hook'lar yalnızca client component'in üst seviyesinde. | [nextjs 05](../../nextjs-frontend/05-server-client.md) |

---

## B · Çalışma zamanı ve mantık (11–25)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 11 | **undefined property okuma** | `?.`; erişim öncesi guard; API şeklini doğrulayın. | [nextjs 07](../../nextjs-frontend/07-data.md) |
| 12 | **Hydration mismatch** | SSR'da `window`/`localStorage` yok; tarayıcı kodu client leaf'te. | [nextjs 05](../../nextjs-frontend/05-server-client.md) |
| 13 | **Sonsuz re-render** | `useEffect` bağımlılıkları; effect'i tetikleyen state güncellemesini düzeltin. | [nextjs 13](../../nextjs-frontend/13-state.md) |
| 14 | **State güncellenmiyor** | Fonksiyonel `setState`; set hemen sonrası eski state okumayın. | [nextjs 13](../../nextjs-frontend/13-state.md) |
| 15 | **useEffect bellek sızıntısı** | Abonelik/timer/listener için cleanup return edin. | [nextjs 05](../../nextjs-frontend/05-server-client.md) |
| 16 | **Maximum call stack** | Recursion taban durumu veya iterasyon. | [backend 01](../../python-fastapi-backend/01-coding-principles.md) |
| 17 | **Dizi sınır dışı** | `length` kontrolü; boş liste UI (dört durum). | [nextjs 01](../../nextjs-frontend/01-design.md) |
| 18 | **Sayfalama off-by-one** | API ile sayfa indeksini hizalayın (0 vs 1). | [backend 12](../../python-fastapi-backend/12-api.md) |
| 19 | **Async race condition** | `await` sırası; eski istekleri iptal; sunucuda idempotent yazma. | [backend 06](../../python-fastapi-backend/06-database.md) |
| 20 | **Stale closure** | `useRef` veya fonksiyonel güncelleme. | [nextjs 13](../../nextjs-frontend/13-state.md) |
| 21 | **Tarih / timezone** | DB'de UTC; gösterimde yerel. | [backend 06](../../python-fastapi-backend/06-database.md) |
| 22 | **Para float hatası** | API/DB'de tam sayı minor birim (kuruş). | [backend 06](../../python-fastapi-backend/06-database.md) |
| 23 | **URL/JSON encoding** | `encodeURIComponent`; parametreli SQL. | [backend 15](../../python-fastapi-backend/15-security.md) |
| 24 | **CSS / Tailwind çakışması** | Token'lar [11-ui](../../nextjs-frontend/11-ui.md)'da. | [nextjs 01](../../nextjs-frontend/01-design.md) |
| 25 | **Loading / error UI yok** | Dört durum: loading, empty, error, success. | [nextjs 01](../../nextjs-frontend/01-design.md) |

---

## C · API ve veri (26–35)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 26 | **CORS** | CORS **FastAPI**'de; UI [09-api-client](../../nextjs-frontend/09-api-client.md) ile çağırır. | [backend 10](../../python-fastapi-backend/10-http.md) |
| 27 | **401** | HttpOnly cookie veya sunucu tarafı Bearer; refresh FastAPI'de. | [backend 13](../../python-fastapi-backend/13-identity-security.md), [nextjs 12](../../nextjs-frontend/12-auth.md) |
| 28 | **404 route** | `http/router.py` mount + modül prefix eşleşmesi. | [backend 10](../../python-fastapi-backend/10-http.md) |
| 29 | **429** | Backoff + cache; limitler `Settings`'te. | [backend 03](../../python-fastapi-backend/03-config.md) |
| 30 | **JSON parse (HTML döndü)** | Yanlış URL veya hata sayfası; Network sekmesi. | [backend 12](../../python-fastapi-backend/12-api.md) |
| 31 | **DB timeout** | Pool/timeout `Settings`'te; oturum kapatma. | [backend 03](../../python-fastapi-backend/03-config.md) |
| 32 | **SQL injection** | Repository/SQLAlchemy — kullanıcı girdisiyle string SQL yok. | [backend 06](../../python-fastapi-backend/06-database.md) |
| 33 | **Eksik env** | `Settings` / `lib/env.ts` + `.env.example`; CI/host kontrolü. | [backend 03](../../python-fastapi-backend/03-config.md), [nextjs 04](../../nextjs-frontend/04-config.md) |
| 34 | **Webhook imza hatası** | Ortama göre secret `Settings`'te. | [extra 09](../../python-fastapi-backend/extra/09-webhooks.md) |
| 35 | **JSON serileştirme** | Pydantic DTO; ham `Date`/NaN yok. | [backend 12](../../python-fastapi-backend/12-api.md) |

**API teşhis sırası:** status kodu → Network → FastAPI log (`request_id`).

---

## D · Kimlik doğrulama ve yetkilendirme (36–43)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 36 | **Login redirect döngüsü** | Public route'ları middleware'den hariç tutun. | [nextjs 06](../../nextjs-frontend/06-routing.md), [12](../../nextjs-frontend/12-auth.md) |
| 37 | **Oturum kalıcı değil** | `HttpOnly`, `Secure`, `SameSite` deployment'a uygun. | [nextjs 12](../../nextjs-frontend/12-auth.md) |
| 38 | **OAuth callback uyuşmazlığı** | IdP redirect URI production ile **birebir** aynı. | [extra SSO](../../python-fastapi-backend/extra/07-sso.md) |
| 39 | **Tenant/RLS her şeyi bloklar** | Repository'de açık tenant filtresi + test. | [extra 01](../../python-fastapi-backend/extra/01-multi-tenant.md) |
| 40 | **Token localStorage'da** | **Yasak** — HttpOnly cookie; yetki FastAPI'de. | [nextjs 12](../../nextjs-frontend/12-auth.md) |
| 41 | **Yetki kontrolü yok** | Korumalı route: kimlik + izin **service** katmanında. | [backend 13](../../python-fastapi-backend/13-identity-security.md) |
| 42 | **Reset token süresiz** | Kısa TTL `Settings`'te; tek kullanımlık. | [backend 13](../../python-fastapi-backend/13-identity-security.md) |
| 43 | **CSRF** | SameSite + mutation FastAPI'de; `NEXT_PUBLIC_` secret yok. | [nextjs 15](../../nextjs-frontend/15-security.md) |

---

## E · Deploy (44–50)

| # | Hata | Hızlı çözüm | Playbook |
|---|------|-------------|----------|
| 44 | **Lokal OK, CI fail** | Node/Python sürümü; tüm env; path case. | [nextjs 04](../../nextjs-frontend/04-config.md), [backend 03](../../python-fastapi-backend/03-config.md) |
| 45 | **Serverless timeout** | Ağır iş **worker** kuyruğuna — HTTP'de değil. | [backend 11](../../python-fastapi-backend/11-workers.md) |
| 46 | **Build OOM** | Görsel/dependency optimizasyonu. | [nextjs 16](../../nextjs-frontend/16-performance.md) |
| 47 | **Mixed content** | Tüm URL'ler HTTPS. | [nextjs 04](../../nextjs-frontend/04-config.md) |
| 48 | **Statik sayfa dinamik veri** | Sunucu fetch; kullanıcıya özel sayfada statik üretim yok. | [nextjs 06](../../nextjs-frontend/06-routing.md), [07](../../nextjs-frontend/07-data.md) |
| 49 | **SSL / DNS** | Yayılım; sertifika doğrulama. | — |
| 50 | **Cold start gecikmesi** | Uzun iş HTTP dışı; pool `Settings`'te. | [backend 16](../../python-fastapi-backend/16-performance.md) |

---

## Nereden başlanır

| Durum | Önce odaklan |
|-------|----------------|
| Vibe coding'e yeni | #1–10, #11–15, #26–28 |
| Backend ağırlıklı | #26–35, #36–41, [15-security](../../python-fastapi-backend/15-security.md) |
| Frontend ağırlıklı | #11–13, #25, [01-design](../../nextjs-frontend/01-design.md) |
| Production olayı | #33, #27, #31, #44 + git revert |

---

**Diğer rehberler:** [01 Buradan başlayın](01-start-here.md) · [02 Prompt yazımı](02-how-to-prompt.md) · [03 Ajan kodunu inceleme](03-review-agent-code.md) · [04 Tuzaklar](04-pitfalls.md) · [05 Hatalar](05-errors.md) · [06 Sözlük](06-glossary.md)
