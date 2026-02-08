# Workflow Documentation

Technical documentation for the Resume Fine Tuner n8n workflow.

## Overview

This document provides in-depth technical details about how the workflow operates, the data flow between nodes, and the logic behind each component.

## Workflow Architecture

```
Manual Trigger
    ↓
Read Resume (Google Docs)
    ↓
Read All Job Rows (Google Sheets)
    ↓
Loop Over Items (Batch Processing)
    ↓
HTTP Request (Scrape Job Description)
    ↓
Conditional Check (If data exists)
    ├─→ True: Use scraped data
    └─→ False: Use manual description
    ↓
Merge Data
    ↓
AI Agent (Analyze Resume vs Job)
    ↓
Parse AI Response (Clean JSON)
    ↓
Update Row in Sheet (Write Results)
```

## Node-by-Node Breakdown

### 1. Manual Trigger
**Type:** `n8n-nodes-base.manualTrigger`

**Purpose:** Initiates the workflow on-demand.

**Configuration:**
- No parameters required
- Can be replaced with Schedule Trigger for automation

**Output:** Single execution signal

---

### 2. Read Resume
**Type:** `n8n-nodes-base.googleDocs`

**Purpose:** Fetches the current version of your master resume.

**Configuration:**
```json
{
  "operation": "get",
  "documentURL": "<YOUR_GOOGLE_DOC_ID>"
}
```

**Output Structure:**
```json
{
  "content": "Full resume text...",
  "documentId": "...",
  "title": "Resume"
}
```

**Key Points:**
- Reads the entire document as plain text
- Updates automatically reflect in next execution
- Requires Google Docs OAuth2 credential

---

### 3. Read All Job Rows
**Type:** `n8n-nodes-base.googleSheets`

**Purpose:** Fetches all job application entries from the tracker spreadsheet.

**Configuration:**
```json
{
  "documentId": "<YOUR_SHEET_ID>",
  "sheetName": "Job Applications",
  "operation": "get",
  "options": {
    "dataLocationOnSheet": {
      "rangeDefinition": "specifyRangeA1",
      "range": "A:D"
    }
  }
}
```

**Output Structure (per row):**
```json
{
  "Job URL": "https://...",
  "Job Title": "Senior Developer",
  "Company": "TechCorp",
  "Job Description": "Optional description..."
}
```

**Key Points:**
- Only reads columns A through D
- Header row is automatically detected
- Empty rows are skipped

---

### 4. Loop Over Items
**Type:** `n8n-nodes-base.splitInBatches`

**Purpose:** Processes job listings one at a time to avoid rate limits and enable error recovery.

**Configuration:**
```json
{
  "options": {},
  "batchSize": 1
}
```

**Behavior:**
- Takes array of jobs from previous node
- Outputs one job at a time
- Loops back to itself until all items processed
- Prevents overwhelming the API

---

### 5. HTTP Request
**Type:** `n8n-nodes-base.httpRequest`

**Purpose:** Attempts to scrape the job description from the provided URL.

**Configuration:**
```json
{
  "url": "={{ $json['Job URL'] }}",
  "method": "GET",
  "options": {
    "response": {
      "responseFormat": "text"
    },
    "timeout": 15000
  },
  "onError": "continueRegularOutput"
}
```

**Error Handling:**
- `onError: continueRegularOutput` ensures workflow continues if scraping fails
- Some sites block scrapers or require authentication
- Falls back to manual description in column D

**Output:**
```json
{
  "data": "HTML content or text from job posting..."
}
```

**Limitations:**
- Cannot bypass login walls or CAPTCHAs
- May violate terms of service for some sites
- Consider ethical implications of scraping

---

### 6. If (Conditional)
**Type:** `n8n-nodes-base.if`

**Purpose:** Routes workflow based on whether job description was successfully scraped.

**Configuration:**
```json
{
  "conditions": {
    "conditions": [
      {
        "leftValue": "={{ $json['data'] }}",
        "operator": "notEmpty"
      }
    ]
  }
}
```

