# How to Prompt Coding Agents / Kodlama Ajanlarına Nasıl Prompt Yazılır

> **For humans** who write prompts to Cursor, Copilot, Claude Code, or similar tools while using this playbook.  
> **İnsanlar için** — bu playbook ile Cursor ve benzeri araçlara prompt yazanlar.

This is **not** a playbook rule file. It does not use `WHEN` / `LOAD` for agents to parse. It teaches **you** how to phrase tasks so agents stay small, correct, and fast.

**Agents:** load [agents/02-turn.md](agents/02-turn.md) and [agents/03-anti-patterns.md](agents/03-anti-patterns.md) instead of this file.

Bu dosya playbook kural dosyası değildir. Ajanlar [agents/02-turn.md](agents/02-turn.md) dosyasını yüklemelidir.

Related: [USER-GUIDE.md](USER-GUIDE.md) (repo setup) · [VIBE-CODING-PITFALLS.md](VIBE-CODING-PITFALLS.md) (speed vs discipline) · [VIBE-CODING-ERRORS.md](VIBE-CODING-ERRORS.md) (50-error checklist) · [README.md](README.md) (agent routing index)

---

## English

### One sentence rule

**Say what to change, where it lives, which playbook files apply, and what “done” looks like — then stop.**

Do not paste the whole playbook into chat. Do not ask for “everything” in one prompt.

---

### Cursor / IDE setup (do this once)

Put the playbook where the agent can `@`-reference it (submodule, sibling folder, or monorepo path).

**Project rule (recommended)** — add to `.cursor/rules` or your IDE’s project instructions:

```text
Application code: follow coding-playbook/README.md routing.
Load only the stack README + the numbered file for this task (+ that file's LOAD: siblings).
Do not load all 01–16 or every extra/ topic.
If a playbook rule conflicts with this repo, adapt the playbook file and commit the reason — do not invent a parallel layout in src/.
Minimize scope: one task per agent turn when possible.
```

**Per chat:** `@coding-playbook/README.md` plus the stack README and **one** topic file (e.g. `@python-fastapi-backend/03-config.md`). Attach the **application** files being edited, not the entire repo.

| Setup | Good | Bad |
|-------|------|-----|
| Rules | Short pointer to playbook + “load minimal files” | Pasting all of `01`–`16` into rules |
| Context | Task file + 1–3 playbook files | `@` entire `coding-playbook/` folder |
| Chat length | New chat per feature slice | One chat for “build backend + frontend + deploy” |

---

### Prompt shape (copy and fill)

```text
Stack: [python-fastapi-backend | nextjs-frontend]
Task: [one sentence — what changes]
Files: [paths in the APPLICATION repo you are editing]
Playbook: load [stack README] + [numbered file, e.g. 03-config]
Constraints:
  - [optional: must not touch X, keep diff small, no new deps]
Done when:
  - [observable checklist — tests pass, env in Settings/env.ts, etc.]
```

**Example — backend config**

```text
Stack: python-fastapi-backend
Task: Add REDIS_URL and redis socket timeout to Settings; wire into infra/cache client.
Files: backend/src/config/settings.py, backend/src/infra/cache/redis.py
Playbook: load python-fastapi-backend/README.md + 03-config.md (+ 08-infra if you add a new infra folder)
Constraints:
  - No os.getenv outside config/
  - No magic numbers — timeout is a Settings field
Done when:
  - .env.example lists REDIS_URL and REDIS_SOCKET_TIMEOUT
  - get_settings() used in redis client
  - Existing tests still pass
```

**Example — frontend env**

```text
Stack: nextjs-frontend
Task: Add server-side API_BASE_URL to lib/env.ts and use it in the fetch wrapper.
Files: frontend/lib/env.ts, frontend/lib/api/client.ts
Playbook: load nextjs-frontend/README.md + 04-config.md + 09-api-client.md
Constraints:
  - No process.env outside lib/env.ts
  - No NEXT_PUBLIC_ for secrets
Done when:
  - Zod parse for API_BASE_URL
  - client.ts imports serverEnv only from lib/env.ts
```

---

### While writing config (most repeated mistakes)

Config prompts fail when the task is vague. Be explicit:

