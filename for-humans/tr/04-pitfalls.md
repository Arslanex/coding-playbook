# Vibe coding tuzakları

> **İnsanlar için** — yapay zeka araçlarıyla gerçek, sahaya çıkan yazılım üreten ekipler.

Vibe coding cazibelidir: düz dille anlatırsınız, çalışan kod çıkar, değişiklik istersiniz, olur. Fikirden uygulamaya döngü dakikalara iner.

Sonra bir şey bozulur ve nedenini bilmezsiniz.

Güç veren hız, tehlikeli yapan hızla aynıdır. Kötü kararlar daha hızlı alınır, kısayollar daha hızlı birikir, ve üretilen kod *profesyonel görünür* — bu yüzden sorunlar, kendiniz zorlanarak yazdığınız koddakinden daha geç ortaya çıkar.

Bunlar teorik riskler değil. Projeleri düzenli olarak öldüren hatalar. Erken yakalarsanız hepsi düzeltilebilir. Playbook disiplinin yerini tutmaz — onu **yönlendirir**: küçük promptlar, açık katmanlar, config ve güvenlik kuralları, kaynak gerçeği olarak git.

**English:** [04-pitfalls.md](../en/04-pitfalls.md) · **Dizin:** [for-humans/](../README.md) · **Ajanlar şunu okur:** [agents/README.md](../../agents/README.md)

---

## 1. Arkadaşa mesaj atar gibi prompt yazmak

**Kötü:** “Bana bir dashboard yap.”

Beş kelime. Ajan metrik, layout, veri kaynağı ve rolü kendisi seçer. Dashboard *gibi* görünen ama ihtiyacınıza uymayan bir şey çıkar — kazandığınız zamandan fazlasını varsayımları geri almakla harcarsınız.

**Çözüm:** Netlik ve playbook yönlendirmesi. Yığın, dosyalar, varlıklar ve bitti kriterlerini yazın. Prompt bin farklı uygulamayı tarif edebiliyorsa çok belirsizdir.

Şablonlar: [02 Prompt yazımı](02-how-to-prompt.md).

---

## 2. Her şeyi tek dev promptta yapmak

**Kötü:** Tek mesajda tüm özellikler, sayfalar ve edge case'ler.

Belirli bir eşiğin ötesinde model sessiz ödünler verir: özellikler birleşir, etkileşimler düşer, katmanlar karışır (HTTP mantığı UI'da, Next.js'ten DB).

**Çözüm:** Playbook dilimleriyle iteratif inşa.

1. Çekirdek akış (tek modül / tek feature paketi)
2. Ayarlar, ödeme, admin — **ayrı promptlar**
3. Her adım: sonraki prompttan önce git'te çalışan checkpoint

Zincir: model + migration → service + API → worker (gerekirse) → UI feature → test. Bkz. [02 Prompt yazımı](02-how-to-prompt.md).

---

## 3. Versiyon kontrolünü atlamak

**Kötü:** Commit yok; X bozulur, alakasız bir şey regrese olur; son iyi sürüme dönemezsiniz.

**Çözüm:** Erken ve sık commit. **Çalışan** her agent çıktısı checkpoint. Riskli prompt öncesi commit. Playbook kuralı ürüne uyarlandıysa **playbook dosyasını da commit edin** — sonraki ajan sohbeti değil git'i okur ([README.md](../../README.md)).

---

## 4. Üretilen kodu incelememek

**Kötü:** Düzgün format yanlış güven verir. Araştırmalar: birçok çözüm işlevsel ama güvenlik incelemesinde (auth, injection, secret, validation) yüksek oranda sorunlu.

**Çözüm:** Her agent çıktısını **taslak** sayın.

- Diff'i okuyun; özellikle auth, validation, env, SQL/sorgu
- Merge öncesi ilgili yığının [15-security.md](../../python-fastapi-backend/15-security.md) dosyası
- Kod inceleyemiyorsanız: deneyimli biri veya otomatik PR kontrolleri

Çalışan kod, güvenli kod değildir.

---

## 5. Veri modelini görmezden gelmek

**Kötü:** “Proje yönetim aracı yap.” Ajan varlıkları uydurur. Altı özellik yanlış iskelet üzerine; model düzeltmesi migration, API ve UI'ya yayılır.

