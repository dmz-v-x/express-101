## Authentication Middleware Pattern in Express

## Understanding the Big Picture

---

### 1. First: What Problem Are We Solving?

In real applications:

❌ Not all routes should be public

Examples:

✔ /login → Public  
✔ /profile → Protected  
✔ /admin → Restricted  

We need a reusable mechanism.

---

### 2. Naive Beginner Approach (Bad Pattern)

Beginners often write:

    app.get("/profile", (req, res) => {
      const token = req.headers.authorization;

      if (!token) {
        return res.status(401).send("Unauthorized");
      }

      // verify token logic here
      // business logic here
    });

Problem:

❌ Auth logic duplicated  
❌ Messy controllers  
❌ Hard to maintain  

---

## The Core Principle (Memorize This)

---

### 3. Authentication Is a Cross-Cutting Concern

Meaning:

> Many routes need the same identity check

Solution:

✅ Middleware

---

## What Is Authentication Middleware?

---

### 4. Definition

Authentication middleware is:

> A function that verifies user identity before route logic runs

---

### 5. Critical Mental Model

Request → Auth Middleware → Route Handler

---

## Hands-On Example — Basic Auth Middleware

---

### 6. Minimal Middleware Example

    const authMiddleware = (req, res, next) => {
      const header = req.headers.authorization;

      if (!header) {
        return res.status(401).json({ error: "No token provided" });
      }

      next();
    };

---

### 7. Applying Middleware

    app.get("/profile", authMiddleware, (req, res) => {
      res.send("Protected data");
    });

---

### 8. What Just Happened?

Before controller runs:

✅ Middleware executes

---

## Real Authentication Middleware (JWT Example)

---

### 9. Proper Token Verification

    const jwt = require("jsonwebtoken");

    const authMiddleware = (req, res, next) => {
      const header = req.headers.authorization;

      if (!header) {
        return res.status(401).json({ error: "No token provided" });
      }

      const token = header.split(" ")[1];

      try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);

        req.user = decoded;

        next();

      } catch (err) {
        res.status(401).json({ error: "Invalid token" });
      }
    };

---

### 10. Critical Responsibilities

Middleware:

✔ Extract token  
✔ Verify token  
✔ Attach user to req  
✔ Pass control  

---

## Why Attach Data to req?

---

### 11. Critical Pattern

    req.user = decoded;

Allows downstream logic to access identity.

---

### 12. Example Usage

    app.get("/profile", authMiddleware, (req, res) => {
      res.json({ userId: req.user.userId });
    });

---

## Advanced Middleware Pattern — Reusability

---

### 13. Middleware Should Be Generic

Bad:

❌ Route-specific middleware

Good:

✅ Reusable middleware

---

## Advanced Pattern — Role-Based Middleware

---

### 14. Authorization Middleware Factory

    const requireRole = (role) => {
      return (req, res, next) => {
        if (req.user.role !== role) {
          return res.status(403).json({ error: "Forbidden" });
        }

        next();
      };
    };

---

### 15. Usage Example

    app.delete(
      "/users/:id",
      authMiddleware,
      requireRole("admin"),
      controller.deleteUser
    );

---

## Why Separate Authentication & Authorization?

---

### 16. Critical Architecture Principle

Authentication:

✔ Who are you?

Authorization:

✔ What can you do?

---

## Middleware Execution Flow (Important)

---

### 17. Flow Example

Request arrives  
   ↓  
Auth middleware runs  
   ↓  
User verified  
   ↓  
Route handler runs  
   ↓  
Response  

---

## Advanced Production Patterns

---

## Pattern 1 — Centralized Middleware

---

### 18. Global Application

    app.use(authMiddleware);

Protects all routes.

⚠ Rarely used directly.

---

## Pattern 2 — Router-Level Middleware

---

### 19. Example

    router.use(authMiddleware);

Protects entire module.

---

## Pattern 3 — Optional Authentication

---

### 20. Example

Some routes behave differently:

✔ Logged-in users → Personalized data  
✔ Guests → Generic data

---

### 21. Optional Middleware Example

    const optionalAuth = (req, res, next) => {
      const header = req.headers.authorization;

      if (!header) return next();

      try {
        const token = header.split(" ")[1];
        req.user = jwt.verify(token, process.env.JWT_SECRET);
      } catch {}

      next();
    };

---

## Pattern 4 — Async Middleware (Very Important)

---

### 22. Async Verification Example

    const authMiddleware = async (req, res, next) => {
      try {
        const token = extractToken(req);
        const user = await findUserFromToken(token);

        req.user = user;

        next();

      } catch (err) {
        res.status(401).json({ error: "Unauthorized" });
      }
    };

---

## Common Middleware Mistakes

---

### 23. Typical Errors

❌ Forgetting next()  
❌ Sending multiple responses  
❌ No return after res.send  
❌ Blocking operations  
❌ Duplicated logic  
❌ Trusting decoded payload blindly  
❌ Heavy logic inside middleware  
❌ Middleware doing business logic  
❌ Wrong status codes  
❌ Silent failures  

---

## Professional Middleware Best Practices

---

### 24. Production-Grade Guidelines

✅ Keep middleware focused  
✅ Only handle identity logic  
✅ Avoid business logic inside middleware  
✅ Always call next()  
✅ Always return after response  
✅ Attach identity to req  
✅ Use reusable middleware  
✅ Separate auth & authorization  
✅ Fail securely  
✅ Use correct status codes  
✅ Log failures  
✅ Avoid blocking operations  
✅ Keep middleware lightweight  

---

## Advanced Mental Model

Middleware:

Traffic checkpoint

Controller:

Business logic

Middleware decides:

✔ Can request proceed?

---

## Final Mental Model

Authentication middleware:

✔ Verifies identity  
✔ Attaches user context  
✔ Protects routes  
✔ Improves architecture  
✔ Reduces duplication  
