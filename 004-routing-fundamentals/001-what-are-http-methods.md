## What are HTTP Methods - GET, POST, PUT, PATCH, DELETE

### 1. Big Picture — What Is an HTTP Method?

When a client talks to your server, it uses an **HTTP method** to say **what it wants to do**.

Think of the method as the **verb** in a sentence:

- **GET** → “Show me”
- **POST** → “Create something for me”
- **PUT** → “Replace this resource”
- **PATCH** → “Update part of this resource”
- **DELETE** → “Remove this”

Important idea:

- The **same URL** can mean **different things** depending on the **method**.

Example:

- `GET /users/42` → fetch user
- `DELETE /users/42` → delete user

---

### 2. Two Critical Terms You Must Memorize

These come up everywhere in backend engineering.

#### Safe

A request that **does NOT change server state**.

- GET is safe
- It should not modify data (no DB writes, no emails)

#### Idempotent

Making the **same request multiple times** has the **same effect as making it once**.

- GET → idempotent
- PUT → idempotent
- DELETE → idempotent
- POST → NOT idempotent
- PATCH → usually NOT idempotent (depends on logic)

#### Why This Matters

- **Retries**: clients, proxies, and retries depend on idempotence
- **Caching**: safe + idempotent requests (GET) are cacheable
- **Reliability**: prevents accidental duplication

---

### 3. GET — Read / Fetch

#### Purpose

- Retrieve data
- No side effects
- Should NOT change server state

#### Typical Responses

- 200 OK → resource found
- 404 Not Found → resource missing

#### Example URL

/users/42

#### Express Example

    app.get("/users/:id", (req, res) => {
      const id = Number(req.params.id);
      const user = users.find(u => u.id === id);
      if (!user) return res.status(404).json({ error: "Not found" });
      res.json(user);
    });

#### Gotchas

- ❌ Do NOT perform writes in GET
- ❌ No DB updates, emails, counters
- ✅ Use query strings for filters

Example:

- `/users?role=admin`

---

### 4. POST — Create 

#### Purpose

- Create a new resource
- Perform an action

#### Idempotence

- NOT idempotent
- Calling twice may create duplicates

#### Typical Responses

- 201 Created → resource created
- 200 OK / 202 Accepted → action performed
- 409 Conflict → duplicate / conflict

#### curl Example

    curl -X POST -H "Content-Type: application/json" \
      -d '{"name":"Alice"}' http://localhost:3000/users

#### Express Example

    app.post("/users", (req, res) => {
      const { name } = req.body;
      if (!name) return res.status(400).json({ error: "Name required" });

      const newUser = { id: nextId++, name };
      users.push(newUser);

      res.status(201)
        .location(`/users/${newUser.id}`)
        .json(newUser);
    });

#### Gotchas

- Retrying POST can create duplicates
- Payment APIs use **idempotency keys**
- Always use 201 for successful creation

---

### 5. PUT — Replace

#### Purpose

- Replace the **entire resource**
- Full overwrite

#### Idempotence

- Idempotent
- Same request repeated → same final state

#### Typical Responses

- 200 OK → replaced
- 204 No Content → replaced, no body
- 201 Created → resource created via PUT (if allowed)

#### Express Example

    app.put("/users/:id", (req, res) => {
      const id = Number(req.params.id);
      const index = users.findIndex(u => u.id === id);
      const newData = { id, name: req.body.name || "" };

      if (index === -1) {
        users.push(newData);
        return res.status(201).json(newData);
      }

      users[index] = newData;
      res.status(200).json(newData);
    });

#### curl Example

    curl -X PUT -H "Content-Type: application/json" \
      -d '{"name":"Alice Updated"}' http://localhost:3000/users/42

#### Gotchas

- Missing fields may be overwritten
- PUT expects a full representation
- Decide whether PUT can create resources and document it

---

### 6. PATCH — Partial Update

#### Purpose

- Update **part** of a resource

#### Idempotence

- Often NOT idempotent
- Depends on semantics

#### Common Semantics

- Merge patch (simple object merge)
- JSON Patch (RFC 6902)

