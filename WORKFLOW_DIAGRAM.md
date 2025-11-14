# EmailAssistantAndClassifier - Workflow Diagram

## Visual Workflow Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        EMAIL ASSISTANT WORKFLOW                             │
│                    C-Level Executive - Interior Design                      │
└─────────────────────────────────────────────────────────────────────────────┘

┌───────────────┐
│  Gmail Inbox  │
│  (Trigger)    │◄── Polls every 5 minutes for new emails
└───────┬───────┘
        │
        ▼
┌───────────────────────┐
│  Extract Email Data   │
│  - ID, Subject        │
│  - From Name/Email    │
│  - Body (Plain/HTML)  │
│  - Attachments Info   │
│  - Timestamp          │
└───────────┬───────────┘
            │
            ▼
┌───────────────────────────────────┐
│  Prepare AI Classification        │
│  - Build context prompt           │
│  - Include industry keywords      │
│  - Define 7 categories            │
│  - Add urgency indicators         │
└───────────┬───────────────────────┘
            │
            ▼
┌───────────────────────────────────┐
│     OpenAI GPT-4 (Classify)       │
│  - Analyze sender                 │
│  - Parse subject & body           │
│  - Detect urgency                 │
│  - Assign category                │
│  - Generate confidence score      │
│  - Extract key points             │
└───────────┬───────────────────────┘
            │
            ▼
┌───────────────────────────────────┐
│  Parse Classification Result      │
│  - Extract category               │
│  - Parse confidence (0-1)         │
│  - Parse priority score (1-10)    │
│  - Extract reasoning              │
│  - Format Gmail label name        │
└───────────┬───────────────────────┘
            │
            ├─────────────────────────┬──────────────────────┐
            ▼                         ▼                      ▼
┌─────────────────────┐   ┌─────────────────────┐   ┌─────────────────────┐
│  Check if High      │   │  Apply Gmail Label  │   │  Log to Google      │
│  Priority           │   │  (All Emails)       │   │  Sheets (All)       │
│  - HIGH_PRIORITY_   │   │                     │   │  - Timestamp        │
│    CLIENTS          │   │  Auto-creates       │   │  - Classification   │
│  - URGENT_ACTION_   │   │  label if missing   │   │  - Confidence       │
│    REQUIRED         │   │                     │   │  - Priority Score   │
│  - Priority >= 8    │   └─────────────────────┘   │  - Reasoning        │
└────┬───────────┬────┘                             │  - Key Points       │
     │ YES       │ NO                                └─────────────────────┘
     │           └──────────────┐                            │
     ▼                          │                            │
┌──────────────────────┐        │                            │
│  Send Telegram       │        │                            │
│  Notification        │        │                            │
│  - Priority alert    │        │                            │
│  - Email summary     │        │                            │
│  - AI analysis       │        │                            │
│  - Link to Gmail     │        │                            │
└────┬─────────────────┘        │                            │
     │                          │                            │
     ▼                          │                            │
┌──────────────────────┐        │                            │
│  Is Urgent Action    │        │                            │
│  Required?           │        │                            │
└────┬───────────┬─────┘        │                            │
     │ YES       │ NO           │                            │
     │           └──────────────┤                            │
     ▼                          │                            │
┌──────────────────────┐        │                            │
│  Format Task Details │        │                            │
│  - Title: URGENT:... │        │                            │
│  - Description       │        │                            │
│  - Due: +4 hours     │        │                            │
└────┬─────────────────┘        │                            │
     │                          │                            │
     ▼                          │                            │
┌──────────────────────┐        │                            │
│  [Task Management]   │        │                            │
│  Connect your system:│        │                            │
│  - Google Tasks      │        │                            │
│  - Todoist           │        │                            │
│  - Asana             │        │                            │
│  - ClickUp           │        │                            │
└──────────────────────┘        │                            │
                                │                            │
                ┌───────────────┴────────────────────────────┘
                │
                ▼
        ┌───────────────┐
        │  Merge Paths  │
        └───────┬───────┘
                │
                ▼
┌───────────────────────────────────┐
│  Should Generate Draft Response?  │
│  - NEW_BUSINESS_OPPORTUNITIES     │
│  - VENDOR_SUPPLIER               │
│  - INTERNAL_COMMUNICATIONS       │
└────┬───────────────────────┬──────┘
     │ YES                   │ NO
     │                       └─────► END
     ▼
┌───────────────────────────────────┐
│  Prepare Draft Response Prompt    │
│  - Category-specific template     │
│  - Include email context          │
│  - Add business tone guidelines   │
└────┬──────────────────────────────┘
     │
     ▼
┌───────────────────────────────────┐
│  OpenAI GPT-4 (Generate Draft)    │
│  - Professional tone              │
│  - Context-aware response         │
│  - Executive-level language       │
└────┬──────────────────────────────┘
     │
     ▼