| Say this | Not this |
|----------|----------|
| “Add field `X` to Settings / `lib/env.ts`” | “Set up config” |
| “Operational limit only; product rule stays in modules/” | “Add constants file” |
| “Update `.env.example` names only” | “Fix env” |
| “Backend: pydantic Settings frozen=True” | “Use env vars” |
| “Frontend: server vs NEXT_PUBLIC_ — browser must not see Y” | “Add API key to frontend” |

Backend playbook: [python-fastapi-backend/03-config.md](python-fastapi-backend/03-config.md)  
Frontend playbook: [nextjs-frontend/04-config.md](nextjs-frontend/04-config.md)

**Split backend vs frontend config in separate prompts.** One agent turn touching both stacks doubles load errors and repeated explanations.

---

### Prompts by task type

| You want | Playbook files to `@` | Prompt tip |
|----------|----------------------|------------|
| New API endpoint | `09-modules`, `10-http`, `12-api` | Name the module noun; say “router mounts in http/router.py only” |
| DB column + migration | `06-database`, `07-migrations` | Model change and Alembic revision in **same** prompt |
| Background job | `11-workers`, relevant module | “Never BackgroundTasks; queue + worker job file” |
| New screen | `01-design`, `03-file-structure`, `10-features` | UI path under `features/<noun>/`, not `app/` city |
| Auth / cookies | backend `13-identity-security`, frontend `12-auth` | Two prompts if both stacks; cite cookie/JWT split |
| PR / security pass | `15-security` (+ `14-testing`) | “Review diff against checklist; do not refactor unrelated” |

Open a file’s `RELATED:` line **only** if your task is also that topic — do not preload siblings “just in case.”

---

### Biggest pain points (and fixes)

#### 1. Repeating the playbook in every message

**Symptom:** You re-paste MUST/MUST NOT blocks; the agent still drifts.  
**Fix:** One `@README.md` + one numbered file per turn. Put standing rules in **project rules**, not chat. Say: “Follow loaded playbook files; do not restate them in your reply.”

#### 2. Context bloat near the end of a long chat

**Symptom:** Early turns were good; later turns ignore constraints, duplicate files, or “fix” working code.  
**Fix:** **New chat** per slice (one module, one migration, one screen). Re-attach only: task description + 2–3 playbook files + current app files. Summarize prior decision in **three lines**, not full thread history.

#### 3. Loading everything

**Symptom:** Agent reads all 16 files or all of `extra/`; slow, contradictory, over-scaffolded.  
**Fix:** Prompt must name **one** numbered file. Explicit: “Do not load other numbered files unless this file’s LOAD: line requires them.”

#### 4. Extra topics too early

**Symptom:** `src/agents/`, `extra/i18n`, outbox tables appear before the product needs them.  
**Fix:** “Do not load or scaffold extra/ unless [feature] already exists in this repo.” Extra is for **shipped** shapes only.

#### 5. Parallel layout in `src/`

**Symptom:** `utils/`, `helpers/`, `services/` next to `modules/`, or FastAPI `http/modules/infra` copied into Next.js.  
**Fix:** “Place files per playbook 02/03 file-structure. If our repo differs, update the playbook file and say why — do not silently invent a second tree.”

#### 6. Config sprawl

**Symptom:** `os.getenv` in modules, magic numbers in repositories, secrets on `NEXT_PUBLIC_`.  
**Fix:** Dedicated config prompt (see examples). Done criteria must include “no env reads outside config/ or lib/env.ts”.

#### 7. “Build the whole feature” in one prompt

**Symptom:** Huge diff, missed tests, wrong layer boundaries.  
**Fix:** Chain prompts: (1) model + migration → (2) service + API → (3) worker if needed → (4) UI feature package → (5) tests. Each step names **done when**.

#### 8. Playbook vs chat disagreement

**Symptom:** Team decided X in Slack; playbook still says Y; next agent reverts X.  
**Fix:** Prompt: “We adapted rule Z — update `python-fastapi-backend/NN-….md` SCOPE with one-line reason, then implement.” Decisions live in git, not chat.

#### 9. Mixing coding agents with in-product LLM agents

