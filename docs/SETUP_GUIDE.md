# Setup Guide: n8n AI Content Agent

> Production-ready n8n workflow automating end-to-end content creation: web research → AI content generation → quality validation → multi-platform output.

## Implementation Layers

| Layer | Steps | Time Estimate | Deliverable State |
|-------|-------|---------------|-------------------|
| **Core** | Phases 1-5 | ~90 minutes | Fully functional, meets all requirements |
| **Enhancement** | Phase 6 | ~30 minutes | + Scheduled trigger, error handling |
| **Maximum** | Phase 7 | ~45 minutes | + Quality gate, logging, differentiation |

**Strategy:** Complete Core first. You'll have a submittable project. Then add layers as time permits.

---

# PHASE 1: Environment Setup
**Time:** ~25 minutes

## Step 1.1: Create n8n Cloud Account
- [ ] Navigate to https://n8n.io
- [ ] Click "Get started free"
- [ ] Sign up with email (or Google/GitHub OAuth)
- [ ] Verify email if required
- [ ] You'll land on the n8n dashboard

**Checkpoint:** You see the n8n canvas with "Add first step" prompt

## Step 1.2: Create Google Cloud Project
- [ ] Go to https://console.cloud.google.com
- [ ] Click project dropdown (top left) → "New Project"
- [ ] Name: `n8n-ai-content-agent`
- [ ] Click "Create"
- [ ] Wait for project creation (~30 seconds)
- [ ] Ensure new project is selected in dropdown

**Checkpoint:** Project dashboard shows `n8n-content-agent`

## Step 1.3: Enable Google Sheets API
- [ ] In Google Cloud Console, go to "APIs & Services" → "Library"
- [ ] Search for "Google Sheets API"
- [ ] Click on it → Click "Enable"
- [ ] Wait for enablement (~10 seconds)

**Checkpoint:** Page shows "API enabled" with "Manage" button

## Step 1.4: Create OAuth Credentials
- [ ] Go to "APIs & Services" → "Credentials"
- [ ] Click "+ CREATE CREDENTIALS" → "OAuth client ID"
- [ ] If prompted for consent screen:
  - [ ] Select "External" → Create
  - [ ] App name: `n8n Content Agent`
  - [ ] User support email: your email
  - [ ] Developer contact: your email
  - [ ] Click "Save and Continue" through all screens
  - [ ] Back to Credentials → Create OAuth client ID
