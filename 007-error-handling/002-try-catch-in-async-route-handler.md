## Using `try...catch` in Async Route Handlers in Express


### 1. First: Why This Topic Matters

Modern Express apps frequently use:

- Databases
- APIs
- File systems
- Authentication services

All of these are **asynchronous operations**.

Async code introduces new error-handling challenges.

---

### 2. Recap: What Does "Async" Mean?

Async code runs:

Later, not immediately.

Example:

    const data = await fetchData();

Execution pauses until the promise resolves.

But:

❌ It may also fail.

---

### 3. What Happens When `await` Fails?

Example:

    const data = await fetchData();

If `fetchData()` fails:

✔ Promise rejects  
✔ Error is thrown  
✔ Route handler breaks  

Without protection:

❌ Unhandled error  
❌ Server instability  

---

## The Problem Without `try...catch`

---

### 4. Example — Unsafe Async Route

    app.get("/users", async (req, res) => {
      const users = await getUsersFromDatabase();
      res.json(users);
    });

If database fails:

❌ Error escapes  
❌ Express may not handle properly  
❌ Unhandled promise rejection  

---

### 5. Important Rule (Memorize This)

> Every async route must handle failures

Async operations are unreliable by nature.

---

## Introducing `try...catch`

---

### 6. Basic Safe Pattern

    app.get("/users", async (req, res, next) => {
      try {
        const users = await getUsersFromDatabase();
        res.json(users);
      } catch (err) {
        next(err);
      }
    });

This is the correct structure.

---

### 7. What `try...catch` Actually Does

`try` block:

✔ Runs risky async operations

`catch` block:

✔ Captures failures  
✔ Prevents crashes  
✔ Allows graceful handling  

---

### 8. Why `next(err)` Is Necessary

`next(err)` tells Express:

"An error occurred — invoke error middleware."

Flow:

Error → next(err) → Error Handler

---

## Hands-On Demonstration

---

### 9. Minimal Working Example

    const express = require("express");
    const app = express();

    app.get("/error-demo", async (req, res, next) => {
      try {
        throw new Error("Something failed");
      } catch (err) {
        next(err);
      }
    });

    app.use((err, req, res, next) => {
      res.status(500).json({
        error: err.message
      });
    });

    app.listen(3000);

Result:

✔ No crash  
✔ Clean error response  

---

## Why Not Just Throw Errors?

---

### 10. Common Beginner Misunderstanding

Beginners often write:

    throw new Error("Failure");

Inside async routes.

Problem:

Async errors behave differently from sync errors.

---

### 11. Why Throwing Alone Is Risky

Without `try...catch`:

❌ Promise rejection may be unhandled  
❌ Server warnings  
❌ Potential crashes  

Always catch async failures.

---

## Advanced Patterns

---

### 12. Early Return Inside try Block

    try {
      const user = await findUser();

      if (!user) {
        return res.status(404).json({
          error: "User not found"
        });
      }

      res.json(user);

    } catch (err) {
      next(err);
    }

Important:

✔ Business logic inside `try`  
✔ Errors captured cleanly  

---

### 13. Catching Specific Errors (Advanced Control)

    catch (err) {
      if (err.name === "ValidationError") {
        return res.status(400).json({ error: err.message });
      }

      next(err);
    }

Allows precision handling.

---

## Avoiding Repetitive try/catch

---

### 14. The Repetition Problem

Every async route:

Needs try/catch.

This becomes repetitive.

---

### 15. Async Wrapper Utility (Professional Pattern)

Reusable helper:

    const asyncHandler = (fn) => (req, res, next) =>
      Promise.resolve(fn(req, res, next)).catch(next);

Usage:

    app.get("/clean", asyncHandler(async (req, res) => {
      const data = await fetchData();
      res.json(data);
    }));

Cleaner & scalable.

---

## Performance & Stability Insights

---

### 16. Why Proper Error Handling Matters

Unhandled async errors may:

❌ Crash Node process  
❌ Break connections  
❌ Cause memory leaks  
❌ Produce inconsistent behavior  

`try...catch` improves stability.

---

## Common Beginner Mistakes

---

### 17. Typical Errors

❌ Missing try/catch  
❌ Forgetting next(err)  
❌ Sending response inside catch AND next(err)  
❌ Catching but ignoring errors  
❌ Mixing sync/async incorrectly  
❌ Overusing try/catch unnecessarily  

---

### 18. Incorrect Pattern Example

❌ Wrong:

    catch (err) {
      res.status(500).json({ error: err.message });
      next(err);
    }

Why wrong?

❌ Two responses  
❌ Headers already sent  

Correct:

Return OR next(err), not both.

---

## Professional Best Practices

---

### 19. Production Guidelines

✅ Always wrap await calls  
✅ Always use try/catch in async routes  
✅ Always forward errors using next(err)  
✅ Avoid duplicate responses  
✅ Use asyncHandler wrappers  
✅ Handle known errors explicitly  
✅ Centralize error handling  
✅ Keep business logic clean  

---

## Final Mental Model

Async route handler:

May succeed  
May fail  

`try...catch` = Safety mechanism

Think:

Await = Potential failure point

Without it → Fragile servers  
With it → Predictable, resilient systems
