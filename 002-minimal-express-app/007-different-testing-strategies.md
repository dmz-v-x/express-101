## Different testing strategies

### 1. Test 1 — Browser 

Your server should already be running:

    node index.js

Open your browser and go to:

    http://localhost:3000

You should see:

- `Hello Express`

This confirms:

- Server is running
- Route is working
- Response is sent correctly

---

### 2. Test 2 — `curl`

`curl` lets you make HTTP requests directly from the terminal.

Run:

    curl http://localhost:3000

Output:

- `Hello Express`

This proves:

- Browsers and tools behave the same
- The server does not care who requests it

---

### 3. Test 3 — Postman

If you use Postman:

- **Method:** `GET`
- **URL:** `http://localhost:3000`
- Click **Send**

**Response body:**

- `Hello Express`

This is how:

- APIs are tested
- Backend developers debug servers

---

### 4. Why Do We Test in Different Ways?

Because each tool represents a different use case:

- **Browser** → real user
- **curl** → automation / scripts
- **Postman** → API testing

👉 Same server. Same route. Same result.

---

### 5. Exercise — Minimal Express Server

**Goal:**

- Create a server that returns `"Hello Express"`

Your complete `index.js`:

    const express = require("express");

    const app = express();

    app.get("/", (req, res) => {
      res.send("Hello Express");
    });

    app.listen(3000, () => {
      console.log("Server is running on port 3000");
    });


