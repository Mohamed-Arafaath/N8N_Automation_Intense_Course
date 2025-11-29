# 📘 PHASE 3: Advanced AI Architecture - Complete Cheat Sheet

> **AI Agent Implementation Guide** - RAG, Multi-Agent Systems, Voice AI, and Enterprise Deployment.

---

## 🧠 1. RAG (Retrieval-Augmented Generation)

### RAG System Architecture

**Two Pipelines:**
1. **Ingestion** = Learn (store knowledge)
2. **Retrieval** = Answer (query knowledge)

---

### PIPELINE A: Ingestion (Data → Vector DB)

```
1. DATA SOURCES
   - Google Drive (new file trigger)
   - Notion (page updated webhook)
   - Website (scheduled scrape)
   - Manual upload (webhook)

2. CONTENT EXTRACTION
   PDF → Text
   HTML → Markdown
   DOCX → Plain text
   Images → OCR

3. CHUNKING
   Node: Recursive Character Text Splitter
   Settings:
   - Chunk Size: 800-1000 tokens
   - Chunk Overlap: 150-200 tokens (CRITICAL!)
   
   Why overlap?
   - Preserves context across boundaries
   - Prevents information splitting
   
   Example:
   Chunk 1: "...refund policy is simple. We offer..."
   Chunk 2: "We offer a 30-day money-back..."
   ↑ "We offer" appears in both = context bridge

4. GENERATE EMBEDDINGS
   Node: OpenAI Embeddings
   Model: text-embedding-3-small (1536 dimensions)
   
   Input: "Our refund policy is 30 days"
   Output: [0.023, -0.891, 0.445, ..., 0.234]
   (Array of 1536 numbers)

5. METADATA ENRICHMENT
   Add context to chunks:
   ```json
   {
     "text": "Our refund policy...",
     "metadata": {
       "source_url": "https://company.com/policies",
       "source_type": "webpage",
       "author": "Legal Team",
       "date_updated": "2025-11-28",
       "category": "Customer Service",
       "access_level": "public",
       "language": "en"
     }
   }
   ```

6. UPSERT TO VECTOR DB
   Node: Pinecone Vector Store / Qdrant
   Operation: Insert
   Collection: company_knowledge_base
```

---

### PIPELINE B: Retrieval (Question → Answer)

```
1. USER QUESTION
   Input: "What's your refund policy?"

2. QUERY EMBEDDING
   Node: OpenAI Embeddings
   Convert question → vector
   Must use SAME model as ingestion!

3. VECTOR SEARCH
   Node: Pinecone/Qdrant Retrieve
   Settings:
   - Top K: 10-20 results
   - Score Threshold: 0.7 (cosine similarity)
   
   Query Vector: [0.019, -0.887, ...]
   Finds: Most similar chunks in DB

4. RERANKING (Optional - Advanced)
   Node: Cohere Rerank
   Input: Top 20 results from vector search
   Output: Top 3-5 most relevant
   
   Why rerank?
   - Vector search = fast but approximate
   - Reranking = slow but precise
   - Accuracy boost: 70% → 90%+

5. CONTEXT ASSEMBLY
   Format retrieved chunks:
   ```
   Context:
   
   [Source 1: company.com/policies, Updated: 2025-11-28]
   Our refund policy is simple: 30-day money-back guarantee.
   
   [Source 2: company.com/faq, Updated: 2025-10-15]
   To request a refund, email support@company.com.
   ```

6. LLM SYNTHESIS
   Node: OpenAI Chat / Claude
   System Prompt:
   """
   You are a helpful customer service assistant.
   
   RULES:
   1. Answer using ONLY the context below
   2. Cite sources using [Source X] format
   3. If answer not in context, say: "I don't have that information"
   4. Never invent information
   
   Context:
   {retrieved_chunks}
   
   Question: {user_question}
   
   Answer:
   """

7. RESPONSE
   Return to user with citations:
   ```
   Our refund policy is 30 days [Source 1]. 
   To request a refund, email support@company.com [Source 2].
   ```
```

