<div align="center">

# Finance Automation Demo

### Automated Bank Statement Processing Pipeline

**Gmail → PDF Unlock → AI Parse → Categorize → Google Drive → Notion → Email Report**

[![Run Demo](https://img.shields.io/badge/▶_Run_Demo-green?style=for-the-badge&logo=github-actions&logoColor=white)](../../actions/workflows/demo.yml)
[![Docker Image](https://img.shields.io/badge/Docker_Image-GHCR-blue?style=for-the-badge&logo=docker)](../../pkgs/container/finance-automation-demo)

</div>

---

## What This Does

This demo processes **11 realistic bank statement PDFs** from 5 major US banks, demonstrating:

| Feature | Description |
|---------|-------------|
| **PDF Unlocking** | Automatically decrypts AES-256 / RC4 password-protected PDFs |
| **AI Transaction Parsing** | Google Gemini 2.0 Flash extracts every transaction with 99%+ accuracy |
| **Smart Categorization** | 50+ regex rules + AI fallback categorize spending into 17 categories |
| **Google Drive Upload** | Organized file storage: `/Finance/{Bank}/{Month}/statement.pdf` |
| **HTML Email Report** | Beautiful summary with charts, top expenses, category breakdown |
| **Deduplication** | SHA-256 hashing prevents duplicate transaction imports |

### Sample Banks Included

| Bank | Statements | Encrypted | Transactions |
|------|-----------|-----------|-------------|
| Chase (Checking) | 3 months | Yes (AES-256) | ~77 |
| Bank of America | 2 months | No | ~53 |
| Wells Fargo | 2 months | Yes (AES-256) | ~54 |
| Citi (Credit Card) | 2 months | Yes (RC4) | ~38 |
| Capital One | 2 months | No | ~47 |
| **Total** | **11 PDFs** | **7 locked** | **~269** |

---

## Quick Start — Run the Demo

### 1. Get a Gemini API Key (Free)

1. Go to [Google AI Studio](https://aistudio.google.com/apikey)
2. Click **"Create API Key"**
3. Copy the key — you'll paste it in Step 3

### 2. (Optional) Set Up Email Delivery

To receive the report via email, you need a **Gmail App Password**:

1. Go to [Google Account Security](https://myaccount.google.com/security)
2. Enable **2-Step Verification** if not already on
3. Go to [App Passwords](https://myaccount.google.com/apppasswords)
4. Generate a new app password for "Mail"
5. Copy the 16-character password

### 3. Run the Demo

1. Click the green **"Run Demo"** button above (or go to **Actions** tab → **Run Finance Demo** → **Run workflow**)
2. Fill in the inputs:

| Input | Required | Description |
|-------|----------|-------------|
| `gemini_api_key` | ✅ | Your Gemini API key from Step 1 |
| `report_email` | ✅ | Email address to receive the report |
| `smtp_user` | ❌ | Gmail address for sending (defaults to report_email) |
| `smtp_password` | ❌ | Gmail App Password from Step 2 |
| `drive_credentials` | ❌ | Base64-encoded Google Drive service account JSON |
| `drive_folder_id` | ❌ | Target Google Drive folder ID |

3. Click **"Run workflow"**
4. Wait ~2 minutes for processing
5. Download the report from **Artifacts** section

---

## What Happens When You Run It

```
┌─────────────────────────────────────────────────────────┐
│  GitHub Actions pulls the Docker image from GHCR        │
│                        ↓                                │
│  11 sample PDFs are loaded from the container           │
│                        ↓                                │
│  7 password-protected PDFs are automatically unlocked   │
│                        ↓                                │
│  Gemini 2.0 Flash parses all ~269 transactions          │
│                        ↓                                │
│  50+ regex rules categorize every transaction           │
│                        ↓                                │
│  (Optional) PDFs uploaded to Google Drive                │
│                        ↓                                │
│  Beautiful HTML report generated                        │
│                        ↓                                │
│  Report emailed + saved as GitHub Artifact              │
└─────────────────────────────────────────────────────────┘
```

---

## Run Locally with Docker

```bash
# Pull the demo image
docker pull ghcr.io/poornimaramakrishnan/finance-automation-demo:latest

# Run with just Gemini (report saved locally)
docker run --rm \
  -e GEMINI_API_KEY="your-gemini-key" \
  -v $(pwd)/output:/app/demo/output \
  ghcr.io/poornimaramakrishnan/finance-automation-demo:latest

# Run with email delivery
docker run --rm \
  -e GEMINI_API_KEY="your-gemini-key" \
  -e REPORT_EMAIL="you@example.com" \
  -e SMTP_USER="you@gmail.com" \
  -e SMTP_PASSWORD="abcd-efgh-ijkl-mnop" \
  -v $(pwd)/output:/app/demo/output \
  ghcr.io/poornimaramakrishnan/finance-automation-demo:latest

# Open the report
open output/finance_report.html
```

---

## Production Architecture

This demo showcases a subset of the full production system:

```
┌──────────────┐     ┌──────────────────┐     ┌──────────────┐
│   Gmail API  │────▶│  n8n Orchestrator │────▶│ Python Parser│
│  (fetch PDF  │     │  (CRON schedule)  │     │  (FastAPI)   │
│  attachments)│     └──────────────────┘     └──────┬───────┘
└──────────────┘                                     │
                                                     ├── PDF Unlock (pikepdf)
                                                     ├── Parse (camelot + Gemini)
                                                     ├── Categorize (rules + AI)
                                                     ├── Dedup (Redis SHA-256)
                                                     ├── Notion Sync (API)
                                                     └── Drive Upload (API)
```

**Full stack:** n8n + FastAPI + Redis, deployed on Docker Compose  
**Cost:** ~$0.05/month (Gemini API for ~50 statements)  
**Deployment:** Any Docker host (Ampere.sh, Railway, Fly.io, VPS)

---

## Tech Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| Orchestration | n8n (self-hosted) | CRON scheduling, Gmail integration |
| Parser | Python 3.12 + FastAPI | PDF processing microservice |
| PDF Unlock | pikepdf | AES-256, RC4 decryption |
| Table Extraction | camelot-py | Structured table parsing |
| AI Parsing | Google Gemini 2.0 Flash | Complex/scanned PDF fallback |
| Categorization | Regex + Gemini | Smart transaction categorization |
| Deduplication | Redis + SHA-256 | Prevents duplicate imports |
| Database | Notion API | Transaction storage & dashboards |
| File Storage | Google Drive API | Organized statement archival |
| Containerization | Docker + GHCR | Reproducible deployments |
| CI/CD | GitHub Actions | Automated builds & demo runner |

---

## License

MIT License — See [LICENSE](LICENSE) for details.

---

<div align="center">
  <sub>Built with precision by a finance automation specialist</sub>
</div>
