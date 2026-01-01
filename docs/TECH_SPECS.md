# Technical Specifications: n8n AI Content Agent

> Production-ready n8n workflow automating end-to-end content creation: web research → AI content generation → quality validation → multi-platform output. Features parallel processing, error handling, and quality gates.

## Project Metadata
| Field | Value |
|-------|-------|
| Platform | n8n (cloud instance) |
| LLM Backend | OpenAI gpt-4o-mini |
| Research API | Tavily Search |
| Data Store | Google Sheets |
| Complexity | Maximum (with differentiation features) |

---

## 1. Architecture Overview

### 1.1 High-Level Flow
```
┌──────────────────────────────────────────────────────────────────────────────┐
│                           CONTENT CREATOR AGENT                               │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐  │
│  │  SCHEDULE   │───►│   GOOGLE    │───►│   FILTER    │───►│   TAVILY    │  │
│  │  TRIGGER    │    │   SHEETS    │    │  (Pending)  │    │   SEARCH    │  │
│  │  (15 min)   │    │   (Read)    │    │             │    │             │  │
│  └─────────────┘    └─────────────┘    └─────────────┘    └──────┬──────┘  │
│                                                                   │         │
│                                         ┌────────────────────────┘         │
│                                         ▼                                   │
│                              ┌─────────────────────┐                       │
│                              │   RESEARCH          │                       │
│                              │   AGGREGATOR        │                       │
│                              │   (Summarize)       │                       │
│                              └──────────┬──────────┘                       │
│                                         │                                   │
│            ┌────────────────────────────┼────────────────────────────┐     │
│            ▼                            ▼                            ▼     │
│   ┌─────────────────┐        ┌─────────────────┐        ┌─────────────────┐│
│   │   OPENAI        │        │   OPENAI        │        │   OPENAI        ││
│   │   LinkedIn      │        │   X/Twitter     │        │   Blog          ││
│   │   Generator     │        │   Generator     │        │   Generator     ││
│   └────────┬────────┘        └────────┬────────┘        └────────┬────────┘│
│            │                          │                          │         │
│            └──────────────────────────┼──────────────────────────┘         │
│                                       ▼                                     │
│                          ┌─────────────────────────┐                       │
│                          │   QUALITY GATE          │  ◄── ENHANCEMENT      │
│                          │   (Validation Node)     │                       │
│                          └────────────┬────────────┘                       │
│                                       │                                     │
│                    ┌──────────────────┴──────────────────┐                 │
│                    ▼                                      ▼                 │
│         ┌─────────────────┐                   ┌─────────────────┐          │
│         │   GOOGLE        │                   │   REGENERATE    │          │
│         │   SHEETS        │                   │   (If Failed)   │          │
│         │   (Write)       │                   └─────────────────┘          │
│         └─────────────────┘                                                │
│                                                                             │
└──────────────────────────────────────────────────────────────────────────────┘
                                       │
                                       ▼
                          ┌─────────────────────────┐
                          │   ERROR WORKFLOW        │  ◄── ENHANCEMENT
                          │   (Error Trigger)       │
                          │   Logs failures to      │
                          │   separate sheet        │
                          └─────────────────────────┘
```

### 1.2 Node Inventory

| Node # | Node Type | Purpose | Layer |
|--------|-----------|---------|-------|
| 1 | Schedule Trigger | Runs every 15 minutes | Enhancement |
| 2 | Google Sheets (Read) | Fetch all rows | Core |
| 3 | IF | Filter Status = "Pending" | Core |
| 4 | Tavily Search | Research topic | Core |
| 5 | Code (JavaScript) | Aggregate/summarize research | Core |
| 6 | OpenAI Chat | Generate LinkedIn post | Core |
| 7 | OpenAI Chat | Generate X/Twitter post | Core |
| 8 | OpenAI Chat | Generate Blog summary | Core |
| 9 | OpenAI Chat | Quality validation gate | Maximum |
| 10 | IF | Check quality pass/fail | Maximum |
| 11 | Google Sheets (Update) | Write outputs + status | Core |
| 12 | Error Trigger | Catch workflow errors | Enhancement |
| 13 | Google Sheets (Append) | Log errors to Error_Log sheet | Enhancement |

---

## 2. External Services Configuration

### 2.1 Google Cloud Project Setup

**Project Requirements:**
- Google Cloud Project with billing enabled (free tier sufficient)
- Google Sheets API enabled
- OAuth 2.0 credentials (Desktop app type)

