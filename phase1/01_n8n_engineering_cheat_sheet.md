# 📘 PHASE 1: Engineering Foundations - Complete Cheat Sheet

> **Copy-Paste Ready Reference** - Every essential pattern, command, and code snippet from Phase 1.

---

## 🐳 1. Docker & Infrastructure

### Essential Docker Commands
```bash
# Start n8n
docker-compose up -d

# Stop n8n  
docker-compose down

# View live logs
docker-compose logs -f n8n

# Update n8n to latest version
docker-compose pull && docker-compose up -d

# Check running containers
docker ps

# Restart n8n
docker restart n8n

# Access container shell
docker exec -it n8n /bin/sh

# Remove all stopped containers
docker container prune

# View disk usage
docker system df
```

### Production Docker Compose File

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_USER: n8n
      POSTGRES_PASSWORD: CHANGE_THIS_PASSWORD
      POSTGRES_DB: n8n
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U n8n']
      interval: 5s
      timeout: 5s
      retries: 10

  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      # Database
      DB_TYPE: postgresdb
      DB_POSTGRESDB_HOST: postgres
      DB_POSTGRESDB_PORT: 5432
      DB_POSTGRESDB_DATABASE: n8n
      DB_POSTGRESDB_USER: n8n
      DB_POSTGRESDB_PASSWORD: CHANGE_THIS_PASSWORD
      
      # Security
      N8N_BASIC_AUTH_ACTIVE: true
      N8N_BASIC_AUTH_USER: admin
      N8N_BASIC_AUTH_PASSWORD: CHANGE_THIS_PASSWORD
      
      # Domain & SSL
      N8N_HOST: n8n.yourdomain.com
      N8N_PROTOCOL: https
      WEBHOOK_URL: https://n8n.yourdomain.com/
      
      # Timezone
      GENERIC_TIMEZONE: America/New_York
      TZ: America/New_York
      
      # Execution
      EXECUTIONS_MODE: queue
      QUEUE_BULL_REDIS_HOST: redis
      
    volumes:
      - n8n_data:/home/node/.n8n
    depends_on:
      postgres:
        condition: service_healthy

  redis:
    image: redis:7-alpine
    restart: always
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  n8n_data:
  redis_data:
```

### Critical Environment Variables

```bash
# .env file
DOMAIN_NAME=n8n.yourdomain.com

# Database
DB_TYPE=postgresdb
DB_POSTGRESDB_PASSWORD=YOUR_SECURE_PASSWORD

# Security
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=YOUR_ADMIN_PASSWORD

# Webhook URL (CRITICAL for OAuth)
WEBHOOK_URL=https://n8n.yourdomain.com/

# Timezone
GENERIC_TIMEZONE=America/New_York
TZ=America/New_York

# Execution Mode (for high volume)
EXECUTIONS_MODE=queue
QUEUE_BULL_REDIS_HOST=redis
```

---

## 🔄 2. Core Concepts & Data Flow

### n8n Data Structure

```json
[
  {
    "json": {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com"
    },
    "binary": {},
    "pairedItem": { "item": 0 }
  }
]
```

**Key Points:**
- Always an **array** (even for single item)
- `json` = your actual data
- `binary` = files (PDFs, images, etc.)
- `pairedItem` = tracks data lineage

### Accessing Data in Expressions

```javascript
// Access current item's field
{{$json.email}}

// Access nested field
{{$json.customer.address.city}}

// Access with special characters
{{$json["field-with-dashes"]}}

// Access previous node
{{$node["Node Name"].json.fieldName}}

// Access first item from previous node
{{$node["Node Name"].first().json.fieldName}}

// Access all items from previous node
{{$node["Node Name"].all()}}

// Current item index
{{$itemIndex}}

// Workflow ID
{{$workflow.id}}

