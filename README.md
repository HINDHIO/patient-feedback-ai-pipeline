<div align="center">

# 🏥 Patient Feedback AI Pipeline

**Google Forms · Make.com · OpenAI GPT-4o · Google Sheets · Gmail**

[![Make.com](https://img.shields.io/badge/Make.com-Automation-6633CC?style=flat-square&logo=make&logoColor=white)](https://make.com)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-412991?style=flat-square&logo=openai&logoColor=white)](https://openai.com)
[![Google Sheets](https://img.shields.io/badge/Google_Sheets-Dashboard-34A853?style=flat-square&logo=googlesheets&logoColor=white)](https://sheets.google.com)
[![Google Forms](https://img.shields.io/badge/Google_Forms-Collection-673AB7?style=flat-square&logo=googleforms&logoColor=white)](https://forms.google.com)
[![Gmail](https://img.shields.io/badge/Gmail-Urgent_Alerts-EA4335?style=flat-square&logo=gmail&logoColor=white)](https://gmail.com)

A no-code automation pipeline that collects patient feedback, uses GPT-4o to
analyse sentiment and urgency in real time, logs structured results to a live
dashboard, and instantly alerts staff when critical feedback is detected —
**zero manual review required.**

</div>

---

## 📸 Screenshots

| Patient Form | Make.com Automation |
|:---:|:---:|
| ![Form](<img width="1887" height="785" alt="image" src="https://github.com/user-attachments/assets/02e52be8-b475-468b-9ba7-cc927ee10429" />
) | ![Make Scenario](<img width="1801" height="773" alt="image" src="https://github.com/user-attachments/assets/f2ab49d0-9cf3-41f9-a18b-452bb94bc75b" />
) |
| **Live Dashboard** | **AI Output Detail** |
| ![Dashboard](!(<img width="1919" height="785" alt="image" src="https://github.com/user-attachments/assets/568959e2-61a6-4139-bc73-83a7c4089894" />
)
) | ![AI Output](<img width="1916" height="661" alt="image" src="https://github.com/user-attachments/assets/a3825a17-7474-46ab-9d76-d941effe4a34" />
) |

---

## 🔍 What It Does

Hospital administrators receive dozens of patient feedback submissions daily.
Manual review is slow, inconsistent, and risks missing urgent complaints.

This pipeline eliminates that bottleneck entirely — from form submission to
structured dashboard entry takes under 10 seconds with no human intervention.

---

## 🔄 Pipeline Architecture

```
Patient submits Google Form (12 questions)
        │
        ▼
Google Sheets receives response  ←  Make.com watches for new rows
        │
        ▼
Make.com extracts free-text feedback field → OpenAI GPT-4o
        │
        │   System prompt enforces strict JSON output only:
        │   {
        │     "sentiment": "positive | neutral | negative",
        │     "urgency":   "high | low",
        │     "category":  "Billing | Waiting Time | Staff Conduct | Medical Care",
        │     "summary":   "1-sentence plain-English summary"
        │   }
        │
        ▼
Make.com parses JSON → 4 structured fields
        │
        ├──► IF urgency = high  OR  sentiment = negative
        │         └── Gmail sends urgent alert to patient contact email
        │              Subject: "URGENT: Patient Feedback Requires Attention"
        │
        └──► ALWAYS
                  └── Write structured row to Dashboard sheet
                        Timestamp · Department · Sentiment · Urgency
                        Category · Overall Rating · AI Summary · Processed
```

---

## 📊 Real Output — Dashboard Sample

These are actual AI-processed results from test submissions:

| Department | Sentiment | Urgency | Category | Rating | AI Summary |
|---|---|---|---|---|---|
| Emergency Department | 🔴 negative | 🚨 high | Waiting Time | 1/5 | Long wait and poor communication; patient felt neglected and unsafe |
| Cardiology | 🟡 neutral | 🟢 low | Waiting Time | 4/5 | Good result communication but longer than expected wait time |
| Radiology | 🔴 negative | 🟢 low | Staff Conduct | 3/5 | Concerns about communication difficulties due to language barriers |
| Pediatrics | 🟢 positive | 🟢 low | Staff Conduct | 5/5 | Kind and communicative paediatrician; clean clinic despite reception delay |

The first row (Emergency, Rating 1, urgency high) triggered an **automatic Gmail alert**
to the patient's provided contact email within seconds of submission.

---

## 📋 Google Form Structure

The form collects 12 fields across structured and free-text inputs:

| # | Question | Type |
|---|---|---|
| 1 | Department / Service Visited | Dropdown |
| 2 | Type of Visit | Multiple choice |
| 3 | Date of Visit | Date |
| 4 | Overall Experience Rating | Scale 1–5 (Very Poor → Excellent) |
| 5 | Waiting Time Satisfaction | Scale 1–5 |
| 6 | Staff Communication Quality | Scale 1–5 |
| 7 | Treatment Quality Satisfaction | Scale 1–5 |
| 8 | **Open-ended feedback** ← *this field is sent to GPT-4o* | Long text |
| 9 | Did you experience any of the following? | Checkbox |
| 10 | Would you like a follow-up? | Yes / No |
| 11 | Optional contact email | Email |
| 12 | Main issue category | Dropdown |

---

## ✨ Key Design Decisions

**Strict JSON enforcement — two layers**
The GPT-4o system prompt instructs the model to return raw JSON only with no
markdown or conversational text. `response_format: json_object` is also set
at the API level — making malformed output structurally impossible.

**Conditional routing, not sequential**
The Make.com router runs both paths independently. A failed email alert never
blocks the dashboard write, and vice versa.

**OR logic on the alert trigger**
The Gmail alert fires if urgency is `high` **OR** sentiment is `negative` —
not both. This catches cases where patient tone is clearly distressed even
when urgency alone would not trigger, maximising safety coverage.

**Unstructured → structured in one step**
A free-text field of any length in any language is transformed into four
clean, consistent, queryable columns that can be filtered, sorted, and
charted without any manual processing.

---

## 🗂️ Dashboard Column Reference

| Column | Source | Values |
|---|---|---|
| Timestamp | Make.com `now` | ISO 8601 datetime |
| Department | Form field 1 | Emergency, Cardiology, Radiology, Pediatrics… |
| Sentiment | GPT-4o | positive / neutral / negative |
| Urgency | GPT-4o | high / low |
| Category | GPT-4o | Waiting Time / Staff Conduct / Billing / Medical Care |
| Rating (overall) | Form field 4 | 1–5 |
| Summary | GPT-4o | 1-sentence plain-English summary |
| Action Required | Make.com | TRUE when AI processing completed |

---

## 🤖 GPT-4o System Prompt

```
You are a medical triage assistant. Analyze the patient feedback provided.

Instructions:
sentiment : Classify as 'positive', 'neutral', or 'negative'.
urgency   : Classify as 'high' (needs immediate attention) or 'low'.
category  : Choose the most relevant department or issue
            (e.g. Billing, Waiting Time, Staff Conduct, Medical Care).
summary   : Write a 1-sentence summary of the main point.

Output Format:
Return ONLY a raw JSON object with this exact structure:
{"sentiment": "text", "urgency": "text", "category": "text", "summary": "text"}

Do not include conversational text, markdown formatting, or code blocks.
```

---

## 📂 Repository Structure

```

├── blueprint.json            # Make.com scenario — importable directly
└── README.md
```

---

## 🚀 How to Reproduce This

### Prerequisites
- [Make.com](https://make.com) account (free tier sufficient)
- [OpenAI API key](https://platform.openai.com/api-keys)
- Google account (Forms + Sheets + Gmail)

### Steps

**1. Create your Google Sheets**
Create a spreadsheet with two sheets:
- Sheet 1: linked to your Google Form (responses land here automatically)
- Sheet 2 (`Dashboard`): this is where Make writes AI results

**2. Import the blueprint**
In Make.com → **Scenarios** → **Create a new scenario** →
click the three dots → **Import Blueprint** → upload `blueprint.json`

**3. Reconnect credentials**
Inside Make, reconnect all three:
- Google Sheets connection → your Google account
- OpenAI connection → your API key
- Gmail connection → your Google account

**4. Update Spreadsheet IDs**
In the Google Sheets modules, replace the spreadsheet IDs with your own
(found in the Google Sheets URL between `/d/` and `/edit`)

**5. Activate**
Toggle the scenario to **ON** — it fires automatically on every new submission

---

## 🎯 Before vs After

| Without this pipeline | With this pipeline |
|---|---|
| Admin reads every submission manually | Zero manual reading required |
| Urgent cases found hours later | Gmail alert sent within seconds |
| Inconsistent manual categorisation | Every submission categorised identically by GPT-4o |
| Raw text, impossible to filter | Structured, sortable, chartable dashboard |
| No audit trail | Full timestamp log of every processed submission |

---

## ⚠️ Privacy & Security

- All screenshots use **fictional test data only** — no real patient information
- No API keys are stored in this repository
- The Gmail address visible in the original blueprint has been removed
- To run this yourself, supply your own credentials — nothing here is reusable as-is

---

## 👩‍💻 Author

**Hind Faiz**
MSc Informatics for Digital Health — University of Pisa

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://linkedin.com/in/hind-faiz-6b466a288)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/HINDHIO)
MDEOF
echo "done"
