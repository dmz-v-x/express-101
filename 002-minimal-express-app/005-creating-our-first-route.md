## Creating our first route

### 1. What Is a “Route”?

A **route** answers two important questions:

- **Where** is the request going? → URL path  
- **What type** of request is it? → HTTP method (`GET`, `POST`, etc.)

Example:

- `GET /`

This means:

- “Someone is visiting the **home page**”

---

### 2. What Does `/` Mean?

`/` is called the **root path**.

So when someone opens:

- `http://localhost:3000`

They are actually visiting:

- `/`

---

### 3. Add Your First Route

Update your `index.js` file like this:

    const express = require("express");

    const app = express();

    app.get("/", (req, res) => {
      
    });

    app.listen(3000, () => {
      console.log("Server is running on port 3000");
    });

---

### 4. Break This Down Very Slowly

Let’s understand each piece.

---

#### `app.get(...)`

- `app` → your server
- `.get` → handle **GET** requests
- `GET` means:
  - “Give me something”
  - Browsers use `GET` by default

---

#### `"/"`

This means:

- “When someone visits the **home page** (`/`)”

---

#### `(req, res) => {}`

This is a function that runs **when the route is hit**.

- `req` → **request** (what the user sent)
- `res` → **response** (what you send back)

Mental model:

- `req` = incoming message  
- `res` = your reply  

---

### 5. Important: Nothing Is Sent Yet

Right now, your route **exists**, but:

- ❌ You are not sending anything back
- ❌ The browser will keep loading forever

---

### 6. Test Right Now

1. Stop the server → `Ctrl + C`
2. Run it again:

        node index.js

3. Open your browser:

        http://localhost:3000

Result:

- The page keeps loading

✅ This teaches a **critical rule**:

> **Every route must send a response**