---

### Pinecone Setup

```bash
# 1. Create account at pinecone.io
# 2. Get API key from dashboard

# 3. Create index via UI:
Index Name: company-kb
Dimensions: 1536
Metric: cosine
Pod Type: s1.x1 (starter)
```

**n8n Pinecone Credentials:**
```
API Key: YOUR_PINECONE_API_KEY
Environment: YOUR_ENVIRONMENT (e.g., us-west1-gcp)
Index: company-kb
```

---

### Qdrant Setup (Self-Hosted)

```yaml
# docker-compose.yml
version: '3.8'

services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_storage:/qdrant/storage
    environment:
      QDRANT__SERVICE__GRPC_PORT: 6334

volumes:
  qdrant_storage:
```

```bash
# Start Qdrant
docker-compose up -d

# Create collection
curl -X PUT 'http://localhost:6333/collections/company_kb' \
  -H 'Content-Type: application/json' \
  -d '{
    "vectors": {
      "size": 1536,
      "distance": "Cosine"
    }
  }'
```

**n8n Qdrant Credentials:**
```
URL: http://localhost:6333
API Key: (leave empty for local)
Collection: company_kb
```

---

### Advanced RAG Patterns

#### 1. Hybrid Search (Vector + Keyword)

```json
{
  "query": {
    "vector": [0.023, -0.891, ...],
    "filter": {
      "must": [
        {
          "key": "product_code",
          "match": { "value": "SKU-12345" }
        }
      ]
    }
  }
}
```
Use when searching for:
- Product codes
- Exact names
- Dates

#### 2. Metadata Filtering

```javascript
// Code Node - Build filtered query
const userDepartment = $input.first().json.user_department;

return [{
  json: {
    filter: {
      must: [
        { key: "access_level", match: { any: ["public", "internal"] } },
        { key: "department", match: { any: [userDepartment, "All"] } }
      ]
    }
  }
}];
```

#### 3. Citation Guard (Prevent Hallucination)

**System Prompt:**
```
Answer using ONLY the context below.

For EVERY claim, cite the source using [Source X].

If the answer is not in the context, respond EXACTLY:
"I don't have that information in my knowledge base."

Do NOT use external knowledge.
Do NOT make assumptions.
```

---

## 🤖 2. Multi-Agent Systems

### CrewAI Architecture

**Core Concepts:**

1. **Agent** = Specialized AI worker
2. **Task** = Specific assignment  
3. **Crew** = Team of agents working together

---

### Market Research Crew Example

```python
# Agent 1: Researcher
researcher = Agent(
    role="Market Researcher",
    goal="Find comprehensive competitor data",
    backstory="Expert researcher with 10 years experience in market analysis",
    tools=[search_tool, scrape_tool, crunchbase_tool],
    verbose=True
)

# Agent 2: Analyst
analyst = Agent(
    role="Business Analyst",
    goal="Identify patterns and insights from research data",
    backstory="Data analyst specializing in competitive intelligence",
    tools=[analysis_tool],
    verbose=True
)

# Agent 3: Writer
writer = Agent(
    role="Business Writer",
    goal="Create professional, actionable reports",
    backstory="Business journalist with MBA background",
    tools=[document_tool],
    verbose=True
)

# Define Tasks
research_task = Task(
    description="""
    Research these competitors: {competitors}
    
    For each, gather:
    - Pricing
    - Features
    - Customer reviews
    - Market position
    - Recent news
    """,
    expected_output="Comprehensive competitor data in structured format",
    agent=researcher
)

analysis_task = Task(
    description="""
    Analyze the research data and identify:
    - Feature comparison
    - Pricing strategies
    - Market gaps
    - SWOT analysis
    """,
    expected_output="Strategic analysis with recommendations",
    agent=analyst
)

writing_task = Task(
    description="""
    Create executive report with:
    - Executive summary
    - Key findings
    - Recommendations
    - Charts and tables
    """,
    expected_output="Professional PDF report",
    agent=writer
)

# Create Crew
crew = Crew(
    agents=[researcher, analyst, writer],
    tasks=[research_task, analysis_task, writing_task],
    process=Process.sequential,
    verbose=True
)

# Execute
result = crew.kickoff(inputs={"competitors": ["Salesforce", "HubSpot"]})
```