**Symptom:** Confusion between Cursor editing code vs `AgentRun` / transcript UI in the product.  
**Fix:** Coding agents use `01`–`16`. In-product agents use [extra/03-agent-teams](python-fastapi-backend/extra/03-agent-teams.md) only when **your product** runs LLM jobs — say which you mean in the prompt.

#### 10. No verifiable “done”

**Symptom:** Agent stops mid-task or argues about completeness.  
**Fix:** Always end with a **Done when** checklist (tests, files touched, env names, no new folders).

---

### Anti-patterns (short)

- “Follow best practices” without stack or file paths  
- “Read all playbook files first”  
- “Refactor while you’re here” without scope  
- Pasting 200-line diffs back into chat for “continue”  
- Asking the agent to copy `coding-playbook/` into `src/`  
- Re-explaining architecture the numbered file already states  

---

### Good vs weak prompt (same task)

**Weak**

```text
Add Redis to the project and configure everything properly.
```

**Strong**

```text
Stack: python-fastapi-backend
Task: Add Redis client in infra/cache; settings fields REDIS_URL and REDIS_SOCKET_TIMEOUT.
Files: backend/src/config/settings.py, backend/src/infra/cache/redis.py, backend/.env.example
Playbook: README.md + python-fastapi-backend/03-config.md + 08-infra.md
Done when: no os.getenv outside config/; .env.example updated; unit test mocks redis get/set
Do not: add workers, caching policy, or frontend changes in this turn.
```

---

## Türkçe

### Tek cümle kural

**Ne değişeceğini, nerede olduğunu, hangi playbook dosyalarının geçerli olduğunu ve “bitti” tanımını yazın — sonra durun.**

Playbook'un tamamını sohbete yapıştırmayın. Tek promptta “her şeyi” istemeyin.

---

### Cursor / IDE kurulumu (bir kez)

Playbook'u ajanın `@` ile referans verebileceği yerde tutun (submodule, yan klasör veya monorepo yolu).

**Proje kuralı (önerilir)** — `.cursor/rules` veya IDE proje talimatlarına:

```text
Uygulama kodu: coding-playbook/README.md yönlendirmesine uy.
Yalnızca ilgili yığın README + bu görevin numaralı dosyasını (+ o dosyanın LOAD: satırındaki kardeşleri) yükle.
01–16'nın tamamını veya tüm extra/ konularını yükleme.
Playbook kuralı bu repoyla çelişirse playbook dosyasını uyarlayıp gerekçeyi commit et — src/ içinde paralel yapı icat etme.
Kapsamı küçük tut: mümkünse ajan turu başına tek görev.
```

**Sohbet başına:** `@coding-playbook/README.md` + yığın README + **tek** konu dosyası (ör. `@python-fastapi-backend/03-config.md`). Düzenlenen **uygulama** dosyalarını ekleyin, tüm repoyu değil.

| Kurulum | İyi | Kötü |
|---------|-----|------|
| Kurallar | Playbook'a kısa işaret + “minimal dosya yükle” | `01`–`16`'nın tamamını kurallara yapıştırmak |
| Bağlam | Görev dosyası + 1–3 playbook dosyası | `@coding-playbook/` klasörünün tamamı |
| Sohbet uzunluğu | Özellik dilimi başına yeni sohbet | “Backend + frontend + deploy” tek sohbette |

---

### Prompt şablonu (kopyala, doldur)

```text
Yığın: [python-fastapi-backend | nextjs-frontend]
Görev: [tek cümle — ne değişecek]
Dosyalar: [APPLICATION reposundaki yollar]
Playbook: [yığın README] + [numaralı dosya, örn. 03-config] yükle
Kısıtlar:
  - [isteğe bağlı: X'e dokunma, diff küçük kalsın, yeni bağımlılık yok]
Bitti sayılır:
  - [gözlemlenebilir checklist — testler geçer, env Settings/env.ts'te, vb.]
```

**Örnek — backend config**

