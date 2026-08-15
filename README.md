# Coding Playbook

Architecture and coding rules that your team **and** your AI coding agents follow, for two stacks: **Python + FastAPI** and **Next.js**.

Ekibinizin **ve** yapay zeka kodlama ajanlarınızın uyduğu mimari ve kodlama kuralları — iki yığın için: **Python + FastAPI** ve **Next.js**.

> **Are you an agent?** → **[AGENTS.md](AGENTS.md)**. Everything in this file is for people.
> **Ajan mısın?** → **[AGENTS.md](AGENTS.md)**. Bu dosyadaki her şey insanlar içindir.

---

## What this is / Bu nedir

**EN** — Not runnable application code. It is a **contract**: your app lives in `backend/` and `frontend/`, this playbook lives beside it and says how those are built. Its value is that the rules survive a chat session — the next person, and the next agent, read the repo instead of scrolling your history.

**TR** — Çalışan uygulama kodu değildir. Bir **sözleşmedir**: uygulamanız `backend/` ve `frontend/` içinde yaşar, bu playbook onun yanında durur ve nasıl inşa edileceğini söyler. Değeri, kuralların bir sohbet oturumundan uzun yaşamasıdır — sonraki kişi ve sonraki ajan, geçmişinizi kaydırmak yerine repoyu okur.

---

## Setup in three steps / Üç adımda kurulum

**1. Put it beside your app / Uygulamanızın yanına koyun**

```
your-project/
├── backend/            # your FastAPI code
├── frontend/           # your Next.js code
└── coding-playbook/    # this repo — clone or submodule
```

Do not copy it into `src/`. · `src/` içine kopyalamayın.

**2. Point your agent at it / Ajanınızı ona yönlendirin**

Most tools read a file at the repo root automatically. Copy **[AGENTS.md](AGENTS.md)** to your project root, or add a rule in Cursor / Claude Code that says: *read `coding-playbook/AGENTS.md` first.*

Çoğu araç repo kökündeki bir dosyayı otomatik okur. **[AGENTS.md](AGENTS.md)**'yi proje kökünüze kopyalayın, ya da Cursor / Claude Code'a bir kural ekleyin: *önce `coding-playbook/AGENTS.md` dosyasını oku.*

**3. Read one page / Bir sayfa okuyun**

**[for-humans/](for-humans/README.md)** — six guides, English and Türkçe. Start with `01 Start here`.

---

## Your first prompt / İlk promptunuz

The playbook only works if the agent is told to use it. These are copy-paste ready.

Playbook ancak ajana kullanması söylendiğinde işe yarar. Aşağıdakiler kopyala-yapıştır hazır.

### Starting a new project / Yeni proje başlatırken

**EN**
```
Read coding-playbook/AGENTS.md and follow it.

I want to build: <one or two sentences, plain language>

Before writing any file, ask me the questions in agents/08-architecture.md
and write docs/data-model.md and docs/architecture.md for me to confirm.
```

**TR**
```
coding-playbook/AGENTS.md dosyasını oku ve ona uy.

Şunu yapmak istiyorum: <bir iki cümle, düz dille>

Herhangi bir dosya yazmadan önce agents/08-architecture.md içindeki soruları
bana sor ve onaylamam için docs/data-model.md ile docs/architecture.md yaz.
```

The agent should come back with **questions, not files**. If it starts writing code, it did not read the playbook.

Ajan dosyalarla değil, **sorularla** dönmeli. Kod yazmaya başladıysa playbook'u okumamıştır.

### A normal change / Normal bir değişiklik

**EN**
```
Read coding-playbook/AGENTS.md.

Stack:      backend
Task:       add a cancel endpoint for orders
Files:      backend/src/modules/orders/
Done when:  PATCH /v1/orders/{id}/cancel returns 200 for an unpaid order,
            409 for a paid one, and a test covers both
```

