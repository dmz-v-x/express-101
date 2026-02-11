## Dependency Injection (Manual DI) in Express Applications

## Understanding the Big Picture

---

### 1. First: What Is a Dependency?

A dependency is:

> Anything your code needs to function

Examples:

- Database connection
- Services
- Repositories
- Logger
- Cache
- External APIs

Example:

    const repo = require("./user.repository");

Here:

repo = Dependency

---

### 2. The Hidden Problem Most Beginners Miss

Typical code:

    const repo = require("./repo");

Dependency is:

❌ Hardcoded  
❌ Tightly coupled  
❌ Hard to replace  
❌ Hard to test  

---

### 3. What Is Dependency Injection?

Dependency Injection means:

> Provide dependencies from outside instead of creating them inside

Think:

❌ "I will create what I need"  
✅ "Someone gives me what I need"

---

### 4. Simple Mental Model

Without DI:

Controller → Directly imports repository

With DI:

Controller → Receives repository

---

## Hands-On Example — Without DI (Tight Coupling)

---

### 5. Service Without DI

`user.service.js`

    const repo = require("../repositories/user.repository");

    exports.getUsers = () => {
      return repo.findAll();
    };

Problem:

❌ Repo hardwired  
❌ Cannot swap easily  
❌ Testing becomes painful  

---

### 6. Why This Becomes a Real Problem

Want to:

- Use mock repo in tests
- Switch database
- Replace logger
- Add caching layer

Hard because dependency is locked.

---

## Introducing Manual Dependency Injection

---

### 7. Core Idea of Manual DI

Instead of:

    require dependency inside file

We:

✔ Pass dependency as parameter

---

## Step 1 — Service Using DI

---

### 8. Service Refactored for DI

    exports.createUserService = (repo) => {
      return {
        getUsers: () => repo.findAll(),

        createUser: (name) => {
          if (!name) throw new Error("Name required");
          return repo.create({ name });
        }
      };
    };

Key change:

repo is injected.

---

## Step 2 — Repository (Unchanged)

---

### 9. Repository Example

    let users = [];

    exports.findAll = () => users;

    exports.create = (userData) => {
      const user = { id: Date.now(), ...userData };
      users.push(user);
      return user;
    };

Repository stays simple.

---

## Step 3 — Injecting Dependencies at Setup

---

### 10. Application Wiring (Very Important)

`index.js`

    const express = require("express");
    const userRepo = require("./repositories/user.repository");
    const { createUserService } = require("./services/user.service");

    const app = express();

    const userService = createUserService(userRepo);

---

### 11. What Just Happened?

Instead of service importing repo:

✔ Repo passed into service  
✔ Dependency injected manually  

---

## Step 4 — Controller Using Injected Service

---

### 12. Controller with DI

    exports.createUserController = (service) => {
      return {
        getUsers: (req, res) => {
          const users = service.getUsers();
          res.json(users);
        },

        createUser: (req, res) => {
          const user = service.createUser(req.body.name);
          res.status(201).json(user);
        }
      };
    };

Controller receives service.

---

## Step 5 — Final Wiring

---

### 13. Full Composition

    const { createUserController } = require("./controllers/user.controller");

    const userController = createUserController(userService);

    app.get("/users", userController.getUsers);
    app.post("/users", userController.createUser);

---

## Why Dependency Injection Is Extremely Powerful

---

### 14. Loose Coupling

Components no longer depend directly on:

❌ Concrete implementations  

They depend on:

✅ Provided dependencies

---

### 15. Easy Testing (Huge Benefit)

Inject mock repository:

    const mockRepo = {
      findAll: () => [{ id: 1, name: "Test" }]
    };

    const service = createUserService(mockRepo);

Testing becomes simple.

---

### 16. Easy Dependency Replacement

Swap DB:

✔ MongoDB repo  
✔ PostgreSQL repo  
✔ In-memory repo  

Only wiring changes.

---

### 17. Better Separation of Concerns

Setup logic lives in:

✅ Composition root (index.js)

Business logic lives in:

✅ Services

---

### 18. Improved Flexibility

Can inject:

- Logger
- Cache
- Metrics
- Feature flags
- External APIs

---

## Advanced Real-World Example

---

### 19. Injecting Multiple Dependencies

    exports.createUserService = ({ repo, logger }) => {
      return {
        getUsers: () => {
          logger.info("Fetching users");
          return repo.findAll();
        }
      };
    };

Cleaner scaling.

---

### 20. Why Large Systems Depend Heavily on DI

Without DI:

❌ Tight coupling explosion  
❌ Testing nightmares  
❌ Hard refactoring  

With DI:

✅ Modular systems  
✅ Replaceable components  

---

## Important Clarification

---

### 21. DI Does NOT Require Fancy Libraries

Manual DI is:

✔ Simple  
✔ Explicit  
✔ Powerful  

Frameworks like NestJS automate this, but Express works perfectly with manual DI.

---

## Common Beginner Mistakes

---

### 22. Typical Errors

❌ Thinking DI is "too advanced"  
❌ Overcomplicating DI  
❌ Injecting everything blindly  
❌ Mixing DI with global variables  
❌ Confusing DI with middleware  
❌ Incorrect wiring order  
❌ Massive dependency objects  
❌ No composition root  

---

## Professional Best Practices

---

### 23. Production-Grade Guidelines

✅ Use DI for replaceable components  
✅ Keep wiring in entry file  
✅ Inject dependencies explicitly  
✅ Avoid hidden imports  
✅ Keep dependencies minimal  
✅ Use DI heavily in services  
✅ Simplify testing via DI  
✅ Avoid over-injection  
✅ Maintain consistency  
✅ Treat DI as architecture tool  

---

## Final Mental Model

Without DI:

Components create their own dependencies  
→ Tight coupling

With DI:

Dependencies provided externally  
→ Loose coupling

Without DI → Rigid system  
With DI → Modular & scalable system
