## JWT + CORS

## Understanding the Big Picture

---

### 1. First: Why JWT + CORS Needs Special Attention

JWT is usually sent via:

Authorization headers

CORS controls:

Cross-origin request rules

Together → Subtle browser behavior.

---

## Foundation Concepts (Critical)

---

### 2. JWT Transmission Methods

JWT can be sent using:

Authorization header  
Cookies  

Each behaves differently with CORS.

---

### 3. Critical Mental Model

How JWT is stored & transmitted  
→ Determines CORS configuration

---

## Pattern 1 — JWT via Authorization Header (Most Common)

---

### 4. Standard Request Example

    GET /profile
    Authorization: Bearer eyJhbGciOi...

---

### 5. Why This Triggers Preflight

Authorization header = Custom header

Browser sends:

OPTIONS preflight

---

### 6. Backend CORS Configuration (Mandatory)

    app.use(cors({
      origin: "http://localhost:3000",
      allowedHeaders: ["Content-Type", "Authorization"]
    }));

Without allowedHeaders:

Browser blocks request.

---

### 7. Critical Insight 🚨

JWT header usage ALWAYS requires:

Access-Control-Allow-Headers: Authorization

---

## Pattern 2 — JWT via Cookies (Advanced & Safer)

---

### 8. Why Use Cookies for JWT?

Advantages:

Automatic sending  
httpOnly protection  
Reduced XSS risk  

---

### 9. Cookie-Based JWT Example

    res.cookie("token", jwtToken, {
      httpOnly: true,
      secure: true,
      sameSite: "None"
    });

---

### 10. Required CORS Configuration

    app.use(cors({
      origin: "http://localhost:3000",
      credentials: true
    }));

Critical rules:

Must enable credentials  
Cannot use wildcard "*"

---

### 11. Client-Side Requirement

    fetch("http://localhost:5000/profile", {
      credentials: "include"
    });

Without this:

Cookies NOT sent.

---

## Pattern Comparison (Critical Insight)

---

### 12. Authorization Header Pattern

Pros:

Simple  
Stateless  
Explicit  

Cons:

Preflight triggered  
Manual header management  
XSS storage risks  

---

### 13. Cookie JWT Pattern

Pros:

Safer storage  
Automatic sending  
Better XSS protection  

Cons:

Requires credentials setup  
CSRF considerations  

---

## Common JWT + CORS Bugs

---

### 14. Bug 1 — Missing Authorization Header Allowance

Symptom:

Preflight fails

Fix:

Allow Authorization header.

---

### 15. Bug 2 — Wildcard Origin + Credentials

INVALID:

    origin: "*",
    credentials: true

Browser rejects.

---

### 16. Bug 3 — JWT Works in Postman but Fails in Browser

Critical Insight:

Postman ignores CORS.

Browser enforces CORS.

---

### 17. Bug 4 — Preflight OPTIONS Blocked

Proxy / firewall blocks OPTIONS:

Actual request never sent.

---

### 18. Bug 5 — Cookies Not Sent

Common causes:

Missing credentials: include  
Wrong sameSite  
Missing secure flag  
HTTP instead of HTTPS  

---

## Advanced JWT + CORS Production Patterns

---

## Pattern — Strict Header Whitelisting

---

### 19. Production Example

    app.use(cors({
      origin: ["https://myfrontend.com"],
      methods: ["GET", "POST", "PUT", "DELETE"],
      allowedHeaders: ["Content-Type", "Authorization"]
    }));

Principle:

Least privilege.

---

## Pattern — Token Refresh + CORS

---

### 20. Common Architecture

Access Token → Authorization header  
Refresh Token → httpOnly cookie  

Why?

Security + usability balance.

---

## Pattern — Avoiding Preflight Overhead (Advanced)

---

### 21. Optimization Strategy

Reduce custom headers.

But:

Authorization header usually unavoidable.

Preflight caching helps.

---

### 22. Preflight Cache Header

    Access-Control-Max-Age: 600

Reduces OPTIONS frequency.

---

## Security Insights (Critical)

---

### 23. CORS ≠ JWT Security

CORS:

Browser enforcement only.

JWT Security Requires:

Verification  
Expiration  
Signature validation  

---

### 24. Cookie JWT + CSRF Consideration

Cookies sent automatically:

Requires CSRF mitigation.

Use:

CSRF tokens  
SameSite strategy  

---

### 25. Authorization Header JWT + CSRF

Less CSRF risk.

Because header NOT automatic.

---

## Debugging JWT + CORS Issues

---

### 26. Critical Debugging Steps

Check Network tab  
Inspect OPTIONS request  
Inspect response headers  
Verify Allow-Headers  
Verify Allow-Origin  
Verify Credentials settings  
Test HTTPS vs HTTP  
Check cookies storage  
Ignore Postman success  

---

## Common Beginner Mistakes

---

### 27. Typical Errors

Forgetting Authorization header allowance  
Using wildcard in production  
Ignoring preflight behavior  
Confusing Postman vs browser behavior  
Misunderstanding credentials rules  
Storing JWT insecurely  
Ignoring expiration  
Ignoring CSRF risks  
Blocking OPTIONS accidentally  

---

## Professional Best Practices

---

### 28. Production-Grade Guidelines

Always whitelist Authorization header  
Restrict origins  
Avoid wildcard "*"  
Use HTTPS  
Understand preflight behavior  
Use httpOnly cookies when appropriate  
Separate access & refresh tokens  
Protect refresh tokens carefully  
Keep JWT payload minimal  
Set expiration  
Verify tokens strictly  
Log failures  
Test in real browsers  
Understand CSRF implications  

---

## Final Mental Model

JWT Transmission Method  
→ Determines Browser Behavior  
→ Determines CORS Configuration

Headers → Preflight + allowedHeaders  
Cookies → credentials + sameSite + secure