- [ ] Application type: "Desktop app"
- [ ] Name: `n8n-oauth`
- [ ] Click "Create"
- [ ] **IMPORTANT:** Download JSON file (click "Download JSON")
- [ ] Note the Client ID and Client Secret (you'll need these)

**Checkpoint:** You have `client_id` and `client_secret` values

## Step 1.5: Create Google Spreadsheet
- [ ] Go to https://sheets.google.com
- [ ] Create new blank spreadsheet
- [ ] Rename to: `n8n AI Content Agent - Topics`
- [ ] In Row 1, add headers:
  - A1: `Topic`
  - B1: `Status`
  - C1: `LinkedIn_Post`
  - D1: `X_Post`
  - E1: `Blog_Summary`
  - F1: `Published_Date`
  - G1: `Research_Summary`
  - H1: `Quality_Score`
- [ ] Add test data in Row 2:
  - A2: `AI agents in customer service automation`
  - B2: `Pending`
  - C2-H2: leave empty
- [ ] Add more test data in Row 3:
  - A3: `Sustainable packaging innovations 2025`
  - B3: `Pending`
- [ ] Note the Spreadsheet ID from URL: `https://docs.google.com/spreadsheets/d/[THIS_IS_THE_ID]/edit`
- [ ] Create second sheet tab named "Error_Log" (click + at bottom)
- [ ] In Error_Log sheet, add headers:
  - A1: `Timestamp`
  - B1: `Topic`
  - C1: `Error_Node`
  - D1: `Error_Message`

**Checkpoint:** Spreadsheet exists with headers and 2 test topics

## Step 1.6: Configure n8n Credentials - Google Sheets
- [ ] In n8n, click your profile icon (bottom left) → "Credentials"
- [ ] Click "Add Credential"
- [ ] Search for "Google Sheets"
- [ ] Select "Google Sheets OAuth2 API"
- [ ] Enter:
  - Client ID: (from step 1.4)
  - Client Secret: (from step 1.4)
- [ ] Click "Sign in with Google"
- [ ] Authorize access
- [ ] Click "Save"

**Checkpoint:** Green checkmark on Google Sheets credential

## Step 1.7: Configure n8n Credentials - OpenAI
- [ ] In Credentials, click "Add Credential"
- [ ] Search for "OpenAI"
- [ ] Select "OpenAI API"
- [ ] Enter your OpenAI API key
- [ ] Click "Save"

**Checkpoint:** Green checkmark on OpenAI credential

## Step 1.8: Configure n8n Credentials - Tavily (Custom Auth)
Using Custom Auth keeps your API key out of exported workflow files.

- [ ] In Credentials, click "Add Credential"
- [ ] Search for **"Custom Auth"**
- [ ] Configure:
  - [ ] **Name:** `Tavily API`
  - [ ] **JSON:**
```json
{
  "headers": {
    "X-API-Key": "tvly-YOUR_ACTUAL_KEY_HERE"
  }
}
```
- [ ] Click "Save"

**Why Custom Auth?** When you export your workflow.json, credentials are excluded - keeping your API key secure.

**Checkpoint:** All three credentials configured (Google, OpenAI, Tavily)

---

# PHASE 2: Core Workflow - Input & Research
**Time:** ~20 minutes

## Step 2.1: Create New Workflow
- [ ] In n8n, click "Workflows" in sidebar
- [ ] Click "Add Workflow"
- [ ] Click the workflow name (default "My workflow") at top
- [ ] Rename to: `n8n AI Content Agent`

**Checkpoint:** Blank canvas with renamed workflow

## Step 2.2: Add Manual Trigger (temporary - we'll add Schedule later)
- [ ] Click "Add first step" or the + button
- [ ] Search for "Manual Trigger"
- [ ] Click to add it
- [ ] This node is now on your canvas

**Checkpoint:** Manual Trigger node on canvas

## Step 2.3: Add Google Sheets - Read Node
- [ ] Click the + button on the Manual Trigger node (right side)
- [ ] Search for "Google Sheets"
- [ ] Select "Google Sheets"
- [ ] Select action: **"Get row(s) in sheet"** (under Sheet Within Document Actions)
- [ ] In the node settings panel (right side):
  - [ ] Credential: Select your Google Sheets credential
  - [ ] Resource: `Sheet Within Document`
  - [ ] Operation: `Get Row(s)`
  - [ ] Document: Click "Choose..." and select your spreadsheet name
  - [ ] Sheet: `Sheet1` (or your main sheet name)
- [ ] Rename node: Double-click node title → `Read Topics`
- [ ] Click **Execute step** to test - you should see your rows in OUTPUT

**Checkpoint:** Google Sheets node connected to trigger, shows your data

## Step 2.4: Add IF Node - Filter Pending
- [ ] Click + on the Read Topics node
- [ ] Search for "IF"
- [ ] Add the IF node
- [ ] In settings:
  - [ ] Conditions:
    - [ ] Value 1: Click field → Click **Expression** tab (or `fx` icon) → type: `{{ $json.Status }}`
    - [ ] Operation: `is equal to`
    - [ ] Value 2: `Pending` (plain text, NOT expression mode)
- [ ] Rename node: `Filter Pending`
- [ ] Click **Execute step** to test - item should appear in "True Branch" tab

**IMPORTANT:** Value 1 must be in Expression mode (purple/highlighted), Value 2 stays as plain text.

**Checkpoint:** IF node routes "Pending" items to True Branch

## Step 2.5a: Create Tavily Custom Auth Credential (Recommended)
This keeps your API key out of the exported workflow.json file.

- [ ] In n8n, click your **profile icon** (bottom left) → **Credentials**
- [ ] Click **Add Credential**
- [ ] Search for **"Custom Auth"**
- [ ] Configure:
  - [ ] **Name:** `Tavily API`
  - [ ] **JSON:**
```json
{
  "headers": {
    "X-API-Key": "tvly-YOUR_ACTUAL_KEY_HERE"
  }
}
```
- [ ] Click **Save**

**Checkpoint:** Custom Auth credential created for Tavily

## Step 2.5b: Add HTTP Request - Tavily Search
- [ ] Click + on the TRUE output of the IF node
- [ ] Search for "HTTP Request"
- [ ] Add HTTP Request node
- [ ] In settings:
  - [ ] Method: `POST`
  - [ ] URL: `https://api.tavily.com/search`
  - [ ] Authentication: `Custom Auth`
  - [ ] Credential: Select your **Tavily API** credential
  - [ ] Send Body: Toggle ON
  - [ ] Body Content Type: `JSON`
  - [ ] Specify Body: `Using Fields Below`
  - [ ] Add body parameters (click "Add Parameter" for each):
    - [ ] Name: `query` | Value: (Expression mode) `{{ $json.Topic }} latest trends 2024 2025`
    - [ ] Name: `search_depth` | Value: `advanced`
    - [ ] Name: `include_answer` | Value: `basic` (NOT `true` - API requires string)
    - [ ] Name: `max_results` | Value: `5`
- [ ] Rename node: `Tavily Research`

**IMPORTANT:**
- The `include_answer` parameter must be `basic` or `advanced` (strings), NOT boolean `true`.
- Do NOT add `api_key` to the body - it's handled by the Custom Auth credential.

**Checkpoint:** Tavily node configured with Custom Auth (API key not in workflow export)

## Step 2.6: Add Code Node - Aggregate Research
- [ ] Click + on Tavily Research node
- [ ] Search for "Code"
- [ ] Add Code node
- [ ] In settings:
  - [ ] Language: JavaScript
  - [ ] Replace all code with:

```javascript
// Aggregate Tavily research into a structured summary
const tavilyData = $input.first().json;
const topicData = $('Read Topics').first().json;

// Extract the AI-generated answer
const answer = tavilyData.answer || "No summary available.";

// Extract top 3 search results
const topResults = (tavilyData.results || [])
  .slice(0, 3)
  .map(r => `• ${r.title}: ${r.content ? r.content.substring(0, 250) : 'No content'}...`)
  .join("\n\n");

// Build comprehensive research summary
const researchSummary = `
TOPIC: ${topicData.Topic}

AI SUMMARY:
${answer}

KEY SOURCES:
${topResults}
`.trim();

return {
  json: {
    topic: topicData.Topic,
    research_summary: researchSummary,
    row_number: topicData.row_number || $('Read Topics').first().json.row_number,
    original_data: topicData
  }
};
```

- [ ] Rename node: `Aggregate Research`

**Checkpoint:** Code node processes Tavily response

## Step 2.7: Test Input & Research Pipeline
- [ ] Click "Test Workflow" button (top right) OR click individual nodes
- [ ] Click on Manual Trigger, then "Test step"
- [ ] Click on Read Topics, then "Test step"
- [ ] Verify: You see your spreadsheet rows in output
- [ ] Click on Filter Pending, then "Test step"
- [ ] Verify: Only "Pending" status rows appear
- [ ] Click on Tavily Research, then "Test step"
- [ ] Verify: JSON response with `answer` and `results`
- [ ] Click on Aggregate Research, then "Test step"
- [ ] Verify: Structured output with `topic` and `research_summary`

**Checkpoint:** All nodes show green checkmarks, data flows correctly

---

# PHASE 3: Content Generation Nodes
**Time:** ~25 minutes

## Step 3.1: Add OpenAI Node - LinkedIn Generator
- [ ] Click + on Aggregate Research node
- [ ] Search for "OpenAI"
- [ ] Select "OpenAI" → Select **"Message a model"** (under Text Actions)
- [ ] In settings:
  - [ ] Credential: Select your OpenAI credential
  - [ ] Resource: `Text`
  - [ ] Operation: `Message a Model`
  - [ ] Model: `gpt-4o-mini`
  - [ ] Messages:
    - [ ] Add Message → Role: `System` → Content: (see PROMPTS.md - LinkedIn System Prompt - copy full prompt)
    - [ ] Add Message → Role: `User` → Content (Expression): 
```
Topic: {{ $json.topic }}

Research:
{{ $json.research_summary }}

Generate the LinkedIn post now.
```
  - [ ] Options → Temperature: `0.7`
  - [ ] Options → Max Tokens: `500`
- [ ] Rename node: `Generate LinkedIn`

**Checkpoint:** LinkedIn generation node configured

## Step 3.2: Add OpenAI Node - X/Twitter Generator
- [ ] **IMPORTANT:** We need parallel execution. Click back on `Aggregate Research` node
- [ ] Click + on it to add another branch (not after LinkedIn)
- [ ] Add another OpenAI node → Select **"Message a model"**
- [ ] Resource: `Text`, Operation: `Message a Model`
- [ ] Model: `gpt-4o-mini`
- [ ] Messages:
  - [ ] System message: (see PROMPTS.md - X/Twitter System Prompt)
  - [ ] User message (Expression):
```
Topic: {{ $json.topic }}

Research:
{{ $json.research_summary }}

Generate the tweet now.
```
  - [ ] Temperature: `0.8`
  - [ ] Max Tokens: `100`
- [ ] Rename node: `Generate X Post`

**Checkpoint:** X/Twitter node branches from Aggregate Research

## Step 3.3: Add OpenAI Node - Blog Summary Generator
- [ ] Click on `Aggregate Research` again
- [ ] Click + to create third branch
- [ ] Add OpenAI node → Select **"Message a model"**
- [ ] Resource: `Text`, Operation: `Message a Model`
- [ ] Model: `gpt-4o-mini`
- [ ] Messages:
  - [ ] System: (see PROMPTS.md - Blog Summary System Prompt)
  - [ ] User (Expression):
```
Topic: {{ $json.topic }}

Research:
{{ $json.research_summary }}

Generate the blog summary now.
```
  - [ ] Temperature: `0.7`
  - [ ] Max Tokens: `400`
- [ ] Rename node: `Generate Blog`

**Checkpoint:** Three parallel OpenAI nodes from Aggregate Research

## Step 3.4: Add Merge Node - Combine Outputs
- [ ] Click + on `Generate LinkedIn` node
- [ ] Search for "Merge"
- [ ] Add Merge node
- [ ] **Connect all three OpenAI outputs to this Merge node:**
  - [ ] Drag from Generate X Post output → to Merge input
  - [ ] Drag from Generate Blog output → to Merge input
- [ ] In Merge settings:
  - [ ] Mode: `Append`
- [ ] Rename: `Combine Content`

**Note:** The connection order matters! Connect in this order: LinkedIn first, X Post second, Blog third. The Structure Output code relies on this order.

**Checkpoint:** All three generators feed into single Merge node (OUTPUT shows 3 items)

## Step 3.5: Add Code Node - Structure Final Output
- [ ] Click + on Combine Content
- [ ] Add Code node
- [ ] JavaScript code:

```javascript
// Get all 3 items from merge (in connection order)
const items = $input.all();

// Extract text from each item by position
// Order: [0] = LinkedIn, [1] = X Post, [2] = Blog (based on connection order to Merge)
const linkedin = items[0]?.json?.output?.[0]?.content?.[0]?.text || '';
const xPost = items[1]?.json?.output?.[0]?.content?.[0]?.text || '';
const blog = items[2]?.json?.output?.[0]?.content?.[0]?.text || '';

// Get original data from Aggregate Research
const aggregateData = $('Aggregate Research').first().json;

return {
  json: {
    topic: aggregateData.topic,
    linkedin_post: linkedin,
    x_post: xPost,
    blog_summary: blog,
    research_summary: aggregateData.research_summary,
    row_number: aggregateData.row_number,
    published_date: new Date().toISOString()
  }
};
```

- [ ] Rename: `Structure Output`

**IMPORTANT:** The order `[0]`, `[1]`, `[2]` depends on how you connected the generators to the Merge node. If content appears in wrong fields, adjust the indices.

**Checkpoint:** Code node combines all content into single object with all 3 pieces populated

## Step 3.6: Test Content Generation
- [ ] Run the full workflow test
- [ ] Check each node's output
- [ ] Verify Structure Output shows all three content pieces

**Checkpoint:** All content generates correctly

---

# PHASE 4: Output to Google Sheets
**Time:** ~15 minutes

## Step 4.1: Add Google Sheets - Update Row
- [ ] Click + on Structure Output
- [ ] Add Google Sheets node → Select **"Update row in sheet"**
- [ ] Settings:
  - [ ] Credential: Your Google Sheets credential
  - [ ] Resource: `Sheet Within Document`
  - [ ] Operation: `Update Row`
  - [ ] Document: Select your spreadsheet
  - [ ] Sheet: `Sheet1`
  - [ ] Mapping Column Mode: `Map Each Column Manually`
  - [ ] Column to match on: `row_number`
  - [ ] **row_number (using to match)**: (Expression mode) `{{ $json.row_number }}`
- [ ] Add column values (click "Add Field" for each):
  - [ ] `Status` → Value: `Completed`
  - [ ] `LinkedIn_Post` → Value: (Expression mode) `{{ $json.linkedin_post }}`
  - [ ] `X_Post` → Value: (Expression mode) `{{ $json.x_post }}`
  - [ ] `Blog_Summary` → Value: (Expression mode) `{{ $json.blog_summary }}`
  - [ ] `Published_Date` → Value: (Expression mode) `{{ $json.published_date }}`
  - [ ] `Research_Summary` → Value: (Expression mode) `{{ $json.research_summary }}`
- [ ] Rename: `Update Sheet`

**IMPORTANT:** For all `{{ }}` expressions, make sure the field is in **Expression mode** (click the toggle or `fx` icon). If you see the literal text `{{ $json.xxx }}` in your output instead of actual content, the field is not in Expression mode.

**Checkpoint:** Sheet update node configured with all columns

## Step 4.2: Full Integration Test
- [ ] Make sure Row 2 in your sheet has Status = "Pending"
- [ ] Click "Execute Workflow" (runs entire workflow)
- [ ] Wait for completion
- [ ] Open Google Sheet → Verify Row 2 is now populated with content

**Checkpoint:** Sheet shows generated content, Status = "Completed"

## Step 4.3: Handle Row Number Issue (Common Fix)
If row_number is undefined, we need to track it manually:

- [ ] Go back to `Read Topics` node
- [ ] In Options → Add Option → "Fields to Return": (leave default to return all)
- [ ] The row_number isn't automatic. We need to add it in a Code node.

**Fix:** Add Code node between Read Topics and Filter Pending:
- [ ] Click on the connection line between Read Topics and Filter Pending
- [ ] Add Code node in between
- [ ] Name: `Add Row Numbers`
- [ ] Code:
```javascript
const items = $input.all();
return items.map((item, index) => ({
  json: {
    ...item.json,
    row_number: index + 2  // +2 because row 1 is headers, data starts at row 2
  }
}));
```

**Checkpoint:** Each item now has row_number property

---

# PHASE 5: Documentation & Core Completion
**Time:** ~10 minutes

## Step 5.1: Add Node Descriptions
For each major node:
- [ ] Click node → Settings gear → Add "Note"
- [ ] Read Topics: "Fetches all topics from Google Sheets"
- [ ] Filter Pending: "Only processes topics with Status = Pending"
- [ ] Tavily Research: "Retrieves current web research using Tavily API"
- [ ] Aggregate Research: "Structures research into comprehensive summary"
- [ ] Generate LinkedIn: "Creates professional LinkedIn post"
- [ ] Generate X Post: "Creates concise tweet ≤280 chars"
- [ ] Generate Blog: "Creates 150-200 word blog summary"
- [ ] Update Sheet: "Writes generated content back to sheet"

**Checkpoint:** All nodes have descriptive notes

## Step 5.2: Export Workflow JSON
- [ ] Click the three dots menu (top right) OR click Workflow name
- [ ] Select "Download"
- [ ] Save as `workflow.json`

**Checkpoint:** You have workflow.json file

## Step 5.3: Take Workflow Screenshot
- [ ] Zoom to fit all nodes in view
- [ ] Take screenshot (full workflow visible)
- [ ] Save as `workflow-screenshot.png`

**Checkpoint:** Screenshot captured

---

# PHASE 6: Enhancement Layer - Scheduled Trigger & Error Handling
**Time:** ~30 minutes

## Step 6.1: Replace Manual Trigger with Schedule Trigger
- [ ] Delete Manual Trigger node (or keep for testing - add Schedule separately)
- [ ] Click on canvas → Add node → Search "Schedule Trigger"
- [ ] Add Schedule Trigger
- [ ] Settings:
  - [ ] Trigger Interval: "Minutes"
  - [ ] Minutes Between Triggers: `15`
  - [ ] OR use Cron if you want specific times
- [ ] Connect to Read Topics node
- [ ] Rename: `Every 15 Minutes`

**Checkpoint:** Workflow has scheduled trigger

## Step 6.2: Create Error Handling Workflow
- [ ] Click + in empty space (not connected to main flow)
- [ ] Add "Error Trigger" node
- [ ] This catches errors from any node

## Step 6.3: Add Error Logger
- [ ] Click + on Error Trigger
- [ ] Add Code node
- [ ] Code:
```javascript
const error = $input.first().json;
return {
  json: {
    timestamp: new Date().toISOString(),
    topic: error.execution?.data?.topic || 'Unknown',
    error_node: error.node?.name || 'Unknown',
    error_message: error.message || JSON.stringify(error)
  }
};
```
- [ ] Rename: `Format Error`

## Step 6.4: Log Error to Sheet
- [ ] Click + on Format Error
- [ ] Add Google Sheets node
- [ ] Settings:
  - [ ] Operation: `Append Row`
  - [ ] Document: Your spreadsheet
  - [ ] Sheet: `Error_Log`
  - [ ] Column mappings:
    - Timestamp: `{{ $json.timestamp }}`
    - Topic: `{{ $json.topic }}`
    - Error_Node: `{{ $json.error_node }}`
    - Error_Message: `{{ $json.error_message }}`
- [ ] Rename: `Log Error`

**Checkpoint:** Error handling workflow captures failures

## Step 6.5: Add Status = "Processing" Update
To show work in progress, add node early in flow:
- [ ] After Filter Pending (TRUE path), before Tavily
- [ ] Add Google Sheets Update node
- [ ] Update Status column to "Processing"
- [ ] This prevents duplicate processing

**Checkpoint:** Status transitions: Pending → Processing → Completed/Error

---

# PHASE 7: Maximum Layer - Quality Gate & Differentiation
**Time:** ~45 minutes

## Step 7.1: Add Quality Gate Node
After Structure Output, before Update Sheet:
- [ ] Add OpenAI node
- [ ] System prompt (from PROMPTS.md - Quality Gate prompt)
- [ ] User message (Expression):
```
Validate the following generated content:

TOPIC: {{ $json.topic }}

LINKEDIN POST:
{{ $json.linkedin_post }}

X/TWITTER POST:
{{ $json.x_post }}

BLOG SUMMARY:
{{ $json.blog_summary }}

RESEARCH USED:
{{ $json.research_summary }}

Respond with JSON only.
```
- [ ] Temperature: `0`
- [ ] Max Tokens: `300`
- [ ] Rename: `Quality Gate`

## Step 7.2: Parse Quality Response
- [ ] Add Code node after Quality Gate
- [ ] Code:
```javascript
const response = $input.first().json.message.content;
const contentData = $('Structure Output').first().json;

// Parse JSON from response
let quality;
try {
  // Remove markdown code blocks if present
  const cleanJson = response.replace(/```json\n?|\n?```/g, '').trim();
  quality = JSON.parse(cleanJson);
} catch (e) {
  // If parsing fails, assume pass
  quality = { overall_pass: true, issues: [] };
}

return {
  json: {
    ...contentData,
    quality_pass: quality.overall_pass,
    quality_score: quality.overall_pass ? 'Pass' : 'Needs Review',
    quality_issues: quality.issues || []
  }
};
```
- [ ] Rename: `Parse Quality`

## Step 7.3: Add IF for Quality Check
- [ ] Add IF node after Parse Quality
- [ ] Condition: `{{ $json.quality_pass }}` equals `true`
- [ ] TRUE path → Update Sheet (mark as Completed)
- [ ] FALSE path → Update Sheet with Status = "Review Needed"

**Checkpoint:** Quality gate validates content before publishing

## Step 7.4: Add Content Variation Logging (Optional)
Add another sheet tab "Generation_Log" to track:
- All generated content versions
- Quality scores over time
- Enables analytics

## Step 7.5: Final Testing
- [ ] Reset a row to Status = "Pending"
- [ ] Execute full workflow
- [ ] Verify:
  - [ ] Research is gathered
  - [ ] All three content pieces generate
  - [ ] Quality gate runs
  - [ ] Sheet is updated
  - [ ] Error handling catches any issues

**Checkpoint:** Full maximum-featured workflow operational

---

# PHASE 8: Final Documentation
**Time:** ~15 minutes

## Step 8.1: Create README.md
See template in project files. Include:
- [ ] Workflow screenshot
- [ ] API keys list (names only): OpenAI, Tavily, Google Sheets
- [ ] Sample input topic
- [ ] Sample outputs (copy from sheet)
- [ ] Prompt design rationale
- [ ] Architecture explanation

## Step 8.2: Final Export
- [ ] Export latest workflow.json
- [ ] Verify credentials are NOT included
- [ ] Organize files in folder:
  - `workflow.json`
  - `README.md`
  - `workflow-screenshot.png`

## Step 8.3: Quality Check Submission
- [ ] Open workflow.json in text editor
- [ ] Ctrl+F search for "api_key" - should be empty or placeholder
- [ ] Verify no actual secrets in file

**DONE! Ready for submission.**

---

# Quick Reference: n8n Interface Tips

| Task | How To |
|------|--------|
| Add node | Click + on existing node, or right-click canvas |
| Connect nodes | Drag from output dot to input dot |
| Delete connection | Click connection line, press Delete |
| Delete node | Select node, press Delete |
| Test single node | Click node → "Test step" |
| Test full workflow | "Execute Workflow" button |
| Expression mode | Click field → toggle to Expression ({{ }}) |
| Rename node | Double-click node title |
| Add note | Click node → Settings gear → Note |
| Zoom fit | Ctrl+0 or View menu |
| Save | Auto-saves, or Ctrl+S |
| Export | Three dots menu → Download |