// Workflow name
{{$workflow.name}}
```

### Implicit Looping Behavior

```
Input Items: 100
↓
HTTP Request Node
↓
Executes 100 times (once per item)
↓
Output Items: 100
```

**⚠️ Common Mistake:** Sending 100 individual Slack messages instead of one batched message.

**✅ Solution:** Use aggregation before output nodes.

### Core Nodes Behavior

| Node | Purpose | Example |
|------|---------|---------|
| **Set** | Add/remove/rename fields | Rename `firstName` → `first_name` |
| **IF** | Two-path routing | `IF score > 80 → Hot Lead` |
| **Switch** | Multi-path routing | Route by customer tier |
| **Merge** | Combine data streams | Join customer + subscription data |
| **Code** | Custom JavaScript | Complex transformations |
| **Split In Batches** | Process in chunks | Handle pagination |

### Merge Node - 3 Modes

#### Mode 1: Append (Stack Vertically)
```
Input 1: [A, B]
Input 2: [C, D]
Output: [A, B, C, D]
```

#### Mode 2: Merge By Index (Combine by Position)
```
Input 1: [{name: "John"}, {name: "Jane"}]
Input 2: [{age: 30}, {age: 25}]
Output: [{name: "John", age: 30}, {name: "Jane", age: 25}]
```

#### Mode 3: Merge By Key (SQL JOIN)
```
Input 1: [{email: "john@ex.com", name: "John"}]
Input 2: [{email: "john@ex.com", score: 95}]
Output: [{email: "john@ex.com", name: "John", score: 95}]
```

---

## 🌐 3. HTTP Request & API Integration

### Authentication Methods

#### API Key (Header)
```
Header Name: Authorization
Header Value: Bearer YOUR_API_KEY
```

#### API Key (Custom Header)
```
Header Name: X-API-Key
Header Value: YOUR_API_KEY
```

#### Basic Auth
```
Username: your_username
Password: your_password
```
(n8n auto-converts to Base64)

#### OAuth2 Configuration
```
Authorization URL: https://accounts.google.com/o/oauth2/auth
Token URL: https://oauth2.googleapis.com/token
Client ID: YOUR_CLIENT_ID
Client Secret: YOUR_CLIENT_SECRET
Scope: https://www.googleapis.com/auth/spreadsheets
```

### Common HTTP Methods

| Method | Purpose | Body Required |
|--------|---------|---------------|
| GET | Retrieve data | No |
| POST | Create data | Yes |
| PUT | Update (full replace) | Yes |
| PATCH | Update (partial) | Yes |
| DELETE | Remove data | No |

### Pagination Patterns

#### Offset-Based
```javascript
// Page 1
GET /api/users?offset=0&limit=50

// Page 2  
GET /api/users?offset=50&limit=50

// Page 3
GET /api/users?offset=100&limit=50
```

#### Cursor-Based
```javascript
// Page 1
GET /api/users?limit=50
// Response: { data: [...], next_cursor: "abc123" }

// Page 2
GET /api/users?limit=50&cursor=abc123
// Response: { data: [...], next_cursor: "def456" }
```

#### n8n Pagination Loop
```
1. HTTP Request (get page 1)
2. Code Node (check if more pages exist)
3. IF has_more_pages == true
   → Loop back to step 1 (increment offset/cursor)
4. IF has_more_pages == false
   → Continue workflow
```

### Rate Limiting Strategies

```javascript
// Strategy 1: Wait Node
// Add 1 second delay between requests
Wait: 1000ms

// Strategy 2: Split In Batches
Batch Size: 10 items
Wait: 5 seconds between batches
= 120 requests per minute

// Strategy 3: Check header limits
const remaining = $response.headers['x-ratelimit-remaining'];
if (remaining < 5) {
  // Wait until reset time
  const resetTime = $response.headers['x-ratelimit-reset'];
  // Calculate wait time
}
```

### Webhook Security - HMAC Validation

```javascript
// Code Node - Verify Stripe Webhook
const crypto = require('crypto');

const signature = $input.item.json.headers['stripe-signature'];
const body = JSON.stringify($input.item.json.body);
const secret = 'whsec_YOUR_WEBHOOK_SECRET';

const expectedSignature = crypto
  .createHmac('sha256', secret)
  .update(body)
  .digest('hex');

if (signature !== expectedSignature) {
  throw new Error('⛔ Invalid webhook signature - Possible attack!');
}

return $input.all();
```

---

## 💻 4. JavaScript & Code Node Mastery

### Accessing Input Data

```javascript
// Get all items
const items = $input.all();

// Get first item
const firstItem = $input.first();

