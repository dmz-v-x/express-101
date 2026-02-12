## Rate Limiting

## Understanding the Big Picture

---

### 1. What Is Rate Limiting?

Rate limiting means:

Restricting how many requests a client can make within a time period.

Example:

100 requests per minute

Think:

“Slow down. You’re sending too much traffic.”

---

### 2. Why Rate Limiting Exists

Rate limiting protects against:

Brute-force attacks  
Spam & bots  
API abuse  
Denial-of-service (DoS)  
Server overload  
Unfair resource usage  

---

### 3. Critical Mental Model

Rate limiting = Traffic control

Like:

Speed limits on roads

---

## What Happens Without Rate Limiting

---

### 4. Real-World Risks

Attacker floods API → Server crashes  
Bot scrapes data → Resources exhausted  
Password guessing → Account compromise  
One user hogs resources → Others suffer  

---

## What Happens When Limit Is Exceeded

---

### 5. Standard Response

HTTP Status Code:

429 Too Many Requests

Optional header:

Retry-After

Example:

Retry-After: 60

Meaning:

Try again in 60 seconds.

---

## Basic Express Implementation (Hands-On)

---

### 6. Install Middleware

    npm install express-rate-limit

---

### 7. Minimal Setup

    const rateLimit = require("express-rate-limit");

    const limiter = rateLimit({
      windowMs: 60 * 1000, // 1 minute
      max: 100              // limit each IP
    });

    app.use(limiter);

---

### 8. What Just Happened?

Every IP:

Allowed 100 requests per minute.

After that:

429 response.

---

## Breaking Down the Configuration

---

### 9. windowMs

Defines time window.

Example:

1 minute  
15 minutes  
1 hour  

---

### 10. max

Defines allowed requests.

Example:

10 → Strict  
100 → Moderate  
1000 → Relaxed  

---

## Global vs Route-Specific Limits

---

### 11. Global Limiting

    app.use(limiter);

Applies everywhere.

---

### 12. Route Limiting (Better Control)

    app.post("/login", strictLimiter);

Protect sensitive endpoints.

---

### 13. Example — Strict Login Limit

    const loginLimiter = rateLimit({
      windowMs: 60 * 1000,
      max: 5
    });

    app.post("/login", loginLimiter, handler);

Prevents brute-force attacks.

---

## Key Limiting Strategies (Core Concepts)

---

## Fixed Window (Simple but Imperfect)

---

### 14. How It Works

Count requests within window.

Reset at boundary.

---

### 15. Problem

Burst at window edges.

Example:

100 requests at 59s  
100 requests at 61s  

Effectively:

200 requests in 2 seconds.

---

## Sliding Window (More Accurate)

---

### 16. How It Works

Tracks requests continuously.

Smoother enforcement.

---

### 17. Advantage

Prevents burst spikes.

---

## Token Bucket (Very Common)

---

### 18. How It Works

Tokens added over time.

Requests consume tokens.

---

### 19. Advantage

Allows bursts + sustained control.

---

## Leaky Bucket (Traffic Smoothing)

---

### 20. How It Works

Requests flow at constant rate.

---

## Choosing the Right Strategy

---

### 21. Practical Guidance

Small apps → Fixed window OK  
APIs at scale → Sliding / Token bucket  

---

## What Should You Limit By?

---

## IP-Based Limiting (Default)

---

### 22. Most Common

Limit per IP address.

---

### 23. Problem

Shared IPs (corporate networks, NAT)

May block legitimate users.

---

## User-Based Limiting (Advanced)

---

### 24. Better Strategy

Limit per authenticated user.

Requires:

Authentication middleware

---

### 25. Example Idea

userId → request count

More fair.

---

## Burst vs Sustained Limits

---

### 26. Burst Control

Allow short spikes.

---

### 27. Sustained Control

Limit long-term usage.

---

## Advanced Production Considerations

---

## Distributed Systems Problem

---

### 28. Why Local Memory Fails

Multiple servers → Separate counters

User bypasses limits via load balancing.

---

## Centralized Store Solution

---

### 29. Use Shared Storage

Redis  
Database  
Cache layer  

---

### 30. Why Redis Is Popular

Fast  
In-memory  
Atomic operations  

---

## Reverse Proxy Interaction

---

### 31. Critical Issue

Behind Nginx / Load Balancer:

All requests appear from proxy IP.

---

### 32. Fix

Enable trust proxy:

    app.set("trust proxy", 1);

Now Express reads real client IP.

---

## Rate Limiting Headers (Professional APIs)

---

### 33. Useful Headers

RateLimit-Limit  
RateLimit-Remaining  
RateLimit-Reset  

Helps clients adapt behavior.

---

## Common Real-World Mistakes

---

### 34. Typical Errors

No rate limiting at all  
Overly strict limits  
Ignoring login endpoints  
Using memory store in cluster  
Not trusting proxy  
Blocking OPTIONS (breaks CORS)  
No Retry-After header  
Wildcard abuse policies  
Ignoring burst patterns  
Not differentiating endpoints  

---

## Best Practices (Production-Grade)

---

### 35. Professional Guidelines

Always rate limit  
Different limits per endpoint  
Strict limits on auth routes  
Allow bursts wisely  
Use Redis for scaling  
Return 429 correctly  
Include Retry-After  
Whitelist trusted systems  
Monitor abuse patterns  
Log limit violations  
Tune limits with real data  
Avoid harming legitimate users  
Combine with CAPTCHA if needed  
Document limits in API spec  

---

## Advanced Security Insight

---

### 36. Rate Limiting ≠ Security Alone

Still require:

Authentication  
Authorization  
Validation  
Monitoring  

Rate limiting = Damage control layer

---

## Final Mental Model

Rate limiting = API survival mechanism

Controls:

Abuse  
Bots  
Attacks  
Traffic spikes  
Resource fairness  
