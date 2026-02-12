## Secure Cookies

## Understanding the Big Picture

---

### 1. What Is a Cookie?

A cookie is:

A small piece of data stored by the browser and automatically sent with requests.

Think:

Browser storage + automatic transmission

---

### 2. Why Cookies Are Sensitive

Cookies often store:

Session IDs  
Authentication tokens  
User identity data  

If stolen → Account takeover

---

## The Core Cookie Security Problem

---

### 3. Default Cookie Risks

Without proper settings:

Accessible via JavaScript (XSS risk)  
Sent over HTTP (MITM risk)  
Sent cross-site (CSRF risk)  
Long-lived (Replay risk)

---

## Critical Cookie Security Flags

---

## httpOnly — XSS Protection

---

### 4. What httpOnly Does

    httpOnly: true

Prevents:

JavaScript access

Meaning:

document.cookie cannot read it.

---

### 5. Why This Matters

Protects against:

XSS token theft

---

## secure — Transport Protection

---

### 6. What secure Does

    secure: true

Cookie sent ONLY over HTTPS.

---

### 7. Why This Matters

Prevents:

Network interception  
MITM attacks  

---

### 8. Critical Rule 🚨

secure cookies REQUIRE HTTPS.

HTTP → Cookie ignored

---

## sameSite — CSRF Protection

---

### 9. What sameSite Controls

Controls cross-site cookie sending.

Values:

Strict  
Lax  
None  

---

### 10. sameSite: "Strict"

Most secure.

Cookie sent ONLY same-site.

---

### 11. sameSite: "Lax"

Balanced default.

Allows limited cross-site navigation usage.

---

### 12. sameSite: "None"

Allows cross-origin cookies.

REQUIRES:

secure: true

---

### 13. Critical Browser Rule 🚨

sameSite=None → secure MUST be true

---

## Cookie Lifetime Controls

---

## maxAge / expires

---

### 14. Why Lifetime Matters

Long-lived cookies:

Increase attack window

---

### 15. Example

    maxAge: 15 * 60 * 1000 // 15 minutes

Shorter = Safer

---

## Domain & Path Restrictions

---

### 16. Domain Restriction

Limits cookie scope.

Example:

    domain: "myapp.com"

---

### 17. Path Restriction

Limits where cookie is sent.

Example:

    path: "/api"

---

## Hands-On Express Example

---

### 18. Minimal Secure Cookie

    res.cookie("sessionId", "abc123", {
      httpOnly: true,
      secure: true,
      sameSite: "Strict"
    });

---

### 19. Production-Grade Cookie

    res.cookie("sessionId", "abc123", {
      httpOnly: true,
      secure: true,
      sameSite: "None",
      maxAge: 15 * 60 * 1000,
      path: "/",
    });

---

## Cookie Security vs Attack Types

---

### 20. XSS Protection

Use:

httpOnly

---

### 21. MITM Protection

Use:

secure + HTTPS

---

### 22. CSRF Protection

Use:

sameSite + CSRF tokens

---

### 23. Replay Risk Reduction

Use:

Short expiration

---

## Advanced Security Insights

---

## Cookies Are Automatically Sent

---

### 24. Critical Behavior

Cookies sent automatically by browser.

Convenient → CSRF risk

---

## Cookies vs Authorization Headers

---

### 25. Key Difference

Cookies → Automatic  
Headers → Manual  

Hence:

Cookies need CSRF protection.

---

## Secure Cookies & HTTPS (Non-Negotiable)

---

### 26. Why HTTPS Is Mandatory

Without HTTPS:

secure flag useless  
Cookies vulnerable  

---

## Common Beginner Mistakes

---

### 27. Dangerous Errors

Missing httpOnly  
Using HTTP in production  
sameSite=None without secure  
Very long maxAge  
Storing sensitive data directly  
Using wildcard domain  
Misunderstanding sameSite  
Trusting cookies alone  
No CSRF protection  
Exposing cookies to JS  

---

## Professional Best Practices

---

### 28. Production-Grade Rules

Always use httpOnly  
Always use secure  
Always use HTTPS  
Prefer short expiration  
Use sameSite carefully  
Use Strict/Lax when possible  
Use None only when required  
Limit cookie scope (domain/path)  
Store identifiers, not secrets  
Use CSRF tokens  
Rotate session identifiers  
Invalidate on logout  
Avoid storing large data  
Monitor suspicious usage  
Protect refresh tokens heavily  

---

## Advanced Production Strategy

---

### 29. Common Secure Architecture

Access Token → Short-lived  
Refresh Token → httpOnly cookie  

Why?

Security + usability balance

---

## Debugging Secure Cookie Issues

---

### 30. Where To Look

Browser DevTools → Application → Cookies

Check:

Cookie present?  
secure flag satisfied?  
HTTPS used?  
sameSite correct?  

---

### 31. Classic Failure Example

Server sets cookie.

Browser silently rejects.

Causes:

HTTP instead of HTTPS  
Invalid sameSite rules  

---

## Final Mental Model

Secure Cookies = Proper Flags + HTTPS + Correct Scope

Flags are security controls, not optional extras.