```text
Yığın: python-fastapi-backend
Görev: REDIS_URL ve redis socket timeout'u Settings'e ekle; infra/cache client'a bağla.
Dosyalar: backend/src/config/settings.py, backend/src/infra/cache/redis.py
Playbook: python-fastapi-backend/README.md + 03-config.md (+ yeni infra klasörü eklersen 08-infra)
Kısıtlar:
  - config/ dışında os.getenv yok
  - Sihirli sayı yok — timeout Settings alanı
Bitti sayılır:
  - .env.example'da REDIS_URL ve REDIS_SOCKET_TIMEOUT
  - redis client get_settings() kullanıyor
  - Mevcut testler geçiyor
```

**Örnek — frontend env**

```text
Yığın: nextjs-frontend
Görev: lib/env.ts'e sunucu tarafı API_BASE_URL ekle; fetch wrapper'da kullan.
Dosyalar: frontend/lib/env.ts, frontend/lib/api/client.ts
Playbook: nextjs-frontend/README.md + 04-config.md + 09-api-client.md
Kısıtlar:
  - lib/env.ts dışında process.env yok
  - Gizli değerler için NEXT_PUBLIC_ yok
Bitti sayılır:
  - API_BASE_URL için Zod parse
  - client.ts yalnızca lib/env.ts'ten serverEnv import ediyor
```

---

### Config yazarken (en sık tekrarlanan hatalar)

Config prompt'ları belirsiz olunca bozulur. Açık olun:

| Bunu yazın | Bunu değil |
|------------|------------|
| “`X` alanını Settings / `lib/env.ts`'e ekle” | “Config'i ayarla” |
| “Yalnızca operasyonel limit; ürün kuralı modules/'da kalır” | “Constants dosyası ekle” |
| “`.env.example`'ı yalnızca isimlerle güncelle” | “Env'i düzelt” |
| “Backend: pydantic Settings frozen=True” | “Env kullan” |
| “Frontend: server vs NEXT_PUBLIC_ — tarayıcı Y'yi görmesin” | “API key'i frontend'e ekle” |

Backend: [python-fastapi-backend/03-config.md](python-fastapi-backend/03-config.md)  
Frontend: [nextjs-frontend/04-config.md](nextjs-frontend/04-config.md)

**Backend ve frontend config'i ayrı promptlarda bölün.** Tek turda iki yığın, yükleme hatası ve tekrarlayan açıklamayı ikiye katlar.

---

### Görev türüne göre prompt

| İstediğiniz | `@` playbook dosyaları | İpucu |
|-------------|------------------------|-------|
| Yeni API endpoint | `09-modules`, `10-http`, `12-api` | Modül ismini söyleyin; “router yalnızca http/router.py'de mount” |
| DB kolon + migration | `06-database`, `07-migrations` | Model ve Alembic revision **aynı** promptta |
| Arka plan işi | `11-workers`, ilgili modül | “BackgroundTasks yok; kuyruk + worker job dosyası” |
| Yeni ekran | `01-design`, `03-file-structure`, `10-features` | Yol `features/<isim>/` altında, `app/` şehri değil |
| Auth / cookie | backend `13`, frontend `12` | İki yığın varsa iki prompt; cookie/JWT ayrımını yazın |
| PR / güvenlik | `15-security` (+ `14-testing`) | “Checklist'e göre diff incele; alakasız refactor yok” |

Bir dosyanın `RELATED:` satırını **yalnızca** görev gerçekten o konuyu da kapsıyorsa açın — “ileride lazım olur” diye kardeş dosyaları önceden yüklemeyin.

---

### En büyük sıkıntılar (ve çözümler)

#### 1. Playbook'u her mesajda tekrar etmek

**Belirti:** MUST/MUST NOT bloklarını yapıştırıyorsunuz; ajan yine sapıyor.  
**Çözüm:** Tur başına bir `@README.md` + bir numaralı dosya. Sabit kuralları **proje kurallarına** koyun, sohbete değil. “Yüklenen playbook dosyalarına uy; cevabında tekrar etme” deyin.

#### 2. Uzun sohbetin sonunda bağlam şişmesi

**Belirti:** İlk turlar iyiydi; sonradan kısıtları unutuyor, dosya çoğaltıyor veya çalışan kodu “düzeltiyor”.  
**Çözüm:** Dilim başına **yeni sohbet** (bir modül, bir migration, bir ekran). Yalnızca: görev + 2–3 playbook + güncel app dosyaları. Önceki kararı **üç satırda** özetleyin, tüm geçmişi değil.

