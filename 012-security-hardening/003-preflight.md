## CORS Preflight Requests

## Understanding the Big Picture

---

### 1. First: What Is a Preflight Request?

A preflight request is:

An automatic OPTIONS request sent by the browser **before** the real request.

Purpose:

Ask the server for permission.

Think:

Preflight = Permission check

---

### 2. Why Does Preflight Exist?

Because some requests are considered:

Potentially dangerous.

Examples:

PUT  
DELETE  
PATCH  
Custom headers  
application/json bodies  

Browser wants confirmation first.

---

## The Critical Trigger Rule

---

### 3. When Does Preflight Happen?

Preflight occurs when request is **NOT "simple"**.

A request is simple only if:

Method = GET / POST / HEAD  
Headers = Standard safe headers  
Content-Type =  
- application/x-www-form-urlencoded  
- multipart/form-data  
- text/plain  

Anything outside this → Preflight triggered

---

### 4. Example That Triggers Preflight

    fetch("http://api.com/users/1", {
      method: "PUT",
      headers: {
        "Content-Type": "application/json"
      }
    });

Why?

PUT → Non-simple method  
application/json → Non-simple content type

---

## What Actually Happens Internally

---

### 5. The Real Hidden Flow

Your code:

Sends PUT request.

Browser internally:

Step 1 → Sends OPTIONS request  
Step 2 → Waits for response  
Step 3 → Validates CORS headers  
Step 4 → Sends actual PUT  

---

## Anatomy of the Preflight Request

---

### 6. Example Preflight Request

    OPTIONS /users/1 HTTP/1.1
    Origin: http://localhost:3000
    Access-Control-Request-Method: PUT
    Access-Control-Request-Headers: Content-Type

Key headers explained:

---

### 7. Origin Header

    Origin: http://localhost:3000

Tells server:

Who is making the request.

---

### 8. Access-Control-Request-Method

    Access-Control-Request-Method: PUT

Asks:

Can I send PUT?

---

### 9. Access-Control-Request-Headers

    Access-Control-Request-Headers: Content-Type

Asks:

Can I send these headers?

---

## Anatomy of the Preflight Response

---

### 10. Proper Server Response

    HTTP/1.1 204 No Content
    Access-Control-Allow-Origin: http://localhost:3000
    Access-Control-Allow-Methods: GET, POST, PUT
    Access-Control-Allow-Headers: Content-Type

Browser now evaluates.

---

## Browser Decision Logic (Critical Insight)

---

### 11. Browser Validates Three Things

✔ Origin allowed?  
✔ Method allowed?  
✔ Headers allowed?  

Fail ANY → Request blocked

---

## Why Your Server Might "Work" But Browser Fails

---

### 12. Critical Mental Model

Server:

Receives requests normally.

Browser:

Blocks JavaScript from reading response.

Hence confusion.

---

## Express + Preflight Behavior

---

### 13. Good News

cors middleware handles preflight automatically.

    app.use(cors());

---

### 14. Manual Handling (Rarely Needed)

    app.options("*", cors());

Allows OPTIONS globally.

---

## Advanced Concept — Preflight Caching

---

### 15. Why Browsers Cache Preflight

Without caching:

❌ OPTIONS before EVERY request

Bad performance.

---

### 16. Access-Control-Max-Age Header

    Access-Control-Max-Age: 600

Meaning:

Cache permission for 600 seconds.

---

### 17. Express Example

    app.use(cors({
      maxAge: 600
    }));

---

## Advanced Debugging Insights

---

### 18. How To Spot Preflight

Open DevTools → Network tab.

Look for:

OPTIONS request before actual request.

---

### 19. Common Preflight Failures

Method missing in Allow-Methods  
Header missing in Allow-Headers  
Wildcard + credentials conflict  
Server not handling OPTIONS  
Reverse proxy interference  

---

## Advanced Security Insight

---

### 20. Preflight Is Browser Security Only

Attackers using:

curl  
Postman  
Direct HTTP  

Do NOT trigger browser protections.

Preflight ≠ API security.

Still require:

Authentication  
Authorization  

---

## Advanced Production Considerations

---

### 21. Why Overly Permissive Policies Are Dangerous

Allowing:

All methods  
All headers  
All origins  

Expands attack surface.

---

### 22. Principle of Least Privilege

Allow only:

Required origins  
Required methods  
Required headers  

---

## Common Beginner Mistakes

---

### 23. Typical Errors

Ignoring OPTIONS requests  
Forgetting allowedHeaders  
Misunderstanding JSON triggers preflight  
Confusing server vs browser behavior  
Using "*" with credentials  
Blocking OPTIONS accidentally  
Thinking preflight is an error  

---

## Final Mental Model (Memorize This)

Your Code → Browser → Preflight → Server → Browser Decision → Real Request

Preflight is:

Browser’s permission handshake.
