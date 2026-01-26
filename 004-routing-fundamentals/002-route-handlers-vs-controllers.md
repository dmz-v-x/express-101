## Route Handlers vs Controllers

### 1. First, the Problem We’re Solving

When building a backend, you will handle requests like:

- Get users
- Create user
- Update user
- Delete user

At the beginning, everything feels simple.

You write code like this:

    app.get("/users", (req, res) => {
      res.json(users);
    });

This works.

But as your app grows:

- Logic increases
- Validation increases
- Database code appears
- Error handling grows

Suddenly, routes become **messy**.

This is where **route handlers vs controllers** comes in.

---

### 2. What Is a Route Handler?

A **route handler** is:

> The function that runs when a specific HTTP request hits a specific URL.

Example:

    app.get("/users", (req, res) => {
      res.send("Users");
    });

Here:

- `app.get("/users", ...)` defines a route
- `(req, res) => {}` is the **route handler**

So:

- Route = address + method
- Route handler = function that executes

---

### 3. What Does a Route Handler Usually Do?

In beginner code, route handlers often do **everything**:

    app.post("/users", (req, res) => {
      if (!req.body.name) {
        return res.status(400).json({ error: "Name required" });
      }

      const user = { id: nextId++, name: req.body.name };
      users.push(user);

      res.status(201).json(user);
    });

This route handler is doing:

- Input validation
- Business logic
- Data storage
- Response formatting

This is **okay for learning**, but not scalable.

---

### 4. The Problem With “Everything in Route Handlers”

As the app grows:

- Routes become very long
- Logic is duplicated
- Testing becomes hard
- Code becomes unreadable

Example of a bad smell:

    app.post("/users", (req, res) => {
      // 100+ lines of logic here
    });

This is where **controllers** are introduced.

---

### 5. What Is a Controller?

A **controller** is:

> A function whose only job is to handle application logic for a request.

Think of it as:

- Route handler → traffic police
- Controller → actual worker

In simple terms:

- **Routes decide WHERE**
- **Controllers decide WHAT**

---

### 6. Mental Model

Think like this:
    
    Request comes in  
          ↓  
    Route matches URL & method  
          ↓  
    Route calls controller  
          ↓  
    Controller does the work  
          ↓  
    Response is sent  

Routes = wiring  
Controllers = logic

---

### 7. First Separation: Routes vs Controllers

Instead of this:

    app.get("/users", (req, res) => {
      res.json(users);
    });

We move logic out:

    app.get("/users", getUsers);

Now:

- Route handler exists
- Controller exists separately

---

### 8. Creating Your First Controller

Project structure (simple):

    project/
      app.js
      controllers/
        userController.js

---

### 9. userController.js 

    const users = [];
    let nextId = 1;

    const getUsers = (req, res) => {
      res.json(users);
    };

    const createUser = (req, res) => {
      const { name } = req.body;

      if (!name) {
        return res.status(400).json({ error: "Name required" });
      }

      const user = { id: nextId++, name };
      users.push(user);

      res.status(201).json(user);
    };

    module.exports = {
      getUsers,
      createUser
    };

What this file does:

- Contains **only logic**
- Knows nothing about routes
- Is reusable and testable

---

### 10. Using Controllers in Routes

app.js:

    const express = require("express");
    const app = express();

    const { getUsers, createUser } = require("./controllers/userController");

    app.use(express.json());

    app.get("/users", getUsers);
    app.post("/users", createUser);

    app.listen(3000, () => console.log("Server running"));

Now:

- Routes are clean
- Controllers handle logic

---

### 11. Why This Is Better

Benefits:

- Cleaner routes
- Easier to read
- Logic reused
- Easier debugging
- Easier testing

Routes now read like English:

    app.get("/users", getUsers);
    app.post("/users", createUser);

---

### 12. Route Handler vs Controller — Clear Comparison

| Aspect | Route Handler | Controller |
|------|--------------|------------|
| Purpose | Match URL & method | Execute logic |
| Location | Route file | Controller file |
| Responsibility | Routing | Business logic |
| Size | Small | Can be large |
| Reusability | Low | High |
| Testability | Hard | Easy |

---

### 13. Important Clarification

A controller is **still a function**.

This is valid:

    app.get("/users", controllerFunction);

Controllers are **not special objects**.
They are **normal functions**, just better organized.

---

### 14. Advanced: Controllers Should Not Do Everything

As apps grow further, controllers should:

- Read input
- Call services
- Return responses

They should NOT:

- Contain database queries directly (eventually)
- Contain complex business rules

That leads to the **service layer** (next evolution).

---

### 15. Professional Layered Architecture

Real-world backend structure:

Request  
→ Route  
→ Controller  
→ Service  
→ Database  

But for now:

- Route + Controller is the **first major improvement**

---

### 16. Common Beginner Mistakes

- Putting logic back into routes
- Naming controllers poorly
- Mixing routing and logic again
- Overengineering too early

Rule:

> Small app → simple controllers  
> Growing app → structured controllers

---

### 17. When Should You Introduce Controllers?

Use controllers when:

- Routes exceed 5–10 lines
- Logic is repeated
- Multiple routes share logic
- App grows beyond a demo

---

### 18. Final Mental Model

Say this in your head:

Routes decide **where**  
Controllers decide **what**


