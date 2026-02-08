# Frequently Asked Questions (FAQ)

## General Questions

### What is Resume Fine Tuner?

Resume Fine Tuner is an n8n workflow automation that uses AI to analyze your resume against job postings. It provides specific, actionable feedback to help you tailor your resume for each application.

### How much does it cost?

The tool itself is free and open-source. However, you'll need:
- **n8n:** Free tier available, or self-host for free
- **OpenAI API:** Pay-per-use, ~$0.001-0.003 per job analysis (gpt-4o-mini)
- **Google Workspace:** Free with any Google account

**Estimated cost:** Processing 100 jobs ≈ $0.10-0.30 in API fees

### Do I need coding experience?

No! The workflow is pre-built. You just need to:
1. Import the JSON file into n8n
2. Connect your Google and OpenAI accounts
3. Update document IDs
4. Run the workflow

Basic understanding of spreadsheets is helpful but not required.

### Is my data private?

Your resume and job descriptions are sent to:
- **OpenAI's API** for analysis (subject to OpenAI's privacy policy)
- **Google's servers** (subject to Google's privacy policy)

The workflow runs in your own n8n instance, so no data is stored on our servers. Consider OpenAI's data retention policies if privacy is a concern.

---

## Setup Questions

### Where do I get the document IDs?

**For Google Docs:**
1. Open your resume in Google Docs
2. Look at the URL: `https://docs.google.com/document/d/DOCUMENT_ID/edit`
3. Copy the `DOCUMENT_ID` portion

**For Google Sheets:**
1. Open your job tracker
2. Look at the URL: `https://docs.google.com/spreadsheets/d/SHEET_ID/edit`
3. Copy the `SHEET_ID` portion

### How do I create Google OAuth credentials?

