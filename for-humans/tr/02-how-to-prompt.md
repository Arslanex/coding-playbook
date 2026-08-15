# Kodlama ajanlarına nasıl prompt yazılır

> **İnsanlar için** — Cursor, Copilot, Claude Code veya benzeri araçlara prompt yazan herkes.

Bu bir kural dosyası değildir. **Size**, ajanın küçük, doğru ve hızlı kalması için görevi nasıl ifade edeceğinizi öğretir. Ajanın kendi sürümü [agents/02-turn.md](../../agents/02-turn.md) — bu sayfayı ona göndermeyin.

**English:** [02-how-to-prompt.md](../en/02-how-to-prompt.md) · **Dizin:** [for-humans/](../README.md) · **Ajanlar şunu okur:** [agents/README.md](../../agents/README.md)

---

## Tek cümle kural

**Ne değişeceğini, nerede olduğunu, hangi playbook dosyalarının geçerli olduğunu ve “bitti” tanımını yazın — sonra durun.**

Playbook'un tamamını sohbete yapıştırmayın. Tek promptta “her şeyi” istemeyin.

---

## Cursor / IDE kurulumu (bir kez)

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

## Prompt şablonu (kopyala, doldur)

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

## Config yazarken (en sık tekrarlanan hatalar)

Config prompt'ları belirsiz olunca bozulur. Açık olun:

| Bunu yazın | Bunu değil |
|------------|------------|
| “`X` alanını Settings / `lib/env.ts`'e ekle” | “Config'i ayarla” |
| “Yalnızca operasyonel limit; ürün kuralı modules/'da kalır” | “Constants dosyası ekle” |
| “`.env.example`'ı yalnızca isimlerle güncelle” | “Env'i düzelt” |
| “Backend: pydantic Settings frozen=True” | “Env kullan” |
| “Frontend: server vs NEXT_PUBLIC_ — tarayıcı Y'yi görmesin” | “API key'i frontend'e ekle” |

Backend: [python-fastapi-backend/03-config.md](../../python-fastapi-backend/03-config.md)  
Frontend: [nextjs-frontend/04-config.md](../../nextjs-frontend/04-config.md)

**Backend ve frontend config'i ayrı promptlarda bölün.** Tek turda iki yığın, yükleme hatası ve tekrarlayan açıklamayı ikiye katlar.

---

## Görev türüne göre prompt

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

## En büyük sıkıntılar (ve çözümler)

### 1. Playbook'u her mesajda tekrar etmek

**Belirti:** MUST/MUST NOT bloklarını yapıştırıyorsunuz; ajan yine sapıyor.  
**Çözüm:** Tur başına bir `@README.md` + bir numaralı dosya. Sabit kuralları **proje kurallarına** koyun, sohbete değil. “Yüklenen playbook dosyalarına uy; cevabında tekrar etme” deyin.

### 2. Uzun sohbetin sonunda bağlam şişmesi

**Belirti:** İlk turlar iyiydi; sonradan kısıtları unutuyor, dosya çoğaltıyor veya çalışan kodu “düzeltiyor”.  
**Çözüm:** Dilim başına **yeni sohbet** (bir modül, bir migration, bir ekran). Yalnızca: görev + 2–3 playbook + güncel app dosyaları. Önceki kararı **üç satırda** özetleyin, tüm geçmişi değil.

### 3. Her şeyi yüklemek

**Belirti:** Ajan 16 dosyanın veya tüm `extra/`'nın tamamını okuyor; yavaş, çelişkili, aşırı iskelet.  
**Çözüm:** Promptta **tek** numaralı dosya adı geçsin. Açıkça: “Bu dosyanın LOAD: satırı gerektirmedikçe diğer numaralı dosyaları yükleme.”

### 4. Extra konularını erken açmak

**Belirti:** Ürün henüz ihtiyaç duymadan `src/agents/`, i18n, outbox tabloları.  
**Çözüm:** “[Özellik] bu repoda **zaten yoksa** extra/ yükleme veya iskelet kurma.” Extra yalnızca **var olan** şekiller içindir.

### 5. `src/` içinde paralel yapı

**Belirti:** `utils/`, `helpers/`, `modules/` yanında `services/`; veya FastAPI ağacının Next.js'e kopyalanması.  
**Çözüm:** “Dosyalar playbook 02/03 file-structure'a göre. Repomuz farklıysa playbook dosyasını güncelle ve nedenini yaz — sessizce ikinci ağaç icat etme.”

### 6. Config dağılması

**Belirti:** Modüllerde `os.getenv`, repository'de sihirli sayılar, gizli anahtar `NEXT_PUBLIC_`'te.  
**Çözüm:** Ayrı config prompt'u (örnekler yukarıda). Bitti kriterinde mutlaka “config/ veya lib/env.ts dışında env okuması yok” olsun.

### 7. Tek promptta “tüm özellik”

**Belirti:** Dev diff, eksik test, yanlış katman sınırları.  
**Çözüm:** Zincir prompt: (1) model + migration → (2) service + API → (3) gerekirse worker → (4) UI feature → (5) test. Her adımda **bitti sayılır** yazın.

### 8. Playbook ile sohbet çelişkisi

**Belirti:** Ekip Slack'te X dedi; playbook hâlâ Y; sonraki ajan X'i geri alıyor.  
**Çözüm:** “Kural Z'yi uyarladık — önce `python-fastapi-backend/NN-….md` SCOPE'a tek satır gerekçe, sonra implementasyon.” Kararlar git'te, sohbette değil.

### 9. Kodlama ajanı ile ürün içi LLM ajanını karıştırmak

**Belirti:** Cursor'un kod düzenlemesi ile üründeki `AgentRun` / transcript UI karışıyor.  
**Çözüm:** Kodlama ajanları `01`–`16` kullanır. Ürün içi ajanlar yalnızca **ürününüz** LLM job çalıştırıyorsa [extra/03-agent-teams](../../python-fastapi-backend/extra/03-agent-teams.md) — promptta hangisini kastettiğinizi yazın.

### 10. Doğrulanabilir “bitti” yok

**Belirti:** Ajan yarıda bırakıyor veya tamamlanma tartışıyor.  
**Çözüm:** Her zaman **Bitti sayılır** checklist'i ile bitirin (test, dokunulan dosyalar, env isimleri, yeni klasör yok).

---

## Kötü vs iyi prompt (aynı görev)

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

**Diğer rehberler:** [01 Buradan başlayın](01-start-here.md) · [02 Prompt yazımı](02-how-to-prompt.md) · [03 Ajan kodunu inceleme](03-review-agent-code.md) · [04 Tuzaklar](04-pitfalls.md) · [05 Hatalar](05-errors.md) · [06 Sözlük](06-glossary.md)