┌───────────────────────────────────┐
│  Create Gmail Draft               │
│  - Save as draft reply            │
│  - Ready for executive review     │
│  - Can edit before sending        │
└───────────────────────────────────┘
                │
                ▼
              [END]
```

## Node Details

### Input Nodes

| Node | Type | Purpose | Configuration |
|------|------|---------|---------------|
| Gmail Trigger | Trigger | Monitor inbox | Polls every 5 min |

### Processing Nodes

| Node | Type | Purpose | Key Parameters |
|------|------|---------|----------------|
| Extract Email Data | Set | Normalize email data | Extract ID, subject, sender, body |
| Prepare AI Prompt | Set | Build classification prompt | Industry context, 7 categories |
| OpenAI Classify | AI | Categorize email | GPT-4, temp=0.3 |
| Parse Result | Code | Extract classification | JSON parsing with fallback |

### Routing Nodes

| Node | Type | Purpose | Conditions |
|------|------|---------|------------|
| Check if High Priority | IF | Route priority emails | Category OR priority >= 8 |
| Is Urgent Action | IF | Detect urgent items | URGENT_ACTION_REQUIRED |
| Should Generate Draft | IF | Determine draft need | 3 specific categories |

### Action Nodes

| Node | Type | Purpose | Integrations |
|------|------|---------|--------------|
| Send Telegram Notification | Telegram | Alert executive | Telegram Bot API |
| Apply Gmail Label | Gmail | Organize inbox | Gmail API |
| Log to Google Sheets | Sheets | Track analytics | Google Sheets API |
| Format Task Details | Set | Prepare task data | Any task system |
| Prepare Draft Prompt | Set | Build draft prompt | Category templates |
| OpenAI Generate Draft | AI | Create response | GPT-4, temp=0.7 |
| Create Gmail Draft | Gmail | Save draft reply | Gmail API |

### Utility Nodes

| Node | Type | Purpose | Details |
|------|------|---------|---------|
| Merge - All Paths | Merge | Combine execution paths | Synchronization point |

## Email Classification Categories

```
┌─────────────────────────────────────────────────────────────┐
│                    EMAIL CATEGORIES                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. HIGH_PRIORITY_CLIENTS                                   │
│     ► VIP clients, large projects (>$100k)                  │
│     ► Urgent client requests                                │
│     ► High-profile client communications                    │
│     Actions: Telegram alert + Label + Log                   │
│                                                             │
│  2. NEW_BUSINESS_OPPORTUNITIES                              │
│     ► Project inquiries, RFPs                               │
│     ► Potential client introductions                        │
│     ► Partnership opportunities                             │
│     Actions: Label + Log + Draft Response                   │
│                                                             │
│  3. URGENT_ACTION_REQUIRED                                  │
│     ► Time-sensitive matters                                │
│     ► Contract deadlines, approvals                         │
│     ► Immediate attention needed                            │
│     Actions: Telegram alert + Label + Log + Task            │
│                                                             │
│  4. INTERNAL_COMMUNICATIONS                                 │
│     ► Team member messages                                  │
│     ► Internal updates, staff coordination                  │
│     Actions: Label + Log + Draft Response                   │
│                                                             │
│  5. VENDOR_SUPPLIER                                         │
│     ► Supplier communications                               │
│     ► Contractors, material vendors                         │
│     ► Service provider updates                              │
│     Actions: Label + Log + Draft Response                   │
│                                                             │
│  6. MARKETING_NEWSLETTERS                                   │
│     ► Marketing emails, industry news                       │
│     ► Promotional content, subscriptions                    │
│     Actions: Label + Log (no notification)                  │
│                                                             │
│  7. SPAM_LOW_PRIORITY                                       │
│     ► Spam, irrelevant content                              │
│     ► Automated notifications                               │
│     ► Low-value emails                                      │
│     Actions: Label + Log (no notification)                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Data Flow

```
┌──────────────┐
│  Gmail API   │──► Email Object
└──────────────┘    │
                    ├─ id
                    ├─ subject
                    ├─ from (name, email)
                    ├─ date
                    ├─ plainText
                    ├─ html
                    └─ attachments[]

                        ▼ Transform

┌──────────────────────────────────────────────────────┐
│  Normalized Email Data                               │
│  {                                                   │
│    emailId: string                                   │
│    subject: string                                   │
│    fromName: string                                  │
│    fromEmail: string                                 │
│    bodyPlain: string                                 │
│    bodyHtml: string                                  │
│    receivedDate: ISO string                          │
│    hasAttachments: boolean                           │
│    processedTimestamp: ISO string                    │
│  }                                                   │
└──────────────────────────────────────────────────────┘

                        ▼ AI Processing

┌──────────────────────────────────────────────────────┐
│  Classification Result                               │
│  {                                                   │
│    category: "HIGH_PRIORITY_CLIENTS"                 │
│    confidence: 0.95 (0-1)                            │
│    reasoning: "High-value client with urgent..."     │
│    priorityScore: 9 (1-10)                           │
│    suggestedAction: "Respond within 2 hours..."      │
│    keyPoints: ["$500k project", "deadline Mon"]      │
│    gmailLabel: "High Priority Clients"               │
│  }                                                   │
└──────────────────────────────────────────────────────┘

                        ▼ Actions

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Telegram   │  │  Gmail Label │  │ Google Sheets│
│ Notification │  │    Applied   │  │   Row Added  │
└──────────────┘  └──────────────┘  └──────────────┘
```

