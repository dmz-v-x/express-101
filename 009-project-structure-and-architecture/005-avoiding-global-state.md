## Avoiding Global State in Express Applications

## Understanding the Big Picture

---

### 1. First: What Is Global State?

Global state means:

> Data that is shared across the entire application implicitly

Example:

    let users = [];

Accessible from anywhere.

---

### 2. Why Beginners Accidentally Use Global State

It feels natural:

✅ Easy to write  
✅ Quick to access  
✅ No setup required  

Example:

    const users = [];

Used directly in controllers/services.

---

### 3. The Hidden Danger

Global state introduces:

❌ Unpredictable behavior  
❌ Hidden dependencies  
❌ Hard-to-debug bugs  
❌ Concurrency issues  
❌ Testing nightmares  
❌ Scaling problems  

---

## Hands-On Example — Global State Problem

---

### 4. Simple Global Variable Example

`user.service.js`

    let users = [];

    exports.createUser = (name) => {
      const user = { id: Date.now(), name };
      users.push(user);
      return user;
    };

Looks harmless.

---

### 5. Why This Becomes Dangerous

Problems appear when:

- Multiple requests hit simultaneously
- Tests run in parallel
- App scales across instances
- Memory resets
- Race conditions occur

---

### 6. Example Real-World Bug

Two requests arrive:

Request A → Reads users  
Request B → Modifies users  

Unexpected behavior possible.

---

### 7. Scaling Disaster Scenario

Production system:

❌ App restarts → Data lost  
❌ Multiple servers → State mismatch  

Each server has its own memory.

Global memory ≠ Persistent storage.

---

## The Core Principle (Memorize This)

---

### 8. Global State = Hidden Coupling

Global variables create:

❌ Invisible dependencies  

Components silently depend on shared data.

---

## Proper Architecture Without Global State

---

### 9. Replace Global State with Explicit Dependencies

Instead of:

❌ Shared memory variables

Use:

✅ Injected dependencies  
✅ Dedicated data layers  
✅ Explicit state ownership  

---

## Hands-On Refactor — Removing Global State

---

### 10. Bad Pattern (Global Memory)

    let users = [];

Used everywhere.

---

### 11. Better Pattern — Repository Owns State

`repositories/user.repository.js`

    exports.createUserRepository = () => {
      let users = [];

      return {
        findAll: () => users,

        create: (userData) => {
          const user = { id: Date.now(), ...userData };
          users.push(user);
          return user;
        }
      };
    };

State is now encapsulated.

---

### 12. Why This Is Better

✅ State is isolated  
✅ No accidental modification  
✅ Clear ownership  
✅ Easier testing  

---

## Injecting State via Dependency Injection

---

### 13. Wiring Repository

`index.js`

    const { createUserRepository } = require("./repositories/user.repository");
    const { createUserService } = require("./services/user.service");

    const repo = createUserRepository();
    const service = createUserService(repo);

---

### 14. Service Using Injected Repo

    exports.createUserService = (repo) => ({
      getUsers: () => repo.findAll(),

      createUser: (name) => {
        if (!name) throw new Error("Name required");
        return repo.create({ name });
      }
    });

No global variables.

---

## Why Avoiding Global State Is Critical

---

### 15. Predictability Improves

Explicit dependencies:

✅ Clear data flow  
✅ No hidden side effects  

---

### 16. Testing Becomes Easy

Each test gets:

✅ Fresh isolated state  

Global state leaks across tests.

---

### 17. Concurrency Issues Reduced

Global state + async code = Bugs

Encapsulated state = Safer execution

---

### 18. Scaling Becomes Possible

Multiple server instances:

✅ Use shared database  
❌ Never rely on memory  

---

### 19. Debugging Becomes Easier

State ownership clarity:

✔ Repository owns data  
✔ Services use repository  

---

## Advanced Real-World Insights

---

### 20. Global State vs Application State

Global State:

❌ Hidden, shared, dangerous

Application State:

✅ Explicit, controlled, scoped

---

### 21. Safe State Locations

State should live in:

✅ Database  
✅ Cache (Redis)  
✅ Request scope  
✅ Explicit modules  

Not in global memory.

---

### 22. Exception — Legitimate Globals

Some safe globals:

✅ Configuration  
✅ Logger instances  
✅ Constants  
✅ Stateless utilities  

Dangerous globals:

❌ Mutable shared data  

---

### 23. Stateless Server Principle

Modern backend systems aim for:

> Stateless servers

Why?

✅ Horizontal scaling  
✅ Reliability  
✅ Predictability  

Global mutable state breaks statelessness.

---

## Common Beginner Mistakes

---

### 24. Typical Errors

❌ Using global arrays for data  
❌ Shared mutable variables  
❌ Global caches without care  
❌ Hidden dependencies  
❌ State modified from multiple places  
❌ Test failures due to shared state  
❌ Memory-based persistence assumptions  
❌ Confusing memory with database  

---

## Professional Best Practices

---

### 25. Production-Grade Guidelines

✅ Avoid mutable global variables  
✅ Encapsulate state inside modules  
✅ Use repositories/data layers  
✅ Prefer databases for persistence  
✅ Use dependency injection  
✅ Keep state ownership explicit  
✅ Design for statelessness  
✅ Isolate test state  
✅ Avoid hidden coupling  
✅ Refactor early  

---

## Final Mental Model

Global State:

Easy → Dangerous at scale

Encapsulated State:

Structured → Predictable & scalable

Global mutable state → Fragile systems  
Explicit controlled state → Stable architecture
