## Middleware Signature in Express `(req, res, next)`

### 1. First: What Is a Middleware Function?

Middleware is simply:

> A function that runs during the request → response lifecycle.

But Express middleware has a **special structure**.

It looks like this:

    (req, res, next)

Example:

    const myMiddleware = (req, res, next) => {
      console.log("Middleware running");
      next();
    };

These three parameters are extremely important.

---

### 2. The Three Parameters

Every Express middleware receives:

- **req** → Request object
- **res** → Response object
- **next** → Control function

Think:

- `req` = input
- `res` = output
- `next` = continue signal

---

### 3. `req` — The Incoming Request

`req` contains everything the client sent.

Examples:

- URL
- Headers
- Query params
- Body
- Route params

Example:

    console.log(req.method);
    console.log(req.url);
    console.log(req.headers);

Middleware can:

✅ Read from `req`  
✅ Modify `req`

---

### 4. `res` — The Outgoing Response

`res` represents what your server sends back.

Examples:

- Status codes
- Headers
- Body

Example:

    res.status(200).send("OK");

Middleware can:

✅ Modify `res`  
✅ Send response  
✅ End pipeline

---

### 5. `next` — The Most Important Piece 

👉 **`next()` tells Express: “Move to the next step”**

Without `next()`:

- Execution stops
- Request hangs
- Route handler never runs

Think of `next()` as:

👉 “Continue the pipeline”

---

### 6. Basic Execution Flow 

Request arrives  
→ Middleware runs  
→ `next()` called  
→ Next middleware / route runs  

Pipeline = chain of functions

---

### 7. What Happens If You Forget `next()`? 

Example:

    app.use((req, res, next) => {
      console.log("Middleware");
      // next() missing
    });

Result:

- Browser loads forever
- No response
- Pipeline blocked ❌

Golden rule:

👉 Always call `next()` OR send response.

---

### 8. Middleware That Ends the Request

Middleware can stop execution.

Example:

    const authMiddleware = (req, res, next) => {
      if (!req.headers.authorization) {
        return res.status(401).send("Unauthorized");
      }
      next();
    };

Flow:

- If invalid → response sent → STOP
- If valid → continue

---

### 9. Important Rule 

Middleware must do ONE of these:

✅ Call `next()`  
✅ Send a response  

Never neither.

---

### 10. Middleware Can Modify `req`

Example:

    app.use((req, res, next) => {
      req.user = { name: "Alice" };
      next();
    });

Later:

    res.json(req.user);

Very common for:

- Auth systems
- Injected data
- Context passing

---

### 11. Middleware Can Modify `res`

Example:

    app.use((req, res, next) => {
      res.set("X-App-Version", "1.0");
      next();
    });

Used for:

- Headers
- Cookies
- Metadata
- Security policies

---

### 12. Advanced: `next()` vs `next(err)` 

Two different meanings.

#### `next()`

Continue normally.

---

#### `next(err)`

Signal an error.

Express skips normal handlers and jumps to:

👉 **Error-handling middleware**

---

### 13. Error Middleware Signature 

Error middleware has FOUR parameters:

    (err, req, res, next)

Example:

    app.use((err, req, res, next) => {
      console.error(err);
      res.status(500).send("Something broke");
    });

Important:

👉 Express detects error middleware by **parameter count**.

---

### 14. Advanced: Middleware Is Just a Function Chain

Internally, Express does:

Middleware 1  
→ Middleware 2  
→ Middleware 3  
→ Route handler  

Each step controlled by `next()`.

No magic.

---

### 15. Advanced: Async Middleware 

Example:

    app.use(async (req, res, next) => {
      const data = await fetchSomething();
      next();
    });

If error occurs:

❌ App may crash unless handled.

Safe pattern:

    app.use(async (req, res, next) => {
      try {
        const data = await fetchSomething();
        next();
      } catch (err) {
        next(err);
      }
    });

---

### 16. Advanced: Skipping Middleware Conditionally

Middleware can decide flow.

Example:

    const checkAdmin = (req, res, next) => {
      if (req.user.role !== "admin") {
        return res.status(403).send("Forbidden");
      }
      next();
    };

Middleware = decision gate.

---

### 17. Common Beginner Mistakes 

❌ Forgetting `next()`  
❌ Calling `next()` after sending response  
❌ Sending multiple responses  
❌ Confusing middleware with controllers  
❌ Ignoring async errors  

---

### 18. Golden Middleware Execution Rule 

Say this slowly:

Middleware runs  
until someone sends a response  
or pipeline ends  

---

### 19. Final Mental Model

`req` → read/modify input  
`res` → prepare/send output  
`next()` → continue pipeline  

Three pieces. One system.
