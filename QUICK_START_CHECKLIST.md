# Email Assistant - Quick Start Checklist

Use this checklist to deploy the EmailAssistantAndClassifier workflow in production.

## Pre-Import Preparation

### Accounts & Services Setup

- [ ] **N8N Instance Ready**
  - [ ] N8N Cloud account OR self-hosted instance running
  - [ ] Admin access to n8n interface
  - [ ] Note your n8n URL: `___________________________`

- [ ] **Gmail Account Access**
  - [ ] Executive's Gmail email: `___________________________`
  - [ ] Can access Gmail settings
  - [ ] 2FA configured (recommended)

- [ ] **Google Cloud Project**
  - [ ] Created project at console.cloud.google.com
  - [ ] Project name: `___________________________`
  - [ ] Billing enabled (free tier sufficient for testing)

- [ ] **OpenAI Account**
  - [ ] Account created at platform.openai.com
  - [ ] Billing/payment method added
  - [ ] API key generated: `sk-...` (keep secure!)
  - [ ] GPT-4 access confirmed (or plan to use GPT-3.5)

- [ ] **Telegram**
  - [ ] Telegram app installed
  - [ ] Created bot via @BotFather
  - [ ] Bot token saved: `___________________________`
  - [ ] Chat ID obtained: `___________________________`

- [ ] **Google Sheets**
  - [ ] Spreadsheet created
  - [ ] Spreadsheet ID copied: `___________________________`
  - [ ] Headers added to first row

---

## Import & Configuration

### Step 1: Import Workflow (5 minutes)

- [ ] Downloaded `EmailAssistantAndClassifier.json`
- [ ] Opened N8N interface
- [ ] Clicked "Workflows" > "Import from File"
- [ ] Selected the JSON file
- [ ] Workflow imported successfully
- [ ] Workflow visible in N8N

### Step 2: Gmail OAuth2 Setup (15 minutes)