// Get current item
const item = $input.item;

// Get specific field
const email = $input.item.json.email;

// Get binary data
const file = $input.item.binary.data;
```

### Returning Data

```javascript
// Return multiple items
return items.map(item => ({
  json: {
    id: item.json.id,
    processed: true
  }
}));

// Return single item
return {
  json: {
    result: "success",
    count: items.length
  }
};

// Return nothing (end execution)
return [];
```

### Essential Array Methods

#### .map() - Transform Each Item
```javascript
const items = $input.all();

return items.map(item => ({
  json: {
    // Combine first + last name
    full_name: `${item.json.firstName} ${item.json.lastName}`,
    // Lowercase email
    email: item.json.email.toLowerCase(),
    // Extract domain
    domain: item.json.email.split('@')[1],
    // Add timestamp
    processed_at: new Date().toISOString()
  }
}));
```

#### .filter() - Keep Only Matching Items
```javascript
const items = $input.all();

return items.filter(item => {
  const score = item.json.lead_score;
  const email = item.json.email;
  
  return score > 80 && // High quality
         email && // Email exists
         email.includes('@') && // Valid email
         !email.endsWith('.edu'); // Not student
});
```

#### .reduce() - Aggregate Data
```javascript
const items = $input.all();

// Calculate total revenue
const totalRevenue = items.reduce((sum, item) => {
  return sum + item.json.amount;
}, 0);

// Calculate average
const average = totalRevenue / items.length;

// Find max
const maxAmount = Math.max(...items.map(item => item.json.amount));

// Find min
const minAmount = Math.min(...items.map(item => item.json.amount));

return [{
  json: {
    total_revenue: totalRevenue,
    average_order_value: average,
    highest_order: maxAmount,
    lowest_order: minAmount,
    order_count: items.length
  }
}];
```

#### Group By Category
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

// Output:
// {
//   "Electronics": [{...}, {...}],
//   "Clothing": [{...}, {...}],
//   "Books": [{...}]
// }
```

### Regular Expressions (Regex)

#### Extract Email
```javascript
const text = "Contact us at support@company.com for help";
const email = text.match(/[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}/)[0];
// Result: "support@company.com"
```

#### Extract All URLs
```javascript
const text = "Visit https://example.com and http://test.com";
const urls = text.match(/https?:\/\/[^\s]+/g);
// Result: ["https://example.com", "http://test.com"]
```

#### Validate Phone Number (US)
```javascript
const phone = "+1-555-123-4567";
const isValid = /^\+?1?[-.]?\(?\d{3}\)?[-.]?\d{3}[-.]?\d{4}$/.test(phone);
// Result: true
```

#### Remove HTML Tags
```javascript
const html = "<p>Hello <strong>World</strong></p>";
const clean = html.replace(/<[^>]*>/g, '');
// Result: "Hello World"
```

#### Extract Hashtags
```javascript
const text = "Love this #automation #n8n #ai";
const hashtags = text.match(/#[a-z0-9_]+/gi);
// Result: ["#automation", "#n8n", "#ai"]
```

#### Extract Digits Only
```javascript
const text = "Order #12345 total: $456.78";
const digits = text.match(/\d+/g).join('');
// Result: "1234545678"
```

### Date Manipulation

```javascript
// Current timestamp
const now = new Date();

// ISO format
const iso = now.toISOString();
// "2025-11-29T16:30:00.000Z"

// Format as YYYY-MM-DD
const date = now.toISOString().split('T')[0];
// "2025-11-29"

// Add 7 days
const nextWeek = new Date();
nextWeek.setDate(nextWeek.getDate() + 7);

// Add 3 months
const future = new Date();
future.setMonth(future.getMonth() + 3);

// Calculate days between dates
const date1 = new Date('2025-11-01');
const date2 = new Date('2025-11-29');
const daysDiff = Math.floor((date2 - date1) / (1000 * 60 * 60 * 24));
// Result: 28 days

// Unix timestamp
const unixTimestamp = Math.floor(now.getTime() / 1000);

// Convert Unix to Date
const fromUnix = new Date(1701360000 * 1000);
```

### Flatten Nested JSON

