## Graceful Shutdown in Node.js & Express (SIGTERM)

## Understanding the Big Picture

---

### 1. First: What Is Graceful Shutdown?

Graceful shutdown means:

> Stopping your server safely without abruptly killing active requests

Think:

❌ Bad shutdown → Instantly kill server  
✅ Graceful shutdown → Finish ongoing work → Exit safely  

---

### 2. Why This Matters in Real Systems

Servers are stopped frequently due to:

- Deployments
- Scaling events
- Container restarts
- Crashes / recoveries
- Infrastructure updates

Without graceful shutdown:

❌ Requests get cut mid-flight  
❌ Data corruption possible  
❌ Clients receive errors  
❌ Poor user experience  

---

### 3. Critical Mental Model

Production servers are **constantly starting and stopping**.

Shutdown behavior must be designed.

---

## What Happens Without Graceful Shutdown

---

### 4. Default Behavior (Dangerous)

If server is killed:

❌ Active requests terminated  
❌ DB operations interrupted  
❌ Files partially written  
❌ Connections dropped  

---

### 5. Real-World Disaster Example

User submits payment.

Server killed mid-request.

Possible results:

❌ Payment processed but response lost  
❌ Duplicate retry  
❌ Data inconsistency  

---

## Enter SIGTERM — The Shutdown Signal

---

### 6. What Is SIGTERM?

SIGTERM is:

> A signal sent to tell your app to stop

Common sources:

- Docker
- Kubernetes
- Process managers
- OS commands

---

### 7. Important Reality

Your server is **not magically shut down**.

It receives a signal.

You must decide what to do.

---

## Basic Graceful Shutdown Pattern

---

### 8. Minimal Express Server

    const express = require("express");
    const app = express();

    const server = app.listen(3000, () => {
      console.log("Server running");
    });

---

### 9. Listening for Shutdown Signal

    process.on("SIGTERM", () => {
      console.log("SIGTERM received");
    });

Now app is signal-aware.

---

## Step 1 — Stop Accepting New Requests

---

### 10. Proper Shutdown Logic

    process.on("SIGTERM", () => {
      console.log("Shutting down gracefully");

      server.close(() => {
        console.log("All requests finished");
        process.exit(0);
      });
    });

---

### 11. What server.close() Does

✅ Stops accepting new connections  
✅ Allows existing requests to finish  
✅ Calls callback when complete  

---

## Step 2 — Why This Is Critical

---

### 12. Behavior Difference

Without close():

❌ Instant kill

With close():

✅ Controlled drain

---

## Handling Long-Running Requests

---

### 13. The Hidden Problem

What if requests never finish?

Shutdown hangs forever.

Dangerous.

---

### 14. Add Safety Timeout

    process.on("SIGTERM", () => {
      console.log("Graceful shutdown started");

      server.close(() => {
        console.log("Shutdown complete");
        process.exit(0);
      });

      setTimeout(() => {
        console.error("Forced shutdown");
        process.exit(1);
      }, 5000);
    });

---

### 15. Why Timeout Is Necessary

Protects against:

❌ Stuck requests  
❌ Hanging DB queries  
❌ Broken clients  

---

## Cleaning Up Resources (Very Important)

---

### 16. Real Shutdown Must Cleanup

Examples:

✅ Close DB connections  
✅ Stop background jobs  
✅ Flush logs  
✅ Release file handles  
✅ Stop message queues  

---

### 17. Example DB Cleanup

    process.on("SIGTERM", async () => {
      console.log("Closing DB connection");

      await db.close();

      server.close(() => {
        process.exit(0);
      });
    });

---

## Advanced Production Insights

---

### 18. Containers & Orchestrators

Modern deployments:

Docker / Kubernetes:

✔ Send SIGTERM  
✔ Wait grace period  
✔ Send SIGKILL if needed  

---

### 19. Kubernetes Behavior (Important)

Flow:

SIGTERM → Grace period → SIGKILL

Your app must exit within grace window.

---

### 20. Load Balancer Interaction

During shutdown:

❌ New traffic must stop  
✅ Existing traffic drains  

Handled via:

✔ server.close()  
✔ Readiness probes  

---

## Tracking Active Connections (Advanced Pattern)

---

### 21. Why This Matters

Some connections persist:

- Keep-alive sockets
- WebSockets
- Slow clients

---

### 22. Connection Tracking Example

    const connections = new Set();

    server.on("connection", (conn) => {
      connections.add(conn);

      conn.on("close", () => {
        connections.delete(conn);
      });
    });

---

### 23. Force Close Stubborn Connections

    setTimeout(() => {
      connections.forEach(conn => conn.destroy());
    }, 5000);

Ensures exit.

---

## Why Graceful Shutdown Improves Reliability

---

### 24. Prevents Request Loss

Users do not experience random failures.

---

### 25. Prevents Data Corruption

Critical operations finish cleanly.

---

### 26. Supports Zero-Downtime Deployments

Old server drains → New server starts.

---

### 27. Improves System Stability

Predictable lifecycle behavior.

---

## Common Beginner Mistakes

---

### 28. Typical Errors

❌ Ignoring SIGTERM  
❌ Using process.exit() immediately  
❌ Not calling server.close()  
❌ No cleanup logic  
❌ No safety timeout  
❌ Forgetting DB close  
❌ Hanging shutdown  
❌ Assuming shutdown is rare  
❌ Killing app abruptly  

---

## Professional Best Practices

---

### 29. Production-Grade Guidelines

✅ Listen for SIGTERM  
✅ Stop accepting new requests  
✅ Allow active requests to finish  
✅ Close DB connections  
✅ Stop background tasks  
✅ Use shutdown timeout  
✅ Track open connections (advanced)  
✅ Log shutdown events  
✅ Exit cleanly  
✅ Align orchestrator grace periods  
✅ Test shutdown behavior  
✅ Design idempotent operations  

---

## Final Mental Model

Graceful shutdown = Landing a plane safely

Bad shutdown = Cutting engine mid-air

Production systems **depend heavily** on proper shutdown handling.
