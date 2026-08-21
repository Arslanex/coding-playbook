# Sözlük

> **İnsanlar için** — playbook'un durup açıklamadan kullandığı her terim.

Bir kural dosyası gözünüzde canlanmayan bir şey söylediğinde buraya bakın. Terimler alfabetik değil, **nerede karşılaştığınıza** göre gruplandı — çünkü onları öyle arayacaksınız.

**English:** [06-glossary.md](../en/06-glossary.md) · **Dizin:** [for-humans/](../README.md) · **Ajanlar şunu okur:** [agents/README.md](../../agents/README.md)

---

## Playbook'un kendisi

| Terim | Ne demek |
|---|---|
| **Playbook** | Bu repo. Ekibinizin ve ajanlarınızın uyduğu kurallar. Uygulama kodu değildir — asla `src/` içine kopyalanmaz. |
| **Yığın (stack)** | İki kural setinden biri: `python-fastapi-backend/` veya `nextjs-frontend/`. Görev başına bir tane seçilir. |
| **Numaralı dosya** | Her yığındaki `01`–`16`. Sıra, önce neye karar verdiğinize göre. Ajan görev başına **bir** tanesini açar, on altısını değil. |
| **Extra** | `extra/` altındaki isteğe bağlı kurallar (multi-tenant, SSO, arama…). Yalnızca ürün o şekle **zaten** sahipse yüklenir — ileride lazım olur diye asla. |
| **`WHEN:` / `LOAD:`** | Ajanlar için yönlendirme satırları: bu dosya ne zaman geçerli, başka ne açılmalı. İnsanlar üstünden geçebilir. |
| **`MUST [critical]`** | İhlali bir sır sızdırır, başka kullanıcının verisini açar veya bir yetki kapısını kaldırır. Tercih meselesi değil. |
| **`MUST` / `SHOULD`** | İşaretsiz `MUST`, playbook'un seçtiği şekildir — tutarlılık, güvenlik değil. `SHOULD` gerekçeyle sapılabilen bir varsayılandır. |
| **Dilim (slice)** | Ajanın bir turda yaptığı tek iş birimi: bir özellik, bir hata, bir migration. "Dashboard'u yap" değil. |
| **Git'te uyarlama** | Bir kural ürününüze uymadığında playbook dosyasını tek satır gerekçeyle değiştirirsiniz — böylece karar sohbet kaydında değil repoda yaşar. |
| **Ürün dokümanları** | Uygulama reposundaki `README.md` ve `docs/`. Bu playbook değil. İlk gün veri modeli ve mimari özet; kurulum, ADR, API, güvenlik ve deploy o şekil varken yazılır. Ajanlar: [`09-docs.md`](../../agents/09-docs.md). |
| **ADR** | Architecture Decision Record — `docs/architecture/decisions/ADR-NNN-….md`. Bir seçimin nedeni, reddedilenler, bedeli. "Ne kullanıyoruz"un tekrarı değil. |
| **Görev planı** | Ürün reposunda `.agent/plan.md`, git-ignore. **Bu** görevin dilimleri ve notları. Görev bitince yaşaması gereken şey burada değil, ürün dokümanında veya playbook'ta durur. |
| **Release notes** | `CHANGELOG.md` — ne çıktı; kullanıcı ve ürün ekibi için. Deploy runbook'u değil. |
| **Runbook** | `docs/operations/runbook.md` — production ayaktayken ve değilken nasıl işletilir. Deploy adımları `deployment.md`'de kalır. |
| **SLO / SLI** | Anlaşılmış operasyon hedefleri (gecikme, hata, uptime) ve onları izleyen dashboard. Uydurulmuş yüzdeler SLO değildir. |
| **Handover** | `docs/handover.md` — başka ekip ürünü alırken indeks, erişim, kişiler ve nereye dokunmanın riskli olduğu. |

---

## Arayüz (Next.js)

| Terim | Ne demek |
|---|---|
| **App Router** | Playbook'un hedeflediği Next.js yönlendirme modeli — `app/` altındaki klasörler URL olur. Eski "Pages Router" farklı bir playbook'tur. |
| **Server Component (RSC)** | Sunucuda çalışıp HTML gönderen bileşen. **Varsayılan** budur. Cookie okuyabilir, API'yi doğrudan çağırabilir ve tarayıcıya JavaScript göndermez. |
| **Client Component** | `"use client"` ile işaretlenmiş, tıklama işleyebilmek ve state tutabilmek için tarayıcıya gönderilen bileşen. Varsayılan değil, bir maliyettir — işareti ihtiyacı olan en küçük yaprağa koyun. |
| **Server Action** | Formun POST edebildiği, `"use server"` ile işaretli fonksiyon. Bu playbook'ta API'ye **ince** bir vekildir. Aynı zamanda herkesin çağırabildiği bir public HTTP endpoint'tir, bu yüzden asla izin kontrolünün yeri değildir. |
| **Hydration** | Tarayıcının, sunucunun zaten render ettiği bileşeni yeniden çalıştırması. "Hydration error" ikisinin uyuşmadığı anlamına gelir. |
| **Dört durum** | Her liste, form ve panelde yükleniyor, boş, hata ve başarı olmalı — sadece başarı yolu değil. |
| **Token** | Ham hex veya piksel yerine adlandırılmış tasarım değeri (`--surface`, `--space-2`). Bileşenler adı kullanır; hex tek dosyada durur. |
| **Primitive** | Ürün anlamı olmayan arayüz parçası: Button, TextField, Modal. `ui/` içinde yaşar. Dosya adında ürün ismi varsa o bir feature'dır. |
| **Feature** | Bir ürün ismi ve ekranda ifade ettiği her şey — `features/orders/`. Sayfalar feature'ları birleştirir; feature'lar sayfayı import etmez. |
| **AI-slop** | Üretken modelin varsayılan görünümü: mor gradyanlar, üç eşit kart, dört istatistik kutusu olan koyu sidebar. Görüldüğü yerde reddedilir. |
| **Şelale (waterfall)** | Birlikte çalışmak yerine birbirini bekleyen istekler — genelde spinner'dan sonra bir client fetch, ardından bir tane daha. |
| **`NEXT_PUBLIC_`** | Bir ortam değişkenini tarayıcıya görünür kılan önek. Build anında bundle'a gömülür ve sonsuza dek geneldir. Asla sır olmaz. |

