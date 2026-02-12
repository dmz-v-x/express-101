## Role-Based Access Control (RBAC) in Express

## Understanding the Big Picture

---

### 1. First: What Problem Does RBAC Solve?

In real systems:

❌ Not every user can perform every action

Examples:

✔ Normal user → View profile  
✔ Admin → Delete users  
✔ Moderator → Manage content  

We need structured permission control.

---

### 2. What Is RBAC?

RBAC means:

> Permissions are assigned based on roles

Think:

Role → Collection of permissions

User → Assigned role

---

### 3. Critical Mental Model

User → Has Role → Role Grants Permissions

---

## Basic Roles Example

---

### 4. Example Roles

Common roles:

- user
- admin
- moderator

---

### 5. Example Permissions

user:

✔ View own data

admin:

✔ View all users  
✔ Delete users  
✔ Modify system  

moderator:

✔ Manage posts  

---

## Authentication vs Authorization (Critical Separation)

---

### 6. Critical Principle

Authentication:

✔ Who are you?

Authorization (RBAC):

✔ What are you allowed to do?

RBAC is **authorization logic**.

---

## Hands-On Setup — Identity Middleware

---

### 7. Assume JWT Auth Middleware

Middleware attaches:

    req.user = {
      userId: 1,
      role: "admin"
    };

---

## Naive Beginner Approach (Bad Pattern)

---

### 8. Hardcoding Inside Routes

    app.delete("/users/:id", (req, res) => {
      if (req.user.role !== "admin") {
        return res.status(403).send("Forbidden");
      }

      // delete logic
    });

Problem:

❌ Duplicated logic  
❌ Messy controllers  

---

## Proper RBAC Middleware Pattern

---

### 9. Basic Role Middleware

    const requireRole = (role) => {
      return (req, res, next) => {
        if (req.user.role !== role) {
          return res.status(403).json({ error: "Forbidden" });
        }

        next();
      };
    };

---

### 10. Usage Example

    app.delete(
      "/users/:id",
      authMiddleware,
      requireRole("admin"),
      controller.deleteUser
    );

---

### 11. What Just Happened?

Request → Auth → Role Check → Controller

Clean separation.

---

## Supporting Multiple Roles

---

### 12. Advanced Middleware

    const requireRoles = (...allowedRoles) => {
      return (req, res, next) => {
        if (!allowedRoles.includes(req.user.role)) {
          return res.status(403).json({ error: "Forbidden" });
        }

        next();
      };
    };

---

### 13. Example Usage

    app.get(
      "/dashboard",
      authMiddleware,
      requireRoles("admin", "moderator"),
      controller.dashboard
    );

---

## Why Middleware Pattern Is Critical

---

### 14. Architectural Benefits

✅ Reusable  
✅ Clean controllers  
✅ Centralized logic  
✅ Easy to maintain  

---

## Advanced RBAC — Permission-Based Variant

---

### 15. Role Limitation Problem

Roles can become:

❌ Too rigid  
❌ Hard to scale  

Alternative:

Permissions.

---

### 16. Permission-Based Middleware

    const requirePermission = (permission) => {
      return (req, res, next) => {
        if (!req.user.permissions.includes(permission)) {
          return res.status(403).json({ error: "Forbidden" });
        }

        next();
      };
    };

---

### 17. Example Identity Object

    req.user = {
      userId: 1,
      role: "admin",
      permissions: ["delete_user", "edit_user"]
    };

---

## Hybrid RBAC (Production Pattern)

---

### 18. Common Real-World Design

User → Role → Permissions

Role defines permissions.

More flexible.

---

## Database Representation (Important Insight)

---

### 19. Typical Tables

Users  
Roles  
Permissions  
RolePermissions  

---

### 20. Why DB-Driven RBAC Matters

✅ Dynamic control  
✅ No redeployment  
✅ Scalable  

---

## Advanced Pattern — Ownership + RBAC

---

### 21. Real-World Rule Example

User can edit:

✔ Own profile  
❌ Others’ profiles  

Admin:

✔ Edit any profile  

---

### 22. Combined Check Example

    if (
      req.user.role !== "admin" &&
      req.user.userId !== resource.ownerId
    ) {
      return res.status(403).json({ error: "Forbidden" });
    }

---

## Common RBAC Mistakes

---

### 23. Typical Errors

❌ Mixing auth & RBAC logic  
❌ Hardcoding everywhere  
❌ No middleware abstraction  
❌ Trusting client role data  
❌ Incorrect status codes  
❌ Too many roles  
❌ No permission granularity  
❌ Business logic inside RBAC  
❌ No ownership checks  
❌ Inconsistent role definitions  

---

## Professional RBAC Best Practices

---

### 24. Production-Grade Guidelines

✅ Keep RBAC logic centralized  
✅ Use middleware  
✅ Separate authentication & authorization  
✅ Use correct status codes (403)  
✅ Prefer permission-based systems for large apps  
✅ Use DB-driven roles/permissions  
✅ Avoid role explosion  
✅ Combine RBAC + ownership rules  
✅ Never trust client-sent roles  
✅ Keep authorization lightweight  
✅ Log access denials  
✅ Document permission model  

---

## Advanced Mental Model

RBAC middleware:

Security gatekeeper

Controller:

Business logic

---

## Final Mental Model (Memorize This)

Authentication:

✔ Establish identity

RBAC:

✔ Enforce permissions

Identity ≠ Permission
