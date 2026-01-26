## Request-Response Lifecycle

### 1. What Is the Request–Response Lifecycle?

The **request–response lifecycle** is:

The complete journey of a request  
from the moment it reaches the server  
until the response is sent and the connection ends.

High-level timeline:

    Request arrives  
          ↓  
    Middleware runs  
          ↓  
    Route handler runs  
          ↓  
    Response is sent  
          ↓  
    Connection closes  

Once the response is sent:

- ❌ You cannot send anything else
- ❌ You cannot modify headers or status
- ❌ The request is finished

---

### 2. The Golden Rule 

Every request must have **exactly ONE response**

- Zero responses ❌ → request hangs forever
- Two responses ❌ → server error / crash
- Exactly one response ✅

This single rule prevents most Express bugs.

---

### 3. What Does “Sending a Response” Mean?

In Express, **any of these end the response lifecycle**:

- `res.send()`
- `res.json()`
- `res.end()`
- `res.redirect()`

After this:

- Headers are locked
- Body is sent
- Connection is considered done

---

### 4. What Happens If You Send NO Response?

Example:

    app.get("/", (req, res) => {
      console.log("Request received");
    });

Result:

- Browser keeps loading forever
- Connection stays open
- User waits endlessly

This is a **bug**.

Reason:

- The server never finished the request
- No response was sent

---

### 5. What Happens If You Send TWO Responses?

Example:

    app.get("/", (req, res) => {
      res.send("Hello");
      res.send("Again");
    });

Result:

- Error: Cannot set headers after they are sent to the client

Why:

- First `res.send()` ends the response
- Second `res.send()` tries to modify headers
- Headers are already locked

This is a **crash**.

---

### 6. Why This Rule Exists

HTTP is:

- Request → Response

It is:

- ❌ Not a chat
- ❌ Not a conversation
- ❌ Not a stream (by default)

Once the server responds:

- The conversation is over
- The lifecycle ends

---

### 7. Very Common Real-World Bug

Buggy code:

    app.get("/user", (req, res) => {
      if (!req.query.id) {
        res.status(400).send("ID required");
      }

      res.send("User data");
    });

Problem:

- If `id` is missing:
  - One response is sent (400)
  - Another response is sent ❌

This violates the golden rule.

---

### 8. Correct Version 

    app.get("/user", (req, res) => {
      if (!req.query.id) {
        return res.status(400).send("ID required");
      }

      res.send("User data");
    });

Why `return` is critical:

- Stops function execution
- Prevents the second response
- Guarantees only ONE response

Always `return` after sending an error response.

---

### 9. Response Lifecycle + Middleware

Middleware runs **between request arrival and response sending**.

Middleware can:

- Read or modify `req`
- Read or modify `res`
- Pass control using `next()`

But:

- Only ONE place should end the response

Example:

    app.use((req, res, next) => {
      console.log("Request came");
      next();
    });

    app.get("/", (req, res) => {
      res.send("Hello");
    });

Lifecycle:

Request  
→ middleware  
→ route handler  
→ response sent  
→ done  

---

### 10. Another Common Bug

Buggy async handler:

    app.get("/data", async (req, res) => {
      const data = await fetchData();
      // forgot to send response
    });

Result:

- Request hangs forever
- Browser keeps loading
- Lifecycle never ends

---

### 11. Safe Async Pattern 

    app.get("/data", async (req, res) => {
      try {
        const data = await fetchData();
        res.json(data);
      } catch (err) {
        res.status(500).json({ error: "Failed" });
      }
    });

Why this is safe:

- Success path → exactly one response
- Error path → exactly one response
- No hanging requests
- No double responses

---

### 12. How Express Knows the Response Is Finished 

Internally, Express considers the response finished when:

- Headers are sent
- Body is sent
- Socket is closed or released

You do **not** manage this manually.

Your responsibility:

- Send exactly one response

Express handles the rest.

---

### 13. Final Mental Model 

One request  
→ one response  
→ lifecycle ends  

Everything in Express revolves around this rule.

