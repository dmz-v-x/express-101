## `express.json()` — Parsing JSON Payloads in Express

### 1. First: The Problem `express.json()` Solves

When a client sends JSON data to your server:

Example request body:

{
  "name": "Alice",
  "age": 25
}

Express initially sees:

❌ Raw text  
❌ Raw bytes  
❌ NOT a JavaScript object  

Without parsing:

    req.body === undefined

We need a parser.

---

### 2. What Is `express.json()`?

`express.json()` is:

> A built-in middleware that parses JSON request bodies

It:

- Reads incoming request body
- Converts JSON text → JavaScript object
- Attaches result to `req.body`

---

### 3. Why Parsing Is Necessary

Incoming JSON:

{
  "name": "Alice"
}

After parsing:

{
  name: "Alice"
}

Without parsing:

❌ Cannot safely access fields  
❌ No structured data  

---

### 4. Enabling JSON Parsing 

You must explicitly enable it:

    app.use(express.json());

This tells Express:

"Parse JSON request bodies."

---

## Hands-On Demo — Minimal Working Server

---

### 5. Basic Server Setup

    const express = require("express");
    const app = express();

    app.use(express.json());

    app.post("/users", (req, res) => {
      console.log(req.body);
      res.json(req.body);
    });

    app.listen(3000, () => {
      console.log("Server running on port 3000");
    });

---

### 6. What This Code Does

- Enables JSON parser
- Accepts POST requests
- Prints parsed body
- Sends parsed JSON back

---

### 7. Testing with curl

    curl -X POST http://localhost:3000/users \
    -H "Content-Type: application/json" \
    -d '{"name":"Alice"}'

Terminal output:

    { name: 'Alice' }

Response:

{
  "name": "Alice"
}

---

### 8. What Happened Internally

Request arrives → Raw JSON  
↓  
`express.json()` runs  
↓  
JSON parsed  
↓  
`req.body` populated  
↓  
Route handler runs  

---

### 9. Critical Rule — Content-Type Header

JSON parsing only works when:

    Content-Type: application/json

Missing header:

❌ Parser ignores body  
❌ `req.body` undefined  

---

### 10. Why Content-Type Matters

Middleware decides behavior based on headers.

Correct header → correct parser  
Wrong header → parser skipped  

---

### 11. What If You Forget `express.json()`?

Example:

    app.post("/users", (req, res) => {
      console.log(req.body);
    });

Output:

    undefined

Because:

No parser enabled.

---

### 12. Parsing Only Happens for Supported Methods

JSON bodies typically exist in:

- POST
- PUT
- PATCH

Rare for:

- GET
- DELETE

---

### 13. Accessing Parsed Data

    app.post("/users", (req, res) => {
      const name = req.body.name;
      res.send(name);
    });

Now structured data is usable.

---

## Advanced Configuration

---

### 14. Limiting Payload Size

    app.use(express.json({ limit: "1mb" }));

Prevents:

- Huge payload attacks
- Memory abuse

---

### 15. Why Size Limits Matter

Large bodies:

❌ Consume memory  
❌ Slow parsing  
❌ Risk crashes  

Production apps always define limits.

---

### 16. Handling Invalid JSON

Invalid JSON:

    {"name": Alice}

Result:

❌ Parser throws error  
❌ Error middleware triggered  

Express automatically detects parsing failures.

---

### 17. Example Error Flow

Bad JSON → Parser fails  
↓  
Error middleware runs  
↓  
Response sent  

---

### 18. Advanced: Strict vs Non-Strict Parsing

Default behavior:

- Only valid JSON accepted
- Invalid syntax rejected

This improves safety.

---

### 19. Performance Insight

JSON parsing:

- Requires CPU work
- Happens per request
- Should not be abused

Avoid unnecessarily large payloads.

---

### 20. Selective JSON Parsing 

Instead of global parsing:

    app.post("/users", express.json(), handler);

Parser runs only where needed.

Useful for performance tuning.

---

### 21. Common Beginner Mistakes

❌ Forgetting middleware  
❌ Wrong Content-Type  
❌ Expecting body in GET  
❌ Sending invalid JSON  
❌ No payload size limits  

---

### 22. Professional Best Practices

✅ Always enable parser when accepting JSON  
✅ Validate parsed data  
✅ Limit payload sizes  
✅ Use consistent JSON structure  
✅ Handle parsing errors  
✅ Never trust client input blindly  

---

### 23. Final Mental Model

Client sends raw JSON  
↓  
`express.json()` parses it  
↓  
Parsed object appears in `req.body`

No parser → No body

Without it → APIs cannot process JSON  
With it → Structured request handling becomes possible
