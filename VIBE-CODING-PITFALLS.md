# Vibe Coding Pitfalls / Vibe Coding Tuzakları

> **For humans** building real software with AI coding tools and this playbook.  
> **İnsanlar için** — yapay zeka ile gerçek yazılım üreten ekipler.

Vibe coding is seductive: you describe something in plain language, working code appears, you describe a change, it happens. The loop from idea to execution shrinks to minutes.

Then something breaks and you do not know why.

The speed that makes it powerful is the same speed that makes it dangerous. Bad decisions happen faster. Shortcuts compound faster. Generated code often *looks* professional, so problems hide longer than in code you struggled to write yourself.

This is not a theoretical risk list. These are mistakes that consistently kill projects when teams use AI to build production software. All are fixable if you catch them early.

This playbook does not replace discipline — it **channels** it: small prompts, explicit layers, config and security rules, git as source of truth. Pair this page with [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) and [USER-GUIDE.md](USER-GUIDE.md). When something breaks, see [VIBE-CODING-ERRORS.md](VIBE-CODING-ERRORS.md) (50-error checklist).

**Agents:** [agents/03-anti-patterns.md](agents/03-anti-patterns.md)

Vibe coding cazibesiz değildir: düz dille anlatırsınız, çalışan kod çıkar, değişiklik istersiniz, olur. Fikirden uygulamaya döngü dakikalara iner.

Sonra bir şey bozulur ve nedenini bilmezsiniz.

Hız güç verdiği kadar tehlikelidir. Kötü kararlar daha hızlı alınır. Kısayollar daha hızlı birikir. Üretilen kod *profesyonel* göründüğü için sorunlar, kendiniz zor yazdığınız koddakinden daha geç ortaya çıkar.

Bu teorik bir risk listesi değil. Ekiplerin yapay zeka ile **gerçek** yazılım inşa ederken tekrar tekrar yaptığı hatalar. Erken yakalarsanız hepsi düzeltilebilir.

Playbook disiplinin yerini tutmaz — onu **yönlendirir**: küçük promptlar, açık katmanlar, config ve güvenlik kuralları, git kaynak gerçeği. Bu sayfayı [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) ve [USER-GUIDE.md](USER-GUIDE.md) ile birlikte kullanın.

---

## English

### 1. Prompting like you're texting a friend

**Bad:** “Build me a dashboard.”

Five words. The agent chooses metrics, layout, data source, and roles. You get something that looks like a dashboard but matches none of your real constraints — then spend more time undoing assumptions than you saved.

**Fix:** Specificity and playbook routing. Name the stack, files, entities, and done criteria. If the prompt could describe a thousand apps, it is too vague.

```text
Stack: nextjs-frontend
Task: Customer success dashboard — monthly churn, 12-month NPS trend, at-risk accounts by health score.
Playbook: 01-design + 10-features + 07-data + 09-api-client
Done when: four UI states (01); data from FastAPI /v1/… only; no client-side secrets
```

More templates: [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md).

---

### 2. Building everything in one giant prompt

**Bad:** One 500-word message with every feature, page, and edge case.

Past a complexity threshold the model makes silent tradeoffs: features merge, interactions drop, layers blur (HTTP logic in UI, DB access from Next.js).

**Fix:** Iterative building aligned with playbook slices.

1. Core workflow (one module / one feature package)
2. Settings, then payments, then admin — **separate prompts**
3. Each step: working checkpoint in git before the next prompt

Chain: model + migration → service + API → worker (if needed) → UI feature → tests. See [HOW-TO-PROMPT.md § Biggest pain points](HOW-TO-PROMPT.md).

---

### 3. Skipping version control

**Bad:** Accept AI diffs quickly with no commits. Feature X breaks; something unrelated regresses; you cannot return to last good state.

**Fix:** Commit early, commit often. Every prompt that produces a **working** result is a checkpoint. Before a risky prompt: commit. When playbook rules change to match a product decision, **commit the playbook file too** — the next agent reads git, not chat ([README.md](README.md) “Not carved in stone”).

