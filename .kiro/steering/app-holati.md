# ALC BioLab — Ilovaning Hozirgi Holati (Snapshot)

> Bu fayl ilovaning joriy arxitekturasi va asosiy funksiyalarini eslab qolish uchun.
> Oxirgi yangilangan: AI savol-javobni avtomatlashtirishni boshlashdan oldin.
> Branch: `colorful-redesign` | Asosiy fayl: `biologiya_ai_oqituvchi.html4444444.html` (bitta HTML fayl, hammasi ichida)

## Umumiy
- Nomi: **ALC BioLab — Biologiya AI Tizimi** (S.T. Biologiya, ALC O'quv Markazi).
- Til: o'zbekcha (lotin). Bitta `index`-siz HTML fayl.
- Tashqi kutubxonalar (CDN): `pdf.js`, `mammoth`, `Chart.js`, `jspdf 2.5.1`, `xlsx 0.18.5`, Google Fonts (Inter, Space Grotesk).
- Logotip: `LOGO` o'zgaruvchisi (base64 JPEG), `L1/L2/L3` `<img>` larga va favicon'ga o'rnatiladi.

## Navigatsiya
- `go(name)` — bo'limni almashtiradi (`.scr#SCR-<name>`).
- Bo'limlar: `chat` (AI), `history`, `analysis`, `parents`, `library`, `tests`, `repeat`, sessiya/dashboard.
- Har bo'limga kirishda tegishli render: `renderHistory`, `renderAnalysis`, `renderParents`, `renderLibrary`, `renderTests`, `renderRepeat`, `chat → updateApiStatus`.
- INIT (sahifa yuklanganda): `loadStudents(); loadLibrary(); initQA(); renderStudents(); renderVids("all"); updateApiStatus();`

## AI yordamchi (Claude)
- `callAI(prompt)` → `fetch("https://api.anthropic.com/v1/messages")` (`x-api-key`, `anthropic-version: 2023-06-01`, `anthropic-dangerous-direct-browser-access: true`).
- API kalit: `localStorage["alc_bio_key"]`, `sk-` bilan boshlanishi tekshiriladi.
- Funksiyalar: `getKey()`, `saveKey()`, `removeKey()`, `openModal(cb)`, `closeModal()`, `updateApiStatus()`, `PEND` (kalit kiritilgach davom etadigan callback).
- UI: modal `#API-MOD` (input `#API-IN`, o'chirish tugmasi `#API-DEL-BTN`), AI bo'limida `🔑 API kalit` tugmasi + holat badge `#API-STATUS` (✅ ulangan / ⚠️ kiritilmagan).
- Chat UI: `#CHAT-BOX`, `#CHAT-IN`, `sendChat()`.

## Savol-javob (sessiya) jarayoni — HOZIR QO'LDA BAHOLANADI
- `startSession()` → `openStuModal()` orqali o'quvchi tanlanadi (`selectStu(id)`, `APP.currentStu`).
- Holat: `APP = { qs:[], qi:0, ok:0, no:0, active, sec, ti, tts, res:[], students:[], currentStu }`.
- `loadQ(i)` — `APP.qs[i]` savolini ko'rsatadi va TTS bilan o'qiydi; `i>=qs.length` bo'lsa `endSession()`.
- `nextQ()` → `APP.qi++; loadQ(APP.qi)`.
- `showAnswer()` — to'g'ri javobni `#ANS-BOX/#ANS-TXT` da ko'rsatadi.
- **`markResult(true|false)` — o'qituvchi QO'LDA bosadi** (o'quvchini eshitib). `APP.ok` / `APP.no` ni oshiradi. Hisoblagichlar: `#C-OK`, `#C-NO`.
- Tugmalar: `#BTN-S` (Boshlash), `#BTN-A` (Javobni ko'rsat), `#BTN-OK`, `#BTN-NO`, `#BTN-NX` (Keyingi), `#BTN-E` (Yakunlash), `#BTN-M`.
- `#MIC-BOX` — mavjud, lekin yashirin (ovozli kiritish hali ulanmagan).
- `endSession()` — foizni hisoblaydi, o'quvchi statistikasini (`stu.avg`, `stu.ses`, `stu.stars`) yangilaydi, tarixga yozadi, `sendParentNotification(pct, tot)` chaqiradi, `go("results")`.
- `calcStars(pct)`, `updScore()`.
- Savol-javob muharriri: `DEF_QA`, `initQA()`, `addQRow(q,a)`, `#QA-ED`.

> 🔜 KELGUSI ISH: Ushbu QO'LDA baholashni AIga o'tkazish (bosqichma-bosqich):
> 1-bosqich: AI matnli javobni semantik baholaydi (ball/holat/izoh JSON), qo'lda tuzatish bilan.
> 2-bosqich: Ovoz→matn (Web Speech API `uz-UZ`).
> 3-bosqich: Kerak bo'lsa Whisper STT.

## O'quvchilar
- `APP.students`, `loadStudents()`, `renderStudents()`, `localStorage` da saqlanadi.
- O'quvchi maydonlari: `id, name, sinf, avg, stars, ses, last, av (avatar rang), ini (initsial)`.

## Telegram integratsiyasi
- Token: `localStorage["alc_tg_token"]` (`getTgToken`), o'qituvchi: `alc_teacher_name`.
- Hisobot uchun admin Chat ID: `alc_report_chat_id` (`getReportChatId`), UI `#REPORT-CHAT-ID`.
- Ota-ona Chat ID lari: `alc_parent_ids` (obyekt: `{studentId: chatId}`), `getParentIds()`, `saveParentIds()`.
- `sendTgMessage(chatId, text)` — matn (`sendMessage`).
- `sendTgDocument(blob, filename, caption)` — hujjat; **BARCHA ota-onalarga + admin** ga yuboradi.
- `getReportRecipients()` — ota-onalar + admin (takrorlarsiz). `sendTgDocumentTo(chatId, token, blob, filename, caption)` — bittaga.
- `sendParentNotification(pct, tot)` — sessiya yakunida `APP.currentStu` ota-onasiga natija.
- Log: `alc_tg_log`, `addTgLog`, `renderTgLog`, UI `#TG-LOG`, `#TG-STATUS`, ro'yxat `#PARENT-LIST`.
- UI: `#TG-TOKEN`, `#TEACHER-NAME`, `testTgBot()` (getMe).

## Excel / PDF eksport
- Profil tugmalari: O'quvchilar (`exportStudentsExcel`, `exportStudentsPDF`), Testlar (`exportTestsExcel`, `exportTestsPDF`), Natijalar (`exportPDF`).
- `collectTestRows()` — test natijalarini yig'adi.
- Ishonchli kutubxona yuklash: `getJsPDF()`, `loadScriptOnce(src)`, `ensureJsPDF()`, `ensureXLSX()` (CDN yuklanmasa qayta yuklaydi). Funksiyalar `async`.
- `pdfLogo(doc, x, y, size)` — PDF sarlavhasiga ALC logotipini qo'yadi (xavfsiz).
- Eksport: faylni yuklab oladi **VA** Telegramga yuboradi (`sendTgDocument`).

## Testlar bo'limi
- `saveTestResults()`, `loadTestResults()` (`alc_test_results`), `updTestStats()`.
- UI: `#TEST-DATE`, `#TEST-TOPIC`, `.test-row`, `#TEST-STATS`.

## localStorage kalitlari (xulosa)
- `alc_bio_key` — Anthropic API kalit
- `alc_tg_token` — Telegram bot token
- `alc_teacher_name` — o'qituvchi ismi
- `alc_report_chat_id` — hisobot uchun admin Chat ID
- `alc_parent_ids` — ota-ona Chat ID lari (JSON obyekt)
- `alc_tg_log` — yuborilgan xabarlar tarixi
- `alc_test_results` — test natijalari
- `alc_sb_collapsed` — sidebar holati
- (o'quvchilar/kutubxona uchun ham localStorage kalitlari bor)

## Git
- Branch: `colorful-redesign` (asosiy ish shu yerda).
- `main` bilan birlashtirish uchun PR hali yaratilmagan (so'ralsa yaratiladi).
