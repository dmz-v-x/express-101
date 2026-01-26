## Multiple Route Handler in Experss

### 1. First: What Is a Route Handler?

A **route handler** is simply:

> A function that runs when a request matches a route.

Example (single handler):

    app.get("/hello", (req, res) => {
      res.send("Hello");
    });

Here:

- `/hello` → route
- `(req, res) => {}` → route handler

So far, one route → one handler.

---

### 2. The Big Idea: What Are Multiple Route Handlers?

**Multiple route handlers** means:

> More than one function runs **for the same route**, in order.

Instead of one function:

    app.get("/users", handler);

You can have:

    app.get("/users", handler1, handler2, handler3);

Each handler runs **one after another**.

---

### 3. Why Would We Need Multiple Handlers?

Real apps need to do steps like:

- Check authentication
- Validate input
- Fetch data
- Send response

Putting everything in one function becomes messy.

So we split logic into **small steps**.

---

### 4. The Key Mechanism: `next()` 

Express moves from one handler to the next using:

    next()

Mental model:

- Handler runs
- Calls `next()`
- Express moves to the next handler

If `next()` is NOT called:

- Execution stops
- No further handlers run

---

### 5. Your First Multiple Route Handler Example

    app.get(
      "/test",
      (req, res, next) => {
        console.log("Handler 1");
        next();
      },
      (req, res) => {
        res.send("Handler 2");
      }
    );

Request:

- GET /test

Execution order:

- Handler 1 runs
- `next()` called
- Handler 2 runs
- Response sent

Output:

    Handler 2

Terminal shows:

    Handler 1

---

### 6. Why This Works

Express treats this as:

- One route
- Multiple handlers
- Executed **left to right**

Only the **last handler** usually sends the response.

---

### 7. What Happens If You Forget `next()`?

Example (BUG):

    app.get(
      "/test",
      (req, res, next) => {
        console.log("Handler 1");
        // next() missing
      },
      (req, res) => {
        res.send("Handler 2");
      }
    );

What happens:

- Handler 1 runs
- Handler 2 NEVER runs
- No response sent
- Request hangs forever

❌ This violates the one-request-one-response rule.

---

### 8. Important Rule 

For multiple route handlers:

- Every handler must either:
  - Call `next()`, OR
  - Send a response

Never do neither.

---

### 9. Using Multiple Handlers for Validation 

Example:

    const validateUser = (req, res, next) => {
      if (!req.query.id) {
        return res.status(400).json({ error: "ID required" });
      }
      next();
    };

    const getUser = (req, res) => {
      res.json({ id: req.query.id });
    };

    app.get("/user", validateUser, getUser);

Flow:

- validateUser runs
- If invalid → response sent → STOP
- If valid → `next()` → getUser runs

---

### 10. This Is NOT Special — It’s Just Functions

Multiple route handlers are just:

- Normal JavaScript functions
- Passed as arguments
- Executed in order

No magic.

---

### 11. Multiple Handlers vs Middleware 

They are conceptually the same.

Difference:

| Feature | Middleware | Multiple Route Handlers |
|-------|-----------|-------------------------|
| Scope | Many routes | One route |
| Defined with | app.use | app.get / post |
| Uses next() | Yes | Yes |

Route handlers are **route-specific middleware**.

---

### 12. Advanced: Splitting Logic Cleanly

Professional style:

    const auth = (req, res, next) => {
      console.log("Auth checked");
      next();
    };

    const authorize = (req, res, next) => {
      console.log("Permissions checked");
      next();
    };

    const controller = (req, res) => {
      res.json({ message: "Success" });
    };

    app.get("/secure", auth, authorize, controller);

This reads like English:

- Authenticate
- Authorize
- Respond

---

### 13. Advanced: Reusing Handlers Across Routes

    const logger = (req, res, next) => {
      console.log(req.method, req.url);
      next();
    };

    app.get("/users", logger, handler);
    app.post("/users", logger, handler);

Same handler reused safely.

---

### 14. Error Handling with Multiple Handlers 

If you call:

    next(err);

Express skips normal handlers and goes to **error handlers**.

Error handlers have this signature:

    (err, req, res, next)

This is advanced, but important later.

---

### 15. Common Beginner Mistakes 

- Forgetting `next()`
- Sending a response AND calling `next()`
- Sending responses in multiple handlers
- Overcomplicating simple routes

Rule:

> One handler sends the response  
> Others prepare the data

---

### 16. Best Practices 

- Keep handlers small
- One responsibility per handler
- Final handler sends response
- Reuse handlers
- Use clear names

---

### 17. Final Mental Model 

Say this slowly:

Multiple route handlers  
run one after another  
using `next()`  

Only one sends the response.