- [ ] Enabled Gmail API in Google Cloud Console
- [ ] Enabled Google Sheets API in Google Cloud Console
- [ ] Configured OAuth consent screen
- [ ] Added test user (executive's email)
- [ ] Created OAuth 2.0 Client ID (Web application)
- [ ] Copied Client ID: `___________________________`
- [ ] Copied Client Secret: `___________________________`
- [ ] Added OAuth redirect URI to Google Cloud Console
- [ ] Created "Gmail OAuth2" credential in N8N
- [ ] Connected and authorized Gmail account
- [ ] Authorization successful (green checkmark)

### Step 3: OpenAI Setup (5 minutes)

- [ ] Created "OpenAI API" credential in N8N
- [ ] Added API key
- [ ] Tested connection (credential shows as valid)
- [ ] Verified OpenAI account has credits

### Step 4: Telegram Setup (10 minutes)

- [ ] Created Telegram bot
- [ ] Sent message to bot to start conversation
- [ ] Retrieved chat ID using getUpdates API
- [ ] Created "Telegram API" credential in N8N
- [ ] Added bot token
- [ ] Updated workflow node with actual chat ID
  - [ ] Found "Send Telegram Notification" node
  - [ ] Replaced `YOUR_TELEGRAM_CHAT_ID` with real ID
  - [ ] Saved workflow

### Step 5: Google Sheets Setup (10 minutes)

- [ ] Created spreadsheet "Email Classification Log"
- [ ] Added sheet named "Email Classification Log"
- [ ] Added column headers:
  ```
  Timestamp | Email ID | From Name | From Email | Subject | Classification |
  Gmail Label | Confidence | Priority Score | Reasoning | Suggested Action |
  Key Points | Has Attachments | Received Date
  ```
- [ ] Copied spreadsheet ID from URL
- [ ] Created "Google Sheets OAuth2" credential in N8N
- [ ] Connected and authorized Google Sheets
- [ ] Updated workflow with spreadsheet ID
  - [ ] Found "Log to Google Sheets" node
  - [ ] Replaced `YOUR_GOOGLE_SHEET_ID`
  - [ ] Saved workflow

### Step 6: Optional - Task Management (10 minutes)

- [ ] Decided on task system: `___________________________`
- [ ] Created credentials for task system
- [ ] Added task creation node
- [ ] Connected to "Format Task Details" node
- [ ] Configured task fields
- [ ] Tested task creation

---

## Testing Phase

### Pre-Activation Tests

- [ ] **Test 1: Manual Execution**
  - [ ] Sent test email to executive's Gmail
  - [ ] Clicked "Execute Workflow" in N8N
  - [ ] Workflow executed without errors
  - [ ] Execution log shows green checkmarks

- [ ] **Test 2: Gmail Integration**
  - [ ] Email was fetched from Gmail
  - [ ] Email data extracted correctly
  - [ ] Subject, sender, body all captured

- [ ] **Test 3: AI Classification**
  - [ ] OpenAI node executed successfully
  - [ ] Classification returned valid category
  - [ ] Confidence score between 0-1
  - [ ] Priority score between 1-10

- [ ] **Test 4: Gmail Label**
  - [ ] Checked Gmail inbox
  - [ ] Test email has new label applied
  - [ ] Label name matches classification

- [ ] **Test 5: Google Sheets Logging**
  - [ ] Opened spreadsheet
  - [ ] New row added with email data
  - [ ] All columns populated correctly

- [ ] **Test 6: Telegram Notification** (if high priority)
  - [ ] Sent high-priority test email
  - [ ] Executed workflow manually
  - [ ] Received Telegram notification
  - [ ] Notification contains email details
  - [ ] Link to Gmail works

- [ ] **Test 7: Draft Response** (if applicable)
  - [ ] Sent email matching draft categories
  - [ ] Executed workflow manually
  - [ ] Checked Gmail drafts
  - [ ] Draft reply created
  - [ ] Draft content is relevant

### Test Scenarios

Run these specific email tests:

- [ ] **High Priority Client Email**
  - Subject: "Urgent: $500k project approval needed"
  - Expected: HIGH_PRIORITY_CLIENTS, Telegram alert
  - Actual classification: `___________________________`
  - Result: ✓ Pass / ✗ Fail

- [ ] **New Business Inquiry**
  - Subject: "Interested in your design services for hotel project"
  - Expected: NEW_BUSINESS_OPPORTUNITIES, Draft created
  - Actual classification: `___________________________`
  - Result: ✓ Pass / ✗ Fail

- [ ] **Vendor Email**
  - Subject: "New fabric catalog available"
  - Expected: VENDOR_SUPPLIER, Draft created
  - Actual classification: `___________________________`
  - Result: ✓ Pass / ✗ Fail

- [ ] **Internal Email**
  - From: team member
  - Expected: INTERNAL_COMMUNICATIONS
  - Actual classification: `___________________________`
  - Result: ✓ Pass / ✗ Fail

- [ ] **Marketing Newsletter**
  - Subject: "Furniture Sale - 50% Off!"
  - Expected: MARKETING_NEWSLETTERS, No notification
  - Actual classification: `___________________________`
  - Result: ✓ Pass / ✗ Fail

- [ ] **Spam Email**
  - Subject: "You've won a prize! Click here now!"
  - Expected: SPAM_LOW_PRIORITY, No notification
  - Actual classification: `___________________________`
  - Result: ✓ Pass / ✗ Fail

### Issue Resolution

If tests fail, check:

- [ ] All credentials show green/connected status
- [ ] No red error nodes in execution log
- [ ] API quotas not exceeded (Google Cloud Console)
- [ ] OpenAI account has sufficient credits
- [ ] Telegram bot isn't blocked
- [ ] Gmail API scopes include modify and compose
- [ ] Spreadsheet ID is correct (no spaces)
- [ ] Chat ID is numeric (not username)

---

## Production Activation

### Final Checks Before Going Live

- [ ] All test scenarios passed
- [ ] Reviewed execution logs (no errors)
- [ ] Confirmed costs are acceptable:
  - [ ] OpenAI API usage estimate: `$_____/month`
  - [ ] N8N execution count estimate: `_____/month`
- [ ] Stakeholder briefed on:
  - [ ] What the workflow does
  - [ ] How to provide feedback
  - [ ] How to adjust settings if needed
- [ ] Documentation reviewed and accessible

### Activation

- [ ] **Activate Workflow**
  - [ ] Clicked toggle switch to "Active" (top right)
  - [ ] Confirmed workflow is active (green dot)
  - [ ] Noted activation time: `___________________________`

- [ ] **Monitor First Hour**
  - [ ] Checked after 10 minutes - emails processing? ✓ / ✗
  - [ ] Checked after 30 minutes - any errors? ✓ / ✗
  - [ ] Checked after 60 minutes - everything stable? ✓ / ✗

### First Day Monitoring

- [ ] Morning check (9 AM): `___________________________`
- [ ] Midday check (12 PM): `___________________________`
- [ ] Afternoon check (3 PM): `___________________________`
- [ ] End of day check (6 PM): `___________________________`

Track:
- Total emails processed: `_____`
- Classification distribution:
  - High Priority Clients: `_____`
  - New Business: `_____`
  - Urgent: `_____`
  - Internal: `_____`
  - Vendor: `_____`
  - Marketing: `_____`
  - Spam: `_____`
- Errors encountered: `_____`
- False positives/negatives: `_____`

---

## Week One Review

### Performance Metrics (After 7 Days)

- [ ] **Volume Statistics**
  - Total emails processed: `_____`
  - Average per day: `_____`
  - Peak hour: `___________________________`

- [ ] **Classification Accuracy**
  - Reviewed sample of 20 emails
  - Correct classifications: `_____/20` (____%)
  - Common misclassifications: `___________________________`

- [ ] **Cost Analysis**
  - OpenAI API costs: `$_____`
  - N8N executions used: `_____`
  - Within budget? ✓ / ✗

- [ ] **User Feedback**
  - Executive satisfaction: ✓ High / ✓ Medium / ✓ Low
  - Most useful feature: `___________________________`
  - Pain points: `___________________________`

### Optimization Actions

- [ ] Adjusted AI prompt for better accuracy
- [ ] Modified priority thresholds
- [ ] Added VIP sender list
- [ ] Refined draft response templates
- [ ] Updated category definitions
- [ ] Other: `___________________________`

---

## Ongoing Maintenance Schedule

### Weekly Tasks

- [ ] Review Google Sheets log
- [ ] Check for classification patterns
- [ ] Verify Telegram notifications are relevant
- [ ] Monitor OpenAI API usage

### Monthly Tasks

- [ ] Review and refine AI prompts
- [ ] Update VIP sender list
- [ ] Clean/archive old Google Sheets data
- [ ] Review workflow execution count
- [ ] Check credential expiration dates

### Quarterly Tasks

- [ ] Full workflow audit
- [ ] Cost-benefit analysis
- [ ] Stakeholder feedback session
- [ ] Consider new features/integrations

---

## Emergency Contacts & Resources

**Workflow Information:**
- Workflow Name: EmailAssistantAndClassifier
- Activated Date: `___________________________`
- Owner: `___________________________`

**Key Credentials:**
- Gmail OAuth2 ID: `___________________________`
- OpenAI API Key (last 4 digits): `___________________________`
- Telegram Bot: `___________________________`
- Google Sheet: `___________________________`

**Support Resources:**
- N8N Community: https://community.n8n.io/
- Setup Guide: `/SETUP_GUIDE.md`
- Customization Examples: `/CUSTOMIZATION_EXAMPLES.md`
- OpenAI Status: https://status.openai.com/
- Gmail API Status: https://www.google.com/appsstatus

**Emergency Procedures:**

If workflow needs to be disabled immediately:
1. Open N8N workflow
2. Toggle "Active" switch to OFF
3. Workflow stops processing new emails
4. Existing executions complete
5. No data is lost

If credentials are compromised:
1. Immediately revoke in service (Google/OpenAI/Telegram)
2. Delete credentials in N8N
3. Generate new credentials
4. Update workflow with new credentials
5. Reactivate workflow

---

## Success Criteria

The workflow is successful when:

- [ ] 95%+ emails are classified correctly
- [ ] High-priority items reach executive within 5 minutes
- [ ] Draft responses are relevant and helpful
- [ ] Executive saves 1+ hour per day
- [ ] Zero false negatives on urgent/high-priority emails
- [ ] Monthly costs are under budget
- [ ] System runs reliably without daily intervention

---

## Sign-Off

**Implementation Completed By:**
Name: `___________________________`
Date: `___________________________`
Signature: `___________________________`

**Approved By (Executive/IT Manager):**
Name: `___________________________`
Date: `___________________________`
Signature: `___________________________`

---

**Notes & Comments:**
```
[Add any additional notes, special configurations, or lessons learned during setup]







```

---

**Checklist Version:** 1.0
**Last Updated:** 2025-11-07
**Next Review Date:** `___________________________`
