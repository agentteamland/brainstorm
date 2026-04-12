---
name: brainstorm
description: "Beyin fırtınası başlatma ve tamamlama. start = yeni brainstorm başlat, done = aktif brainstorm'u tamamla ve döküman zincirine işle. 3 kapsam: proje (varsayılan), --global, --team."
argument-hint: "<start|done> [--global|--team] [ilk mesaj]"
---

# /brainstorm Skill

## Parametre Ayrıştırma

İlk kelime modu belirler: `start` veya `done`. Geri kalan metin (varsa) kullanıcının ilk mesajıdır.

## Üç Kapsam

| Flag | Hedef Dizin | Ne Zaman |
|------|------------|----------|
| *(hiçbiri)* | `.claude/brain-storms/` | Bu projeye özel konular (varsayılan) |
| `--global` | `~/.claude/brain-storms/` | Projeler arası, kişisel konular |
| `--team` | `~/agent-teams/{team}/brain-storms/` | Team repo'suyla ilgili konular (agent kuralı, team stratejisi) |

**`--team` aktif team tespiti:**
- `~/.claude/agents/` altındaki symlink'lerin `readlink` sonucundan `~/agent-teams/{team-name}/` çıkarılır
- Tek team varsa otomatik kullanılır
- Birden fazla team varsa AskUserQuestion ile sorulur
- Hiç team yoksa hata: "Kurulu team bulunamadı. Önce /team install çalıştırın."

---

## `start` Modu

Kullanıcı `/brainstorm start ...` dediğinde:

### 1. Konuyu Anla
Kullanıcının mesajından konuyu çıkar. Konu başlığını kullanıcı vermez — sen mesajdan anlarsın ve uygun bir `kebab-case` dosya ismi belirlersin.

### 2. Kapsam Belirle
- `--global` varsa → `base_dir = ~/.claude/`
- `--team` varsa → `base_dir = ~/agent-teams/{team-name}/` (aktif team tespit et)
- İkisi de yoksa → `base_dir = .claude/` (proje kökü)

### 3. Dizini Oluştur (yoksa)
`{base_dir}/brain-storms/` dizini yoksa oluştur.

### 4. Dosya Oluştur
`{base_dir}/brain-storms/{isim}.md` dosyasını oluştur:

```markdown
---
status: active
scope: {project|global|team}
team: {team-name veya null}
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

### 5. Yanıt Ver
Kullanıcıya brainstorm'un başladığını, dosya ismini ve kapsamını bildir ve konuya gir.

### 6. Sonraki Mesajlarda
Her mesaj döngüsünde brainstorm dosyasını güncelle:
- Yeni fikirler, kararlar, reddedilen alternatifler ve nedenleri ekle
- Kullanıcının önemli ifadelerini aynen koru (tırnak içinde)
- Kronolojik sırayı koru — tartışma bölümüne yeni alt başlıklar ekle
- Açık konular listesini güncelle (çözülenler kaldırılır, yeniler eklenir)

**ÖNEMLİ:** Dosya o kadar detaylı olmalı ki, yeni bir context'te okuyan Claude sanki o konuşmada oradaymış gibi devam edebilsin.

---

## `done` Modu

Kullanıcı `/brainstorm done` dediğinde:

### 1. Aktif Brainstorm'u Bul
**Üç konumda da** ara:
- `.claude/brain-storms/` (proje)
- `~/.claude/brain-storms/` (global)
- `~/agent-teams/*/brain-storms/` (tüm team'ler)

`status: active` olan dosyaları bul. Birden fazla varsa kullanıcıya listele (hangi scope'ta olduğunu da göster) ve hangisini tamamlamak istediğini sor.

### 2. Brainstorm Dosyasını Tamamla
- `status: active` → `status: completed` yap
- Tartışma bölümünün sonuna son notları ekle
- "Açık Konular" bölümünü güncelle (çözülmemiş olanlar kalır)
- "Kesinleşen Kararlar" bölümü ekle — tüm tartışmadan çıkan net kararların özeti

### 3. Docs Dosyası Oluştur/Güncelle
Brainstorm'un scope'una göre docs dosyasının konumunu belirle:
- **Proje brainstorm** → `.claude/docs/` altına yaz
- **Global brainstorm** → `~/.claude/docs/` altına yaz
- **Team brainstorm** → `~/agent-teams/{team}/docs/` altına yaz

Dosyanın başında brainstorm'a referans ver.

### 4. CLAUDE.md / README Güncelle
- **Proje brainstorm** → Proje kökündeki `CLAUDE.md` güncelle
- **Global brainstorm** → `~/.claude/CLAUDE.md` güncelle
- **Team brainstorm** → `~/agent-teams/{team}/README.md` güncelle

### 5. Team Brainstorm İse Git Push
Team scope'unda tamamlanan brainstorm sonrası:
```bash
cd ~/agent-teams/{team-name}
git add -A
git commit -m "brainstorm: {konu özeti}"
git push
```

### 6. Yanıt Ver
Kullanıcıya brainstorm'un tamamlandığını, oluşturulan/güncellenen dosyaları bildir.

---

## Önemli Kurallar

1. **Birden fazla aktif brainstorm olabilir.** Her biri kendi dosyasında bağımsız yaşar. Farklı scope'larda aynı anda aktif olabilir.
2. **Context kırılmasına dayanıklılık.** Brainstorm dosyası persistent state'tir. Yeni context'te rule sayesinde aktif brainstorm algılanır ve dosya okunarak devam edilir.
3. **Dosya ismi kullanıcıdan istenmez.** Sen mesajdan anlarsın ve uygun kebab-case isim verirsin.
4. **Brainstorm dosyaları asla silinmez.** Tarihsel kayıt olarak kalır.
5. **Her brainstorm tek konuya odaklanır.** Farklı konular farklı dosyalarda.
6. **Aktif brainstorm araması üç konumda da yapılır.** `done` modunda proje + global + tüm team dizinleri taranır.
7. **Scope frontmatter'da belirtilir.** `scope: project|global|team`, `team: {name}` — done modunda doğru hedefi belirler.
8. **Team brainstorm'da otomatik git push.** Done sonrası commit + push yapılır.
