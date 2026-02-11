## Route Groups Using `express.Router()`

This blog explains **route grouping with `express.Router()`** from **very scratch**, using **simple language**, **clear mental models**, and **practical examples**.  
Assume **zero backend knowledge**.

---

### 1. First: What Problem Are Route Groups Solving?

When your app is small, you may write routes like this:

    app.get("/users", ...);
    app.post("/users", ...);
    app.get("/users/:id", ...);
    app.delete("/users/:id", ...);

This works.

But as your app grows:

- Routes increase
- Files become very long
- Everything is in `app.js`
- Navigation becomes painful

👉 We need a way to **group related routes together**.

That’s exactly what `express.Router()` does.

---

### 2. What Is `express.Router()`? 

`express.Router()` is:

> A mini Express app that handles a group of routes.

Think of it as:

- `app` → main server
- `router` → small, focused sub-server

Each router handles **one feature or resource**.

---

### 3. Mental Model 

Think in terms of folders:

- `/users` routes → users router
- `/products` routes → products router
- `/auth` routes → auth router

So instead of one huge route file:

- You split routes by **domain**

---

### 4. Without Router vs With Router

#### Without Router (Messy as App Grows)

    app.get("/users", ...);
    app.post("/users", ...);
    app.get("/products", ...);
    app.post("/products", ...);
    app.get("/orders", ...);

Everything is mixed.

---

#### With Router (Clean & Scalable)

- `usersRouter` → handles `/users`
- `productsRouter` → handles `/products`
- `ordersRouter` → handles `/orders`

Each group lives in its own file.

---

### 5. Your First Router 

#### Step 1: Create a Router

In `routes/users.js`:

    const express = require("express");
    const router = express.Router();

This creates a **router instance**.

---

### 6. Add Routes to the Router

Still in `routes/users.js`:

    router.get("/", (req, res) => {
      res.send("All users");
    });

    router.post("/", (req, res) => {
      res.send("Create user");
    });

    router.get("/:id", (req, res) => {
      res.send(`User ${req.params.id}`);
    });

Important:

- Notice paths start with `/`
- But **NOT `/users`**

Why? You’ll see next.

---

### 7. Export the Router

At the bottom of `routes/users.js`:

    module.exports = router;

Now this router can be plugged into the app.

---

### 8. Mount the Router in the Main App

In `app.js`:

    const express = require("express");
    const app = express();

    const usersRouter = require("./routes/users");

    app.use("/users", usersRouter);

    app.listen(3000, () => {
      console.log("Server running on port 3000");
    });

This line is critical:

    app.use("/users", usersRouter);

Meaning:

> “For any request starting with `/users`, forward it to `usersRouter`.”

---

### 9. How URL Matching Works Now

With this setup:

| Request URL        | Router Path | Final Match |
|--------------------|-------------|-------------|
| GET /users         | /           | usersRouter.get("/") |
| POST /users        | /           | usersRouter.post("/") |
| GET /users/42      | /:id        | usersRouter.get("/:id") |

The prefix `/users` is **automatically added**.

---

### 10. Why Routers Are So Powerful

Routers give you:

- Clean separation
- Smaller files
- Easier navigation
- Better scalability
- Team-friendly structure

This is how **real production apps** are built.

---

### 11. Router + Controllers 

Real-world structure:

    project/
      app.js
      routes/
        users.js
      controllers/
        userController.js

#### Controller (`controllers/userController.js`)

    const getUsers = (req, res) => {
      res.json([]);
    };

    const getUserById = (req, res) => {
      res.json({ id: req.params.id });
    };

    module.exports = { getUsers, getUserById };

---

#### Router (`routes/users.js`)

    const express = require("express");
    const router = express.Router();
    const { getUsers, getUserById } = require("../controllers/userController");

    router.get("/", getUsers);
    router.get("/:id", getUserById);

    module.exports = router;

---

### 12. Router-Level Middleware 

You can attach middleware **only to a router**.

Example:

    router.use((req, res, next) => {
      console.log("Users router hit");
      next();
    });

This middleware runs **only for `/users` routes**.

Not for `/products`.

---

### 13. Example: Authentication Per Route Group

    const authMiddleware = (req, res, next) => {
      console.log("Auth checked");
      next();
    };

    router.use(authMiddleware);

Now:

- Every `/users/*` route is protected
- Other routes are unaffected

This is extremely powerful.

---

### 14. Router + Multiple Handlers

Routers fully support multiple handlers:

    router.get(
      "/:id",
      validateUser,
      authorizeUser,
      getUserById
    );

This combines:

- Route grouping
- Middleware
- Controllers

---

### 15. Route Matching Order Inside Routers

IMPORTANT:

- Routers also follow **top-to-bottom order**
- Same rules apply as `app.get`

Bad order:

    router.get("/:id", ...);
    router.get("/profile", ...);

Correct order:

    router.get("/profile", ...);
    router.get("/:id", ...);

Order rules never change.

---

### 16. Advanced: Nested Routers 

You can nest routers:

Example:

- `/users/:id/posts`

Structure:

    app.use("/users", usersRouter);
    usersRouter.use("/:id/posts", postsRouter);

Each router handles its own concern.

This is advanced but very powerful.

---

### 17. Common Beginner Mistakes 

- Putting full paths inside routers (`/users/users`)
- Forgetting to export the router
- Forgetting to mount the router with `app.use`
- Wrong route order inside router
- Overusing routers too early

---

### 18. When Should You Use Routers?

Use routers when:

- Routes exceed ~10–15
- App has multiple resources
- Code readability drops
- Multiple developers work together

For tiny demos, `app.js` is fine.

---

### 19. Final Mental Model 

Say this slowly:

`express.Router()`  
groups related routes  
under one URL prefix  

It’s a **mini app inside your app**.
