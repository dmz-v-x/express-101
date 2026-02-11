## Difference Between Synchronous and Asynchronous Errors in Express

### 1. First: What Is an Error?

An error is simply:

> Something that prevents normal program execution

Examples:

- Invalid input
- Database failure
- Network failure
- Unexpected undefined value
- JSON parsing failure

Errors are unavoidable.

Handling them correctly is critical.

---

## Understanding Sync vs Async Execution

---

### 2. What Does "Synchronous" Mean?

Synchronous (sync) code:

> Runs line by line, immediately

Example:

    const x = 10;
    const y = x + 5;

Everything executes instantly.

---

### 3. What Does "Asynchronous" Mean?

Asynchronous (async) code:

> Runs later, not immediately

Example:

    setTimeout(() => {
      console.log("Runs later");
    }, 1000);

Execution is deferred.

---

## 4. Why This Matters for Errors

Errors behave differently depending on:

- When they occur
- Where they occur
- How Express captures them

This distinction is extremely important.

---

## Synchronous Errors

---

### 5. Example of a Sync Error

    app.get("/sync-error", (req, res) => {
      throw new Error("Something broke");
    });

What happens?

✔ Error occurs immediately  
✔ Express catches it automatically  
✔ Error middleware runs  

---

### 6. Why Express Catches Sync Errors Easily

Because sync code executes:

Inside Express's control flow.

Express wraps route handlers internally.

Thrown errors are captured.

---

## Asynchronous Errors

---

### 7. Example of an Async Error (Classic Problem)

    app.get("/async-error", (req, res) => {
      setTimeout(() => {
        throw new Error("Async failure");
      }, 1000);
    });

What happens?

❌ Express does NOT catch this  
❌ Server may crash  
❌ Error middleware NOT triggered  

---

### 8. Why Express Cannot Catch This

Because the error occurs:

Outside Express’s execution context.

The callback runs later.

Express is no longer in control.

---

## 9. Critical Insight (Memorize This)

> Express automatically catches sync errors  
> Express does NOT automatically catch async errors

This is a fundamental rule.

---

## Proper Async Error Handling

---

### 10. Correct Pattern — Using try/catch with async/await

    app.get("/safe-async", async (req, res, next) => {
      try {
        const data = await fetchData();
        res.json(data);
      } catch (err) {
        next(err);
      }
    });

Now Express can handle the error.

---

### 11. Why This Works

Because:

✔ Error is captured manually  
✔ Passed to Express using `next(err)`  
✔ Error middleware triggered  

---

## 12. Async/Await Error Behavior

Important rule:

> `await` errors behave like thrown sync errors  
> But still require try/catch

Without try/catch:

❌ Unhandled promise rejection  

---

### 13. Example Without try/catch (Bug)

    app.get("/bad-async", async (req, res) => {
      const data = await fetchData(); // may throw
      res.json(data);
    });

Failure → Unhandled rejection.

---

## Passing Errors to Express

---

### 14. The Role of `next(err)`

`next(err)` tells Express:

"An error occurred. Handle it."

Flow:

Async failure → next(err) → Error middleware

---

### 15. Universal Async Error Strategy

Always:

    try {
      await riskyOperation();
    } catch (err) {
      next(err);
    }

---

## Advanced Insight — Promise-Based Errors

---

### 16. Promise Rejection Example

    app.get("/promise-error", (req, res, next) => {
      Promise.reject(new Error("Failure"))
        .catch(next);
    });

Same principle.

---

## 17. Express Error Handling Pipeline

When `next(err)` is called:

Normal middleware skipped  
↓  
Error middleware executed  
↓  
Response sent  

---

## Advanced Pattern — Async Wrapper Utility

---

### 18. Avoid Repeating try/catch Everywhere

Reusable wrapper:

    const asyncHandler = (fn) => (req, res, next) =>
      Promise.resolve(fn(req, res, next)).catch(next);

Usage:

    app.get("/clean", asyncHandler(async (req, res) => {
      const data = await fetchData();
      res.json(data);
    }));

Cleaner and scalable.

---

## 19. Why Async Errors Are Dangerous

Unhandled async errors may:

❌ Crash server  
❌ Cause memory leaks  
❌ Break connections  
❌ Create unpredictable behavior  

---

## 20. Sync vs Async Error Comparison

| Aspect | Sync Errors | Async Errors |
|--------|-------------|--------------|
| Occur | Immediately | Later |
| Express Catches Automatically | Yes | No |
| Can Crash Server | Rarely | Very easily |
| Require Manual Handling | Usually no | Always yes |

---

## 21. Common Beginner Mistakes

❌ Assuming Express catches all errors  
❌ Throwing inside async callbacks  
❌ Forgetting try/catch  
❌ Forgetting next(err)  
❌ Ignoring promise rejections  
❌ Mixing sync & async incorrectly  

---

## 22. Professional Best Practices

✅ Always wrap async code  
✅ Always use try/catch with await  
✅ Always call next(err)  
✅ Never throw inside setTimeout / callbacks  
✅ Use asyncHandler wrappers  
✅ Handle promise rejections  
✅ Centralize error handling  

---

## 23. Final Mental Model

Sync error:

Occurs inside Express → Auto handled

Async error:

Occurs outside Express → Must manually forward

Think:

Timing determines error behavior.
