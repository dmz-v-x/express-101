## Mounting Routers in Express (`app.use(path, router)`)

### 1. First: What Does “Mounting a Router” Mean?

When you create a router using `express.Router()`, it **does nothing by itself**.

It is just a collection of routes sitting somewhere.

To make those routes work, you must **attach (mount)** the router to your main app.

That attachment is done using:

    app.use(path, router);

👉 **Mounting = telling Express where this router should live in the URL**

---

### 2. Why Mounting Is Needed 

Think of your app like a building 🏢:

- `app` → the whole building
- `router` → a department (users, products, auth)

Mounting decides:

- On which **floor / section (URL path)** the department lives

Without mounting:

- Requests never reach the router
- Routes inside the router never run

---

### 3. The Simplest Possible Example

#### users router (`routes/users.js`)

    const express = require("express");
    const router = express.Router();

    router.get("/", (req, res) => {
      res.send("All users");
    });

    module.exports = router;

At this point:

- Router exists
- But Express does not know about it yet

---

### 4. Mounting the Router 

#### main app (`app.js`)

    const express = require("express");
    const app = express();

    const usersRouter = require("./routes/users");

    app.use("/users", usersRouter);

    app.listen(3000);

This line is the key:

    app.use("/users", usersRouter);

It means:

> “For any request that starts with `/users`, send it to `usersRouter`.”

---

### 5. How URLs Are Resolved After Mounting

Inside `usersRouter` you wrote:

    router.get("/", ...);

Because the router is mounted at `/users`:

- `/users` → matches `router.get("/")`
- `/users/` → same
- `/users/42` → matches `router.get("/:id")` (if defined)

Think of it as:

Final URL = mount path + router path

---

### 6. Visual Mental Model 

Mounting does this internally:

    app.use("/users", usersRouter);

is conceptually like:

    app.get("/users/", usersRouter.get("/"));
    app.get("/users/:id", usersRouter.get("/:id"));

You never write this manually, but this is how matching works.

---

### 7. Multiple Routers Mounted Together

Example:

    app.use("/users", usersRouter);
    app.use("/products", productsRouter);
    app.use("/auth", authRouter);

Now:

- `/users/*` → users router
- `/products/*` → products router
- `/auth/*` → auth router

This keeps your app clean and scalable.

---

### 8. Router Paths vs App Paths 

Inside router:

    router.get("/", ...);
    router.get("/:id", ...);

❌ Do NOT do this:

    router.get("/users", ...);

Because mounting already adds `/users`.

Correct rule:

- App decides **prefix**
- Router decides **remaining path**

---

### 9. Mounting Without a Path 

You can mount a router without a prefix:

    app.use(usersRouter);

Now router paths are used **as-is**.

This is rare, but useful for:

- Shared middleware routers
- Global routes

---

### 10. Router-Level Middleware + Mounting

Middleware inside a router only runs for mounted paths.

Example:

    router.use((req, res, next) => {
      console.log("Users router middleware");
      next();
    });

Mounted as:

    app.use("/users", usersRouter);

This middleware runs ONLY for:

- `/users`
- `/users/*`

Not for `/products`.

---

### 11. Mount Path Is Also a Filter

Mounting acts as a **filter**.

If request path does NOT start with mount path:

- Router is completely skipped
- No middleware inside router runs

This improves performance and clarity.

---

### 12. Order Matters When Mounting Routers 

Express processes `app.use()` **top to bottom**.

Example (bug):

    app.use("/users", usersRouter);
    app.use("/", catchAllRouter);

If `catchAllRouter` sends a response, it may block others.

Best practice:

- Mount specific routers first
- Catch-all / error routers last

---

### 13. Mounting Nested Routers

Example URL:

- `/users/42/posts`

Structure:

    app.use("/users", usersRouter);
    usersRouter.use("/:id/posts", postsRouter);

Now:

- `/users/42/posts` → postsRouter
- `req.params.id` is available in postsRouter

This is powerful for large APIs.

---

### 14. Mounting + Route Matching Order Combined

Inside a router:

- Routes still match **top to bottom**
- Dynamic routes (`/:id`) can steal requests

So this inside a router is BAD:

    router.get("/:id", ...);
    router.get("/profile", ...);

Correct order:

    router.get("/profile", ...);
    router.get("/:id", ...);

Mounting does NOT change this rule.

---

### 15. Common Beginner Mistakes 

- Forgetting to mount the router
- Using full paths inside router
- Mounting routers in wrong order
- Expecting router to work without `app.use`
- Over-nesting routers too early

---

### 16. When Should You Use Router Mounting?

You should mount routers when:

- App has multiple resources
- Routes exceed ~10–15
- You want clean separation
- Multiple devs work together

This is **standard professional practice**.

---

### 17. Final Mental Model 

Say this slowly:

`express.Router()`  
defines grouped routes  

`app.use(path, router)`  
decides **where they live in the URL**
