<div align="center">

# Finance Automation Demo

### Automated Bank Statement Processing Pipeline

**Gmail IMAP → PDF Unlock → Gemini 2.5 Flash Parse → Categorize → Google Drive → Email Report**

[![Run Demo](https://img.shields.io/badge/▶_Run_Demo-green?style=for-the-badge&logo=github-actions&logoColor=white)](../../actions/workflows/demo.yml)
[![Docker Image](https://img.shields.io/badge/Docker_Image-GHCR-blue?style=for-the-badge&logo=docker)](../../pkgs/container/finance-automation-demo)

</div>

---

## What This Does

This demo connects to a **real Gmail inbox** via IMAP, fetches bank statement PDFs, and processes them end-to-end:

| Feature | Description |
|---------|-------------|
| **Gmail IMAP Fetch** | Connects to a live inbox and downloads PDF statement attachments |
| **PDF Unlocking** | Automatically decrypts AES-256 / RC4 password-protected PDFs |
| **AI Transaction Parsing** | Google Gemini 2.5 Flash extracts every transaction with 99%+ accuracy |
| **Smart Categorization** | 50+ regex rules + AI fallback categorize spending into 17 categories |
| **Google Drive Upload** | Organized file storage: `/Finance/{Bank}/{Month}/statement.pdf` |
| **HTML Email Report** | Beautiful summary with charts, top expenses, category breakdown |
| **Deduplication** | SHA-256 hashing prevents duplicate transaction imports |

### Two Operating Modes

| Mode | What Happens | You Provide |
|------|-------------|-------------|
| **Demo Mode** | Fetches sample PDFs from a pre-loaded demo Gmail inbox | Just your email address |
| **Live Mode** | Fetches real statements from YOUR Gmail inbox | Your Gmail + App Password |

---

## Quick Start — Run the Demo

### Option 1: Demo Mode (Easiest — 30 Seconds)

1. Click the green **"Run Demo"** button above (or go to **Actions** → **Run Finance Demo** → **Run workflow**)
2. Enter just your **email address** in the `report_email` field
3. Click **"Run workflow"**
4. Wait ~5 minutes → Check your inbox for a full HTML finance report

That's it! The demo auto-connects to a Gmail inbox with sample bank statements from Chase, Bank of America, Wells Fargo, and Citi.

### Option 2: Live Mode (Analyze Your Own Gmail)

Want to test with your own bank statements? Add these optional inputs:

| Input | Required | Description |
|-------|----------|-------------|
| `report_email` | ✅ | Email address to receive the report |
| `gmail_address` | ❌ | Your Gmail address (for IMAP fetch) |
| `gmail_app_password` | ❌ | Gmail App Password ([how to get one](https://myaccount.google.com/apppasswords)) |
| `drive_credentials` | ❌ | Base64-encoded Google Drive service account JSON |
| `drive_folder_id` | ❌ | Target Google Drive folder ID |

> **Note:** All API keys and credentials are pre-configured as encrypted secrets. You only need to provide your email address for demo mode.

---

## What Happens When You Run It

```
┌─────────────────────────────────────────────────────────┐
│  GitHub Actions pulls the Docker image from GHCR        │
│                        ↓                                │
│  Pipeline connects to Gmail via IMAP (SSL, port 993)    │
│                        ↓                                │
│  Downloads PDF bank statement attachments               │
│                        ↓                                │
│  Password-protected PDFs are automatically unlocked     │
│                        ↓                                │
│  Gemini 2.5 Flash parses all transactions into JSON     │
│                        ↓                                │
│  50+ regex rules categorize every transaction           │
│                        ↓                                │
│  (Optional) PDFs uploaded to Google Drive                │
│                        ↓                                │
│  Beautiful HTML report generated                        │
│                        ↓                                │
│  Report emailed to you + saved as GitHub Artifact       │
└─────────────────────────────────────────────────────────┘
```

All processing logs stream in real-time in the Actions UI.

---

## Run Locally with Docker

```bash
# Pull the demo image
docker pull ghcr.io/poornimaramakrishnan/finance-automation-demo:latest

# Run in demo mode (fetches from demo Gmail inbox)
mkdir output
docker run --rm \
  -e GEMINI_API_KEY="your-gemini-key" \
  -e REPORT_EMAIL="you@example.com" \
  -e SMTP_USER="you@gmail.com" \
  -e SMTP_PASSWORD="your-app-password" \
  -e DEMO_GMAIL_ADDRESS="demo-inbox@gmail.com" \
  -e DEMO_GMAIL_APP_PASSWORD="demo-app-password" \
  -v $(pwd)/output:/app/demo/output \
  ghcr.io/poornimaramakrishnan/finance-automation-demo:latest

# Run in live mode (your own Gmail inbox)
docker run --rm \
  -e GEMINI_API_KEY="your-gemini-key" \
  -e REPORT_EMAIL="you@example.com" \
  -e SMTP_USER="you@gmail.com" \
  -e SMTP_PASSWORD="your-app-password" \
  -e GMAIL_ADDRESS="you@gmail.com" \
  -e GMAIL_APP_PASSWORD="your-gmail-app-password" \
  -v $(pwd)/output:/app/demo/output \
  ghcr.io/poornimaramakrishnan/finance-automation-demo:latest

# Open the report
open output/finance_report.html
```

---

## Production Architecture

This demo showcases the core of the full production system:

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Gmail IMAP  │────▶│  Python Pipeline │────▶│  Gemini 2.5 Flash│
│  (fetch PDF  │     │  (standalone,    │     │  (AI parsing)    │
│  attachments)│     │   Dockerized)    │     └────────┬─────────┘
└──────────────┘     └──────────────────┘              │
                                                       ├── Categorize (rules + AI)
                                                       ├── Dedup (SHA-256)
                                                       ├── Notion Sync (API)
                                                       └── Drive Upload (API)
```

**Full stack:** Python pipeline in Docker, scheduled via n8n or CRON  
**Cost:** ~$0.05/month (Gemini API for ~50 statements)  
**Deployment:** Any Docker host (Ampere.sh, Railway, Fly.io, VPS)

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Email Fetch | Gmail IMAP (SSL) | Connect to inbox, download PDF attachments |
| PDF Unlock | pikepdf | AES-256, RC4 decryption |
| AI Parsing | Google Gemini 2.5 Flash | Structured JSON transaction extraction |
| SDK | google-genai >= 1.0.0 | Official Google Gen AI Python SDK |
| Categorization | Regex + Gemini | Smart transaction categorization (17 categories) |
| Deduplication | SHA-256 | Prevents duplicate imports |
| Database | Notion API | Transaction storage & dashboards |
| File Storage | Google Drive API | Organized statement archival |
| Container | Docker + GHCR | Reproducible deployments (python:3.12-slim) |
| CI/CD | GitHub Actions | Automated builds & demo runner |
| Log Streaming | PYTHONUNBUFFERED + flush | Real-time output in Actions UI |

---

## License

MIT License — See [LICENSE](LICENSE) for details.

---

<div align="center">
  <sub>Built with precision by a finance automation specialist</sub>
</div>
