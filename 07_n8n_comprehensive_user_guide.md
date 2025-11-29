# 📖 The Complete n8n Encyclopedia (A-Z Guide)

> **Everything You Need to Know About n8n** - From installation to advanced AI agents, production deployment, and beyond.

---

## 📑 Table of Contents

### Part I: Fundamentals
1. [Installation & Setup](#1-installation--setup)
2. [Interface & Navigation](#2-interface--navigation)
3. [Data Structures](#3-data-structures)
4. [Workflow Building](#4-workflow-building)

### Part II: Nodes & Features
5. [Node Types & Categories](#5-node-types--categories)
6. [Core Nodes Reference](#6-core-nodes-reference)
7. [AI & Vector Database Nodes](#7-ai--vector-database-nodes)
8. [Trigger Nodes](#8-trigger-nodes)

### Part III: Advanced Concepts
9. [Expressions & Functions](#9-expressions--functions)
10. [Credentials Management](#10-credentials-management)
11. [Workflow Settings](#11-workflow-settings)
12. [Execution Modes](#12-execution-modes)

### Part IV: Production
13. [Environment Variables](#13-environment-variables)
14. [Error Handling](#14-error-handling)
15. [Monitoring & Logging](#15-monitoring--logging)
16. [Performance & Scaling](#16-performance--scaling)

### Part V: Expert Topics
17. [CLI & API](#17-cli--api)
18. [Custom Nodes](#18-custom-nodes)
19. [Version Control](#19-version-control)
20. [Troubleshooting](#20-troubleshooting)

---

# PART I: FUNDAMENTALS

## 1. Installation & Setup

### Cloud (n8n.cloud)
**Best for:** Quick start, no maintenance
```
1. Go to n8n.cloud
2. Sign up
3. Start building immediately
```
**Pricing:** Starts at €20/month

### Desktop (npm)
**Best for:** Local development
```bash
# Install Node.js 18+ first
npm install n8n -g

# Start n8n
n8n start

# Access at http://localhost:5678
```

### Docker (Self-Hosted)
**Best for:** Production, full control
```bash
# Quick start
docker run -it --rm \
  --name n8n \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n

# Production with PostgreSQL (see Phase 1 cheat sheet)
```

### Docker Compose (Recommended Production)
See [n8n_engineering_cheat_sheet.md](file:///Users/arafathjazeeb/.gemini/antigravity/brain/3610de29-3aa6-4744-a8be-84520263a5cd/n8n_engineering_cheat_sheet.md) for full setup.

### Kubernetes
```bash
# Using Helm
helm repo add n8n https://n8n-io.github.io/n8n-helm-chart/
helm install n8n n8n/n8n
```

---

## 2. Interface & Navigation

### The Canvas (Main Workspace)
- **Infinite Grid:** Pan by clicking and dragging empty space
- **Zoom:** Mouse wheel or `+`/`-` buttons
- **Node Selection:** Click individual nodes or drag-select multiple
- **Connection Lines:** Show data flow between nodes
- **Mini-Map:** Bottom right shows overview of large workflows

### Top Bar
| Element | Purpose |
|---------|---------|
| **Workflow Name** | Click to rename (auto-saves) |
| **Active Toggle** | Turn workflow on/off |
| **Save** | Manual save (auto-saves exist) |
| **Execute Workflow** | Test run entire workflow |
| **Settings** | Workflow-specific config |

### Left Sidebar

#### Workflows Tab
- List of all workflows
- Star favorites
- Search/filter
- Duplicate workflows

#### Templates Tab
- Pre-built workflows from community
- Filter by app or use case
- One-click import

#### Credentials Tab
- Secure storage for API keys
- OAuth tokens
- Database connections
- **Never hardcode credentials in nodes!**

#### Executions Tab
- History of past runs
- Filter by status (Success/Error/Waiting)
- Click to view execution details
- **Debug mode:** Shows exact data at each step

#### Settings Tab
- General: Timezone, owner
- Community Nodes: Install 3rd party nodes
- API: Generate API keys
- Audit Logs: Track changes

### Node Panel (Add Nodes)
Click `+` or press `Tab` to open

**Categories:**
- **Triggers:** Start the workflow
- **Actions:** Do something (send email, create row, etc.)
- **Flow Logic:** IF, Switch, Merge, etc.
- **AI:** LLM nodes, Vector stores, Agents
- **Utility:** Code, HTTP Request, etc.

---

## 3. Data Structures

### The n8n Data Format
Every node works with this structure:
```json
[
  {
    "json": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    },
    "binary": {},
    "pairedItem": {
      "item": 0
    }
  }
]
```

**Key Components:**
- **Array:** Always an array (even for 1 item)
- **json:** Your actual data
- **binary:** Files (PDFs, images)
- **pairedItem:** Tracks data lineage

### Input vs Output
- **Input:** Data coming INTO a node (from left)
- **Output:** Data leaving a node (to right)
- A node can have multiple outputs (e.g., IF has True/False)

### Implicit Looping
**Critical Concept:**
If a node receives 100 items, it executes 100 times automatically.

Example:
```
Google Sheets (100 rows)
↓
HTTP Request (executes 100 times)
↓
100 API calls made
```

**To prevent this:**
- Use "Split in Batches" node
- Or Code Node with `$input.all()` to process as one batch

### Binary Data
Files are stored separately:
```json
{
  "json": {
    "fileName": "report.pdf"
  },
  "binary": {
    "data": {
      "data": "base64_encoded_file_content",
      "mimeType": "application/pdf",
      "fileName": "report.pdf",
      "fileSize": 51200
    }
  }
}
```

**Access in expressions:** `{{ $binary.data }}`

---

## 4. Workflow Building

### Anatomy of a Workflow
```
[Trigger] → [Logic/Transform] → [Action] → [Output]
```

Example:
```
Schedule Trigger (Every Monday 9am)
↓
HTTP Request (Get data from API)
↓
IF (Price > $100)
↓ TRUE
Send Email
↓ FALSE
(Do nothing)
```

### Building Steps

#### 1. Add a Trigger
Every workflow needs exactly ONE trigger.
- Click `+`
- Search `Schedule` or `Webhook`
- Configure timing/URL

#### 2. Add Processing Nodes
- Transform data (Set node)
- Filter data (IF node)
- Call APIs (HTTP Request)

#### 3. Add Actions
- Send notifications (Slack, Email)
- Update databases (Google Sheets, PostgreSQL)
- Create records (CRM systems)

#### 4. Test
- Click `Execute Workflow`
- Check each node (green = success, red = error)
- View data by clicking on node

#### 5. Activate
- Toggle `Active` switch to ON
- Workflow runs automatically in background

### Node Connections
- **Solid line:** Normal data flow
- **Dotted line:** Error trigger flow
- **Multiple outputs:** IF, Switch have multiple paths

---

# PART II: NODES & FEATURES

## 5. Node Types & Categories

### By Function

#### Trigger Nodes (Start Point)
- **Schedule:** Cron/interval timing
- **Webhook:** HTTP endpoint
- **Email:** On new email received
- **Chat:** Messages from Slack, Discord, Telegram
- **File:** Monitor folder for changes
- **Database:** On new row/record

#### Action Nodes (Do Something)
- **Communication:** Email, Slack, SMS
- **Data Storage:** Sheets, Airtable, databases
- **CRM:** HubSpot, Salesforce, Pipedrive
- **Development:** GitHub, GitLab, Jira

#### Logic Nodes
- **IF:** Conditional branching
- **Switch:** Multi-path routing
- **Merge:** Combine data streams
- **Split:** Divide arrays

#### Transform Nodes
- **Set:** Modify fields
- **Code:** JavaScript processing
- **Aggregate:** Group/sum data

#### AI Nodes (2024+)
- **OpenAI:** GPT models
- **Claude:** Anthropic models
- **Local AI:** Ollama integration
- **Vector Store:** Pinecone, Qdrant
- **AI Agent:** Autonomous decision-making

### By Data Type
- **Text:** String manipulation
- **Numbers:** Math operations
- **Dates:** DateTime formatting
- **Files:** Binary data handling
- **JSON:** Object transformation

---

## 6. Core Nodes Reference

### Set Node (Edit Fields)
**Purpose:** Create, rename, remove, or calculate fields

**Operations:**
1. **Add fields:** `newField = "value"`
2. **Remove fields:** Select fields to delete
3. **Keep only:** Whitelist specific fields
4. **Rename:** `oldName` → `newName`

**Example:**
```
Input: { firstName: "John", lastName: "Doe" }
↓
Set: fullName = {{$json.firstName}} {{$json.lastName}}
↓
Output: { firstName: "John", lastName: "Doe", fullName: "John Doe" }
```

---

### IF Node
**Purpose:** Split workflow into two paths

**Conditions:**
- String: equals, contains, regex
- Number: >, <, >=, <=, =
- Boolean: true/false
- Date: before, after
- Exists: field is/isn't present

**Example:**
```
IF {{ $json.price }} > 100
  TRUE → Send VIP Email
  FALSE → Send Standard Email
```

**Multiple Conditions:**
- Use `AND` / `OR` logic
- Combine any number of rules

---

### Switch Node
**Purpose:** Route to multiple paths (like switch/case)

**Modes:**
1. **Rules:** IF-like conditions
2. **Expression:** Based on value

**Example:**
```
SWITCH {{ $json.priority }}
  CASE "urgent" → Slack Alert
  CASE "high" → Email Manager
  CASE "medium" → Create Ticket
  DEFAULT → Log Only
```

---

### Merge Node
**Purpose:** Combine data from multiple branches

**3 Modes:**

#### 1. Append (Stack)
```
Branch A: [item1, item2]
Branch B: [item3, item4]
Result: [item1, item2, item3, item4]
```

#### 2. Merge By Index (Zip)
```
Branch A: [{name: "John"}, {name: "Jane"}]
Branch B: [{age: 30}, {age: 25}]
Result: [{name: "John", age: 30}, {name: "Jane", age: 25}]
```

#### 3. Merge By Key (SQL JOIN)
```
Branch A: [{email: "john@ex.com", name: "John"}]
Branch B: [{email: "john@ex.com", score: 95}]
Result: [{email: "john@ex.com", name: "John", score: 95}]
```

**Join Types:**
- Inner: Only matching items
- Left: All from left, matching from right
- Outer: All from both

---

### Code Node
**Purpose:** Run custom JavaScript

**Access Data:**
```javascript
// Get all items
const items = $input.all();

// Get first item
const first = $input.first();

// Get specific field
const email = $input.item.json.email;
```

**Return Data:**
```javascript
// Return array
return items.map(item => ({
  json: { processed: true }
}));

// Return single item
return { json: { result: "success" } };
```

**Available Libraries:**
- `crypto`: Encryption/hashing
- `luxon`: Date manipulation (built-in)
- All Node.js built-ins

---

### HTTP Request Node
**Purpose:** Connect to ANY API

**Methods:**
- GET: Retrieve data
- POST: Create data
- PUT: Update (full replace)
- PATCH: Update (partial)
- DELETE: Remove data

**Authentication:**
- None
- Basic Auth
- Header Auth (API Key)
- OAuth1/OAuth2
- Digest Auth
- Custom

**Advanced:**
- **Pagination:** Auto-follow next pages
- **Batching:** Send multiple requests
- **Retry:** On error, retry with backoff

**Example:**
```
POST https://api.example.com/users
Headers:
  Authorization: Bearer {{$credentials.apiKey}}
Body:
  {
    "name": "{{$json.name}}",
    "email": "{{$json.email}}"
  }
```

---

### Split In Batches
**Purpose:** Process large datasets in chunks

**Use Cases:**
- Avoid rate limits (100 requests/min)
- Reduce memory usage (10,000 rows)
- Control API costs

**Example:**
```
Google Sheets (1000 rows)
↓
Split In Batches (batch size: 50)
↓
HTTP Request (runs 20 times)
↓
Wait Node (5 seconds)
↓
Loop back until complete
```

---

### Wait Node
**Purpose:** Pause execution

**Options:**
- **After Previous Node:** Wait X seconds/minutes/hours
- **Until Date:** Wait until specific time
- **On Webhook Call:** Resume when webhook hit

**Use Cases:**
- Rate limiting between API calls
- Human approval workflows
- Delayed notifications

---

### Aggregate Node
**Purpose:** Group and summarize data

**Operations:**
- Count
- Sum
- Average
- Min/Max
- First/Last
- Array of values

**Example:**
```
Input: [
  {product: "A", sales: 100},
  {product: "A", sales: 200},
  {product: "B", sales: 150}
]
↓
Aggregate: Group by "product", Sum "sales"
↓
Output: [
  {product: "A", total: 300},
  {product: "B", total: 150}
]
```

---

## 7. AI & Vector Database Nodes

### OpenAI Node
**Models Available:**
- GPT-4o: Latest, best quality
- GPT-4-Turbo: Fast, cheaper
- GPT-3.5-Turbo: Cheapest

**Operations:**
- **Chat:** Conversational
- **Image:** DALL-E generation
- **Embeddings:** Vector conversion
- **Audio:** Whisper transcription

---

### AI Agent Node
**Purpose:** Autonomous decision-making

**Components:**
1. **System Prompt:** Define role
2. **Tools:** Functions AI can call
3. **Memory:** Chat history

**Example:**
```
Agent: "Customer Support Bot"
Tools: [Search_Knowledge_Base, Create_Ticket, Send_Email]

User: "My order hasn't arrived"
↓
Agent decides: Call Search_Knowledge_Base("order status")
↓
Agent responds: "Your order #12345 is in transit..."
```

---

### Vector Store Nodes

#### Pinecone
**Use for:** Cloud vector database
```
Operations:
- Insert: Store embeddings
- Retrieve: Similarity search
- Update: Modify vectors
- Delete: Remove data
```

#### Qdrant
**Use for:** Self-hosted vector database
```
Same operations as Pinecone
Supports:
- Hybrid search (vector + keyword)
- Metadata filtering
- Collections
```

#### Supabase Vector (pgvector)
**Use for:** PostgreSQL-based vectors

---

### Document Loaders
**Purpose:** Convert files to text for RAG

**Supported:**
- PDF
- Word (DOCX)
- CSV
- JSON
- Markdown
- HTML
- Plain text

---

### Text Splitters
**Recursive Character Splitter:**
- Chunk size: 800-1000 tokens
- Overlap: 150-200 tokens

**Why overlap?** Prevents context loss at chunk boundaries.

---

### Chat Models
**Purpose:** Conversational AI

**Providers:**
- OpenAI (GPT)
- Anthropic (Claude)
- Google (Gemini)
- Ollama (Local models)
- Azure OpenAI

---

## 8. Trigger Nodes

### Schedule Trigger
**Options:**
- **Interval:** Every X minutes/hours/days
- **Cron:** Advanced scheduling
- **Days/Times:** Specific times only

**Cron Examples:**
```
0 9 * * 1-5   # Weekdays at 9am
0 */2 * * *   # Every 2 hours
0 0 1 * *     # 1st of month at midnight
```

---

### Webhook Trigger
**Creates:** Unique URL n8n listens to

**URL:** `https://your-n8n.com/webhook/unique-id`

**Methods:**
- GET: Simple triggers
- POST: Receive data
- PUT/DELETE: Custom logic

**Authentication:**
- None (public)
- Header Auth
- Basic Auth
- HMAC signature

**Production vs Test:**
- Test URL: Changes on edit
- Production URL: Permanent

---

### Email Trigger (IMAP)
**Monitors:** Email inbox
**Triggers:** On new email, attachment, specific sender

**Use Cases:**
- Parse invoices from emails
- Support ticket creation
- Order processing

---

### Chat Trigger
**Creates:** Chat interface

**Modes:**
- **Normal:** Simple Q&A
- **Agent:** With tools and memory
- **Memory:** Remembers conversation

**Integrations:**
- Webchat widget
- Slack
- Telegram
- WhatsApp (via Twilio)

---

### Form Trigger
**Creates:** Web form

**Fields:**
- Text
- Number
- Email
- Dropdown
- File upload

**Returns:** Form submission data

---

### Error Trigger
**Purpose:** Catch errors from OTHER workflows

**Use Case:**
- Central error monitoring
- Send alerts when ANY workflow fails
- Log to external system

---

# PART III: ADVANCED CONCEPTS

## 9. Expressions & Functions

### Expression Editor
**Access:** Click gear icon on any field

**Drag & Drop:**
1. See input data on left
2. Drag field into expression
3. Result: `{{ $json.fieldName }}`

### Variables

#### $json
Current item's data
```javascript
{{ $json.email }}
{{ $json.customer.name }}
{{ $json.items[0].price }}
```

#### $binary
File data
```javascript
{{ $binary.data.fileName }}
```

#### $node
Reference other nodes
```javascript
{{ $node["Get Customers"].json.count }}
{{ $node["HTTP Request"].first().json.id }}
```

#### $input
All input data
```javascript
{{ $input.all().length }}  // Count items
{{ $input.first().json.id }}
```

#### $now
Current timestamp
```javascript
{{ $now }}  // ISO format
{{ $now.toFormat('yyyy-MM-dd') }}  // Custom format
```

#### $today
Today at midnight
```javascript
{{ $today }}
```

#### $workflow
Workflow metadata
```javascript
{{ $workflow.id }}
{{ $workflow.name }}
{{ $workflow.active }}
```

#### $execution
Execution details
```javascript
{{ $execution.id }}
{{ $execution.mode }}  // "trigger", "manual", "webhook"
{{ $execution.resumeUrl }}
```

#### $itemIndex
Current position in array
```javascript
{{ $itemIndex }}  // 0, 1, 2, 3...
```

#### $runIndex
Execution attempt number
```javascript
{{ $runIndex }}  // 0 on first run, 1 on retry, etc.
```

### Functions

#### String Functions
```javascript
{{ $json.email.toLowerCase() }}
{{ $json.name.toUpperCase() }}
{{ $json.text.trim() }}
{{ $json.url.split('/')[3] }}  // Get part
{{ $json.text.replace('old', 'new') }}
{{ $json.data.includes('search') }}  // true/false
{{ $json.text.length }}
```

#### Number Functions
```javascript
{{ $json.price.toFixed(2) }}  // 2 decimals
{{ Math.round($json.value) }}
{{ Math.floor($json.value) }}
{{ Math.ceil($json.value) }}
{{ Math.max($json.a, $json.b) }}
{{ Math.min($json.a, $json.b) }}
{{ Math.random() }}
```

#### Date Functions (Luxon)
```javascript
{{ $now.toFormat('yyyy-MM-dd') }}
{{ $now.plus({ days: 7 }).toISO() }}
{{ $now.minus({ hours: 2 }).toISO() }}
{{ DateTime.fromISO($json.date).diff(DateTime.now(), 'days').days }}
```

#### Array Functions
```javascript
{{ $json.items.length }}
{{ $json.items[0] }}  // First item
{{ $json.items.join(', ') }}
{{ $json.items.map(x => x.name) }}
{{ $json.items.filter(x => x.price > 100) }}
```

#### Conditional
```javascript
{{ $json.score > 80 ? 'Hot' : 'Cold' }}
{{ $json.email ? $json.email : 'no-email@example.com' }}
```

---

## 10. Credentials Management

### Creating Credentials
1. Go to **Credentials** tab
2. Click **+ Add Credential**
3. Select type (OAuth2, API Key, etc.)
4. Fill in details
5. **Test** connection
6. **Save**

### Credential Types

#### API Key
```
Name: My API Key
Type: Header Auth
Header Name: Authorization
Header Value: Bearer YOUR_KEY
```

#### OAuth2
```
Grant Type: Authorization Code
Auth URL: https://provider.com/oauth/authorize
Token URL: https://provider.com/oauth/token
Client ID: your_client_id
Client Secret: your_client_secret
Scope: read write
```

#### Basic Auth
```
Username: admin
Password: secure_password
```

#### Database
```
Host: localhost
Port: 5432
Database: mydb
User: postgres
Password: dbpassword
```

### Using Credentials
In nodes:
1. Click **Credential to connect with**
2. Select existing or create new
3. Node auto-fills auth headers

### Accessing in Expressions
```javascript
{{ $credentials.apiKey }}
{{ $credentials.username }}
```

**⚠️ Security:** Never log/expose credentials!

---

## 11. Workflow Settings

### General Settings
**Access:** Click `...` (three dots) → Settings

#### Basic
- **Name:** Workflow name
- **Tags:** Organize workflows
- **Timezone:** Execution timezone

#### Execution
- **Timeout:** Max runtime (seconds)
- **Error Workflow:** Route errors
- **Timezone:** Override default

#### Execution Data
- **Save manually:** Save every run
- **Save on error:** Only save failures
- **Save on success:** Only save successes
- **Save nothing:** Don't save

**💡 Tip:** "Save on error" is best for production (saves space)

---

### Workflow Variables
**Static Variables:**
```javascript
{{ $workflow.settings.variableName }}
```

**Use Case:** Store constants (API URLs, company name, etc.)

---

## 12. Execution Modes

### Regular Mode (Default)
- Executes on main thread
- Good for: Low-volume workflows
- Limit: ~10 executions/second

### Queue Mode (Production)
- Uses Redis queue
- Scales horizontally
- Good for: High-volume workflows
- Limit: Thousands/second

**Enable Queue Mode:**
```bash
# Environment variables
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=redis
QUEUE_BULL_REDIS_PORT=6379
```

### Worker Mode
Dedicated workers process queue jobs

**Start worker:**
```bash
docker run -it --rm \
  -e EXECUTIONS_MODE=queue \
  -e QUEUE_BULL_REDIS_HOST=redis \
  n8nio/n8n worker
```

**Scale:** Run multiple workers for load balancing

---

# PART IV: PRODUCTION

## 13. Environment Variables

### Essential Variables

#### Database
```bash
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=postgres
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=secure_password
```

#### Security
```bash
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=admin_password

N8N_JWT_AUTH_ACTIVE=true
N8N_JWT_AUTH_HEADER=Authorization
```

#### Domain & SSL
```bash
N8N_HOST=n8n.yourdomain.com
N8N_PROTOCOL=https
WEBHOOK_URL=https://n8n.yourdomain.com/
```

#### Timezone
```bash
GENERIC_TIMEZONE=America/New_York
TZ=America/New_York
```

#### Execution
```bash
EXECUTIONS_MODE=queue
EXECUTIONS_TIMEOUT=300
EXECUTIONS_TIMEOUT_MAX=3600
```

#### Queue (if using queue mode)
```bash
QUEUE_BULL_REDIS_HOST=redis
QUEUE_BULL_REDIS_PORT=6379
QUEUE_BULL_REDIS_DB=0
QUEUE_HEALTH_CHECK_ACTIVE=true
```

---

## 14. Error Handling

### Node-Level Error Handling
**Continue On Fail:**
- Enable on node
- Workflow continues even if node fails
- Error data passed to next node

### Try-Catch Pattern
```
[Node with Continue On Fail: ON]
↓
[IF Node: Check for error]
  TRUE → Error Handler
  FALSE → Normal Flow
```

### Error Workflow
**Global error handler:**
1. Create workflow with Error Trigger
2. Receives error from ANY workflow
3. Send alerts, log, retry, etc.

**Error data:**
```json
{
  "execution": { "id": "...", "name": "..." },
  "workflow": { "id": "...", "name": "..." },
  "node": { "name": "...", "type": "..." },
  "error": { "message": "..." }
}
```

---

## 15. Monitoring & Logging

### Execution History
**View:** Executions tab

**Filter by:**
- Status (Success/Error/Waiting)
- Workflow
- Date range

**Details:**
- Execution time
- Data at each node
- Error messages

### External Logging
**Send to:**
- Elasticsearch
- DataDog
- New Relic
- Custom webhook

**Example workflow:**
```
Error Trigger
↓
Format Error
↓
HTTP Request → DataDog API
```

---

## 16. Performance & Scaling

### Optimization Tips

#### 1. Use Queue Mode
Handles high volume better

#### 2. Limit Executions Data
```bash
EXECUTIONS_DATA_SAVE_ON_ERROR=all
EXECUTIONS_DATA_SAVE_ON_SUCCESS=none
EXECUTIONS_DATA_PRUNE=true
```

#### 3. Use Pagination
Don't load 10,000 rows at once - use batching

#### 4. Optimize Code Nodes
```javascript
// ❌ Slow
for (let item of items) {
  await apiCall(item);  // Sequential
}

// ✅ Fast
await Promise.all(
  items.map(item => apiCall(item))  // Parallel
);
```

#### 5. Redis Cache
Cache API responses to reduce calls

### Horizontal Scaling
```
Main n8n instance (handles webhooks, UI)
↓
Redis Queue
↓
Worker 1, Worker 2, Worker 3... (process jobs)
```

---

# PART V: EXPERT TOPICS

## 17. CLI & API

### n8n CLI Commands

```bash
# Start n8n
n8n start

# Import workflow
n8n import:workflow --input=workflow.json

# Export workflow
n8n export:workflow --output=backup.json --id=5

# Export credentials
n8n export:credentials --output=creds.json

# Execute workflow (manual)
n8n execute --id=5
```

### n8n API
**Enable:**
```bash
N8N_API_KEY_AUTH_ENABLED=true
```

**Endpoints:**
```
GET /workflows
GET /workflows/:id
POST /workflows
PUT /workflows/:id
DELETE /workflows/:id

POST /workflows/:id/activate
POST /workflows/:id/deactivate

GET /executions
GET /executions/:id
```

**Example:**
```bash
curl -X GET https://n8n.example.com/api/v1/workflows \
  -H "X-N8N-API-KEY: your_api_key"
```

---

## 18. Custom Nodes

### Community Nodes
**Install:**
```bash
# Via UI: Settings → Community Nodes → Install
# Via CLI:
npm install n8n-nodes-package-name
```

**Popular Community Nodes:**
- n8n-nodes-browserless
- n8n-nodes-firecrawl
- n8n-nodes-langchain-advanced

### Creating Custom Nodes
**Structure:**
```
my-node/
  credentials/
    MyApi.credentials.ts
  nodes/
    MyNode/
      MyNode.node.ts
      MyNode.node.json
package.json
```

**Basic node:**
```typescript
export class MyNode implements INodeType {
  description: INodeTypeDescription = {
    displayName: 'My Node',
    name: 'myNode',
    group: ['transform'],
    version: 1,
    inputs: ['main'],
    outputs: ['main'],
    properties: [
      {
        displayName: 'Operation',
        name: 'operation',
        type: 'options',
        options: [
          { name: 'Get', value: 'get' }
        ],
        default: 'get'
      }
    ]
  };

  async execute(this: IExecuteFunctions) {
    const items = this.getInputData();
    // Process items
    return [items];
  }
}
```

---

## 19. Version Control

### Export Workflows
**Manual:**
1. Workflow → `...` → Download
2. Saves as JSON

**Programmatic:**
```bash
n8n export:workflow --backup --output=./backups/
```

### Git Integration
**Recommended structure:**
```
/n8n-workflows/
  /workflows/
    workflow1.json
    workflow2.json
  /credentials/ (⚠️ Don't commit!)
    .gitignore
  README.md
```

**.gitignore:**
```
credentials/
.env
*.log
```

### CI/CD
```yaml
# GitHub Actions example
name: Deploy n8n
on: push

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Import workflows
        run: |
          for file in workflows/*.json; do
            n8n import:workflow --input=$file
          done
```

---

## 20. Troubleshooting

### Common Issues

#### "Workflow didn't finish"
**Cause:** Timeout
**Fix:**
```bash
EXECUTIONS_TIMEOUT=600  # 10 minutes
EXECUTIONS_TIMEOUT_MAX=3600  # 1 hour max
```

#### "Item 0 doesn't exist"
**Cause:** Previous node returned 0 items
**Fix:** Check IF node or filter - might be filtering out all data

#### "Cannot read property 'x' of undefined"
**Cause:** Field doesn't exist
**Fix:** Use optional chaining
```javascript
{{ $json.customer?.address?.city || 'Unknown' }}
```

#### "Rate limit exceeded"
**Cause:** Too many API calls
**Fix:**
- Add Wait node (1-2 seconds between calls)
- Use Split In Batches
- Implement exponential backoff

#### "Workflow stuck on Webhook"
**Cause:** Waiting for webhook response
**Fix:** Check webhook configuration, ensure sender is hitting correct URL

#### Memory Issues
**Cause:** Processing too much data at once
**Fix:**
- Use pagination (load 100 rows at a time)
- Enable streaming for large files
- Increase Docker memory limit

---

## 📚 Quick Reference

### Essential Keyboard Shortcuts
| Shortcut | Action |
|----------|--------|
| `Tab` | Add node |
| `Ctrl/Cmd + S` | Save |
| `Ctrl/Cmd + C/V` | Copy/Paste nodes |
| `Ctrl/Cmd + Z` | Undo |
| `Delete` | Delete selected |
| `Ctrl/Cmd + Click` | Multi-select |
| `Space + Drag` | Pan canvas |

### Data Access Patterns
```javascript
// Current item
{{ $json.field }}

// All items
{{ $input.all().length }}

// Previous node
{{ $node["Node Name"].json.field }}

// First/last
{{ $input.first().json.field }}
{{ $input.last().json.field }}

// Array access
{{ $json.items[0] }}
{{ $json.items.map(x => x.name) }}
```

### Best Practices Checklist
- ✅ Name all nodes descriptively
- ✅ Use credentials (never hardcode keys)
- ✅ Add notes for complex logic
- ✅ Test with small data first
- ✅ Handle errors (Continue On Fail + IF check)
- ✅ Use Queue Mode for production
- ✅ Monitor executions
- ✅ Version control (export workflows)
- ✅ Set execution timeouts
- ✅ Use environment variables

---

**🎓 You now have the complete A-Z knowledge of n8n as of 2025!**