**Logic:**
- **True path:** Scraped data exists → use it directly
- **False path:** No scraped data → fetch from Google Sheets column D

**Output Routing:**
- True → Merge node (input 0)
- False → Edit Fields node

---

### 7. Edit Fields
**Type:** `n8n-nodes-base.set`

**Purpose:** Creates a fallback data structure when scraping fails.

**Configuration:**
```json
{
  "assignments": [
    {
      "name": "job_description",
      "value": "={{ $('Read All Job Rows').item.json['Job Description'] }}"
    }
  ]
}
```

**Purpose:**
- Retrieves manual job description from spreadsheet
- Ensures consistent data structure for merge

---

### 8. Merge
**Type:** `n8n-nodes-base.merge`

**Purpose:** Combines scraped data or manual description with resume content.

**Configuration:**
```json
{
  "mode": "combine",
  "combineBy": "combineAll"
}
```

**Input Sources:**
1. Scraped job description (from HTTP Request via If-True)
2. Manual job description (from Edit Fields via If-False)

**Output:**
- Combined data with both resume and job description available
- All previous node data accessible via `$(nodeName)`

---

### 9. OpenAI Chat Model
**Type:** `@n8n/n8n-nodes-langchain.lmChatOpenAi`

**Purpose:** Provides the AI model for analysis.

**Configuration:**
```json
{
  "model": "gpt-4o-mini"
}
```

**Model Choice:**
- **gpt-4o-mini**: Cost-effective, good for structured outputs
- Can be changed to `gpt-4` for higher quality (higher cost)
- Connected to AI Agent node via LangChain

**Cost Considerations:**
- gpt-4o-mini: ~$0.15 per 1M input tokens, ~$0.60 per 1M output tokens
- Typical resume analysis: ~2,000-4,000 tokens total
- Estimated cost: $0.001-0.003 per job analysis

---

### 10. Analyze Resume vs. Job
**Type:** `@n8n/n8n-nodes-langchain.agent`

**Purpose:** The core AI analysis engine that compares resume to job posting.

**Input Prompt Structure:**
```
JOB POSTING:
Title: {{ Job Title }}
Company: {{ Company }}
URL: {{ Job URL }}

RESUME:
{{ Resume Content }}

Description:
{{ Job Description }}
```

