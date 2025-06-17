Build a Frontend
```cmd
npm create vite@latest . -- --template react
npm install

# axios and react router
npm install axios react-router-dom

# tailwindcss
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
