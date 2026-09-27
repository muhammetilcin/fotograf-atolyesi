# Fotoğraf Atölyesi — GitHub + Cloudflare Pages

Bu paket bağımsız fotoğraf düzenleme web uygulamasıdır. Ana kaynak GitHub'da tutulur; GitHub değişiklikleri Cloudflare Pages tarafından yayımlanır. Bilgisayara Node.js kurmanız gerekmez; derleme Cloudflare'da yapılır.

## Şu anki durum

- Fotoğraf yükleme, komut yazma, öneri seçme, korunacak kişiyi işaretleme, önce/sonra, geri alma ve indirme akışı hazırlanmıştır.
- Gerçek görsel düzenleme isteği sunucu üzerinden OpenAI Images Edits API'ına gönderilir.
- Bu pakette API anahtarı yoktur. Anahtar eklenmeden yapay zekâ düzenlemesi başlamaz.
- Sunucu testleri yapay servis yanıtlarıyla geçti. Hesabınızla gerçek görsel düzenlemesi henüz test edilmedi.
- Bu ZIP, GitHub veya Cloudflare hesabınızda otomatik proje oluşturmaz.

## 1. GitHub'a yükleme

1. GitHub hesabınızda yeni bir depo oluşturun. Önerilen ad: `fotograf-atolyesi`.
2. ZIP'i çıkarın.
3. GitHub'da **Add file → Upload files** ile klasörün İÇİNDEKİ dosyaları yükleyin; `src`, `scripts`, `public` ve `package.json` depo kökünde olmalı.
4. `main` dalına kaydedin. ZIP dosyasını tek başına yüklemek yeterli değildir.

## 2. Cloudflare Pages'e bağlama

Cloudflare'da yeni bir **Pages** projesi açıp bu GitHub deposunu bağlayın.

| Ayar | Değer |
|---|---|
| Production branch | `main` |
| Framework preset | `None` |
| Build command | `node scripts/build.mjs` |
| Build output directory | `public` |
| Root directory | Boş (depo kökü) |

Paket `public/_worker.js` üretir. Bu dosya hem sayfayı hem `/api/status` ve `/api/edit` uçlarını sunar. Bu nedenle GitHub bağlantılı Pages yayını kullanın; yalnızca HTML dosyasını başka bir statik servise atmak yeterli değildir.

## 3. Yapay zekâ bağlantısını açma — gerekli adım

Cloudflare Pages projenizde **Settings → Variables and Secrets** bölümünde Production ortamı için aşağıdakini ekleyin:

| Tür | Ad | Değer |
|---|---|---|
| Secret | `OPENAI_API_KEY` | Kendi OpenAI API anahtarınız |
| Variable (isteğe bağlı) | `OPENAI_IMAGE_MODEL` | Varsayılan: `gpt-image-2.5-sunburst` |

Gizli anahtarı GitHub'a, HTML'e veya mesajlara yazmayın. OpenAI Developers ChatGPT eklentisi bu kurulum için zorunlu değildir. Uygulama doğrudan API'a bağlanır. Modelin hesabınızda erişilebilir olması ve API kullanımının etkin olması gerekir.

Kaydettikten sonra Pages dağıtımını yeniden çalıştırın. Sayfayı yenileyin. Eksik bağlantı uyarısı kalkınca bir fotoğrafla gerçek düzenleme testi yapın.

## Uyarının anlamı

**“Otomatik düzenleme bağlantısı henüz etkin değil”** uyarısı `OPENAI_API_KEY` sunucuda eksik veya boş olduğunda çıkar. Fotoğraf, internet tarayıcısı veya kişi seçimi hatası değildir. Yeni pakette bu durumda düzenleme butonu devre dışı kalır; fotoğraf gönderilmez.

Anahtarın bulunması sadece yapılandırmanın mevcut olduğunu gösterir. Anahtarın geçerliliği, model erişimi ve kullanım kotası gerçek API isteğinde kontrol edilir. Yetki veya kota hatası olursa uygulama ayrıca bildirir.

## Dosyalar

- `src/index.html`: Kullanıcı arayüzü
- `src/worker.mjs`: Görsel düzenleme API bağlantısı
- `scripts/build.mjs`: Pages çıktısını üretir
- `scripts/api.test.mjs`: İstek ve hata işleme testleri
- `public/`: Yayımlanacak hazır çıktı (kaynak değişince derlemeyle yenilenir)

## Teknik doğrulama

Geliştirici ortamında: `node scripts/build.mjs` ardından `node --test scripts/api.test.mjs`.

Fotoğraf ve komut yalnızca Düzenle'ye basıldığında OpenAI'a gönderilir. Uygulama bunları kalıcı veritabanına yazmaz. Sonuçlar açık sekmede tutulur; sayfa yenilenirse kaybolur.

## Resmî başvuru

- https://developers.cloudflare.com/pages/functions/advanced-mode/
- https://developers.cloudflare.com/pages/functions/bindings/
- https://developers.openai.com/api/docs/guides/image-generation
