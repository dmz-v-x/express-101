## Handling Malformed JSON Errors in Express

### 1. First: What Is Malformed JSON?

Malformed JSON means:

> JSON data that is syntactically incorrect

Valid JSON:

{
  "name": "Alice"
}

Malformed JSON:

{
  "name": Alice
}

Problem:

❌ Missing quotes around `Alice`

---

### 2. Why This Matters in Real Applications

Clients can send:

- Broken JSON
- Truncated JSON
- Invalid syntax
- Corrupted payloads

Without proper handling:

❌ Server may crash  
❌ Users receive unclear errors  
❌ Debugging becomes difficult  

---

### 3. What Happens Inside Express

When JSON parsing is enabled:

    app.use(express.json());

Express:

1. Reads request body
2. Attempts JSON parsing
3. Throws error if parsing fails

Malformed JSON → Parsing error

---

### 4. Typical Parsing Error

Invalid JSON triggers:

> SyntaxError

Example message:

Unexpected token A in JSON

---

## Hands-On Demonstration

---

### 5. Minimal Server Setup

    const express = require("express");
    const app = express();

    app.use(express.json());

    app.post("/data", (req, res) => {
      res.json({ received: true });
    });

    app.listen(3000);

---

### 6. Sending Valid JSON

    curl -X POST http://localhost:3000/data \
    -H "Content-Type: application/json" \
    -d '{"name":"Alice"}'

Response:

{
  "received": true
}

Works normally.

---

### 7. Sending Malformed JSON

    curl -X POST http://localhost:3000/data \
    -H "Content-Type: application/json" \
    -d '{"name":Alice}'

Result:

❌ Parsing failure  
❌ Error triggered  

---

### 8. Default Express Behavior

Without custom error handling:

- Express returns generic error
- Stack traces may appear (development)
- Response may be unclear

Not ideal for production.

---

## Proper Error Handling Strategy

---

### 9. Centralized Error Middleware

Add error handler:

    app.use((err, req, res, next) => {
      res.status(500).json({
        error: "Server error"
      });
    });

But this is too generic.

We need precision.

---

### 10. Detecting JSON Parsing Errors (Critical Step)

Malformed JSON errors have a specific signature:

    err.type === "entity.parse.failed"

---

### 11. Correct Production-Safe Error Handler

    app.use((err, req, res, next) => {
      if (err.type === "entity.parse.failed") {
        return res.status(400).json({
          error: "Invalid JSON payload"
        });
      }

      next(err);
    });

---

### 12. Why Status Code 400?

Malformed JSON is:

> Client error

Not server failure.

Correct classification:

400 Bad Request

---

### 13. Resulting Behavior

Malformed JSON → Clean response:

{
  "error": "Invalid JSON payload"
}

Instead of confusing server errors.

---

## Understanding the Error Internals

---

### 14. What Actually Fails?

JSON parsing fails during:

    express.json()

Before route handlers run.

Pipeline:

Request → JSON Parser → ERROR → Error Middleware

---

### 15. Important Insight

Malformed JSON:

❌ Never reaches your route  
❌ Handler never executes  

Parser blocks execution early.

---

## Advanced Defensive Techniques

---

### 16. Logging Malformed JSON Attempts

    app.use((err, req, res, next) => {
      if (err.type === "entity.parse.failed") {
        console.error("Malformed JSON:", err.message);

        return res.status(400).json({
          error: "Invalid JSON payload"
        });
      }

      next(err);
    });

Useful for:

- Debugging clients
- Security monitoring
- Detecting abuse

---

### 17. Avoid Leaking Internal Errors

Bad practice:

    res.json({ error: err.message });

Why dangerous?

❌ Exposes parser internals  
❌ Reveals implementation details  

Better:

Return controlled messages.

---

### 18. Limiting Payload Size (Related Protection)

Malformed JSON + huge payload = attack vector.

Safe setup:

    app.use(express.json({ limit: "1mb" }));

---

### 19. Handling Multiple Error Types (Production Pattern)

    app.use((err, req, res, next) => {
      if (err.type === "entity.parse.failed") {
        return res.status(400).json({
          error: "Invalid JSON payload"
        });
      }

      if (err.type === "entity.too.large") {
        return res.status(413).json({
          error: "Payload too large"
        });
      }

      res.status(500).json({
        error: "Internal server error"
      });
    });

---

## Common Beginner Mistakes

---

### 20. Typical Errors

❌ Ignoring parser errors  
❌ Returning 500 instead of 400  
❌ Exposing raw error messages  
❌ Forgetting error middleware  
❌ Assuming malformed JSON reaches routes  
❌ Not setting payload limits  

---

## Professional Best Practices

---

### 21. Production Guidelines

✅ Always handle JSON parsing errors  
✅ Return 400 Bad Request  
✅ Use clean error messages  
✅ Avoid leaking internals  
✅ Log parsing failures  
✅ Define payload limits  
✅ Combine with validation  
✅ Treat malformed JSON as client fault  

---

### 22. Final Mental Model

Client sends JSON  
↓  
`express.json()` attempts parsing  
↓  
If invalid → Parsing error  
↓  
Error middleware handles it  

Malformed JSON = Pipeline failure before routes

