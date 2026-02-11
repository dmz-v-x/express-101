## Custom Error Classes in Express

### 1. First: The Problem With Generic Errors

Basic errors:

    throw new Error("Something failed");

Work fine.

But in real applications:

❌ No structured metadata  
❌ No clear classification  
❌ Hard to control status codes  
❌ Hard to standardize responses  

We need better error objects.

---

### 2. What Are Custom Error Classes?

Custom error classes are:

> Specialized error objects with extra information

They allow us to attach:

- Status codes
- Error types
- Operational flags
- Custom metadata

---

### 3. Why This Pattern Is Powerful

Custom errors provide:

✅ Clean architecture  
✅ Consistent responses  
✅ Better debugging  
✅ Better logging  
✅ Better scalability  
✅ Separation of concerns  

---

## Understanding JavaScript Error Inheritance

---

### 4. Errors Are Just Objects

In JavaScript:

    Error → Base class

We can extend it.

---

### 5. Minimal Custom Error Class

    class AppError extends Error {
      constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
      }
    }

Now errors carry extra data.

---

### 6. Why `super(message)` Is Required

`super()` calls the parent constructor.

Without it:

❌ Error object breaks

---

## Building a Production-Ready Error Class

---

### 7. Improved Custom Error

    class AppError extends Error {
      constructor(message, statusCode) {
        super(message);

        this.statusCode = statusCode;
        this.success = false;
        this.isOperational = true;

        Error.captureStackTrace(this, this.constructor);
      }
    }

Adds:

- Status code
- Operational flag
- Clean stack trace

---

### 8. Why `isOperational` Matters

Distinguishes:

Operational errors → Expected failures  
Programmer errors → Bugs  

Useful in advanced systems.

---

## Complete Hands-On Demo

---

### 9. Full Working Express Server

    const express = require("express");
    const app = express();

    app.use(express.json());

---

### 10. Custom Error Class

    class AppError extends Error {
      constructor(message, statusCode) {
        super(message);

        this.statusCode = statusCode;
        this.success = false;
        this.isOperational = true;

        Error.captureStackTrace(this, this.constructor);
      }
    }

---

### 11. Example Route — Validation Error

    app.post("/users", (req, res, next) => {
      const { name } = req.body || {};

      if (!name) {
        return next(new AppError("Name is required", 400));
      }

      res.status(201).json({
        success: true,
        data: { name }
      });
    });

---

### 12. Example Route — Not Found Error

    app.get("/users/:id", (req, res, next) => {
      const user = null;

      if (!user) {
        return next(new AppError("User not found", 404));
      }

      res.json(user);
    });

---

## Centralized Error Handler

---

### 13. Smart Error Middleware

    app.use((err, req, res, next) => {
      console.error(err);

      const statusCode = err.statusCode || 500;

      res.status(statusCode).json({
        success: false,
        error: {
          message: err.message || "Internal server error"
        }
      });
    });

---

### 14. Server Start

    app.listen(3000, () => {
      console.log("Server running on port 3000");
    });

---

## Testing the Behavior

---

### 15. Missing Name Field

    curl -X POST http://localhost:3000/users \
    -H "Content-Type: application/json" \
    -d '{}'

Response:

{
  "success": false,
  "error": {
    "message": "Name is required"
  }
}

---

### 16. Not Found Scenario

    curl http://localhost:3000/users/123

Response:

{
  "success": false,
  "error": {
    "message": "User not found"
  }
}

---

## Creating Specialized Error Types (Advanced Pattern)

---

### 17. Validation Error Class

    class ValidationError extends AppError {
      constructor(message) {
        super(message, 400);
      }
    }

---

### 18. Not Found Error Class

    class NotFoundError extends AppError {
      constructor(resource) {
        super(`${resource} not found`, 404);
      }
    }

---

### 19. Usage Example

    return next(new ValidationError("Invalid input"));
    return next(new NotFoundError("User"));

Cleaner & semantic.

---

## Advanced Error Handler Logic

---

### 20. Differentiating Operational Errors

    if (!err.isOperational) {
      console.error("Unexpected bug:", err);
    }

Useful in large production systems.

---

### 21. Environment-Based Responses

    const isDev = process.env.NODE_ENV === "development";

    res.json({
      message: err.message,
      stack: isDev ? err.stack : undefined
    });

Prevents leaking internals.

---

## Why This Pattern Scales Extremely Well

---

### 22. Benefits at Scale

✅ Consistent error structure  
✅ Clean route handlers  
✅ Easy debugging  
✅ Better logging  
✅ Semantic error types  
✅ Cleaner business logic  
✅ Easier maintenance  

---

## Common Beginner Mistakes

---

### 23. Typical Errors

❌ Forgetting `super()`  
❌ Not extending Error properly  
❌ Missing statusCode property  
❌ Sending response AND next(err)  
❌ Creating overly complex classes  
❌ Leaking raw errors  
❌ Skipping centralized handler  

---

## Professional Best Practices

---

### 24. Production Guidelines

✅ Use base AppError class  
✅ Extend for specific error types  
✅ Always attach status codes  
✅ Keep error structure consistent  
✅ Log errors centrally  
✅ Avoid leaking internals  
✅ Differentiate dev vs prod  
✅ Use semantic error names  
✅ Keep routes clean  
✅ Treat errors as part of architecture  

---

## Final Mental Model

Generic Error:

✔ Message only

Custom Error:

✔ Message  
✔ Status Code  
✔ Metadata  
✔ Classification  

Better control. Better structure.
