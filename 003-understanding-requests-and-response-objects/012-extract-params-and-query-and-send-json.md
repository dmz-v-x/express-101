## Extract Data from `req.params` & `req.query` and Respond with JSON

### Example Scenario

URL:

/users/42?active=true&role=admin

Meaning:

- `42` → user ID (URL param)
- `active=true` → filter (query)
- `role=admin` → filter (query)

---

### Express Route

    const express = require("express");
    const app = express();

    app.get("/users/:id", (req, res) => {
      const userId = req.params.id;
      const isActive = req.query.active;
      const role = req.query.role;

      res.status(200).json({
        userId: userId,
        active: isActive,
        role: role
      });
    });

    app.listen(3000, () => {
      console.log("Server running on port 3000");
    });

---

### What Happens Internally

Request arrives:

- URL param extracted from path → `req.params.id`
- Query values extracted from URL → `req.query`

Response sent:

- Status code → `200`
- Body → JSON object

---

### Example Response

    {
      "userId": "42",
      "active": "true",
      "role": "admin"
    }

⚠️ Note:

- All values are **strings**
- Convert manually if needed

---

### Optional: Type Conversion

    const userId = Number(req.params.id);
    const isActive = req.query.active === "true";

---

### Final Mental Model

- `req.params` → identify **which resource**
- `req.query` → control **how the resource is fetched**
- `res.json()` → return structured data
