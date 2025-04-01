Build a React app from Scratch
```cmd
https://react.dev/learn/build-a-react-app-from-scratch
```

Get started with Tailwind CSS
```cmd
https://tailwindcss.com/docs/installation/using-vite
```

---

Build a Backend
```cmd
npm init -y
npm install express sqlite3 cors body-parser jsonwebtoken bcryptjs
npm install --save-dev nodemon
```

Install package.json
```json
 "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  }
```

database.js
```js
const sqlite3 = require("sqlite3").verbose();
// เพิ่มข้อมูล admin เริ่มต้น
const bcrypt = require("bcryptjs");
const salt = bcrypt.genSaltSync(10);
const adminPassword = bcrypt.hashSync("admin123", salt);
// สร้างการเชื่อมต่อกับฐานข้อมูล
const db = new sqlite3.Database("bookings.db", (err) => {
  if (err) {
    console.error("เกิดข้อผิดพลาดในการเชื่อมต่อฐานข้อมูล:", err);
  } else {
    console.log("เชื่อมต่อฐานข้อมูลสำเร็จ");
    createTables();
  }
});

// สร้างตารางที่จำเป็น
const createTables = () => {
  // ตาราง users สำหรับเก็บข้อมูลผู้ใช้
  db.run(`
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT UNIQUE NOT NULL,
            password TEXT NOT NULL,
            role TEXT NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    `);

  // ตาราง bookings สำหรับเก็บข้อมูลการจอง
  db.run(`
        CREATE TABLE IF NOT EXISTS bookings (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            fullname TEXT NOT NULL,
            email TEXT NOT NULL,
            phone TEXT NOT NULL,
            checkin DATE NOT NULL,
            checkout DATE NOT NULL,
            roomtype TEXT NOT NULL,
            guests INTEGER NOT NULL,
            status TEXT DEFAULT 'pending',
            comment TEXT,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    `);

  db.run(
    `
        INSERT OR IGNORE INTO users (username, password, role)
        VALUES ('admin', ?, 'admin')
    `,
    [adminPassword]
  );
};

module.exports = db;
```

server.js
```js
const express = require("express");
const cors = require("cors");
const bodyParser = require("body-parser");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");
const db = require("./database");

const app = express();
const port = 3001; // เปลี่ยนเป็น port 3001
const JWT_SECRET = "your-secret-key"; // ในการใช้งานจริงควรเก็บไว้ใน environment variable

// Middleware
app.use(cors());
app.use(bodyParser.json());

// Middleware ตรวจสอบ token
const authenticateToken = (req, res, next) => {
  const authHeader = req.headers["authorization"];
  const token = authHeader && authHeader.split(" ")[1];

  if (!token) {
    return res.status(401).json({ error: "กรุณาเข้าสู่ระบบ" });
  }

  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) {
      return res.status(403).json({ error: "Token ไม่ถูกต้องหรือหมดอายุ" });
    }
    req.user = user;
    next();
  });
};

// Login endpoint
app.post("/api/login", async (req, res) => {
  const { username, password } = req.body;

  try {
    // ดึงข้อมูลผู้ใช้
    db.get(
      "SELECT * FROM users WHERE username = ?",
      [username],
      async (err, user) => {
        if (err) {
          return res.status(500).json({ error: err.message });
        }
        if (!user) {
          return res
            .status(401)
            .json({ error: "ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง" });
        }

        // ตรวจสอบรหัสผ่าน
        const validPassword = await bcrypt.compare(password, user.password);
        if (!validPassword) {
          return res
            .status(401)
            .json({ error: "ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง" });
        }

        // สร้าง token
        const token = jwt.sign(
          { id: user.id, username: user.username, role: user.role },
          JWT_SECRET,
          { expiresIn: "1h" }
        );

        res.json({
          token,
          user: {
            id: user.id,
            username: user.username,
            role: user.role,
          },
        });
      }
    );
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Booking endpoints
// สร้างการจองใหม่
app.post("/api/bookings", async (req, res) => {
  const { fullname, email, phone, checkin, checkout, roomtype, guests } =
    req.body;

  const sql = `INSERT INTO bookings (fullname, email, phone, checkin, checkout, roomtype, guests)
                 VALUES (?, ?, ?, ?, ?, ?, ?)`;

  db.run(
    sql,
    [fullname, email, phone, checkin, checkout, roomtype, guests],
    function (err) {
      if (err) {
        return res.status(400).json({ error: err.message });
      }

      db.get(
        "SELECT * FROM bookings WHERE id = ?",
        [this.lastID],
        (err, row) => {
          if (err) {
            return res.status(400).json({ error: err.message });
          }
          res.status(201).json(row);
        }
      );
    }
  );
});

// ดึงข้อมูลการจองทั้งหมด (ต้องมีข้อมูลการ login)
app.get("/api/bookings", authenticateToken, (req, res) => {
  const sql = "SELECT * FROM bookings ORDER BY created_at DESC";

  db.all(sql, [], (err, rows) => {
    if (err) {
      return res.status(400).json({ error: err.message });
    }
    res.json(rows);
  });
});

// ดึงข้อมูลการจองตาม ID (ต้องมีข้อมูลการ login)
app.get("/api/bookings/:id", authenticateToken, (req, res) => {
  const sql = "SELECT * FROM bookings WHERE id = ?";

  db.get(sql, [req.params.id], (err, row) => {
    if (err) {
      return res.status(400).json({ error: err.message });
    }
    if (!row) {
      return res.status(404).json({ error: "ไม่พบข้อมูลการจอง" });
    }
    res.json(row);
  });
});

// อัพเดตข้อมูลการจอง (ต้องมีข้อมูฃการ login)
app.put("/api/bookings/:id", authenticateToken, (req, res) => {
  const { fullname, email, phone, checkin, checkout, roomtype, guests } =
    req.body;

  const sql = `UPDATE bookings 
                 SET fullname = ?, email = ?, phone = ?, 
                     checkin = ?, checkout = ?, roomtype = ?, guests = ?
                 WHERE id = ?`;

  db.run(
    sql,
    [
      fullname,
      email,
      phone,
      checkin,
      checkout,
      roomtype,
      guests,
      req.params.id,
    ],
    function (err) {
      if (err) {
        return res.status(400).json({ error: err.message });
      }
      if (this.changes === 0) {
        return res.status(404).json({ error: "ไม่พบข้อมูลการจอง" });
      }

      db.get(
        "SELECT * FROM bookings WHERE id = ?",
        [req.params.id],
        (err, row) => {
          if (err) {
            return res.status(400).json({ error: err.message });
          }
          res.json(row);
        }
      );
    }
  );
});

// ลบข้อมูลการจอง (ต้องมีข้อมูการ login)
app.delete("/api/bookings/:id", authenticateToken, (req, res) => {
  const sql = "DELETE FROM bookings WHERE id = ?";

  db.run(sql, [req.params.id], function (err) {
    if (err) {
      return res.status(400).json({ error: err.message });
    }
    if (this.changes === 0) {
      return res.status(404).json({ error: "ไม่พบข้อมูลการจอง" });
    }
    res.status(204).send();
  });
});

// เริ่ม server
app.listen(port, () => {
  console.log(`Server กำลังทำงานที่ port ${port}`);
});
```

---

Build a Frontend
```cmd
npm create vite@latest . -- --template react
npm install
npm install axios react-router-dom
npm install -D tailwindcss@3 postcss autoprefixer
```

tailwind.config.js
```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

src/index.css
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

AppRoute.jsx
```jsx
import { BrowserRouter as Router, Routes, Route, Link } from "react-router-dom";

<Link to="/booking">

<Router>
  <Routes>
    <Route path="/" element={<HomePage />} />
  </Routes>  
</Router>
```
