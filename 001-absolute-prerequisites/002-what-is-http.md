## What is HTTP

### 1. Why HTTP Exists

Imagine this conversation:

**Client:**
- “Give me users”

**Server:**
- “Users in what format?”
- “From where?”
- “How many?”

❌ **Confusion**

Without rules, both sides would constantly misunderstand each other.

👉 So we need **clear, strict rules** for communication.

---

### 2. What HTTP Actually Is

**HTTP** stands for:

- **HyperText Transfer Protocol**

Ignore the long name.

In simple terms, **HTTP is:**

- A **rulebook**
- That defines **how a client and a server talk to each other**

That’s it. Nothing more.

---

### 3. Client–Server Roles in HTTP

HTTP strictly follows this rule:

- **Client always starts the conversation**
- **Server always responds**

❌ Never the opposite.

📌 **Important**

- A server **cannot message the client on its own**
- The client **must ask first**

---

### 4. What an HTTP Request Means

When a client sends a request, it is saying:

- “Server, please do something for me”

Examples of requests:

- “Give me a webpage”
- “Save this data”
- “Delete this record”

Every request must follow **HTTP rules**.

---

### 5. What an HTTP Response Means

When a server responds, it is saying:

- “Here is the result of what you asked”

A response can contain:

- Data (JSON, text, etc.)
- A webpage (HTML)
- An error message

---

### 6. Statelessness

HTTP is **stateless**.

This means:

- The server **does NOT remember past requests**
- Each request is treated as **brand new**

---

### 7. What Statelessness Implies

Because HTTP is stateless:

- The server **forgets you immediately after responding**
- Every request must include **all required information**

This is why login systems need:

- Cookies
- Tokens
- Sessions

👉 These mechanisms exist **because HTTP itself is stateless**
