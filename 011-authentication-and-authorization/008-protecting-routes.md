## Protecting Routes in Express

## Understanding the Big Picture

---

### 1. First: What Does “Protecting a Route” Mean?

A protected route is:

> A route that cannot be accessed without proper checks

Examples:

✔ Logged-in users only  
✔ Admins only  
✔ Resource owners only  

---

### 2. Why Route Protection Is Critical

Without protection:

❌ Anyone can access private data  
❌ Anyone can modify resources  
❌ Severe security vulnerabilities  

---

### 3. Critical Mental Model

Public Route → No restrictions  
Protected Route → Requires verification

---

## Step 1 — The Basic Protection Mechanism

---

### 4. What Controls Access?

Access is controlled using:

✅ Middleware

Middleware acts as:

Security checkpoint

---

## Step 2 — Authentication Middleware

---

### 5. Basic Auth Middleware Example

    const authMiddleware = (req, res, next) => {
      const header = req.headers.authorization;

      if (!header) {
        return res.status(401).json({ error: "Unauthorized" });
      }

      next();
    };

---

### 6. Applying Route Protection

    app.get("/profile", authMiddleware, (req, res) => {
      res.send("Protected profile");
    });

---

### 7. Flow

Request → Auth Check → Controller

---

## Real-World Protection (JWT Example)

---

### 8. Proper Identity Verification

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

### 9. Protected Route Example

    app.get("/profile", authMiddleware, (req, res) => {
      res.json({ user: req.user });
    });

---

## Step 3 — Authorization Layer

---

### 10. Why Authentication Alone Is Not Enough

Authenticated user:

✔ Identity known

But still:

❌ May lack permission

---

## Role-Based Protection

---

### 11. Role Middleware Example

    const requireRole = (role) => {
      return (req, res, next) => {
        if (req.user.role !== role) {
          return res.status(403).json({ error: "Forbidden" });
        }

        next();
      };
    };

---

### 12. Admin-Only Route

    app.delete(
      "/users/:id",
      authMiddleware,
      requireRole("admin"),
      controller.deleteUser
    );

---

## Step 4 — Ownership-Based Protection

---

### 13. Real-World Scenario

User can edit:

✔ Own resources  
❌ Others’ resources  

---

### 14. Ownership Check Example

    app.put("/posts/:id", authMiddleware, (req, res) => {
      const post = findPost(req.params.id);

      if (!post) {
        return res.status(404).json({ error: "Not found" });
      }

      if (post.ownerId !== req.user.userId) {
        return res.status(403).json({ error: "Forbidden" });
      }

      res.json({ message: "Updated" });
    });

---

## Step 5 — Router-Level Protection (Scalable Pattern)

---

### 15. Protecting Route Groups

    const router = express.Router();

    router.use(authMiddleware);

    router.get("/profile", controller.profile);
    router.get("/settings", controller.settings);

    app.use("/user", router);

---

### 16. Benefit

✅ All routes protected automatically

---

## Step 6 — Optional Protection (Advanced Pattern)

---

### 17. Scenario

✔ Guests allowed  
✔ Logged-in users get enhanced data  

---

### 18. Optional Auth Middleware

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

## Step 7 — Common Route Protection Patterns

---

### 19. Typical Strategies

✅ Authentication-only  
✅ Role-based  
✅ Permission-based  
✅ Ownership-based  
✅ Hybrid protection  

---

## Critical HTTP Status Codes

---

### 20. Important Distinction

401 → Not authenticated  
403 → Not authorized  

---

## Common Beginner Mistakes

---

### 21. Typical Errors

❌ Protecting routes inside controllers  
❌ Duplicating auth logic  
❌ Missing middleware  
❌ Wrong status codes  
❌ Trusting client data  
❌ No ownership checks  
❌ Forgetting next()  
❌ Blocking operations  
❌ Middleware doing business logic  
❌ Sending multiple responses  

---

## Professional Best Practices

---

### 22. Production-Grade Guidelines

✅ Always use middleware for protection  
✅ Separate authentication & authorization  
✅ Attach identity to req  
✅ Keep middleware lightweight  
✅ Use correct status codes  
✅ Protect sensitive routes  
✅ Use router-level middleware for modules  
✅ Combine RBAC + ownership rules  
✅ Never trust client-sent roles  
✅ Validate resources exist  
✅ Fail securely  
✅ Log access denials  
✅ Document protection rules  

---

## Advanced Mental Model

Middleware:

Security gates

Route handler:

Business logic

---

## Final Mental Model

Protecting routes = Controlling access

Authentication:

✔ Who are you?

Authorization:

✔ What can you do?

Both required.