#### 3. Her şeyi yüklemek

**Belirti:** Ajan 16 dosyanın veya tüm `extra/`'nın tamamını okuyor; yavaş, çelişkili, aşırı iskelet.  
**Çözüm:** Promptta **tek** numaralı dosya adı geçsin. Açıkça: “Bu dosyanın LOAD: satırı gerektirmedikçe diğer numaralı dosyaları yükleme.”

#### 4. Extra konularını erken açmak

**Belirti:** Ürün henüz ihtiyaç duymadan `src/agents/`, i18n, outbox tabloları.  
**Çözüm:** “[Özellik] bu repoda **zaten yoksa** extra/ yükleme veya iskelet kurma.” Extra yalnızca **var olan** şekiller içindir.

#### 5. `src/` içinde paralel yapı

**Belirti:** `utils/`, `helpers/`, `modules/` yanında `services/`; veya FastAPI ağacının Next.js'e kopyalanması.  
**Çözüm:** “Dosyalar playbook 02/03 file-structure'a göre. Repomuz farklıysa playbook dosyasını güncelle ve nedenini yaz — sessizce ikinci ağaç icat etme.”

#### 6. Config dağılması

**Belirti:** Modüllerde `os.getenv`, repository'de sihirli sayılar, gizli anahtar `NEXT_PUBLIC_`'te.  
**Çözüm:** Ayrı config prompt'u (örnekler yukarıda). Bitti kriterinde mutlaka “config/ veya lib/env.ts dışında env okuması yok” olsun.

#### 7. Tek promptta “tüm özellik”

**Belirti:** Dev diff, eksik test, yanlış katman sınırları.  
**Çözüm:** Zincir prompt: (1) model + migration → (2) service + API → (3) gerekirse worker → (4) UI feature → (5) test. Her adımda **bitti sayılır** yazın.

#### 8. Playbook ile sohbet çelişkisi

**Belirti:** Ekip Slack'te X dedi; playbook hâlâ Y; sonraki ajan X'i geri alıyor.  
**Çözüm:** “Kural Z'yi uyarladık — önce `python-fastapi-backend/NN-….md` SCOPE'a tek satır gerekçe, sonra implementasyon.” Kararlar git'te, sohbette değil.

#### 9. Kodlama ajanı ile ürün içi LLM ajanını karıştırmak

**Belirti:** Cursor'un kod düzenlemesi ile üründeki `AgentRun` / transcript UI karışıyor.  
**Çözüm:** Kodlama ajanları `01`–`16` kullanır. Ürün içi ajanlar yalnızca **ürününüz** LLM job çalıştırıyorsa [extra/03-agent-teams](python-fastapi-backend/extra/03-agent-teams.md) — promptta hangisini kastettiğinizi yazın.

#### 10. Doğrulanabilir “bitti” yok

**Belirti:** Ajan yarıda bırakıyor veya tamamlanma tartışıyor.  
**Çözüm:** Her zaman **Bitti sayılır** checklist'i ile bitirin (test, dokunulan dosyalar, env isimleri, yeni klasör yok).

---

### Kötü vs iyi prompt (aynı görev)

**Zayıf**

```text
Projeye Redis ekle ve her şeyi düzgün configure et.
```

**Güçlü**

```text
Yığın: python-fastapi-backend
Görev: infra/cache'te Redis client; Settings'te REDIS_URL ve REDIS_SOCKET_TIMEOUT.
Dosyalar: backend/src/config/settings.py, backend/src/infra/cache/redis.py, backend/.env.example
Playbook: README.md + python-fastapi-backend/03-config.md + 08-infra.md
Bitti sayılır: config/ dışında os.getenv yok; .env.example güncel; redis get/set için unit test mock
Bu turda: worker, cache policy veya frontend değişikliği yok.
```

---

## Quick links / Hızlı bağlantılar

- [User guide / Kullanıcı rehberi](USER-GUIDE.md)
- [Agent index / Ajan indeksi](README.md)
- [Backend config](python-fastapi-backend/03-config.md)
- [Frontend config](nextjs-frontend/04-config.md)
