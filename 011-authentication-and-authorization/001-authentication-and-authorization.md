## Authentication & Authorization in Express Applications

## Understanding the Big Picture

---

### 1. First: Why Do We Need Security?

Without security:

❌ Anyone can access your data  
❌ Anyone can modify resources  
❌ Anyone can impersonate users  

Real systems must answer:

✔ Who are you?  
✔ What are you allowed to do?  

---

## Authentication vs Authorization (Critical Foundation)

---

### 2. Authentication — Identity Verification

Authentication means:

> Proving WHO the user is

Example:

✔ Login with username/password  
✔ Verify token  
✔ Verify session  

Think:

Authentication = Identity check

---

### 3. Authorization — Permission Control

Authorization means:

> Deciding WHAT the user can do

Example:

✔ Can view profile  
✔ Can delete users  
✔ Can access admin panel  

Think:

Authorization = Access control

---

### 4. Critical Mental Model (Memorize This)

Authentication → Who are you?  
Authorization → What can you do?

---

## Hands-On Example — Real Scenario

---

### 5. Example System

User logs in:

Step 1 → Authenticate user  
Step 2 → Authorize actions  

---

## Authentication Deep Dive

---

### 6. Basic Login Flow

Client sends:

POST /login  
Body → Credentials

Server:

✔ Validate credentials  
✔ Verify identity  
✔ Issue proof (token/session)

---

### 7. Example Login Route (Simplified)

    app.post("/login", async (req, res) => {
      const { email, password } = req.body;

      const user = await findUserByEmail(email);

      if (!user) {
        return res.status(401).json({ error: "Invalid credentials" });
      }

      const valid = await comparePassword(password, user.password);

      if (!valid) {
        return res.status(401).json({ error: "Invalid credentials" });
      }

      res.json({ message: "Login successful" });
    });

---

### 8. Why Password Hashing Is Mandatory

Never store:

❌ Plain passwords

Always store:

✅ Hashed passwords

---

### 9. Hashing Example (bcrypt)

    const hashed = await bcrypt.hash(password, 10);

Verification:

    await bcrypt.compare(password, hashed);

---

## How Server Remembers Identity

---

### 10. Problem After Login

HTTP is stateless.

Server forgets user after response.

Solution needed.

---

## Two Common Authentication Strategies

---

### 11. Strategy 1 — Sessions (Stateful)

Server stores session.

Client stores session ID (cookie).

---

### 12. Strategy 2 — Tokens (Stateless)

Server issues token.

Client sends token on each request.

---

## Token-Based Authentication (Modern Standard)

---

### 13. JWT Concept

Server issues JWT:

Contains:

✔ User identity  
✔ Claims  
✔ Signature  

---

### 14. JWT Issuance Example

    const token = jwt.sign(
      { userId: user.id },
      process.env.JWT_SECRET,
      { expiresIn: "1h" }
    );

---

### 15. Client Sends Token

Headers:

Authorization: Bearer TOKEN

---

### 16. Middleware Verification Example

    const authMiddleware = (req, res, next) => {
      const header = req.headers.authorization;

      if (!header) {
        return res.status(401).json({ error: "No token provided" });
      }

      const token = header.split(" ")[1];

      try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = decoded;
        next();
      } catch (err) {
        res.status(401).json({ error: "Invalid token" });
      }
    };

---

### 17. Protected Route Example

    app.get("/profile", authMiddleware, (req, res) => {
      res.json({ user: req.user });
    });

---

## Authorization Deep Dive

---

### 18. Authorization Happens AFTER Authentication

Once identity known:

Decide permissions.

---

### 19. Role-Based Authorization

User roles:

✔ user  
✔ admin  
✔ moderator  

---

### 20. Authorization Middleware Example

    const requireAdmin = (req, res, next) => {
      if (req.user.role !== "admin") {
        return res.status(403).json({ error: "Access denied" });
      }

      next();
    };

---

### 21. Usage Example

    app.delete("/users/:id",
      authMiddleware,
      requireAdmin,
      controller.deleteUser
    );

---

### 22. Why Separation Matters

Authentication:

✔ Validates identity

Authorization:

✔ Validates permission

---

## Advanced Authorization Patterns

---

### 23. Permission-Based Authorization

Instead of roles:

Use permissions:

✔ can_delete_user  
✔ can_edit_product  

More flexible.

---

### 24. Ownership-Based Authorization

Example:

User can edit:

✔ Own profile  
❌ Others’ profiles

---

### 25. Example Ownership Check

    if (req.user.userId !== resource.ownerId) {
      return res.status(403).json({ error: "Forbidden" });
    }

---

## Common Security Mistakes

---

### 26. Typical Errors

❌ Trusting client data  
❌ Storing plain passwords  
❌ No hashing  
❌ No token verification  
❌ Skipping authorization checks  
❌ Using 200 for failures  
❌ Weak secrets  
❌ No expiration  
❌ Token leakage  
❌ No HTTPS  
❌ Hardcoded credentials  

---

## Critical HTTP Status Codes

---

### 27. Important Codes

401 → Not authenticated  
403 → Not authorized  

Difference:

401 = Who are you?  
403 = You are known but forbidden

---

## Advanced Production Insights

---

### 28. Token Expiration Is Mandatory

Never issue:

❌ Infinite tokens

Always:

✅ Expiring tokens

---

### 29. Refresh Tokens (Advanced Concept)

Short-lived access tokens.

Long-lived refresh tokens.

Improves security.

---

### 30. JWT Payload Caution

Never store:

❌ Sensitive secrets inside JWT

JWT visible to client.

---

### 31. Secure Secret Management

Use:

✅ Environment variables  
❌ Hardcoded secrets  

---

### 32. HTTPS Is Non-Negotiable

Without HTTPS:

❌ Tokens intercepted  

---

## Authentication vs Authorization Recap

---

### 33. Flow Diagram (Mental Model)

Request arrives  
   ↓  
Authentication middleware  
   ↓  
Authorization middleware  
   ↓  
Route handler  
   ↓  
Response  

---

## Professional Best Practices

---

### 34. Production-Grade Guidelines

✅ Hash passwords  
✅ Never store plain credentials  
✅ Use strong secrets  
✅ Use token/session expiration  
✅ Always verify identity  
✅ Always verify permissions  
✅ Separate auth & business logic  
✅ Use middleware  
✅ Protect sensitive routes  
✅ Validate ownership  
✅ Use HTTPS  
✅ Fail securely  
✅ Log security events  
✅ Rate-limit login endpoints  
✅ Protect against brute force  
✅ Sanitize inputs  

---

## Final Mental Model

Authentication = Identity proof  
Authorization = Permission decision  

Both are mandatory.

Missing authentication → Open system  
Missing authorization → Broken system