```javascript
// Input: Complex nested structure
const input = $input.first().json;

// Flatten
return [{
  json: {
    customer_id: input.customer.id,
    customer_name: input.customer.profile.name,
    customer_email: input.customer.profile.contact.email,
    customer_phone: input.customer.profile.contact.phone,
    billing_plan: input.billing.plan,
    billing_mrr: input.billing.mrr,
    company_size: input.company.metadata.employee_count
  }
}];
```

### Error Handling

```javascript
const items = $input.all();

return items.map(item => {
  try {
    // Risky operation
    const domain = item.json.email.split('@')[1];
    
    return {
      json: {
        ...item.json,
        domain: domain,
        valid: true
      }
    };
  } catch (error) {
    // Handle error gracefully
    return {
      json: {
        ...item.json,
        domain: null,
        valid: false,
        error_message: error.message
      }
    };
  }
});
```

---

## 📊 5. JSON & Data Transformation

### Rename Fields (Set Node)

```
Operation: Keep Only Set
Fields to Set:
  first_name = {{$json.firstName}}
  last_name = {{$json.lastName}}
  email_address = {{$json.email}}
```

### Convert Array to Object

```javascript
// Input: [{key: "name", value: "John"}, {key: "age", value: 30}]
const items = $input.all();

const obj = items.reduce((acc, item) => {
  acc[item.json.key] = item.json.value;
  return acc;
}, {});

return [{ json: obj }];
// Output: {name: "John", age: 30}
```

### CSV to JSON

```javascript
// After "Read Binary File" node
const csvparse = require('csv-parse/sync');

const csvData = $input.first().binary.data;
const records = csvparse.parse(Buffer.from(csvData, 'base64'), {
  columns: true,
  skip_empty_lines: true
});

return records.map(record => ({ json: record }));
```

### JSON to CSV

```javascript
const items = $input.all();

// Create header row
const headers = Object.keys(items[0].json).join(',');

// Create data rows
const rows = items.map(item => {
  return Object.values(item.json)
    .map(val => `"${val}"`) // Wrap in quotes
    .join(',');
});

// Combine
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

### Safe Nested Access

```javascript
// Problem: Crashes if field doesn't exist
const city = item.json.customer.address.city; // ❌ Error if address is null

// Solution: Optional chaining
const city = item.json.customer?.address?.city || 'Unknown'; // ✅ Safe
```

---

## 🛡️ 6. Error Handling & Retry Logic

### Basic Try-Catch in Code Node

```javascript
const items = $input.all();

return items.map(item => {
  try {
    // Your logic here
    const result = riskyOperation(item.json);
    return { json: { success: true, result } };
  } catch (error) {
    return { 
      json: { 
        success: false, 
        error: error.message,
        originalData: item.json 
      } 
    };
  }
});
```

### Exponential Backoff Retry

```javascript
async function retryWithBackoff(fn, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // Wait: 1s, 2s, 4s, 8s...
      const waitTime = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
}

// Usage
const result = await retryWithBackoff(async () => {
  // Your API call here
  return await $http.get('https://api.example.com/data');
});

return [{ json: result }];
```

---

## 🎯 7. Quick Reference Table

| Task | Code Snippet |
|------|--------------|
| **Lowercase email** | `item.json.email.toLowerCase()` |
| **Extract domain** | `item.json.email.split('@')[1]` |
| **Calculate sum** | `items.reduce((sum, i) => sum + i.json.amount, 0)` |
| **Filter by score** | `items.filter(i => i.json.score > 80)` |
| **Current timestamp** | `new Date().toISOString()` |
| **Wait 5 seconds** | `await new Promise(r => setTimeout(r, 5000))` |
| **Random number** | `Math.floor(Math.random() * 100)` |
| **Round to 2 decimals** | `Math.round(num * 100) / 100` |
| **Check if exists** | `item.json.field?.exists || false` |
| **Remove duplicates** | `[...new Set(array)]` |

---

**🎓 Pro Tips:**
1. Always use `try-catch` for risky operations
2. Test with 1-2 items before processing thousands
3. Use `console.log()` for debugging (check execution logs)
4. Keep Code Nodes under 50 lines for readability
5. Name variables clearly: `customerEmail` not `e`