---

### 4. Not reviewing generated code

**Bad:** Polished formatting creates false confidence. Research consistently shows: many AI solutions are functionally correct but a large share fail security review (auth, injection, secrets, validation).

**Fix:** Treat every agent output as a **draft**.

- Read the diff, especially auth, input validation, env handling, SQL/query construction
- Run [python-fastapi-backend/15-security.md](python-fastapi-backend/15-security.md) or [nextjs-frontend/15-security.md](nextjs-frontend/15-security.md) before merge
- If you cannot review code yourself, pair with someone who can or use automated security checks on PRs

Working code is not safe code.

---

### 5. Ignoring the data model

**Bad:** “Build a project management tool.” The agent invents entities and relations. You build six features on the wrong skeleton; fixing the model cascades through migrations, APIs, and UI.

**Fix:** Describe **data before features**.

- Entities, relations, constraints, status enums
- Backend: [06-database.md](python-fastapi-backend/06-database.md) + [07-migrations.md](python-fastapi-backend/07-migrations.md) in the **same** prompt as the model change
- Connecting to an existing DB: paste or `@` schema; do not let the agent guess table names

Example constraint block:

```text
Projects have many Tasks. Task → one assignee, many watchers.
Status: todo | in_progress | review | done. Priority: P0–P3.
```

---

### 6. Trusting the AI's security defaults

**Bad:** Generated code optimizes for “works on my machine.” Common failures: hardcoded secrets, missing validation, SQL injection, secrets in client bundles (`NEXT_PUBLIC_`), permissive authz.

**Fix:** Security rules live in the playbook **independently** of whatever the agent wrote today.

| Area | Playbook |
|------|----------|
| Backend secrets, authz, JWT | [13-identity-security.md](python-fastapi-backend/13-identity-security.md), [15-security.md](python-fastapi-backend/15-security.md) |
| Frontend cookies, XSS, env | [12-auth.md](nextjs-frontend/12-auth.md), [15-security.md](nextjs-frontend/15-security.md) |
| Config (no leaked keys) | [03-config.md](python-fastapi-backend/03-config.md), [04-config.md](nextjs-frontend/04-config.md) |

Prompt explicitly: “No secrets in client code; all limits in Settings / lib/env.ts; authz stays on FastAPI.”

---

### 7. Building too much before validating with users

**Bad:** Vibe coding removes the old cost of building the wrong thing. You ship eight features in a day; users only needed the first one, differently.

**Fix:** Minimum useful workflow first. Real users, then the next prompt. Playbook Extra topics (SSO, multi-tenant, agents, search) only after the shape is **shipped and validated** — not because the doc exists ([USER-GUIDE.md](USER-GUIDE.md)).

---

### 8. Context window overflow

**Bad:** Long threads “forget” requirements. Login breaks though you did not touch auth. Random regressions.

**Fix:** Fresh chat per module or feature slice. Re-attach: short decision summary (3 lines) + minimal playbook `@` files + current app paths. Do not rebuild the whole app in one thread.

| Tooling | Helps | Does not eliminate |
|---------|-------|---------------------|
| Codebase indexing (Cursor, etc.) | Finds files without pasting everything | Long chat drift |
| Playbook `LOAD:` routing | Limits rules per task | Mega-prompts |
| Decision doc in repo | Paste into new chats | Need for new threads |

Full playbook: [HOW-TO-PROMPT.md § Context bloat](HOW-TO-PROMPT.md).

---

### 9. No error handling or edge cases

**Bad:** Happy path only — full forms, APIs always 200, DB always returns rows. Production: timeouts, empty results, validation failures.

**Fix:** After happy path works, **second prompt** for defense.

```text
Task: Error paths for [feature] — API 4xx/5xx, empty list, network timeout, invalid form fields.
Playbook: backend 05-errors + 12-api OR frontend 01-design (four states) + 08-forms
Done when: user-visible message; no silent failure; logged server-side with request_id
```

