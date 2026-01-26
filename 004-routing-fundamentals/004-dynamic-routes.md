## Dynamic Routes

### 1. First: What Is a Route?

A **route** answers two questions:

- Where is the request going? (URL path)
- What kind of request is it? (GET, POST, etc.)

Example of a fixed route:

    app.get("/users", (req, res) => {
      res.send("All users");
    });

This route matches **only**:

- `/users`

Not `/users/1`  
Not `/users/profile`

---

### 2. The Problem With Only Fixed Routes

Real applications deal with **dynamic data**:

- User with ID `1`
- User with ID `42`
- Product with ID `99`

If you try to do this with fixed routes, you’d need:

- `/users/1`
- `/users/2`
- `/users/3`
- `/users/4`

This is impossible.

👉 **We need routes that can change dynamically.**

---

### 3. What Is a Dynamic Route? 

A **dynamic route** is a route where **part of the URL is a variable**.

Example:

    /users/42

Here:

- `/users` → fixed part
- `42` → dynamic part

Express lets you name that dynamic part.

---

### 4. Your First Dynamic Route 

Example:

    app.get("/users/:id", (req, res) => {
      res.send("User route");
    });

What does `:id` mean?

- `:` tells Express this is **dynamic**
- `id` is the **name** of the variable

This route matches:

- `/users/1`
- `/users/42`
- `/users/abc`

---

### 5. How Express Extracts Dynamic Values

When a request comes in:

    GET /users/42

Express does this internally:

- Sees `:id`
- Takes the value from the URL
- Stores it in `req.params`

So you get:

    req.params = {
      id: "42"
    }

---

### 6. Accessing Dynamic Route Values

Example:

    app.get("/users/:id", (req, res) => {
      const userId = req.params.id;
      res.send(`User ID is ${userId}`);
    });

Request:

- `/users/42`

Response:

- `User ID is 42`


---

### 7. Important Rule

Dynamic route values are **ALWAYS strings**.

Even if they look like numbers:

    typeof req.params.id  → "string"

Convert manually if needed:

    const id = Number(req.params.id);

---

### 8. Multiple Dynamic Segments

You can have **more than one dynamic part**.

Example:

    app.get("/users/:userId/posts/:postId", (req, res) => {
      res.json(req.params);
    });

Request:

- `/users/10/posts/99`

Result:

    {
      userId: "10",
      postId: "99"
    }

Order matters.

---

### 9. Dynamic Routes vs Query Strings 

Dynamic route:

- `/users/42`

Query string:

- `/users?id=42`

Difference:

- Dynamic routes → identify a resource
- Query strings → filter or modify behavior

Rule of thumb:

- **Which thing?** → dynamic route
- **How / filter / options?** → query string

---

### 10. Real-World Use Cases for Dynamic Routes

Dynamic routes are used when:

- You are targeting a **specific resource**

Examples:

- `/users/5`
- `/products/99`
- `/orders/abc123`

This is core REST API design.

---

### 11. Route Matching Order 

Dynamic routes can match **too much**.

Example (BUG):

    app.get("/users/:id", (req, res) => {
      res.send(`User ${req.params.id}`);
    });

    app.get("/users/profile", (req, res) => {
      res.send("Profile page");
    });

Request:

- `/users/profile`

What happens?

- `:id` matches `"profile"`
- First route wins
- Profile route never runs

❌ Bug caused by order.

---

### 12. Correct Ordering Rule 

👉 **Specific routes first, dynamic routes later**

Correct order:

    app.get("/users/profile", ...);
    app.get("/users/settings", ...);
    app.get("/users/:id", ...);

This rule alone prevents many bugs.

---

### 13. Advanced: Optional Dynamic Routes

You can make parameters optional:

    app.get("/users/:id?", (req, res) => {
      res.send(req.params.id || "All users");
    });

Matches:

- `/users`
- `/users/42`

⚠️ Use carefully — optional params can reduce clarity.

---

### 14. Advanced: Validating Dynamic Parameters

Never trust URL input blindly.

Example:

    app.get("/users/:id", (req, res) => {
      const id = Number(req.params.id);

      if (Number.isNaN(id)) {
        return res.status(400).send("Invalid user ID");
      }

      res.send(`User ID is ${id}`);
    });

This prevents invalid URLs like:

- `/users/abc`

---

### 15. Advanced: Dynamic Routes + Middleware

Middleware still runs before the route.

Example:

    app.use("/users/:id", (req, res, next) => {
      console.log("User route accessed");
      next();
    });

    app.get("/users/:id", (req, res) => {
      res.send(`User ${req.params.id}`);
    });

Middleware has access to `req.params` too.

---

### 16. Common Beginner Mistakes 

- Putting `/:id` before fixed routes
- Forgetting params are strings
- Using params instead of query strings
- Overusing optional params
- Not validating input

---

### 17. Best Practices 

- Use dynamic routes for identification
- Use clear param names (`:userId`, not `:id` everywhere)
- Validate params
- Keep routes readable
- Place dynamic routes at the bottom

---

### 18. Final Mental Model 

Say this slowly:

Dynamic routes let the URL carry data  
That data lives in `req.params`  
Order matters — first match wins  


