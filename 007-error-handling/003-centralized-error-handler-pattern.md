## Centralized Error Handler Pattern in Express

### 1. First: The Problem With Scattered Error Handling

Beginners often handle errors like this:

    app.get("/users/:id", async (req, res) => {
      try {
        const user = await findUser();
        if (!user) {
          return res.status(404).json({ error: "Not found" });
        }
        res.json(user);
      } catch (err) {
        res.status(500).json({ error: "Database failed" });
      }
    });

This works.

But at scale:

❌ Repetitive  
❌ Inconsistent responses  
❌ Hard to maintain  
❌ Hard to debug  
❌ Logic duplication  

We need structure.

---

### 2. What Is a Centralized Error Handler?

A centralized error handler is:

> One place where ALL errors are processed

Instead of:

Handling errors inside every route.

We delegate error logic.

---

### 3. Why Centralization Is Critical

Centralized error handling provides:

✅ Consistent responses  
✅ Cleaner route handlers  
✅ Easier debugging  
✅ Better logging  
✅ Better scalability  
✅ Separation of concerns  

---

## Understanding Express Error Flow

---

### 4. How Errors Travel in Express

Error occurs  
↓  
Passed using `next(err)`  
↓  
Express skips normal middleware  
↓  
Error middleware runs  

Important:

Express has a built-in error pipeline.

---

### 5. What Makes Middleware an Error Handler?

Error middleware has **four parameters**:

    (err, req, res, next)

This signature is mandatory.

---

## Minimal Centralized Error Handler

---

### 6. Basic Server Setup

    const express = require("express");
    const app = express();

---

### 7. Example Route That Throws Errors

    app.get("/error-demo", (req, res, next) => {
      try {
        throw new Error("Something failed");
      } catch (err) {
        next(err);
      }
    });

---

### 8. Centralized Error Handler

    app.use((err, req, res, next) => {
      res.status(500).json({
        error: err.message
      });
    });

---

### 9. What Happens Now

Any forwarded error → One handler

Cleaner & predictable.

---

## Why `next(err)` Is the Key

---

### 10. Important Rule

> Routes should NOT decide how errors are formatted

Routes detect problems.

Error handler decides response.

---

### 11. Clean Route Example

    app.get("/users/:id", async (req, res, next) => {
      try {
        const user = await findUser();

        if (!user) {
          const err = new Error("User not found");
          err.statusCode = 404;
          throw err;
        }

        res.json(user);

      } catch (err) {
        next(err);
      }
    });

Route stays clean.

---

## Advanced Centralized Error Pattern

---

### 12. Problem With Basic Handler

Naive handler:

    res.status(500)

Treats all errors as server failures.

Not accurate.

---

### 13. Smart Error Handler

    app.use((err, req, res, next) => {
      const status = err.statusCode || 500;

      res.status(status).json({
        error: err.message || "Internal server error"
      });
    });

Now supports dynamic status codes.

---

### 14. Why This Is Powerful

Routes define error meaning.

Handler defines error format.

Perfect separation.

---

## Error Classification Strategy

---

### 15. Typical Error Types

**Operational Errors**
- Invalid input
- Resource not found
- Unauthorized access

**Programmer Errors**
- Bugs
- Undefined variables
- Logic failures

Both flow through same handler.

---

### 16. Attaching Metadata to Errors

    const err = new Error("Invalid input");
    err.statusCode = 400;

Enables precise responses.

---

## Production-Grade Error Handler

---

### 17. Realistic Centralized Handler

    app.use((err, req, res, next) => {
      console.error(err);

      const status = err.statusCode || 500;

      res.status(status).json({
        success: false,
        error: {
          message: err.message || "Internal server error"
        }
      });
    });

Adds:

✅ Logging  
✅ Consistent structure  
✅ Debug visibility  

---

## Avoiding try/catch Repetition

---

### 18. Async Wrapper Utility

    const asyncHandler = (fn) => (req, res, next) =>
      Promise.resolve(fn(req, res, next)).catch(next);

Usage:

    app.get("/users", asyncHandler(async (req, res) => {
      const users = await getUsers();
      res.json(users);
    }));

Cleaner routes.

---

## Advanced Error Handling Features

---

### 19. Handling Known Error Types

    if (err.name === "ValidationError") {
      return res.status(400).json({ error: err.message });
    }

Supports granular control.

---

### 20. Preventing Information Leaks

Bad practice:

    res.json(err);

Why dangerous?

❌ Exposes stack traces  
❌ Leaks internals  

Always sanitize responses.

---

### 21. Environment-Based Error Detail (Advanced)

    const isDev = process.env.NODE_ENV === "development";

    res.json({
      message: err.message,
      stack: isDev ? err.stack : undefined
    });

Safer production behavior.

---

## Common Beginner Mistakes

---

### 22. Typical Errors

❌ Missing error middleware  
❌ Wrong middleware signature  
❌ Handling errors inside every route  
❌ Sending response AND calling next(err)  
❌ Treating all errors as 500  
❌ Leaking stack traces  
❌ Inconsistent error formats  

---

## Professional Best Practices

---

### 23. Production Guidelines

✅ Always use centralized error handler  
✅ Always forward errors using next(err)  
✅ Keep routes focused on business logic  
✅ Attach status codes to errors  
✅ Log errors centrally  
✅ Use consistent response structure  
✅ Avoid leaking internals  
✅ Differentiate dev vs prod responses  
✅ Use asyncHandler wrappers  
✅ Classify operational vs programmer errors  

---

## Final Mental Model

Routes detect problems.

Centralized handler decides:

- Status code
- Response format
- Logging
- Debugging behavior

Single responsibility principle.