---

### n8n Multi-Agent Workflow

```
1. WEBHOOK TRIGGER
   Input: {
     "industry": "SaaS CRM",
     "competitors": ["Salesforce", "HubSpot","Pipedrive"]
   }

2. RESEARCH AGENT (Parallel Execution)
   For each competitor:
   - Google Search (SerpApi)
   - Website Scrape (Firecrawl)
   - Crunchbase API (funding data)
   - G2/Capterra (reviews)

3. CONSOLIDATE DATA
   Node: Code Node
   Combine all research into structured format

4. ANALYSIS AGENT
   Node: Claude 3.5 Sonnet
   Prompt:
   """
   You are a business analyst.
   
   Analyze this competitor data:
   {research_data}
   
   Provide:
   1. Feature comparison matrix
   2. Pricing analysis
   3. Market positioning
   4. Competitive advantages/disadvantages
   5. Market gaps/opportunities
   """

5. WRITING AGENT
   Node: GPT-4
   Prompt:
   """
   You are a business writer.
   
   Create executive report using:
   {analysis}
   
   Format:
   - Executive Summary (1 page)
   - Detailed Findings (3-5 pages)
   - Recommendations (1 page)
   - Appendix (data tables)
   
   Style: Professional, actionable
   """

6. EDITOR AGENT
   Node: Claude 3.5 Sonnet
   Prompt:
   """
   You are a quality editor.
   
   Review this report:
   {draft_report}
   
   Check:
   - Accuracy (cite sources)
   - Grammar and style
   - Logical flow
   - Actionable recommendations
   
   Return: Edited version
   """

7. GENERATE PDF
   Node: HTTP Request → DocRaptor API

8. DELIVER
   - Email to stakeholder
   - Save to Google Drive
   - Log in Notion
```

---

## 🎙️ 3. Voice AI Agents

### Vapi Integration (AI Phone Receptionist)

```
USE CASE: Dental office receptionist

CAPABILITIES:
- Answer calls 24/7
- Book appointments
- Answer FAQs
- Transfer to human if needed

SETUP:

1. CREATE VAPI ASSISTANT
   Dashboard: vapi.ai
   Configuration:
   {
     "name": "Dental Receptionist",
     "voice": "jennifer",
     "model": {
       "provider": "openai",
       "model": "gpt-4",
       "temperature": 0.7
     },
     "firstMessage": "Thank you for calling ABC Dental. How can I help you today?",
     "systemPrompt": """
     You are a friendly dental receptionist.
     
     Your capabilities:
     1. Book appointments
     2. Answer common questions
     3. Take messages
     
     RULES:
     - Be warm and professional
     - Confirm all details
     - If medical question → transfer to dentist
     - Always end with: "Is there anything else I can help with?"
     """
   }

2. CONNECT TO n8n
   Vapi → Webhook → n8n
   
   When call ends:
   - Send transcript
   - Send call recording
     - Send metadata (duration, caller ID)

3. n8n WORKFLOW
   ```
   1. Webhook (from Vapi)
   
   2. IF booking_made:
      → Add to Google Calendar
      → Send confirmation email
      → Add to CRM
      → Send SMS reminder (24hr before)
   
   3. IF message_taken:
      → Send to Slack
      → Email to dentist
      → Log in CRM
   
   4. LOG CALL
      → Save transcript to Google Drive
      → Update call metrics dashboard
   ```
```

---

### ElevenLabs Voice Cloning