---

## Sunucu (FastAPI)

| Terim | Ne demek |
|---|---|
| **Modül** | `modules/` altındaki tek bir ürün yeteneği — orders, billing, auth. Kurallarının sahibidir. Silinmesi kullanıcının yapabildiği bir şeyi kaldırır. |
| **Infra** | Anlamına sahip olmadığınız sistemlere adaptörler: Postgres, Redis, S3, Stripe. İş kuralı barındırmaz. |
| **Repository** | Tek bir tabloyu okuyup yazan ince katman. İş kuralı yok, commit de etmez. |
| **Service** | Ürün kuralının yaşadığı yer ("ödendiyse iptal edilemez"). HTTP route'ları da worker'lar da onu çağırır. |
| **Worker** | HTTP isteğinin dışında, kuyruğa alınmış bir işi tüketen süreç — yavaş, tekrar denenebilir veya dağıtılan işler için. |
| **Alembic revision** | Bir şema değişikliğini ve nasıl geri alınacağını anlatan tek dosya. Şema geçmişidir; release job'ı tarafından oynatılır — uygulama açılışta migration çalıştırmaz. |
| **Expand / contract** | Yıkıcı bir şema değişikliğini iki sürüme yayarak, rolling deploy sırasında eski ve yeni kodun aynı veritabanını paylaşabilmesini sağlamak. |
| **Cursor pagination** | `page=2` yerine "bu kayıttan sonrasını ver" ile sayfalama. Altınıza satır eklenirken bile tutarlı kalır. |
| **Idempotency key** | İstemcinin gönderdiği, API'nin tekrarlanan isteği tanıyıp iki kez ücretlendirmemesini sağlayan kimlik. |
| **DLQ** | Dead-letter queue — bir işin çok kez başarısız olduktan sonra gittiği yer; kaybolmak yerine incelenebilir olur. |
| **Outbox** | "Bu olay yayınlanmalı" kaydını, veri değişikliğiyle **aynı** transaction'a yazmak; böylece ikisi birbiriyle çelişemez. |

---

## Güvenlik ve bağımlılıklar

| Terim | Ne demek |
|---|---|
| **Kimlik doğrulama (authn)** | Kim olduğunuz. Kimliği kanıtlamak — giriş yapmak. |
| **Yetkilendirme (authz)** | Ne yapabildiğiniz. Bu playbook'ta her zaman API'de uygulanır, buton gizleyerek asla. |
| **Sahibi olmayana 404** | Kayıt var ama sizin değilse "yasak" yerine "bulunamadı" dönmek — çünkü "yasak" kaydın var olduğunu doğrular. |
| **HttpOnly cookie** | JavaScript'in okuyamadığı cookie. Oturumun yaşadığı yer; enjekte edilen bir script bile onu çalamaz. |
| **XSS** | Sayfaya script enjekte etmek. React metni varsayılan olarak kaçırır; `dangerouslySetInnerHTML` bu korumadan çıkmaktır. |
| **CSRF** | Başka bir sitenin sizin adınıza kimlikli istek atması. `SameSite` cookie veya API'nin istediği bir token ile engellenir. |
| **BFF** | Backend-for-frontend — cookie'yi tutan ve tarayıcı adına gerçek API ile konuşan ince sunucu katmanı. |
| **Lockfile** | Her bağımlılığın — dolaylı olanlar dahil — tam olarak hangi sürümünün kurulduğunu kaydeden dosya. Commit edilir; CI **ondan** kurar. |
| **Pinleme** | Build'lerin tekrarlanabilir olması için bağımlılık sürümünü sabitlemek. Asla yükseltmemekle aynı şey değildir. |
| **Advisory / CVE** | Belirli bir paket sürümünde bilinen bir açık olduğunu duyuran yayın. Böyle bir bağımlılık backlog değil, bulgudur. |
| **Slopsquatting** | Yapay zeka araçlarının uydurma ihtimali yüksek paket adlarını önceden kaydetmek; böylece halüsinasyon import saldırganın kodunu kurar. Her yeni bağımlılığın registry'den doğrulanma sebebi. |
| **Tedarik zinciri** | Kurduğunuz her şey ve onun kurduğu her şey. Siz yazmadan uygulamanızın yetkileriyle çalışan kod. |

---

**Diğer rehberler:** [01 Buradan başlayın](01-start-here.md) · [02 Prompt yazımı](02-how-to-prompt.md) · [03 Ajan kodunu inceleme](03-review-agent-code.md) · [04 Tuzaklar](04-pitfalls.md) · [05 Hatalar](05-errors.md) · [06 Sözlük](06-glossary.md)
