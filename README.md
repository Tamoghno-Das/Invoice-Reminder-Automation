# 🧾 Invoice Reminder Automation

### AI-Powered Accounts Receivable Follow-Up System built on n8n

An end-to-end, no-code/low-code workflow that monitors outstanding invoices in a spreadsheet, automatically calculates how close (or overdue) each invoice is to its due date, and uses an **AI Agent (OpenAI GPT)** to draft and send **personalized, stage-appropriate reminder emails** — with zero manual follow-up required.

<p align="left">
  <img src="https://img.shields.io/badge/n8n-Workflow%20Automation-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" />
  <img src="https://img.shields.io/badge/OpenAI-GPT%20Agent-412991?style=for-the-badge&logo=openai&logoColor=white" />
  <img src="https://img.shields.io/badge/Google%20Sheets-Data%20Source-34A853?style=for-the-badge&logo=googlesheets&logoColor=white" />
  <img src="https://img.shields.io/badge/Gmail-Auto%20Delivery-EA4335?style=for-the-badge&logo=gmail&logoColor=white" />
  <img src="https://img.shields.io/badge/License-MIT-blue?style=for-the-badge" />
</p>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Demo Video](#-demo-video)
- [Key Features](#-key-features)
- [How It Works](#-how-it-works)
- [Reminder Stage Logic](#-reminder-stage-logic)
- [Tech Stack](#-tech-stack)
- [Data Source Schema](#-data-source-schema)
- [Repository Structure](#-repository-structure)
- [Prerequisites](#-prerequisites)
- [Setup & Installation](#-setup--installation)
- [Configuration](#-configuration)
- [Usage](#-usage)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [License](#-license)
- [Author](#-author--contact)

---

## 📌 Overview

Late payments are one of the most persistent operational drains for any finance or accounts receivable team. **Invoice Reminder Automation** solves this by turning a simple invoice tracker (Google Sheets / Excel) into a self-running collections assistant.

The workflow runs on a schedule, reads every invoice row, filters out anything already paid, calculates how many days remain until (or have passed since) the due date, classifies each invoice into a **reminder stage**, and hands that context to an **AI Agent** that writes a natural, professional, context-aware email before sending it automatically through Gmail.

No manual tracking spreadsheets to babysit, no forgotten follow-ups, no generic copy-pasted reminder templates.

---

## 🎥 Demo Video

A full walkthrough of the workflow running end-to-end is included in this repository at [`assets/demo/invoice-automation-workflow.mp4`](assets/demo/invoice-automation-workflow.mp4).

<video src="assets/demo/invoice-automation-workflow.mp4" controls width="100%">
  Your browser (or GitHub's preview) does not support inline video playback.
  <a href="assets/demo/invoice-automation-workflow.mp4">Click here to download and watch the demo</a>.
</video>

> **Note for viewing on GitHub:** GitHub renders inline video playback for files committed directly to the repo, but if it doesn't autoplay in your browser, use the link above to download it, or open the raw file. For guaranteed inline playback in the README preview, you can alternatively drag-and-drop the `.mp4` into a new GitHub Issue/Discussion comment — GitHub will host it on its CDN and generate a permanent, always-playable `<video>` embed link you can paste in place of the tag above.

**[▶ Download / Watch the Demo Video](assets/demo/invoice-automation-workflow.mp4)**

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🔁 **Scheduled Automation** | Runs automatically on a defined schedule — no manual trigger needed. |
| 📊 **Live Spreadsheet Sync** | Pulls invoice data directly from Google Sheets / Excel in real time. |
| 🧮 **Automatic Due-Date Calculation** | Computes days remaining or days overdue for every invoice. |
| 🚦 **Multi-Stage Reminder Logic** | Classifies invoices into distinct follow-up stages (upcoming, due today, overdue, escalation, etc.). |
| 🤖 **AI-Generated Emails** | An OpenAI-powered AI Agent drafts a personalized, professional reminder for each stage instead of a static template. |
| 📧 **Automatic Delivery** | Sends the finished email directly via Gmail — fully hands-off. |
| 🧩 **No-Code / Visual Workflow** | Built entirely in n8n, so it's easy to inspect, extend, and customize without writing a backend service. |
| 🧠 **Memory & Tool-Ready AI Agent** | The AI Agent node is pre-wired with Memory and Tool ports for future extensions (e.g., CRM lookups, payment links). |

---

## ⚙️ How It Works

The entire pipeline is a single visual n8n workflow. The diagram below is the actual workflow canvas from this repository:

![Invoice Reminder Automation – n8n Workflow Diagram](assets/workflow-diagram.png)

### Node-by-node breakdown

| # | Node | Type | Purpose |
|---|---|---|---|
| 1 | **Schedule Trigger** | Trigger | Kicks off the workflow automatically at a defined interval (e.g., daily). |
| 2 | **Get row(s) in sheet** | Google Sheets (Read) | Fetches every invoice record from the tracking spreadsheet. |
| 3 | **Filter Unpaid** | Filter | Discards invoices already marked `Paid`, keeping only outstanding ones. |
| 4 | **prepare due date** | Edit Fields / Set | Normalizes and prepares the due-date field for calculation. |
| 5 | **Calculate Days Until Due** | Edit Fields / Set (expression) | Computes how many days remain until — or have passed since — the due date. |
| 6 | **Determine Reminder Stage** | Edit Fields / Set (expression) | Converts the day count into a discrete reminder stage. |
| 7 | **Select Reminder Stage** | Switch (Rules mode) | Routes each invoice down one of six possible stages (0–5) into the AI Agent. |
| 8 | **OpenAI Chat Model** | LangChain Chat Model | Supplies the underlying GPT model that powers the AI Agent. |
| 9 | **AI Agent** | LangChain AI Agent | Generates a stage-appropriate, personalized reminder email (Memory & Tool ports available for future use). |
| 10 | **prepare email** | Edit Fields / Set | Assembles the final subject line and email body from the AI Agent's output. |
| 11 | **Send Invoice Reminder** | Gmail (Send) | Delivers the finished reminder email to the customer. |

**Flow summary:**

```
Schedule Trigger
   └─▶ Get row(s) in sheet
          └─▶ Filter Unpaid (Kept)
                 └─▶ prepare due date
                        └─▶ Calculate Days Until Due
                               └─▶ Determine Reminder Stage
                                      └─▶ Select Reminder Stage (0–5)
                                             └─▶ AI Agent ⇄ OpenAI Chat Model
                                                    └─▶ prepare email
                                                           └─▶ Send Invoice Reminder (Gmail)
```

---

## 🚦 Reminder Stage Logic

The **Select Reminder Stage** node classifies each unpaid invoice into one of six stages based on days until/since the due date, and the AI Agent tailors tone and urgency accordingly. Update the table below to match the exact thresholds configured in your `Determine Reminder Stage` node:

| Stage | Typical Trigger | Tone of AI-Generated Email |
|---|---|---|
| 0 | Due in several days (upcoming) | Friendly, informational heads-up |
| 1 | Due soon (1–2 days) | Polite reminder |
| 2 | Due today | Neutral, courteous notice |
| 3 | Overdue (early) | Firm but professional follow-up |
| 4 | Overdue (extended) | Escalated urgency |
| 5 | Severely overdue | Final notice / escalation tone |

> 💡 *These thresholds and tones are fully configurable inside the `Determine Reminder Stage` and `AI Agent` nodes — adjust the numbers above to reflect your actual business rules before sharing this document externally.*

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Automation Engine | [n8n](https://n8n.io/) (self-hosted or n8n Cloud) |
| Data Source | Google Sheets (Excel-compatible template included) |
| AI / NLP | OpenAI GPT via LangChain Chat Model node |
| Email Delivery | Gmail API |
| Trigger | n8n Schedule Trigger (cron-based) |

---

## 📊 Data Source Schema

The workflow reads from a spreadsheet with the following columns. A ready-to-use template is included at [`data/Invoice_Reminder_Template.xlsx`](data/Invoice_Reminder_Template.xlsx).

| Column | Type | Description |
|---|---|---|
| `Invoice ID` | Text | Unique identifier for the invoice (e.g., `INV-001`). |
| `Customer Name` | Text | Name of the client/company being billed. |
| `Customer Email` | Email | Recipient address for the reminder email. |
| `Invoice Amount` | Currency | Total amount due on the invoice. |
| `Invoice Date` | Date | Date the invoice was issued. |
| `Due Date` | Date | Payment due date — used for all stage calculations. |
| `Payment Status` | Text | `Paid` / `Pending` — determines whether the invoice enters the reminder pipeline. |

**Sample rows:**

| Invoice ID | Customer Name | Invoice Amount | Due Date | Payment Status |
|---|---|---|---|---|
| INV-001 | ABC LTD | 50,000/- | 2026-08-19 | Pending |
| INV-002 | XYZ PVT LTD | 85,000/- | 2026-08-15 | Paid |
| INV-003 | JASOP CORP | 90,000/- | 2026-08-12 | Pending |
| INV-004 | TECH SOLUTIONS | 35,000/- | 2026-08-09 | Pending |

**[⬇ Download the Excel Template](data/Invoice_Reminder_Template.xlsx)**

---

## 📁 Repository Structure

```
Invoice_Reminder_Automation/
├── README.md
├── workflow/
│   └── invoice-reminder-automation.json     # Exported n8n workflow — import directly into n8n
├── assets/
│   ├── workflow-diagram.png                 # Workflow canvas screenshot (used above)
│   └── demo/
│       └── invoice-automation-workflow.mp4  # Full demo recording
└── data/
    └── Invoice_Reminder_Template.xlsx       # Sample / starter invoice tracker
```

> ℹ️ Adjust the tree above (especially the `workflow/` JSON filename) to match the actual file names in your repository.

---

## ✅ Prerequisites

Before setting this up, make sure you have:

- An [n8n](https://n8n.io/) instance (self-hosted via Docker/npm, or n8n Cloud)
- A Google account with access to Google Sheets + Google Sheets API credentials configured in n8n
- An [OpenAI API key](https://platform.openai.com/) for the Chat Model node
- A Gmail account with OAuth2 credentials configured in n8n for sending mail
- Basic familiarity with importing/editing workflows in n8n

---

## 🚀 Setup & Installation

1. **Clone this repository**
   ```bash
   git clone https://github.com/Tamoghno-Das/Invoice_Reminder_Automation.git
   cd Invoice_Reminder_Automation
   ```

2. **Import the workflow into n8n**
   - Open your n8n instance
   - Click **Import from File** (or **Import from URL**)
   - Select the exported workflow JSON from the `workflow/` folder

3. **Connect your Google Sheet**
   - Copy `data/Invoice_Reminder_Template.xlsx` into a new Google Sheet (or upload directly)
   - Update the **Get row(s) in sheet** node with your Sheet ID and tab name

4. **Add your credentials**
   - Google Sheets OAuth2 credential
   - OpenAI API credential (for the **OpenAI Chat Model** node)
   - Gmail OAuth2 credential (for the **Send Invoice Reminder** node)

5. **Set your schedule**
   - Open the **Schedule Trigger** node and set your preferred run frequency (e.g., every day at 9 AM)

6. **Activate the workflow**
   - Toggle the workflow to **Active**, or click **Execute workflow** to run a manual test

---

## 🔧 Configuration

| Credential | Used By Node | Where to Generate |
|---|---|---|
| Google Sheets OAuth2 | `Get row(s) in sheet` | [Google Cloud Console](https://console.cloud.google.com/) |
| OpenAI API Key | `OpenAI Chat Model` | [OpenAI Platform](https://platform.openai.com/api-keys) |
| Gmail OAuth2 | `Send Invoice Reminder` | [Google Cloud Console](https://console.cloud.google.com/) |

---

## ▶️ Usage

Once activated, the workflow requires no further intervention:

1. The **Schedule Trigger** fires automatically at the configured interval.
2. Invoice data is pulled fresh from the spreadsheet.
3. Paid invoices are automatically excluded.
4. Each remaining invoice is scored against its due date and assigned a reminder stage.
5. The AI Agent drafts a suitable email for that stage.
6. The email is sent automatically via Gmail — no human step required.

To test manually at any time, open the workflow in n8n and click **Execute workflow**.

---

## 🗺 Roadmap

- [ ] Add WhatsApp/SMS reminder channel alongside email
- [ ] Log every sent reminder back to the sheet (timestamp + stage) for audit trail
- [ ] Add a payment-link generator tool to the AI Agent
- [ ] Slack/Teams internal notification for severely overdue invoices
- [ ] Multi-currency support

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m "Add your feature"`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author & Contact

**Tamoghno Das**

- GitHub: [@Tamoghno-Das](https://github.com/Tamoghno-Das)
- Email: *your.email@example.com*
- LinkedIn: *linkedin.com/in/your-profile*

---

<p align="center"><i>Built with n8n to eliminate manual invoice follow-up — one automated reminder at a time.</i></p>
