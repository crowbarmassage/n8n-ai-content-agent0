# Quick Reference: n8n AI Content Agent

## CRITICAL PATH (Core - ~90 minutes)

### 1. Setup Checklist
- [ ] n8n cloud account (n8n.io → free tier)
- [ ] Google Cloud project → Enable Sheets API → OAuth credentials
- [ ] Google Sheet with columns: Topic, Status, LinkedIn_Post, X_Post, Blog_Summary, Published_Date, Research_Summary, Quality_Score
- [ ] 2 test rows with Status = "Pending"
- [ ] n8n credentials: Google Sheets OAuth, OpenAI API, Tavily (in body)

### 2. Node Sequence
```
Manual Trigger → Read Topics → Add Row Numbers (Code) → Filter Pending (IF) 
→ Tavily Research (HTTP) → Aggregate Research (Code)
→ [Parallel: LinkedIn | X | Blog] (3 OpenAI nodes)
→ Combine Content (Merge) → Structure Output (Code) → Update Sheet (Google Sheets)
```

### 3. Key Expressions

**Filter Pending IF condition:**
```
{{ $json.Status }}  equals  Pending
```

**Tavily query:**
```
{{ $json.Topic }} latest trends 2024 2025
```

**Row number reference:**
```
{{ $json.row_number }}
```

**Current timestamp:**
```
{{ new Date().toISOString() }}
```

---

## ENHANCEMENT PATH (+30 min)

### Add Scheduled Trigger
- Replace Manual Trigger with Schedule Trigger
- Interval: 15 minutes

### Add Error Handling
- Error Trigger node (standalone)
- Code node: Format error details
- Google Sheets: Append to Error_Log sheet

---

## MAXIMUM PATH (+45 min)

### Add Quality Gate
- OpenAI node after Structure Output
- IF node: quality_pass equals true
- TRUE → Update Sheet (Completed)
- FALSE → Update Sheet (Review Needed)

---

## n8n SHORTCUTS

| Action | How |
|--------|-----|
| Expression mode | Click field → {{ }} icon |
| Test single node | Click node → "Test step" |
| Test workflow | "Execute Workflow" button |
| Reference other node | `$('Node Name').first().json` |
| Current item | `$json.fieldName` |
| All items | `$input.all()` |

---

## COMMON ISSUES & FIXES

**"row_number is undefined"**
→ Add Code node after Read Topics that adds row_number = index + 2

**"Cannot read property of undefined"**
→ Check node connections, ensure previous node output matches expected input

**Tweet exceeds 280 chars**
→ Lower max_tokens to 80, add "STRICT 280 CHARACTER LIMIT" to prompt

**Tavily returns empty**
→ Check API key is correct, try simpler query without date filters

**Google Sheets auth fails**
→ Re-authenticate in n8n credentials, check OAuth consent screen is published

---

## SUBMISSION FILES

```
submission/
├── workflow.json          # Download from n8n
├── README.md              # Use template, add your samples
└── workflow-screenshot.png # Zoom to fit all nodes
```

## EXPORT SAFETY CHECK
Open workflow.json in editor → Ctrl+F "api_key" → Should be empty/placeholder
