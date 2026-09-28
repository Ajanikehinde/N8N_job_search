# AI Job Hunter — Approval-Gated Auto-Apply Workflow

An n8n-based workflow that automates a job-search pipeline from company job-board discovery through filtering, scoring, deduplication, human approval, AI-assisted cover-letter generation, application submission, and logging.

> **Portfolio / public-export note:** This repository version is sanitized. Personal email addresses, integration IDs, credential IDs, spreadsheet/database identifiers, and instance-specific metadata have been replaced with placeholders. Configure your own credentials and destinations before importing it into n8n.

## Workflow Overview

```text
Daily 7 AM Trigger
        ↓
Seed Companies
        ↓
Route by ATS
   ┌────┴─────┐
   ↓          ↓
Greenhouse   Ashby
   ↓          ↓
Normalize Job Fields
        ↓
Keyword Filter
        ↓
Rule-Based Job Scoring
        ↓
Sort & Select Top 15
        ↓
Merge with Existing Jobs
        ↓
Remove Duplicates
        ↓
Log / Digest
        ↓
Process Jobs Individually
        ↓
Human Email Approval
        ↓
     ┌──┴──────────────┐
 APPROVED             REJECTED
     ↓                    ↓
LLM Cover Letter     Return to Loop
     ↓
Playwright Submission API
     ↓
Submission Log
```

## What It Does

1. Runs on a daily 7 AM schedule.
2. Seeds a configurable list of companies and their ATS platforms.
3. Retrieves job postings from Greenhouse and Ashby public job-board APIs.
4. Normalizes job data into a common structure.
5. Filters opportunities using configurable data, AI, engineering, security, cloud, and analytics keywords.
6. Applies a rule-based relevance score.
7. Sorts jobs and keeps the top 15 opportunities for the next stage.
8. Checks existing job records and removes duplicate URLs.
9. Logs opportunities to Google Sheets and Notion.
10. Builds and sends a daily job digest.
11. Processes jobs one at a time and requests explicit human approval by email.
12. Only the approved branch proceeds to AI cover-letter generation.
13. Sends the approved application payload to a local Playwright submission service.
14. Logs submission information back to a spreadsheet.

## Job Scoring

The workflow uses a transparent rule-based scoring approach rather than an opaque model. Examples of configured role signals include:

- Data Engineer: +10
- Data Scientist: +10
- Machine Learning: +9
- Analyst: +8
- AI: +7
- Security: +6
- Platform: +5
- Remote: +5
- United Kingdom: +6
- Europe: +4

An additional company-specific bonus is also present in the workflow configuration.

The scoring rules can be edited in the **Score Jobs** node.

## Human-in-the-Loop Safety Gate

The workflow does **not** send every discovered opportunity directly to the application stage.

Each selected job is sent to an approval step. The workflow checks the approval response and follows two paths:

- **Approved:** generate a tailored cover letter and continue to the submission service.
- **Rejected:** skip the application path and return to the job loop.

This keeps final application authorization with the human operator.

## AI Cover Letters

The workflow sends the selected job information to a locally hosted Ollama-compatible endpoint and requests a professional three-paragraph cover letter emphasizing relevant data, AI, cloud, and analytics experience.

The public export uses a local endpoint placeholder:

```text
http://127.0.0.1:11434/v1/chat/completions
```

The configured model in the workflow is:

```text
llama3:8b
```

You can change the model and endpoint in the **LLM Cover Letter** node.

## Application Submission

The workflow sends approved application data to a local HTTP endpoint used by a Playwright-based submission service:

```text
http://127.0.0.1:3001/submit-application
```

The submission service is intentionally kept separate from the n8n workflow. This allows the workflow to handle orchestration while the browser automation service handles website interaction.

## Integrations

The workflow is designed around the following components:

- **n8n** — workflow orchestration
- **Greenhouse Job Board API** — job discovery
- **Ashby Job Board API** — job discovery
- **Google Sheets** — job and submission logging
- **Notion** — job CRM logging
- **Gmail** — daily digest and approval requests
- **Ollama** — local LLM cover-letter generation
- **Playwright service** — browser-based application submission

## Configuration Before Import

Replace the following placeholders with your own configuration:

```text
YOUR_EMAIL@example.com
YOUR_GOOGLE_SHEET_ID
YOUR_SUBMISSION_LOG_SHEET_ID
YOUR_NOTION_DATABASE_ID
YOUR_RESUME_URL
YOUR_CREDENTIAL_ID
YOUR_CREDENTIAL_NAME
```

Configure the required n8n credentials for:

- Gmail
- Google Sheets
- Notion
- Ollama

Also make sure the local Ollama service and Playwright submission service are running before testing the approval-to-submission path.

## Public vs Private Configuration

The public workflow deliberately does not contain personal integration identifiers or credential references from the working environment.

Keep the following private in a real deployment:

- OAuth credentials and tokens
- Personal email addresses
- Google Sheet IDs
- Notion database IDs
- Resume URLs containing private information
- n8n instance metadata
- Any application-service authentication details

## Project Structure

```text
AI-Job-Hunter/
├── README.md
├── workflows/
│   └── AI_Job_Hunter_Public.json
└── .gitignore
```

## Technologies

`n8n` · `JavaScript` · `Greenhouse API` · `Ashby API` · `Google Sheets` · `Notion` · `Gmail` · `Ollama` · `HTTP APIs` · `Playwright`

## Portfolio Focus

This project demonstrates workflow orchestration, API integration, data normalization, rule-based ranking, deduplication, human-in-the-loop controls, local LLM integration, browser-automation orchestration, and operational logging.

It is intended as a portfolio implementation and should be adapted to the terms of service, application policies, and technical requirements of each target job platform before production use.
