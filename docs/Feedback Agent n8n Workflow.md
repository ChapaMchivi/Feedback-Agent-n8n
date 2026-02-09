# **Feedback Agent (n8n Workflow)**

An automated **Feedback Agent** built in **n8n** that processes customer reviews submitted through a QR‑code form. The workflow reads each new entry from Google Sheets, analyzes it with Google Gemini, and routes it to the correct team automatically — turning raw feedback into structured, actionable insights.

## Badges

![Built with n8n](https://img.shields.io/badge/Built%20with-n8n-orange)
![Google Sheets](https://img.shields.io/badge/Data%20Source-Google%20Sheets-34A853)
![Google Gemini](https://img.shields.io/badge/AI-Google%20Gemini-4285F4)
![LangChain](https://img.shields.io/badge/Structured%20Output-LangChain-6C2DC7)
![Gmail](https://img.shields.io/badge/Email-Gmail-D14836)
![Automation](https://img.shields.io/badge/Automation-Always%20On-brightgreen)
![Status](https://img.shields.io/badge/Project%20Status-Active-success)

---

## 🎥 **Demo Video**

**Watch the full walkthrough:**  
🎥 Demo Video  
[Watch the full walkthrough](https://new.express.adobe.com/id/urn:aaid:sc:US:869415ef-76a3-4f4c-b248-3bc9b90ae966)


---

## **What This Project Does**

- Automates the entire feedback‑handling pipeline  
- Uses AI to classify customer messages  
- Updates the source sheet with clean, normalized labels  
- Routes each item to the correct team via Gmail  
- Eliminates manual sorting, scanning, and triage  

This agent acts like a small digital employee — always on, always consistent.

---

## **Documentation**

For a full breakdown of the workflow, including node‑by‑node explanations, routing logic, and diagrams, see:

👉 **`workflow-overview.md`**

This README intentionally stays high‑level to avoid repeating the detailed documentation.

---

## **Tech Stack**

- **n8n** for workflow automation  
- **Google Sheets** as the data source  
- **Google Gemini** for AI classification  
- **LangChain** for structured output parsing  
- **Gmail** for notifications  

---

## **Authentication in n8n**

The Feedback Agent connects to several external apps — Google Sheets, Google Gemini, and Gmail — and each one needs a secure way to confirm your identity.

In n8n, apps authenticate through three methods (see image in repo):

- **OAuth2** — log in and approve access; the service issues a temporary token  
- **API Keys** — provide a secret key; anyone with the key can access the service  
- **Basic Authentication** — username + password; typically used for older internal tools  

For this project:

- **Google Sheets** uses OAuth2  
- **Google Gemini** uses an API Key  
- **Gmail** uses OAuth2  

Once these connections are set up, the Feedback Agent can read new entries, analyze them with AI, and route each message automatically.

---

## **How the Feedback Agent Fits Together (High‑Level)**

- All customer feedback comes in through a **QR‑code form**  
- Every submission lands in **one Google Sheet**  
- The agent detects new rows, analyzes the text with **Google Gemini**, and routes the message to the right team  

This is the only workflow summary included in the README — the deeper logic, nodes, and routing rules remain in `workflow-overview.md`.

---

## **Repository Contents**

- `workflow.json` — exported n8n workflow  
- `workflow-overview.md` — full technical documentation  
- `README.md` — project summary and demo link  

---

## **Why It Matters**

Small cafés and local businesses often receive valuable feedback but lack the time to process it.  
This agent closes that gap by delivering:

- Faster response times  
- Cleaner data  
- Better visibility into customer needs  
- Zero manual overhead  

A practical example of how automation and AI can elevate everyday operations.

---

