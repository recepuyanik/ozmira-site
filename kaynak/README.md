# OZMIRA

Türkçe · [English](README.en.md) · [Deutsch](README.de.md) · [Español](README.es.md)

Kendi yapay zekâ sistemlerini tek bir 3B sahnede gösteren yerel uygulama. Her sistem kendi renginde ve biçiminde görünür (galaksi, hologram lif, çift sarmal, kara delik, küre). Bir ajan çalışmaya başlayınca merkezden ona bir sinyal gider, işi bitince geri döner ve harcanan token sahnede görünür.

Hesap yok, bulut yok. Sistem tanımların, olayların ve erişim anahtarın yalnızca senin bilgisayarında durur. İsteğe bağlı, varsayılan olarak kapalı bir anonim kullanım paylaşımı vardır (aşağıda ayrıntılı).

## Başlatma

Gereken: Python 3.8 veya üstü (3.13'te sınandı) ve güncel bir tarayıcı. Başka kurulum yok.

| Sistem | Çalıştır |
| --- | --- |
| Windows | `ozmira.bat` dosyasına çift tıkla |
| macOS | `ozmira.command` dosyasına çift tıkla (ilk seferde sağ tık, Aç) |
| Linux | `./ozmira.sh` |
| Hepsi | `python3 ozmira.py` (Windows'ta `py ozmira.py`) |

Tarayıcı kendiliğinden açılır. Açılmazsa terminalde yazan `http://localhost:8787/` adresine git. Durdurmak için terminalde Ctrl+C.

İlk açılışta örnek bir sahne ve kurulum sihirbazı gelir. Sihirbazda merkezin adını seçer, sistemlerini eklersin. Hazır şablonlar var (Claude, ChatGPT, Gemini, Grok, Başka); ad, stil ve rengi istediğin gibi değiştirebilirsin. Sonradan alttaki Düzenle düğmesiyle her şeyi değiştirebilirsin.

## Uygulama penceresi olarak açmak

OZMIRA'yı tarayıcı sekmesi yerine kendi penceresinde, adres çubuğu olmadan açabilirsin. Bunun için bilgisayarında Chrome, Edge ya da Brave kurulu olması gerekir (Windows 10 ve 11'de Edge zaten vardır).

| Sistem | Çalıştır |
| --- | --- |
| Windows | `ozmira-uygulama.bat` dosyasına çift tıkla (konsol penceresi açmaz) |
| macOS | `ozmira-uygulama.command` dosyasına çift tıkla |
| Linux | `./ozmira-uygulama.sh` |
| Hepsi | `python3 ozmira.py --uygulama` |

Pencereyi kapatınca OZMIRA da durur. Pencere, kişisel tarayıcı oturumunun dışında ayrı bir profil klasörü kullanır (`.ozmira/pencere`); oradaki sitelere girişlerin bu pencereyle karışmaz.

Masaüstüne ve (Linux'ta) uygulama menüsüne kısayol koymak için bir kez şunu çalıştır:

```
python3 ozmira.py kisayol
```

Kısayol uygulama penceresiyle açar ve OZMIRA simgesini kullanır (`ozmira.ico`, `web/icon.png`). Bu bir kısayoldur; tek dosyalık bir `.exe` ya da yükleyici değildir ve Python'un kurulu olmasını gerektirir.

## Dil

Arayüz Türkçe, İngilizce, Almanca ve İspanyolca olarak gelir. İlk açılışta tarayıcının dili seçilir (desteklenmeyen bir dilde İngilizce). Dock'ın sağındaki dil düğmesinden (TR, EN, DE, ES) istediğin zaman değiştirebilirsin; seçimin bu tarayıcıda hatırlanır. Komut satırı çıktıları sistem dilini izler; `OZMIRA_DIL=tr|en|de|es` ile zorlayabilirsin. Bu README'nin diğer dillerdeki sürümleri: `README.en.md`, `README.de.md`, `README.es.md`.

## Ücretsiz ve Pro

| | Ücretsiz | Pro |
| --- | --- | --- |
| 3B sahne, olay bildirme, token sayacı, yerel çalışma | Var | Var |
| Yapay zekâ sistemi (Claude, ChatGPT, Gemini vb.) | En fazla 2 | Sınır yok* |
| Ajan | Toplam en fazla 7 | Sınır yok* |
| Görsel stiller | Galaksi, Küre | Beşi birden (Lif, Çift sarmal, Kara delik dahil) |

\* Pro'da ücretli bir sınır yoktur. Sahne akıcı kalsın diye yalnızca teknik bir tavan vardır: 24 sistem ve sistem başına 200 ajan.

Ücretsiz sürümde profilin 7'den fazla ajan içeriyorsa (klasörden okunanlar dahil) kaydederken uyarı alırsın. Sonradan ajan klasörün büyürse sahnede ilk 7 ajan görünür, fazlası ekranda "+N ajan Pro'da görünür" notuyla belirtilir; hiçbir şey silinmez.

Pro lisans anahtarını dock'taki Hesap düğmesinden (ücretsizken "Ücretsiz" yazar) yapıştırırsın. Anahtar bu bilgisayarda çevrimdışı doğrulanır; sunucuya bağlanmaz. Lisans süreli olabilir; süresi dolunca uygulama ücretsiz sınırlara döner, kayıtlı profilin silinmez ama yeniden kaydederken sınırlar uygulanır.

## Ajanlarını eklemek

Her sistem için iki yol var, ikisini birlikte de kullanabilirsin:

- **Klasör:** `.md` dosyalarından oluşan bir ajan klasörü göster. Her dosyanın adı ajan adı olur; dosyanın başında `name:` ve `description:` satırları varsa onlar kullanılır. Yalnızca dosyanın ilk 4 KB'ı okunur ve yalnızca bu iki satır alınır, içerik kaydedilmez.
- **Elle:** ajan adlarını satır satır ya da virgülle yaz.

## Çalışmayı canlı göstermek (Bağlan)

OZMIRA bir ajanın ne zaman çalıştığını kendiliğinden bilemez; ona haber vermen gerekir. Dock'taki Bağlan düğmesi, sistemlerine göre hazır komutları kopyalamanı sağlar.

```
python3 ozmira.py olay <sistem> <ajan> basla
python3 ozmira.py olay <sistem> <ajan> bitti 1200 "iş etiketi"
```

`<sistem>` sistemin kısa kimliğidir (adından türetilir; Bağlan penceresi kimlikleri listeler). `1200` harcanan token sayısıdır, isteğe bağlıdır. Aynı şeyi HTTP ile de yapabilirsin:

```
curl -X POST http://localhost:8787/api/olay \
  -H "X-Token: <anahtarın>" -H "Content-Type: application/json" \
  -d '{"sistem":"<sistem>","ajan":"<ajan>","olay":"bitti","token":1200}'
```

Bu komutu kendi betiklerinden, otomasyonlarından ya da araçlarının çalışma öncesi ve sonrası adımlarından çağırabilirsin. Gönderilen tek şey sistem, ajan adı, olay türü, token sayısı ve istersen 120 karakterlik bir iş etiketidir. Prompt, yanıt ya da kod gönderilmez.

Bir ajan `basla` bildirip 30 dakika içinde `bitti` bildirmezse sahnede çalışıyor görünmeyi bırakır.

## Gizlilik ve güvenlik

- **Veri nerede:** ev dizinindeki `.ozmira` klasöründe (`profil.json`, `olaylar.jsonl`, `ayar.json`, `kullanim.json`, `.token`). Başka bir yer için `OZMIRA_HOME` ortam değişkenini ayarla. Bu klasörü paylaşmadığın sürece kimse senin sistemini görmez. Uygulama klasörünün kendisinde kişisel veri yoktur ve paylaşılabilir.
- **Ağa kapalı:** sunucu yalnızca `localhost`'u dinler. Başka bilgisayarlar bağlanamaz.
- **Anahtar:** ilk çalıştırmada rastgele bir anahtar üretilir (`.ozmira/.token`; macOS ve Linux'ta yalnızca senin kullanıcın okuyabilir, Windows'ta dosya izinleri farklı çalıştığı için bilgisayarını başkalarıyla paylaşıyorsan klasörü korumalı bir yere koy). Tüm API istekleri bu anahtarı ister; Host ve Origin başlıkları denetlenir.
- **Paylaşım varsayılan olarak kapalı.** Hesap penceresinden izin verirsen ve bu kopyayla birlikte bir toplayıcı adresi (`geri_bildirim_adresi.txt`) geldiyse, günde en çok bir kez şunlar anonim olarak gönderilir: sürüm, plan, rastgele kurulum kimliği ve sayaçlar (alınan olay sayısı, profil kaydetme sayısı). Sistem adların, ajan adların, iş etiketlerin, klasör yolların, token miktarların ve IP adresin gönderilmez. Geri bildirim penceresine yazdığın metin de yalnızca bu izinle gönderilir; yazdıkların her durumda ayrıca bu bilgisayardaki veri klasörüne kaydedilir.
- **Başka dış bağlantı:** three.js kütüphanesinin jsdelivr'dan yüklenmesi. Bunu kapatmak için aşağıdaki "Çevrimdışı" bölümüne bak.
- **Tarayıcı koruması:** katı bir Content-Security-Policy kullanılır; ajan adı ya da açıklaması gibi dosyalardan gelen metinler sayfaya HTML olarak değil, düz metin olarak yazılır.
- **Sıfırlamak için:** uygulamayı durdur ve `.ozmira` klasörünü sil.

## Çevrimdışı çalışma

```
python3 ozmira.py vendor
```

Bu komut three.js'i bir kez indirip `web/vendor/three` içine koyar (MIT lisansıyla birlikte). Sonrasında OZMIRA internete hiç bağlanmadan çalışır. İnternet yoksa komut açık bir hata mesajıyla durur, yarım dosya bırakmaz.

## Telefondan bakmak (isteğe bağlı, dikkatli kullan)

```
python3 ozmira.py --lan
```

Sunucu aynı ağdan erişilebilir olur ve terminal telefon için bir adres yazar. Adresin sonundaki `?k=` anahtarı olmadan sayfa açılmaz. Yine de aynı ağdaki herkes sunucuya ulaşabilir; ortak Wi-Fi'larda (kafe, otel, okul) kullanma. İşin bitince normal şekilde yeniden başlat.

## Dosyalar

| Dosya | Görevi |
| --- | --- |
| `ozmira.py` | Sunucu ve komut satırı (başlat, `--uygulama`, olay, vendor, bilgi, kisayol) |
| `ozmira.bat`, `ozmira.command`, `ozmira.sh` | Tarayıcıda başlatıcılar |
| `ozmira-uygulama.bat`, `.command`, `.sh` | Uygulama penceresinde başlatıcılar |
| `ozmira.ico`, `web/icon.png`, `web/icon.svg` | Simgeler |
| `web/index.html`, `web/stil.css` | Sayfa ve görünüm |
| `web/app.js` | Panel, kurulum sihirbazı, Bağlan, Hesap, Geri bildirim ve Dil pencereleri |
| `web/i18n.js` | Dört dilin metinleri ve sunucu hata iletilerinin çevirisi |
| `lisans_acik.json` | Lisans doğrulama anahtarı (herkese açık olabilir; satıcı ekler) |
| `web/sahne.js` | 3B sahne (beş stil, 1 ile 24 sistem) |

`python3 ozmira.py bilgi` veri klasörünü ve olay sayısını yazar.

## Sınırlar

- Sahne ağır bir 3B görüntüdür. Eski ekran kartlarında ya da pil tasarrufundayken kalite kendiliğinden düşer; ışıma kapanabilir.
- Teknik tavan (Pro dahil): 24 sistem, sistem başına 200 ajan. 8'den fazla sistemde sahne büyür ve kamera uzaklaşır; bu yerleşim gerçek ekran kartında henüz görsel olarak denenmedi.
- Sürüm 0.2.0. Yalnızca yerel kullanım için tasarlandı.
- Uygulama penceresi ve masaüstü kısayolu Linux'ta otomatik testlerle sınandı; Windows ve macOS'ta kısayol komutu henüz gerçek bir bilgisayarda denenmedi. Çalışmazsa `ozmira-uygulama.bat` / `.command` dosyalarını kullan.
- Dört dilin çevirileri yapay zekâ ile hazırlandı; ana dili İngilizce, Almanca ya da İspanyolca olan biri bir gözden geçirmeli.
- Sahne tek bir Windows bilgisayarda gerçek ekran kartıyla denendi. Diğer ekran kartlarında ilk açılışta sahne görünmezse tarayıcıda WebGL'in açık olduğunu kontrol et.
