# video-effects

[English](README.md) | **Azərbaycanca**

Şaquli (1080x1920, 30 fps) "danışan baş" videoları üçün 21 hərəkətli qrafika effekti. Layihə Remotion ilə qurulub. Yanında bir Claude skill-i də var: o, çəkilişi hazırlamağı, deyilən sözlərə uyğun effekt seçməyi, montaj siyahısını (cut list) yazmağı və videonu render etməyi bilir.

Görünüş boz fon üzərində açıq rəngli "şüşə" kartlardan və yumşaq, sıçramayan (critically damped) yay animasiyalarından ibarətdir. Bəzi effektlər danışanı otaqdan kəsib çıxarır və qrafikanı onun arxasına yerləşdirir. Repozitoriyada adi bir çəkilişdən bu kəsikləri (cut-out) hazırlayan proses də var.

## Claude skill-i kimi istifadə

Repozitoriyanı `~/.claude/skills/video-effects` (və ya layihənin `.claude/skills/video-effects`) qovluğuna klonlayın, sonra Claude-dan videonuzu montaj etməsini istəyin. `SKILL.md` Claude-un izlədiyi təlimatdır: quraşdırma, çəkilişin hazırlanması, deyilən sözlərə görə effekt seçimi, `src/edit/edit.json` faylının yazılması, kadrların yoxlanılması və render.

## Sürətli başlanğıc

```sh
npm install
pip install -r requirements.txt   # numpy, pillow
npm run sample                    # public/sample/ qovluğunda süni çəkiliş + maskalar + kəsiklər (ffmpeg lazımdır)
npm run studio                    # Remotion Studio: FxShowcase, FxPreview, ExampleEdit
npm run render:showcase           # out/showcase.mp4, 21 effektin hamısı ardıcıl
```

Digər skriptlər: `npm run render:edit` faylı `src/edit/edit.json`-u `out/edit.mp4`-ə render edir; `npm run stills -- <qovluq> <effektId> [from] [kadr,kadr]` bir effektin ayrı kadrlarını çəkir; `npm run bench` hər effektin sürətini ölçür; `npm run typecheck` tipləri yoxlayır.

## Öz videonuz

```sh
scripts/prepare-take.sh path/to/input.mov myname    # public/takes/myname/ qovluğunu yaradır
```

Proses belədir: ffmpeg videonu 1080x1920, 30 fps formatına çevirir (kənarları kəsməklə), `tools/matte` Apple Vision ilə hər kadr üçün insan maskası yaradır, `scripts/make-fg.py` maskaları hamarlanmış, yumşaq kənarlı RGBA kəsiklərə çevirir, `scripts/make-sticker-mattes.py` Paper Cutout effektinin istifadə etdiyi stiker kənarını hazırlayır, `tools/face` üzün yerini tapır və skript `src/edit/edit.json`-a yapışdırmaq üçün `public/takes/myname/source.json` faylını yazır.

Maska və üz addımları Apple-ın Vision kitabxanasından istifadə edir, ona görə yalnız macOS-da işləyir (Xcode komanda xətti alətlərindəki `swiftc` ilə kompilyasiya olunur). Başqa sistemlərdə maskaları istənilən seqmentasiya aləti ilə hazırlayın (çəkilişin ölçüsündə `0000.png`, `0001.png`, ... adlı boz PNG-lər, insan ağ rəngdə), `make-fg.py`-dən davam edin və `face` dəyərini bir kadra baxaraq əl ilə yazın. Təfərrüatlar `SKILL.md`-dədir.

`public/sample/` və `public/takes/` gitignore-dadır. Öz videonuzdan render etdiyiniz heç nə git-də izlənmir.

## Montaj formatı

`src/edit/edit.json` bir çəkiliş üzərində montaj siyahısıdır: `source` (video, kəsiklər qovluğu, stiker qovluğu, kadr sayı, üzün yeri) və ardıcıl oynayan `segments`. Hər seqment ya adi görüntüdür (`aroll`), ya da effektdir (`fx` və `id`). O, mənbədəki `at` kadrından başlayır və `frames` qədər davam edir. Səs hər seqmentdə çəkilişin öz audiosundan gəlir, ona görə fasiləsiz çəkilişdə hər seqment əvvəlkinin bitdiyi yerdən başlayır. Effektlərin öz səs effektləri var; istəyə görə hər şeyin altında fon musiqisi (`music`) ola bilər.

## 21 effekt