Backend error JSON: [05-errors.md](python-fastapi-backend/05-errors.md), [12-api.md](python-fastapi-backend/12-api.md). UI states: [01-design.md](nextjs-frontend/01-design.md).

---

### 10. Treating AI output as finished code

**Bad:** The meta-mistake. Deploy without review, scale without tests, extend without understanding. Looks good until it does not.

**Fix:** Match generation speed with review discipline.

```
AI writes → you review diff → you test → AI fixes → commit checkpoint → next slice
```

Playbook = contract for **both** humans and agents. When reality diverges, update the playbook file with a one-line reason — do not leave chat as the only source of truth.

---

### Quick checklist before you ship

- [ ] Prompt was specific (stack, files, playbook files, done when)
- [ ] Git checkpoint before and after the change
- [ ] Diff reviewed; security files checked for this stack
- [ ] Data model explicit; migrations if backend changed
- [ ] No secrets in frontend env; config in Settings / `lib/env.ts`
- [ ] Error and empty states handled
- [ ] Not “everything in one thread”

---

## Türkçe

### 1. Arkadaşa mesaj atar gibi prompt yazmak

**Kötü:** “Bana bir dashboard yap.”

Beş kelime. Ajan metrik, layout, veri kaynağı ve rolü kendisi seçer. Dashboard *gibi* görünen ama ihtiyacınıza uymayan bir şey çıkar — kazandığınız zamandan fazlasını varsayımları geri almakla harcarsınız.

**Çözüm:** Netlik ve playbook yönlendirmesi. Yığın, dosyalar, varlıklar ve bitti kriterlerini yazın. Prompt bin farklı uygulamayı tarif edebiliyorsa çok belirsizdir.

Şablonlar: [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md).

---

### 2. Her şeyi tek dev promptta yapmak

**Kötü:** Tek mesajda tüm özellikler, sayfalar ve edge case'ler.

Belirli bir eşiğin ötesinde model sessiz ödünler verir: özellikler birleşir, etkileşimler düşer, katmanlar karışır (HTTP mantığı UI'da, Next.js'ten DB).

**Çözüm:** Playbook dilimleriyle iteratif inşa.

1. Çekirdek akış (tek modül / tek feature paketi)
2. Ayarlar, ödeme, admin — **ayrı promptlar**
3. Her adım: sonraki prompttan önce git'te çalışan checkpoint

Zincir: model + migration → service + API → worker (gerekirse) → UI feature → test. Bkz. [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md).

---

### 3. Versiyon kontrolünü atlamak

**Kötü:** Commit yok; X bozulur, alakasız bir şey regrese olur; son iyi sürüme dönemezsiniz.

**Çözüm:** Erken ve sık commit. **Çalışan** her agent çıktısı checkpoint. Riskli prompt öncesi commit. Playbook kuralı ürüne uyarlandıysa **playbook dosyasını da commit edin** — sonraki ajan sohbeti değil git'i okur ([README.md](README.md)).

---

### 4. Üretilen kodu incelememek

**Kötü:** Düzgün format yanlış güven verir. Araştırmalar: birçok çözüm işlevsel ama güvenlik incelemesinde (auth, injection, secret, validation) yüksek oranda sorunlu.

**Çözüm:** Her agent çıktısını **taslak** sayın.

- Diff'i okuyun; özellikle auth, validation, env, SQL/sorgu
- Merge öncesi ilgili yığının [15-security.md](python-fastapi-backend/15-security.md) dosyası
- Kod inceleyemiyorsanız: deneyimli biri veya otomatik PR kontrolleri

Çalışan kod, güvenli kod değildir.

---

### 5. Veri modelini görmezden gelmek

**Kötü:** “Proje yönetim aracı yap.” Ajan varlıkları uydurur. Altı özellik yanlış iskelet üzerine; model düzeltmesi migration, API ve UI'ya yayılır.

**Çözüm:** **Özelliklerden önce veriyi** tarif edin.

