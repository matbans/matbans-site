---
description: Push öncesi hızlı kontrol
---
Push etmeden önce şunları kontrol et ve kısa raporla:
1. `git status` ve `git diff` — beklenmeyen değişiklik var mı?
2. `index.html` içindeki tüm dış linkler (Cal.com, Stripe, WhatsApp, marka linkleri) hâlâ doğru mu?
3. `rel="noopener"` eksik `target="_blank"` var mı?
4. Title, meta description ve gtag etiketi yerinde mi?
Sorun yoksa commit mesajı öner; push'u kullanıcı onaylamadan yapma.
