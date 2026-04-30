# 📬 Gmail AI Email Agent — n8n Workflow

An intelligent email automation workflow built with **n8n**, **Google Gemini**, and the **Gmail API**. It monitors your inbox in real time, classifies incoming emails by priority, applies Gmail labels, and either auto-replies or drafts a response — all without manual intervention.

---

## 📸 Workflow Overview

```
Gmail Trigger (every minute, unread only)
    └── Extract Email Information
            └── Classify Email Priority (keyword engine)
                    └── Route by Priority (Switch)
                            ├── [0] High Priority  → Label (IMPORTANT + STARRED) → Gemini AI → Parse → Send Auto-Reply
                            ├── [1] Medium Priority → Label (IMPORTANT)           → Gemini AI → Parse → Create Draft
                            └── [2] Low Priority   → Label (custom) or no-op
```

---

## ✨ Features

- **Real-time inbox monitoring** — polls Gmail every minute, only processes unread emails
- **3-tier priority classification** — keyword-based engine with 190+ keywords across High, Medium, and Low categories
- **Automatic Gmail labeling** — applies IMPORTANT, STARRED, or custom labels per priority
- **AI-powered responses via Gemini** — generates contextually relevant, professional email replies
- **Auto-send for high priority** — job offers, meeting requests, urgent work emails get an immediate reply
- **Draft creation for medium priority** — learning, career development, and tool emails get a draft for your review
- **Robust JSON parsing** — strips markdown fences from AI output, handles malformed responses gracefully
- **Sender-aware prompting** — AI uses the actual sender name for personalised salutations

---

## 🧠 Priority Classification Logic

The workflow classifies each email by scanning subject, body, and sender address against keyword lists.

### 🔴 High Priority — Job / Work

Triggers **auto-reply** + IMPORTANT + STARRED labels.

Keywords include: `interview`, `offer letter`, `shortlisted`, `deadline`, `urgent`, `asap`, `action required`, `contract`, `salary`, `invoice`, `legal notice`, `performance review`, `production issue`, `onboarding`, `resignation`, and 60+ more.

### 🟡 Medium Priority — Learning / Career

Triggers **draft creation** + IMPORTANT label.

Keywords include: `udemy`, `coursera`, `certification`, `webinar`, `hackathon`, `open source`, `github`, `machine learning`, `aws`, `docker`, `kubernetes`, `react`, `newsletter`, `product launch`, and 60+ more.

### ⚪ Low Priority — Promotions / Other

No reply, no draft. Optionally labeled with a custom Gmail label.

Keywords include: `sale`, `discount`, `coupon`, `unsubscribe`, `flash sale`, `order shipped`, `no-reply`, `donotreply`, `follow us`, and 40+ more.

---

## 🔧 Tech Stack

