# Matbans.com — site repo

Matbans LLC'nin tanıtım/danışmanlık sitesi (ABD pazarına açılmak isteyen Türk üretici markalar için e-ticaret danışmanlığı). Şemsiye şirket; altında KukuLondon, aryalovin.com, MatbansDesignStudio gibi ayrı markalar var.

## Stack
- Tek dosya: `index.html` (HTML + inline CSS/JS). Build adımı yok.
- `sitemap.xml` (tek URL: https://www.matbans.com/)
- Deploy: GitHub `matbans/matbans-site` → Vercel (main'e push = canlıya çıkar). Repo public.
- DNS GoDaddy'de, mail Google Workspace. DNS/mail'e dokunma.

## Çalışma komutları
- Yerelde önizleme: `python3 -m http.server 8000` (repo kökünde)
- Deploy: `git add -A && git commit -m "..." && git push` (Vercel otomatik alır)

## Kurallar
- Site dili Türkçe. Metinleri değiştirirken ton: sade, profesyonel, iddiasız.
- Fontlar: Newsreader (başlık), Inter (gövde), IBM Plex Mono (etiket). Renk değişkenleri `:root` içinde; yeni renk eklemeden önce mevcutları kullan.
- Dış linkler `target="_blank" rel="noopener"`.
- Canlı linkler (Cal.com randevu, Stripe ödeme, WhatsApp) sadece kullanıcı isterse değişir. Sessizce değiştirme.
- Rakam, fiyat, iddia eklerken kaynak/veri iste; varsayımla yazma. Veri yoksa "veri yok" de.
- Küçük, cerrahi değişiklik yap; istenmeyen refactor yapma.
- Ayrıntılı kurallar: `.claude/rules/`. Tekrarlanan işler: `.claude/commands/`.