| | id | Nə edir |
|--|----|---------|
| 01 | `orbit` | Proqram ikonlarından ibarət halqa danışanın başı ətrafında, arxada və önündə fırlanır |
| 02 | `chat-takeover` | Video söhbət pəncərəsində qoşmaya çevrilir; sual, cavab gəlir, addımlar işarələnir |
| 03 | `frame-escape` | Video əyilmiş çap şəklinə çevrilir, danışanın başı kənarından çölə çıxır |
| 04 | `snap-zoom` | Yavaş yaxınlaşma, sonra üzə kəskin zoom |
| 05 | `checklist-pop` | Danışanın yanında dörd şüşə kapsul görünür və yaşıl işarələnir |
| 06 | `headline-storm` | Başlıq bannerləri hər iki tərəfdən gəlib danışanın ətrafında yığılır |
| 07 | `zoom-portal` | Dairəvi pəncərə açılır; kamera onun içindən keçib yaxınlaşır |
| 08 | `growth-curve` | Danışanın arxasında xətti qrafik çəkilir, sayğac 10x-ə qədər qalxır |
| 09 | `broll-overlay` | Video montaj proqramının pəncərəsinə kiçilir; V2-yə klip düşür və üstə keçir |
| 10 | `paper-cutout` | Danışan kağız posterdə ağ kənarlı stikerə çevrilir |
| 11 | `magazine-cover` | Danışanın ətrafında jurnal başlığı, yazılar, ştrix-kod və nişan peyda olur |
| 12 | `liquid-glass` | Şüşə damcı sürüşüb gəlir, üzü böyüdür və gedir |
| 13 | `zoom-through-cut` | Bulanıq zoom ilə yaxınlaşma, bulanıqlıqda kəsik, geri uzaqlaşma |
| 14 | `green-screen` | Danışanın arxasına yaşıl fon gəlir, sonra boz səhnə qalxır |
| 15 | `frame-break` | Boz səhnə çatlayıb sınır və otaq görünür |
| 16 | `digital-zoom` | Vizör görünüşü; kadr pillə-pillə üzə yaxınlaşır və geri qayıdır |
| 17 | `phone-mockup` | Video qısa video interfeysli telefon ekranına kiçilir |
| 18 | `checklist-tally` | Bağlanış kartı: sayğac və 21 bəndlik siyahı işarələnir |
| 19 | `glass-widgets` | Danışanın ətrafında, arxada və önündə beş buzlu vidjet qalxır; rəqəmlər artır, halqa dolur, render zolağı tamamlanır |
| 20 | `checklist` | Sinə hissəsinin üstündə tünd kart qalxır və beş addımı işarələyir: gözləyir, işləyir, hazırdır |
| 21 | `split` | Kağız qrafik yuxarı yarıya sürüşür və kadrı aşağı itələyir; sütunlar böyüyür, sonra orta xətt çəkilir |

Hər effekt `src/fx/<Ad>.tsx` faylındadır və başlıq şərhində hərəkəti və səs effektləri təsvir olunub. Ümumi müqavilə (`FxSource`, `ARoll`, `Cutout`, `useFace`, yaylar) `src/fx/shared.tsx`-dədir; mövzu (rənglər, `GlassCard`, `Backdrop`) `src/fx/theme.tsx`-dədir; səs kitabxanası `src/fx/sfx.tsx`-dədir.

Bəzi effektlərdəki ekran yazıları Claude ilə montaj haqqında bir demo üçün yazılıb. Bu yazılar hər effekt faylında adi mətn kimi durur; onları öz videonuza uyğun dəyişin, məsələn Azərbaycan dilinə çevirin.

## Lisenziyalar

- Kod: `LICENSE` faylına baxın.
- Şriftlər: `public/fonts/` qovluğundakı Inter, Montserrat və Caveat SIL Open Font License 1.1 altındadır (`public/fonts/OFL.txt`).
- `public/sfx/` qovluğundakı səs effektləri bu layihə üçün ElevenLabs ilə yaradılıb və bu effektlərlə istifadə üçün daxil edilib.
- Remotion-un öz lisenziyası var. Fərdlər və kiçik şirkətlər onu pulsuz istifadə edə bilər; müəyyən ölçüdən böyük şirkətlərə Remotion şirkət lisenziyası lazımdır. Bax: https://remotion.dev/license.

## PostZen ilə paylaşın

Claude videonu montaj etdikdən sonra PostZen Claude-a onu paylaşmağa imkan verir. PostZen Claude-u rəsmi API-lər vasitəsilə sosial hesablarınıza qoşan MCP serveridir: paylaşım etmək, planlaşdırmaq, növbəyə qoymaq və ya qaralama yaratmaq (Instagram Reels, şəkillər, karusellər və hekayələr; LinkedIn və digər şəbəkələr), şərhlərə və mesajlara cavab vermək, şərhdən mesaja avtomatlaşdırmalar qurmaq və statistikaya baxmaq. Bir komanda ilə əlavə olunur:

```sh
claude mcp add --transport http postzen https://mcp.postzen.dev/mcp
```

https://www.postzen.dev/?utm_source=github&utm_medium=skill&utm_campaign=video-effects-skill. Agent üçün sürətli başlanğıc: https://www.postzen.dev/agent-quickstart.md.

Müəllif: Jesse Eisenbart, https://jesseeisenbart.com.