**Required Scopes:**
```
https://www.googleapis.com/auth/spreadsheets
https://www.googleapis.com/auth/drive.file
```

**Spreadsheet Schema:**

| Column | Data Type | Description |
|--------|-----------|-------------|
| A: Topic | String | Content topic/keyword |
| B: Status | Enum | "Pending" / "Processing" / "Completed" / "Error" |
| C: LinkedIn_Post | String | Generated LinkedIn content |
| D: X_Post | String | Generated tweet (≤280 chars) |
| E: Blog_Summary | String | Generated blog summary |
| F: Published_Date | DateTime | Timestamp of completion |
| G: Research_Summary | String | Tavily research output (for debugging) |
| H: Quality_Score | String | Pass/Fail from quality gate |

**Error Log Sheet Schema (separate sheet in same spreadsheet):**

| Column | Data Type | Description |
|--------|-----------|-------------|
| A: Timestamp | DateTime | When error occurred |
| B: Topic | String | Topic being processed |
| C: Error_Node | String | Which node failed |
| D: Error_Message | String | Error details |

### 2.2 Tavily API Configuration

**Endpoint:** `https://api.tavily.com/search`

**Request Parameters:**
```json
{
  "api_key": "{{TAVILY_API_KEY}}",
  "query": "{{topic}} latest trends 2024 2025",
  "search_depth": "advanced",
  "include_answer": "basic",
  "max_results": 5
}
```

**Why these settings:**
- `search_depth: advanced` → More comprehensive results (worth the extra latency)
- `include_answer: "basic"` → Tavily provides an AI-summarized answer (must be string "basic" or "advanced", NOT boolean)
- `max_results: 5` → Balance between comprehensiveness and token cost

### 2.3 OpenAI Configuration

**Model:** `gpt-4o-mini`
**Temperature:** 0.7 (balanced creativity/consistency)
**Max Tokens per call:**
- LinkedIn: 500
- X/Twitter: 100
- Blog: 400
- Quality Gate: 200

---

## 3. Data Flow Specification

### 3.1 Stage 1: Input Retrieval

**Input:** Google Sheet row where Status = "Pending"
**Output:** Topic string + Row number (for later update)

```javascript
// Expected data structure after Google Sheets node
{
  "Topic": "AI agents in healthcare",
  "Status": "Pending",
  "LinkedIn_Post": "",
  "X_Post": "",
  "Blog_Summary": "",
  "Published_Date": "",
  "row_number": 2
}
```

### 3.2 Stage 2: Research

**Input:** Topic string
**Output:** Structured research summary

```javascript
// Expected Tavily response structure
{
  "answer": "AI agents are transforming healthcare by...",
  "results": [
    {
      "title": "How AI Agents Are Revolutionizing...",
      "url": "https://...",
      "content": "...",
      "score": 0.95
    },
    // ... more results
  ]
}
```

**Aggregation Logic (Code Node):**
```javascript
// Combine Tavily answer with top 3 result snippets
const tavilyData = $input.first().json;
const answer = tavilyData.answer || "";
const topResults = (tavilyData.results || [])
  .slice(0, 3)
  .map(r => `- ${r.title}: ${r.content.substring(0, 200)}`)
  .join("\n");

return {
  research_summary: `${answer}\n\nKey Sources:\n${topResults}`,
  topic: $('Google Sheets').first().json.Topic
};
```

### 3.3 Stage 3: Content Generation

**Input:** Research summary + Topic
**Output:** Three content pieces (LinkedIn, X, Blog)

Each OpenAI node receives:
```javascript
{
  "topic": "AI agents in healthcare",
  "research_summary": "AI agents are transforming healthcare by..."
}
```

### 3.4 Stage 4: Quality Validation (Maximum Layer)

**Input:** All three generated content pieces
**Output:** Quality assessment with pass/fail

```javascript
// Quality gate prompt asks LLM to verify:
// 1. Factual grounding (references research)
// 2. Platform appropriateness (tone, length)
// 3. No hallucinations
// 4. Coherence

// Expected output structure
{
  "linkedin_valid": true,
  "x_valid": true,
  "blog_valid": true,
  "overall_pass": true,
  "issues": []
}
```

### 3.5 Stage 5: Output Writing

**Input:** Generated content + quality results
**Output:** Updated Google Sheet row

