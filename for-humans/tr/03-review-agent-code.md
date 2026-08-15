# Ajanın yazdığı kodu incelemek

> **İnsanlar için** — pull request'i onaylayan kişi. Yazmadığınız bir kodu merge ediyorsunuz.

Her yığında bir `15-security.md` ve içinde "Agent pass (PR)" listesi var. O, ajanın kendi işini kontrol etmesidir. Bu sayfa ise **sizin** ajanı kontrol etmenizdir ve ikisi aynı iş değildir: ajan hatırladığı kuralları doğrular, aşağıdaki hatalar ise yaptığını fark etmediği hatalardır.

Her satırı okumanız gerekmiyor. **Doğru** satırları, doğru sırayla okumanız gerekiyor.

**English:** [03-review-agent-code.md](../en/03-review-agent-code.md) · **Dizin:** [for-humans/](../README.md) · **Ajanlar şunu okur:** [agents/README.md](../../agents/README.md)

---

## Diff'i bu sırayla okuyun

İlk yanlışta durun. 1. adımdaki kötü bir cevap, 3–5. adımları anlamsız kılar.

**0. Varsa plan.** Birden fazla tur süren bir işte ajan `.agent/plan.md` içinde bir plan tutar — görev, done-when, dilimler ve yol boyunca verdiği kararlar. Diff'ten **önce** onu okuyun. Ajanın *yapacağım dediğiyle* yaptığını karşılaştırmak, kod okumaktan hızlıdır; `## Decisions` bölümü de diff'in hiç göstermediği şeyleri söyler: bir tip neden seçildi, neyin yapılmayacağına karar verildi. Plan ile diff çelişiyorsa, bulgunuz odur.

**1. Koda bakmadan önce dosya listesi.** Ne değişti değil — *hangi dosyalar* değişti. Bu küme istediğiniz şeyle örtüşüyor mu? Tek alan ekleme talebinin on bir dosyaya dokunması bulgunun kendisidir ve beş saniyede görürsünüz.

**2. Veri modeli ve varsa migration.** Şema, geri alması en zor şeydir. Bir sürüm sonra yanlış kolonun içinde veri vardır. Migration'ın tek mantıksal değişiklik yaptığını, `downgrade()`'in dolu olduğunu ve eski kodun yeni şemada hâlâ ayağa kalkabildiğini kontrol edin.

**3. Sınır.** Bu kod, kimin neye izinli olduğuna nerede karar veriyor? Bu playbook'ta cevap her zaman API'dir, asla arayüz değil. Bir React bileşeninde veya Server Action içinde beliren izin kontrolü, ne kadar doğru görünürse görünsün bir kusurdur.

**4. Kimsenin istemediği durumlar.** Yükleniyor, boş, hata. Ajanlar varsayılan olarak başarı yolunu üretir. Diff bir liste ekliyor ve sadece listeyi gösteriyorsa, iş bitmemiştir.

**5. Testler.** Var olduklarını değil, **ne iddia ettiklerini** okuyun. Sonra tekrar okuyup şunu sorun: özellik bozulsaydı bu test patlar mıydı?

---

## Ajan kodunda sekiz tipik hata

Her birinin, tüm diff'i okumadan doğrudan arayabileceğiniz bir işareti var.

### 1. Kapsam kayması

**İşaret:** diff'te hiç bahsetmediğiniz dosyalar, açıklamada "hazır oradayken", mantık değişikliğine karışmış biçimlendirme.

Alakasız kısmın açıklanmasını değil, geri alınmasını isteyin. Playbook'un kendi kuralı tur başına tek dilimdir ([agents/02-turn.md](../../agents/02-turn.md)). Sessizce iki iş yapan bir diff'i, yarısı yanlış çıktığında temiz şekilde geri alamazsınız.

### 2. Var olmayan bir paket

**İşaret:** `package.json` veya `pyproject.toml` içindeki her yeni satır.

Ajanlar hangi paket adlarının gerçek olduğunu güvenilir biçimde bilmez ve saldırganlar, uydurulması muhtemel kulağa makul gelen adları önceden kaydeder. Eklenen her bağımlılık için registry sayfasını açın. Son sürüm tarihine ve bağlı repoya bakın — gerçek ama terk edilmiş bir paket, farklı bir sorundur ama çözümü aynıdır: eklemeyin.

Bir de şunu sorun: platform bunu zaten yapmıyor mu? Tarih biçimlendirme, debounce, UUID bir bağımlılık değildir.

### 3. Kural kodda esnetilmiş ama playbook'ta esnetilmemiş

**İşaret:** kod, playbook dosyasının "yapma" dediği şeyi yapıyor ve o playbook dosyası diff'te değişmemiş.

