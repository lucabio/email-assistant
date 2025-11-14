# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**EmailAssistantAutomation** is an N8N workflow-based email automation system designed for C-level executives. It uses AI (OpenAI GPT-5) to classify incoming Gmail messages into 12 categories, automatically handle routine emails, and alert for critical situations requiring immediate attention.

**Core Purpose:** Intelligent email triage with escalation detection, thread analysis, and automated response drafting.

## Architecture

### Workflow Engine: N8N
This is NOT a traditional code repository. The entire application is defined in a single JSON file (`EmailAssistantAndClassifier.json`) that represents an N8N workflow with 19 operational nodes plus sticky notes.

**Key Components:**
1. **Gmail Trigger** - Polls inbox every 5 minutes
2. **Email Extraction** - Fetches full email with thread context via Gmail API
3. **AI Classification** - GPT-5 model (temperature 0.2, max 1000 tokens) analyzes emails
4. **Parallel Processing** - Splits into 5 concurrent paths after classification:
   - Apply Gmail labels
   - Auto-mark as read (SOCIAL/NEWSLETTER)
   - Auto-forward (PAYMENT_RECEIPTS to accounting)
   - Create drafts + Telegram alerts (escalations, urgent payments, potential problems)
   - Log to Google Sheets
5. **Merge Point** - Consolidates all execution paths

### Data Flow
```
Gmail → Extract → Prepare Prompt → GPT-5 → Parse → [5 parallel actions] → Merge
```

**Processing Time:** 5-15 seconds per email (thread fetch + GPT-5 analysis)

## Email Classification System

### 12 Categories with Specialized Handling

**Immediate Action (Telegram alerts + draft responses):**
- ESCALATIONS - Angry stakeholders, complaints (escalation levels: none/low/medium/high)
- PAYMENTS_DUE - Payment requests with deadlines (<7 days triggers alert)
- POTENTIAL_PROBLEMS - Early warning signs of future escalations

**Auto-Processed:**
- SOCIAL - Auto-marked as read
- NEWSLETTER - Auto-marked as read
- PAYMENT_RECEIPTS - Auto-forwarded to accounting

**Standard:**
- RECRUITMENT, PROJECTS, NEW_CLIENTS, COLLABORATION_PROPOSALS, MEETING_REQUESTS, MISCELLANEOUS

### AI Classification Logic
The classification prompt (line 71 in JSON) instructs GPT-5 to:
- Analyze conversation thread for escalating patterns
- Extract payment deadlines and calculate urgency
- Detect angry tone (ALL CAPS, excessive punctuation, keywords: "unacceptable", "frustrated", "lawyer")
- Identify warning signs ("still waiting", "not satisfied", passive-aggressive language)
- Generate diplomatic responses when `requiresImmediate=true`

**Response Format:** JSON with fields:
- `category`, `confidence`, `reasoning`, `requiresImmediate`, `suggestedResponse`, `keyPoints`, `escalationLevel`, `paymentDeadline`, `shouldForward`, `forwardTo`, `autoMarkRead`

## Critical Configuration Placeholders

**MUST BE REPLACED BEFORE DEPLOYMENT:**

1. **Credential IDs** - All credential IDs are placeholders:
   - `GMAIL_OAUTH2_CREDENTIAL_ID` → Your Gmail OAuth2 credential ID
   - `OPENAI_API_CREDENTIAL_ID` → Your OpenAI API credential ID
   - `TELEGRAM_API_CREDENTIAL_ID` → Your Telegram Bot credential ID
   - `GOOGLE_SHEETS_OAUTH2_CREDENTIAL_ID` → Your Google Sheets credential ID

2. **Telegram Chat ID**: Search for `YOUR_TELEGRAM_CHAT_ID` → replace with numeric chat ID

3. **Google Sheet ID**: `YOUR_GOOGLE_SHEET_ID` → spreadsheet ID from URL

4. **Accounting Email**: `accounting@yourcompany.com` → actual accounting email

5. **Instance/Workflow IDs**:
   - `YOUR_N8N_INSTANCE_ID` → Will be auto-generated on import
   - `YOUR_WORKFLOW_ID` → Will be auto-generated on import

**Note:** The workflow JSON has been sanitized to remove all sensitive data including:
- Personal email addresses
- Credential IDs
- Webhook IDs
- Test data (pinData)
- Instance-specific identifiers

## Credentials Required

