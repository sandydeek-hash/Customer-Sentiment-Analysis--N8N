# 🧠 Intelligent Customer Feedback & Sentiment Analyser — n8n Workflow

> AI-powered pipeline that reads customer feedback, performs sentiment analysis, and auto-generates personalized emails to both the customer and the product team — using n8n, Groq (Llama 3.1), and OpenAI GPT-4o Mini.

---

## 🧩 Problem

Customer feedback is high-signal but high-volume. Manually reading, categorizing, and responding to each submission delays resolution and risks inconsistent communication. Product teams also lack a reliable, structured loop to receive feedback summaries with actionable insights in real time.

---

## 💡 Solution

An intelligent n8n workflow that:
1. Reads customer feedback responses directly from a Google Sheets form
2. Performs AI-powered sentiment analysis (Positive / Neutral / Negative) using Groq's Llama 3.1
3. Generates a personalized, empathetic response email to the customer — tailored to their sentiment
4. Simultaneously generates a structured internal email to the product team with feedback details and recommended actions
5. Sends both emails automatically via Gmail — no human intervention required

---

## 🏗️ Workflow Architecture

```
Manual / Schedule Trigger
        ↓
  Google Sheets Node (Read feedback form responses)
        ↓
  Limit Node (Control batch size)
        ↓
  Sentiment Analysis Node ← Groq Llama 3.1 (AI Language Model)
     [Classifies: Positive / Neutral / Negative]
        ↓ (parallel branches)
   ┌────────────────────────────────────┐
   │                                    │
   ▼                                    ▼
OpenAI GPT-4o Mini               OpenAI GPT-4o Mini
(Customer email draft)           (Product team email draft)
   │                                    │
   ▼                                    ▼
Gmail → Customer                 Gmail → Product Team
```

**Two AI models working in tandem:**
- **Groq Llama 3.1 (8B Instant)** — fast, lightweight sentiment classification
- **OpenAI GPT-4o Mini** — higher-quality email generation for both output streams

---

## 🔧 Tech Stack

| Component | Tool |
|-----------|------|
| Workflow Automation | [n8n](https://n8n.io) |
| Feedback Source | Google Sheets (Google Forms response sheet) |
| Sentiment Analysis | n8n LangChain Sentiment Node + Groq Llama 3.1 |
| Email Generation (Customer) | OpenAI GPT-4o Mini |
| Email Generation (Product Team) | OpenAI GPT-4o Mini |
| Email Delivery | Gmail (OAuth2) |
| Trigger | Manual (extensible to form submission trigger or schedule) |

---

## 🤖 AI Design Decisions

### Sentiment Analysis
- Uses n8n's built-in LangChain Sentiment Analysis node with three categories: **Positive, Neutral, Negative**
- Powered by **Groq Llama 3.1 (8B Instant)** — chosen for speed and cost efficiency at the classification stage
- Detailed results enabled for richer downstream context

### Customer Email Prompt
```
You are an empathetic email writer. Based on the sentiment analysis performed,
generate a warm draft email with a subject and body to the customer based on
the sentiment feedback (positive, negative, neutral). Reflect on the email
and write the draft again if required.
Format: Subject / Body
```
**Design decision:** The self-reflection instruction ("write the draft again if required") improves output quality — the model critiques its own draft before finalizing.

### Product Team Email Prompt
```
You are an expert AI assistant. Create a professional and concise email to the
marketing team summarizing the customer feedback and sentiment category.
Include: Customer Name, Email, Feedback Category, Suggestion, Sentiment.
Format: Bullet points
```

---

## 📋 Input Data Schema

Feedback is collected via Google Forms and stored in Google Sheets with these fields:

| Field | Description |
|-------|-------------|
| `Email` | Customer email address |
| `Your Name` | Customer name |
| `Feedback Category` | e.g. Product, Support, Billing |
| `Your feedback` | Free-text feedback |
| `Your Suggestions` | Optional improvement suggestions |

---

## ⚙️ Setup Instructions

### Prerequisites
- n8n account (cloud or self-hosted)
- Google account with Sheets + Gmail OAuth2 credentials configured in n8n
- OpenAI API key
- Groq API key (free at [console.groq.com](https://console.groq.com))

### Steps

1. **Import the workflow**
   - In n8n → **Workflows → Import from File**
   - Upload `workflow/feedback-analyser-sanitized.json`

2. **Connect Google Sheets**
   - In the `get form data` node → set your Google Sheets OAuth2 credential
   - Update the `documentId` to point to your own Google Sheet
   - Sheet must have columns: `Email`, `Your Name`, `Feedback Category`, `Your feedback`, `Your Suggestions`

3. **Connect Groq**
   - In the `Groq Chat Model` node → add your Groq API key
   - Model: `llama-3.1-8b-instant` (free tier available)

4. **Connect OpenAI**
   - In both `Message a model` nodes → add your OpenAI API key

5. **Configure Gmail**
   - In `Send a message` (customer email) → set Gmail OAuth2 credential; `sendTo` is auto-populated from the sheet
   - In `Send a message1` (product team email) → set Gmail credential and update `sendTo` to your product team email address

6. **Test and activate**
   - Run manually to validate both email outputs
   - Switch trigger to **Schedule** or **Google Sheets trigger** for fully automated operation

---

## 📌 Key Design Decisions & Tradeoffs

| Decision | Rationale |
|----------|-----------|
| Groq for sentiment, OpenAI for email | Groq is faster/cheaper for classification; GPT-4o Mini produces higher-quality prose |
| Parallel email generation | Both emails are drafted simultaneously — reduces total execution time |
| Limit node before analysis | Controls API usage, prevents rate limit errors on large feedback batches |
| Self-reflection in customer prompt | Improves empathy and tone quality in the final email draft |
| Bullet-point structure for product team | Structured format is easier to action in a team standup or triage session |

---

## 🚧 Limitations & Future Improvements

- Limit node currently hardcoded — make dynamic based on unprocessed row count
- No deduplication logic — same feedback row could be processed twice on re-run
- Add a Google Sheets "Processed" flag column to track which rows have been handled
- Extend to Slack notification for the product team in addition to email
- Add routing logic: if sentiment is Negative, escalate to a human support agent

---

## 📁 Repo Structure

```
feedback-analyser-n8n/
├── README.md                                ← Full case study (this file)
├── workflow/
│   └── feedback-analyser-sanitized.json    ← Importable n8n workflow (credentials removed)
└── screenshots/
    └── workflow-canvas.png                 ← Add your workflow screenshot here
```