Repoyu çürüten şey budur. Playbook'un sözleşmesi, kararın sohbet kaydında değil git'te yaşamasıdır ([01 Buradan başlayın](01-start-here.md)). Kural gerçekten ürününüz için esnemek zorunda kaldıysa, playbook dosyası **aynı PR içinde** tek satır gerekçeyle değişir. Esnemek zorunda değildiyse, onun yerine kod değişir. Olmaması gereken şey ikisinin birbirinden ayrılmasıdır — çünkü sonraki ajan dosyayı okur ve az önce kabul ettiğiniz şeyi geri getirir.

### 4. Patlaması imkânsız testler

**İşaret:** iddiası olmayan test, tüm sayfanın snapshot'ı, tam da iddianın kontrol ettiği şeyi döndüren mock.

Üretilmiş arayüz üzerinde snapshot testi, testsizlikten kötüdür: ajanın ürettiği her şeyi — reddedeceğiniz kısımlar dahil — sabitler. Testin hangi davranışı koruduğunu sorun. Cevap "render oluyor" ise, hiçbir şeyi korumuyor.

### 5. Yalnızca mutlu yol

**İşaret:** boş durumu olmayan yeni ekran; hata dalı olmayan bir `fetch`; API'nin evet dediğini varsayan form.

### 6. Yetkilendirmenin arayüzde olması

**İşaret:** bileşen içinde `if (user.role === …)`, gizlenmiş buton, izin diye tarif edilen disabled kontrol.

Gizli buton bir kontrol değildir — curl kontroldür. Kontrol API'de olmak zorundadır; arayüzün onu gizlemesi üstüne eklenen bir nezakettir. Butonun kaybolduğunu değil, API'nin çağrıyı reddettiğini doğrulayın.

### 7. Tarayıcı bundle'ında sır veya ortama özel değer

**İşaret:** yeni bir `NEXT_PUBLIC_*` değişkeni.

Bu tek işaret iki ayrı kusuru barındırır. Sırsa, artık sonsuza dek geneldir ve döndürmek onu taşıyan her deploy'u yeniden build etmek demektir. Sadece ortama özgüyse, build anında gömülmüştür ve image staging'den prod'a terfi ettiği anda yanlış olur. Hiçbiri host'taki değeri değiştirerek düzelmez.

### 8. Geri alınamayan veya ship olduktan sonra düzenlenmiş migration

**İşaret:** içinde `pass` olan `downgrade()`, ya da `main`'de olan bir revision dosyasında değişiklik.

---

## Ne zaman itiraz etmeli, ne zaman bırakmalı

Playbook kurallarını üç güç seviyesinde işaretliyor ve bunlar sizin için de bir şey ifade ediyor.

| İşaret | Neyi korur | Ne yaparsınız |
|---|---|---|
| `MUST [critical]` | Bir sır, başka kullanıcının verisi, ya da bir yetki kapısı | Merge etmeyin. Bu bir tercih meselesi değil. |
| `MUST` (işaretsiz) | Playbook'un seçtiği şekil — tutarlılık, güvenlik değil | Ürün gerçekten istisnaya ihtiyaç duyuyorsa merge edin; playbook dosyasını aynı PR'da güncelleyin |
| `SHOULD` | Makul bir varsayılan | İnceleyenin takdiri. Tek başına bir tur gidip gelmeye değmez. |

Adı konması gereken hata: işaretsiz bir `MUST`'ı kritikmiş gibi ele almak. Klasör adlandırması yüzünden bir PR'ı, sızmış bir anahtar kadar sert bloke ederseniz, insanlar bu ayrıma güvenmeyi bırakır ve bir sonraki gerçek kritik hata elini kolunu sallayarak geçer.

---

## Altmış saniyelik sürüm

- [ ] Değişen dosya listesi istediğim şeyle örtüşüyor
- [ ] Her yeni bağımlılık var, bakımı sürüyor ve gerçekten gerekli
- [ ] Şema değişikliği: tek mantıksal değişiklik, `downgrade()` dolu, eski kod hâlâ kalkıyor
- [ ] Her izin kararı API'de, arayüzde değil
- [ ] Ekrana gelen her yeni şeyin yükleniyor / boş / hata durumu var
- [ ] Özellik bozulsa testler patlar
- [ ] Sır veya ortama özel değer taşıyan yeni `NEXT_PUBLIC_*` yok
- [ ] Bir playbook kuralı esnetildiyse, playbook dosyası aynı PR'da değişti

---

**Diğer rehberler:** [01 Buradan başlayın](01-start-here.md) · [02 Prompt yazımı](02-how-to-prompt.md) · [03 Ajan kodunu inceleme](03-review-agent-code.md) · [04 Tuzaklar](04-pitfalls.md) · [05 Hatalar](05-errors.md) · [06 Sözlük](06-glossary.md)