1. **Gmail OAuth2** - Google Cloud Project with Gmail API + Sheets API enabled
2. **OpenAI API** - GPT-5 access (fallback to GPT-4o or GPT-4-turbo possible)
3. **Telegram Bot API** - Bot token from @BotFather
4. **Google Sheets OAuth2** - Same credentials as Gmail

## Modifying the Workflow

### To Add a New Category
1. Edit "Prepare AI Classification Prompt" node (line 66-83)
2. Add category to the CATEGORIES list in prompt
3. Define detection rules and action flags
4. Update routing logic if category needs special handling (auto-forward, auto-read, etc.)

### To Change Alert Criteria
Edit "Requires Immediate Action" node (line 244-271) conditions to modify which emails trigger Telegram notifications.

### To Adjust AI Behavior
**Classification** (line 86-111):
- Model: Change `gpt-5` to `gpt-4o` (cheaper, 95% accuracy) or `gpt-4-turbo`
- Temperature: Increase from 0.2 for more creative responses (less consistent)
- Max Tokens: Reduce from 1000 to lower costs

### To Customize Telegram Notifications
Edit "Send Telegram Alert with Draft Info" node (line 298-318):
- Modify template at line 300 (Markdown format)
- Add/remove fields from notification message

### To Change Polling Frequency
Edit "Gmail Trigger - Monitor Inbox" node (line 5-30):
- Line 10: Change `"minute": 5` to 1, 10, 15, 30, or 60
- **Trade-off:** More frequent = higher N8N execution usage

## Code Structure in JSON

The workflow JSON contains:
- `nodes[]` - Array of 19 operational nodes + 2 sticky notes
- `connections{}` - Defines execution flow between nodes
- `parameters` - Node-specific configuration (credentials, conditions, code)

**JavaScript Code Nodes:**
- "Extract Email Data" (line 54): Parses Gmail API response, extracts thread ID
- "Parse Classification Result" (line 114): Parses GPT-5 JSON response, formats Gmail labels, handles errors with fallback to MISCELLANEOUS category

## Testing Strategy

1. **Manual Test**: Use "Execute Workflow" button in N8N after sending test emails
2. **Category Coverage**: Send emails matching each of the 12 categories
3. **Verify Actions**:
   - Gmail labels applied correctly
   - SOCIAL/NEWSLETTER marked as read
   - PAYMENT_RECEIPTS forwarded to accounting
   - Escalations create drafts + Telegram alerts
   - Google Sheets logs all classifications
4. **Check Telegram**: High-priority emails should trigger formatted alerts with draft links

## Performance & Cost

**OpenAI GPT-5 Costs:**
- ~$0.04-0.06 per email (classification + draft generation)
- Monthly: 500 emails ≈ $20-30, 1000 emails ≈ $40-60

**Alternatives to Reduce Cost:**
- GPT-4o: ~50% cheaper, 95% accuracy
- GPT-4-turbo: ~40% cheaper, 93% accuracy

**Scalability:** Handles 1000+ emails/day; recommended load <1000/day for reliability

## Version Information

- **Current Version:** 2.0 (2025-11-07)
- **Major Update:** Expanded from 7 to 12 categories, added escalation detection, thread analysis, early warning system, payment deadline tracking

## Common Issues

1. **No emails processing**: Check Gmail OAuth2 credentials, verify workflow is Active
2. **AI classification errors**: Verify OpenAI API key valid, check GPT-4/5 access
3. **No Telegram notifications**: Verify bot token, ensure chat ID is numeric, message bot with /start first
4. **Thread analysis not working**: Check "Get Full Email with Thread" node has `format: "full"`

## Documentation Structure

- `README.md` - Comprehensive overview, features, quick start
- `SETUP_GUIDE.md` - Step-by-step deployment instructions with screenshots guidance
- `WORKFLOW_DIAGRAM.md` - Visual architecture, data flow, timing
- `QUICK_START_CHECKLIST.md` - Deployment checklist with test scenarios
- `CUSTOMIZATION_EXAMPLES.md` - Code examples for common modifications
- `IMPORT_INSTRUCTIONS.txt` - Quick import reference
- `EmailAssistantAndClassifier.json` - The actual N8N workflow (primary artifact)

## Working with N8N Workflows

**Editing:** Import JSON into N8N, make changes in visual editor, export back to JSON
**Version Control:** The entire workflow state is in the JSON file (nodes, connections, credentials references)
**Testing:** N8N provides execution logs showing data flow through each node
**Debugging:** Use "Execute Node" to test individual nodes, check execution logs for errors