See the detailed [Setup Guide](SETUP.md#step-2-configure-google-docs-credential) for complete instructions. In summary:
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project
3. Enable Google Docs and Sheets APIs
4. Create OAuth 2.0 credentials
5. Add authorized redirect URIs for n8n

### What if I don't have an OpenAI API key?

1. Go to [OpenAI Platform](https://platform.openai.com/)
2. Sign up or log in
3. Go to **API Keys** in your account
4. Click **Create new secret key**
5. Copy it immediately (you won't see it again!)
6. Add billing information (required to use the API)

### Can I use a different AI model?

Yes! In the "OpenAI Chat Model" node, you can change the model to:
- `gpt-4o` – Highest quality, more expensive
- `gpt-4o-mini` – Default, best value
- `gpt-3.5-turbo` – Fastest, cheapest (may be less accurate)

---

## Usage Questions

### How many jobs can I process at once?

The workflow processes jobs one at a time to avoid rate limits. However, you can:
- Add as many jobs as you want to your spreadsheet
- The workflow will loop through all of them
- Estimated processing: ~20-30 seconds per job

For 50 jobs, expect ~20-25 minutes of runtime.

### Can I schedule automatic runs?

Yes! Replace the "Manual Trigger" node with a "Schedule Trigger":
1. Delete the "Manual Trigger" node
2. Add a "Schedule Trigger" node
3. Configure it (e.g., "Every Monday at 9 AM")
4. Connect it to "Read Resume"

### What if a job URL is blocked from scraping?

The workflow will automatically fall back to using the description you manually entered in column D of your spreadsheet. To improve success rates:
- Try using the "Careers" or "Jobs" page URL directly
- For LinkedIn jobs, manual description is often necessary
- Some sites require authentication (manual description needed)

### How accurate is the AI analysis?

The AI provides valuable insights, but it's not perfect:
- **Good at:** Keyword matching, structure analysis, ATS optimization
- **Limited:** Understanding context, industry nuance, personal fit

Always review suggestions critically and apply your own judgment.

### Can I customize the feedback?

Yes! Edit the system prompt in the "Analyze Resume vs. Job" node. You can:
- Change the tone (more encouraging, less harsh)
- Add industry-specific criteria
- Request different output categories
- Adjust scoring rubric

---

## Troubleshooting

### "Document not found" error

**Cause:** Incorrect document ID or missing permissions

**Solutions:**
1. Double-check the document ID is correct
2. Ensure the Google account used in n8n has access to the document
3. Try making the document "Anyone with the link can view"
4. Re-authenticate your Google credentials in n8n

### "Could not parse AI response" error

**Cause:** OpenAI API error, rate limit, or malformed response

**Solutions:**
1. Check your OpenAI API key is valid
2. Verify you have billing set up at OpenAI
3. Check for rate limit errors in n8n logs
4. Try running with just one job to isolate the issue
5. Review the raw AI response in the execution log

### HTTP Request fails for all URLs

**Cause:** Network restrictions or blocked user agent

**Solutions:**
1. Check if your n8n instance has internet access
2. Try accessing the URL manually in a browser
3. Some sites block automated requests
4. Use manual job descriptions in column D as workaround

### Google Sheets not updating

**Cause:** Credential issues or incorrect configuration

**Solutions:**
1. Check OAuth credentials haven't expired
2. Verify sheet name is exactly "Job Applications"
3. Ensure the "Job URL" column exists and has data
4. Try re-creating the Google Sheets credential

### Workflow times out

**Cause:** Large resume, long job descriptions, or API delays

**Solutions:**
1. Increase timeout in HTTP Request node (default: 15 seconds)
2. Process fewer jobs at once
3. Consider using a faster AI model
4. Check your internet connection speed

### Output formatting is messy

**Cause:** JSON parsing issues or incorrect line break handling

**Solutions:**
1. Check the "Parse AI Response" node's `cleanText()` function
2. Verify the AI is using `\n` for line breaks
3. Manually format cells in Google Sheets (Format → Text wrapping → Wrap)
4. Consider adjusting the system prompt to enforce formatting

---

## Advanced Questions

### Can I use this with multiple resumes?

Currently, the workflow reads one master resume. To use multiple:

**Option 1:** Manual switching
- Update the document ID in "Read Resume" for different resume versions
- Run the workflow separately for each

**Option 2:** Add resume selection logic
- Modify the workflow to read from a "Resume Version" column in your sheet
- Add conditional logic to select the correct resume document

### Can I integrate with LinkedIn?

LinkedIn doesn't provide a public API for job scraping, but you can:
- Manually copy job descriptions to column D
- Use a LinkedIn data extraction tool first
- Consider using LinkedIn's Apply with Seek API (requires approval)

### How do I export the results?

The results are already in Google Sheets! You can:
- Download the sheet as Excel/CSV: File → Download
- Use Google Sheets API to export programmatically
- Add an n8n "Export" node to generate PDF reports

### Can I analyze cover letters too?

With modifications, yes:
1. Add a "Read Cover Letter" node (Google Docs)
2. Modify the AI prompt to include cover letter analysis
3. Add new columns for cover letter feedback
4. Update the "Parse AI Response" node for new fields

### What if I want to compare against industry standards?

You could:
1. Create a separate "Industry Standards" Google Doc
2. Include it in the AI prompt
3. Ask the AI to compare your resume vs industry best practices
4. This would require prompt engineering

---

## Best Practices

### For Best Results

1. **Keep your master resume comprehensive** – Include all experience, even if not relevant to every job
2. **Update regularly** – Changes to your resume automatically reflect in the next run
3. **Review AI suggestions critically** – Not all advice will be relevant to your situation
4. **Track changes** – Keep a version history of your resume before making major edits
5. **Test with a few jobs first** – Don't process 100 jobs on your first run

### Resume Writing Tips

Based on how the AI analyzes resumes:

- **Use industry keywords** – The AI scans for keyword matches
- **Quantify achievements** – Numbers stand out to both AI and humans
- **Match job requirements** – Mirror the language in the job posting
- **Keep formatting simple** – Complex formatting can confuse parsing
- **Be specific** – Vague descriptions hurt your alignment score

### Job Application Strategy

1. **Start with high-score matches (8+)** – These are your strongest opportunities
2. **Fix low-score weaknesses** – If a job scores 4-5, major tailoring is needed
3. **Don't ignore mid-scores (6-7)** – These are often worth the effort
4. **Track application outcomes** – Add columns for "Applied" and "Response" to learn what works

---

## Common Misconceptions

### "The AI will write my resume for me"

❌ **Not quite.** The AI provides suggestions and rewrites, but you need to:
- Review each suggestion
- Adapt it to your true experience
- Ensure accuracy and honesty
- Maintain your unique voice

### "A score of 10 guarantees an interview"

❌ **Not true.** The score reflects resume-job alignment, not:
- Your actual qualifications
- Company culture fit
- Timing and competition
- Hiring manager preferences

### "I should apply to all 10-score jobs"

✅ **Consider carefully.** High alignment is great, but also think about:
- Do you actually want this job?
- Is the compensation acceptable?
- Does it align with your career goals?
- Are there red flags in the company?

### "I need to implement every suggestion"

❌ **Selective is better.** Focus on:
- High-impact changes (from "Tailoring Suggestions")
- Missing critical keywords
- Major weaknesses
- Changes that feel authentic to you

---

## Still Have Questions?

1. Check the [README](README.md) for overview
2. Review the [Setup Guide](SETUP.md) for installation
3. Read the [Workflow Documentation](WORKFLOW_DOCS.md) for technical details
4. Open an [Issue](https://github.com/your-repo/issues) on GitHub
5. Start a [Discussion](https://github.com/your-repo/discussions) for general questions

---

**Last Updated:** February 2026
