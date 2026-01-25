## What is a Server

### 1. What is a server? 

A **server** is simply:

- A **program**
- Running on a **computer**
- That **waits for requests**
- And **sends back responses**

---

### 2. The Request → Response Loop

A server always works in this cycle:

- Client sends a **request**
- Server **receives** the request
- Server **does some work**
- Server sends a **response**
- Server waits for the **next request**

This repeats **forever**.

This continuous cycle is called the **request → response loop**.

---

### 3. Very Simple Real-World Example

You open your browser and type:

- `https://google.com`

#### What actually happens?

**Browser sends a request:**

- “Hey Google server, give me your homepage”

**Google’s server then:**

- Receives the request
- Prepares the HTML
- Attaches CSS
- Attaches JavaScript

**Server sends the response:**

- HTML
- CSS
- JavaScript

**Browser:**

- Receives the response
- Renders the page on your screen

💡 **Important mental model**

- Google is **not sending pages randomly**
- It **only responds when requested**

---

### 4. Key Takeaways 

- A server is just a **program**
- It **listens** for requests
- It **responds** with data
- It repeats this process **again and again**


