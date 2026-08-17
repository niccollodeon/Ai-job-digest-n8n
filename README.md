# AI Job Hunt Assistant (n8n)

A daily automation workflow built in **n8n** that finds fresh "AI Automation Engineer" job postings across LinkedIn, Indeed, JobStreet, and other job boards, filters them down to the most recent listings, avoids repeat notifications, and delivers a clean digest straight to my inbox — no manual searching required.

## What it does

Every day at 8 AM, the workflow:

1. Queries the **JSearch API** (via RapidAPI) for "AI Automation Engineer" roles based in the Philippines.
2. Filters results down to jobs posted within a configurable recency window (currently 7 days).
3. Cross-checks against a **Google Sheet** acting as a lightweight database, so previously seen jobs are never sent twice.
4. Caps the output at a fixed number of listings per day, so the digest stays short and actionable.
5. Logs the newly sent jobs back into the sheet.
6. Builds an HTML summary — job title, company, location, and a direct apply link — and emails it via **Gmail**.

## Why I built this

Manually checking multiple job boards every day for a specific, fairly narrow role is repetitive and easy to fall behind on. This automates the discovery step so I only spend time on jobs actually worth applying to, instead of searching from scratch daily.

## Architecture

```
Schedule Trigger (daily 8AM)
        │
        ▼
HTTP Request → JSearch API (RapidAPI)
        │
        ▼
Code Node → Flatten API response into clean job objects
        │
        ▼
Filter Node → Keep only jobs posted within the last N days
        │
        ├──────────────► Google Sheets → Read "seen" job IDs
        │                         │
        ▼                         ▼
Code Node → Remove already-sent jobs (dedupe)
        │
        ▼
Limit Node → Cap at N jobs/day
        │
        ├──────────────► Google Sheets → Append newly sent job IDs
        │
        ▼
Code Node → Build HTML digest
        │
        ▼
Gmail → Send digest email
```

## Stack

- **n8n** — workflow orchestration
- **JSearch API (RapidAPI)** — aggregated job search across LinkedIn, Indeed, JobStreet, Foundit, JobLeads, and other publishers
- **Google Sheets API** — persistent state for deduplication (acts as a simple database)
- **Gmail API** — email delivery
- **JavaScript** (n8n Code nodes) — response parsing, filtering logic, and HTML generation

## Key engineering decisions

- **Recency filtering** is done in-workflow (via timestamp comparison), not relied on solely from the API's own `date_posted` parameter, since that parameter proved looser than expected.
- **Deduplication** uses a Google Sheet as a minimal persistent store rather than a full database, since the state needed (a list of seen job IDs) is small and simple.
- **Graceful fallbacks** were added for inconsistent fields across API response versions (e.g. missing apply links, differing response nesting between endpoint versions).
- **Daily cap** keeps the digest genuinely useful rather than overwhelming — quality over volume.

## Setup

To run this yourself, you'll need:

1. An **n8n** instance (cloud or self-hosted).
2. A **RapidAPI** account subscribed to JSearch, with your API key.
3. A **Google Sheet** with a tab containing the headers: `job_id | title | company | date_sent`.
4. A **Gmail account** connected to n8n via OAuth2.

Import `AI Job Hunt Assistant.json` into n8n, then update the HTTP Request node's headers with your own RapidAPI key, and point the Google Sheets nodes at your own spreadsheet.

## Status

Actively running daily as part of my own job search for AI Automation Engineer roles in the Philippines.
