# Contributing to Resume Fine Tuner

Thank you for considering contributing to Resume Fine Tuner! This document provides guidelines and instructions for contributing.

## How Can I Contribute?

### Reporting Bugs

Before creating bug reports, please check existing issues to avoid duplicates. When creating a bug report, include:

- **Clear title and description**
- **Steps to reproduce** the issue
- **Expected vs actual behavior**
- **Screenshots** if applicable
- **Environment details:**
  - n8n version
  - Self-hosted or cloud
  - Node.js version (if self-hosted)
  - OpenAI model used

**Bug Report Template:**
```markdown
### Description
[Clear description of the bug]

### Steps to Reproduce
1. [First step]
2. [Second step]
3. [...]

### Expected Behavior
[What you expected to happen]

### Actual Behavior
[What actually happened]

### Environment
- n8n version: 
- Hosting: [cloud/self-hosted]
- OpenAI model: 
- Browser (if relevant): 

### Additional Context
[Screenshots, error logs, etc.]
```

### Suggesting Enhancements

Enhancement suggestions are welcome! Please provide:

- **Clear use case** – Why is this enhancement needed?
- **Proposed solution** – How would you implement it?
- **Alternatives considered** – What other approaches did you think about?
- **Impact** – Who would benefit from this change?

### Pull Requests

1. **Fork the repository** and create your branch from `main`
2. **Make your changes** with clear, descriptive commits
3. **Test your changes** thoroughly
4. **Update documentation** if needed
5. **Submit a pull request** with a clear description

## Development Setup

### Prerequisites
- n8n instance (local or cloud)
- Node.js 18+ (for testing custom code)
- Google account for testing integrations
- OpenAI API key for testing AI features

### Testing Your Changes

1. **Import the modified workflow** into n8n
2. **Test with sample data:**
   - Create a test resume document
   - Add test job listings
   - Run the workflow end-to-end
3. **Verify outputs:**
   - Check Google Sheets updates
   - Validate JSON parsing
   - Test error handling
4. **Export the workflow** as JSON

### Code Style

#### JavaScript in Code Nodes
```javascript
// Use clear variable names
const jobDescription = $json['Job Description'];

// Add comments for complex logic
// Extract text and remove HTML tags
const cleanText = html.replace(/<[^>]*>/g, '');

// Use try-catch for error handling
try {
  const parsed = JSON.parse(response);
} catch (error) {
  console.error('Parse error:', error);
  // Provide fallback
}
```

#### Workflow Organization
- **Clear node names** – "Analyze Resume vs. Job" not "AI Agent"
- **Logical flow** – Left to right, top to bottom
- **Error handling** – Use `onError` appropriately
- **Comments** – Add notes for complex logic

## Contribution Guidelines

### What We're Looking For

**High Priority:**
- Bug fixes with test cases
- Performance optimizations
- Better error handling
- Documentation improvements
- New integration options (LinkedIn, Indeed, etc.)

**Medium Priority:**
- UI/UX improvements in output formatting
- Additional AI prompt templates
- Support for more languages
- Resume template detection

**Lower Priority:**
- Cosmetic changes
- Personal preference adjustments

### What to Avoid

- Breaking changes without discussion
- Removing existing features
- Adding dependencies without justification
- Undocumented changes

## Specific Contribution Ideas

### Beginner-Friendly
- [ ] Add more example job descriptions to documentation
- [ ] Create video tutorial for setup
- [ ] Improve error messages
- [ ] Add FAQ section
- [ ] Fix typos in documentation

### Intermediate
- [ ] Add support for PDF resume parsing
- [ ] Create alternative AI prompt templates
- [ ] Add email notification feature
- [ ] Implement resume version tracking
- [ ] Add batch processing optimization

### Advanced
- [ ] Integration with LinkedIn API
- [ ] Custom NLP for keyword extraction
- [ ] Multi-language support
- [ ] Machine learning scoring model
- [ ] Chrome extension for one-click analysis

## AI Prompt Contributions

If you'd like to improve or add alternative AI prompts:

1. **Test thoroughly** with diverse resume types
2. **Document the use case** (e.g., "For technical roles" or "Entry-level focus")
3. **Provide examples** of outputs
4. **Include scoring rubric** if different from default

Example contribution:
```javascript
// For creative industry roles
const creativePrompt = `
You are a creative director reviewing portfolios.
Focus on:
- Creative achievements and awards
- Portfolio presentation
- Unique voice and style
- Industry-specific keywords
...
`;
```

## Documentation Contributions

Documentation is crucial! We welcome:

- **Tutorials** for specific use cases
- **Troubleshooting guides** for common issues
- **Integration guides** for new platforms
- **Translation** to other languages
- **Video walkthroughs**

Documentation style:
- Clear, concise language
- Step-by-step instructions
- Screenshots where helpful
- Code examples with syntax highlighting

## Testing

### Manual Testing Checklist

Before submitting a PR, verify:

- [ ] Workflow imports successfully
- [ ] All credentials work after import
- [ ] Resume is read correctly
- [ ] Job listings are processed
- [ ] HTTP scraping handles errors
- [ ] AI analysis completes
- [ ] JSON parsing succeeds
- [ ] Spreadsheet updates correctly
- [ ] Error handling works as expected

### Test Cases to Consider

1. **Valid job with scrapable URL** → Full analysis
2. **Job with blocked URL** → Falls back to manual description
3. **Job with no description** → Handles gracefully
4. **Malformed AI response** → Returns error structure
5. **Invalid credentials** → Clear error message
6. **Empty spreadsheet** → No processing, no errors
7. **Large resume (10+ pages)** → Completes within timeout

## Communication

### Questions?
- Open a **Discussion** for general questions
- Open an **Issue** for bugs or feature requests
- Tag maintainers for urgent items

### Response Times
- We aim to respond to issues within 3-5 business days
- PRs are typically reviewed within 1 week
- Complex changes may take longer

### Code Reviews
- Be open to feedback
- Respond to review comments
- Make requested changes promptly
- Ask questions if unclear

## Recognition

Contributors will be:
- Listed in README.md acknowledgments
- Mentioned in release notes for significant contributions
- Credited in code comments for major features

## License

By contributing, you agree that your contributions will be licensed under the MIT License.

## Getting Help

Stuck? Need clarification?

1. Check the [README](README.md) and [SETUP.md](SETUP.md)
2. Search existing issues
3. Open a discussion thread
4. Tag @maintainers for help

---

Thank you for making Resume Fine Tuner better! 🙌
