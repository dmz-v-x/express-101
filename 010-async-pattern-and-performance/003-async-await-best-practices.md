## Async / Await Best Practices in Node.js & Express

## Understanding the Big Picture

---

### 1. First: What Problem Does async / await Solve?

Before async / await, we had:

❌ Callback hell  
❌ Nested logic  
❌ Hard-to-read code  

Example:

    doSomething(() => {
      doAnotherThing(() => {
        doMoreWork(() => {});
      });
    });

Messy.

---

### 2. What async / await Gives Us

Async / await allows:

✅ Cleaner syntax  
✅ Linear-looking async code  
✅ Easier reasoning  
✅ Better error handling  

---

### 3. Mental Model (Very Important)

async / await = Syntactic sugar over Promises

It **does NOT make code synchronous**.

It only makes it look synchronous.

---

## Absolute Basics

---

### 4. What Does async Mean?

`async` before a function means:

> This function returns a Promise

Example:

    async function test() {
      return "hello";
    }

Actually returns:

    Promise { "hello" }

---

### 5. What Does await Mean?

`await` means:

> Pause THIS function until Promise resolves

Example:

    const result = await fetchData();

Only pauses inside function.

Event loop continues working.

---

## Hands-On Example

---

### 6. Basic Async Route

    app.get("/user", async (req, res) => {
      const user = await getUserFromDB();
      res.json(user);
    });

Clean and readable.

---

## Critical Best Practices

---

## Best Practice 1 — Always Handle Errors

---

### 7. The Dangerous Mistake

    app.get("/data", async (req, res) => {
      const data = await fetchData(); // ❌ may throw
      res.json(data);
    });

If error occurs:

❌ App may crash  
❌ Unhandled promise rejection  

---

### 8. Correct Pattern

    app.get("/data", async (req, res, next) => {
      try {
        const data = await fetchData();
        res.json(data);
      } catch (err) {
        next(err);
      }
    });

Always wrap async logic.

---

## Best Practice 2 — Never Forget await

---

### 9. Common Beginner Bug

    const user = getUserFromDB(); // ❌ forgot await

Result:

user = Promise, NOT actual data.

---

### 10. Correct Version

    const user = await getUserFromDB();

---

## Best Practice 3 — Avoid Sequential Awaits When Possible

---

### 11. Slow Pattern (Bad Performance)

    const user = await getUser();
    const orders = await getOrders();
    const payments = await getPayments();

Runs one by one.

Slow.

---

### 12. Better Pattern (Parallel Execution)

    const [user, orders, payments] = await Promise.all([
      getUser(),
      getOrders(),
      getPayments()
    ]);

Huge performance improvement.

---

## Best Practice 4 — Use await Only When Needed

---

### 13. Over-Awaiting (Inefficient)

    return await someAsyncFunction();

Unnecessary.

---

### 14. Better

    return someAsyncFunction();

Avoid extra microtask overhead.

---

## Best Practice 5 — Keep Async Functions Focused

---

### 15. Bad Pattern

Massive async function doing everything:

❌ Hard to debug  
❌ Hard to test  

---

### 16. Better

Split logic:

✅ Small async functions  
✅ Clear responsibilities  

---

## Best Practice 6 — Avoid Blocking Code Inside Async Functions

---

### 17. Dangerous Illusion

async ≠ non-blocking automatically

This still blocks:

    async function test() {
      while(true) {} // ❌ blocks event loop
    }

---

## Best Practice 7 — Always Return After Sending Response

---

### 18. Common Bug

    if (!user) {
      res.status(404).send("Not found");
    }

    res.json(user); // ❌ second response

---

### 19. Correct Version

    if (!user) {
      return res.status(404).send("Not found");
    }

---

## Best Practice 8 — Validate Early

---

### 20. Clean Pattern

    if (!req.body.email) {
      return res.status(400).json({ error: "Email required" });
    }

Avoid wasted async work.

---

## Advanced Best Practices

---

## Best Practice 9 — Centralized Async Error Wrapper

---

### 21. Cleaner Pattern (Production Style)

    const asyncHandler = (fn) => (req, res, next) =>
      Promise.resolve(fn(req, res, next)).catch(next);

Usage:

    app.get("/data", asyncHandler(async (req, res) => {
      const data = await fetchData();
      res.json(data);
    }));

Removes try/catch repetition.

---

## Best Practice 10 — Be Careful with Large Promise.all()

---

### 22. Hidden Danger

Too many parallel tasks:

❌ DB overload  
❌ Memory pressure  

Use batching when needed.

---

## Best Practice 11 — Avoid Async Constructors

Bad pattern:

    class Service {
      async constructor() {} // ❌ invalid
    }

Use factory functions instead.

---

## Best Practice 12 — Avoid Fire-and-Forget in Routes

---

### 23. Dangerous Pattern

    sendEmail(); // ❌ not awaited
    res.send("OK");

Errors lost.

---

### 24. Safer Pattern

Either:

✅ await sendEmail()

Or:

✅ Queue background job

---

## Best Practice 13 — Timeouts for Async Work

---

### 25. Production Safety

External APIs may hang.

Use timeouts / abort controllers.

---

## Best Practice 14 — Avoid Mixing Callback & Await

---

### 26. Bad Pattern

    await fs.readFile("file", (err, data) => {}); // ❌

Use Promise-based APIs.

---

## Common Beginner Mistakes

---

### 27. Typical Errors

❌ Forgetting try/catch  
❌ Forgetting await  
❌ Awaiting sequentially unnecessarily  
❌ Blocking inside async  
❌ Sending multiple responses  
❌ Ignoring Promise.all  
❌ Fire-and-forget mistakes  
❌ Misunderstanding async nature  
❌ Assuming async = fast automatically  

---

## Professional Best Practices

---

### 28. Production-Grade Guidelines

✅ Always handle errors  
✅ Use await correctly  
✅ Prefer Promise.all when safe  
✅ Avoid unnecessary awaits  
✅ Keep functions small  
✅ Avoid blocking logic  
✅ Return after responses  
✅ Validate early  
✅ Use async wrappers  
✅ Avoid silent failures  
✅ Protect async calls with timeouts  
✅ Prefer Promise-based APIs  
✅ Design for concurrency  

---

## Final Mental Model

Async / Await:

Tool for readability & correctness

NOT automatic performance magic

Correct usage → Stable & scalable system  
Incorrect usage → Hidden production bugs
