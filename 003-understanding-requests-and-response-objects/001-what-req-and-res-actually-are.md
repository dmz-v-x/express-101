## What requests and response actually are

### 1. What Problem Are `req` and `res` Solving?

Whenever someone opens a website or calls an API, **two things always happen**:

- The user **sends something** to your server
- Your server **sends something back**

Express gives you **two objects** to handle this communication:

| Object | Meaning                                      |
|------|----------------------------------------------|
| `req` | What the user sent **to your server**        |
| `res` | What your server sends **back to the user** |

These two objects are the **core of everything in Express**.

If you understand `req` and `res`, you understand Express.

---

### 2. Where Do `req` and `res` Come From?

Look at this route:

    app.get("/", (req, res) => {
      res.send("Hello Express");
    });

What Express does internally:

1. A request comes in from the user
2. Express **creates a request object**
3. Express **creates a response object**
4. Express **passes both objects into your route function**

Important:

- ❌ You do **not** create `req`
- ❌ You do **not** create `res`
- ✅ Express creates them **automatically**
- ✅ This happens **for every single request**

---

### 3. Very Important Beginner Clarification 

`req` and `res` are:

- ❌ NOT plain variables
- ❌ NOT random objects
- ❌ NOT global
- ❌ NOT shared between users

They are:

- ✅ Fresh objects
- ✅ Created **per request**
- ✅ Destroyed after the response is sent

That means:

- Every user → new `req`
- Every request → new `res`

No sharing. No memory leaks. No confusion.

---

### 4. What Kind of Objects Are They?

#### `req` — Request Object

The `req` object contains **everything about the incoming request**, such as:

- HTTP method (`GET`, `POST`, `PUT`, `DELETE`)
- URL path (`/`, `/users`, `/products`)
- Headers (browser info, auth tokens, etc.)
- Query parameters (`?page=1&sort=asc`)
- Body (data sent by the user, usually JSON)

Think of `req` as:

> **A box containing everything the user sent**

---

#### `res` — Response Object

The `res` object contains **everything you use to reply**, such as:

- Status code (`200`, `404`, `500`)
- Headers
- Data to send back
- Ending the request

Common things you do with `res`:

- `res.send()` → send text / HTML / JSON
- `res.json()` → send JSON
- `res.status()` → set HTTP status code

Think of `res` as:

> **Your reply message back to the user**

---

### 5. Think of It Like a Timeline

The full lifecycle looks like this:

User  
→ sends request  
→ `req` object created  
→ your route function runs  
→ you use `res` to respond  
→ response sent back  
→ request ends  

After this:

- `req` is gone
- `res` is gone

---

### 6. Very Important Rule 

**One request → one response**

You must:

- Send **exactly one response**
- Using `res.send()`, `res.json()`, etc.

❌ This is invalid:

    res.send("Hello");
    res.send("Again"); // ERROR

Once a response is sent, the request is finished.

---

### 7. Minimal Mental Model 

If you remember only **one thing**, remember this:

> **`req` = input to your server**  
> **`res` = output from your server**

Everything else builds on top of this idea.

---

### 8. Tiny Experiment 

Update your route like this:

    app.get("/", (req, res) => {
      console.log(req);
      res.send("Check terminal");
    });

Now:

1. Run your server
2. Open the browser
3. Look at the terminal

You will see a **huge object printed**.

That object is:

👉 **The entire incoming HTTP request**

This proves:

- `req` contains real data
- Express gives you full control over it

---

### 9. Extra

Later, you will commonly use:

- `req.params` → data from URL paths
- `req.query` → data after `?`
- `req.body` → data sent by user
- `res.status()` → HTTP status codes
- `res.json()` → API responses

For now, just remember:

> **Express = handling `req` and replying with `res`**

You’ve just learned the **heart of backend development**.
