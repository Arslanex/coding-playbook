# Coding Playbook — User Guide / Kullanıcı Rehberi

> **For humans** — product owners, tech leads, and developers who adopt or maintain this playbook.  
> **İnsanlar için** — playbook'u projeye alan veya güncelleyen ekip liderleri ve geliştiriciler.

The numbered files below this guide use `WHEN` / `LOAD` / `MUST` language aimed at **AI coding agents**. You do not need that syntax to use the repo; read this page first.

**Agent-readable versions** (WHEN / WHERE / HOW / LOAD — no tables): [agents/README.md](agents/README.md)

Aşağıdaki numaralı dosyalar `WHEN` / `LOAD` / `MUST` sözdizimiyle **yapay zeka ajanlarına** yöneliktir. Repoyu kullanmak için o sözdizimini bilmeniz gerekmez; önce bu sayfayı okuyun.

**Ajanların okuyacağı sürüm:** [agents/README.md](agents/README.md)

---

## English

### What this repo is

**Coding Playbook** is a reference library of architecture and coding rules for two stacks:

| Stack | Folder |
|-------|--------|
| Python + FastAPI (API, workers) | [`python-fastapi-backend/`](python-fastapi-backend/README.md) |
| Next.js (UI) | [`nextjs-frontend/`](nextjs-frontend/README.md) |

It is **not** application source code. Do not copy the whole folder into `src/`. Use it as a **contract** your team and your agents follow while building the real app elsewhere.

### How to use it in your project

1. **Keep it separate.** Clone or submodule this repo beside your app, or copy only the stack folder you need. The playbook stays in something like `coding-playbook/`; your app stays in `backend/`, `frontend/`, etc.

2. **Pick one stack map.** Open the README for the stack you are building:
   - Backend → [`python-fastapi-backend/README.md`](python-fastapi-backend/README.md)
   - Frontend → [`nextjs-frontend/README.md`](nextjs-frontend/README.md)

3. **Read files in order when onboarding.** Each stack has files `01`–`16` (principles → structure → config → … → tests / security / performance). You do not need every file on day one; start with `01`, `02`, and `03`, then open the file that matches what you are building (auth, forms, workers, …).

4. **Use Extra only when the feature already exists.** Folders under `extra/` (multi-tenant, SSO, agents, search, …) add rules on top of `01`–`16`. Do not scaffold those shapes just because the doc exists.

5. **Adapt rules to your product.** These files are a starting map, not immutable law. When a rule conflicts with your stack, host, or shipped design, **change the playbook file** and leave a one-line reason in that file. That way the next person (or agent) reads the decision in git, not in chat history.

6. **Point agents at the playbook.** In Cursor (or similar), add a rule or `@` reference: load [`README.md`](README.md) first, then the stack README, then only the numbered file for the current task. Agents should not load all sixteen files or every Extra topic at once. For prompt templates, config pitfalls, and long-chat fixes: **[HOW-TO-PROMPT.md](HOW-TO-PROMPT.md)**.

### Typical workflows

| Goal | Where to start |
|------|----------------|
| New FastAPI service | `python-fastapi-backend/01` → `02` → `03`, then topic files |
| New Next.js app | `nextjs-frontend/01` (design) → `02` → `03`, then topic files |
| Code review checklist | `15-security.md` and `14-testing.md` for that stack |
| Add SSO / tenants / agents later | Matching file under `extra/` **after** the feature exists |

### What to ignore as a human reader

- `WHEN:` / `LOAD:` / `MUST NOT:` lines are routing instructions for agents. Skim them or read the prose sections underneath.
- Content below the line **“Agent instructions”** in [`README.md`](README.md) is the same agent-oriented index; use this user guide instead for onboarding.

---

## Türkçe

### Bu repo nedir?

**Coding Playbook**, iki yığın için mimari ve kodlama kurallarından oluşan bir **referans kütüphanesidir**:

