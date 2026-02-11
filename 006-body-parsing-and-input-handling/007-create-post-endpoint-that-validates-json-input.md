## Creating a POST Endpoint That Validates JSON Input in Express

### 1. First: Why Validation Is Necessary

When clients send JSON data:

❌ You cannot trust it  
❌ Fields may be missing  
❌ Data types may be wrong  
❌ Values may be invalid  
❌ Payload may be malicious  

Without validation:

- Bugs appear
- Database corruption occurs
- Security risks increase

Validation is mandatory.

---

### 2. Example Scenario

Client sends:

POST /users  
Body:

{
  "name": "Alice",
  "email": "alice@test.com"
}

Server must verify:

- Required fields exist
- Data types are correct
- Values are acceptable

---

### 3. Enabling JSON Parsing (Required Step)

Before validation:

    app.use(express.json());

Without this:

    req.body === undefined

---

## Hands-On Demo — Minimal Validated POST Endpoint

---

### 4. Basic Server Setup

    const express = require("express");
    const app = express();

    app.use(express.json());

---

### 5. Creating the POST Endpoint

    app.post("/users", (req, res) => {
      const { name, email } = req.body || {};

      if (!name) {
        return res.status(400).json({
          error: "Name is required"
        });
      }

      if (!email) {
        return res.status(400).json({
          error: "Email is required"
        });
      }

      res.status(201).json({
        message: "User created",
        data: { name, email }
      });
    });

---

### 6. What This Code Does

- Reads JSON body safely
- Checks required fields
- Returns errors if invalid
- Sends success response if valid

---

### 7. Testing Valid Input

    curl -X POST http://localhost:3000/users \
    -H "Content-Type: application/json" \
    -d '{"name":"Alice","email":"alice@test.com"}'

Response:

{
  "message": "User created",
  "data": {
    "name": "Alice",
    "email": "alice@test.com"
  }
}

---

### 8. Testing Missing Fields

    -d '{"name":"Alice"}'

Response:

{
  "error": "Email is required"
}

---

## Validating Data Types (Very Important)

---

### 9. Why Field Presence Is Not Enough

Client may send:

{
  "name": 12345
}

Presence check passes.

But data type is wrong.

---

### 10. Type Validation Example

    if (typeof name !== "string") {
      return res.status(400).json({
        error: "Name must be a string"
      });
    }

---

### 11. Full Improved Validation

    app.post("/users", (req, res) => {
      const { name, email } = req.body || {};

      if (!name) {
        return res.status(400).json({ error: "Name is required" });
      }

      if (typeof name !== "string") {
        return res.status(400).json({ error: "Name must be a string" });
      }

      if (!email) {
        return res.status(400).json({ error: "Email is required" });
      }

      if (typeof email !== "string") {
        return res.status(400).json({ error: "Email must be a string" });
      }

      res.status(201).json({
        message: "User created",
        data: { name, email }
      });
    });

---

## Validating Business Rules

---

### 12. Beyond Types — Validating Values

Correct type does NOT guarantee correctness.

Example:

{
  "email": "not-an-email"
}

Still a string.

Still invalid.

---

### 13. Example Business Rule Validation

    if (!email.includes("@")) {
      return res.status(400).json({
        error: "Invalid email format"
      });
    }

---

## Advanced Validation Patterns

---

### 14. Early Return Strategy (Professional Pattern)

    if (!name || typeof name !== "string") {
      return res.status(400).json({ error: "Invalid name" });
    }

Keeps logic clean.

---

### 15. Validation Explosion Problem

Manual validation becomes repetitive:

❌ Hard to maintain  
❌ Easy to forget checks  
❌ Poor scalability  

Real applications use validation libraries.

---

## Schema Validation (Production Best Practice)

---

### 16. Why Schema Validation Is Better

Libraries like:

- Zod
- Joi
- Yup
- express-validator

Provide:

✅ Centralized rules  
✅ Cleaner code  
✅ Better errors  
✅ Stronger guarantees  

---

### 17. Example Conceptual Schema Validation

    const result = schema.safeParse(req.body);

    if (!result.success) {
      return res.status(400).json(result.error);
    }

Cleaner & scalable.

---

## Proper Status Codes (Very Important)

---

### 18. Correct Status Code Usage

| Scenario | Status Code |
|----------|-------------|
| Invalid Input | 400 Bad Request |
| Unauthorized | 401 |
| Forbidden | 403 |
| Not Found | 404 |
| Validation Error (semantic) | 422 |
| Created Successfully | 201 |

Choose accurately.

---

### 19. 400 vs 422 (Advanced Insight)

- **400** → Syntax / structural problems
- **422** → Valid JSON but invalid business rules

Example:

Valid JSON but invalid values → 422

---

## Security Considerations

---

### 20. Never Trust Client Input

Even validated fields may contain:

❌ Malicious strings  
❌ Injection attempts  

Always sanitize & validate deeper in real apps.

---

## Common Beginner Mistakes

---

### 21. Typical Errors

❌ Forgetting JSON parser  
❌ Blindly trusting req.body  
❌ Checking only field presence  
❌ Ignoring data types  
❌ No validation at all  
❌ Wrong status codes  
❌ Sending inconsistent error formats  
❌ Overcomplicated validation  

---

## Professional Best Practices

---

### 22. Production Guidelines

✅ Always parse JSON bodies  
✅ Always validate required fields  
✅ Always validate data types  
✅ Always validate business rules  
✅ Use early returns  
✅ Return meaningful error messages  
✅ Use consistent error structure  
✅ Use correct status codes  
✅ Prefer schema validation for real apps  
✅ Never trust client input blindly  

---

### 23. Final Mental Model

POST endpoint = Data entry point

Everything entering your system must be:

Validated  
Checked  
Controlled  

Validation prevents bugs, crashes, and security issues.
