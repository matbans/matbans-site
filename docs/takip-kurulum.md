# Müşteri takibi — uçtan uca kurulum (matbans.com)

Hedef: bir ziyaretçinin hangi kaynaktan geldiğini, hangi butona bastığını, randevu/ödeme yapıp yapmadığını ve CRM'de hangi kişi olduğunu tek zincirde görmek.

```
Google arama / reklam / sosyal
        │  (Search Console: arama verisi)
        ▼
   matbans.com ──► GA4 (G-SXCJRCN9VC): sayfa + tıklama olayları
        │            └─ kaynak (UTM/gclid) tarayıcıda saklanır
        ├─► Cal.com randevu  ─┐  metadata[utm_*, ga_client_id] taşınır
        ├─► Stripe ödeme     ─┤  client_reference_id = ga_<client_id>
        ├─► WhatsApp / tel / mail (tıklama olarak ölçülür)
        ▼                     ▼
      CRM  ◄── webhook (Cal.com BOOKING_CREATED, Stripe checkout.session.completed)
```

## Durum

| Parça | Durum |
|---|---|
| GA4 `G-SXCJRCN9VC` | Sitede kurulu (gtag.js) |
| Olay takibi (booking/payment/whatsapp/phone/email/brand_click) | Kodda hazır, canlıya çıkınca çalışır |
| Kaynak yakalama (ilk + son temas) | Kodda hazır (localStorage `mb_first`, `mb_last`) |
| Cal.com / Stripe'a kimlik taşıma | Kodda hazır — **Cal.com metadata alanını test et** (aşağıda) |
| Search Console | Sizin yapmanız gerekir |
| GA4 paneli ayarları | Sizin yapmanız gerekir |
| GTM | İsteğe bağlı, sizin hesap açmanız gerekir |
| CRM | Seçim ve hesap sizde |

## 1. Search Console (hesap sizde)
1. search.google.com/search-console → **Mülk ekle → Alan adı** → `matbans.com`.
2. Doğrulama TXT kaydı GoDaddy DNS'e eklenir (DNS'e dokunma kuralı: bu kaydı siz ekleyin, ben dokunmadım).
3. **Sitemaps** → `https://www.matbans.com/sitemap.xml` gönder.
4. GA4 → Yönetici → Ürün bağlantıları → **Search Console bağlantıları** → mülkü bağla (arama sorguları GA4'te görünür).

## 2. GA4 paneli (hesap sizde)
1. Yönetici → Olaylar: tıklama olayları ilk tıklamadan sonra görünür. Şu olayları **Anahtar olay (Key event)** işaretle:
   `booking_click`, `payment_click`, `whatsapp_click`, `phone_click`, `email_click`.
2. Yönetici → Özel tanımlar → olay kapsamlı özel boyut ekle: `link_url`, `link_text`, `location`, `first_source`, `last_source`.
3. Veri akışı → Gelişmiş ölçüm açık kalsın (scroll, giden tıklama).
4. Google Ads kullanacaksan Yönetici → Google Ads bağlantısı.
5. Veri saklama: 14 ay.

Not: `booking_click` randevu *tıklaması*dır, tamamlanmış randevu değildir. Gerçek dönüşüm CRM webhook'larından gelir (adım 4).

## 3. Tag Manager (isteğe bağlı)
Şu an yalnızca GA4 var; GTM'e şimdi gerek yok. Meta/LinkedIn/Google Ads pikseli eklenecekse kur:
1. tagmanager.google.com → hesap + web konteyneri → `GTM-XXXXXXX`.
2. `<head>`'e GTM snippet'i, `<body>` başına noscript ekle; **eski gtag.js bloğunu kaldır** (aksi halde sayfa görüntüleme çift sayılır).
3. `index.html` içindeki takip script'inde `GTM_ID = 'GTM-XXXXXXX'` yap → olaylar `dataLayer`'a `event: booking_click` vb. olarak düşer.
4. GTM'de: GA4 Yapılandırma etiketi (`G-SXCJRCN9VC`), her olay için "Özel Etkinlik" tetikleyici + GA4 Olay etiketi.
Kodu o zaman ben güncellerim; GTM ID'sini yazman yeterli.

## 4. CRM + webhook (seçim sizde)
Öneri: küçük hacim için **HubSpot Free** (form/kişi/deal ücretsiz). Alternatif: Pipedrive.
1. Cal.com → Ayarlar → Developer → **Webhooks** → `BOOKING_CREATED` → CRM'in webhook/Zapier-Make adresi. Payload'daki `metadata` içinde `utm_*`, `first_source`, `ga_client_id` gelir → kişiye "lead source" olarak yaz.
2. Stripe → Developers → Webhooks → `checkout.session.completed` → aynı yer. `client_reference_id` = `ga_<GA client id>` → ödemeyi aynı kişiye bağlar.
3. WhatsApp/telefon/e-posta ile gelenler: CRM'e elle "kaynak" alanıyla girin (ölçüm sadece tıklamayı bilir).
4. GA4 → CRM bağı: Measurement Protocol ile `purchase`/`generate_lead` olayını `client_id` ile GA4'e geri gönder (isteğe bağlı, Zapier/Make ile).
HubSpot MCP bu oturumda yetkilendirilmemiş; yetkilendirirsen (`/mcp`) alanları ve pipeline'ı ben kurabilirim.

## 5. Açık riskler / doğrulanacaklar
- **Cal.com `metadata[...]` URL parametresi**: sürümünüzde destekleniyorsa webhook'ta görünür. Test: siteden randevu linkine tıkla, bir deneme randevusu al, webhook payload'unu kontrol et. Çalışmazsa Cal.com'da "booking question" olarak gizli alan eklenip `?utm_source=` ile doldurulur.
- **Stripe `client_reference_id`**: Payment Link'lerde destekli; Dashboard → ödeme detayında görünmeli.
- **Çerez rızası (KVKK/GDPR)**: Ziyaretçilerin çoğu TR. Şu an rıza banner'ı yok; GA4 çerezleri rıza öncesi çalışıyor. Canlı reklam/piksel eklemeden önce Consent Mode v2 + banner ekle. Hukuki danışmanlık gerektirebilir.
- Kaynak bilgisi tarayıcı depolamasında tutulur; ziyaretçi temizlerse veya gizli sekmedeyse kaybolur.

## Doğrulama (canlıya çıktıktan sonra)
1. GA4 → Raporlar → **Gerçek zamanlı**; siteyi `?utm_source=test&utm_medium=manual` ile aç, butonlara bas → olaylar görünmeli.
2. Tarayıcı konsolu: `localStorage.mb_first` ve `mb_last` dolu mu.
3. Randevu/ödeme linkinin URL'sinde `metadata[...]` / `client_reference_id` var mı.
