# Brainstorm Kuralları

## Aktif Brainstorm Kontrolü

Her konuşma başlangıcında `.claude/brain-storms/` dizininde `status: active` olan dosya var mı kontrol et. Varsa:

1. Dosyayı oku ve bağlamı anla
2. Kullanıcıya aktif brainstorm olduğunu bildir
3. Her mesaj döngüsünde dosyayı güncelle

## Canlı Tutma (Aktif Brainstorm Varken)

Her mesaj döngüsünde:
- **Mesaj alındığında:** Brainstorm dosyasını oku (context hatırlamak için)
- **Yanıt verdikten sonra:** Yeni kararları, reddedilen fikirleri, nedenleri dosyaya işle

### Dosyada ne olmalı:
- Sadece "X kararı alındı" DEĞİL → "X önerildi, Y nedeniyle reddedildi, Z nedeniyle X'in modifiye hali kabul edildi"
- Kullanıcının exact ifadeleri (önemli yerlerde) — ruh orada
- Açık kalan sorular ve sonraki adımlar
- Kronolojik akış — hangi fikir hangisinden sonra geldi

### Amaç:
Brainstorm dosyası, yeni bir context açıldığında okuyan Claude'un sanki o konuşmada oradaymış gibi devam edebileceği kadar detaylı ve ruhlu olmalı.

## Döküman Zinciri

Her tartışma ve karar üç katmanlı zinciri izler:

```
brain-storms/ (süreç) → docs/ (sonuç) → CLAUDE.md (özet)
                     ↘
                       backlog.md (sonraya bırakılanlar)
```

- Brainstorm olmadan karar alınmaz
- Brainstorm dosyaları silinmez (tarihsel kayıt)
- Kararlar değişirse yeni brainstorm açılır, eskisine "superseded by X" notu eklenir

## Backlog Disiplini

Brainstorm sırasında "şu an yapmıyoruz, sonra" denilen her madde **`.claude/backlog.md`**'ye yansıtılmalı. Bu, scope creep'i önlemek için kritik — şimdi yapmadığımızı kayda alıyoruz ki ileride feature ihtiyacı doğduğunda hatırlayalım.

### Ne zaman backlog'a eklenir:
- Brainstorm sırasında bir alt-konu için "bu prematüre, sonraya bırakalım" denildiğinde → **anında** backlog'a yaz, "sonra" deme
- Bir karar maddesi explicitly "bu adıma dahil değil" olarak işaretlendiğinde
- Tier 3 olarak işaretlenen ve tek bir cümle olmayan, kalıcı kayıt gerektiren konular
- Geliştirme sırasında "bunu sonra yaparız" diye not düşülen her şey

### Format:
- **Prepend** (yeni en üstte) — `.claude/backlog.md` dosyasının başına eklenir, eskiler aşağıda kalır
- Her madde için: tarih + kategori başlık + bağlam linki + detaylı konu açıklaması + "ne zaman gündeme gelir" notu + ilgili kaynaklar
- Dosyanın başındaki şablonu izle

### Brainstorm done sırasında zorunlu kontrol:
`/brainstorm done` çağrıldığında, brainstorm dosyasındaki tüm "sonraya bırakılan" notları tarayıp **her birinin backlog'da karşılığı olduğundan emin ol**. Eksik varsa kullanıcıya sor ve ekle. Bu, brainstorm'u kapatmadan önce bir checklist adımı.

### Backlog'dan çıkarma:
Bir madde implement edildiğinde backlog'dan **silinir** (done diye işaretlenip bırakılmaz). Çünkü artık aktif altyapının parçası, ilgili docs/CLAUDE.md güncellenir.
