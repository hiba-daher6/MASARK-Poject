# Masarak — مسارك

🌐 **الموقع:** [https://cool-tapioca-2797a1.netlify.app](https://cool-tapioca-2797a1.netlify.app)

منصّة تساعد الخريجين الجدد على مسارهم المهني بميزتين رئيسيتين:

1. **Job Match:** ترفع السيرة الذاتية (CV) ويطلع لك تحليل ووظائف مناسبة.
2. **AI Mock Interview:** مقابلة تجريبية بالصوت مع تقييم تفصيلي لإجاباتك.

---

## 📁 محتويات المشروع

| الملف | الوصف |
|---|---|
| `index.html` | الواجهة الأمامية (ملف واحد، عربي RTL) |
| `Done.json` | Workflow الـ n8n الخاص بتحليل الـ CV ومطابقة الوظائف |
| `AI_Mock_Interview.json` | Workflow الـ n8n الخاص بالمقابلة التجريبية |

---

## 🏗️ المعمارية

```
Frontend (index.html)
   │  fetch / POST
   ▼
n8n Webhooks ──► LLM (OpenAI gpt-4o-mini) ──► Code node (تنظيف JSON) ──► Respond to Webhook
```

---

## ⚙️ Workflow 1: AI Mock Interview

### Endpoint 1 — توليد الأسئلة
`POST /webhook/4d64a4f0-e8d4-4ec6-868b-9245fdf4ae7b`

**Body:**
```json
{
  "job_title": "Frontend Developer",
  "job_description": "...",
  "experience_level": "Fresh Graduate",
  "language": "English",
  "num_questions": "5"
}
```

**Response:**
```json
{
  "text": "1. ...\n2. ...",
  "questions": [{ "id": 1, "question": "...", "type": "Self-Introduction" }],
  "parse_error": false
}
```

- أول سؤال دايمًا تعريف بالنفس، والباقي خليط (Technical / Behavioral / Problem-Solving).
- الأسئلة قصيرة عشان تنقال بالصوت.

### Endpoint 2 — التقييم
`POST /webhook/interview-feedback`

**Body:**
```json
{
  "job_title": "...",
  "language": "...",
  "transcript": [{ "question": "...", "answer": "..." }]
}
```

**Response:**
```json
{
  "overall_score": 0,
  "summary": "...",
  "strengths": [],
  "improvements": [],
  "per_question": [
    { "question": "...", "score": 0, "feedback": "...", "suggested_answer": "..." }
  ],
  "next_steps": [],
  "parse_error": false
}
```

**قواعد التقييم:**
- الإجابات جاية من Speech Recognition، فبيتجاهل أخطاء الإملاء والتشكيل.
- الإجابة الفاضية = صفر.
- درجة كل سؤال من 10، والدرجة الكلية من 100.
- لا يعلّق على النبرة أو الثقة أو اللهجة.

---

## ⚙️ Workflow 2: Job Match (Done.json)

`POST /webhook/31e5c3f3-491a-4bb9-b943-272de1f46c08`

الخطوات تقريبًا: استقبال الـ CV ← استخراج النص (Extract from File) ← تحليل عبر HTTP Request لنموذج AI ← تنظيف الـ JSON بالـ Code ← حفظ صف في Google Sheets ← الرد على الواجهة.
الواجهة بتجيب النتيجة من `/webhook/result?id=...` لو الـ webhook رجّع `id` بدل النتيجة الكاملة.

---

## 🚀 طريقة التشغيل

1. **n8n:** استورد `Done.json` و `AI_Mock_Interview.json` (Workflows ← Import from File).
2. **Credentials:**
   - OpenAI API (للـ Mock Interview)
   - Google Sheets (لـ Job Match)
   - أي API Key يستخدمه الـ HTTP Request
3. **فعّل** الـ Workflows (Active).
4. **حدّث** إعدادات `CFG` داخل `index.html`:
   ```js
   API_BASE:            'https://<your-n8n>/webhook'
   JOBMATCH_URL:        'https://<your-n8n>/webhook/31e5c3f3-...'
   JOBMATCH_RESULT_URL: 'https://<your-n8n>/webhook/result'
   INTERVIEW_URL:       'https://<your-site>'
   ```
5. افتح `index.html` أو انشره (Netlify مثلًا).

> ⚠️ استخدم Chrome أو Edge عشان ميزة التعرّف على الصوت (Web Speech API).

---

## ⚠️ ملاحظات مهمة

- الـ Webhooks مفتوحة (`allowedOrigins: *`)، فيُفضّل تقييدها بدومين موقعك قبل النشر.
- الـ Webhook URLs الظاهرة داخل `index.html` تخص حسابات n8n Cloud خاصة، فاستبدلها بحساباتك ولا ترفعها على GitHub علني.
- لو الـ LLM رجّع JSON غير صالح، الـ Code node بيرجّع `parse_error: true` مع النص الخام، فتعامل معها في الواجهة.

---

## 🛠️ التقنيات

n8n • OpenAI (gpt-4o-mini) • HTML/CSS/JS • Web Speech API • Google Sheets • Netlify