```
1. RECORD VOICE SAMPLES
   - 10-30 minutes of clean audio
   - Your voice or client's voice
   - No background noise

2. UPLOAD TO ELEVENLABS
   Dashboard → Voice Lab → Add Voice
   - Upload samples
   - Name voice: "CEO_Voice"
   - Wait for processing (5-10 min)

3. USE IN n8n
   Node: HTTP Request → ElevenLabs API
   ```javascript
   POST https://api.elevenlabs.io/v1/text-to-speech/{voice_id}
   
   Headers:
   xi-api-key: YOUR_ELEVENLABS_API_KEY
   
   Body:
   {
     "text": "Welcome to our company. I'm excited to share...",
     "model_id": "eleven_multilingual_v2",
     "voice_settings": {
       "stability": 0.5,
       "similarity_boost": 0.75,
       "style": 0.3
     }
   }
   ```

4. USE CASES
   - Personalized sales videos
   - Internal announcements
   - Course content
   - Podcast intros
```

---

## 🔌 4. Model Context Protocol (MCP)

**What is MCP?**
Universal standard for connecting AI models to data sources.

**Think of it as:** USB for AI
- USB = Universal connector for devices
- MCP = Universal connector for data → AI

---

### MCP Server Example

```javascript
// MCP Server (connects Google Drive to Claude)

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  {
    name: "google-drive-mcp",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Define tool: Search Drive
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "search_drive",
      description: "Search Google Drive files",
      inputSchema: {
        type: "object",
        properties: {
          query: {
            type: "string",
            description: "Search query",
          },
        },
      },
    },
  ],
}));

// Handle tool execution
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "search_drive") {
    // Call Google Drive API
    const results = await searchGoogleDrive(request.params.arguments.query);
    return { content: [{ type: "text", text: JSON.stringify(results) }] };
  }
});
```

---

## 🏢 5. Enterprise AI Deployment

### Security Best Practices

```
1. DATA ISOLATION
   - Separate vector DBs per client
   - Tenant-specific indexes
   - Access control via metadata

2. ENCRYPTION
   - TLS/SSL for all API calls
   - Encrypt vectors at rest
   - Secure API keys in vault

3. COMPLIANCE
   GDPR:
   - Right to be forgotten (delete vectors)
   - Data portability (export data)
   - Audit logs
   
   HIPAA (Healthcare):
   - PHI encryption
   - Access logs
   - BAA with providers

4. RATE LIMITING
   - Prevent abuse
   - Cost control
   - Fair usage policies
```

---

### Monitoring & Observability

```javascript
// Log every RAG query
{
  timestamp: "2025-11-29T20:30:00Z",
  user_id: "user_123",
  query: "What's your refund policy?",
  retrieval_time_ms: 45,
  chunks_retrieved: 5,
  llm_time_ms: 1200,
  total_time_ms: 1245,
  cost_usd: 0.003,
  answer_length: 156,
  user_rated: "helpful"
}
```

**Key Metrics:**
- Query latency
- Retrieval accuracy
- LLM costs
- User satisfaction
- Error rates

---

## 📊 Ultimate Prompting Cheat Sheet

### zero-Shot Prompting
```
Task: Classify this email as spam or not spam.

Email: {email_content}

Classification:
```

### Few-Shot Prompting
```
Classify emails:

Email: "Congratulations! You've won $1M!"
Classification: Spam

Email: "Meeting at 2pm tomorrow"
Classification: Not Spam

Email: {new_email}
Classification:
```

### Chain-of-Thought
```
Question: {complex_question}

Let's think step by step:
1. First, identify...
2. Then, analyze...
3. Finally, conclude...
```

### ReAct (Reasoning + Acting)
```
Thought: I need to find competitor pricing
Action: search_web("competitor X pricing 2025")
Observation: Found pricing page
Thought: Now I need to compare with our pricing
Action: retrieve_internal_docs("our pricing strategy")
Observation: Retrieved our pricing
Thought: I can now make recommendation
Final Answer: ...
```

---

**💡 Pro Tips:**
- Always use temperature 0.3-0.5 for RAG (less creative = more accurate)
- Chunk size affects accuracy: Test 500, 800, 1000, 1200 tokens
- Reranking is expensive but worth it for critical applications
- Monitor LLM costs daily (can explode quickly)
- Cache frequent queries to reduce API calls
