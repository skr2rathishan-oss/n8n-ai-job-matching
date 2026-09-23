# Detailed Setup Guide

This guide walks you through setting up and running the **AI Job Matching Automation** workflow on your local machine using n8n and Docker.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step 1: Installing and Starting n8n via Docker](#step-1-installing-and-starting-n8n-via-docker)
3. [Step 2: Accessing the n8n Web Interface](#step-2-accessing-the-n8n-web-interface)
4. [Step 3: Setting Up Google Sheets](#step-3-setting-up-google-sheets)
5. [Step 4: Obtaining NVIDIA Nemotron API Access](#step-4-obtaining-nvidia-nemotron-api-access)
6. [Step 5: Importing the Workflow into n8n](#step-5-importing-the-workflow-into-n8n)
7. [Step 6: Configuring Credentials in n8n](#step-6-configuring-credentials-in-n8n)
8. [Step 7: Binding the Google Spreadsheet](#step-7-binding-the-google-spreadsheet)
9. [Step 8: Testing and Running the Automation](#step-8-testing-and-running-the-automation)
10. [Troubleshooting & FAQs](#troubleshooting--faqs)

---

## Prerequisites

Before starting, ensure you have:

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.
- A **Google Account** with access to Google Sheets and [Google Cloud Console](https://console.cloud.google.com/).
- An **NVIDIA Developer Account** with access to [build.nvidia.com](https://build.nvidia.com/) (free tier available).
- A sample resume in **PDF** format.

---

## Step 1: Installing and Starting n8n via Docker

You can launch n8n using either a direct Docker command or Docker Compose.

### Option A: Direct Docker Run (Recommended for Quickstart)

Open your terminal or PowerShell and run:

```bash
docker pull n8nio/n8n:latest

docker run -d \
  --name n8n \
  -p 5678:5678 \
  -e N8N_DEFAULT_BINARY_DATA_MODE=separate \
  -e WEBHOOK_URL=http://localhost:5678/ \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n:latest
```

### Option B: Docker Compose

Create a file named `docker-compose.yml`:

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n-automation
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_DEFAULT_BINARY_DATA_MODE=separate
      - WEBHOOK_URL=http://localhost:5678/
      - GENERIC_TIMEZONE=UTC
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

Start the container:

```bash
docker compose up -d
```

---

## Step 2: Accessing the n8n Web Interface

1. Open your web browser and navigate to:
   ```
   http://localhost:5678
   ```
2. On first launch, follow the on-screen setup to create your local owner account (email and password).
3. Once completed, you will be taken to the n8n canvas dashboard.

---

## Step 3: Setting Up Google Sheets

The workflow stores and reads data across three dedicated sheets inside one Google Spreadsheet workbook.

### 3.1 Create the Google Spreadsheet

1. Go to [Google Sheets](https://sheets.new) and create a new spreadsheet named **AI Job Matching Database**.
2. Note the **Spreadsheet ID** from your browser address bar:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_GOOGLE_SHEET_ID/edit
   ```
   *(The string between `/d/` and `/edit` is your `YOUR_GOOGLE_SHEET_ID`)*.

### 3.2 Create the Three Required Tabs and Headers

Rename the default sheet to `Jobs`, and add two more tabs named `My_Profile` and `AI_Analysis`. Add the exact column headers to row 1 of each tab:

#### Tab 1: `Jobs`
| Column | Header Name |
|---|---|
| A | `job_id` |
| B | `title` |
| C | `company` |
| D | `location` |
| E | `job_type` |
| F | `experience_level` |
| G | `requirements` |
| H | `description` |
| I | `source` |
| J | `url` |
| K | `posted_date` |
| L | `collected_at` |
| M | `deadline` |
| N | `status` |
| O | `match_score` |

#### Tab 2: `My_Profile`
| Column | Header Name |
|---|---|
| A | `name` |
| B | `email` |
| C | `phone` |
| D | `location` |
| E | `education` |
| F | `degree` |
| G | `university` |
| H | `gpa` |
| I | `skills` |
| J | `programming_languages` |
| K | `frameworks` |
| L | `databases` |
| M | `cloud_technologies` |
| N | `tools` |
| O | `projects` |
| P | `experience` |
| Q | `certifications` |
| R | `preferred_roles` |
| S | `experience_level` |

#### Tab 3: `AI_Analysis`
| Column | Header Name |
|---|---|
| A | `job_id` |
| B | `job_relevance` |
| C | `why_match` |
| D | `matching_skills` |
| E | `missing_skills` |
| F | `recommendation` |
| G | `analyzed_at` |

---

## Step 4: Obtaining NVIDIA Nemotron API Access

This project uses NVIDIA Nemotron models for both structured resume extraction and contextual candidate-to-job matching:

1. Visit [NVIDIA NIM Catalog](https://build.nvidia.com/).
2. Sign in or create a developer account.
3. Select any Nemotron model (e.g., `nvidia/nemotron-3.5-lightning-30b-a3b` or `nvidia/nemotron-3-super-120b-a12b`).
4. Click **Get API Key** and generate an API key starting with `nvapi-...`.
5. Keep this key safe; do not share or commit it.

---

## Step 5: Importing the Workflow into n8n

1. In the n8n UI, click **Workflows** in the left sidebar.
2. Click the **+ Add Workflow** button in the top right corner.
3. In the workflow menu (three dots `...` in top right), select **Import from File**.
4. Choose `workflow/ai-job-matching-workflow.json` from this repository.
5. The full 27-node workflow canvas will load immediately.

---

## Step 6: Configuring Credentials in n8n

### 6.1 Google Sheets OAuth2 Credential

1. Go to **Credentials** in the left sidebar of n8n.
2. Click **Add Credential** and search for **Google Sheets OAuth2 API**.
3. Follow n8n's standard OAuth2 setup:
   - In Google Cloud Console, enable **Google Sheets API** and **Google Drive API**.
   - Create an **OAuth 2.0 Client ID** (Web application).
   - Set the Authorized Redirect URI to n8n's callback URL (shown in the credential window, usually `http://localhost:5678/rest/oauth2-credential/callback`).
   - Copy the Client ID and Client Secret into n8n.
4. Click **Sign in with Google** and grant permission to read/write spreadsheets.
5. Save the credential as **Google Sheets account**.

### 6.2 NVIDIA API Credential (for Chat Model node)

1. In **Credentials**, click **Add Credential**.
2. Search for **NVIDIA API** (or select NVIDIA Nemotron).
3. Paste your API key (`nvapi-...`) into the API Key field.
4. Save the credential as **NVIDIA Nemotron account**.

### 6.3 NVIDIA API Key in Resume Extraction Node

1. On the workflow canvas, double-click the node named **`AI - Analyze Resume`** (HTTP Request node).
2. Under **Headers Parameters**:
   - Find the `Authorization` header.
   - Replace `Bearer YOUR_NVIDIA_API_KEY` with your actual key:
     `Bearer YOUR_NVIDIA_API_KEY`
3. Click outside or close the node drawer to save.

---

## Step 7: Binding the Google Spreadsheet

You need to connect each of the Google Sheets nodes to your newly created Google Sheet:

1. Double-click each of the following 7 nodes one by one:
   - `Get Existing Jobs`
   - `Delete rows or columns from sheet`
   - `Append row in sheet`
   - `Get My Profile`
   - `Update Job Match Scores`
   - `Append row in sheet1`
   - `Append row in sheet2`
2. In the **Credential to connect with** dropdown, select your **Google Sheets account**.
3. Under **Document**, select **By URL** or **From list**, and pick or paste your Google Spreadsheet ID (`YOUR_GOOGLE_SHEET_ID`).
4. Ensure the **Sheet Name** matches the appropriate tab (`Jobs`, `My_Profile`, or `AI_Analysis`).
5. Save the workflow (**Ctrl+S** / **Cmd+S**).

---

## Step 8: Testing and Running the Automation

The workflow is split into two logical pipelines:

### Pipeline 1: Ingesting Your Resume

1. Locate the **`LankaJob Intelligence - Resume Upload`** node (Form Trigger).
2. Click **Test step** or copy the **Production URL / Test URL** of the form.
3. Open the form URL in your browser:
   ```
   http://localhost:5678/form/...
   ```
4. Upload your resume in **PDF** format and click **Submit**.
5. Observe the nodes executing:
   - `Extract from File` converts PDF pages to text.
   - `AI - Analyze Resume` queries Nemotron to extract skills, degrees, GPA, and roles as JSON.
   - `Parse Resume Profile` structures the profile.
   - `Append row in sheet1` writes your profile directly into the `My_Profile` Google Sheet tab.
6. Check your Google Sheet to verify that the `My_Profile` row is populated.

### Pipeline 2: Discovering, Scoring, and Analyzing Jobs

1. Once your profile exists in the `My_Profile` tab, click **Execute workflow** from the manual trigger node (**`When clicking ‘Execute workflow’`**).
2. The automation executes the following sequence:
   - Deletes old job entries in the `Jobs` sheet to keep the database fresh.
   - Fetches live jobs concurrently from LinkedIn and ITPro.lk RSS feeds.
   - Merges both sources and eliminates duplicates based on title, company, and location.
   - Computes algorithmic match scores (Technical Skills 50%, Role Fit 30%, Internship Fit 10%, Education Fit 10%).
   - Populates and updates `Jobs` in Google Sheets.
   - Hands top-ranking matches to the **Basic LLM Chain** powered by NVIDIA Nemotron 120B.
   - Formulates structured relevance assessments and saves them to `AI_Analysis`.

---

## Troubleshooting & FAQs

### Q: Why did the HTTP Request node return a `401 Unauthorized`?
Ensure your `Authorization` header in the `AI - Analyze Resume` node contains `Bearer ` followed immediately by your valid key (`Bearer nvapi-...`). Verify your key is active at [build.nvidia.com](https://build.nvidia.com/).

### Q: How do I change the job search keywords or locations?
- In the `Linkdin` node, update the URL query parameters (e.g. change `lk.linkedin.com/jobs/internship-jobs` to your preferred target role or country).
- In the `ITPro.lk` node, update the RSS URL to target your desired category.

### Q: How do I change the number of jobs sent for deep AI analysis?
Double-click the **`Limit`** node (placed right before the LLM chain) and adjust the maximum items (default is configured to process top matches to manage API rate limits efficiently).