- Varlıklar, ilişkiler, kısıtlar, durum enum'ları
- Backend: [06-database.md](python-fastapi-backend/06-database.md) + [07-migrations.md](python-fastapi-backend/07-migrations.md) model değişikliğiyle **aynı** promptta
- Mevcut DB: şemayı yapıştırın veya `@` ile verin; tablo adı tahmin ettirmeyin

---

### 6. Yapay zekanın güvenlik varsayımlarına güvenmek

**Kötü:** “Benim makinemde çalışıyor” optimizasyonu. Sık hatalar: gömülü secret, eksik validation, SQL injection, istemci bundle'ında secret (`NEXT_PUBLIC_`), gevşek yetkilendirme.

**Çözüm:** Güvenlik kuralları playbook'ta, agent'ın o gün yazdığı koddan **bağımsız**.

Prompt'a açıkça: “İstemci kodunda secret yok; tüm limitler Settings / lib/env.ts'te; yetkilendirme FastAPI'de.”

---

### 7. Kullanıcı doğrulamadan çok fazla inşa etmek

**Kötü:** Vibe coding yanlış şeyi inşa etmenin maliyetini düşürür. Bir günde sekiz özellik; kullanıcı yalnızca birincisine, farklı şekilde ihtiyaç duyuyordu.

**Çözüm:** Önce minimum faydalı akış. Gerçek kullanıcı, sonra sonraki prompt. Extra (SSO, tenant, agent, arama) yalnızca şekil **doğrulandıktan ve vardıktan sonra** — doküman diye değil ([USER-GUIDE.md](USER-GUIDE.md)).

---

### 8. Bağlam penceresinin taşması

**Kötü:** Uzun sohbetler gereksinimleri “unutur”. Auth'a dokunmadınız ama login bozuldu. Rastgele regresyonlar.

**Çözüm:** Modül veya özellik dilimi başına **yeni sohbet**. Yeniden ekleyin: kısa karar özeti (3 satır) + minimal playbook `@` + güncel app yolları. Tüm uygulamayı tek thread'de kurmayın.

Ayrıntı: [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md).

---

### 9. Hata yönetimi ve edge case yokluğu

**Kötü:** Yalnızca mutlu yol — dolu formlar, API hep 200, DB hep satır döner.

**Çözüm:** Mutlu yol çalıştıktan sonra **ikinci prompt** savunma için.

Backend: [05-errors.md](python-fastapi-backend/05-errors.md), [12-api.md](python-fastapi-backend/12-api.md). UI dört durum: [01-design.md](nextjs-frontend/01-design.md).

---

### 10. Yapay zeka çıktısını bitmiş kod sanmak

**Kötü:** Üst hata. İncelemeden deploy, testsiz ölçekleme, anlamadan genişletme. Kötü olana kadar iyi görünür.

**Çözüm:** Üretim hızını inceleme disipliniyle eşleştirin.

```
AI yazar → diff incelersiniz → test → AI düzeltir → commit checkpoint → sonraki dilim
```

Playbook hem insan hem ajan sözleşmesidir. Gerçeklik saparsa playbook dosyasını tek satır gerekçeyle güncelleyin — tek kaynak sohbet olmasın.

---

### Gönderim öncesi kısa checklist

- [ ] Prompt net (yığın, dosyalar, playbook, bitti sayılır)
- [ ] Değişiklik öncesi/sonrası git checkpoint
- [ ] Diff incelendi; ilgili güvenlik dosyası
- [ ] Veri modeli açık; backend değiştiyse migration
- [ ] Frontend'te secret yok; config Settings / lib/env.ts'te
- [ ] Hata ve boş durumlar ele alındı
- [ ] “Her şey tek sohbette” değil

---

## Related / İlgili

- [USER-GUIDE.md](USER-GUIDE.md) — repo setup
- [HOW-TO-PROMPT.md](HOW-TO-PROMPT.md) — prompt templates and context fixes
- [README.md](README.md) — agent routing (for `@` references)