**Update payload:**
```javascript
{
  "LinkedIn_Post": "{{linkedin_content}}",
  "X_Post": "{{x_content}}",
  "Blog_Summary": "{{blog_content}}",
  "Status": "Completed",
  "Published_Date": "{{current_timestamp}}",
  "Research_Summary": "{{research_summary}}",
  "Quality_Score": "Pass"
}
```

---

## 4. Error Handling Specification

### 4.1 Error Categories

| Error Type | Trigger | Response |
|------------|---------|----------|
| Empty Research | Tavily returns no results | Use fallback prompt, mark as "Partial" |
| API Rate Limit | 429 from OpenAI/Tavily | Wait 60s, retry once |
| Invalid Content | Quality gate fails | Log issue, mark as "Review Needed" |
| Network Error | Connection timeout | Log to Error sheet, mark as "Error" |
| Sheet Access | Google API error | Halt workflow, send notification |

### 4.2 Error Trigger Workflow

```
Error Trigger (catches any error)
       │
       ▼
┌─────────────────┐
│ Extract Error   │
│ - Node name     │
│ - Error message │
│ - Input data    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Google Sheets   │
│ (Append Row to  │
│  Error_Log)     │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Update Original │
│ Row Status to   │
│ "Error"         │
└─────────────────┘
```

---

## 5. Prompt Engineering Specifications

See **PROMPTS.md** for complete prompts. Key design principles:

### 5.1 LinkedIn Prompt Design
- **Role:** Senior content strategist at a thought-leadership consultancy
- **Voice:** Professional, authoritative, but accessible
- **Structure:** Hook → Insight → Evidence → Engagement question
- **Constraints:** 150-300 words, 3-5 paragraphs, 3-5 relevant hashtags

### 5.2 X/Twitter Prompt Design
- **Role:** Viral social media specialist
- **Voice:** Punchy, curious, slightly provocative
- **Structure:** Hook → Core insight → CTA or question
- **Constraints:** ≤280 characters (hard limit), 1-2 hashtags, optional emoji

### 5.3 Blog Summary Prompt Design
- **Role:** SEO content writer
- **Voice:** Informative, scannable, value-forward
- **Structure:** Context → Key points → Takeaway
- **Constraints:** 150-200 words, no fluff, include keywords naturally

### 5.4 Quality Gate Prompt Design
- **Role:** Content QA analyst
- **Task:** Validate all three outputs against criteria
- **Output:** Structured JSON with pass/fail per piece

---

## 6. Performance Expectations

| Metric | Target | Rationale |
|--------|--------|-----------|
| End-to-end latency | < 45 seconds | Tavily (~5s) + 4 OpenAI calls (~8s each) + Sheets (~2s) |
| Token consumption per run | ~2,500 tokens | Research summary + 3 generations + quality check |
| Cost per topic | ~$0.005 | gpt-4o-mini pricing |
| Success rate | > 95% | With error handling |

---

## 7. Differentiation Features (Maximum Layer)

### 7.1 Quality Gate Node
- Validates content before writing to sheet
- Catches hallucinations, off-topic content, wrong format
- Enables automatic regeneration or human review flag

### 7.2 Research Summary Persistence
- Stores Tavily research in sheet for debugging/audit
- Enables manual review of source quality

### 7.3 Scheduled Automation
- True "set and forget" operation
- Processes new topics automatically every 15 minutes

### 7.4 Comprehensive Error Logging
- Separate Error_Log sheet with full context
- Enables debugging without losing data

### 7.5 Status State Machine
```
Pending → Processing → Completed
    │         │
    │         └──► Error (recoverable)
    │         └──► Review Needed (quality fail)
    └──────────► Skipped (no research found)
```

---

## 8. Security Considerations

| Secret | Storage Location | Notes |
|--------|-----------------|-------|
| OpenAI API Key | n8n Credentials | Never in workflow JSON |
| Tavily API Key | n8n Credentials | Never in workflow JSON |
| Google OAuth | n8n Credentials | Refresh token stored |

**Export Safety:** When exporting workflow JSON, credentials are excluded by default. Verify before submission.

---

## 9. Submission Checklist

- [ ] workflow.json exported from n8n
- [ ] README.md with:
  - [ ] Workflow screenshot
  - [ ] API keys list (Tavily, OpenAI, Google Sheets)
  - [ ] Sample input topic
  - [ ] Sample LinkedIn output
  - [ ] Sample X output
  - [ ] Sample Blog output
  - [ ] Prompt design rationale
- [ ] All files in single folder
- [ ] Credentials NOT included in export