**TR**
```
coding-playbook/AGENTS.md dosyasını oku.

Yığın:      backend
Görev:      siparişler için iptal endpoint'i ekle
Dosyalar:   backend/src/modules/orders/
Bittiğinde: PATCH /v1/orders/{id}/cancel ödenmemiş siparişte 200,
            ödenmişte 409 dönüyor ve ikisinin de testi var
```

The four lines matter more than their wording: **which stack**, **one task**, **which files**, **how you will know it is done**.

Dört satırın kendisi, nasıl yazıldığından önemli: **hangi yığın**, **tek görev**, **hangi dosyalar**, **bittiğini nasıl anlayacaksınız**.

More templates and the mistakes that cost the most: [for-humans 02 — How to prompt](for-humans/en/02-how-to-prompt.md) · [Prompt yazımı](for-humans/tr/02-how-to-prompt.md).

### Something broke / Bir şey bozulduğunda

Paste the **exact** error text. Never "it doesn't work."
Hata metnini **birebir** yapıştırın. Asla "çalışmıyor" demeyin.

```
Read coding-playbook/AGENTS.md, then agents/04-errors.md.

<paste the full error, unedited>
```

---

## The six human guides / Altı insan rehberi

| # | English | Türkçe | |
|---|---|---|---|
| 01 | [Start here](for-humans/en/01-start-here.md) | [Buradan başlayın](for-humans/tr/01-start-here.md) | adopt it, change it |
| 02 | [How to prompt](for-humans/en/02-how-to-prompt.md) | [Prompt yazımı](for-humans/tr/02-how-to-prompt.md) | keep the agent small and correct |
| 03 | [Review agent code](for-humans/en/03-review-agent-code.md) | [Ajan kodunu inceleme](for-humans/tr/03-review-agent-code.md) | before you merge |
| 04 | [Pitfalls](for-humans/en/04-pitfalls.md) | [Tuzaklar](for-humans/tr/04-pitfalls.md) | what kills AI-built projects |
| 05 | [50 errors](for-humans/en/05-errors.md) | [50 hata](for-humans/tr/05-errors.md) | find it, fix it |
| 06 | [Glossary](for-humans/en/06-glossary.md) | [Sözlük](for-humans/tr/06-glossary.md) | every term explained |

---

## What is inside / İçinde ne var

| Folder | For | Contains |
|---|---|---|
| [`for-humans/`](for-humans/README.md) | you | six guides, EN + TR |
| [`AGENTS.md`](AGENTS.md) | agents | the entry point and routing map |
| [`agents/`](agents/README.md) | agents | how an agent should work: understand, plan, verify, stop |
| [`python-fastapi-backend/`](python-fastapi-backend/README.md) | both | 16 rule files + optional Extra shapes |
| [`nextjs-frontend/`](nextjs-frontend/README.md) | both | 16 rule files + optional Extra shapes |

The stack files are written in `WHEN:` / `MUST:` lines because an agent parses them. You can read them too — that syntax is the only unusual thing about them.

Yığın dosyaları `WHEN:` / `MUST:` satırlarıyla yazılmıştır, çünkü ajan onları ayrıştırır. Siz de okuyabilirsiniz — o sözdizimi, onlarla ilgili tek alışılmadık şey.

---

## Rules are not scripture / Kurallar kutsal metin değildir

If a rule fights something real in **your** product — your host, your legal constraints, a design you already shipped — change the playbook file and leave a one-line reason in it. Then the decision lives in git, where the next person and the next agent will actually find it.

Bir kural **sizin** üründe gerçek bir şeyle çakışıyorsa — host'unuz, yasal kısıtınız, çoktan çıkardığınız bir tasarım — playbook dosyasını değiştirin ve içine tek satırlık gerekçe bırakın. Böylece karar git'te yaşar; sonraki kişinin ve sonraki ajanın gerçekten bulacağı yerde.

The one thing not to do: let the code and the playbook disagree in silence. The next agent reads the file and undoes what you decided.

Yapılmaması gereken tek şey: kodun ve playbook'un sessizce çelişmesi. Sonraki ajan dosyayı okur ve kararınızı geri alır.
