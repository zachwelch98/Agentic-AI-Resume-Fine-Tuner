# Setup Guide

This guide walks you through setting up the Resume Fine Tuner workflow step-by-step.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Google Workspace Setup](#google-workspace-setup)
- [n8n Installation](#n8n-installation)
- [Workflow Configuration](#workflow-configuration)
- [Testing & Validation](#testing--validation)

## Prerequisites

### Required Accounts
- [ ] Google account with Drive access
- [ ] OpenAI account with API access
- [ ] n8n instance (cloud or self-hosted)

### Estimated Setup Time
- First-time setup: 30-45 minutes
- Subsequent configurations: 10-15 minutes

## Google Workspace Setup

### 1. Create Your Resume Document

1. Go to [Google Docs](https://docs.google.com/)
2. Create a new document for your master resume
3. Format your resume with clear sections (Experience, Education, Skills, etc.)
4. Copy the document ID from the URL:
   ```
   https://docs.google.com/document/d/DOCUMENT_ID_HERE/edit
   ```
5. Save this ID for later

**Tips:**
- Keep one master resume with all your experience
- The workflow reads this each time, so updates are automatic
- Use consistent formatting for best AI analysis

### 2. Create Your Job Tracker Spreadsheet

1. Go to [Google Sheets](https://sheets.google.com/)
2. Create a new spreadsheet
3. Name the first sheet exactly: **"Job Applications"**
4. Set up these column headers in Row 1:

   | A | B | C | D |
   |---|---|---|---|
   | Job URL | Job Title | Company | Job Description |

5. Add a few test job listings:
   ```
   Row 2: https://example.com/job1 | Senior Developer | TechCorp | [Optional description]
   Row 3: https://example.com/job2 | Product Manager | StartupCo | [Optional description]
   ```

6. Copy the spreadsheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit
   ```
7. Save this ID for later

**Important Notes:**
- Column D (Job Description) is optional
- The workflow will try to scrape from the URL first
- If scraping fails, it falls back to column D
- Don't add the output columns yet—the workflow creates them automatically

## n8n Installation

### Option A: n8n Cloud (Recommended for Beginners)

1. Go to [n8n.cloud](https://n8n.cloud/)
2. Sign up for an account
3. Choose a plan (free tier available)
4. Your instance will be ready in a few minutes

### Option B: Self-Hosted with Docker

```bash
# Install n8n with Docker
docker volume create n8n_data
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  n8nio/n8n
```

Access n8n at `http://localhost:5678`

### Option C: npm Installation

```bash
# Install globally
npm install n8n -g

# Start n8n
n8n start
```

For detailed installation options, see the [official n8n documentation](https://docs.n8n.io/hosting/).

## Workflow Configuration

### Step 1: Import the Workflow

1. Download `Resume_Fine_Tuner.json` from this repository
2. In n8n, click **Workflows** in the sidebar
3. Click **Add Workflow** → **Import from File**
4. Select `Resume_Fine_Tuner.json`
5. The workflow will open in the editor

### Step 2: Configure Google Docs Credential

1. Click on the **"Read Resume"** node
2. Under **Credential to connect with**, click **Create New Credential**
3. Select **Google Docs OAuth2 API**
4. You'll need to set up a Google Cloud Project:

   **First-time Google OAuth Setup:**
   1. Go to [Google Cloud Console](https://console.cloud.google.com/)
   2. Create a new project (e.g., "Resume Fine Tuner")
   3. Enable these APIs:
      - Google Docs API
      - Google Sheets API
   4. Create OAuth 2.0 credentials:
      - Go to **APIs & Services** → **Credentials**
      - Click **Create Credentials** → **OAuth client ID**
      - Application type: **Web application**
      - Add authorized redirect URI: Your n8n OAuth callback URL
        - n8n Cloud: `https://YOUR_INSTANCE.n8n.cloud/rest/oauth2-credential/callback`
        - Self-hosted: `http://localhost:5678/rest/oauth2-credential/callback`
   5. Copy the Client ID and Client Secret

   **In n8n:**
   1. Enter your Client ID and Client Secret
   2. Click **Connect my account**
   3. Complete the Google OAuth flow
   4. Click **Save**

### Step 3: Configure Google Sheets Credential

1. Click on the **"Read All Job Rows"** node
2. Under **Credential to connect with**, click **Create New Credential**
3. Select **Google Sheets OAuth2 API**
4. If you created Google OAuth credentials above, you can reuse them:
   - Enter the same Client ID and Client Secret
   - Click **Connect my account**
   - Authorize access
5. Click **Save**

### Step 4: Configure OpenAI Credential

1. Get your OpenAI API key:
   - Go to [OpenAI Platform](https://platform.openai.com/api-keys)
   - Click **Create new secret key**
   - Copy the key immediately (you won't see it again!)

2. In n8n, click on the **"OpenAI Chat Model"** node
3. Click **Create New Credential**
4. Select **OpenAI**
5. Paste your API key
6. Click **Save**

**Note:** The workflow uses GPT-4o-mini by default, which is cost-effective. You can change the model in the "OpenAI Chat Model" node if desired.

### Step 5: Update Document IDs

You need to replace the placeholder IDs with your actual Google document IDs.

#### Update Resume Document ID

1. Click on the **"Read Resume"** node
2. Find the **Document** field
3. Replace the existing ID with your resume document ID from earlier:
   ```
   YOUR_GOOGLE_DOC_ID_HERE
   ```

#### Update Spreadsheet ID (2 locations)

**Location 1: "Read All Job Rows" node**
1. Click on the node
2. Under **Document**, select **By ID**
3. Replace with your spreadsheet ID:
   ```
   YOUR_GOOGLE_SHEET_ID_HERE
   ```
4. Verify **Sheet** is set to "Job Applications"

**Location 2: "Update row in sheet" node**
1. Click on the node
2. Under **Document**, select **By ID**
3. Replace with the same spreadsheet ID
4. Verify **Sheet** is set to "Job Applications"

### Step 6: Save and Activate

1. Click **Save** in the top-right corner
2. Toggle the workflow to **Active**
3. You're ready to test!

## Testing & Validation

### Run a Test Execution

1. Make sure you have at least one job listing in your spreadsheet
2. Click **Execute Workflow** in the n8n editor
3. Watch the workflow run through each node
4. Check for any errors (they'll be highlighted in red)

### Verify the Output

1. Open your Google Sheets tracker
2. You should see new columns added to the right of column D:
   - Alignment Score
   - Score Rationale
   - Keywords
   - Strengths
   - Improve Strengths
   - Weaknesses
   - Improve Weaknesses
   - Tailoring Suggestions

3. Review the analysis for your first job listing
4. The data should be formatted with clean line breaks and bullets

### Common Setup Issues

#### "Could not parse AI response"
- **Cause:** OpenAI API error or rate limit
- **Fix:** Check your API key and billing status at OpenAI

#### "HTTP Request failed"
- **Cause:** Job site blocks scraping or URL is invalid
- **Fix:** Add the job description manually to column D

#### "Document not found"
- **Cause:** Incorrect document ID or missing permissions
- **Fix:** Double-check the document ID and ensure your Google account has access

#### "Invalid credentials"
- **Cause:** OAuth token expired or misconfigured
- **Fix:** Delete and recreate the credential, re-authorizing with Google

## Advanced Configuration

### Schedule Automatic Runs

1. Click on the **"When clicking 'Execute workflow'"** trigger node
2. Replace it with a **Schedule Trigger** node:
   - Set **Trigger Times** → **Intervals**
   - Example: Run every Monday at 9 AM

### Add Email Notifications

1. Add a **Send Email** node after "Update row in sheet"
2. Configure with your email provider (Gmail, SendGrid, etc.)
3. Include a summary of jobs analyzed

### Customize AI Prompt

1. Click on **"Analyze Resume vs. Job"** node
2. Under **Options** → **System Message**, modify the prompt
3. Common customizations:
   - Change tone (less harsh, more encouraging)
   - Add industry-specific criteria
   - Request different output format
   - Emphasize certain skills or keywords

## Next Steps

Once setup is complete:
1. Add your real job listings to the tracker
2. Run the workflow and review the feedback
3. Tailor your resume based on suggestions
4. Re-run for new jobs as you apply

## Support

If you encounter issues:
- Check the [Troubleshooting section](README.md#-troubleshooting) in the main README
- Review n8n execution logs for detailed error messages
- Open an issue on GitHub with your error details

Happy job hunting! 🚀
