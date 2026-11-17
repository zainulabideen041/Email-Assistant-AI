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
                            └── [2] Low Priority   → Stop
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

The low-priority branch ends without a label, a draft, or a reply.

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
| `Gmail Trigger1`                  | Trigger   | Polls inbox every minute for unread emails      |
| `Extract Email Information1`      | Code (JS) | Parses raw Gmail data into clean fields         |
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

### Step 1 — Import the workflow

1. Download [Email Assistant Agent.json](./Email%20Assistant%20Agent.json) from this repository.
2. Create a workflow in n8n. Open the workflow menu, select **Import from File**, and choose the downloaded JSON.
3. Save the workflow and leave it unpublished while you configure the credentials.

### Step 2 — Connect Gmail and Gemini

- Select your Gmail OAuth2 credential in `Gmail Trigger1`, `Label High Priority`, `Label Medium Priority`, `Send Auto-Reply`, and `Create Draft`. Use a test mailbox for your first run. [n8n Gmail credentials](https://docs.n8n.io/integrations/builtin/credentials/google/)
- Select your Gemini credential and an available model in `Message a model` and `Message a model1`. Both branches send email content to Gemini, so use synthetic messages for testing.

### Step 3 — Test with three messages

1. Disconnect `Send Auto-Reply` from `Parse High Priority Response` on the canvas to prevent outbound mail during the test.
2. In the trigger, narrow the **Sender** filter to a test sender you control. Send these messages to the test inbox and leave them unread.
3. Run a manual execution. Confirm each result in the execution data.

| Example subject | Expected result |
| --- | --- |
| `Interview tomorrow` | Applies IMPORTANT and STARRED. The high-priority parser produces a reply for inspection. |
| `Free webinar on TypeScript` | Applies IMPORTANT and creates a draft. |
| `Weekend sale` | Routes to low priority and stops without sending mail or creating a draft. |

Keep the send node disconnected for draft-only testing. Reconnect it only after you choose to enable automatic replies and verify the intended recipients. Publish the workflow when you are ready for recurring Gmail polling.

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
