## Sending response when someone visits our route

### 1. What Is `res.send()`?

`res.send()` means:

- **“Send a response back to the user”**

Until you call **one** of these:

- `res.send()`
- `res.json()`
- `res.end()`

…the request is **not finished**.

---

### 2. Update Your Route

Modify your code like this:

    const express = require("express");

    const app = express();

    app.get("/", (req, res) => {
      res.send("Hello Express");
    });

    app.listen(3000, () => {
      console.log("Server is running on port 3000");
    });

---

### 3. Break This Line Very Slowly

Let’s understand this step by step.

---

#### `res`

- Short for **response**
- Represents **what your server sends back** to the client

---

#### `res.send("Hello Express")`

This single line does **three things**:

- Sends text: `"Hello Express"`
- Ends the request
- Tells the browser:
- “We are done”

So now the browser:

- ✅ Receives data
- ✅ Stops loading
- ✅ Shows the text

---

### 4. Run the Server Again

1. Stop the old server:

        Ctrl + C

2. Start it again:

        node index.js

3. Open your browser:

        http://localhost:3000

You should see:

- `Hello Express`

---

### 5. Important Beginner Rules

#### Rule 1 — One Response Per Route

Every route must send **exactly one response**.

❌ This is wrong:

    res.send("Hi");
    res.send("Again"); // ERROR

---

#### Rule 2 — No Response = Hanging Request

If you don’t send a response:

❌ The request hangs forever

You already experienced this earlier ✅

---

### 6. What Can `res.send()` Send?

For now, just know it can send:

- Text ✅
- HTML ✅
- JSON ✅

We’ll explore each of these later.