## Integration Points

```
┌─────────────────────────────────────────────────────────┐
│              EXTERNAL INTEGRATIONS                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Gmail (Google Workspace)                               │
│  ├─ Trigger: Poll for new emails                        │
│  ├─ Action: Apply labels                                │
│  └─ Action: Create draft replies                        │
│                                                         │
│  OpenAI (GPT-4)                                         │
│  ├─ Classification (temp=0.3, max_tokens=500)           │
│  └─ Draft Generation (temp=0.7, max_tokens=500)         │
│                                                         │
│  Telegram (Bot API)                                     │
│  └─ Action: Send formatted notifications                │
│                                                         │
│  Google Sheets                                          │
│  └─ Action: Append classification log                   │
│                                                         │
│  [Optional] Task Management                             │
│  └─ Action: Create urgent tasks                         │
│     - Google Tasks                                      │
│     - Todoist                                           │
│     - Asana                                             │
│     - ClickUp                                           │
│     - Trello                                            │
│     - Microsoft To Do                                   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Execution Flow Timing

```
Timeline for processing one email:

T+0.0s  ─► Gmail Trigger fires (new email detected)
T+0.5s  ─► Email data extracted and normalized
T+1.0s  ─► AI classification prompt prepared
T+1.5s  ─► OpenAI API call initiated
T+4.0s  ─► Classification result received
T+4.5s  ─► Result parsed and validated
T+5.0s  ─► [Branch 1] Telegram notification sent (if high priority)
         ─► [Branch 2] Gmail label applied
         ─► [Branch 3] Google Sheets row added
T+6.0s  ─► Paths merged
T+6.5s  ─► Draft check performed
T+7.0s  ─► [If needed] Draft generation prompt prepared
T+7.5s  ─► [If needed] OpenAI API call for draft
T+10.0s ─► [If needed] Draft reply created in Gmail
T+10.5s ─► Execution complete

Total: 5-10 seconds per email (depending on draft generation)
```

## Error Handling

```
┌─────────────────────────────────────────────────────────┐
│                   ERROR SCENARIOS                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Gmail API Error                                        │
│  ├─ Retry: Automatic (3 attempts)                       │
│  ├─ Fallback: Log error, skip email                     │
│  └─ Alert: Check credentials                            │
│                                                         │
│  OpenAI API Error                                       │
│  ├─ Timeout: 30 second limit                            │
│  ├─ Fallback: Default to SPAM_LOW_PRIORITY              │
│  └─ Alert: Check API key and credits                    │
│                                                         │
│  Classification Parse Error                             │
│  ├─ Fallback: Safe default classification               │
│  ├─ Reasoning: "Failed to parse AI response"            │
│  └─ Continue: Workflow doesn't stop                     │
│                                                         │
│  Telegram Send Error                                    │
│  ├─ Retry: 1 attempt                                    │
│  ├─ Fallback: Continue without notification             │
│  └─ Note: Email still labeled and logged                │
│                                                         │
│  Google Sheets Error                                    │
│  ├─ Retry: 2 attempts                                   │
│  ├─ Fallback: Continue workflow                         │
│  └─ Note: Email still labeled                           │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## Performance Characteristics

| Metric | Value | Notes |
|--------|-------|-------|
| Processing Time | 5-10s | Per email |
| Classification Accuracy | 95%+ | With proper setup |
| Throughput | 360-720/hour | Based on 5-10s each |
| Daily Capacity | 8,640-17,280 | Theoretical max |
| Recommended Load | <1,000/day | For reliability |
| Concurrent Executions | 1 | Sequential processing |

## Resource Usage

| Resource | Usage | Cost Factor |
|----------|-------|-------------|
| N8N Executions | 1 per email | Plan-dependent |
| OpenAI Tokens (Classification) | ~500 | $0.01 per email |
| OpenAI Tokens (Draft) | ~800 | $0.016 per email |
| Gmail API Calls | 2-3 per email | Free (quota: 1B/day) |
| Sheets API Calls | 1 per email | Free (quota: 500/min) |
| Telegram API Calls | 0-1 per email | Free (unlimited) |

---

**Diagram Version:** 1.0
**Last Updated:** 2025-11-07
**Workflow:** EmailAssistantAndClassifier
