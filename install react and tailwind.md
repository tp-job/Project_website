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

const db = new sqlite3.Database("./db.sqlite", (err) => {
  if (err) console.error(err.message);
  console.log("Connected to SQLite database.");
});

db.run(`
  CREATE TABLE IF NOT EXISTS files (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    filename TEXT,
    filepath TEXT,
    type TEXT
  )
`);

module.exports = db;

```

server.js
```js
const express = require("express");
const cors = require("cors");
const multer = require("multer");
const path = require("path");
const db = require("./database"); // นำเข้า database.js

const app = express();
app.use(cors());
app.use(express.json());
app.use("/uploads", express.static(path.join(__dirname, "uploads")));

const storage = multer.diskStorage({
  destination: "./uploads/",
  filename: (req, file, cb) => {
    cb(null, Date.now() + path.extname(file.originalname));
  },
});

const upload = multer({ storage });

// อัปโหลดไฟล์และบันทึกลงฐานข้อมูล
app.post("/upload", upload.single("file"), (req, res) => {
  const { filename, path: filepath, mimetype } = req.file;
  db.run(`INSERT INTO files (filename, filepath, type) VALUES (?, ?, ?)`,
    [filename, filepath, mimetype],
    function (err) {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ id: this.lastID, filename, filepath, type: mimetype });
    }
  );
});

// ดึงข้อมูลไฟล์ทั้งหมดจากฐานข้อมูล
app.get("/files", (req, res) => {
  db.all("SELECT * FROM files", [], (err, rows) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(rows);
  });
});

app.listen(5000, () => console.log("Server running on port 5000"));

```

---

Build a Frontend
```cmd
npm create vite@latest . -- --template react
npm install
npm install axios react-router-dom
npm install -D tailwindcss@3 postcss autoprefixer
```

vite.config.js
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vite.dev/config/
export default defineConfig({
  server: {
    host: true,  // เปิดให้เข้าถึงจากเครือข่าย
    port: 10005,  // กำหนดพอร์ตใหม่
  },
  plugins: [react()],
})

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