**Çözüm:** **Özelliklerden önce veriyi** tarif edin.

- Varlıklar, ilişkiler, kısıtlar, durum enum'ları
- Backend: [06-database.md](../../python-fastapi-backend/06-database.md) + [07-migrations.md](../../python-fastapi-backend/07-migrations.md) model değişikliğiyle **aynı** promptta
- Mevcut DB: şemayı yapıştırın veya `@` ile verin; tablo adı tahmin ettirmeyin

---

## 6. Yapay zekanın güvenlik varsayımlarına güvenmek

**Kötü:** “Benim makinemde çalışıyor” optimizasyonu. Sık hatalar: gömülü secret, eksik validation, SQL injection, istemci bundle'ında secret (`NEXT_PUBLIC_`), gevşek yetkilendirme.

**Çözüm:** Güvenlik kuralları playbook'ta, agent'ın o gün yazdığı koddan **bağımsız**.

Prompt'a açıkça: “İstemci kodunda secret yok; tüm limitler Settings / lib/env.ts'te; yetkilendirme FastAPI'de.”

---

## 7. Kullanıcı doğrulamadan çok fazla inşa etmek

**Kötü:** Vibe coding yanlış şeyi inşa etmenin maliyetini düşürür. Bir günde sekiz özellik; kullanıcı yalnızca birincisine, farklı şekilde ihtiyaç duyuyordu.

**Çözüm:** Önce minimum faydalı akış. Gerçek kullanıcı, sonra sonraki prompt. Extra (SSO, tenant, agent, arama) yalnızca şekil **doğrulandıktan ve vardıktan sonra** — doküman diye değil ([01 Buradan başlayın](01-start-here.md)).

---

## 8. Bağlam penceresinin taşması

**Kötü:** Uzun sohbetler gereksinimleri “unutur”. Auth'a dokunmadınız ama login bozuldu. Rastgele regresyonlar.

**Çözüm:** Modül veya özellik dilimi başına **yeni sohbet**. Yeniden ekleyin: kısa karar özeti (3 satır) + minimal playbook `@` + güncel app yolları. Tüm uygulamayı tek thread'de kurmayın.

Ayrıntı: [02 Prompt yazımı](02-how-to-prompt.md).

---

## 9. Hata yönetimi ve edge case yokluğu

**Kötü:** Yalnızca mutlu yol — dolu formlar, API hep 200, DB hep satır döner.

**Çözüm:** Mutlu yol çalıştıktan sonra **ikinci prompt** savunma için.

Backend: [05-errors.md](../../python-fastapi-backend/05-errors.md), [12-api.md](../../python-fastapi-backend/12-api.md). UI dört durum: [01-design.md](../../nextjs-frontend/01-design.md).

---

## 10. Yapay zeka çıktısını bitmiş kod sanmak

**Kötü:** Üst hata. İncelemeden deploy, testsiz ölçekleme, anlamadan genişletme. Kötü olana kadar iyi görünür.

**Çözüm:** Üretim hızını inceleme disipliniyle eşleştirin.

```
AI yazar → diff incelersiniz → test → AI düzeltir → commit checkpoint → sonraki dilim
```

Playbook hem insan hem ajan sözleşmesidir. Gerçeklik saparsa playbook dosyasını tek satır gerekçeyle güncelleyin — tek kaynak sohbet olmasın.

---

## Gönderim öncesi kısa checklist

- [ ] Prompt net (yığın, dosyalar, playbook, bitti sayılır)
- [ ] Değişiklik öncesi/sonrası git checkpoint
- [ ] Diff incelendi; ilgili güvenlik dosyası
- [ ] Veri modeli açık; backend değiştiyse migration
- [ ] Frontend'te secret yok; config Settings / lib/env.ts'te
- [ ] Hata ve boş durumlar ele alındı
- [ ] “Her şey tek sohbette” değil

---

**Diğer rehberler:** [01 Buradan başlayın](01-start-here.md) · [02 Prompt yazımı](02-how-to-prompt.md) · [03 Ajan kodunu inceleme](03-review-agent-code.md) · [04 Tuzaklar](04-pitfalls.md) · [05 Hatalar](05-errors.md) · [06 Sözlük](06-glossary.md)
