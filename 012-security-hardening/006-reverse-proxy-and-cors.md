## Reverse Proxy & CORS 

## Understanding the Big Picture

---

### 1. First: What Is a Reverse Proxy?

A reverse proxy is:

A server that sits **between the client and your backend**

Think:

Client → Proxy → Backend

Examples:

Nginx  
Cloudflare  
AWS ALB  
API Gateway  

---

### 2. Why Reverse Proxies Exist

Reverse proxies help with:

Load balancing  
SSL termination  
Caching  
Security  
Routing  
Compression  

---

## Where CORS Confusion Begins

---

### 3. Critical Mental Model

CORS is based on:

Origin (protocol + domain + port)

Browser checks:

Response headers

---

### 4. Proxy Changes the Traffic Flow

Without proxy:

Browser → Backend

With proxy:

Browser → Proxy → Backend

This changes behavior.

---

## The Classic Problem Scenario

---

### 5. Example Setup

Frontend:

https://myapp.com

Proxy:

https://api.myapp.com

Backend:

http://internal-service:5000

Suddenly:

CORS errors appear.

Why?

Headers missing or modified.

---

## Critical Insight — Who Must Send CORS Headers?

---

### 6. The Real Rule

CORS headers must exist in:

Final HTTP response to browser

NOT internal backend response.

---

### 7. Why This Matters

If backend sends:

Access-Control-Allow-Origin

But proxy strips it:

❌ Browser sees no header  
❌ CORS failure  

---

## Reverse Proxy Header Interference

---

### 8. Common Proxy Behaviors

Proxies may:

Override headers  
Remove headers  
Add headers  
Cache responses incorrectly  

---

## Scenario 1 — Backend Sets CORS, Proxy Breaks It

---

### 9. Backend Response

    Access-Control-Allow-Origin: https://myapp.com

---

### 10. Proxy Response

Header missing.

Result:

❌ Browser blocks request

---

### 11. Fix Strategy

Ensure proxy forwards headers.

---

## Scenario 2 — Proxy Must Handle CORS

---

### 12. Alternative Strategy

Handle CORS at proxy layer.

Example (Nginx):

    add_header Access-Control-Allow-Origin https://myapp.com;
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE";
    add_header Access-Control-Allow-Headers "Content-Type, Authorization";

---

### 13. Why This Works

Browser sees headers from proxy.

CORS satisfied.

---

## Scenario 3 — SSL Termination & Origin Mismatch

---

### 14. Critical Issue

Proxy:

Handles HTTPS

Backend:

Runs HTTP

Browser origin:

HTTPS

Backend origin:

HTTP

Potential mismatch confusion.

---

### 15. Important Insight

CORS checks:

Browser ↔ Proxy response

Backend protocol irrelevant to browser.

---

## Scenario 4 — Preflight Requests & Proxy

---

### 16. Critical Behavior

Browser sends:

OPTIONS request

Proxy must:

Allow OPTIONS  
Forward OPTIONS  
Return proper headers  

---

### 17. Common Failure

Proxy blocks OPTIONS:

❌ Preflight fails  
❌ Actual request never sent  

---

### 18. Fix Strategy

Allow OPTIONS in proxy config.

---

## Scenario 5 — Credentials & Proxy

---

### 19. Critical Rule

If using cookies:

Must include:

Access-Control-Allow-Credentials: true

---

### 20. Common Proxy Failure

Proxy allows origin:

But missing credentials header.

Result:

❌ Cookies blocked  

---

## Advanced Production Considerations

---

### 21. Proxy Caching Issues

Improper caching may reuse:

Wrong CORS headers

Dangerous.

---

### 22. Vary Header (Critical Advanced Insight)

Proper responses should include:

    Vary: Origin

Prevents cache poisoning issues.

---

## Best Practice — Where Should CORS Live?

---

### 23. Two Valid Strategies

Backend-managed CORS  
Proxy-managed CORS  

---

### 24. Professional Rule

Choose ONE layer.

Avoid:

❌ Mixed CORS logic

---

## Debugging Reverse Proxy CORS Issues

---

### 25. Critical Debugging Steps

Check browser Network tab  
Inspect final response headers  
Ignore backend internal headers  
Test via curl directly to proxy  
Test OPTIONS requests  
Check proxy config  

---

### 26. Classic Debugging Mistake

Checking backend logs only.

Browser cares about:

Final response headers.

---

## Common Reverse Proxy CORS Failures

---

### 27. Typical Issues

Headers stripped  
Headers duplicated  
OPTIONS blocked  
Wildcard + credentials conflict  
Wrong origin returned  
Cache interference  
HTTPS/HTTP confusion  
Proxy misconfiguration  

---

## Professional Best Practices

---

### 28. Production-Grade Guidelines

Define CORS at one layer only  
Verify headers at browser level  
Allow OPTIONS requests  
Handle credentials carefully  
Avoid wildcard origins  
Use Vary: Origin when needed  
Avoid header duplication  
Test behind real proxy  
Understand SSL termination impact  
Log rejected origins  
Keep policies restrictive  
Use HTTPS  
Document architecture  

---

## Final Mental Model

Browser ↔ Final Response = CORS Check

Proxy becomes:

The server from browser’s perspective.

Backend headers invisible if proxy modifies them.
