# Buradan başlayın

> **İnsanlar için** — playbook'u projeye alan veya sürdüren ürün sahipleri, ekip liderleri ve geliştiriciler.

Her yığındaki numaralı dosyalar `WHEN` / `LOAD` / `MUST` diliyle yazılmıştır — o sözdizimi **yapay zeka ajanları** içindir. Repoyu kullanmak için bilmeniz gerekmez. Önce bu sayfayı okuyun, sonra yaptığınız işe uyan rehbere geçin.

**English:** [01-start-here.md](../en/01-start-here.md) · **Dizin:** [for-humans/](../README.md) · **Ajanlar şunu okur:** [agents/README.md](../../agents/README.md)

---

## Bu repo nedir?

**Coding Playbook**, iki yığın için mimari ve kodlama kurallarından oluşan bir **referans kütüphanesidir**:

| Yığın | Klasör |
|-------|--------|
| Python + FastAPI (API, worker'lar) | [`python-fastapi-backend/`](../../python-fastapi-backend/README.md) |
| Next.js (arayüz) | [`nextjs-frontend/`](../../nextjs-frontend/README.md) |

Bu klasör **uygulama kaynak kodu değildir**. Tümünü `src/` içine kopyalamayın. Gerçek uygulamayı başka yerde geliştirirken ekibinizin ve ajanlarınızın uyacağı bir **sözleşme** olarak kullanın.

## Projede nasıl kullanılır?

1. **Ayrı tutun.** Repoyu uygulamanızın yanına klonlayın veya submodule olarak ekleyin; yalnızca ihtiyacınız olan yığın klasörünü kopyalayabilirsiniz. Playbook `coding-playbook/` gibi kalır; uygulama `backend/`, `frontend/` vb. dizinlerde durur.

2. **Tek bir yığın haritası seçin.** Geliştirdiğiniz yığının README dosyasını açın:
   - Backend → [`python-fastapi-backend/README.md`](../../python-fastapi-backend/README.md)
   - Frontend → [`nextjs-frontend/README.md`](../../nextjs-frontend/README.md)

3. **Ekibe alırken sırayla okuyun.** Her yığında `01`–`16` numaralı dosyalar vardır (ilkeler → yapı → config → … → test / güvenlik / performans). İlk günde hepsine gerek yok; `01`, `02`, `03` ile başlayın, sonra yaptığınız işe uygun dosyayı açın (auth, form, worker, …).

4. **Extra yalnızca özellik zaten varsa.** `extra/` altındaki konular (multi-tenant, SSO, agent, arama, …) `01`–`16` kurallarına **ek** gelir. Doküman diye o yapıları boşuna kurmayın.

5. **Kuralları ürününüze uyarlayın.** Dosyalar başlangıç haritasıdır; değişmez kanun değildir. Bir kural yığınınız, host'unuz veya mevcut tasarımınızla çelişiyorsa **playbook dosyasını güncelleyin** ve dosyaya tek satırlık bir gerekçe bırakın. Böylece sonraki kişi (veya ajan) kararı sohbet geçmişinde değil, git'te okur.

6. **Ajanları playbook'a yönlendirin.** Cursor (veya benzeri) içinde kural veya `@` referansı verin: önce [`README.md`](../../README.md), sonra ilgili yığın README'si, en son yalnızca o görevle ilgili numaralı dosya. Ajanlar on altı dosyanın veya tüm Extra konularının tamamını aynı anda yüklememelidir. Prompt şablonları, config tuzakları ve uzun sohbet çözümleri: **[02 Prompt yazımı](02-how-to-prompt.md)**.

## Ürün dokümanları

Bu playbook kurallardır. **Uygulama reposunun** kendi README'si ve `docs/` klasörü hâlâ gerekir — sonraki kişinin (ve sonraki ajanın) *sizin* ürününüz hakkında okuduğu yer orasıdır.

Mimari sorulardan sonra ilk gün: `docs/data-model.md` ve `docs/architecture/overview.md`. Ağacın geri kalanı o şekil var olduğunda yazılır, boş şablon olarak değil. Ajanlar [`agents/09-docs.md`](../../agents/09-docs.md) dosyasına uyar. Görev notları sohbette değil, git-ignore edilen `.agent/plan.md` içindedir.

```
repo/
├── README.md                          # indeks, teknik doküman değil
├── CHANGELOG.md                       # ne çıktı, kullanıcı için
└── docs/
    ├── data-model.md                  # isimler → ER / validasyon büyür
    ├── api.md                         # herkese açık HTTP sözleşmesi varken
    ├── security.md                    # kimlik veya kişisel veri varken
    ├── known-issues.md                # ilk yayın veya teslim — ne bitmedi
    ├── handover.md                    # başka ekip ürünü alırken
    ├── lessons-learned.md             # proje veya faz kapanırken
    ├── architecture/
    │   ├── overview.md                # sistem *şu an* nasıl çalışıyor
    │   └── decisions/                 # ADR: neden, ne değil
    ├── development/
    │   ├── setup.md
    │   ├── development-guide.md
    │   └── testing.md
    └── operations/
        ├── deployment.md              # nasıl yayınlarız, nasıl tekrarlarız
        ├── runbook.md                 # production nasıl işletilir
        └── slo.md                     # anlaşılmış hedefler, varken
```

Bu dosyaların hiçbirine secret koymayın. Kodun mimarisi veya API'si değiştiyse eşleşen doküman aynı değişiklikle gider. Yayın veya teslimde: yaşayan mimari / veri modeli / API güncellenir — ayrı bir "final architecture" dosyası açılmaz. Known issues, teslimin her şeyin bittiği izlenimini vermesin diye vardır.

## Sık senaryolar

| Amaç | Nereden başlanır |
|------|------------------|
| Yeni FastAPI servisi | `python-fastapi-backend/01` → `02` → `03`, ardından konu dosyaları |
| Yeni Next.js uygulaması | `nextjs-frontend/01` (tasarım) → `02` → `03`, ardından konu dosyaları |
| Code review kontrol listesi | İlgili yığında `15-security.md` ve `14-testing.md` |
| Sonradan SSO / tenant / agent | Özellik **vardıktan sonra** `extra/` altındaki eşleşen dosya |
| Yayın, teslim veya proje kapanışı | [`agents/09-docs.md`](../../agents/09-docs.md) — changelog, known issues, runbook; yaşayan dokümanlar, yeni mimari dosyası değil |

## İnsan okuyucu olarak nelere takılmayın

- `WHEN:` / `LOAD:` / `MUST NOT:` satırları ajanlar için yönlendirme talimatıdır. Üstünden geçin veya altındaki açıklama bölümlerini okuyun.
- [`README.md`](../../README.md) içinde **“Agent instructions”** çizgisinin altı ajanlara yönelik indekstir; ekibe alırken bunun yerine bu kullanıcı rehberini kullanın.

---

**Diğer rehberler:** [01 Buradan başlayın](01-start-here.md) · [02 Prompt yazımı](02-how-to-prompt.md) · [03 Ajan kodunu inceleme](03-review-agent-code.md) · [04 Tuzaklar](04-pitfalls.md) · [05 Hatalar](05-errors.md) · [06 Sözlük](06-glossary.md)
