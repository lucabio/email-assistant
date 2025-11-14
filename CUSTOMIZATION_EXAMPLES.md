# Email Assistant - Customization Examples

This guide provides practical examples for common customization requests for the EmailAssistantAndClassifier workflow.

## Table of Contents
1. [Adding VIP Sender Detection](#adding-vip-sender-detection)
2. [Custom Priority Rules](#custom-priority-rules)
3. [Industry-Specific Keywords](#industry-specific-keywords)
4. [Email Digest for Low Priority](#email-digest-for-low-priority)
5. [Auto-Forwarding Specific Categories](#auto-forwarding-specific-categories)
6. [Adding Budget Detection](#adding-budget-detection)
7. [Time-Based Classification Rules](#time-based-classification-rules)

---

## Adding VIP Sender Detection

Add a pre-classification step to automatically flag emails from VIP clients.

### Implementation

**1. Add a Set node after "Extract Email Data":**

```json
{
  "parameters": {
    "assignments": {
      "assignments": [
        {
          "name": "isVIP",
          "value": "={{ ['client1@example.com', 'client2@bigproject.com', 'ceo@luxuryhotel.com'].includes($json.fromEmail.toLowerCase()) }}",
          "type": "boolean"
        },
        {
          "name": "vipBoost",
          "value": "={{ $json.isVIP ? 3 : 0 }}",
          "type": "number"
        }
      ]
    }
  },
  "name": "Check VIP Senders",
  "type": "n8n-nodes-base.set"
}
```

**2. Modify "Prepare AI Classification Prompt":**

Add to the prompt:
```
VIP Sender Status: {{ $json.isVIP ? 'YES - This is a high-priority VIP client' : 'No' }}
```

**3. Adjust priority score in "Parse Classification Result":**

```javascript
priorityScore: classification.priority_score + $('Check VIP Senders').item.json.vipBoost
```

---

## Custom Priority Rules

Override AI classification with business rules for specific scenarios.

### Example: Auto-flag emails with specific keywords

**Add a Code node after "Parse Classification Result":**

```javascript
// Get classification data
const data = $input.item.json;
const subject = data.subject.toLowerCase();
const body = data.bodyPlain.toLowerCase();

// Define urgent keywords
const urgentKeywords = ['urgent', 'asap', 'emergency', 'today', 'immediately'];
const contractKeywords = ['contract', 'agreement', 'proposal', 'quote'];
const budgetKeywords = ['$', 'budget', 'payment', 'invoice'];

// Check for urgent keywords
const hasUrgentKeyword = urgentKeywords.some(keyword =>
  subject.includes(keyword) || body.includes(keyword)
);

// Check for contract-related content
const isContractRelated = contractKeywords.some(keyword =>
  subject.includes(keyword) || body.includes(keyword)
);

// Override classification if needed
let finalClassification = data.classification;
let finalPriority = data.priorityScore;

if (hasUrgentKeyword && isContractRelated) {
  finalClassification = 'URGENT_ACTION_REQUIRED';
  finalPriority = Math.max(finalPriority, 9);
}

// Check for large budget mentions
const budgetMatch = body.match(/\$[\d,]+/g);
if (budgetMatch) {
  const amounts = budgetMatch.map(m => parseInt(m.replace(/[$,]/g, '')));
  const maxAmount = Math.max(...amounts);

  if (maxAmount >= 100000) {
    finalClassification = 'HIGH_PRIORITY_CLIENTS';
    finalPriority = Math.max(finalPriority, 9);
  }
}

return {
  json: {
    ...data,
    classification: finalClassification,
    priorityScore: finalPriority,
    ruleOverride: finalClassification !== data.classification,
    detectedBudget: budgetMatch ? Math.max(...amounts.map(m => parseInt(m.replace(/[$,]/g, '')))) : 0
  }
};
```

---

## Industry-Specific Keywords

Enhance classification with interior design industry context.

### Update the AI Classification Prompt

**In "Prepare AI Classification Prompt" node, add:**

```
**INTERIOR DESIGN INDUSTRY CONTEXT:**

High-Priority Client Indicators:
- Project types: luxury residential, boutique hotels, corporate headquarters, restaurant design
- Client types: developers, hotel chains, corporate real estate, private wealth
- Keywords: "full home renovation", "luxury", "high-end", "bespoke", "custom"

New Business Signals:
- "looking for designer", "interested in your services", "saw your portfolio"
- "project scope", "design consultation", "RFP", "request for proposal"
- "timeline for completion", "budget range"

Vendor/Supplier Patterns:
- Companies: furniture manufacturers, textile suppliers, lighting designers
- Keywords: "catalog", "samples", "lead time", "wholesale pricing", "inventory"
- Typical senders: sales@, orders@, accounts@

Urgent Indicators:
- "client meeting tomorrow", "presentation deadline", "installation date"
- "permit deadline", "contractor waiting", "delivery scheduled"

Common Spam/Low Priority:
- "unsubscribe", "webinar", "free trial", "limited time offer"
- Mass marketing from furniture retailers
- Automated shipping notifications (unless from active project vendors)
```

---

## Email Digest for Low Priority

Instead of notifications for every email, send a daily digest of low-priority items.

### Implementation

**1. Modify workflow to store low-priority emails:**

Add after "Parse Classification Result":

```json
{
  "parameters": {
    "conditions": {
      "conditions": [
        {
          "leftValue": "={{ $json.classification }}",
          "rightValue": "MARKETING_NEWSLETTERS,SPAM_LOW_PRIORITY",
          "operator": {
            "type": "string",
            "operation": "contains"
          }
        }
      ]
    }
  },
  "name": "Is Low Priority",
  "type": "n8n-nodes-base.if"
}
```

**2. Create a separate workflow for daily digest:**

```json
{
  "name": "Daily Email Digest Sender",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "cronExpression",
              "expression": "0 8 * * *"
            }
          ]
        }
      },
      "name": "Schedule Trigger - 8 AM Daily",
      "type": "n8n-nodes-base.scheduleTrigger"
    },
    {
      "parameters": {
        "operation": "read",
        "documentId": "YOUR_GOOGLE_SHEET_ID",
        "sheetName": "Email Classification Log",
        "filters": {
          "conditions": [
            {
              "column": "Timestamp",
              "condition": "after",
              "value": "={{ $now.minus({days: 1}).toISO() }}"
            },
            {
              "column": "Classification",
              "condition": "in",
              "value": "MARKETING_NEWSLETTERS,SPAM_LOW_PRIORITY"
            }
          ]
        }
      },
      "name": "Get Yesterday's Low Priority Emails",
      "type": "n8n-nodes-base.googleSheets"
    },
    {
      "parameters": {
        "content": "=**Daily Low-Priority Email Digest**\n📅 {{ $now.toFormat('MMMM dd, yyyy') }}\n\nYou received {{ $json.length }} low-priority emails yesterday:\n\n{{ $json.map((email, i) => `${i+1}. **${email.Classification}**: ${email.Subject} (from ${email['From Name']})`).join('\\n') }}\n\n[View full log in Google Sheets](https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID)"
      },
      "name": "Format Digest",
      "type": "n8n-nodes-base.set"
    },
    {
      "parameters": {
        "chatId": "YOUR_TELEGRAM_CHAT_ID",
        "text": "={{ $json.content }}"
      },
      "name": "Send Telegram Digest",
      "type": "n8n-nodes-base.telegram"
    }
  ]
}
```

---

## Auto-Forwarding Specific Categories

Automatically forward certain email types to team members.

### Example: Forward vendor emails to operations manager

**Add after "Apply Gmail Label" for VENDOR_SUPPLIER category:**

```json
{
  "parameters": {
    "conditions": {
      "conditions": [
        {
          "leftValue": "={{ $json.classification }}",
          "rightValue": "VENDOR_SUPPLIER",
          "operator": "equals"
        }
      ]
    }
  },
  "name": "Is Vendor Email",
  "type": "n8n-nodes-base.if"
}
```

**Then add Gmail forward node:**

```json
{
  "parameters": {
    "operation": "send",
    "sendTo": "operations@yourcompany.com",
    "subject": "=FWD: {{ $json.subject }}",
    "message": "=Vendor email auto-forwarded from executive inbox.\n\nClassification: {{ $json.gmailLabel }}\nPriority Score: {{ $json.priorityScore }}/10\nAI Analysis: {{ $json.reasoning }}\n\n---Original Email---\nFrom: {{ $json.fromName }} <{{ $json.fromEmail }}>\nDate: {{ $json.receivedDate }}\n\n{{ $json.bodyPlain }}",
    "options": {
      "ccList": "executive@yourcompany.com"
    }
  },
  "name": "Forward to Operations",
  "type": "n8n-nodes-base.gmail"
}
```

---

## Adding Budget Detection

Automatically detect and extract budget information from emails.

### Add after "Extract Email Data":

```javascript
// Budget Detection Code Node
const bodyText = $input.item.json.bodyPlain + ' ' + $input.item.json.subject;

// Regex patterns for currency detection
const patterns = {
  usd: /\$\s?([\d,]+(?:\.\d{2})?)/g,
  range: /\$\s?([\d,]+)\s?-\s?\$?\s?([\d,]+)/g,
  words: /budget of ([\d,]+)/gi
};

let budgets = [];

// Extract dollar amounts
let match;
while ((match = patterns.usd.exec(bodyText)) !== null) {
  const amount = parseFloat(match[1].replace(/,/g, ''));
  if (amount >= 1000) { // Ignore small amounts
    budgets.push(amount);
  }
}

// Extract ranges
while ((match = patterns.range.exec(bodyText)) !== null) {
  const low = parseFloat(match[1].replace(/,/g, ''));
  const high = parseFloat(match[2].replace(/,/g, ''));
  budgets.push(high); // Use high end of range
}

// Determine budget tier
const maxBudget = budgets.length > 0 ? Math.max(...budgets) : 0;
let budgetTier = 'Unknown';

if (maxBudget >= 500000) budgetTier = 'Enterprise (>$500k)';
else if (maxBudget >= 100000) budgetTier = 'High ($100k-$500k)';
else if (maxBudget >= 50000) budgetTier = 'Medium ($50k-$100k)';
else if (maxBudget >= 10000) budgetTier = 'Standard ($10k-$50k)';
else if (maxBudget > 0) budgetTier = 'Small (<$10k)';

return {
  json: {
    ...$input.item.json,
    detectedBudgets: budgets,
    maxBudget: maxBudget,
    budgetTier: budgetTier,
    hasBudgetInfo: budgets.length > 0
  }
};
```

**Then update the AI prompt to include:**
```
Detected Budget: {{ $json.budgetTier }} ({{ $json.maxBudget > 0 ? '$' + $json.maxBudget.toLocaleString() : 'None detected' }})
```

---

## Time-Based Classification Rules

Adjust priority based on time of day or day of week.

### Add after "Extract Email Data":

```javascript
// Time-Based Priority Adjustment
const now = new Date();
const hour = now.getHours();
const day = now.getDay(); // 0 = Sunday, 6 = Saturday

let timeAdjustment = 0;
let timeContext = '';

// After hours (6 PM - 8 AM) - increase priority
if (hour >= 18 || hour < 8) {
  timeAdjustment = 1;
  timeContext = 'Received after business hours';
}

// Weekend
if (day === 0 || day === 6) {
  timeAdjustment = 2;
  timeContext = 'Received on weekend';
}

// Friday afternoon (potential deadline before weekend)
if (day === 5 && hour >= 14) {
  timeAdjustment = 1;
  timeContext = 'Received Friday afternoon';
}

// Monday morning (backlog clearing)
if (day === 1 && hour < 10) {
  timeAdjustment = 0;
  timeContext = 'Monday morning email';
}

return {
  json: {
    ...$input.item.json,
    timeAdjustment: timeAdjustment,
    timeContext: timeContext,
    receivedHour: hour,
    receivedDay: ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'][day]
  }
};
```

**Then in "Parse Classification Result", add:**
```javascript
priorityScore: Math.min(10, classification.priority_score + data.timeAdjustment)
```

---

## Additional Customization Ideas

### 1. Client Relationship Status
- Integrate with CRM to check if sender is existing client
- Boost priority for active project clients
- Flag emails from prospects in pipeline

### 2. Attachment Intelligence
```javascript
// In Extract Email Data node
const attachments = $json.attachments || [];
const hasContract = attachments.some(a =>
  a.filename.includes('contract') ||
  a.filename.includes('agreement') ||
  a.mimeType === 'application/pdf'
);
const hasImages = attachments.some(a => a.mimeType.startsWith('image/'));

return {
  json: {
    ...$input.item.json,
    hasContract: hasContract,
    hasImages: hasImages,
    attachmentCount: attachments.length
  }
};
```

### 3. Sentiment Analysis
Add to AI prompt:
```
Also analyze the email sentiment (positive, neutral, negative, urgent) and urgency level.
Include in JSON: "sentiment": "positive", "urgencyLevel": "high"
```

### 4. Smart Threading
- Check if email is part of existing thread
- Inherit classification from parent email
- Track conversation history

### 5. Response Time Tracking
```javascript
// Log when email arrived vs when responded
const receivedTime = new Date($json.receivedDate);
const respondedTime = new Date(); // When draft created or sent
const responseTimeHours = (respondedTime - receivedTime) / (1000 * 60 * 60);

return {
  json: {
    ...$input.item.json,
    responseTimeHours: Math.round(responseTimeHours * 10) / 10
  }
};
```

---

## Testing Customizations

Always test customizations before activating:

1. **Disable workflow** (toggle off)
2. **Add test node** with sample data:
```json
{
  "fromEmail": "test@example.com",
  "subject": "Test Email with $250,000 budget",
  "bodyPlain": "We have an urgent project...",
  "receivedDate": "2025-11-07T10:00:00Z"
}
```
3. **Execute manually** and verify output
4. **Check all branches** execute correctly
5. **Review logs** for errors
6. **Re-activate** when confident

---

## Performance Optimization

### Reduce AI API Calls

For obvious spam, skip AI classification:

```javascript
// Quick spam detection before AI
const subject = $json.subject.toLowerCase();
const body = $json.bodyPlain.toLowerCase();

const spamIndicators = [
  'unsubscribe',
  'click here now',
  'limited time offer',
  'you have won',
  'claim your prize'
];

const isObviousSpam = spamIndicators.some(indicator =>
  subject.includes(indicator) || body.includes(indicator)
);

if (isObviousSpam) {
  return {
    json: {
      ...$input.item.json,
      classification: 'SPAM_LOW_PRIORITY',
      confidence: 0.95,
      skipAI: true
    }
  };
}
```

### Batch Processing

If receiving high email volume, consider batch processing:
- Collect emails for 15 minutes
- Process in batch
- Send single digest notification

---

## Common Pitfalls to Avoid

1. **Don't over-complicate rules** - AI is good at handling nuance
2. **Test with real data** - Edge cases matter
3. **Monitor false positives** - Adjust thresholds based on feedback
4. **Keep prompts concise** - Long prompts don't always improve accuracy
5. **Version control** - Export workflow before major changes

---

For questions or support, refer to the main SETUP_GUIDE.md or N8N community forums.
