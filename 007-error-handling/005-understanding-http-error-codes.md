## Understanding HTTP Error Code Categories — 2xx vs 3xx vs 4xx vs 5xx

### 1. First: What Is an HTTP Status Code?

When a server responds to a request, it sends:

- Status Code
- Headers
- Response Body

The **status code** answers:

> Did the request succeed, fail, or require further action?

---

### 2. Why Status Codes Exist

Status codes allow clients (browsers, apps, APIs) to quickly understand:

- Success
- Failure
- Redirection
- Server problems

Without reading the full body.

---

### 3. Big Picture — Status Code Families

HTTP status codes are grouped into categories:

| Range | Category |
|-------|----------|
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client Errors |
| 5xx | Server Errors |

Each family communicates a different type of outcome.

---

## 2xx — Success Responses

---

### 4. What 2xx Means

> The request was successfully received, understood, and processed

This is the "everything worked" category.

---

### 5. Common 2xx Codes

**200 OK**  
Standard success response.

Example:

GET /users → Data returned

---

**201 Created**  
Used when a new resource is created.

Example:

POST /users → User created

---

**204 No Content**  
Success, but nothing returned.

Example:

DELETE /users/42 → Successfully deleted

---

### 6. When to Use 2xx

Use 2xx when:

✔ Request is valid  
✔ Operation succeeded  
✔ No errors occurred  

---

## 3xx — Redirection Responses

---

### 7. What 3xx Means

> The client must take additional action to complete the request

Usually involves URL changes.

---

### 8. Common 3xx Codes

**301 Moved Permanently**  
Resource permanently moved.

Example:

Old URL → Redirect to new URL

---

**302 Found (Temporary Redirect)**  
Temporary redirection.

---

**304 Not Modified**  
Used with caching.

Meaning:

"Use cached version."

---

### 9. When 3xx Appears

Mostly relevant for:

- Browsers
- SEO
- Caching systems
- URL restructuring

Rare in pure APIs.

---

## 4xx — Client Error Responses

---

### 10. What 4xx Means

> The problem is caused by the client

Important concept:

Server is working correctly.

Client request is incorrect.

---

### 11. Common 4xx Codes

**400 Bad Request**  
Invalid request structure.

Example:

Missing required fields  
Malformed JSON  
Invalid syntax

---

**401 Unauthorized**  
Authentication required or failed.

---

**403 Forbidden**  
Client is authenticated but not allowed.

---

**404 Not Found**  
Requested resource does not exist.

---

**409 Conflict**  
Request conflicts with server state.

Example:

Duplicate resource

---

**422 Unprocessable Entity**  
Valid JSON, invalid business rules.

Example:

Invalid email format

---

### 12. When to Use 4xx

Use 4xx when:

✔ Client input is invalid  
✔ Resource missing  
✔ Auth issues  
✔ Validation failures  

Critical idea:

4xx = Client fault

---

## 5xx — Server Error Responses

---

### 13. What 5xx Means

> The failure is caused by the server

Client did nothing wrong.

Server could not complete request.

---

### 14. Common 5xx Codes

**500 Internal Server Error**  
Generic unexpected failure.

Example:

Unhandled exceptions  
Crashes  
Unknown errors

---

**502 Bad Gateway**  
Upstream server failure.

---

**503 Service Unavailable**  
Server overloaded or down.

---

### 15. When to Use 5xx

Use 5xx when:

✔ Database crashes  
✔ External services fail  
✔ Unexpected bugs  
✔ Infrastructure issues  

Critical idea:

5xx = Server fault

---

## Comparing 4xx vs 5xx (Very Important)

---

### 16. The Most Important Distinction

**4xx → Client did something wrong**  
**5xx → Server did something wrong**

Example:

Invalid input → 400  
Database failure → 500

---

### 17. Why This Distinction Matters

Correct classification improves:

✅ Debugging  
✅ Monitoring  
✅ Logging  
✅ Client behavior  
✅ Retry logic  
✅ API reliability  

---

## Practical Express Examples

---

### 18. Client Error Example

    if (!req.body.email) {
      return res.status(400).json({
        error: "Email required"
      });
    }

Correct:

400 = Client mistake

---

### 19. Server Error Example

    try {
      const data = await databaseCall();
      res.json(data);
    } catch (err) {
      res.status(500).json({
        error: "Database failure"
      });
    }

Correct:

500 = Server failure

---

## Advanced Insights

---

### 20. Why You Should Not Always Use 200

Incorrect:

    res.status(200).json({ error: "Something failed" });

Confusing:

✔ Status says success  
✔ Body says failure  

Causes bugs.

---

### 21. Status Code + Body Relationship

| Concern | Use |
|---------|------|
| Did it succeed? | Status Code |
| What happened? | Response Body |

Never mix meanings.

---

### 22. Retry Behavior & Status Codes

Clients often retry:

✔ 5xx errors → Server instability  
❌ 4xx errors → Client mistake  

Correct codes improve resilience.

---

### 23. Monitoring & Analytics Impact

Incorrect codes break:

❌ Error tracking  
❌ Metrics  
❌ Logs  
❌ Alerts  
❌ SLA monitoring  

Production systems depend on accuracy.

---

## Common Beginner Mistakes

---

### 24. Typical Errors

❌ Always returning 200  
❌ Using 500 for validation errors  
❌ Confusing 401 vs 403  
❌ Ignoring 201 for creation  
❌ Misusing 204  
❌ Inconsistent responses  

---

## Professional Best Practices

---

### 25. Production Guidelines

✅ Use correct status families  
✅ Distinguish client vs server errors  
✅ Use 201 for creation  
✅ Use 204 for successful deletions  
✅ Use 400/422 for validation failures  
✅ Use 401/403 correctly  
✅ Avoid ambiguous responses  
✅ Keep status & body consistent  
✅ Document status codes  
✅ Design APIs intentionally  

---

## Final Mental Model

2xx → Everything worked  
3xx → Go somewhere else  
4xx → Client mistake  
5xx → Server failure

Status code = Outcome classification
