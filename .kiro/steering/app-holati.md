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


---

## YANGILANISH: AI savol-javob (1 & 2-bosqich) bajarildi

### 1-bosqich (AI semantik baholash)
- `#MIC-TXT` endi tahrirlanadigan `textarea` (yozish + ovoz).
- `aiGrade()` — Claude'ga savol+to'g'ri javob+o'quvchi javobi yuboradi, JSON qaytaradi: `{ball, holat, izoh}`.
- `parseAIEval(raw)` — JSON ni robust parse qiladi. `showEval(ev)` — `#AI-EVAL`/`#AI-EVAL-TXT` da rangli ball+izoh.
- Tugma `#BTN-AI` ("AI baholash"). `markResult` natijaga `ball/holat/izoh/stuAns` qo'shadi (`APP.res` ga, hisobotlarda ko'rinadi).
- Natija ekranida (`renderResult`) har savolga AI bali, o'quvchi javobi, izoh ko'rsatiladi.

### 2-bosqich (Avto rejim — ovozli, avtomatik)
- `APP.autoAI` (yoq/o'chiq), `APP.autoTimer`. Tugma `#AUTO-BTN` (`toggleAutoAI`, `refreshAutoBtn`).
- Mic `REC.onend` → avto rejimda javob bo'lsa `aiGrade()` o'zi chaqiriladi.
- `startAutoConfirm(sok)` — 4s hisob, keyin `markResult` avtomatik (o'qituvchi tugma bossa bekor).
- `autoStartMic()` — keyingi savolda TTS o'qib bo'lgach mikrofonni avto yoqadi (avto-loop).
- Baho threshold: `ball>=60` → to'g'ri.

### Ma'lum cheklov / kelgusi
- O'zbek STT (Web Speech `uz-UZ`) sifati cheklangan — 3-bosqichda Whisper STT ko'rib chiqilishi mumkin.
- Avto rejimda TTS+mikrofon aks-sadosi bo'lishi mumkin (naushnik tavsiya etiladi).
- Umumiy o'zlashtirish hozircha to'g'ri/noto'g'ri sanog'i (`pct`) asosida; kelajakda AI ballari o'rtachasi bilan aniqroq qilish mumkin.


### 3-bosqich (AI sessiya tahlili va o'zlashtirish darajasi)
- `computeAiMastery()` — `APP.res` dagi AI ballarining o'rtachasi (mavjud bo'lsa).
- `aiSessionAnalysis()` — sessiya yakunida Claude butun natijani tahlil qiladi, JSON qaytaradi: `{daraja: Yuqori/O'rta/Past, ball, kuchli, zaif, tavsiya}` (o'zbekcha).
- `parseSessionAnalysis()`, `renderSessionAnalysis()` — natija ekranidagi `#RES-AI` / `#RES-AI-BODY` kartasida ko'rsatadi. Tugma `#RES-AI-BTN`.
- API kalit bo'lsa, natija ekranida avtomatik ishga tushadi (`renderResult` oxirida).
- `saveAnalysisToHistory()` — tahlil va `aiMastery` ni oxirgi sessiya tarixiga (`alc_bio_history` h[0]) saqlaydi.
- Eslatma: Telegram ota-ona xabari `endSession`da darrov ketadi (AI tahlil async, unga ulanmagan). Kelajakda: tahlilni kutib, xabarga qo'shish mumkin.


### YANGILANISH: AI tahlil -> Telegram & PDF, stu.avg AI ball asosida
- `endSession` endi `computeAiMastery()` ni hisoblab, `stu.avg` ni AI ball o'rtachasi asosida yangilaydi (AI ball bo'lmasa, pct ga qaytadi).
- `finishSessionAI(pct, tot, aiM)` — AI tahlilni (`aiSessionAnalysis()` endi obyekt qaytaradi) kutib, so'ng `sendParentNotification(pct, tot, summary, aiM)` chaqiradi.
- `sendParentNotification` ota-ona xabariga "AI ball o'rtachasi" + AI tahlil (daraja/kuchli/zaif/tavsiya) qo'shadi.
- `exportPDF` (sessiya PDF) ga "AI o'zlashtirish tahlili" bo'limi qo'shildi (`APP.aiSummary` + `computeAiMastery`); maxsus apostroflar ASCII ga tozalanadi (helvetica uchun).
- `saveSessionToHistory` entry'ga `aiMastery` qo'shildi.
- `renderResult` dagi avto-trigger olib tashlandi (endi `finishSessionAI` boshqaradi).
