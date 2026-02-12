## Tokens in Express Applications

## Understanding the Big Picture

---

### 1. First: Why Do Tokens Exist?

HTTP is stateless.

Meaning:

❌ Server forgets everything after responding.

Problem:

User logs in → Server responds → Server forgets user.

Tokens solve this.

---

### 2. What Is a Token?

A token is:

> A piece of data that proves identity

Think:

Token = Digital identity card

---

### 3. Critical Mental Model

Session:

Server remembers user.

Token:

Client carries proof.

---

## How Token-Based Authentication Works

---

### 4. High-Level Flow

User logs in  
   ↓  
Server verifies credentials  
   ↓  
Server issues token  
   ↓  
Client stores token  
   ↓  
Client sends token on each request  
   ↓  
Server verifies token  

---

### 5. Important Clarification

With tokens:

✅ Server stores NOTHING about user session

Token contains identity proof.

---

## Hands-On Example — Basic Token Flow

---

### 6. Login Route (Simplified)

    app.post("/login", async (req, res) => {
      const { email, password } = req.body;

      const user = await findUser(email);

      if (!user) {
        return res.status(401).json({ error: "Invalid credentials" });
      }

      const valid = await comparePassword(password, user.password);

      if (!valid) {
        return res.status(401).json({ error: "Invalid credentials" });
      }

      const token = generateToken(user);

      res.json({ token });
    });

---

### 7. What Just Happened?

Server created:

✅ Identity proof → Token

---

## Where Client Stores Token

---

### 8. Common Storage Options

Browser:

- LocalStorage
- SessionStorage
- Memory (safer)

Mobile apps:

- Secure storage

---

### 9. Critical Security Warning

LocalStorage:

❌ Vulnerable to XSS

Safer:

✅ httpOnly cookies (advanced pattern)

---

## Sending Token to Server

---

### 10. Standard Pattern — Authorization Header

    Authorization: Bearer TOKEN

---

### 11. Example Request

    GET /profile
    Authorization: Bearer abc123xyz

---

## Token Verification Middleware

---

### 12. Example Middleware

    const authMiddleware = (req, res, next) => {
      const header = req.headers.authorization;

      if (!header) {
        return res.status(401).json({ error: "No token provided" });
      }

      const token = header.split(" ")[1];

      try {
        const decoded = verifyToken(token);
        req.user = decoded;
        next();
      } catch (err) {
        res.status(401).json({ error: "Invalid token" });
      }
    };

---

### 13. Protected Route Example

    app.get("/profile", authMiddleware, (req, res) => {
      res.json({ user: req.user });
    });

---

## What Is Inside a Token?

---

### 14. Token Typically Contains

Examples:

✅ User ID  
✅ Claims  
✅ Expiration  
✅ Signature  

---

### 15. Important Clarification

Token is:

❌ NOT encrypted by default  
✅ Signed (tamper-proof)

---

## JWT — Most Common Token Format

---

### 16. What Is JWT?

JWT = JSON Web Token

Structure:

    header.payload.signature

---

### 17. JWT Issuance Example

    const jwt = require("jsonwebtoken");

    const token = jwt.sign(
      { userId: user.id },
      process.env.JWT_SECRET,
      { expiresIn: "1h" }
    );

---

### 18. JWT Verification

    const decoded = jwt.verify(token, process.env.JWT_SECRET);

---

## Critical Token Concepts

---

### 19. Expiration (Very Important)

Never issue:

❌ Infinite tokens

Always:

✅ Expiring tokens

---

### 20. Why Expiration Matters

Protects against:

❌ Stolen tokens  
❌ Long-term abuse  

---

### 21. Stateless Nature

Token-based auth:

✅ Server does NOT track sessions

Good for scaling.

---

## Access Token vs Refresh Token (Advanced)

---

### 22. Why Two Tokens?

Security + Usability balance.

---

### 23. Access Token

✅ Short-lived  
✅ Sent frequently  

---

### 24. Refresh Token

✅ Long-lived  
✅ Used to obtain new access tokens  

---

### 25. Flow Example

Access token expires  
   ↓  
Client sends refresh token  
   ↓  
Server issues new access token  

---

## Token Revocation Problem (Advanced Insight)

---

### 26. Sessions vs Tokens

Sessions:

✅ Easy invalidation

Tokens:

❌ Harder invalidation

---

### 27. Solutions

Use:

✅ Token blacklist  
✅ Short expiration  
✅ Versioning strategy  

---

## Security Best Practices

---

### 28. Never Store Sensitive Data in Token

JWT payload visible.

Avoid:

❌ Passwords  
❌ Secrets  
❌ Personal sensitive data  

---

### 29. Use Strong Secrets

Weak secret:

❌ Token forgery risk

---

### 30. Always Use HTTPS

Without HTTPS:

❌ Token interception

---

### 31. Protect Against XSS

XSS → Token theft.

---

### 32. Token Size Matters

Large payload:

❌ Slower requests

Keep lightweight.

---

## Common Beginner Mistakes

---

### 33. Typical Errors

❌ No expiration  
❌ Weak secret  
❌ Storing sensitive data  
❌ Trusting client payload  
❌ No verification middleware  
❌ Using tokens without HTTPS  
❌ LocalStorage misuse  
❌ Ignoring token revocation  
❌ Massive JWT payloads  
❌ Confusing encoding vs encryption  

---

## Professional Best Practices

---

### 34. Production-Grade Guidelines

✅ Always verify tokens  
✅ Use expiration  
✅ Use strong secrets  
✅ Use HTTPS  
✅ Keep payload minimal  
✅ Avoid sensitive data  
✅ Consider refresh tokens  
✅ Handle token revocation  
✅ Protect storage strategy  
✅ Use httpOnly cookies when appropriate  
✅ Log auth failures  
✅ Rate-limit login endpoints  

---

## Final Mental Model

Token:

Portable identity proof

Server:

Trusts token only after verification

Tokens = Stateless identity mechanism
