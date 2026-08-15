# For humans

Guides for the people using this playbook — not for the agents. Everything here is prose: no `WHEN:` / `LOAD:` routing, no `MUST NOT:` lines to parse.

Bu playbook'u kullanan **insanlar** için rehberler — ajanlar için değil. Buradaki her şey düz metindir: ajanlara yönelik `WHEN:` / `LOAD:` yönlendirmesi yok.

**English → [`en/`](en/)** · **Türkçe → [`tr/`](tr/)**

> **Editing these guides:** every page exists twice. A change to `en/NN-….md` must land in `tr/NN-….md` in the same commit, and the reverse. A guide that is true in one language and stale in the other is worse than one that only exists in one.
> **Bu rehberleri düzenlerken:** her sayfa iki kez var. `en/NN-….md`'deki bir değişiklik aynı commit'te `tr/NN-….md`'ye de girmeli, tersi de geçerli. Bir dilde doğru, diğerinde eskimiş bir rehber, tek dilde var olandan kötüdür.

> **Agents:** do not load this directory. Your routing index is [agents/README.md](../agents/README.md).
> **Ajanlar:** bu klasörü yüklemeyin. Yönlendirme indeksiniz [agents/README.md](../agents/README.md).

---

## The six guides / Altı rehber

| # | English | Türkçe | What it answers / Neyi cevaplar |
|---|---|---|---|
| 01 | [Start here](en/01-start-here.md) | [Buradan başlayın](tr/01-start-here.md) | What this repo is, how to adopt it, how to change it |
| 02 | [How to prompt](en/02-how-to-prompt.md) | [Prompt yazımı](tr/02-how-to-prompt.md) | How to phrase a task so the agent stays small and correct |
| 03 | [Review agent code](en/03-review-agent-code.md) | [Ajan kodunu inceleme](tr/03-review-agent-code.md) | What to check before you merge code you did not write |
| 04 | [Pitfalls](en/04-pitfalls.md) | [Tuzaklar](tr/04-pitfalls.md) | The ten mistakes that kill AI-built projects |
| 05 | [50 errors](en/05-errors.md) | [50 hata](tr/05-errors.md) | Something broke — find the error, get the fix |
| 06 | [Glossary](en/06-glossary.md) | [Sözlük](tr/06-glossary.md) | Every term the rule files use without explaining |

---

## Where to start / Nereden başlanır

**Adopting the playbook for the first time** → 01, then 02.
Playbook'u ilk kez projeye alıyorsanız → 01, sonra 02.

**About to review a pull request** → 03. It is the only page you need open.
Bir pull request inceleyecekseniz → 03. Açık tutmanız gereken tek sayfa odur.

**Something is broken right now** → 05. Copy the exact error text and find it in the list.
Şu anda bir şey bozuksa → 05. Hata metnini olduğu gibi kopyalayıp listede arayın.

**A rule file says something you cannot picture** → 06.
Bir kural dosyası gözünüzde canlanmayan bir şey söylüyorsa → 06.

**Things keep going wrong and you are not sure why** → 04.
Sürekli bir şeyler ters gidiyor ama sebebini bilmiyorsanız → 04.

---

## By role / Role göre

| Role / Rol | Read / Okuyun |
|---|---|
| Product owner, non-engineer | 01, 04, 06 |
| Tech lead adopting the playbook | 01, 02, 03 — then skim each stack's `README.md` |
| Developer writing prompts daily | 02, 05 — keep 03 open at review time |
| Reviewer / approver | 03, and the `15-security.md` of the stack in the diff |

---

## What is not here / Burada olmayanlar

The rule files themselves. Those are written for agents and live in the stack folders — read them when you want the detail behind a rule, but you do not need them to use the playbook.

Kuralların kendisi. Onlar ajanlar için yazılmıştır ve yığın klasörlerinde durur — bir kuralın ardındaki detayı merak ettiğinizde okuyun, ama playbook'u kullanmak için gerekmez.

- [Playbook root / Kök](../README.md) — agent routing index
- [FastAPI backend](../python-fastapi-backend/README.md) · [Next.js frontend](../nextjs-frontend/README.md)
- [Agent operations / Ajan operasyonları](../agents/README.md)
