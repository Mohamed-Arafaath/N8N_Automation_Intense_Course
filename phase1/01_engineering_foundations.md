# Phase 1: Engineering Foundations

> **Goal**: Master the technical substrate that powers all AI automation. Before building autonomous agents, you must master the deterministic rails upon which they run.

---

## 📋 Table of Contents

1. [Installation & Infrastructure Setup](#11-installation--infrastructure-setup)
2. [Core Concepts & Data Flow](#12-core-concepts--data-flow)
3. [Webhooks, APIs & HTTP Requests](#13-webhooks-apis--http-requests)
4. [JavaScript & Code Node Mastery](#14-javascript--code-node-mastery)
5. [JSON & Data Transformation](#15-json--data-transformation)
6. [Phase 1 Checkpoint](#phase-1-checkpoint)

---

## 1.1 Installation & Infrastructure Setup

### 📚 Udemy Course Reference

**Section 3: Self-Hosting & Maintenance**

- Lecture: Understanding Three Ways to Run n8n
- Lecture: How to Self-Host n8n for Unlimited Workflows (Hostinger VPS)
- Lecture: How to Update Your Self-Hosted n8n Instance (Docker)
- Lecture: How to Self-Host n8n Locally (Windows/Mac with Node.js)

### 🎥 Essential YouTube Tutorials

#### Beginner Setup

1. **[n8n Tutorial for Beginners | Ultimate Quick Start Guide (2025)](https://www.youtube.com/watch?v=example)**
   - Complete beginner walkthrough
   - Desktop installation
   - First workflow creation
   - Duration: ~30 minutes

2. **[How To Automate Workflows (Full Tutorial) | n8n + Hostinger](https://www.youtube.com/watch?v=example)**
   - Production VPS setup
   - Domain configuration
   - SSL certificate setup
   - Duration: ~45 minutes

3. **[Install N8N Easy Node.js and NPM Setup Guide](https://www.youtube.com/watch?v=example)**
   - Local development environment
   - Node.js installation
   - NPM configuration
   - Duration: ~20 minutes

4. **[Let's Make n8n EASY (Beginner Tutorial)](https://www.youtube.com/watch?v=example)**
   - Simplified setup process
   - Common troubleshooting
   - Best practices
   - Duration: ~25 minutes

### 🛠️ Hands-On Project: "Build Your Fortress"

**Objective**: Set up a production-ready n8n instance with proper security and persistence.

#### Task Checklist:

1. **Set up n8n using Docker Compose on a VPS**
   - Choose provider: Hostinger, DigitalOcean, or Hetzner
   - Minimum specs: 2GB RAM, 1 CPU core
   - Ubuntu 22.04 LTS recommended

2. **Configure SSL certificate with custom domain**
   - Register domain (e.g., `n8n.youragency.com`)
   - Point DNS A record to VPS IP
   - Use Traefik or Caddy for automatic SSL

3. **Set up PostgreSQL for production data persistence**
   - SQLite is file-based and locks during writes
   - PostgreSQL handles high-concurrency operations
   - Essential for AI agents with async loops

4. **Implement automated backup workflow**
   - Daily database backups
   - Workflow export automation
   - Cloud storage integration (S3/Google Drive)

#### Docker Compose Configuration Example:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: your_secure_password
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data

  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: n8n
      DB_POSTGRESDB_USER: n8n
      DB_POSTGRESDB_PASSWORD: your_secure_password
      N8N_BASIC_AUTH_ACTIVE: true
      N8N_BASIC_AUTH_USER: admin
      N8N_BASIC_AUTH_PASSWORD: your_admin_password
      N8N_HOST: n8n.yourdomain.com
      N8N_PROTOCOL: https
      WEBHOOK_URL: https://n8n.yourdomain.com/
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      - postgres

volumes:
  postgres_data:
  n8n_data:
```

### 💡 What You'll Master:

- ✅ Docker containerization fundamentals
- ✅ Production server management
- ✅ SSL/HTTPS configuration for secure data transmission
- ✅ Database setup for high-concurrency AI workflows
- ✅ Understanding the difference between cloud vs. self-hosted

### 📖 Additional Resources:

- [How to self-host n8n: Setup, architecture, and pricing guide](https://northflank.com/guides/how-to-self-host-n8n)
- [n8n Self-Hosting Documentation](https://docs.n8n.io/hosting/)
- [n8n Docker Compose Examples](https://github.com/n8n-io/n8n/tree/master/docker/compose)

### 🎯 Pro Tips:

> **Hosting Provider Choice**: For AI agents requiring real-time voice interaction, host your n8n instance in the same region as your LLM provider's API endpoint (e.g., US-East for OpenAI) to reduce latency by 50-100ms.

> **Cost Optimization**: Self-hosted n8n is significantly cheaper for high-volume AI agent workflows. Cloud n8n charges per execution; self-hosted has unlimited executions.

---

## 1.2 Core Concepts & Data Flow

### 📚 Udemy Course Reference

**Section 5: Theory and Foundations**

- Lecture: Exploring Node Types and Categories in n8n
- Lecture: Core Nodes in n8n (Set, Merge, IF, Switch)
- Lecture: **CRITICAL**: JSON in n8n - How Data Really Flows Between Nodes
- Lecture: Mastering Expressions: Interactive Template

### 🎥 Essential YouTube Tutorials

1. **[n8n Nodes Explained: Beginner's Guide to Workflow Automation](https://www.youtube.com/watch?v=example)**
   - Understanding node types (Trigger, Regular, Output)
   - Node categories and use cases
   - Duration: ~15 minutes

2. **[A (free) practical guide to n8n - start here | COMPLETE n8n crash course [Part 1]](https://www.youtube.com/watch?v=example)**
   - Foundation concepts
   - Building your first workflow
   - Data flow visualization
   - Duration: ~40 minutes

3. **[Filter data fast with n8n expressions](https://www.youtube.com/watch?v=example)**
   - Expression syntax
   - Filtering techniques
   - Conditional logic
   - Duration: ~12 minutes

4. **[Full n8n guide - 2-minute recap](https://www.youtube.com/watch?v=example)**
   - Quick reference guide
   - Essential concepts summary
   - Duration: 2 minutes

### 🧠 The Node-Based Execution Model

#### Understanding n8n's Data Structure

Every node in n8n receives and outputs data in this format:

```json
[
  {
    "json": {
      "name": "John Doe",
      "email": "john@example.com",
      "company": "Acme Corp"
    },
    "binary": {}
  }
]
```

**Key Concept**: n8n processes data as an **Array of Objects**, not a single blob.

#### Implicit Looping

If a node receives an array with 100 items (e.g., 100 rows from Google Sheets), it will execute its logic **100 times**, once for each item.

**Example**:
- Google Sheets returns 100 customer records
- HTTP Request node will make 100 separate API calls
- Slack node will send 100 separate messages

**⚠️ Common Mistake**: Accidentally sending 100 individual Slack notifications instead of one batched summary.

**✅ Solution**: Use aggregation nodes or the Code node to batch data before output.

### 🔧 Core Nodes Deep Dive

#### 1. Set Node (Edit Fields)

**Purpose**: Transform, rename, or filter JSON fields

**Use Cases**:
- Rename fields to match API requirements
- Calculate new values
- Remove unnecessary data
- Format dates/strings

**Example**:
```javascript
// Input
{ "firstName": "John", "lastName": "Doe" }

// Set Node Configuration
fullName = {{$json.firstName}} {{$json.lastName}}

// Output
{ "firstName": "John", "lastName": "Doe", "fullName": "John Doe" }
```

#### 2. IF Node

**Purpose**: Create conditional logic paths

**Use Cases**:
- Route high-value leads to sales team
- Filter spam from legitimate contacts
- Trigger different actions based on status

**Example Logic**:
```
IF lead_score > 80
  → Send to Sales Rep (Slack notification)
ELSE
  → Add to nurture campaign (Email sequence)
```

#### 3. Switch Node

**Purpose**: Multi-path routing (like a switch statement in programming)

**Use Cases**:
- Route by customer tier (Free, Pro, Enterprise)
- Handle different event types
- Geographic routing

**Example**:
```
SWITCH customer_tier
  CASE "Enterprise" → Assign to dedicated account manager
  CASE "Pro" → Standard support queue
  CASE "Free" → Self-service knowledge base
```

#### 4. Merge Node

**Purpose**: Combine data from multiple sources

**The Most Complex Node in n8n** - Three modes:

##### Mode 1: Append
Stacks data from two branches vertically.

```
Branch A: [item1, item2]
Branch B: [item3, item4]
Result: [item1, item2, item3, item4]
```

##### Mode 2: Merge by Index
Combines data based on position.

```
Branch A: [{"name": "John"}, {"name": "Jane"}]
Branch B: [{"age": 30}, {"age": 25}]
Result: [{"name": "John", "age": 30}, {"name": "Jane", "age": 25}]
```

##### Mode 3: Merge by Key (SQL JOIN)
Matches data based on a shared identifier.

```
Branch A: [{"email": "john@ex.com", "name": "John"}]
Branch B: [{"email": "john@ex.com", "score": 95}]
Result: [{"email": "john@ex.com", "name": "John", "score": 95}]
```

### 🛠️ Hands-On Project: "The Data Transformer"

**Objective**: Master data flow and transformation through a real-world scenario.

#### Scenario: Lead Enrichment Pipeline

1. **Trigger**: Webhook receives form submission (Typeform/Google Form)
   ```json
   {
     "email": "prospect@company.com",
     "company": "Acme Corp",
     "interest": "Enterprise Plan"
   }
   ```

2. **Set Node**: Restructure and clean data
   - Lowercase email
   - Trim whitespace
   - Add timestamp

3. **HTTP Request**: Enrich with Clearbit API
   - Get company size
   - Get industry
   - Get tech stack

4. **IF Node**: Qualify lead
   - IF company_size > 500 employees → High Value
   - ELSE → Standard

5. **Merge Node**: Combine form data + enrichment data

6. **Switch Node**: Route based on qualification
   - High Value → Slack notification to sales team
   - Standard → Add to HubSpot nurture campaign

#### Expected Output:
```json
{
  "email": "prospect@company.com",
  "company": "Acme Corp",
  "interest": "Enterprise Plan",
  "company_size": 1200,
  "industry": "SaaS",
  "tech_stack": ["Salesforce", "AWS", "React"],
  "lead_score": 95,
  "qualification": "High Value",
  "timestamp": "2025-11-28T18:54:34Z"
}
```

### 💡 What You'll Master:

- ✅ Understanding n8n's implicit looping mechanism
- ✅ JSON data structure manipulation
- ✅ Conditional workflow logic (IF/Switch)
- ✅ Data merging patterns (Append, Index, Key)
- ✅ Building multi-step enrichment pipelines

### 📖 Additional Resources:

- [Designing the workflow - n8n Docs](https://docs.n8n.io/workflows/create/)
- [Automation Patterns in n8n](https://aminrj.com/automation-patterns-in-n8n)
- [n8n Best Practices: What 2,000+ Production Workflows Taught Me](https://www.youtube.com/watch?v=example)

---

## 1.3 Webhooks, APIs & HTTP Requests

### 📚 Udemy Course Reference

**Section 5: Theory and Foundations**

- Lecture: Introduction to Webhooks & APIs
- Lecture: HTTP Request Node Deep Dive
- Lecture: Authentication Methods (API Keys, OAuth2, JWT)

### 🎥 Essential YouTube Tutorials

1. **[n8n HTTP Request Node: The Ultimate Guide to API Integrations](https://www.youtube.com/watch?v=example)**
   - Comprehensive HTTP guide
   - All request methods (GET, POST, PUT, DELETE)
   - Header configuration
   - Duration: ~35 minutes

2. **[The n8n HTTP Request Node](https://www.youtube.com/watch?v=example)**
   - Core functionality
   - Common use cases
   - Duration: ~15 minutes

3. **[Step-by-Step: N8N Webhooks (From Beginner to Pro)](https://www.youtube.com/watch?v=example)**
   - Webhook fundamentals
   - Testing webhooks
   - Production setup
   - Duration: ~30 minutes

4. **[N8N Webhook Authentication - 3 ways to authenticate in n8n](https://www.youtube.com/watch?v=example)**
   - Security patterns
   - HMAC verification
   - API key validation
   - Duration: ~20 minutes

5. **[Understanding APIs in n8n (as a beginner)](https://www.youtube.com/watch?v=example)**
   - API fundamentals
   - REST concepts
   - Duration: ~18 minutes

6. **[How I connect to ANY API in 2 minutes (N8N and NoCode)](https://www.youtube.com/watch?v=example)**
   - Rapid integration technique
   - API documentation reading
   - Duration: ~10 minutes

### 🌐 The Universal Connector: HTTP Request Node

The HTTP Request Node is your gateway to connecting n8n with **any service** that has an API, regardless of whether a pre-built node exists.

#### Authentication Methods

##### 1. API Key Authentication

**Most Common Method** - Simple header or query parameter

```
Header: Authorization: Bearer sk_live_abc123xyz
```

**Example Services**: OpenAI, Stripe, SendGrid

##### 2. OAuth2 Authentication

**Complex but Secure** - Used by Google, LinkedIn, Facebook

**Flow**:
1. User authorizes your app
2. You receive authorization code
3. Exchange code for access token
4. Use access token for API calls
5. n8n automatically refreshes expired tokens

**Example Configuration**:
```
Authorization URL: https://accounts.google.com/o/oauth2/auth
Token URL: https://oauth2.googleapis.com/token
Client ID: your_client_id
Client Secret: your_client_secret
Scope: https://www.googleapis.com/auth/spreadsheets
```

##### 3. Basic Auth

**Username + Password** - Encoded in Base64

```
Header: Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

##### 4. Custom Headers

**Flexible Authentication** - Service-specific

```
Header: X-API-Key: your_api_key
Header: X-Custom-Auth: custom_value
```

### 🔄 Handling Pagination

Many APIs return data in "pages" (e.g., 50 results per request). You must loop to fetch all data.

#### Pattern 1: Offset-Based Pagination

```
Request 1: /api/users?offset=0&limit=50
Request 2: /api/users?offset=50&limit=50
Request 3: /api/users?offset=100&limit=50
```

#### Pattern 2: Cursor-Based Pagination

```
Request 1: /api/users?limit=50
Response: { "data": [...], "next_cursor": "abc123" }

Request 2: /api/users?limit=50&cursor=abc123
Response: { "data": [...], "next_cursor": "def456" }
```

#### n8n Implementation:

Use **Split in Batches** node + **Loop** to handle pagination automatically.

### ⏱️ Rate Limiting

**Problem**: APIs limit requests (e.g., 100 requests/minute)

**Solution**: Throttle your requests

**Methods**:
1. **Wait Node**: Add delay between requests
   ```
   Wait: 600ms between each request
   = 100 requests/minute
   ```

2. **Split in Batches**: Process in chunks
   ```
   Batch size: 10 items
   Wait: 1 second between batches
   ```

### 🛠️ Hands-On Project: "The Universal Connector"

**Objective**: Master API integration, authentication, and data retrieval.

#### Project Steps:

1. **Set up Webhook Trigger**
   - Create production webhook URL
   - Configure CORS if needed
   - Test with Postman/curl

2. **Query Public API (OpenWeatherMap)**
   ```
   GET https://api.openweathermap.org/data/2.5/weather
   Query Parameters:
     q: London
     appid: your_api_key
   ```

3. **Implement OAuth2 (Google Sheets API)**
   - Create Google Cloud project
   - Enable Sheets API
   - Configure OAuth2 credentials
   - Authorize n8n to access your sheets

4. **Handle Pagination (GitHub API)**
   ```
   GET https://api.github.com/repos/n8n-io/n8n/issues
   Query: per_page=100&page=1
   
   Loop until: response.length < 100
   ```

5. **Implement Rate Limiting**
   - Add Wait node: 1 second
   - Monitor API response headers:
     ```
     X-RateLimit-Remaining: 45
     X-RateLimit-Reset: 1638360000
     ```

### 🔐 Webhook Security

**Problem**: Anyone with your webhook URL can send data to your workflow.

**Solutions**:

#### 1. HMAC Signature Verification

Services like Shopify, Stripe sign their webhooks.

```javascript
// Code Node
const crypto = require('crypto');

const signature = $request.headers['x-shopify-hmac-sha256'];
const body = JSON.stringify($request.body);
const secret = 'your_webhook_secret';

const hash = crypto
  .createHmac('sha256', secret)
  .update(body)
  .digest('base64');

if (hash !== signature) {
  throw new Error('Invalid signature');
}

return items;
```

#### 2. API Key in Header

```
Require header: X-API-Key: your_secret_key
```

#### 3. IP Whitelisting

Only accept webhooks from specific IPs.

### 💡 What You'll Master:

- ✅ RESTful API integration with any service
- ✅ Authentication methods (API Key, OAuth2, Bearer Token, Basic Auth)
- ✅ Webhook security (HMAC verification, API keys)
- ✅ Error handling for API calls (retry logic, fallbacks)
- ✅ Pagination strategies (offset, cursor, page-based)
- ✅ Rate limiting to avoid API bans

### 📖 Additional Resources:

- [Beginner's Guide to n8n: Mastering REST APIs With No-Code](https://n8n-automation.com/rest-apis)
- [Definitive Guide to API Integration for Engineers](https://blog.n8n.io/api-integration-guide/)
- [n8n HTTP Request Node Documentation](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)

---

## 1.4 JavaScript & Code Node Mastery

### 📚 Udemy Course Reference

**Section 5: Theory and Foundations**

- Lecture: Introduction to JavaScript + Code Node Basics
- Lecture: Advanced JavaScript Patterns in n8n
- Lecture: Working with External Libraries

### 🎥 Essential YouTube Tutorials

1. **[N8N Code Node: Full Guide for Non-Coders! (Better Agents)](https://www.youtube.com/watch?v=example)**
   - Complete guide for beginners
   - No prior coding experience needed
   - Real-world examples
   - Duration: ~45 minutes

2. **[n8n Code Node Explained: Adding Custom Logic to Your Workflows](https://www.youtube.com/watch?v=example)**
   - Core concepts
   - When to use Code vs. built-in nodes
   - Duration: ~20 minutes

3. **[Only 1% of n8n Builders Know How To Use The Code Node](https://www.youtube.com/watch?v=example)**
   - Advanced patterns
   - Pro techniques
   - Duration: ~30 minutes

4. **[n8n Code Node Tutorial | Add Custom JavaScript for Advanced Workflow Automation](https://www.youtube.com/watch?v=example)**
   - Step-by-step tutorial
   - Multiple examples
   - Duration: ~35 minutes

5. **[n8n LangChain Code Node: From Beginner to Expert (3 Real Examples)](https://www.youtube.com/watch?v=example)**
   - Advanced AI integration
   - LangChain patterns
   - Duration: ~50 minutes

### 💻 Code Node Fundamentals

#### Accessing Input Data

```javascript
// Access all items
const items = $input.all();

// Access first item
const firstItem = $input.first();

// Access specific item
const item = $input.item;

// Access JSON data
const email = $input.item.json.email;
```

#### Returning Data

```javascript
// Return modified items
return items.map(item => ({
  json: {
    ...item.json,
    processed: true
  }
}));

// Return single item
return {
  json: {
    result: "success"
  }
};
```

### 🔧 Essential JavaScript Patterns

#### 1. Array Methods

##### .map() - Transform each item

```javascript
// Convert all emails to lowercase
const items = $input.all();

return items.map(item => ({
  json: {
    ...item.json,
    email: item.json.email.toLowerCase(),
    domain: item.json.email.split('@')[1]
  }
}));
```

##### .filter() - Keep only matching items

```javascript
// Keep only high-value leads
const items = $input.all();

return items.filter(item => 
  item.json.lead_score > 80
);
```

##### .reduce() - Aggregate data

```javascript
// Calculate total revenue
const items = $input.all();

const totalRevenue = items.reduce((sum, item) => 
  sum + item.json.amount, 0
);

return [{
  json: {
    total_revenue: totalRevenue,
    average: totalRevenue / items.length,
    count: items.length
  }
}];
```

#### 2. Regular Expressions (Regex)

##### Extract Email Domains

```javascript
const email = "john.doe@company.com";
const domain = email.match(/@(.+)$/)[1];
// Result: "company.com"
```

##### Validate Phone Numbers

```javascript
const phone = "+1-555-123-4567";
const isValid = /^\+?1?[-.]?\(?\d{3}\)?[-.]?\d{3}[-.]?\d{4}$/.test(phone);
// Result: true
```

##### Extract URLs from Text

```javascript
const text = "Check out https://example.com and http://test.com";
const urls = text.match(/https?:\/\/[^\s]+/g);
// Result: ["https://example.com", "http://test.com"]
```

#### 3. Data Structure Transformation

##### Flatten Nested JSON

```javascript
// Input: Nested Stripe webhook
const input = {
  customer: {
    name: "John Doe",
    email: "john@example.com",
    address: {
      city: "New York",
      country: "US"
    }
  }
};

// Flatten
const flattened = {
  customer_name: input.customer.name,
  customer_email: input.customer.email,
  customer_city: input.customer.address.city,
  customer_country: input.customer.address.country
};

return [{ json: flattened }];
```

##### Group by Category

```javascript
const items = $input.all();

const grouped = items.reduce((acc, item) => {
  const category = item.json.category;
  if (!acc[category]) {
    acc[category] = [];
  }
  acc[category].push(item.json);
  return acc;
}, {});

return [{ json: grouped }];
```

#### 4. Cryptographic Verification

##### Verify Webhook Signatures (Stripe Example)

```javascript
const crypto = require('crypto');

const signature = $request.headers['stripe-signature'];
const payload = JSON.stringify($request.body);
const secret = 'whsec_your_webhook_secret';

const expectedSignature = crypto
  .createHmac('sha256', secret)
  .update(payload)
  .digest('hex');

if (signature !== expectedSignature) {
  throw new Error('Invalid webhook signature');
}

return $input.all();
```

#### 5. Date Manipulation

```javascript
// Current timestamp
const now = new Date();

// Add 7 days
const nextWeek = new Date();
nextWeek.setDate(nextWeek.getDate() + 7);

// Format date
const formatted = now.toISOString().split('T')[0];
// Result: "2025-11-28"

// Calculate days between dates
const date1 = new Date('2025-11-01');
const date2 = new Date('2025-11-28');
const daysDiff = Math.floor((date2 - date1) / (1000 * 60 * 60 * 24));
// Result: 27
```

### 🛠️ Hands-On Project: "The Data Scientist"

**Objective**: Use Code Node for complex data processing that built-in nodes can't handle.

#### Scenario: Stripe Webhook Processing

1. **Parse Complex Nested JSON**

```javascript
const webhook = $input.first().json;

// Extract relevant data from deeply nested structure
const charge = {
  id: webhook.data.object.id,
  amount: webhook.data.object.amount / 100, // Convert cents to dollars
  currency: webhook.data.object.currency.toUpperCase(),
  customer_email: webhook.data.object.billing_details.email,
  card_last4: webhook.data.object.payment_method_details.card.last4,
  card_brand: webhook.data.object.payment_method_details.card.brand,
  created: new Date(webhook.data.object.created * 1000).toISOString()
};

return [{ json: charge }];
```

2. **Extract Email Domains with Regex**

```javascript
const items = $input.all();

return items.map(item => {
  const email = item.json.email;
  const domain = email.match(/@(.+)$/)[1];
  
  // Categorize by domain type
  let category;
  if (domain.endsWith('.edu')) {
    category = 'Education';
  } else if (domain.endsWith('.gov')) {
    category = 'Government';
  } else if (['gmail.com', 'yahoo.com', 'hotmail.com'].includes(domain)) {
    category = 'Personal';
  } else {
    category = 'Business';
  }
  
  return {
    json: {
      ...item.json,
      domain,
      category
    }
  };
});
```

3. **Process Arrays with .map(), .filter(), .reduce()**

```javascript
const orders = $input.all();

// Calculate statistics
const stats = {
  total_orders: orders.length,
  total_revenue: orders.reduce((sum, order) => sum + order.json.amount, 0),
  high_value_orders: orders.filter(order => order.json.amount > 1000).length,
  average_order_value: 0
};

stats.average_order_value = stats.total_revenue / stats.total_orders;

// Group by product
const byProduct = orders.reduce((acc, order) => {
  const product = order.json.product;
  if (!acc[product]) {
    acc[product] = { count: 0, revenue: 0 };
  }
  acc[product].count++;
  acc[product].revenue += order.json.amount;
  return acc;
}, {});

return [{
  json: {
    stats,
    by_product: byProduct
  }
}];
```

4. **Verify Webhook Signatures (HMAC + SHA256)**

```javascript
const crypto = require('crypto');

const items = $input.all();

return items.map(item => {
  const signature = item.json.signature;
  const payload = JSON.stringify(item.json.data);
  const secret = 'your_webhook_secret';
  
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  return {
    json: {
      ...item.json,
      signature_valid: signature === expectedSignature
    }
  };
});
```

5. **Custom Date/Time Calculations**

```javascript
const items = $input.all();

return items.map(item => {
  const createdAt = new Date(item.json.created_at);
  const now = new Date();
  
  // Calculate age in days
  const ageInDays = Math.floor((now - createdAt) / (1000 * 60 * 60 * 24));
  
  // Determine lifecycle stage
  let stage;
  if (ageInDays < 7) {
    stage = 'New';
  } else if (ageInDays < 30) {
    stage = 'Active';
  } else if (ageInDays < 90) {
    stage = 'Engaged';
  } else {
    stage = 'At Risk';
  }
  
  return {
    json: {
      ...item.json,
      age_in_days: ageInDays,
      lifecycle_stage: stage
    }
  };
});
```

### 💡 What You'll Master:

- ✅ JavaScript array methods (.map, .filter, .reduce)
- ✅ Regular expressions for pattern matching
- ✅ Data structure transformation (flatten, group, aggregate)
- ✅ Cryptography basics (HMAC, SHA256 for webhook verification)
- ✅ Date manipulation and calculations
- ✅ When to use Code Node vs. built-in nodes

### 📖 Interactive Learning:

- [Learn Code Node (JavaScript) with an Interactive Hands-On Tutorial](https://n8n.io/learn/code-node-tutorial)

### 🎯 Pro Tips:

> **Performance**: Code Node executes for EACH item. If processing 1000 items, your code runs 1000 times. For heavy operations, use `$input.all()` and process in batch.

> **Debugging**: Use `console.log()` to debug. Output appears in n8n execution logs.

> **External Libraries**: You can use Node.js built-in modules (`crypto`, `fs`, `path`) but NOT npm packages unless running n8n locally.

---

## 1.5 JSON & Data Transformation

### 📚 Udemy Course Reference

**Section 5: Theory and Foundations**

- Lecture: **CRITICAL**: JSON in n8n - How Data Really Flows Between Nodes
- Lecture: Data Transformation Strategies
- Lecture: Working with Complex Data Structures

### 🎥 Essential YouTube Tutorials

1. **⭐ [Master JSON & Data Transformation in 19 Minutes (Most Valuable n8n Skill)](https://www.youtube.com/watch?v=example)**
   - **MUST WATCH** - Most important n8n skill
   - Complete JSON mastery
   - Real-world transformations
   - Duration: 19 minutes

2. **[Master n8n JSON & Data Transformation in 30 Minutes](https://www.youtube.com/watch?v=example)**
   - Extended deep dive
   - Advanced techniques
   - Duration: 30 minutes

3. **[Learn n8n JSON & Data Transformation (Must Have n8n Skill)](https://www.youtube.com/watch?v=example)**
   - Fundamentals
   - Common patterns
   - Duration: 25 minutes

4. **[n8n Data Transformation Node | n8n Tutorial series for Beginners](https://www.youtube.com/watch?v=example)**
   - Node-specific guide
   - Practical examples
   - Duration: 15 minutes

5. **[How to Import a JSON File in n8n (Quick & Easy Tutorial) 2025](https://www.youtube.com/watch?v=example)**
   - File handling
   - Import/export
   - Duration: 8 minutes

6. **[n8n File Handling Made Easy | Convert to File Node](https://www.youtube.com/watch?v=example)**
   - File conversions
   - Binary data handling
   - Duration: 12 minutes

### 📊 Understanding JSON in n8n

#### The n8n Data Structure

```json
[
  {
    "json": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "metadata": {
        "source": "website",
        "campaign": "summer-2025"
      }
    },
    "binary": {},
    "pairedItem": {
      "item": 0
    }
  }
]
```

**Key Components**:
- **Array**: n8n always works with arrays (even single items)
- **json**: Contains your actual data
- **binary**: Contains file data (images, PDFs, etc.)
- **pairedItem**: Tracks data lineage through workflow

### 🔄 Common Transformation Patterns

#### 1. Accessing Nested Data

```javascript
// Expression syntax
{{$json.metadata.campaign}}
// Result: "summer-2025"

// Deep nesting
{{$json.customer.address.city}}
```

#### 2. Flattening Nested Structures

**Before** (API Response):
```json
{
  "user": {
    "profile": {
      "name": "John Doe",
      "contact": {
        "email": "john@example.com",
        "phone": "+1-555-0123"
      }
    }
  }
}
```

**After** (Flattened):
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "phone": "+1-555-0123"
}
```

**Code Node Implementation**:
```javascript
const items = $input.all();

return items.map(item => ({
  json: {
    name: item.json.user.profile.name,
    email: item.json.user.profile.contact.email,
    phone: item.json.user.profile.contact.phone
  }
}));
```

#### 3. Renaming Keys

**Use Case**: API expects different field names

```javascript
// Set Node (Edit Fields)
// Rename: firstName → first_name
// Rename: lastName → last_name

// Or Code Node:
const items = $input.all();

return items.map(item => ({
  json: {
    first_name: item.json.firstName,
    last_name: item.json.lastName,
    email_address: item.json.email
  }
}));
```

#### 4. Aggregation

**Calculate totals, averages, counts**

```javascript
const items = $input.all();

const total = items.reduce((sum, item) => sum + item.json.amount, 0);
const average = total / items.length;
const max = Math.max(...items.map(item => item.json.amount));
const min = Math.min(...items.map(item => item.json.amount));

return [{
  json: {
    total,
    average,
    max,
    min,
    count: items.length
  }
}];
```

#### 5. Array to Object Conversion

```javascript
// Input: Array of key-value pairs
const items = [
  { json: { key: "name", value: "John" } },
  { json: { key: "email", value: "john@example.com" } }
];

// Convert to object
const obj = items.reduce((acc, item) => {
  acc[item.json.key] = item.json.value;
  return acc;
}, {});

// Result: { name: "John", email: "john@example.com" }
```

### 📁 File Handling

#### CSV to JSON

```javascript
// Use "Read Binary File" node + "Convert to File" node
// Or Code Node with csv-parse

const csv = require('csv-parse/sync');

const csvData = $input.first().binary.data;
const records = csv.parse(csvData, {
  columns: true,
  skip_empty_lines: true
});

return records.map(record => ({ json: record }));
```

#### JSON to CSV

```javascript
const items = $input.all();

// Create CSV header
const headers = Object.keys(items[0].json).join(',');

// Create CSV rows
const rows = items.map(item => 
  Object.values(item.json).join(',')
);

const csv = [headers, ...rows].join('\n');

return [{
  json: {},
  binary: {
    data: Buffer.from(csv).toString('base64'),
    mimeType: 'text/csv',
    fileName: 'export.csv'
  }
}];
```

### 🛠️ Hands-On Project: "The Data Alchemist"

**Objective**: Master all data transformation techniques in one comprehensive workflow.

#### Project Steps:

1. **Import CSV File**
   - Use "Read Binary File" node
   - Point to sample CSV (customers.csv)
   - Convert to JSON

2. **Flatten Nested JSON**
   
   **Input** (API Response):
   ```json
   {
     "customer": {
       "id": 123,
       "profile": {
         "name": "Acme Corp",
         "industry": "SaaS"
       },
       "billing": {
         "plan": "Enterprise",
         "mrr": 5000
       }
     }
   }
   ```
   
   **Output** (Flattened):
   ```json
   {
     "customer_id": 123,
     "name": "Acme Corp",
     "industry": "SaaS",
     "plan": "Enterprise",
     "mrr": 5000
   }
   ```

3. **Rename Keys with Set Node**
   - `customer_id` → `id`
   - `mrr` → `monthly_revenue`

4. **Aggregate Data with Code Node**
   
   ```javascript
   const items = $input.all();
   
   const stats = {
     total_customers: items.length,
     total_mrr: items.reduce((sum, item) => sum + item.json.monthly_revenue, 0),
     industries: [...new Set(items.map(item => item.json.industry))],
     by_plan: items.reduce((acc, item) => {
       const plan = item.json.plan;
       acc[plan] = (acc[plan] || 0) + 1;
       return acc;
     }, {})
   };
   
   return [{ json: stats }];
   ```

5. **Export to Google Sheets**
   - Use "Google Sheets" node
   - Append rows
   - Format as table

#### Expected Final Output:

```json
{
  "total_customers": 150,
  "total_mrr": 450000,
  "average_mrr": 3000,
  "industries": ["SaaS", "E-commerce", "Healthcare"],
  "by_plan": {
    "Starter": 80,
    "Pro": 50,
    "Enterprise": 20
  }
}
```

### 💡 What You'll Master:

- ✅ JSON structure understanding (arrays, objects, nesting)
- ✅ Data flattening techniques for complex APIs
- ✅ Key renaming strategies for API compatibility
- ✅ Aggregation methods (sum, average, count, group by)
- ✅ CSV/JSON conversion for data import/export
- ✅ Binary file handling (images, PDFs, documents)

### 📖 Additional Resources:

- [How I'm Using n8n to Automate the Boring Parts of Data Science](https://medium.com/data-science-automation)
- [n8n Data Transformation Best Practices](https://docs.n8n.io/data/)

### 🎯 Pro Tips:

> **Expression Syntax**: Use `{{$json.fieldName}}` in any text field to reference data. Use `{{$json["field-with-dashes"]}}` for fields with special characters.

> **Debugging JSON**: Use the "Edit Fields (Set)" node in "Keep Only Set" mode to see exactly what fields you're working with.

> **Performance**: For large datasets (10,000+ items), use pagination and batch processing to avoid memory issues.

---

## ✅ Phase 1 Checkpoint

Before proceeding to Phase 2, you should be able to:

- ✅ Deploy and maintain a self-hosted n8n instance with PostgreSQL
- ✅ Understand n8n's implicit looping and item structure
- ✅ Create workflows with triggers, logic nodes, and actions
- ✅ Connect to any REST API with proper authentication (API Key, OAuth2)
- ✅ Write custom JavaScript in Code Nodes for complex transformations
- ✅ Transform complex JSON data structures (flatten, aggregate, rename)
- ✅ Implement error handling and retry logic
- ✅ Secure workflows with proper authentication and webhook verification

### 🎯 Validation Project: "The Complete Pipeline"

**Build a workflow that demonstrates all Phase 1 skills:**

1. **Trigger**: Webhook receives new customer data
2. **Data Cleaning**: Use Set node to normalize fields
3. **API Call**: Enrich with Clearbit API (OAuth2)
4. **Conditional Logic**: IF node routes by company size
5. **Data Transformation**: Code node calculates lead score
6. **Aggregation**: Group customers by industry
7. **Output**: Send formatted Slack notification with results
8. **Error Handling**: Catch API failures and retry

**Success Criteria**:
- Workflow runs without errors
- Data flows correctly through all nodes
- Slack notification contains enriched, formatted data
- Errors are caught and handled gracefully

---

## 🎓 Next Steps

You've mastered the engineering foundations! You now understand:
- How n8n processes data (JSON arrays, implicit looping)
- How to connect to any API (authentication, pagination, rate limiting)
- How to write custom logic (JavaScript, regex, data transformation)
- How to build production-ready infrastructure (Docker, PostgreSQL, SSL)

**Ready for Phase 2?** → [Infinite Use Cases - Business Automation](./02_infinite_use_cases.md)

In Phase 2, you'll apply these foundations to build production-ready workflows across:
- Marketing automation (SEO, social media, video content)
- Sales intelligence (lead generation, cold email, CRM)
- Operations (invoices, finance, onboarding)
- 50+ industry-specific solutions (Real Estate, Healthcare, Legal, etc.)

---

**📚 Quick Reference Card**

| Concept | Key Takeaway |
|---------|-------------|
| **Data Structure** | Always an array of objects with `json` and `binary` properties |
| **Implicit Looping** | Nodes execute once per item automatically |
| **Set Node** | Rename, add, or remove fields |
| **IF Node** | Two-path conditional routing |
| **Switch Node** | Multi-path routing |
| **Merge Node** | Combine data (Append, Index, Key) |
| **HTTP Request** | Universal API connector |
| **Code Node** | Custom JavaScript for complex logic |
| **Expressions** | `{{$json.fieldName}}` to reference data |

---

*Last Updated: November 28, 2025*
