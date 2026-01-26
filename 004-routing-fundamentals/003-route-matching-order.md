## Route Matching Order


### 1. First: What Does “Route Matching” Even Mean?

When a request comes to your server, Express must decide:

Which route should handle this request?

Example request:

GET /users/42

Your server might have many routes. Express must **match** the request to **one route**.

This process is called **route matching**.

---

### 2. The Single Most Important Rule

👉 **Express checks routes from TOP to BOTTOM, in the order they are written.**

The **first matching route wins**.

After a match:

- That route runs
- No other routes are checked (unless `next()` is used)

This rule explains **90% of routing bugs**.

---

### 3. How Express Thinks

Imagine Express reading your file like a human reading a book:

Line 1 → check  
Line 2 → check  
Line 3 → check  
…  
STOP when a match is found

It does **not** look ahead.  
It does **not** try to be smart.  
It follows order strictly.

---

### 4. A Very Simple Example

Code:

    app.get("/home", (req, res) => {
      res.send("Home page");
    });

Request:

GET /home

What happens:

- Express checks `/home`
- Match found
- Handler runs
- Response sent

Simple.

---

### 5. Multiple Routes — Order Starts to Matter

Code:

    app.get("/users/profile", (req, res) => {
      res.send("User profile");
    });

    app.get("/users/:id", (req, res) => {
      res.send("User by ID");
    });

Request:

GET /users/profile

What happens:

- Express checks `/users/profile` → MATCH
- It never checks `/users/:id`

Response:

User profile

Everything works.

---

### 6. Now Let’s Reverse the Order 

Code:

    app.get("/users/:id", (req, res) => {
      res.send(`User ID: ${req.params.id}`);
    });

    app.get("/users/profile", (req, res) => {
      res.send("User profile");
    });

Same request:

GET /users/profile

What happens now?

Step by step:

- Express checks `/users/:id`
- `:id` can match **anything**
- `"profile"` becomes `id`

So Express thinks:

    req.params.id = "profile"

It matches the FIRST route and stops.

Response:

User ID: profile

❌ Wrong behavior  
❌ Bug  
❌ Very common beginner mistake

---

### 7. Why This Happens

Dynamic routes like:

    /users/:id

Mean:

> “Match `/users/ANYTHING`”

So:

- `/users/42` ✅
- `/users/profile` ✅
- `/users/settings` ✅

If this route appears **before** specific routes, it will steal requests.

---

### 8. The Golden Ordering Rule

👉 **Always put more specific routes BEFORE more general routes**

Correct order:

    app.get("/users/profile", ...);
    app.get("/users/settings", ...);
    app.get("/users/:id", ...);

Think:

- Fixed paths first
- Dynamic paths later

---

### 9. Another Example: Static vs Dynamic

Bad order:

    app.get("/:page", (req, res) => {
      res.send(`Page: ${req.params.page}`);
    });

    app.get("/about", (req, res) => {
      res.send("About page");
    });

Request:

GET /about

Result:

Page: about

Because `/:page` matches **everything**.

Correct order:

    app.get("/about", ...);
    app.get("/:page", ...);

---

### 10. Route Matching Is Method + Path

Important clarification:

Express matches **both**:

- HTTP method
- Path

These are different routes:

    app.get("/users", ...);
    app.post("/users", ...);

Order matters **within the same method**, not across methods.

---

### 11. Middleware Also Follows Top-to-Bottom Order

This applies to middleware too.

Example:

    app.use((req, res, next) => {
      console.log("Middleware 1");
      next();
    });

    app.use((req, res, next) => {
      console.log("Middleware 2");
      next();
    });

Execution order:

→ Middleware 1  
→ Middleware 2  
→ Route handler

Same rule: **top to bottom**.

---

### 12. Routes Stop the Lifecycle Unless `next()` Is Used

Example:

    app.get("/test", (req, res) => {
      res.send("Hello");
    });

    app.get("/test", (req, res) => {
      res.send("World");
    });

Only the FIRST route runs.

Second route is **never reached**.

---

### 13. Advanced: Multiple Handlers for the Same Route

This works:

    app.get(
      "/users/:id",
      (req, res, next) => {
        console.log("Handler 1");
        next();
      },
      (req, res) => {
        res.send("Handler 2");
      }
    );

Why?

- `next()` explicitly tells Express to continue
- Otherwise, matching stops

---

### 14. Advanced: Router-Level Order

Inside routers, the same rule applies.

usersRouter.js:

    router.get("/profile", ...);
    router.get("/:id", ...);

Mounting order matters too:

    app.use("/users", usersRouter);

So final matching order is:

- app-level order
- then router-level order
- all still top to bottom

---

### 15. Debugging Tip

If a route is not working:

Ask yourself:

- Is another route above it matching first?
- Is there a `/:param` catching it?
- Is middleware ending the response early?

Most routing bugs are **order bugs**.

---

### 16. Common Beginner Mistakes

- Putting `/:id` before fixed routes
- Putting catch-all routes too early
- Forgetting that middleware follows order
- Assuming Express “figures it out”

Express does NOT guess. It obeys order.

---

### 17. Best Practices

- Specific → General
- Static → Dynamic
- Auth middleware before protected routes
- Catch-all routes last

Example catch-all (last line only):

    app.use((req, res) => {
      res.status(404).send("Not found");
    });

---

### 18. Final Mental Model

Say this slowly:

Express reads routes  
from top to bottom  
and stops at the first match  

---