#### Express Example 

    app.patch("/users/:id", (req, res) => {
      const id = Number(req.params.id);
      const user = users.find(u => u.id === id);
      if (!user) return res.status(404).json({ error: "Not found" });

      Object.assign(user, req.body);
      res.json(user);
    });

#### curl Example

    curl -X PATCH -H "Content-Type: application/json" \
      -d '{"name":"NewName"}' http://localhost:3000/users/42

#### Gotchas

- PATCH behavior must be documented
- Be careful with arrays and nested objects
- Increment operations break idempotence

---

### 7. DELETE — Remove 

#### Purpose

- Remove a resource

#### Idempotence

- Idempotent
- Deleting twice should not break clients

#### Typical Responses

- 204 No Content → deleted
- 200 OK → deleted with body
- 404 Not Found → resource never existed

#### Express Example

    app.delete("/users/:id", (req, res) => {
      const id = Number(req.params.id);
      const index = users.findIndex(u => u.id === id);
      if (index === -1) return res.status(404).json({ error: "Not found" });

      users.splice(index, 1);
      res.status(204).send();
    });

#### Gotchas

- Decide on soft delete vs hard delete
- Consider cascading deletes
- Document behavior clearly

---

### 8. Extra Practical Details & Best Practices

#### Content-Type and Bodies

- GET / DELETE → usually no body
- POST / PUT / PATCH → accept bodies
- Always require `Content-Type: application/json` for JSON

#### Standard Status Codes

- 200 OK
- 201 Created
- 204 No Content
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 409 Conflict
- 422 Unprocessable Entity
- 500 Internal Server Error

#### Method Not Allowed (405)

- Path exists but method not supported
- Return 405 instead of 404

#### CORS & Preflight

- PUT, PATCH, DELETE may trigger OPTIONS requests
- Use CORS middleware or handle OPTIONS

#### HTML Forms Limitation

- Native forms support only GET and POST
- Use method-override or JS fetch for others

#### Idempotency Keys

- Used for POST retries (payments, orders)
- Detect duplicate requests safely

---

### 9. Mini Cheat-Sheet

- GET /resources → list or read
- GET /resources/:id → read one
- POST /resources → create
- PUT /resources/:id → full replace
- PATCH /resources/:id → partial update
- DELETE /resources/:id → delete

---

### 10. Tiny Hands-On Server Skeleton

    const express = require("express");
    const app = express();
    app.use(express.json());

    let users = [];
    let nextId = 1;

    app.get("/users", (req, res) => res.json(users));

    app.get("/users/:id", (req, res) => {
      const u = users.find(x => x.id === Number(req.params.id));
      if (!u) return res.status(404).json({ error: "Not found" });
      res.json(u);
    });

    app.post("/users", (req, res) => {
      const user = { id: nextId++, name: req.body.name || "unnamed" };
      users.push(user);
      res.status(201).location(`/users/${user.id}`).json(user);
    });

    app.put("/users/:id", (req, res) => {
      const id = Number(req.params.id);
      const idx = users.findIndex(u => u.id === id);
      const newUser = { id, name: req.body.name || "" };

      if (idx === -1) {
        users.push(newUser);
        return res.status(201).json(newUser);
      }

      users[idx] = newUser;
      res.json(newUser);
    });

    app.patch("/users/:id", (req, res) => {
      const u = users.find(x => x.id === Number(req.params.id));
      if (!u) return res.status(404).json({ error: "Not found" });
      Object.assign(u, req.body);
      res.json(u);
    });

    app.delete("/users/:id", (req, res) => {
      const idx = users.findIndex(u => u.id === Number(req.params.id));
      if (idx === -1) return res.status(404).json({ error: "Not found" });
      users.splice(idx, 1);
      res.status(204).send();
    });

    app.listen(3000, () => console.log("Listening on 3000"));

---

### 11. Common Beginner Mistakes & Gotchas

- Using POST instead of PUT (or vice versa)
- Ignoring idempotence during retries
- Overwriting fields accidentally with PUT
- Accepting inconsistent payload formats
- Not handling CORS / OPTIONS
- Using wrong status codes (200 instead of 201)

---


