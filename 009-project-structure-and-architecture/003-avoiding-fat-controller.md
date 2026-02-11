## Avoiding Fat Controllers in Express Applications

## Understanding the Big Picture

---

### 1. First: What Is a Controller?

A controller is responsible for:

- Reading the request (`req`)
- Calling business logic
- Sending the response (`res`)

Simple role:

> Controller = HTTP handler

---

### 2. What Is a Fat Controller?

A fat controller is one that contains:

❌ Business logic  
❌ Validation rules  
❌ Database queries  
❌ Complex calculations  
❌ Decision-making logic  

Everything dumped into one place.

---

### 3. Why Fat Controllers Happen

Beginners often write:

- Everything inside route handlers
- Quick logic mixing
- No separation mindset

It feels easy at first.

---

## Hands-On Example — Fat Controller (Bad Pattern)

---

### 4. Example of a Fat Controller

    app.post("/users", async (req, res) => {
      if (!req.body.name) {
        return res.status(400).send("Name required");
      }

      const existingUser = await db.findUserByName(req.body.name);

      if (existingUser) {
        return res.status(409).send("User already exists");
      }

      const user = await db.createUser({
        name: req.body.name,
        createdAt: new Date(),
      });

      if (user.age < 18) {
        await sendWarningEmail(user);
      }

      res.status(201).json(user);
    });

---

### 5. Problems with This Code

❌ Validation logic inside controller  
❌ Database logic inside controller  
❌ Business rules inside controller  
❌ Side effects inside controller  
❌ Hard to read  
❌ Hard to test  
❌ Hard to reuse logic  

This controller is doing too much.

---

## The Core Principle (Memorize This)

---

### 6. Controller Should Be Thin

Controller should only:

✔ Read input  
✔ Call service  
✔ Send output  

Nothing else.

---

## Proper Layered Implementation

---

### 7. Clean Controller (Thin Version)

`controllers/user.controller.js`

    const service = require("../services/user.service");

    exports.createUser = async (req, res, next) => {
      try {
        const user = await service.createUser(req.body);
        res.status(201).json(user);
      } catch (err) {
        next(err);
      }
    };

Simple and clean.

---

## Moving Logic to Service Layer

---

### 8. Service Handles Business Logic

`services/user.service.js`

    const repo = require("../repositories/user.repository");

    exports.createUser = async (userData) => {
      if (!userData.name) {
        throw new Error("Name required");
      }

      const existingUser = await repo.findByName(userData.name);

      if (existingUser) {
        const err = new Error("User already exists");
        err.statusCode = 409;
        throw err;
      }

      const user = await repo.create({
        ...userData,
        createdAt: new Date(),
      });

      if (user.age < 18) {
        await sendWarningEmail(user);
      }

      return user;
    };

Now logic is properly placed.

---

## Repository Handles Database Access

---

### 9. Repository Example

`repositories/user.repository.js`

    exports.findByName = async (name) => {
      return db.findUserByName(name);
    };

    exports.create = async (userData) => {
      return db.createUser(userData);
    };

Controller no longer touches DB.

---

## Why Thin Controllers Are Critical

---

### 10. Readability Improves

Thin controller:

✅ Easy to scan  
✅ Easy to understand  
✅ No noise  

---

### 11. Reusability Increases

Service logic reusable by:

- Other controllers
- Background jobs
- Scheduled tasks
- CLI tools

Fat controller logic is trapped.

---

### 12. Testing Becomes Easier

Test business logic separately:

✅ Service tests  
✅ Controller tests  

Fat controllers require heavy mocks.

---

### 13. Debugging Becomes Easier

Bug location clarity:

HTTP issue → Controller  
Logic issue → Service  
DB issue → Repository  

Fat controllers blur boundaries.

---

### 14. Scaling Becomes Possible

Large systems demand:

✔ Clean separation  
✔ Predictable structure  

Fat controllers collapse under growth.

---

## Advanced Insight — The "Controller Smell"

---

### 15. Warning Signs of Fat Controllers

If your controller contains:

❌ Complex if/else logic  
❌ Calculations  
❌ Business rules  
❌ DB queries  
❌ External service calls  
❌ Data transformation logic  
❌ Multiple responsibilities  

You likely have a fat controller.

---

## Proper Responsibility Breakdown

---

### 16. Responsibility Mapping

Controller:

✔ Read req  
✔ Call service  
✔ Send res  

Service:

✔ Business rules  
✔ Validation  
✔ Decisions  
✔ Orchestration  

Repository:

✔ Data access  
✔ Queries  
✔ Persistence  

---

## Common Beginner Mistakes

---

### 17. Typical Errors

❌ Validation inside controller  
❌ DB queries inside controller  
❌ Business logic inside controller  
❌ Massive controller files  
❌ Duplicate logic across controllers  
❌ Hardcoded business rules  
❌ Mixing concerns  
❌ No service layer  
❌ Overloading controllers  

---

## Professional Best Practices

---

### 18. Production-Grade Guidelines

✅ Keep controllers thin  
✅ Move logic to services  
✅ Move DB logic to repositories  
✅ Controllers should orchestrate, not decide  
✅ Avoid business rules in HTTP layer  
✅ Reuse services  
✅ Maintain separation of concerns  
✅ Watch for controller bloat  
✅ Refactor early  
✅ Stay consistent  

---

## Final Mental Model

Controller = Traffic manager  
Service = Brain  
Repository = Data manager  

Controller coordinates.

Service decides.

Repository persists.

Fat controllers → Fragile systems  
Thin controllers → Scalable architecture
