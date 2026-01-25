## require vs import

### 1. Why Do We Need `require` or `import`?

Right now:

- Express is **installed**
- But your JavaScript file **does not know about it**

Installing a library does **not** automatically make it available in your code.

👉 To use Express, we must **load it into our file**.

---

### 2. Create Your First File

Inside your project folder (`express-app`), create a file called:

- `index.js`

This file will be:

- Your **server file**
- The starting point of your Express app

---

### 3. Way 1 (Most Common for Beginners): `require`

Inside `index.js`, write:

    const express = require("express");

Let’s break this down **very slowly**.

---

#### `require("express")`

This tells Node.js:

- “Go to the `node_modules` folder”
- “Find the Express package”
- “Load it and give it to me”

---

#### `const express = ...`

This:

- Stores the Express package in a variable named `express`

So now:

- `express` = the **Express library**

---

### 4. Important Mental Model

After this line:

    const express = require("express");

You can think:

> **“I now have Express in my hands.”**

---

### 5. What Is `express` Actually?

This is very important:

- `express` is a **function**

Later, we will write:

    const app = express();

Which means:

- “Call the Express function”
- “Create an Express application”

---

### 6. Way 2 (Modern Syntax): `import`

Some projects use modern syntax like:

    import express from "express";

But this requires extra setup, such as:

- Adding `"type": "module"` in `package.json`

👉 For now, we **stick to `require`** to keep things simple.

---

### 7. Common Beginner Mistake 

❌ Forgetting to install Express:

    require("express"); // ERROR if Express is not installed

Always remember:

- **Install first**
- **Then require/import**
