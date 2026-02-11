### Building Logger Middleware + Error Handler in Express — Hands-On

### PART 1 — The Big Picture

---

### 1. What Are We Trying to Achieve?

Real applications ALWAYS need:

✅ Logging → to monitor requests  
✅ Error handling → to prevent crashes  

Without these:

❌ Debugging becomes painful  
❌ Errors crash servers  
❌ Production becomes unstable  

We’ll build both properly.

---

### PART 2 — Logger Middleware

---

### 2. What Should a Good Logger Capture?

A useful logger records:

- HTTP method
- URL
- Status code
- Response time

Professional logs look like:

GET /users 200 12ms

---

### 3. Writing the Logger Middleware

    const logger = (req, res, next) => {
      const start = Date.now();

      res.on("finish", () => {
        const duration = Date.now() - start;

        console.log(
          req.method,
          req.originalUrl,
          res.statusCode,
          `${duration}ms`
        );
      });

      next();
    };

---

### 4. Understanding This (Beginner-Friendly)

---

### `Date.now()`

Stores request start time.

---

### `res.on("finish")`

Runs AFTER response is sent.

Why important?

👉 Now we know:

- Final status code
- Total time taken

---

### `req.originalUrl`

Full request URL.

---

### `res.statusCode`

Actual result of request.

---

### PART 3 — Error Handling Middleware

---

### 5. Why Centralized Error Handling?

Without it:

❌ Errors crash server  
❌ Inconsistent responses  
❌ Repetitive try/catch  

We need ONE place to manage failures.

---

### 6. Writing the Error Middleware

    const errorHandler = (err, req, res, next) => {
      console.error("🔥 ERROR:", err.message);

      res.status(err.status || 500).json({
        success: false,
        error: err.message || "Internal Server Error"
      });
    };

---

### 7. Critical Rule 🚨 (Memorize This)

Error middleware MUST have:

    (err, req, res, next)

FOUR parameters.

Otherwise Express ignores it ❌

---

### PART 4 — Full Working Demo Server

---

### 8. Complete Example (Copy-Paste Ready)

    const express = require("express");
    const app = express();

    app.use(express.json());

    // LOGGER MIDDLEWARE
    const logger = (req, res, next) => {
      const start = Date.now();

      res.on("finish", () => {
        const duration = Date.now() - start;

        console.log(
          req.method,
          req.originalUrl,
          res.statusCode,
          `${duration}ms`
        );
      });

      next();
    };

    // ERROR HANDLER
    const errorHandler = (err, req, res, next) => {
      console.error("🔥 ERROR:", err.message);

      res.status(err.status || 500).json({
        success: false,
        error: err.message || "Internal Server Error"
      });
    };

    // APPLY LOGGER GLOBALLY
    app.use(logger);

    // ROUTES
    app.get("/", (req, res) => {
      res.send("Home page");
    });

    app.get("/error", (req, res) => {
      throw new Error("Something exploded!");
    });

    app.get("/manual-error", (req, res, next) => {
      next(new Error("Manual failure"));
    });

    // ERROR MIDDLEWARE (MUST BE LAST)
    app.use(errorHandler);

    app.listen(3000, () => {
      console.log("Server running on port 3000");
    });

---

### PART 5 — Testing the System

---

### 9. Normal Request → GET /

Response:

Home page

Terminal log:

GET / 200 3ms

---

### 10. Error Request → GET /error

Response:

    {
      "success": false,
      "error": "Something exploded!"
    }

Terminal:

🔥 ERROR: Something exploded!  
GET /error 500 2ms

---

### 11. Manual Error → GET /manual-error

Same flow ✅

---

### PART 6 — Why This Design Is Professional

---

### 12. Logger Uses Response Lifecycle

Instead of logging early:

❌ Before knowing status code  

We log AFTER response.

Much more accurate ✅

---

### 13. Centralized Error Handling

Routes stay clean:

    throw new Error("Boom");

Instead of repetitive logic.

---

### 14. Consistent Error Responses

All errors → Same JSON format

Very important for APIs ✅

---

### PART 7 — Advanced Improvements (Production-Level Thinking)

---

### 15. Structured Logs (Better Format)

    console.log({
      method: req.method,
      url: req.originalUrl,
      status: res.statusCode,
      duration
    });

Useful for log aggregators.

---

### 16. Log Levels (Advanced Concept)

Production loggers use:

- INFO
- WARN
- ERROR

Instead of raw console.log.

---

### 17. Custom Error Classes (Very Powerful)

Example:

    class AppError extends Error {
      constructor(message, status) {
        super(message);
        this.status = status;
      }
    }

Then:

    next(new AppError("Invalid input", 400));

---

### 18. Avoid Stack Traces in Production ⚠️

Good practice:

- Show clean errors to users
- Log full details internally

---

### PART 8 — Common Beginner Mistakes 🚫

---

❌ Forgetting `next()` in logger  
❌ Putting error handler before routes  
❌ Missing 4 parameters  
❌ Sending multiple responses  
❌ Logging sensitive data  
❌ Heavy logic in middleware  

---

### FINAL MENTAL MODEL 

Say this slowly:

Logger middleware → observes requests  
Error middleware → handles failures  

Both are core pillars of Express apps.
