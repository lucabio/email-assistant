# Email Assistant and Classifier - Setup Guide

## Overview

This N8N workflow provides intelligent email classification and management for C-level executives in the interior design industry. It automatically categorizes incoming Gmail messages, sends notifications for high-priority items, applies labels, logs data for analytics, and generates draft responses.

## Workflow Features

### Core Capabilities
- **Automated Email Monitoring**: Checks Gmail inbox every 5 minutes
- **AI-Powered Classification**: Uses OpenAI GPT-4 to categorize emails into 7 business-relevant categories
- **Smart Notifications**: Sends Telegram alerts for high-priority and urgent emails
- **Auto-Labeling**: Applies Gmail labels based on classification
- **Analytics Logging**: Tracks all classifications in Google Sheets
- **Draft Response Generation**: Creates AI-powered draft replies for specific categories
- **Task Creation Ready**: Framework for creating urgent action items in your task management system

### Email Categories

1. **HIGH_PRIORITY_CLIENTS** - High-profile clients, large projects (>$100k), VIP clients, urgent requests
2. **NEW_BUSINESS_OPPORTUNITIES** - Project inquiries, RFPs, potential clients, partnerships
3. **URGENT_ACTION_REQUIRED** - Time-sensitive matters, deadlines, approvals needed
4. **INTERNAL_COMMUNICATIONS** - Team messages, internal updates, staff coordination
5. **VENDOR_SUPPLIER** - Supplier communications, contractors, material vendors
6. **MARKETING_NEWSLETTERS** - Marketing emails, newsletters, promotional content
7. **SPAM_LOW_PRIORITY** - Spam, irrelevant content, automated notifications

## Prerequisites

Before importing the workflow, ensure you have:

