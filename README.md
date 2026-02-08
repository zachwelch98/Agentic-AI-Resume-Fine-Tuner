# Resume Fine Tuner

An automated n8n workflow that uses AI to analyze your resume against job postings, providing brutally honest feedback and actionable suggestions to improve your chances of landing interviews.

## 🎯 What It Does

This workflow automatically:
- Reads your resume from Google Docs
- Fetches job listings from a Google Sheets tracker
- Scrapes job descriptions from URLs
- Uses GPT-4o-mini to analyze resume-job alignment
- Provides detailed, actionable feedback including:
  - **Alignment Score** (1-10) with honest rationale
  - **ATS Keywords** you're missing
  - **Strengths** with specific examples from your resume
  - **Weaknesses** and exactly how to fix them
  - **Tailored Suggestions** with specific rewrites
- Updates your Google Sheets tracker with all analysis results

## 🏗️ Architecture

The workflow follows this process:

1. **Manual Trigger** → Starts the workflow
2. **Read Resume** → Fetches your master resume from Google Docs
3. **Read Job Listings** → Pulls all job applications from Google Sheets (columns A-D)
4. **Loop Over Items** → Processes each job one at a time
5. **HTTP Request** → Scrapes the job description from the URL
6. **Conditional Logic** → Uses scraped description if available, falls back to manual description
7. **AI Analysis** → GPT-4o-mini analyzes resume vs job posting
8. **Parse Response** → Cleans and structures the AI feedback
9. **Update Sheet** → Writes results back to your tracker

## 📋 Prerequisites

### Required Services
- **n8n** (self-hosted or cloud)
- **Google Account** with access to:
  - Google Docs
  - Google Sheets
- **OpenAI API** key (uses GPT-4o-mini)

### Google Sheets Structure

Your job tracker spreadsheet should have these columns (A-D):
- **Column A**: Job URL
- **Column B**: Job Title
- **Column C**: Company
- **Column D**: Job Description (optional, used as fallback)

The workflow will add these columns for results:
- Alignment Score
- Score Rationale
- Keywords
- Strengths
- Improve Strengths
- Weaknesses
- Improve Weaknesses
- Tailoring Suggestions

## 🚀 Setup Instructions

### 1. Import the Workflow

1. Download `Resume_Fine_Tuner.json`
2. In n8n, go to **Workflows** → **Import from File**
3. Select the downloaded JSON file

### 2. Configure Credentials

You'll need to set up three credential sets:

#### Google Docs OAuth2
1. In n8n, go to **Credentials** → **Add Credential**
2. Select "Google Docs OAuth2 API"
3. Follow the OAuth flow to authorize

#### Google Sheets OAuth2
1. Add credential for "Google Sheets OAuth2 API"
2. Authorize with the same Google account

#### OpenAI API
1. Get your API key from [OpenAI Platform](https://platform.openai.com/api-keys)
2. Add "OpenAI" credential in n8n
3. Paste your API key

### 3. Update Document IDs

Replace the hardcoded Google document IDs with your own:

**Resume Document** (in "Read Resume" node):
```json
"documentURL": "YOUR_GOOGLE_DOC_ID_HERE"
```

**Job Tracker Sheet** (in "Read All Job Rows" and "Update row in sheet" nodes):
```json
"documentId": "YOUR_GOOGLE_SHEET_ID_HERE"
```

To find these IDs:
- Google Doc: Open doc → URL shows `docs.google.com/document/d/DOCUMENT_ID/edit`
- Google Sheet: Open sheet → URL shows `docs.google.com/spreadsheets/d/SHEET_ID/edit`

### 4. Activate the Workflow

1. Click **Save** in n8n
2. Toggle **Active** to enable it
3. Click **Execute Workflow** to run manually

## 🔧 Customization

### Modify the AI Prompt

The AI analysis is controlled by the system prompt in the "Analyze Resume vs. Job" node. You can adjust it to:
- Change the scoring criteria
- Request different analysis categories
- Modify the tone (currently set to "brutally honest")
- Add industry-specific requirements

### Adjust Processing

- **Batch Size**: Modify the "Loop Over Items" node to process multiple jobs at once
- **Timeout**: Change HTTP request timeout (default: 15 seconds)
- **Error Handling**: Currently continues on HTTP errors; adjust `onError` setting

### Add More Data Sources

You can extend the workflow to:
- Pull job descriptions from LinkedIn API
- Parse PDF job postings
- Integrate with applicant tracking systems
- Add email notifications when analysis completes

## 📊 Understanding the Output

### Alignment Score (1-10)
- **9-10**: Near-perfect match, ready to apply
- **7-8**: Strong match, minor tweaks needed
- **5-6**: Decent foundation, needs work
- **3-4**: Weak match, major gaps
- **1-2**: Poor fit for this role

### Keywords
Critical terms and skills that ATS systems will scan for. Missing these means your resume might be filtered out before a human sees it.

### Strengths & Improvements
Specific elements of your resume that match the job, plus exact rewrites to make them even stronger.

### Weaknesses & Fixes
Gaps, missing skills, and mismatches with concrete solutions for each.

### Tailoring Suggestions
Top 5 highest-impact changes ranked by priority, with specific copy you can use.

## 🛠️ Troubleshooting

### HTTP Request Fails
- Some job sites block web scraping
- Increase timeout in HTTP Request node
- Manually paste job description in Google Sheets column D as fallback

### AI Response Parse Errors
- Check OpenAI API quota and rate limits
- Verify API key is valid
- Check n8n logs for detailed error messages

### Google Sheets Not Updating
- Ensure the sheet name exactly matches "Job Applications"
- Verify OAuth credentials haven't expired
- Check that columns A-D contain data

### Rate Limiting
- OpenAI has rate limits on API calls
- Add delay between requests in Loop Over Items node
- Consider upgrading OpenAI API tier for higher limits

## 📝 Example Usage

1. Update your master resume in Google Docs
2. Add new job listings to your Google Sheets tracker (URL, Title, Company)
3. Run the workflow manually or on a schedule
4. Review the analysis in your spreadsheet
5. Make the suggested edits to tailor your resume
6. Apply with confidence!

## 🤝 Contributing

Contributions are welcome! Feel free to:
- Report bugs or request features via Issues
- Submit pull requests with improvements
- Share your customizations and use cases

## 📄 License

This project is open source and available under the MIT License.

## ⚠️ Disclaimer

This tool provides AI-generated suggestions. Always review and verify recommendations before making changes to your resume. The AI's feedback should be one input in your job search strategy, not the only one.

## 🙏 Acknowledgments

Built with:
- [n8n](https://n8n.io/) - Workflow automation platform
- [OpenAI GPT-4o-mini](https://openai.com/) - AI analysis engine
- [Google Workspace APIs](https://developers.google.com/) - Document and spreadsheet integration

---

**Made with ❤️ for job seekers everywhere**