**System Prompt:** See [AI Prompt Documentation](#ai-prompt-details) below

**Output:** JSON string with structured analysis

**Configuration:**
```json
{
  "promptType": "define",
  "options": {
    "systemMessage": "<detailed prompt>",
    "enableStreaming": false
  }
}
```

---

### 11. Parse AI Response
**Type:** `n8n-nodes-base.code`

**Purpose:** Cleans and structures the AI's JSON response for spreadsheet insertion.

**Functions:**

#### 1. `cleanText(value)`
Processes text fields to ensure proper formatting:
- Removes JSON artifacts (`["`, `"]`)
- Converts `\n` to actual line breaks
- Adds line breaks before bullet points
- Cleans up double line breaks
- Handles arrays by joining with newlines

#### 2. Main Parsing Logic
```javascript
// Extract AI response from various possible fields
const aiResponse = $input.first().json.text || 
                   $input.first().json.response || 
                   $input.first().json.output;

// Clean markdown code fences
const cleanResponse = aiResponse
  .replace(/```json\n?/g, '')
  .replace(/```\n?/g, '')
  .trim();

// Parse JSON
const parsed = JSON.parse(cleanResponse);
```

#### 3. Error Handling
```javascript
try {
  // parsing logic
} catch (e) {
  // Returns error structure with default values
  parsed = {
    alignment_score: "Error",
    score_rationale: "Could not parse AI response",
    // ... other fields
  };
}
```

**Output Structure:**
```json
{
  "job_url": "https://...",
  "alignment_score": "7",
  "score_rationale": "Strong technical match...",
  "keywords": "Python, AWS, Docker...",
  "strengths": "- Led team of 5 engineers...",
  "improve_strengths": "- Change 'Led team' to...",
  "weaknesses": "- Missing Kubernetes experience...",
  "improve_weaknesses": "- Add section highlighting...",
  "tailoring_suggestions": "1. Update summary to..."
}
```

---

### 12. Update Row in Sheet
**Type:** `n8n-nodes-base.googleSheets`

**Purpose:** Writes analysis results back to the job tracker spreadsheet.

**Configuration:**
```json
{
  "operation": "update",
  "documentId": "<YOUR_SHEET_ID>",
  "sheetName": "Job Applications",
  "columnToMatchOn": "Job URL",
  "valueToMatchOn": "={{ $json['job_url'] }}"
}
```

**Column Mappings:**
- `alignment_score` → "Alignment Score"
- `score_rationale` → "Score Rationale"
- `keywords` → "Keywords"
- `strengths` → "Strengths"
- `improve_strengths` → "Improve Strengths"
- `weaknesses` → "Weaknesses"
- `improve_weaknesses` → "Improve Weaknesses"
- `tailoring_suggestions` → "Tailoring Suggestions"

**Update Logic:**
- Matches on "Job URL" column
- Updates only the matched row
- Creates columns if they don't exist
- Preserves existing data in unmatched columns

---

## AI Prompt Details

### System Prompt Philosophy

The prompt is designed to be:
1. **Brutally Honest** – No sugarcoating or generic advice
2. **Specific** – References exact resume content
3. **Actionable** – Provides copy-paste rewrites
4. **ATS-Focused** – Prioritizes keyword optimization

### Scoring Rubric

```
9-10: Near-perfect match. Interview-ready today.
7-8:  Strong match with minor gaps. Quick fixes needed.
5-6:  Decent foundation but meaningful gaps. Real work required.
3-4:  Weak match. Major missing qualifications.
1-2:  Wrong fit. Resume doesn't support this application.
```

### JSON Response Schema

```json
{
  "alignment_score": "<number 1-10>",
  "score_rationale": "<one honest sentence>",
  "keywords": "<comma-separated ATS keywords>",
  "strengths": "<bullet list with specific examples>",
  "improve_strengths": "<specific rewrites for each strength>",
  "weaknesses": "<bullet list of gaps and missing elements>",
  "improve_weaknesses": "<concrete fixes for each weakness>",
  "tailoring_suggestions": "<top 5 highest-impact changes, numbered>"
}
```

### Prompt Rules

1. **Never give generic advice** – Every suggestion must reference specific content
2. **If missing critical requirements, say so directly** – No softening
3. **Prioritize ATS optimization** – Flag missing keywords that cause filtering
4. **When suggesting improvements, write actual copy** – Not descriptions
5. **Be specific about WHERE changes go** – Don't just say "improve your summary"
6. **Use `\n` for line breaks** – Ensures proper formatting in sheets

---

## Data Flow Example

Let's trace a single job through the workflow:

### Input (from Google Sheets)
```json
{
  "Job URL": "https://example.com/careers/senior-dev",
  "Job Title": "Senior Python Developer",
  "Company": "TechCorp",
  "Job Description": ""
}
```

### After HTTP Request
```json
{
  "data": "<p>We're seeking a Senior Python Developer with...</p>"
}
```

### After Merge
```json
{
  "Job URL": "https://example.com/careers/senior-dev",
  "Job Title": "Senior Python Developer",
  "Company": "TechCorp",
  "data": "We're seeking a Senior Python Developer with...",
  "resume_content": "John Doe\nSoftware Engineer\n..."
}
```

### After AI Analysis
```json
{
  "text": "```json\n{\"alignment_score\": 8, \"score_rationale\": ...}\n```"
}
```

### After Parsing
```json
{
  "job_url": "https://example.com/careers/senior-dev",
  "alignment_score": "8",
  "score_rationale": "Strong Python background...",
  "keywords": "Python, Django, PostgreSQL, AWS...",
  "strengths": "- 5 years Python development\n- Led Django migration...",
  "improve_strengths": "- Change '5 years Python' to...",
  "weaknesses": "- No AWS certification mentioned\n- Limited CI/CD...",
  "improve_weaknesses": "- Add bullet: 'Implemented AWS...'",
  "tailoring_suggestions": "1. Add AWS certification to header..."
}
```

### Final Output (to Google Sheets)
All parsed fields are written to their respective columns in the matched row.

---

## Performance Considerations

### Processing Time
- **Per job:** 15-30 seconds
  - HTTP request: 1-5 seconds
  - AI analysis: 10-20 seconds
  - Sheet update: 1-2 seconds

### Rate Limits
- **OpenAI API:** 3,500 requests/min (Tier 1)
- **Google Sheets API:** 100 requests/100 seconds/user
- **HTTP scraping:** Depends on target site

### Optimization Tips
1. Process jobs in batches of 5-10 if using higher API tier
2. Add 1-second delay between HTTP requests
3. Cache resume content if processing many jobs
4. Use webhook trigger for real-time updates

---

## Error Handling Strategy

### HTTP Request Failures
- **Strategy:** Continue workflow, use fallback description
- **Implementation:** `onError: continueRegularOutput`

### AI Parsing Errors
- **Strategy:** Return error structure with default values
- **Implementation:** Try-catch in Parse AI Response node

### Sheet Update Failures
- **Strategy:** Fail loudly to avoid data loss
- **Implementation:** Default n8n error handling (stops workflow)

---

## Security Considerations

### Credentials
- All OAuth tokens stored encrypted by n8n
- API keys never exposed in workflow JSON
- Use credential references, not hardcoded keys

### Data Privacy
- Resume content sent to OpenAI API
- Job descriptions may be copyrighted
- Consider data residency requirements

### Best Practices
1. Use environment variables for sensitive IDs
2. Rotate API keys regularly
3. Review OpenAI's data usage policies
4. Don't share workflow JSON with embedded credentials

---

## Extending the Workflow

### Add More Data Sources
```javascript
// Example: Parse JSON API instead of HTML scraping
{
  "url": "https://api.example.com/jobs/{{ $json['job_id'] }}",
  "headers": {
    "Authorization": "Bearer {{ $credentials.apiToken }}"
  },
  "options": {
    "response": {
      "responseFormat": "json"
    }
  }
}
```

### Add Notifications
After "Update row in sheet":
```javascript
// Send email with results
{
  "to": "{{ $credentials.email }}",
  "subject": "Resume Analysis Complete: {{ $json['Job Title'] }}",
  "body": "Score: {{ $json['alignment_score'] }}\n\n{{ $json['tailoring_suggestions'] }}"
}
```

### Add Resume Version Control
Before "Analyze Resume vs. Job":
```javascript
// Create versioned copy in Drive
{
  "operation": "copy",
  "fileId": "{{ $credentials.resumeId }}",
  "name": "Resume - {{ $json['Company'] }} - {{ $now }}"
}
```

---

## Maintenance

### Regular Tasks
- [ ] Review AI prompt effectiveness monthly
- [ ] Update model if newer versions available
- [ ] Monitor API costs and optimize if needed
- [ ] Clean up old analysis data in spreadsheet

### Troubleshooting Checklist
1. Check credential expiration
2. Verify document IDs are correct
3. Review recent n8n execution logs
4. Test HTTP scraping manually
5. Validate AI responses in OpenAI Playground
6. Ensure spreadsheet structure hasn't changed

---

## Changelog

### Version 1.0 (Current)
- Initial release
- GPT-4o-mini integration
- Google Workspace integration
- Basic error handling

### Planned Features
- [ ] Support for multiple resume versions
- [ ] LinkedIn integration
- [ ] Email notifications
- [ ] Batch processing optimization
- [ ] Custom scoring criteria per industry
- [ ] Resume diff tracking

---

For questions or contributions, see the main [README](README.md).
