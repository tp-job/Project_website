Build a Frontend
```cmd
npm create vite@latest . -- --template react
npm install
npm install axios react-router-dom

# old
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p

# new
npm install tailwindcss @tailwindcss/vite
```

vite.config.js
```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import tailwindcss from '@tailwindcss/vite'

// https://vite.dev/config/
export default defineConfig({
  server: {
    host: true,  // เปิดให้เข้าถึงจากเครือข่าย
    port: 10005,  // กำหนดพอร์ตใหม่
  },
  plugins: [
    react(),
    tailwindcss(),
  ],
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
# old
@tailwind base;
@tailwind components;
@tailwind utilities;

# new
@import "tailwindcss";
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