| Yığın | Klasör |
|-------|--------|
| Python + FastAPI (API, worker'lar) | [`python-fastapi-backend/`](python-fastapi-backend/README.md) |
| Next.js (arayüz) | [`nextjs-frontend/`](nextjs-frontend/README.md) |

Bu klasör **uygulama kaynak kodu değildir**. Tümünü `src/` içine kopyalamayın. Gerçek uygulamayı başka yerde geliştirirken ekibinizin ve ajanlarınızın uyacağı bir **sözleşme** olarak kullanın.

### Projede nasıl kullanılır?

1. **Ayrı tutun.** Repoyu uygulamanızın yanına klonlayın veya submodule olarak ekleyin; yalnızca ihtiyacınız olan yığın klasörünü kopyalayabilirsiniz. Playbook `coding-playbook/` gibi kalır; uygulama `backend/`, `frontend/` vb. dizinlerde durur.

2. **Tek bir yığın haritası seçin.** Geliştirdiğiniz yığının README dosyasını açın:
   - Backend → [`python-fastapi-backend/README.md`](python-fastapi-backend/README.md)
   - Frontend → [`nextjs-frontend/README.md`](nextjs-frontend/README.md)

3. **Ekibe alırken sırayla okuyun.** Her yığında `01`–`16` numaralı dosyalar vardır (ilkeler → yapı → config → … → test / güvenlik / performans). İlk günde hepsine gerek yok; `01`, `02`, `03` ile başlayın, sonra yaptığınız işe uygun dosyayı açın (auth, form, worker, …).

4. **Extra yalnızca özellik zaten varsa.** `extra/` altındaki konular (multi-tenant, SSO, agent, arama, …) `01`–`16` kurallarına **ek** gelir. Doküman diye o yapıları boşuna kurmayın.

5. **Kuralları ürününüze uyarlayın.** Dosyalar başlangıç haritasıdır; değişmez kanun değildir. Bir kural yığınınız, host'unuz veya mevcut tasarımınızla çelişiyorsa **playbook dosyasını güncelleyin** ve dosyaya tek satırlık bir gerekçe bırakın. Böylece sonraki kişi (veya ajan) kararı sohbet geçmişinde değil, git'te okur.

6. **Ajanları playbook'a yönlendirin.** Cursor (veya benzeri) içinde kural veya `@` referansı verin: önce [`README.md`](README.md), sonra ilgili yığın README'si, en son yalnızca o görevle ilgili numaralı dosya. Ajanlar on altı dosyanın veya tüm Extra konularının tamamını aynı anda yüklememelidir. Prompt şablonları, config tuzakları ve uzun sohbet çözümleri: **[HOW-TO-PROMPT.md](HOW-TO-PROMPT.md)**.

### Sık senaryolar

| Amaç | Nereden başlanır |
|------|------------------|
| Yeni FastAPI servisi | `python-fastapi-backend/01` → `02` → `03`, ardından konu dosyaları |
| Yeni Next.js uygulaması | `nextjs-frontend/01` (tasarım) → `02` → `03`, ardından konu dosyaları |
| Code review kontrol listesi | İlgili yığında `15-security.md` ve `14-testing.md` |
| Sonradan SSO / tenant / agent | Özellik **vardıktan sonra** `extra/` altındaki eşleşen dosya |

### İnsan okuyucu olarak nelere takılmayın

- `WHEN:` / `LOAD:` / `MUST NOT:` satırları ajanlar için yönlendirme talimatıdır. Üstünden geçin veya altındaki açıklama bölümlerini okuyun.
- [`README.md`](README.md) içinde **“Agent instructions”** çizgisinin altı ajanlara yönelik indekstir; ekibe alırken bunun yerine bu kullanıcı rehberini kullanın.

---

## Quick links / Hızlı bağlantılar

- [Agent operations / Ajan operasyonları](agents/README.md)
- [How to prompt agents / Ajan prompt rehberi](HOW-TO-PROMPT.md) → human · [agents/02-turn.md](agents/02-turn.md) → agent
- [Vibe coding pitfalls / Vibe coding tuzakları](VIBE-CODING-PITFALLS.md)
- [50 common errors / 50 yaygın hata](VIBE-CODING-ERRORS.md)
- [Playbook root (agents) / Kök (ajanlar)](README.md)
- [FastAPI backend map](python-fastapi-backend/README.md)
- [Next.js frontend map](nextjs-frontend/README.md)
