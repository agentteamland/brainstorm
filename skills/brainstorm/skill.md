---
name: brainstorm
description: Beyin fırtınası başlatma ve tamamlama. start = yeni brainstorm başlat, done = aktif brainstorm'u tamamla ve döküman zincirine işle. --global ile tüm projelere ait konularda ~/.claude/ altında çalışır.
argument-hint: "<start|done> [--global] [ilk mesaj]"
---

# /brainstorm Skill

## Parametre Ayrıştırma

İlk kelime modu belirler: `start` veya `done`. Geri kalan metin (varsa) kullanıcının ilk mesajıdır.

**`--global` flag:** Argümanlarda `--global` varsa, brainstorm dosyası `~/.claude/brain-storms/` altında oluşturulur (tüm projeler arası ortak konular için). Flag yoksa `.claude/brain-storms/` altında oluşturulur (proje-özel).

**Kapsam belirleme:**
- `~/.claude/brain-storms/` → Global: altyapı framework'ü, ortak skill/agent tasarımı, projeler arası kararlar
- `.claude/brain-storms/` → Proje: o projeye özel feature, mimari, teknik kararlar

---

## `start` Modu

Kullanıcı `/brainstorm start ...` dediğinde:

### 1. Konuyu Anla
Kullanıcının mesajından konuyu çıkar. Konu başlığını kullanıcı vermez — sen mesajdan anlarsın ve uygun bir `kebab-case` dosya ismi belirlersin.

### 2. Kapsam Belirle
- `--global` varsa → `base_dir = ~/.claude/`
- `--global` yoksa → `base_dir = .claude/` (proje kökü)

### 3. Dosya Oluştur
`{base_dir}/brain-storms/{isim}.md` dosyasını oluştur:

```markdown
---
status: active
scope: {global|project}
date: {bugünün tarihi}
participants: Mesut, Claude
---

# {Konu Başlığı}

## Bağlam

{Kullanıcının ilk mesajından anlaşılan bağlam ve motivasyon}

## Tartışma

### {Tarih} — Başlangıç

{İlk mesajın özeti ve varsa ilk fikirler}

## Açık Konular

- {İlk mesajdan çıkan sorular veya tartışılması gereken noktalar}
```

### 4. Yanıt Ver
Kullanıcıya brainstorm'un başladığını, dosya ismini ve kapsamını (global/project) bildir ve konuya gir.

### 5. Sonraki Mesajlarda
Her mesaj döngüsünde brainstorm dosyasını güncelle:
- Yeni fikirler, kararlar, reddedilen alternatifler ve nedenleri ekle
- Kullanıcının önemli ifadelerini aynen koru (tırnak içinde)
- Kronolojik sırayı koru — tartışma bölümüne yeni alt başlıklar ekle
- Açık konular listesini güncelle (çözülenler kaldırılır, yeniler eklenir)

**ÖNEMLİ:** Dosya o kadar detaylı olmalı ki, yeni bir context'te okuyan Claude sanki o konuşmada oradaymış gibi devam edebilsin. Sadece kararları değil, kararların arkasındaki mantığı, reddedilen alternatifleri ve nedenlerini de yaz.

---

## `done` Modu

Kullanıcı `/brainstorm done` dediğinde:

### 1. Aktif Brainstorm'u Bul
**Her iki konumda da** ara:
- `~/.claude/brain-storms/` (global)
- `.claude/brain-storms/` (proje)

`status: active` olan dosyaları bul. Birden fazla varsa kullanıcıya listele (hangi scope'ta olduğunu da göster) ve hangisini tamamlamak istediğini sor. Tek aktif varsa onu tamamla. Hiç yoksa kullanıcıya bildir.

### 2. Brainstorm Dosyasını Tamamla
- `status: active` → `status: completed` yap
- Tartışma bölümünün sonuna son notları ekle
- "Açık Konular" bölümünü güncelle (çözülmemiş olanlar kalır)
- "Kesinleşen Kararlar" bölümü ekle — tüm tartışmadan çıkan net kararların özeti

### 3. Docs Dosyası Oluştur/Güncelle
Brainstorm'un scope'una göre docs dosyasının konumunu belirle:
- **Global brainstorm** → `~/.claude/docs/` altına yaz
- **Proje brainstorm** → `.claude/docs/` altına yaz

Dosyanın başında brainstorm'a referans ver: `> Brainstorm kaynağı: [dosya-adı](../brain-storms/dosya-adı.md)`

### 4. CLAUDE.md Güncelle
- **Global brainstorm** → `~/.claude/CLAUDE.md` güncelle
- **Proje brainstorm** → Proje kökündeki `CLAUDE.md` güncelle

### 5. Yanıt Ver
Kullanıcıya brainstorm'un tamamlandığını, oluşturulan/güncellenen dosyaları bildir.

---

## Önemli Kurallar

1. **Birden fazla aktif brainstorm olabilir.** Her biri kendi dosyasında bağımsız yaşar. Global ve proje brainstorm'ları aynı anda aktif olabilir.
2. **Context kırılmasına dayanıklılık.** Brainstorm dosyası persistent state'tir. Yeni context'te rule sayesinde aktif brainstorm algılanır ve dosya okunarak devam edilir.
3. **Dosya ismi kullanıcıdan istenmez.** Sen mesajdan anlarsın ve uygun kebab-case isim verirsin.
4. **Brainstorm dosyaları asla silinmez.** Tarihsel kayıt olarak kalır.
5. **Her brainstorm tek konuya odaklanır.** Farklı konular farklı dosyalarda.
6. **Aktif brainstorm araması her iki konumda da yapılır.** `done` modunda hem global hem proje dizini taranır.
7. **Scope frontmatter'da belirtilir.** `scope: global` veya `scope: project` — bu, done modunda doğru docs/CLAUDE.md hedefini belirler.
