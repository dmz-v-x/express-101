## Global Middleware vs Route Middleware in Express

## 1. First: What Is Middleware? 

Middleware is simply:

> A function that runs during the request → response lifecycle.

It sits between:

Request → Middleware → Route → Response

Middleware can:

- Read request (`req`)
- Modify request
- Modify response (`res`)
- Stop execution
- Pass control using `next()`

---

## 2. The Core Question of This Blog

Where should middleware run?

Two common choices:

1️⃣ **Global Middleware** → runs for ALL requests  
2️⃣ **Route Middleware** → runs for SPECIFIC routes

Understanding this difference is critical for real applications.

---

# PART 1 — Global Middleware

---

## 3. What Is Global Middleware?

Global middleware is:

> Middleware that runs for **every incoming request**

Defined using:

    app.use(middlewareFunction)

Think:

👉 “Run this logic for everything”

---

## 4. Your First Global Middleware

    app.use((req, res, next) => {
      console.log("Global middleware executed");
      next();
    });

Now EVERY request triggers this.

Examples:

- GET /
- GET /users
- POST /login
- DELETE /products/42

All pass through it.

---

## 5. Mental Model 

Request arrives  
→ Global middleware runs  
→ Route matching happens  
→ Route handler runs  

Global middleware = entry gate

---

## 6. Real-World Uses of Global Middleware

Global middleware is commonly used for:

✅ Logging  
✅ Body parsing  
✅ Security headers  
✅ CORS  
✅ Rate limiting  
✅ Authentication (sometimes)

---

## 7. Example: Logger Middleware 

    app.use((req, res, next) => {
      console.log(req.method, req.url);
      next();
    });

Now every request is logged.

---

# PART 2 — Route Middleware

---

## 8. What Is Route Middleware?

Route middleware is:

> Middleware that runs **only for specific routes**

Defined like this:

    app.get("/users", middleware, handler)

Think:

👉 “Run this logic only here”

---

## 9. Your First Route Middleware

    const authMiddleware = (req, res, next) => {
      if (!req.query.token) {
        return res.status(401).send("Unauthorized");
      }
      next();
    };

    app.get("/users", authMiddleware, (req, res) => {
      res.send("Users data");
    });

Now:

- `/users?token=abc` ✅ Works
- `/users` ❌ Blocked

Middleware runs ONLY for `/users`.

---

## 10. Mental Model

Request → Route match → Route middleware → Handler

Unlike global middleware:

- It runs AFTER route matching

---

## 11. Real-World Uses of Route Middleware

Route middleware is perfect for:

✅ Authentication  
✅ Authorization  
✅ Validation  
✅ Permissions  
✅ Special checks

---

# PART 3 — Seeing the Difference in Action 

---

## 12. Complete Demo Server

    const express = require("express");
    const app = express();

    // GLOBAL MIDDLEWARE
    app.use((req, res, next) => {
      console.log("🌍 Global middleware");
      next();
    });

    // ROUTE MIDDLEWARE
    const routeMiddleware = (req, res, next) => {
      console.log("🎯 Route middleware");
      next();
    };

    app.get("/", (req, res) => {
      res.send("Home page");
    });

    app.get("/users", routeMiddleware, (req, res) => {
      res.send("Users page");
    });

    app.listen(3000, () => {
      console.log("Server running on port 3000");
    });

---

## 13. What Happens for Different Requests

---

### Request 1 → GET /

Execution:

🌍 Global middleware  
→ Route handler  

Why?

- Global runs for everything
- No route middleware on `/`

---

### Request 2 → GET /users

Execution:

🌍 Global middleware  
→ 🎯 Route middleware  
→ Route handler  

Why?

- Global always runs
- Route middleware only runs for `/users`

---

# PART 4 — Clear Comparison 

---

## 14. Global vs Route Middleware

| Aspect | Global Middleware | Route Middleware |
|--------|------------------|------------------|
| Scope | All requests | Specific routes |
| Defined with | `app.use()` | `app.get/post()` |
| Runs when | Before route match | After route match |
| Common use | Logging, parsing | Auth, validation |
| Flexibility | Broad | Precise |

---

# PART 5 — Advanced Understanding

---

## 15. Middleware Execution Order 

Express runs middleware **top to bottom**.

Example:

    app.use(middleware1);
    app.use(middleware2);

Execution:

middleware1 → middleware2 → route

Order ALWAYS matters.

---

## 16. Combining Global + Route Middleware 

Professional pattern:

    app.use(globalSecurity);
    app.use(globalLogger);

    app.get("/admin", auth, authorize, handler);

Flow:

Global middleware → Route middleware → Handler

---

## 17. Performance Insight 

Global middleware runs for ALL requests.

Heavy global middleware:

❌ Slows entire app

Better approach:

👉 Keep global middleware lightweight  
👉 Use route middleware for expensive checks

---

## 18. Router-Level Middleware 

Inside routers:

    router.use(authMiddleware);

Now middleware runs only for router group.

Example:

- `/users/*` protected
- `/products/*` unaffected

Very powerful for large apps.

---

# PART 6 — Common Beginner Mistakes 

---

❌ Putting heavy logic in global middleware  
❌ Forgetting `next()`  
❌ Sending multiple responses  
❌ Confusing route middleware with handlers  
❌ Overusing middleware unnecessarily

---

# PART 7 — Best Practices

---

✅ Global middleware → shared logic  
✅ Route middleware → specific logic  
✅ Keep middleware small & focused  
✅ Respect execution order  
✅ Always call `next()` OR send response  

---

# FINAL MENTAL MODEL

Say this slowly:

Global middleware → runs everywhere  
Route middleware → runs only where attached  