| Tool                    | Purpose                                         |
| ----------------------- | ----------------------------------------------- |
| [n8n](https://n8n.io)   | Workflow automation engine                      |
| Gmail API (OAuth2)      | Inbox monitoring, labeling, sending, drafts     |
| Google Gemini 2.0 Flash | AI email response generation                    |
| JavaScript (Code nodes) | Email parsing, classification, response parsing |

---

## 📁 Workflow Nodes

| Node                             | Type      | Role                                            |
| -------------------------------- | --------- | ----------------------------------------------- |
| `Gmail Trigger`                  | Trigger   | Polls inbox every minute for unread emails      |
| `Extract Email Information`      | Code (JS) | Parses raw Gmail data into clean fields         |
| `Classify Email Priority`        | Code (JS) | Keyword engine — assigns category, score, label |
| `Route by Priority`              | Switch    | Routes flow to 3 branches based on category     |
| `Label High Priority`            | Gmail     | Applies IMPORTANT + STARRED                     |
| `Label Medium Priority`          | Gmail     | Applies IMPORTANT                               |
| `Message a model`                | Gemini    | Drafts auto-reply for high priority emails      |
| `Message a model1`               | Gemini    | Drafts response for medium priority emails      |
| `Parse High Priority Response`   | Code (JS) | Extracts and sanitizes AI JSON output           |
| `Parse Medium Priority Response` | Code (JS) | Extracts and sanitizes AI JSON output           |
| `Send Auto-Reply`                | Gmail     | Sends the reply immediately                     |
| `Create Draft`                   | Gmail     | Saves draft for manual review                   |

---

## ⚙️ Setup & Installation

### Prerequisites

- n8n instance (self-hosted or cloud)
- Google account with Gmail API access
- Google Gemini API key

### Step 1 — Import the Workflow

1. Copy the contents of `workflow.json`
2. In n8n, go to **Workflows → Import from JSON**
3. Paste and import

### Step 2 — Configure Gmail OAuth2

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a project → Enable **Gmail API**
3. Create OAuth2 credentials (Desktop or Web App)
4. In n8n, create a new **Gmail OAuth2** credential and connect your account
5. Replace the credential reference in all Gmail nodes with your credential

### Step 3 — Configure Gemini API

1. Get a Gemini API key from [Google AI Studio](https://aistudio.google.com)
2. In n8n, create a **Google Gemini (PaLM) API** credential
3. Replace the credential reference in both Gemini nodes with your credential

### Step 4 — (Optional) Create a Low Priority Gmail Label

1. In Gmail → Settings → Labels → **Create new label**
2. Name it `Low Priority` (or anything you prefer)
3. In the `Label Low Priority` node, select that label from the dropdown

### Step 5 — Activate

Toggle the workflow to **Active**. It will start polling immediately.

---

## 🐛 Known Limitations & Notes

- **Keyword-only classification** — no semantic understanding. A promotional email containing the word `project` will be misclassified as high priority. Future improvement: add an AI pre-classification step.
- **Gemini output structure** — the parse nodes read `output[0].content[0].text`. If your n8n version or Gemini node version outputs a different structure, you may need to adjust the path in the parse nodes.
- **Auto-reply risk** — high priority emails trigger an immediate send. Test with a private inbox first. Consider changing `Send Auto-Reply` to `Create Draft` until you're confident in the classification accuracy.
- **Rate limits** — polling every minute on a busy inbox may hit Gmail API rate limits. Increase the poll interval to every 5 minutes if you notice failures.

---

## 🔮 Possible Improvements

- [ ] Add AI-based pre-classification using Gemini to replace/augment keyword matching
- [ ] Add a Slack or Telegram notification node for high priority emails
- [ ] Store processed email IDs in a database to prevent duplicate processing
- [ ] Add sender whitelist/blacklist support
- [ ] Multi-language keyword support (Urdu, Arabic, etc.)
- [ ] Thread-aware replies (reply in the same thread instead of a new email)

---

## 👤 Author

**Zain**
Full-Stack Engineer · AI & Automation Builder · Based in Pakistan

I build production-grade web applications and intelligent automation systems. My stack spans React, Next.js, Node.js, TypeScript, AWS, and Docker on the engineering side — and OpenAI, Gemini, LangChain, Pinecone, and n8n on the AI/automation side.

Most of my automation work focuses on reducing manual overhead for small teams and founders — things like this email agent, LinkedIn content pipelines, async job processing, and AI-powered workflows.

- 🔗 [GitHub](https://github.com/zainulabideen041)
- 💼 [LinkedIn](https://linkedin.com/in/zainulabideen041)
- 🌐 [Portfolio](https://zainulabideen-portfolio.netlify.app)
- 📱[Instagram](https://instagram.com/zainulabideen.rizvi)
- [X](https://twitter.com/m_zainrizvi)

---

## 📄 License

MIT License — free to use, modify, and distribute. Attribution appreciated but not required.

---

> Built with n8n · Powered by Google Gemini · Automated with ☕ and too many terminal tabs
