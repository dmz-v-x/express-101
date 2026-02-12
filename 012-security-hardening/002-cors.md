## CORS

## Understanding the Big Picture

---

### 1. First: What Problem Does CORS Solve?

Browsers enforce a rule called:

Same-Origin Policy (SOP)

Meaning:

A webpage cannot freely call APIs from another origin.

Without this rule:

Massive security risks  
Data theft vulnerabilities

---

### 2. What Is an Origin?

An origin consists of:

Protocol  
Domain  
Port

Example:

http://localhost:3000

Protocol → http  
Domain → localhost  
Port → 3000

Change ANY → Different origin

---

### 3. Example of Different Origins

These are different origins:

http://localhost:3000  
http://localhost:4000  
https://localhost:3000  
http://example.com

Even ports matter.

---

## The Core CORS Problem

---

### 4. Scenario

Frontend:

http://localhost:3000

Backend:

http://localhost:5000

Browser blocks request:

Not same origin.

---

### 5. Critical Mental Model

CORS is:

NOT server security  
Browser security mechanism

Server always receives requests.

Browser decides whether JavaScript can read the response.

---

## What Is CORS?

---

### 6. Definition

CORS = Cross-Origin Resource Sharing

It is:

A mechanism allowing servers to tell browsers which origins are allowed.

---

## How CORS Works (High-Level Flow)

---

### 7. Flow

Browser sends request  
Server sends special headers  
Browser checks headers  
Browser allows or blocks response

---

## Key CORS Header

---

### 8. Access-Control-Allow-Origin

Example:

Access-Control-Allow-Origin: http://localhost:3000

Meaning:

This origin is allowed.

---

## Hands-On Setup in Express

---

### 9. Install Middleware

    npm install cors

---

### 10. Basic Usage

    const cors = require("cors");

    app.use(cors());

---

### 11. What Just Happened?

Server now allows:

All origins (default).

Good for development, risky for production.

---

## Restricting Origins (Important)

---

### 12. Controlled CORS Example

    app.use(cors({
      origin: "http://localhost:3000"
    }));

---

### 13. Why Restrict Origins?

Avoid:

Any website calling your API.

---

## Multiple Allowed Origins

---

### 14. Example

    const allowedOrigins = [
      "http://localhost:3000",
      "https://myapp.com"
    ];

    app.use(cors({
      origin: (origin, callback) => {
        if (!origin || allowedOrigins.includes(origin)) {
          callback(null, true);
        } else {
          callback(new Error("Not allowed by CORS"));
        }
      }
    }));

---

## Simple vs Non-Simple Requests (Critical Concept)

---

### 15. Simple Requests

No preflight required.

Conditions:

GET / POST / HEAD  
Standard headers only  
Safe Content-Type

---

### 16. Non-Simple Requests

Triggers:

Preflight request (OPTIONS).

Examples:

PUT / DELETE / PATCH  
Custom headers  
application/json Content-Type

---

## Preflight Requests Explained

---

### 17. What Is a Preflight?

Browser sends:

OPTIONS /endpoint

Before actual request.

Purpose:

Ask server permission.

---

### 18. Example Flow

Browser:

OPTIONS → "Can I send PUT?"

Server:

Yes / No

---

### 19. Important Headers in Preflight

Access-Control-Allow-Methods  
Access-Control-Allow-Headers

---

## Handling Preflight in Express

---

### 20. Good News

cors middleware handles preflight automatically.

---

## Allowing Specific Methods

---

### 21. Example

    app.use(cors({
      origin: "http://localhost:3000",
      methods: ["GET", "POST", "PUT"]
    }));

---

## Allowing Headers

---

### 22. Example

    app.use(cors({
      origin: "http://localhost:3000",
      allowedHeaders: ["Content-Type", "Authorization"]
    }));

---

## Credentials (Very Important Advanced Topic)

---

### 23. What Are Credentials?

Includes:

Cookies  
Authorization headers  
TLS client certificates

---

### 24. Enabling Credentials

    app.use(cors({
      origin: "http://localhost:3000",
      credentials: true
    }));

---

### 25. Critical Rule

If credentials enabled:

Cannot use origin: "*"

Must specify exact origin.

---

### 26. Client-Side Requirement

Frontend must send:

    credentials: "include"

Example (fetch):

    fetch("http://localhost:5000/data", {
      credentials: "include"
    });

---

## Common CORS Errors (Very Important)

---

### 27. Typical Browser Error

Blocked by CORS policy

Meaning:

Browser rejected response.

---

### 28. Common Causes

Wrong origin  
Missing headers  
Credentials mismatch  
Preflight failure  
Proxy confusion

---

## Critical Debugging Mental Model

---

### 29. Key Insight

CORS errors = Browser issues.

Check:

Network tab  
Response headers

---

## Security Implications (Critical Understanding)

---

### 30. CORS Is NOT API Security

CORS does NOT block:

curl  
Postman  
Direct HTTP calls

Only browser-enforced.

Still need:

Authentication  
Authorization

---

## Advanced Production Best Practices

---

### 31. Production Guidelines

Restrict origins  
Avoid wildcard (*)  
Allow only needed methods  
Allow only required headers  
Use credentials carefully  
Use HTTPS  
Log rejected origins

---

## Typical Real-World Setup

---

### 32. Common Production Example

    app.use(cors({
      origin: [
        "https://myfrontend.com",
        "https://admin.myfrontend.com"
      ],
      credentials: true,
      methods: ["GET", "POST", "PUT", "DELETE"],
      allowedHeaders: ["Content-Type", "Authorization"]
    }));

---

## Common Beginner Mistakes

---

### 33. Typical Errors

Using "*" in production  
Confusing browser vs server behavior  
Ignoring credentials rules  
Misunderstanding preflight  
Trusting CORS for security  
Not checking response headers

---

## Final Mental Model

Same-Origin Policy:

Browser restriction

CORS:

Server telling browser what is allowed

Browser:

Final decision maker
