# Read Professional Portfolio Command

**Shortcut**: `/read-portfolio`
**Author**: <AUTHOR_NAME>
**Purpose**: Read and extract content from professional portfolio site using Playwright

---

## Usage

```bash
# Basic usage - Extract all content
/read-portfolio

# Extract specific sections
/read-portfolio skills
/read-portfolio experience
/read-portfolio projects

# Update documentation with portfolio data
/read-portfolio update-docs
```

---

## What It Does

Uses Playwright MCP server to:
1. Navigate to https://abhishekpandeyoracle.netlify.app/abhishekbit731
2. Extract content (skills, experience, projects)
3. Optionally update local documentation

---

## Examples

### Example 1: Read Full CV
```
Use Playwright to navigate to https://abhishekpandeyoracle.netlify.app/abhishekbit731
and extract all visible text content. Organize the output into sections:
- Personal Info
- Skills
- Experience
- Projects
- Education
```

### Example 2: Extract Skills Only
```
Use Playwright to navigate to my CV at https://abhishekpandeyoracle.netlify.app/abhishekbit731
and extract only the skills section. List all technologies, languages, and tools mentioned.
```

### Example 3: Update PROJECT_CONTEXT.md
```
1. Use Playwright to read my CV at https://abhishekpandeyoracle.netlify.app/abhishekbit731
2. Extract my current skills and expertise
3. Update .ai-config/PROJECT_CONTEXT.md with the latest information
4. Preserve existing project-specific context
```

### Example 4: Compare CV with Current Work
```
1. Read my CV from https://abhishekpandeyoracle.netlify.app/abhishekbit731
2. Compare listed skills with technologies used in current storage interface project
3. Identify gaps or new skills to add to CV
4. Generate recommendations
```

### Example 5: Generate Summary
```
Use Playwright to read my CV at https://abhishekpandeyoracle.netlify.app/abhishekbit731
and generate a 100-word professional summary highlighting:
- Years of experience
- Key technologies
- Major achievements
- Current role/focus
```

---

## Technical Details

**MCP Server**: Playwright (`@executeautomation/playwright-mcp-server`)

**Allowed Operations**:
- `playwright_navigate` - Navigate to URL
- `playwright_screenshot` - Take screenshots
- `playwright_content` - Extract page content
- `playwright_evaluate` - Run JavaScript
- `playwright_pdf` - Generate PDF

**URL**: https://abhishekpandeyoracle.netlify.app/abhishekbit731

---

## Use Cases

1. **Keep Documentation Updated**: Sync CV skills with `.ai-config/PROJECT_CONTEXT.md`
2. **Generate Cover Letters**: Extract experience for job applications
3. **Skills Gap Analysis**: Compare CV with job requirements
4. **Portfolio Review**: Screenshot portfolio for presentations
5. **Automated Updates**: Check if CV reflects current projects

---

## Prerequisites

```bash
# Install Playwright browsers
npx playwright install chromium

# Verify Playwright MCP server
npx -y @executeautomation/playwright-mcp-server --version

# Load environment (if using auth)
source .env.mcp
```

---

## Troubleshooting

### Issue: Playwright fails to load page
```bash
# Test manually
npx playwright codegen https://abhishekpandeyoracle.netlify.app/abhishekbit731

# Check internet connection
curl -I https://abhishekpandeyoracle.netlify.app/abhishekbit731
```

### Issue: Content extraction incomplete
```javascript
// Run JavaScript in browser to extract specific elements
"Use Playwright to navigate to my CV and run this JavaScript:
document.querySelectorAll('h2').forEach(h => console.log(h.textContent))"
```

### Issue: Screenshot quality
```
"Use Playwright to navigate to my CV and take a full-page screenshot
at 1920x1080 resolution, save as cv-screenshot.png"
```

---

**@author <AUTHOR_NAME>**