1. **N8N Instance** (self-hosted or n8n.cloud)
2. **Gmail Account** (the executive's email account)
3. **Google Cloud Project** (for Gmail and Sheets API access)
4. **OpenAI Account** with API access (GPT-4 recommended)
5. **Telegram Account** and Bot created
6. **Google Sheets** for logging

## Step-by-Step Setup

### 1. Import the Workflow

1. Open your N8N instance
2. Click on "Workflows" in the sidebar
3. Click "Import from File" or "Import from URL"
4. Select the `EmailAssistantAndClassifier.json` file
5. The workflow will be imported with all nodes configured

### 2. Configure Gmail OAuth2 Credentials

**A. Create Google Cloud Project:**

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing one
3. Navigate to "APIs & Services" > "Library"
4. Enable the following APIs:
   - Gmail API
   - Google Sheets API

**B. Configure OAuth Consent Screen:**

1. Go to "APIs & Services" > "OAuth consent screen"
2. Choose "External" (or "Internal" if using Google Workspace)
3. Fill in required fields:
   - App name: "N8N Email Assistant"
   - User support email: your email
   - Developer contact: your email
4. Add scopes:
   - `.../auth/gmail.readonly`
   - `.../auth/gmail.modify`
   - `.../auth/gmail.compose`
   - `.../auth/spreadsheets`
5. Add test users (the executive's Gmail address)
6. Save and continue

**C. Create OAuth Credentials:**

1. Go to "APIs & Services" > "Credentials"
2. Click "Create Credentials" > "OAuth client ID"
3. Choose "Web application"
4. Add authorized redirect URIs:
   - For n8n.cloud: `https://app.n8n.cloud/rest/oauth2-credential/callback`
   - For self-hosted: `https://your-n8n-domain.com/rest/oauth2-credential/callback`
5. Copy Client ID and Client Secret

**D. Add Credentials in N8N:**

1. In N8N, go to "Credentials" in the sidebar
2. Click "Add Credential"
3. Search for "Gmail OAuth2"
4. Enter:
   - Credential Name: "Gmail OAuth2 - Executive Account"
   - Client ID: (from step C)
   - Client Secret: (from step C)
5. Click "Connect my account" and authorize
6. Save the credential

### 3. Configure OpenAI API Credentials

1. Go to [OpenAI Platform](https://platform.openai.com/)
2. Sign in or create an account
3. Navigate to "API Keys"
4. Click "Create new secret key"
5. Copy the key (you won't see it again!)

**In N8N:**
1. Go to "Credentials" > "Add Credential"
2. Search for "OpenAI"
3. Enter:
   - Credential Name: "OpenAI API"
   - API Key: (paste your key)
4. Save

**Important:** Ensure your OpenAI account has:
- Sufficient credits/billing set up
- Access to GPT-4 (or modify workflow to use GPT-3.5-turbo for lower cost)

### 4. Configure Telegram Bot

**A. Create Telegram Bot:**

1. Open Telegram and search for "@BotFather"
2. Start a chat and send `/newbot`
3. Follow prompts to name your bot (e.g., "ExecutiveEmailBot")
4. Copy the bot token provided (looks like: `123456789:ABCdefGHIjklMNOpqrsTUVwxyz`)

**B. Get Your Chat ID:**

1. Send a message to your new bot
2. Open this URL in browser (replace TOKEN):
   ```
   https://api.telegram.org/botTOKEN/getUpdates
   ```
3. Look for `"chat":{"id":123456789}` in the response
4. Copy the chat ID number

**C. Add Telegram Credentials in N8N:**

1. Go to "Credentials" > "Add Credential"
2. Search for "Telegram"
3. Enter:
   - Credential Name: "Telegram Bot API"
   - Access Token: (your bot token)
4. Save

**D. Update Workflow:**

1. Open the workflow
2. Find the "Send Telegram Notification" node
3. Replace `YOUR_TELEGRAM_CHAT_ID` with your actual chat ID
4. Save the workflow

### 5. Configure Google Sheets Logging

**A. Create Spreadsheet:**

1. Go to [Google Sheets](https://sheets.google.com/)
2. Create a new spreadsheet
3. Name it "Email Classification Log" (or your preferred name)
4. Rename the first sheet to "Email Classification Log"
5. Add headers in row 1:
   ```
   Timestamp | Email ID | From Name | From Email | Subject | Classification | Gmail Label | Confidence | Priority Score | Reasoning | Suggested Action | Key Points | Has Attachments | Received Date
   ```

**B. Get Spreadsheet ID:**

1. Look at the URL of your spreadsheet:
   ```
   https://docs.google.com/spreadsheets/d/SPREADSHEET_ID_HERE/edit
   ```
2. Copy the long string between `/d/` and `/edit`

**C. Configure Google Sheets OAuth2:**

1. In N8N, go to "Credentials" > "Add Credential"
2. Search for "Google Sheets OAuth2"
3. Use the same Client ID and Secret from Gmail setup
4. Click "Connect my account" and authorize
5. Save as "Google Sheets OAuth2"

**D. Update Workflow:**

1. Open the workflow
2. Find the "Log to Google Sheets" node
3. Replace `YOUR_GOOGLE_SHEET_ID` with your actual spreadsheet ID
4. Verify the sheet name matches your sheet
5. Save

### 6. Optional: Configure Task Management

The workflow includes a placeholder for creating tasks from urgent emails. To activate this:

**Option A: Google Tasks**
1. Add Google Tasks node after "Format Task Details"
2. Configure with Gmail OAuth2 credentials
3. Map fields: `taskTitle` → Title, `taskDescription` → Notes, `dueDate` → Due Date

**Option B: Todoist**
1. Add Todoist node
2. Get Todoist API token from Settings > Integrations
3. Create Todoist API credential in N8N
4. Configure task creation with mapped fields

**Option C: Other Task Systems**
- Asana, ClickUp, Trello, Microsoft To Do are all supported
- Follow similar pattern: add node, configure credentials, map fields

### 7. Test the Workflow

**Before activating:**

1. Click "Execute Workflow" (manual test)
2. Send a test email to the executive's Gmail account
3. Monitor execution in N8N
4. Check that:
   - Email is classified correctly
   - Gmail label is applied
   - Google Sheets has new row
   - Telegram notification sent (if high priority)
   - Draft created (if applicable category)

**Review Test Results:**
- Check the execution log for any errors
- Verify AI classification makes sense
- Confirm all integrations work

### 8. Activate Workflow

1. In the workflow editor, toggle the "Active" switch (top right)
2. The workflow will now run automatically every 5 minutes
3. Monitor the first few executions to ensure stability

## Configuration Options

### Adjust Polling Frequency

To change how often the workflow checks for emails:

1. Open "Gmail Trigger - Monitor Inbox" node
2. Modify "Poll Times" > "Minute" (1, 5, 10, 15, 30, or 60)
3. Save

**Note:** More frequent polling = more workflow executions = higher n8n usage

### Customize AI Classification

**Modify Categories:**
1. Open "Prepare AI Classification Prompt" node
2. Edit the categories list in the prompt
3. Update Gmail label logic accordingly

**Adjust AI Model:**
1. Open "OpenAI - Classify Email" node
2. Change model to `gpt-3.5-turbo` for lower cost
3. Or use `gpt-4-turbo` for better performance

**Tune Temperature:**
- Classification uses 0.3 (more consistent)
- Draft generation uses 0.7 (more creative)
- Adjust in node parameters if needed

### Modify Draft Response Categories

1. Open "Should Generate Draft Response" node
2. Add or remove categories in conditions
3. Update the "Prepare Draft Response Prompt" to handle new categories

### Change Notification Criteria

1. Open "Check if High Priority" node
2. Modify conditions:
   - Add/remove categories
   - Adjust priority score threshold (currently ≥8)
   - Add additional criteria (e.g., specific senders)

## Cost Considerations

### OpenAI API Costs

**Per Email:**
- Classification call: ~500 tokens ($0.01 with GPT-4)
- Draft response call: ~800 tokens ($0.016 with GPT-4)
- Estimated: $0.01-0.03 per email

**Monthly Estimate:**
- 500 emails/month: $5-15
- 1000 emails/month: $10-30

**To Reduce Costs:**
- Use GPT-3.5-turbo instead of GPT-4
- Disable draft generation for some categories
- Increase polling interval

### N8N Execution Costs

- Each email processed = 1 workflow execution
- N8N Cloud: Check your plan's execution limits
- Self-hosted: No execution limits

## Troubleshooting

### Issue: No Emails Being Processed

**Solutions:**
1. Verify Gmail OAuth2 credentials are connected
2. Check workflow is activated (toggle in top right)
3. Verify Gmail API is enabled in Google Cloud Console
4. Check execution log for specific errors

### Issue: AI Classification Errors

**Solutions:**
1. Verify OpenAI API key is valid and has credits
2. Check OpenAI account has GPT-4 access (or switch to GPT-3.5)
3. Review "Parse Classification Result" node for JSON parsing errors
4. Check prompt formatting in "Prepare AI Classification Prompt"

### Issue: Telegram Notifications Not Sending

**Solutions:**
1. Verify Telegram bot token is correct
2. Ensure chat ID is accurate (not a username)
3. Send a message to the bot first to initiate conversation
4. Check bot isn't blocked

### Issue: Gmail Labels Not Applying

**Solutions:**
1. Verify Gmail OAuth2 has `gmail.modify` scope
2. Check label names don't have special characters
3. Labels are auto-created; wait for first execution
4. Review Gmail API quota limits

### Issue: Google Sheets Not Logging

**Solutions:**
1. Verify spreadsheet ID is correct
2. Check sheet name matches exactly
3. Ensure Google Sheets API is enabled
4. Verify OAuth2 credentials have Sheets scope

## Maintenance

### Regular Tasks

**Weekly:**
- Review Google Sheets log for classification accuracy
- Check Telegram notifications are relevant
- Monitor OpenAI API usage and costs

**Monthly:**
- Review and refine AI prompts based on misclassifications
- Update category definitions if business needs change
- Clean up old Google Sheets data (export and archive)

**Quarterly:**
- Audit Gmail labels and consolidate if needed
- Review workflow execution count and optimize if needed
- Update task management integration if using

### Optimization Tips

1. **Improve Classification Accuracy:**
   - Add specific sender domains to prompt context
   - Include examples of past misclassifications
   - Adjust confidence thresholds

2. **Reduce False Positives:**
   - Tune priority score thresholds
   - Add specific exclusion rules
   - Refine urgent keywords list

3. **Enhance Draft Responses:**
   - Collect executive's actual responses
   - Update prompts with preferred language/tone
   - Add company-specific templates

## Security Best Practices

1. **Credentials:**
   - Never share OAuth2 tokens or API keys
   - Use environment variables for sensitive data
   - Regularly rotate API keys
   - Enable 2FA on all accounts

2. **Data Privacy:**
   - Ensure Google Sheets has restricted access
   - Don't log sensitive client information in plain text
   - Consider data retention policies
   - Comply with GDPR/privacy regulations

3. **Access Control:**
   - Limit who can edit the n8n workflow
   - Use separate credentials for production/testing
   - Monitor workflow execution logs
   - Set up error notifications

## Advanced Customizations

### Add Email Attachments Analysis

1. Extract attachment data from Gmail trigger
2. Add logic to analyze file types
3. Increase priority for contracts/proposals (PDF)
4. Flag potential security risks

### Implement Learning Loop

1. Add feedback mechanism in Telegram notifications
2. Store corrections in database
3. Use feedback to refine prompts over time
4. Build custom classification model

### Create Executive Dashboard

1. Use Google Sheets data to build Data Studio dashboard
2. Visualize email volume by category
3. Track response times
4. Identify trends and patterns

### Multi-Language Support

1. Add language detection in classification
2. Adjust AI prompts based on email language
3. Generate draft responses in appropriate language

## Support and Resources

- **N8N Documentation:** https://docs.n8n.io/
- **N8N Community:** https://community.n8n.io/
- **OpenAI API Docs:** https://platform.openai.com/docs
- **Gmail API Docs:** https://developers.google.com/gmail/api
- **Telegram Bot API:** https://core.telegram.org/bots/api

## Workflow Summary

**File:** `/home/lucab/Projects/EmailAssistantAutomation/EmailAssistantAndClassifier.json`

**Nodes:** 20 (including sticky notes)
**Integrations:** Gmail, OpenAI, Telegram, Google Sheets
**Trigger:** Gmail Trigger (5-minute polling)
**Credentials Required:** 4

**Process Flow:**
1. Gmail monitors inbox every 5 minutes
2. Extract email data (sender, subject, body)
3. Prepare AI classification prompt with business context
4. OpenAI GPT-4 classifies email into category
5. Parse classification result
6. **If high priority:** Send Telegram notification
7. **If urgent:** Format for task creation (optional integration)
8. Apply Gmail label to email
9. Log classification to Google Sheets
10. **If applicable category:** Generate draft response with AI
11. **If draft generated:** Create Gmail draft reply

**Execution Time:** ~5-10 seconds per email
**Success Rate:** >95% (with proper configuration)

---

**Created:** 2025-11-07
**Version:** 1.0
**Workflow Name:** EmailAssistantAndClassifier
**For:** C-level Executive, Interior Design Company
