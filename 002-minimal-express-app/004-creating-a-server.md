## Creating a Server

### 1. What Is a “Port”? 

Your computer has many **doors** for network traffic.  
Each door has a **number** — that number is called a **port**.

Examples of ports:

- 3000
- 4000
- 8080

Think of it like this:

- **Port = door number where people can knock**

---

### 2. Why Does a Server Need a Port?

When a browser sends a request like:

- `http://localhost:3000`

The `3000` tells the computer:

- “Which application should handle this request?”

Without a port:

- ❌ No one can reach your server
- ❌ The computer doesn’t know where to send the request

---

### 3. Add `app.listen()` to Your File

Update your `index.js` file so it looks like this:

    const express = require("express");

    const app = express();

    app.listen(3000);

---

#### `app.listen(3000)`

This line means:

- “Hey app, start listening on **port 3000**”

What happens now:

- The server starts running
- It waits for requests on port 3000

---

### 4. But How Do We Know It Started?

Right now:

- The server starts **silently**
- There is **no confirmation message**

So we usually add a **callback function**.

---

### 5. Better Version 

Update `app.listen` like this:

    app.listen(3000, () => {
      console.log("Server is running on port 3000");
    });

---

### 6. What Is This `() => {}` Thing?

This function runs:

- **After** the server has successfully started

So when you see:

- `Server is running on port 3000`

You know:

- ✅ Server is alive
- ✅ Port is open
- ✅ App is listening for requests

---

### 7. Run Your Server 

In your terminal, run:

    node index.js

You should see:

- `Server is running on port 3000`

---

### 8. What Happens If You Open the Browser Now?

Open your browser and go to:

- `http://localhost:3000`

You will see:

- `Cannot GET /`

⚠️ **This is GOOD**

It means:

- ✅ Server is running
- ✅ Express is listening
- ❌ No routes are defined yet

---

### 9. Common Beginner Mistakes

- ❌ Forgetting to run the file (`node index.js`)
- ❌ Port already in use (try `3001` instead)
- ❌ Expecting output without defining routes
