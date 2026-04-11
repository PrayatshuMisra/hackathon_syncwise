# Expense Tracker Frontend — Complete Line-by-Line Technical Documentation

Welcome to the comprehensive technical reference for the Expense Tracker frontend. This document explains **every file, every function, and every line** in a way that assumes you know absolutely nothing about React or JavaScript. Read it top to bottom before the workshop and you will be ready to explain it all.

---

## 🔎 Table of Contents
1. [Foundations & Configuration](#-1-foundations--configuration)
2. [Global State Management (Context)](#-2-global-state-management-context)
3. [Core Entry Points](#-3-core-entry-points)
4. [Custom Hooks](#-4-custom-hooks)
5. [Reusable UI Components](#-5-reusable-ui-components)
6. [Visualizing Data (Charts)](#-6-visualizing-data-charts)
7. [Layouts](#-7-layouts)
8. [Feature Components (Dashboard, Expense, Income)](#-8-feature-components)
9. [Pages](#-9-pages)

---

## 📂 1. Foundations & Configuration

These files set up how the application talks to the backend server and provide helper functions used everywhere in the app.

---

### 📄 `apiPaths.js`

**Purpose**: A single "address book" that lists every URL the frontend will ever call on the backend. Instead of scattering URL strings all over the code, we put them all here so that if a URL ever changes, we only fix it in one place.

**Full Code with Explanation**:

```javascript
// Line 1
export const BASE_URL = import.meta.env.VITE_API_URL || "http://localhost:8000";
```
- `export` — Makes this variable available to other files that want to import it.
- `const` — Declares a constant: a variable whose value cannot be reassigned later.
- `BASE_URL` — The name we give to the root address of the backend server.
- `import.meta.env.VITE_API_URL` — Vite (our build tool) allows us to store sensitive or environment-specific values in a `.env` file. This reads a variable called `VITE_API_URL` from that file. When you deploy to a real server, this will hold the live domain name.
- `||` — The JavaScript "OR" operator. It means: "if the left side is empty/undefined/null, use the right side instead."
- `"http://localhost:8000"` — The fallback. During development on your computer, the backend runs here.

```javascript
// Lines 3–37
export const API_PATHS = {
  AUTH: {
    LOGIN: "/api/v1/auth/login",
    REGISTER: "/api/v1/auth/register",
    GET_USER_INFO: "/api/v1/auth/getUser",
    UPDATE_BUDGET: "/api/v1/auth/update-budget",
  },
  DASHBOARD: {
    GET_DATA: "/api/v1/dashboard",
  },
  AI: {
    GET_INSIGHTS: "/api/v1/ai/insights",
  },
  INCOME: {
    ADD_INCOME: "/api/v1/income/add",
    GET_ALL_INCOME: "/api/v1/income/get",
    DELETE_INCOME: (incomeId) => `/api/v1/income/${incomeId}`,
    UPDATE_INCOME: (incomeId) => `/api/v1/income/${incomeId}`,
    DOWNLOAD_INCOME: `/api/v1/income/downloadexcel`,
  },
  EXPENSE: {
    ADD_EXPENSE: "/api/v1/expense/add",
    GET_ALL_EXPENSE: "/api/v1/expense/get",
    DELETE_EXPENSE: (expenseId) => `/api/v1/expense/${expenseId}`,
    UPDATE_EXPENSE: (expenseId) => `/api/v1/expense/${expenseId}`,
    DOWNLOAD_EXPENSE: `/api/v1/expense/downloadexcel`,
  },
  IMAGE: {
    UPLOAD_IMAGE: "/api/v1/auth/upload-image",
  },
};
```
- `API_PATHS` is a JavaScript **object** — a collection of key-value pairs, like a dictionary.
- Inside `API_PATHS`, we have nested objects grouped by feature: `AUTH`, `DASHBOARD`, `AI`, `INCOME`, `EXPENSE`, `IMAGE`.
- Each key (like `LOGIN`) holds a **string** which is just the path after the base URL. So the full login URL becomes `http://localhost:8000/api/v1/auth/login`.
- `DELETE_INCOME: (incomeId) => \`/api/v1/income/${incomeId}\`` — This is an **arrow function**. Instead of a static string, it is a tiny function that takes an `incomeId` (a unique number for each record) and inserts it into the URL using a **template literal** (the backtick string with `${}`). This is needed because you need a different URL for each specific income entry you want to delete.

---

### 📄 `axiosInstance.js`

**Purpose**: Sets up a pre-configured "messenger" that handles all HTTP communication with the backend. We use a library called **Axios**, which makes it easier to send and receive data compared to the browser's built-in `fetch`.

```javascript
import axios from "axios";
import { BASE_URL } from "./apiPaths";
```
- `import axios from "axios"` — Brings in the Axios library (installed via npm).
- `import { BASE_URL } from "./apiPaths"` — Pulls in the server address we defined above. The `{ }` here is called **destructuring** — it picks only `BASE_URL` from the exports of that file.

```javascript
console.log("API Base URL:", BASE_URL);
```
- This prints the base URL to the browser's developer console. It helps developers confirm the correct server is being targeted. Not important for users.

```javascript
const axiosInstance = axios.create({
  baseURL: BASE_URL,
  timeout: 10000,
  headers: {
    "Content-Type": "application/json",
    Accept: "application/json",
  },
});
```
- `axios.create({})` — Creates a *custom copy* of Axios with preset settings.
- `baseURL: BASE_URL` — Every request made with this instance will automatically have `http://localhost:8000` (or the production URL) prepended. So you only write `/api/v1/auth/login` in your code, not the full URL.
- `timeout: 10000` — If the server doesn't respond within 10,000 milliseconds (10 seconds), the request is automatically cancelled. This prevents the app from hanging forever.
- `headers` — Extra information sent along with every request. `Content-Type: application/json` tells the server "I'm sending data in JSON format." `Accept: application/json` tells the server "I expect JSON back."

```javascript
axiosInstance.interceptors.request.use(
  (config) => {
    console.log("Making API request to:", config.url);
    const accessToken = localStorage.getItem("token");
    if (accessToken) {
      config.headers.Authorization = `Bearer ${accessToken}`;
    }
    return config;
  },
  (error) => {
    console.error("Request error:", error);
    return Promise.reject(error);
  }
);
```
- **Interceptors** are like checkpoints — pieces of code that run automatically before every request leaves the app, or after every response arrives.
- `interceptors.request.use(...)` — This runs **before** the request is sent.
- `(config)` — The `config` object contains all the settings for the outgoing request (URL, method, headers, etc.).
- `localStorage.getItem("token")` — Looks in the browser's permanent storage (LocalStorage) for a "token". A token is a string generated by the server after you log in — it proves you are who you say you are.
- `config.headers.Authorization = \`Bearer ${accessToken}\`` — If a token exists, it is attached to the outgoing request's header. The server reads this header to confirm your identity before responding. This is the **JWT (JSON Web Token)** authentication pattern.
- `return config` — Always return the config object, otherwise the request won't actually send.
- The second function `(error) => { ... }` runs if something goes wrong *while setting up* the request (rare).
- `Promise.reject(error)` — Passes the error along so the calling code can also handle it.

```javascript
axiosInstance.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    if (error.response) {
      console.error("API Error Response:", error.response);
      if (error.response.status === 401) {
        window.location.href = "/login";
      } else if (error.response.status === 500) {
        console.log("Server Error. Please try again later");
      }
    } else if (error.code === "ECONNABORTED") {
      console.log("Request timeout. Please try again.");
    } else {
      console.error("Network Error:", error.message);
    }
    return Promise.reject(error);
  }
);
```
- `interceptors.response.use(...)` — This runs **after** every response comes back from the server.
- The first function `(response) => { return response; }` handles successful responses. It simply passes them through unchanged.
- `error.response.status === 401` — HTTP status 401 means "Unauthorized" (your login session has expired or token is invalid). When this happens, we automatically redirect to `/login` using `window.location.href`.
- `error.response.status === 500` — HTTP status 500 means the server crashed. We just log it.
- `error.code === "ECONNABORTED"` — This error code specifically means the request timed out (took longer than 10 seconds).
- The else branch handles cases where there's no response at all (e.g., internet is disconnected).

```javascript
export default axiosInstance;
```
- Makes this configured instance available for import anywhere in the project.

---

### 📄 `helper.js`

**Purpose**: A toolbox of small, reusable functions that transform data — like formatting numbers, validating emails, or preparing chart data.

```javascript
import moment from "moment";
```
- `moment` is a popular JavaScript library for working with dates and times. It makes it easy to format dates like `"12th Apr 2026"`.

```javascript
export const validateEmail = (email) => {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
};
```
- `validateEmail` — A function that takes an email string and returns `true` or `false`.
- `const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/` — A **Regular Expression** (regex). It's a pattern-matching language. This specific pattern means:
  - `^` — start of string
  - `[^\s@]+` — one or more characters that are NOT a space or `@`
  - `@` — a literal `@` symbol
  - `[^\s@]+` — more characters (the domain name)
  - `\.` — a literal dot
  - `[^\s@]+$` — the extension (like `com`, `in`)
  - So it matches strings that look like `hello@example.com`.
- `regex.test(email)` — Runs the pattern against the provided email and returns `true` if it matches, `false` if not.

```javascript
export const getInitials = (name) => {
  if (!name) return "";
  const words = name.split(" ");
  let initials = "";
  for (let i = 0; i < Math.min(words.length, 2); i++) {
    initials += words[i][0];
  }
  return initials.toUpperCase();
};
```
- `getInitials("John Doe")` → returns `"JD"`.
- `if (!name) return ""` — If the name is empty or null, return an empty string (safety check).
- `name.split(" ")` — Breaks the name string into an array of words: `["John", "Doe"]`.
- `for (let i = 0; i < Math.min(words.length, 2); i++)` — A loop that runs at most 2 times. `Math.min(words.length, 2)` picks the smaller of 2 or the number of words, so it never crashes on single-word names.
- `words[i][0]` — Takes the first character (`[0]`) of each word.
- `initials += ...` — Appends each initial character to build the result string.
- `.toUpperCase()` — Converts to capital letters.

```javascript
export const addThousandsSeparator = (num) => {
  if (num == null || isNaN(num)) return "";
  return Number(num).toLocaleString("en-IN", {
    maximumFractionDigits: 2,
  });
};
```
- Formats a plain number into a readable currency string with commas.
- `if (num == null || isNaN(num)) return ""` — Guard clause: if the number is null or not a number at all, return an empty string instead of crashing.
- `Number(num)` — Makes sure we're working with a number type (converts strings like `"5000"` to the actual number `5000`).
- `.toLocaleString("en-IN", {...})` — JavaScript's built-in formatter. `"en-IN"` is the Indian locale, which formats `100000` as `"1,00,000"`. `maximumFractionDigits: 2` means at most 2 decimal places.

```javascript
export const prepareExpenseBarChartData = (data = []) => {
  const charData = data.map((item) => ({
    category: item?.category,
    amount: item?.amount,
  }));
  return charData;
};
```
- Converts raw expense data from the server into the shape the bar chart component expects.
- `data = []` — Default parameter: if `data` is not provided, treat it as an empty array.
- `.map((item) => ({...}))` — **Array map** creates a new array by transforming each item. For each expense item, it picks only the `category` and `amount` fields, discarding everything else.
- `item?.category` — The `?.` is the **optional chaining operator**. It safely accesses the property only if `item` is not null/undefined, preventing crashes.

```javascript
export const prepareIncomeBarChartData = (data = []) => {
  const sortedData = [...data].sort(
    (a, b) => new Date(a.date) - new Date(b.date)
  );
  const chartData = sortedData.map((item) => ({
    category: moment(item?.date).format("Do MMM"),
    amount: item?.amount,
  }));
  return chartData;
};
```
- Similar to above, but for income, and it also **sorts by date** first.
- `[...data]` — The **spread operator** creates a *copy* of the data array. We copy it because `.sort()` modifies the original array in place, and we don't want to mess with the original data.
- `.sort((a, b) => new Date(a.date) - new Date(b.date))` — Sorts oldest to newest. `new Date(a.date)` converts a date string into a JavaScript Date object that can be compared mathematically.
- `moment(item?.date).format("Do MMM")` — Formats the date as `"1st Jan"`, `"15th Mar"`, etc. This is what gets shown on the X-axis of the chart.

```javascript
export const prepareExpenseLineChartData = (data = []) => {
  const sortedData = [...data].sort(
    (a, b) => new Date(a.date) - new Date(b.date)
  );
  const charData = sortedData.map((item) => ({
    month: moment(item?.date).format("Do MMM"),
    amount: item?.amount,
    category: item?.category,
  }));
  return charData;
};
```
- Same idea as the income bar chart, but for the **line chart** on the Expense page. It includes the `category` field as well (needed for the chart tooltip).

---

### 📄 `data.js`

**Purpose**: Stores static (non-changing) data, specifically the configuration of the sidebar menu items.

```javascript
import {
  LuLayoutDashboard,
  LuHandCoins,
  LuWalletMinimal,
  LuLogOut,
} from "react-icons/lu";
```
- Imports four icon components from the **Lucide React** icon library. These are ready-made SVG icons you can use like normal React components.

```javascript
export const SIDE_MENU_DATA = [
  { id: "01", label: "Dashboard", icon: LuLayoutDashboard, path: "/dashboard" },
  { id: "02", label: "Income",    icon: LuWalletMinimal,   path: "/income"    },
  { id: "03", label: "Expense",   icon: LuHandCoins,       path: "/expense"   },
  { id: "06", label: "Logout",    icon: LuLogOut,          path: "/logout"    },
];
```
- An **array of objects**, where each object represents one menu item.
- `id` — A unique identifier for each item.
- `label` — The text shown in the sidebar button.
- `icon` — The icon component (stored as a value, not rendered yet — notice no `<>` angle brackets). It will be rendered later with `<item.icon />`.
- `path` — The URL that the app should navigate to when this button is clicked.

---

### 📄 `uploadImage.js`

**Purpose**: Handles uploading a profile photo to the server. Images can't be sent as JSON — they require a special format called `FormData`.

```javascript
import { API_PATHS } from "./apiPaths";
import axiosInstance from "./axiosInstance";
```
- Imports the upload URL and the preconfigured Axios messenger.

```javascript
const uploadImage = async (imageFile) => {
  const formData = new FormData();
  formData.append("image", imageFile);
```
- `async` — This function does something that takes time (uploading to a server), so it is **asynchronous**.
- `new FormData()` — Creates a special container that can hold files, not just text. This is the format web servers expect for file uploads.
- `formData.append("image", imageFile)` — Attaches the actual file to the form data under the key `"image"`. The server will look for a file field named `"image"`.

```javascript
  try {
    const response = await axiosInstance.post(
      API_PATHS.IMAGE.UPLOAD_IMAGE,
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }
    );
    return response.data;
  } catch (error) {
    console.error("Error uploading the image", error);
    throw error;
  }
};
```
- `await axiosInstance.post(...)` — Sends the image to the server and **waits** for the response.
- `"Content-Type": "multipart/form-data"` — Overrides the default `application/json` header because we're uploading a file, not JSON.
- `return response.data` — Returns the server's response (which will contain the URL of the uploaded image).
- `throw error` — Re-throws the error so the calling code (SignUp.jsx) also knows something went wrong.

```javascript
export default uploadImage;
```
- Exports the function as the default export of this file.

---

## 🌍 2. Global State Management (Context)

In React, **Context** solves the problem of "prop drilling" — passing data down through many layers of components. Instead, you put the data in a "global box" and any component can reach in and grab it directly.

---

### 📄 `UserContext.jsx`

**Purpose**: Stores the logged-in user's data (name, profile picture, etc.) so any component anywhere in the app can access it without being passed it manually.

```javascript
import React, { createContext, useState } from "react";
```
- `createContext` — A React function that creates a Context object (the "global box").
- `useState` — A React "hook" that lets a component remember a value between re-renders.

```javascript
export const UserContext = createContext();
```
- Creates the Context object and exports it. Other components import this to either read or provide data.

```javascript
const UserProvider = ({ children }) => {
  const [user, setUser] = useState(null);
```
- `UserProvider` is a React **component** that wraps the entire app.
- `{ children }` — In React, `children` is a special prop that refers to everything placed **inside** this component. So `<UserProvider><App /></UserProvider>` means `children` = `<App />`.
- `useState(null)` — The initial value of `user` is `null` (nobody is logged in yet). `user` holds the data, and `setUser` is the function used to change it.

```javascript
  const updateUser = (userData) => {
    setUser(userData);
  };

  const clearUser = () => {
    setUser(null);
  };
```
- `updateUser` — Called after a successful login or signup to store the user's info.
- `clearUser` — Called during logout to wipe the user data from memory.

```javascript
  return (
    <UserContext.Provider value={{ user, updateUser, clearUser }}>
      {children}
    </UserContext.Provider>
  );
};
```
- `<UserContext.Provider value={...}>` — This is the "broadcast tower". It makes the `user`, `updateUser`, and `clearUser` values available to every component nested inside it.
- `value={{ user, updateUser, clearUser }}` — The object passed here is what any child component receives when it "tunes in" using `useContext(UserContext)`.
- `{children}` — Renders whatever was wrapped inside `<UserProvider>...</UserProvider>`.

```javascript
export default UserProvider;
```
- Exports the Provider component as the default.

---

### 📄 `ThemeContext.jsx`

**Purpose**: Manages the visual theme of the entire app. Currently, the app is locked to **Dark Mode only**.

```javascript
import React, { createContext, useContext, useEffect, useState } from "react";

const ThemeContext = createContext();
```
- Creates the Theme Context box. Note it is NOT exported — it is only used inside this file.

```javascript
export const ThemeProvider = ({ children }) => {
  const [theme] = useState("dark");
```
- `const [theme] = useState("dark")` — Notice only one variable, not two. The `setTheme` function is intentionally left out because we don't want anyone to change the theme. Dark mode is permanently set.

```javascript
  useEffect(() => {
    const root = window.document.documentElement;
    root.classList.add("dark");
    localStorage.setItem("theme", "dark");
  }, []);
```
- `useEffect(() => {...}, [])` — Runs this code **once** when the component first mounts (appears on screen). The empty array `[]` is the "dependency array" — it tells React to only run this once, not every time something changes.
- `window.document.documentElement` — This is the `<html>` tag at the very root of your web page.
- `root.classList.add("dark")` — Adds the CSS class `"dark"` to `<html>`. Tailwind CSS uses this class to know when to apply dark-mode color variants (`dark:bg-slate-900`, etc.).
- `localStorage.setItem("theme", "dark")` — Saves the preference in the browser so it persists across page refreshes.

```javascript
  const toggleTheme = () => {
    console.warn("Theme toggle is disabled. Dark mode is the only available theme.");
  };
```
- A stub function. It exists so that other components can call `toggleTheme()` without crashing, but it doesn't actually do anything. A warning is logged to the console to explain why.

```javascript
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};
```
- Provides `theme` (which is always `"dark"`) and `toggleTheme` to any child component that wants them.

```javascript
export const useTheme = () => {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
};
```
- `useTheme` is a **custom hook** — a reusable function that wraps `useContext`.
- Instead of writing `const context = useContext(ThemeContext)` in every component, you just write `const { theme } = useTheme()`.
- The `if (!context)` guard ensures you get a helpful error message if you accidentally use `useTheme` outside a `<ThemeProvider>`.

---

## 🏗️ 3. Core Entry Points

These files start the entire React application and connect it to the HTML page.

---

### 📄 `main.jsx`

**Purpose**: The very first JavaScript file that runs. It "mounts" React onto the HTML page.

```javascript
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'
```
- `StrictMode` — A wrapper that activates extra warnings and checks during development. It has no effect in production.
- `createRoot` — The modern React 18 way to initialize a React app in the browser.
- `'./index.css'` — Loads the global stylesheet. No variable is needed; the import itself applies the CSS.
- `App` — The root component of our entire application.

```javascript
createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```
- `document.getElementById('root')` — Searches the HTML file for a `<div id="root"></div>` element. This `<div>` is the anchor point where all React content will be injected.
- `.render(...)` — Tells React to render (draw) the provided component tree inside that `<div>`.
- `<StrictMode><App /></StrictMode>` — Wraps the entire app in StrictMode so all warnings are surfaced.

---

### 📄 `App.jsx`

**Purpose**: The "Grand Central Station" of the app. It wraps everything in global providers and maps URL paths to specific page components.

```javascript
import React from "react";
import { BrowserRouter as Router, Route, Navigate, Routes } from "react-router-dom";
import Income from "./pages/Dashboard/Income";
import Home from "./pages/Dashboard/Home";
import SignUp from "./pages/Auth/SignUp";
import Login from "./pages/Auth/Login";
import Expense from "./pages/Dashboard/Expense";
import UserProvider from "./context/UserContext";
import { ThemeProvider } from "./context/ThemeContext";
import { Toaster } from "react-hot-toast";
```
- `BrowserRouter as Router` — Enables URL-based navigation. The `as Router` part renames `BrowserRouter` to `Router` for convenience.
- `Route, Navigate, Routes` — Components for defining page routes.
  - `Routes` — Container for all Route definitions.
  - `Route` — Maps one specific URL path to one component.
  - `Navigate` — Automatically redirects the user to a different URL.
- `Toaster` — The component that renders popup notifications (toast messages) on screen. You only need it once in the app.

```javascript
const App = () => {
  return (
    <ThemeProvider>
      <UserProvider>
        <div>
          <Router>
            <Routes>
              <Route path="/" element={<Root />} />
              <Route path="/login" exact element={<Login />} />
              <Route path="/signUp" exact element={<SignUp />} />
              <Route path="/dashboard" exact element={<Home />} />
              <Route path="/income" exact element={<Income />} />
              <Route path="/expense" exact element={<Expense />} />
            </Routes>
          </Router>
        </div>
        <Toaster
          toastOptions={{
            className: "",
            style: { fontSize: "13px" },
          }}
        />
      </UserProvider>
    </ThemeProvider>
  );
};
```
- `<ThemeProvider>` wraps everything so every component can access the theme.
- `<UserProvider>` wraps everything inside ThemeProvider so every component can access the user data.
- `<Router>` enables URL routing for everything inside it.
- Each `<Route path="..." element={<Component />} />` — When the browser URL matches `path`, React renders the corresponding `element`.
- `exact` — (though less critical in React Router v6) means the path must match exactly, not just partially.
- `<Toaster toastOptions={{...}}>` — Placed outside `<Router>` but inside `<UserProvider>`. It renders the notification container. `fontSize: "13px"` makes all toast messages slightly smaller text.

```javascript
export default App;
```
- Makes this component importable from `main.jsx`.

```javascript
const Root = () => {
  const isAuthenticated = !!localStorage.getItem("token");

  return isAuthenticated ? (
    <Navigate to="/dashboard" />
  ) : (
    <Navigate to="/login" />
  );
};
```
- `Root` is a "gatekeeper" component for the `/` URL (the home root).
- `localStorage.getItem("token")` — Looks in the browser's persistent storage for a login token.
- `!!` — The double exclamation mark converts any value to a strict boolean. `!!null` = `false`, `!!"sometoken"` = `true`.
- `isAuthenticated ? <Navigate to="/dashboard" /> : <Navigate to="/login" />` — A **ternary operator** (shorthand if/else). If authenticated → go to dashboard. If not → go to login.

---

### 📄 `index.css`

**Purpose**: The global stylesheet. Uses **Tailwind CSS** for utility classes and defines custom reusable CSS classes.

```css
@import url("https://fonts.googleapis.com/css2?family=Poppins:...");
@import "tailwindcss";
```
- The first line imports the **Poppins** font from Google Fonts and makes it available.
- `@import "tailwindcss"` — Loads the entire Tailwind CSS framework, making thousands of utility classes available (like `text-sm`, `bg-white`, `rounded-lg`, etc.).

```css
@theme {
  --font-display: "Poppins", "sans-serif";
  --breakpoint-3xl: 1920px;
  --color-primary: #875cf5;
}
```
- `@theme` is a Tailwind v4 syntax for defining design tokens (custom values).
- `--color-primary: #875cf5` — Defines our app's main purple colour. Anywhere you write `text-primary` or `bg-primary` in JSX, this exact colour is used.

```css
@layer base {
  html { font-family: var(--font-display); }
  body {
    background-color: #f0eeee;
    color: #1a1a1a;
    overflow-x: hidden;
    --chart-text-primary: #333;
    --chart-text-secondary: #666;
  }
  .dark body {
    background-color: #0b0f1a;
    color: #f9fafb;
    --chart-text-primary: #fff;
    --chart-text-secondary: #999;
  }
}
```
- `@layer base` — Tailwind's way of saying "these are base HTML element styles, apply them at the lowest priority."
- `font-family: var(--font-display)` — Applies Poppins to the entire page via the CSS variable defined above.
- `overflow-x: hidden` — Prevents horizontal scrollbars from appearing.
- `--chart-text-primary` / `--chart-text-secondary` — Custom CSS variables for chart label colours. They are redefined inside `.dark body` to switch to lighter colours when dark mode is active.

```css
.input-box {
  @apply w-full flex justify-between gap-3 text-sm text-black bg-slate-100 ...;
}
```
- `.input-box` — A custom CSS class we created.
- `@apply` — A Tailwind directive that takes utility class names and applies their styles into this custom class. This way, any element with `className="input-box"` gets all those styles without repeating them.

```css
.btn-primary { @apply w-full text-sm font-medium text-white bg-violet-500 ...; }
.card { @apply bg-white/70 p-6 rounded-2xl shadow-md ...; }
.card-btn { @apply flex items-center gap-3 text-[12px] ...; }
.add-btn { @apply flex items-center gap-1.5 text-xs ...; }
.add-btn-fill { @apply text-white bg-primary; }
```
- Each of these is a reusable custom class:
  - `.btn-primary` — The main submit button on forms (LOGIN, SIGN UP).
  - `.card` — The white/glass card with rounded corners and a shadow, used everywhere on the dashboard.
  - `.card-btn` — Small "See All" or "Download" buttons inside cards.
  - `.add-btn` — The outlined "Add Income" / "Add Expense" button.
  - `.add-btn-fill` — Combined with `.add-btn` to make a filled (solid background) version of the button.

---

## ⚓ 4. Custom Hooks

Custom Hooks are JavaScript functions whose name starts with `use`. They let you reuse stateful logic across components without copy-pasting.

---

### 📄 `useUserAuth.jsx`

**Purpose**: A "security guard" that every protected page calls. It checks whether the user's session is still valid by asking the server. If not, it logs them out.

```javascript
import { useContext, useEffect } from "react";
import { UserContext } from "../context/UserContext";
import { useNavigate } from "react-router-dom";
import axiosInstance from "../utils/axiosInstance";
import { API_PATHS } from "../utils/apiPaths";
```
- Standard imports. `useNavigate` is a React Router hook that gives us a `navigate` function to programmatically redirect the user.

```javascript
export const useUserAuth = () => {
  const { user, updateUser, clearUser } = useContext(UserContext);
  const navigate = useNavigate();
```
- `useContext(UserContext)` — "Tunes in" to the UserContext and grabs `user`, `updateUser`, and `clearUser`.
- `useNavigate()` — Gives us the `navigate` function. Calling `navigate("/login")` is the same as the user typing that URL in the browser bar.

```javascript
  useEffect(() => {
    if (user) return;
```
- `useEffect` runs after the component renders. The empty dependency array `[]` makes it run only once.
- `if (user) return` — **Early return / Optimization**: If we already have user data in context (because the user navigated from another page), don't bother fetching again.

```javascript
    let isMounted = true;

    const fetchUserInfo = async () => {
      console.log("Fetching user info...");
      try {
        const response = await axiosInstance.get(API_PATHS.AUTH.GET_USER_INFO);
        console.log("User info response:", response);

        if (isMounted && response.data) {
          updateUser(response.data);
        }
```
- `let isMounted = true` — A flag. If the component unmounts (user navigates away) while the request is still in-flight, we don't want to try updating state on an unmounted component (React would throw a warning).
- `async/await` — The `async` keyword makes the function return a Promise. `await` pauses execution inside the function until the Promise resolves (the server responds), without blocking the rest of the browser.
- `if (isMounted && response.data)` — Only update state if the component is still mounted AND we got actual data back.

```javascript
      } catch (error) {
        console.error("Failed to fetch user info:", error);
        console.error("User info fetch error details:", {
          message: error.message,
          code: error.code,
          response: error.response,
        });
        if (isMounted) {
          clearUser();
          navigate("/login");
        }
      }
    };
```
- If the request fails (401 unauthorized, network error, etc.), we:
  1. Log the details to the console for debugging.
  2. Call `clearUser()` to wipe any cached user data.
  3. Navigate to `/login` to force re-authentication.

```javascript
    fetchUserInfo();

    return () => {
      isMounted = false;
    };
  }, [user, updateUser, clearUser, navigate]);
};
```
- `fetchUserInfo()` — Call the async function we just defined.
- `return () => { isMounted = false; }` — This is the **cleanup function**. React calls this when the component unmounts. Setting `isMounted = false` prevents state updates on dead components.
- `[user, updateUser, clearUser, navigate]` — The **dependency array**. The effect re-runs if any of these values change. This is required by React's rules of hooks.

---

## 🧱 5. Reusable UI Components

---

### 📄 `Input.jsx`

**Purpose**: A universal, styled text field that supports all input types including a password field with a show/hide toggle.

```javascript
import React, { useState } from "react";
import { FaRegEye, FaRegEyeSlash } from "react-icons/fa6";
```
- `FaRegEye` / `FaRegEyeSlash` — Eye and crossed-eye icons from the FontAwesome icon set, used for the show/hide password toggle.

```javascript
const Input = ({ value, onChange, label, placeholder, type }) => {
```
- `Input` is a **functional component** that accepts **props** (properties passed to it from the parent).
- `value` — The current text in the field (controlled by the parent).
- `onChange` — A function the parent provides; called every time the user types a character.
- `label` — The text above the field (e.g., "Email Address").
- `placeholder` — The grey hint text shown when the field is empty.
- `type` — The input type: `"text"`, `"password"`, `"number"`, or `"date"`.

```javascript
  const [showPassword, setShowPassword] = useState(false);
  const toggleShowPassword = () => {
    setShowPassword(!showPassword);
  };
```
- `useState(false)` — `showPassword` starts as `false` (password is hidden by default).
- `toggleShowPassword` — Flips the boolean: `!showPassword`. If it's `false`, it becomes `true`, and vice versa.

```javascript
  return (
    <div>
      <label className="text-[13px] text-slate-800 dark:text-white/70">{label}</label>
      <div className="input-box">
        <input
          type={
            type == "password" ? (showPassword ? "text" : "password") : type
          }
```
- `type={...}` — Determines what the browser renders:
  - If `type` is `"password"` AND `showPassword` is true → set type to `"text"` (so you can see the characters).
  - If `type` is `"password"` AND `showPassword` is false → set type to `"password"` (shows dots).
  - Otherwise (text, number, date) → use `type` as-is.

```javascript
          placeholder={placeholder}
          className="w-full bg-transparent outline-none"
          value={value}
          onChange={(e) => onChange(e)}
        />
```
- `value={value}` — This makes the input a **controlled component**. React controls what's displayed in the field, not the browser. The value always reflects what's in the parent's state.
- `onChange={(e) => onChange(e)}` — Every keystroke triggers the browser's change event `e`. We pass this event up to the parent's `onChange` function, which updates the state.

```javascript
        {type === "password" && (
          <>
            {showPassword ? (
              <FaRegEye size={22} className="text-primary cursor-pointer"
                onClick={() => toggleShowPassword()} />
            ) : (
              <FaRegEyeSlash size={22} className="text-primary cursor-pointer"
                onClick={() => toggleShowPassword()} />
            )}
          </>
        )}
```
- `{type === "password" && (...)}` — **Conditional rendering**: this block only appears if the input type is `"password"`.
- `<>...</>` — **React Fragment**: a way to return multiple elements without adding an extra `<div>` to the DOM.
- The eye/eye-slash icon swaps based on `showPassword`. Clicking either icon calls `toggleShowPassword()`.

---

### 📄 `ProfilePhotoSelector.jsx`

**Purpose**: A component used on the Signup page that lets users pick a profile photo. It shows a preview immediately and lets the user remove the photo.

```javascript
import React, { useRef, useState } from "react";
import { LuUser, LuUpload, LuTrash } from "react-icons/lu";
```
- `useRef` — A React hook for directly referencing a DOM element (more on this below).

```javascript
const ProfilePhotoSelector = ({ image, setImage }) => {
  const inputRef = useRef(null);
  const [previewurl, setPreviewUrl] = useState(null);
```
- `image` — The current image file object (or `null` if none selected). Managed by the parent.
- `setImage` — Function from parent to update the image.
- `useRef(null)` — Creates a "direct wire" to a specific DOM element. We will use this to programmatically click the hidden file input.
- `previewurl` — A temporary URL for displaying the image before uploading.

```javascript
  const handleImageChange = (event) => {
    const file = event.target.files[0];
    if (file) {
      setImage(file);
      const preview = URL.createObjectURL(file);
      setPreviewUrl(preview);
    }
  };
```
- `event.target.files[0]` — `files` is an array of all selected files. `[0]` gets the first (and only) one.
- `setImage(file)` — Passes the actual File object up to the parent so it can be uploaded later.
- `URL.createObjectURL(file)` — Creates a temporary local URL (like `blob:http://localhost/abc123`) that the browser can use as an `<img src>` to show a preview instantly, without actually uploading anything yet.

```javascript
  const handleRemoveImage = () => {
    setImage(null);
    setPreviewUrl(null);
  };
```
- Resets both the file and the preview URL, effectively clearing the selection.

```javascript
  const onChooseFile = () => {
    inputRef.current.click();
  };
```
- `inputRef.current` — This is the actual DOM `<input>` element. `.click()` programmatically triggers a click on it, which opens the OS file picker dialog.
- We do this because the native `<input type="file">` is hidden (it's ugly), and we have a pretty custom button instead.

```javascript
  return (
    <div className="flex justify-center mb-6">
      <input
        type="file"
        accept="image/*"
        ref={inputRef}
        onChange={handleImageChange}
        className="hidden"
      />
```
- `accept="image/*"` — The file picker only shows image files.
- `ref={inputRef}` — Attaches the `inputRef` "wire" to this element.
- `className="hidden"` — Makes it invisible. Users interact with our custom button instead.

```javascript
      {!image ? (
        <div className="w-20 h-20 ... rounded-full relative">
          <LuUser className="text-4xl text-primary" />
          <button ... onClick={onChooseFile}><LuUpload /></button>
        </div>
      ) : (
        <div className="relative">
          <img src={previewurl} alt="profile photo" className="w-20 h-20 rounded-full object-cover" />
          <button ... onClick={handleRemoveImage}><LuTrash /></button>
        </div>
      )}
    </div>
  );
```
- `{!image ? (...) : (...)}` — **Conditional render**: If no image is selected, show the placeholder with an upload button. If an image is selected, show the preview with a remove button.
- `object-cover` — Tailwind class that makes the image fill the circle without stretching (it crops as needed).

---

### 📄 `CharAvatar.jsx`

**Purpose**: Shows a circular avatar with the user's initials when no profile photo is available. For example, "John Doe" → shows "JD" in a grey circle.

```javascript
import React from "react";
import { getInitials } from "../../utils/helper";

const CharAvatar = ({ fullName, width, height, style }) => {
  return (
    <div
      className={`${width || "w-12"} ${height || "h-12"} ${
        style || ""
      } flex items-center justify-center rounded-full text-gray-900 font-medium bg-gray-100`}
    >
      {getInitials(fullName || "")}
    </div>
  );
};

export default CharAvatar;
```
- `{ fullName, width, height, style }` — Props. The parent passes the user's name and optional sizing/style overrides.
- `${width || "w-12"}` — If the parent passes a `width` prop (e.g., `"w-20"`), use it. Otherwise, default to `"w-12"` (48px wide). Same logic for `height` and `style`.
- `getInitials(fullName || "")` — Calls our helper function to get initials from the name. The `|| ""` ensures we don't pass `null` or `undefined` to the function.
- `rounded-full` — Makes the `<div>` a perfect circle.

---

### 📄 `InfoCard.jsx`

**Purpose**: The large summary cards at the top of the dashboard showing Total Balance, Total Income, and Total Expense.

```javascript
const InfoCard = ({ icon, label, value, color }) => {
  return (
    <div className="flex gap-6 bg-white/70 p-6 rounded-2xl shadow-md ... border border-amber-500 ...">
      <div className={`w-14 h-14 flex items-center justify-center text-[26px] text-white ${color} rounded-full drop-shadow-xl`}>
        {icon}
      </div>
      <div>
        <h6 className="text-sm text-gray-700 ... mb-1">{label}</h6>
        <span className="text-[22px] text-gray-900 ... font-bold">
          ₹{value}
        </span>
      </div>
    </div>
  );
};
```
- `icon` — A React icon component (e.g., `<IoMdCard />`). Rendered inside a colored circle.
- `label` — Text like "Total Balance".
- `value` — The formatted number (e.g., `"1,00,000"`).
- `color` — A Tailwind background class like `"bg-primary"`, `"bg-orange-500"`, or `"bg-red-500"`. This is how all three cards look the same structurally but have different colours.
- `${color}` — Dynamically inserts the colour class using a **template literal**.
- `₹{value}` — The Rupee symbol followed by the value. JSX allows you to mix text and variables like this.

---

### 📄 `TransactionInfoCard.jsx`

**Purpose**: Each individual row in transaction lists. Shows the category/source, icon, date, amount, and edit/delete buttons that appear on hover.

```javascript
import { motion } from "framer-motion";
```
- `framer-motion` is an animation library. `motion.div` is a div that can be animated.

```javascript
const TransactionInfoCard = ({
  title, icon, date, amount, type,
  hideDeleteBtn, onDelete, onEdit,
}) => {
  const getAmountStyles = () =>
    type === "income" ? "bg-green-50 text-green-500" : "bg-red-50 text-red-500";
```
- `getAmountStyles` — A tiny function that returns CSS classes. Green badge for income, red for expense. Not actually used directly in the JSX below (the JSX duplicates this logic inline), but defined here for clarity.
- `type` — Either `"income"` or `"expense"`. Controls colours and icons.

```javascript
  return (
    <motion.div
      initial={{ opacity: 0, x: -10 }}
      animate={{ opacity: 1, x: 0 }}
      transition={{ duration: 0.3 }}
      className="group relative flex items-center gap-4 mt-2 p-3 rounded-lg ..."
    >
```
- `motion.div` — An animated `<div>`.
- `initial={{ opacity: 0, x: -10 }}` — Starts invisible (`opacity: 0`) and shifted 10px to the left.
- `animate={{ opacity: 1, x: 0 }}` — Animates to fully visible and normal position.
- `transition={{ duration: 0.3 }}` — The animation takes 0.3 seconds.
- `className="group ..."` — `group` is a Tailwind class that enables child elements to react to the parent being hovered. The edit/delete buttons use `group-hover:opacity-100` to appear on hover.

```javascript
      <div className="w-12 h-12 ... rounded-full">
        {icon ? (
          <img
            src={
              icon.startsWith("http")
                ? icon.replace("http://localhost:8000", import.meta.env.VITE_API_URL || "http://localhost:8000")
                : `${import.meta.env.VITE_API_URL || "http://localhost:8000"}${icon}`
            }
            alt={title}
            className="w-6 h-6"
          />
        ) : (
          <LuUtensils />
        )}
      </div>
```
- If an `icon` URL exists, it renders an `<img>`. If not, it shows a default utensils icon.
- The `src` logic handles two cases:
  - If the icon URL already starts with `"http"` (a full URL), it replaces the hardcoded `localhost` part with the environment variable, so it works on both local and production.
  - If it's a relative path (like `/uploads/emoji.png`), it prepends the base URL.

```javascript
      {onEdit && (
        <button
          className="... opacity-0 group-hover:opacity-100 transition-opacity cursor-pointer"
          onClick={onEdit}
        >
          <LuPencil size={18} />
        </button>
      )}
      {!hideDeleteBtn && (
        <button
          className="... opacity-0 group-hover:opacity-100 transition-opacity cursor-pointer"
          onClick={onDelete}
        >
          <LuTrash2 size={18} />
        </button>
      )}
```
- `{onEdit && (...)}` — Only renders the edit button if an `onEdit` function was passed as a prop.
- `{!hideDeleteBtn && (...)}` — Only renders the delete button if `hideDeleteBtn` is NOT true. On the dashboard's "Recent Transactions" panel, delete buttons are hidden (only shown on the full list pages).
- `opacity-0 group-hover:opacity-100` — The buttons are invisible by default and fade in when you hover over the parent card (the `group`).

```javascript
      <div className={`flex items-center gap-2 px-3 py-1.5 rounded-md ${
        type === "income"
          ? "bg-green-50 text-green-500 dark:bg-green-500/10 dark:text-green-400"
          : "bg-red-50 text-red-500 dark:bg-red-500/10 dark:text-red-400"
      }`}>
        <h6 className="text-xs font-medium">
          {type === "income" ? "+" : "-"} ₹{amount}
        </h6>
        {type === "income" ? <LuTrendingUp /> : <LuTrendingDown />}
      </div>
```
- Shows the amount badge. Green `+₹amount` with an up-arrow for income, red `-₹amount` with a down-arrow for expense.

---

### 📄 `DeleteAlert.jsx`

**Purpose**: A simple confirmation popup body used inside the `<Modal>` component when deleting a transaction.

```javascript
const DeleteAlert = ({ content, onDelete }) => {
  return (
    <div>
      <p className="text-sm text-gray-700 dark:text-gray-300">{content}</p>
      <div className="flex justify-end mt-6">
        <button
          type="button"
          className="add-btn add-btn-fill"
          onClick={onDelete}
        >
          Delete
        </button>
      </div>
    </div>
  );
};
```
- `content` — The confirmation message (e.g., "Are you sure you want to delete this expense detail?").
- `onDelete` — A function passed from the parent. When the "Delete" button is clicked, this runs the actual API delete call.
- `type="button"` — Prevents accidentally submitting any parent `<form>` element. Always good practice for non-submit buttons.

---

### 📄 `EmojiPickerPopup.jsx`

**Purpose**: Lets users pick an emoji/icon image when creating or editing an income/expense entry.

```javascript
import EmojiPicker from "emoji-picker-react";
import React, { useState } from "react";
import { LuImage, LuX } from "react-icons/lu";
```
- `emoji-picker-react` — A third-party library that provides a fully built emoji picker panel.

```javascript
const EmojiPickerPopup = ({ icon, onSelect }) => {
  const [isOpen, setIsOpen] = useState(false);
```
- `icon` — The currently selected icon URL (or empty if none).
- `onSelect` — Called with the selected emoji's image URL when the user picks one.
- `isOpen` — Boolean tracking whether the picker panel is visible.

```javascript
  return (
    <div className="flex flex-col md:flex-row items-start gap-5 mb-6">
      <div className="flex items-center gap-4 cursor-pointer" onClick={() => setIsOpen(true)}>
        <div className="w-12 h-12 ... bg-purple-50 rounded-lg">
          {icon ? (
            <img src={
              icon.startsWith("http")
                ? icon.replace("http://localhost:8000", import.meta.env.VITE_API_URL || "http://localhost:8000")
                : `${import.meta.env.VITE_API_URL || "http://localhost:8000"}${icon}`
            } alt="icon" className="w-12 h-12" />
          ) : (
            <LuImage />
          )}
        </div>
        <p>{icon ? "Change Icon" : "Pick Icon"}</p>
      </div>
```
- Clicking anywhere on this div sets `isOpen` to `true`, revealing the picker.
- Shows the current icon if one exists, or a placeholder image icon if not.
- The label text changes between "Pick Icon" and "Change Icon" based on whether one is already set.

```javascript
      {isOpen && (
        <div className="relative">
          <button
            className="w-7 h-7 ... rounded-full absolute -top-2 -right-2 z-10"
            onClick={() => setIsOpen(false)}
          >
            <LuX />
          </button>
          <EmojiPicker
            open={isOpen}
            onEmojiClick={(emoji) => onSelect(emoji?.imageUrl || "")}
          />
        </div>
      )}
    </div>
  );
};
```
- `{isOpen && (...)}` — Conditionally renders the picker panel.
- The `<LuX>` close button is positioned absolutely in the top-right corner of the panel.
- `onEmojiClick={(emoji) => onSelect(emoji?.imageUrl || "")}` — When a user clicks an emoji, the library fires this callback with an object. `emoji?.imageUrl` gets the image URL of the selected emoji. We pass this URL to the parent via `onSelect`.

---

### 📄 `Modal.jsx`

**Purpose**: A reusable, animated popup dialog used for forms (Add Expense, Add Income, Set Budget) and confirmations (Delete Alert).

```javascript
import { HiOutlineX } from "react-icons/hi";
import { motion, AnimatePresence } from "framer-motion";
```
- `AnimatePresence` — A special Framer Motion wrapper that allows components to animate when they are removed from the DOM (exit animations). Without it, exit animations don't work.

```javascript
const Modal = ({ children, isOpen, onClose, title }) => {
  return (
    <AnimatePresence>
      {isOpen && (
        <div className="fixed top-0 right-0 left-0 z-50 flex justify-center items-center w-full h-full overflow-y-auto overflow-x-hidden bg-black/40 backdrop-blur-sm ...">
```
- `isOpen` — Boolean that controls whether the modal is shown.
- `onClose` — Function called to close the modal (sets `isOpen` to false in the parent).
- `children` — The content inside the modal (e.g., `<AddExpenseForm />`).
- `fixed top-0 right-0 left-0` — Fixes the backdrop to cover the entire viewport.
- `z-50` — Places it on top of everything else (high z-index).
- `bg-black/40` — Semi-transparent black background (40% opacity). `backdrop-blur-sm` blurs the content behind the modal.

```javascript
          <motion.div
            initial={{ opacity: 0, scale: 0.95, y: 20 }}
            animate={{ opacity: 1, scale: 1, y: 0 }}
            exit={{ opacity: 0, scale: 0.95, y: 20 }}
            transition={{ duration: 0.3, ease: "easeOut" }}
            className="relative p-4 w-full max-w-2xl max-h-full"
          >
```
- `initial` — Starting state: invisible, slightly shrunk, slightly below.
- `animate` — End state: fully visible, full size, original position.
- `exit` — When `isOpen` becomes false, it plays the `exit` animation before disappearing.
- `ease: "easeOut"` — The animation decelerates towards the end, giving a natural feel.

```javascript
            <div className="relative bg-white/80 rounded-lg shadow-2xl backdrop-blur-xl ... border border-amber-500 ...">
              <div className="flex items-center justify-between p-4 ... border-b ...">
                <h3 className="text-lg font-medium text-gray-900 dark:text-white">{title}</h3>
                <button ... onClick={onClose}>
                  <HiOutlineX className="text-xl" />
                </button>
              </div>
              <div className="p-4 md:p-5 space-y-4">{children}</div>
            </div>
```
- `bg-white/80` — 80% opaque white background with `backdrop-blur-xl` giving it a frosted glass look.
- The header shows the `title` and an `×` close button.
- `{children}` — This is where the form content (e.g., `<AddExpenseForm />`) is rendered.

---

## 📊 6. Visualizing Data (Charts)

We use the **Recharts** library to render all charts. Recharts provides pre-built chart components as React components.

---

### 📄 `customTooltip.jsx`

**Purpose**: A custom styled tooltip that appears when you hover over a chart data point. Used by `CustomPieChart`.

```javascript
const customTooltip = ({ active, payload }) => {
  if (active && payload && payload.length) {
    return (
      <div className="bg-slate-900/90 backdrop-blur-md shadow-2xl rounded-lg p-3 border border-white/10">
        <p className="text-xs font-semibold text-purple-400 mb-1">
          {payload[0].name}
        </p>
        <p className="text-sm text-gray-300">
          Amount:{" "}
          <span className="text-sm font-semibold text-white">
            ₹{payload[0].value}
          </span>
        </p>
      </div>
    );
  }
  return <div>customTooltip</div>;
};
```
- `active` — A boolean provided by Recharts: `true` when the user is hovering over a data point.
- `payload` — An array of data objects provided by Recharts about the hovered point. `payload[0]` is the first (and usually only) data set.
- `payload[0].name` — The name of the slice (e.g., "Food", "Rent").
- `payload[0].value` — The numeric value of the slice (e.g., 5000).
- `{" "}` — A JSX trick to add a space character between "Amount:" and the value.
- If `active` is false (not hovering), returns a fallback div — this fallback text would never actually be seen by users.

---

### 📄 `customLegend.jsx`

**Purpose**: A custom styled legend that lists each chart category with its colour dot. Used by `CustomPieChart`.

```javascript
const customLegend = ({ payload }) => {
  return (
    <div className="flex flex-wrap justify-center gap-2 mt-4 space-x-6">
      {payload.map((entry, index) => (
        <div className="flex items-center space-x-2" key={`legend-${index}`}>
          <div
            className="w-2.5 h-2.5 rounded-full"
            style={{ backgroundColor: entry.color }}
          ></div>
          <span className="text-xs text-gray-700 font-medium">
            {entry.value}
          </span>
        </div>
      ))}
    </div>
  );
};
```
- `payload` — Recharts provides this array containing each legend item's `color` and `value` (name).
- `.map((entry, index) => (...))` — Loops through each legend item and renders a coloured dot and label for it.
- `style={{ backgroundColor: entry.color }}` — Inline style using the exact colour from the chart data.
- `key={`legend-${index}`}` — In React, when you render a list, each item needs a unique `key` prop so React can efficiently update only the items that changed.

---

### 📄 `CustomBarChart.jsx`

**Purpose**: Renders a bar chart where each bar represents a category (e.g., each income source or expense category).

```javascript
import { BarChart, Bar, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Cell } from "recharts";
```
- All these are Recharts components. Each one is a building block of the chart.

```javascript
const CustomBarChart = ({ data }) => {
  const getBarColor = (index) => {
    return index % 2 === 0 ? "#875cf5" : "#cfbefb";
  };
```
- `data` — Array of objects like `[{ category: "Food", amount: 500 }, ...]`.
- `getBarColor` — Returns a dark purple for even-indexed bars and light purple for odd-indexed bars. `index % 2 === 0` checks if the index is even.

```javascript
  const customTooltip = ({ active, payload }) => {
    if (active && payload && payload.length) {
      return (
        <div className="bg-white dark:bg-slate-800 shadow-md rounded-lg p-2 border ...">
          <p className="text-xs font-semibold text-purple-800 ... mb-1">
            {payload[0].payload.category}
          </p>
          <p className="text-sm text-gray-600 ...">
            Amount:
            <p className="text-sm font-semibold text-gray-900 ...">
              ₹{payload[0].payload.amount}
            </p>
          </p>
        </div>
      );
    }
    return null;
  };
```
- This is a locally defined tooltip (different from the one in `customTooltip.jsx` — this one is specific to bar charts).
- `payload[0].payload` — Recharts nests the original data item inside `payload.payload`. So we access the original `category` and `amount` this way.

```javascript
  return (
    <div className="bg-white dark:bg-slate-900 mt-6 transition-colors">
      <ResponsiveContainer width="100%" height={300}>
        <BarChart data={data}>
```
- `<ResponsiveContainer>` — Makes the chart fill its parent container's width and sets a fixed height of 300px.
- `<BarChart data={data}>` — The root chart component. `data` is the array of items to chart.

```javascript
          <CartesianGrid stroke="none" />
          <XAxis dataKey="category" tick={{ fontSize: 12, fill: "var(--chart-text-secondary)" }} stroke="none" />
          <YAxis dataKey="amount" tick={{ fontSize: 12, fill: "var(--chart-text-secondary)" }} stroke="none" />
```
- `<CartesianGrid stroke="none" />` — Disables the grid lines for a cleaner look.
- `<XAxis dataKey="category">` — The horizontal axis reads the `category` field from each data object.
- `<YAxis dataKey="amount">` — The vertical axis shows the amount values.
- `var(--chart-text-secondary)` — Uses the CSS variable defined in `index.css` to ensure labels are readable in both light and dark mode.

```javascript
          <Tooltip content={customTooltip} />
          <Bar dataKey="amount" fill="#FF8042" radius={[10, 10, 0, 0]} activeDot={{ r: 8, fill: "yellow" }} activeStyle={{ fill: "green" }}>
            {data.map((entry, index) => (
              <Cell key={index} fill={getBarColor(index)} />
            ))}
          </Bar>
```
- `<Tooltip content={customTooltip} />` — Uses our custom-styled tooltip on hover.
- `<Bar dataKey="amount" ...>` — Tells Recharts to draw bars based on the `amount` field.
- `radius={[10, 10, 0, 0]}` — Rounds only the top-left and top-right corners of each bar (CSS border-radius equivalent).
- `{data.map((entry, index) => <Cell key={index} fill={getBarColor(index)} />)}` — Each `<Cell>` overrides the colour of one specific bar. This is how we get alternating colours.

---

### 📄 `CustomLineChart.jsx`

**Purpose**: Renders a smooth area chart (filled line chart) showing spending trends over time.

```javascript
import { XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, Area, AreaChart } from "recharts";
```
- `AreaChart` — Like a line chart, but the area under the line is filled.
- `Area` — Defines the specific data line and its fill.

```javascript
  return (
    <div className="bg-white dark:bg-slate-900 transition-colors">
      <ResponsiveContainer width="100%" height={300}>
        <AreaChart data={data}>
          <defs>
            <linearGradient id="incomeGradient" x1="0" y1="0" x2="0" y2="1">
              <stop offset="5%" stopColor="#875cf5" stopOpacity={0.4} />
              <stop offset="95%" stopColor="#875cf5" stopOpacity={0} />
            </linearGradient>
          </defs>
```
- `<defs>` — SVG definitions block. Used to define reusable elements like gradients.
- `<linearGradient id="incomeGradient" x1="0" y1="0" x2="0" y2="1">` — Defines a gradient going from top (`y1="0"`) to bottom (`y2="1"`).
- `<stop offset="5%" stopColor="#875cf5" stopOpacity={0.4} />` — At 5% from the top, the purple colour has 40% opacity.
- `<stop offset="95%" stopColor="#875cf5" stopOpacity={0} />` — At 95% (near the bottom), the colour fades to fully transparent. This creates the "fade to nothing" effect.

```javascript
          <XAxis dataKey="month" tick={{ fontSize: 12, fill: "var(--chart-text-secondary)" }} stroke="none" />
          <YAxis tick={{ fontSize: 12, fill: "var(--chart-text-secondary)" }} stroke="none" />
          <Tooltip content={<CustomToolTip />} />
          <Area
            type="monotone"
            dataKey="amount"
            stroke="#875cf5"
            fill="url(#incomeGradient)"
            strokeWidth={3}
            dot={{ r: 3, fill: "#ab8df8" }}
          />
```
- `<XAxis dataKey="month">` — The X-axis uses the `month` field (formatted date string like "1st Apr").
- `type="monotone"` — Makes the line smooth and curved rather than jagged/sharp.
- `stroke="#875cf5"` — The colour of the line itself.
- `fill="url(#incomeGradient)"` — Uses the gradient defined in `<defs>` to fill the area beneath the line.
- `strokeWidth={3}` — The line is 3px thick.
- `dot={{ r: 3, fill: "#ab8df8" }}` — Small circles drawn at each data point (radius 3px, light purple).

---

### 📄 `CustomPieChart.jsx`

**Purpose**: A reusable donut chart that shows proportional breakdowns (e.g., expenses by category, income vs. expense vs. balance).

```javascript
import { PieChart, Pie, Cell, Tooltip, ResponsiveContainer, Legend } from "recharts";
import customTooltip from "./customTooltip";
import customLegend from "./customLegend";

const CustomPieChart = ({ data, label, totalAmount, colors, showTextAnchor }) => {
  return (
    <ResponsiveContainer width="100%" height={380}>
      <PieChart>
        <Pie
          data={data}
          dataKey="amount"
          nameKey="name"
          cx="50%"
          cy="50%"
          outerRadius={130}
          innerRadius={100}
          labelLine={false}
        >
```
- `dataKey="amount"` — Which field in each data object holds the numeric value.
- `nameKey="name"` — Which field holds the label.
- `cx="50%"` / `cy="50%"` — Center the pie horizontally and vertically.
- `outerRadius={130}` — The outer radius of the donut (in pixels).
- `innerRadius={100}` — The inner radius — creating the "hole" that makes it a donut chart instead of a pie chart.
- `labelLine={false}` — Disables the lines that usually connect slice labels to the slices.

```javascript
          {data.map((entry, index) => (
            <Cell key={`cell-${index}`} fill={colors[index % colors.length]} />
          ))}
        </Pie>
        <Tooltip content={customTooltip} />
        <Legend content={customLegend} />
```
- Each slice gets a colour from the `colors` array. `index % colors.length` wraps around if there are more slices than colours.
- `<Tooltip content={customTooltip} />` — Uses our styled dark tooltip from `customTooltip.jsx`.
- `<Legend content={customLegend} />` — Uses our custom legend from `customLegend.jsx`.

```javascript
        {showTextAnchor && (
          <>
            <text x="50%" y="50%" dy={-25} textAnchor="middle" fill="var(--chart-text-secondary)" fontSize="14px">
              {label}
            </text>
            <text x="50%" y="50%" dy={8} textAnchor="middle" fill="var(--chart-text-primary)" fontSize="24px" fontWeight="semi-bold">
              {totalAmount}
            </text>
          </>
        )}
```
- `showTextAnchor` — A boolean prop. When true, draws two `<text>` SVG elements inside the donut hole.
- `dy={-25}` / `dy={8}` — Vertical offsets relative to the center `y="50%"`. This stacks the two lines of text (label above, amount below).
- `textAnchor="middle"` — Centers the text horizontally around the `x` point.
- This renders something like "Total Expense" on one line and "₹50,000" on the next, right in the middle of the donut.

---

## 🖼️ 7. Layouts

Layouts define the overall page skeleton — where the navbar, sidebar, and content area go.

---

### 📄 `AuthLayout.jsx`

**Purpose**: The full-screen background layout used for the Login and Signup pages. It has the form on the left and decorative animated analytics graphics on the right.

```javascript
import { FiPieChart, FiTrendingUp, FiBarChart } from "react-icons/fi";

const AuthLayout = ({ children, bgImage }) => {
  return (
    <div
      className="flex relative min-h-screen bg-cover bg-center bg-no-repeat overflow-hidden text-slate-200 font-sans"
      style={{ backgroundImage: `url(${bgImage})` }}
    >
```
- `children` — The login or signup form passed from the page component.
- `bgImage` — The background image path (passed as `"/bg.png"` from the Login/Signup page).
- `style={{ backgroundImage: \`url(${bgImage})\` }}` — Inline style that dynamically sets the background image. Uses a template literal to build the CSS `url()` value.
- `bg-cover bg-center bg-no-repeat` — Tailwind classes ensuring the image covers the whole screen without repeating.

```javascript
      <div className="relative z-10 w-full flex flex-col md:flex-row h-screen">
        {/* Left Side: Auth Form */}
        <div className="w-full md:w-[45vw] lg:w-[35vw] px-8 sm:px-16 pt-16 pb-12 flex flex-col justify-center">
          <h2 className="text-2xl font-bold text-white mb-10 ... flex items-center gap-3">
            <FiBarChart className="text-purple-400" size={28} />
            Expense Tracker
          </h2>
          <div className="w-full max-w-sm">
            {children}
          </div>
        </div>
```
- `z-10` — Stacks this content above any background layers.
- `md:w-[45vw]` — On medium screens, the left panel takes 45% of the viewport width. On large screens (`lg:`), 35%.
- `{children}` — This is where the Login or Signup form appears.

```javascript
        {/* Right Side: Floating Analytics Elements — Pure decorative UI */}
        <div className="hidden md:flex flex-1 h-screen relative items-center justify-center ...">
```
- `hidden md:flex` — Hidden on small screens (mobile), visible as a flex container on medium and larger screens.

The right side contains three decorative widgets:
1. **Main Floating Card** — A fake dashboard card with an image showing charts.
2. **Circular Analytical Widget** (top-right) — A spinning ring with mini bars inside. All purely decorative; numbers are hardcoded.
3. **Bottom-Left Stat Card** — Uses `<StatInfoCard>`, defined below.

```javascript
              <div className="absolute inset-2 rounded-full border border-dashed border-white/20 animate-[spin_40s_linear_infinite]"></div>
```
- `animate-[spin_40s_linear_infinite]` — A custom Tailwind animation that makes this dotted ring rotate 360° over 40 seconds, continuously. Purely cosmetic.

```javascript
const StatInfoCard = ({ icon, label, value, color }) => {
  return (
    <div className="flex gap-4 items-center bg-slate-900/80 ... p-4 rounded-2xl shadow-2xl border border-white/10 min-w-[200px]">
      <div className={`w-12 h-12 flex items-center justify-center text-xl rounded-full shadow-lg ${color}`}>
        {icon}
      </div>
      <div>
        <h6 className="text-[11px] font-semibold text-slate-400 uppercase tracking-widest mb-1">{label}</h6>
        <span className="text-lg font-bold text-white tracking-wide">{value}</span>
      </div>
    </div>
  );
};
```
- This is a **locally defined component** (not exported). It's only used inside `AuthLayout` for the decorative bottom-left card.
- The value (`"+ 4,200.50"`) is hardcoded — this is fake data for display purposes only.

---

### 📄 `DashboardLayout.jsx`

**Purpose**: The standard skeleton for all protected pages (Dashboard, Income, Expense).

```javascript
import { useContext } from "react";
import { UserContext } from "../../context/UserContext";
import Navbar from "./Navbar";
import SideMenu from "./SideMenu";
import { motion, AnimatePresence } from "framer-motion";

const DashboardLayout = ({ children, activeMenu }) => {
  const { user } = useContext(UserContext);
```
- `useContext(UserContext)` — Gets the current user object. We need it to conditionally render the layout (don't show the sidebar if no user is loaded yet).
- `activeMenu` — A string like `"Dashboard"`, `"Income"`, or `"Expense"` passed from the page. This is forwarded to `Navbar` and `SideMenu` so the active item can be highlighted.

```javascript
  return (
    <div className="bg-[url('/bg.png')] bg-cover bg-fixed bg-center bg-no-repeat min-h-screen">
      <Navbar activeMenu={activeMenu} />

      {user && (
        <div className="flex">
          <div className="max-[1080px]:hidden">
            <SideMenu activeMenu={activeMenu} />
          </div>
          <motion.div
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            exit={{ opacity: 0, y: -20 }}
            transition={{ duration: 0.4 }}
            className="grow mx-5"
          >
            {children}
          </motion.div>
        </div>
      )}
    </div>
  );
};
```
- `bg-[url('/bg.png')]` — Tailwind's "arbitrary value" syntax for setting a custom background image.
- `bg-fixed` — The background image stays fixed while you scroll (parallax effect).
- `{user && (...)}` — **Conditional render**: Only shows the sidebar and content after the user data has loaded. Prevents layout flashing before auth is confirmed.
- `max-[1080px]:hidden` — Tailwind's arbitrary breakpoint. Hides the sidebar on screens narrower than 1080px (it's replaced by the mobile hamburger menu in `Navbar`).
- `<motion.div initial={{...}} animate={{...}} exit={{...}}>` — Page content slides up and fades in smoothly when you navigate between pages.
- `grow` — A flexbox utility that makes this div expand to fill all remaining horizontal space after the sidebar.

---

### 📄 `Navbar.jsx`

**Purpose**: The sticky top navigation bar. On desktop it shows the app title. On mobile it shows a hamburger menu button that opens the sidebar.

```javascript
const Navbar = ({ activeMenu }) => {
  const [openSideMenu, setOpenSideMenu] = useState(false);

  return (
    <div className="flex items-center justify-between bg-white/40 border-b ... backdrop-blur-md py-4 px-7 sticky top-0 z-30 ...">
```
- `sticky top-0 z-30` — The navbar "sticks" to the top of the viewport as you scroll. `z-30` ensures it overlaps page content.

```javascript
      <div className="flex items-center gap-5">
        <button
          className="block lg:hidden text-black dark:text-white"
          onClick={() => { setOpenSideMenu(!openSideMenu); }}
        >
          {openSideMenu ? <HiOutlineX className="text-2xl" /> : <HiOutlineMenu className="text-2xl" />}
        </button>
        <h2 className="text-lg font-medium text-black dark:text-white">Expense Tracker</h2>
      </div>
```
- `block lg:hidden` — The hamburger button is visible on all screen sizes except large (`lg:`) and above, where the sidebar is always visible.
- `setOpenSideMenu(!openSideMenu)` — Toggles the sidebar open/closed.
- `{openSideMenu ? <HiOutlineX /> : <HiOutlineMenu />}` — Shows an `×` icon when the menu is open, and a hamburger (`≡`) icon when closed.

```javascript
      {openSideMenu && (
        <div className="fixed top-[61px] -ml-4 bg-white dark:bg-slate-900 border-r ... h-screen">
          <SideMenu activeMenu={activeMenu} />
        </div>
      )}
```
- When `openSideMenu` is true, a dropdown version of the sidebar appears, fixed at `top-[61px]` (the height of the navbar) and full screen height.

---

### 📄 `SideMenu.jsx`

**Purpose**: The vertical navigation menu on the left side showing the user's avatar, name, and navigation buttons.

```javascript
const SideMenu = ({ activeMenu }) => {
  const { user, clearUser } = useContext(UserContext);
  const navigate = useNavigate();
```
- Gets the `user` (to display name/avatar) and `clearUser` (for logout).
- `useNavigate()` — Provides the `navigate` function for URL changes.

```javascript
  const handleClick = (route) => {
    if (route === "/logout") {
      handleLogout();
      return;
    }
    navigate(route);
  };

  const handleLogout = () => {
    localStorage.clear();
    clearUser();
    navigate("/login");
  };
```
- `handleClick` — Called when any menu button is clicked. It intercepts the `/logout` route specially; for all other routes, it just navigates.
- `handleLogout`:
  - `localStorage.clear()` — Wipes ALL data from the browser's local storage (removes the auth token).
  - `clearUser()` — Sets user to `null` in UserContext.
  - `navigate("/login")` — Sends the user to the login screen.

```javascript
  return (
    <div className="w-64 h-[calc(100vh-61px)] bg-white/40 ... sticky top-[61px] z-20 ...">
      <div className="flex flex-col items-center ... gap-3 mt-3 mb-7">
        {user?.profileImageUrl ? (
          <img
            src={
              user.profileImageUrl?.startsWith("http")
                ? user.profileImageUrl.replace("http://localhost:8000", import.meta.env.VITE_API_URL || "http://localhost:8000")
                : `${import.meta.env.VITE_API_URL || "http://localhost:8000"}${user.profileImageUrl}`
            }
            alt="Profile Image"
            className="w-20 h-20 bg-slate-400 rounded-full object-cover"
          />
        ) : (
          <CharAvatar fullName={user?.fullName} width="w-20" height="h-20" style="text-xl" />
        )}
        <h5 className="text-gray-950 dark:text-gray-100 font-medium leading-6">
          {user?.fullName || ""}
        </h5>
      </div>
```
- `h-[calc(100vh-61px)]` — Custom height: full viewport height minus 61px (the navbar height). The sidebar fills the rest of the screen below the navbar.
- `sticky top-[61px]` — Stays fixed to 61px from the top as you scroll, perfectly aligned below the navbar.
- Profile image URL handling (same logic as `TransactionInfoCard` and `EmojiPickerPopup`) ensures it works on both local and production environments.
- `user?.fullName` — Optional chaining; renders `""` if user is null.

```javascript
      <div className="space-y-2">
        {SIDE_MENU_DATA.map((item, index) => (
          <button
            key={`menu_${index}`}
            className={`w-full flex items-center gap-4 text-[15px] font-medium transition-all ${
              activeMenu === item.label
                ? "text-white bg-primary shadow-lg shadow-primary/20"
                : "text-gray-600 dark:text-gray-400 hover:bg-gray-50 dark:hover:bg-slate-800 hover:text-primary"
            } py-3 px-6 rounded-lg`}
            onClick={() => handleClick(item.path)}
          >
            <item.icon className="text-xl" />
            {item.label}
          </button>
        ))}
      </div>
```
- `SIDE_MENU_DATA.map((item, index) => (...))` — Loops through the four menu items defined in `data.js` and renders a `<button>` for each.
- `activeMenu === item.label` — Compares the current page name with each menu item's label. If they match, apply the highlighted style (purple background, white text). Otherwise, apply the normal greyed-out style.
- `<item.icon className="text-xl" />` — Renders the icon stored in the menu item object. This works because `item.icon` holds a React component (like `LuLayoutDashboard`), and JSX lets you render a component stored in a variable this way.

---

## 🚀 8. Feature Components

---

### 📄 `AIInsights.jsx`

**Purpose**: A dashboard card that fetches personalised financial advice from the Groq AI API (via the backend) and displays it with a loading skeleton.

```javascript
const AIInsights = () => {
  const [insights, setInsights] = useState("");
  const [loading, setLoading] = useState(false);

  const fetchInsights = async () => {
    setLoading(true);
    try {
      const response = await axiosInstance.get(API_PATHS.AI.GET_INSIGHTS);
      if (response.data?.insights) {
        setInsights(response.data.insights);
      }
    } catch (error) {
      console.error("Error fetching AI insights:", error);
      setInsights("Failed to fetch insights. Please ensure your Groq API Key is configured.");
    } finally {
      setLoading(false);
    }
  };
```
- `setLoading(true)` — Triggered at the start; causes the UI to show the shimmer skeleton instead of the button.
- `response.data?.insights` — Optional chaining to safely access the nested field.
- `finally { setLoading(false); }` — `finally` always runs, whether the request succeeded or failed. This ensures `loading` is always reset.

```javascript
  return (
    <div className="card">
      <div className="flex items-center justify-between mb-4">
        ...
        <button onClick={fetchInsights} disabled={loading} ... title="Refresh Insights">
          <LuRefreshCw size={20} className={loading ? "animate-spin" : ""} />
        </button>
      </div>

      <div className="min-h-[100px] flex flex-col justify-center">
        {loading ? (
          <div className="space-y-3">
            <div className="h-3 bg-gray-200 dark:bg-slate-700 rounded-full w-3/4 animate-pulse"></div>
            <div className="h-3 bg-gray-200 dark:bg-slate-700 rounded-full w-full animate-pulse"></div>
            <div className="h-3 bg-gray-200 dark:bg-slate-700 rounded-full w-5/6 animate-pulse"></div>
          </div>
        ) : insights ? (
          <div className="text-sm text-gray-800 dark:text-gray-200 leading-relaxed whitespace-pre-line">
            {insights}
          </div>
        ) : (
          <div className="text-center py-4">
            <p className="text-sm ...">Get personalized financial advice based on your recent activity.</p>
            <button onClick={fetchInsights} className="px-4 py-2 bg-purple-600 ... text-white rounded-lg ...">
              Generate Insights
            </button>
          </div>
        )}
      </div>
```
- `{loading ? (...) : insights ? (...) : (...)}` — A **nested ternary**: three states:
  1. Loading → show shimmer bars (`animate-pulse` makes them pulse in and out).
  2. Has insights → show the AI text.
  3. Neither → show the "Generate Insights" prompt button.
- `whitespace-pre-line` — Respects newline characters in the AI response text, so it displays in proper paragraphs.
- `disabled={loading}` — Prevents clicking the refresh button while a request is in progress.
- `className={loading ? "animate-spin" : ""}` — The refresh icon rotates while loading.

---

### 📄 `BudgetProgressCard.jsx`

**Purpose**: Shows a progress bar of how much of the monthly budget has been spent, with colour-coded warnings.

```javascript
const BudgetProgressCard = ({ totalExpenses, monthlyBudget, onSetBudget }) => {
  const progress = monthlyBudget > 0 ? (totalExpenses / monthlyBudget) * 100 : 0;
  const isOverBudget = progress > 100;
```
- `progress` — The percentage spent. If no budget is set (monthlyBudget = 0), avoid dividing by zero and default to 0.
- `isOverBudget` — True if spending exceeded 100% of the budget.

```javascript
  return (
    <div className="card">
      ...
      <button className="text-primary hover:underline text-sm font-medium" onClick={onSetBudget}>
        {monthlyBudget > 0 ? "Edit Budget" : "Set Budget"}
      </button>
      ...
      <div className="w-full bg-gray-200 dark:bg-slate-700 rounded-full h-4 overflow-hidden">
        <div
          className={`h-full rounded-full transition-all duration-500 ${
            isOverBudget ? "bg-red-500" : progress > 80 ? "bg-orange-500" : "bg-primary"
          }`}
          style={{ width: `${Math.min(progress, 100)}%` }}
        ></div>
      </div>
      <p className={`text-xs mt-3 ${isOverBudget ? "text-red-500 font-medium" : "text-gray-500 dark:text-gray-400"}`}>
        {isOverBudget
          ? `You've exceeded your budget by ₹${addThousandsSeparator(totalExpenses - monthlyBudget)}!`
          : monthlyBudget > 0
            ? `${Math.round(progress)}% of your monthly budget used.`
            : "No budget set for this month."}
      </p>
```
- The button label changes between "Set Budget" and "Edit Budget" depending on whether a budget exists.
- `transition-all duration-500` — The progress bar width changes smoothly over 0.5 seconds when data updates.
- `Math.min(progress, 100)%` — Caps the bar width at 100% even if spending exceeds the budget (the bar can't overflow its container).
- Nested ternaries for colour: red if over budget → orange if over 80% → purple (primary) if safe.
- The message below also uses nested ternaries to show the appropriate text for each state.

---

### 📄 `ExpenseCategoryChart.jsx`

**Purpose**: Groups expense data by category and displays it as a donut pie chart.

```javascript
const COLORS = ["#875CF5", "#FA2C37", "#FF6900", "#4f39f6", "#27ae60", "#f1c40f"];
```
- Six distinct colours for up to six categories. If there are more, they repeat.

```javascript
const ExpenseCategoryChart = ({ data }) => {
  const [chartData, setChartData] = useState([]);
  const [totalAmount, setTotalAmount] = useState(0);

  const prepareChartData = () => {
    const categoryMap = {};
    let total = 0;

    data?.forEach((item) => {
      const category = item.category || "Other";
      categoryMap[category] = (categoryMap[category] || 0) + item.amount;
      total += item.amount;
    });
```
- `categoryMap` — A plain JavaScript object used as a dictionary. Keys are category names, values are summed amounts.
- `(categoryMap[category] || 0) + item.amount` — If this category already exists in the map, add to it. Otherwise, start from 0 and add the current amount.
- This aggregates multiple "Food" entries into one total "Food" entry.

```javascript
    const formattedData = Object.keys(categoryMap).map((key) => ({
      name: key,
      amount: categoryMap[key],
    }));

    setChartData(formattedData);
    setTotalAmount(total);
  };
```
- `Object.keys(categoryMap)` — Gets an array of all category names from the map object.
- `.map((key) => ({...}))` — Converts each key into an object with `name` and `amount` fields, which is the shape `CustomPieChart` expects.

```javascript
  useEffect(() => {
    prepareChartData();
  }, [data]);
```
- `[data]` in the dependency array means: re-run `prepareChartData()` every time the `data` prop changes (i.e., when new expenses are loaded).

---

### 📄 `ExpenseTransactions.jsx`

**Purpose**: Shows the 5 most recent expense transactions on the dashboard (read-only, no delete buttons).

```javascript
const ExpenseTransactions = ({ transactions, onSeeMore }) => {
  return (
    <div className="card">
      ...
      <button className="card-btn" onClick={onSeeMore}>
        See All <LuArrowRight className="text-base" />
      </button>
      ...
      {transactions?.slice(0, 5)?.map((expense) => (
        <TransactionInfoCard
          key={expense._id}
          title={expense.category}
          icon={expense.icon}
          date={moment(expense.date).format("Do MMM YYYY")}
          amount={expense.amount}
          type="expense"
          hideDeleteBtn
        />
      ))}
```
- `transactions?.slice(0, 5)` — Optional chaining + `slice`: safely takes the first 5 items from the array (or fewer if there are less than 5). Won't crash if `transactions` is undefined.
- `moment(expense.date).format("Do MMM YYYY")` — Formats a date like `"2025-04-10T00:00:00.000Z"` into `"10th Apr 2025"`.
- `key={expense._id}` — The `_id` is a unique MongoDB ID for each record. React requires a unique `key` on each item in a rendered list.
- `hideDeleteBtn` — Passed as a prop to `TransactionInfoCard` to hide the delete button (this is a read-only view).

---

### 📄 `RecentTransactions.jsx`

**Purpose**: Shows up to 5 recent transactions (both income and expenses) on the main dashboard.

```javascript
{transactions?.slice(0, 5)?.map((item) => (
  <TransactionInfoCard
    key={item._id}
    title={item.type == "expense" ? item.category : item.source}
    icon={item.icon}
    date={moment(item.date).format("Do MMM YYYY")}
    amount={item.amount}
    type={item.type}
    hideDeleteBtn
  />
))}
```
- `title={item.type == "expense" ? item.category : item.source}` — Expenses use a `category` field for the title; income entries use a `source` field. This ternary picks the right one.

---

### 📄 `FinanceOverview.jsx`

**Purpose**: A pie chart showing the proportional breakdown of Total Balance, Total Income, and Total Expense.

```javascript
const COLORS = ["#875CF5", "#FA2C37", "#FF6900"];

const FinanceOverview = ({ totalBalance, totalIncome, totalExpense }) => {
  const balanceData = [
    { name: "Total Balance", amount: totalBalance },
    { name: "Total Expense", amount: totalExpense },
    { name: "Total Income",  amount: totalIncome  },
  ];

  return (
    <div className="card">
      <h5 ...>Financial Overview</h5>
      <CustomPieChart
        data={balanceData}
        label="Total Balance"
        totalAmount={`₹${totalBalance}`}
        colors={COLORS}
        showTextAnchor
      />
    </div>
  );
};
```
- The `balanceData` array is constructed directly from the props — no async calls needed.
- `showTextAnchor` — Passed without a value, which means it's `true`. Renders "Total Balance" and the amount in the donut hole.

---

### 📄 `Last30DaysExpenses.jsx`

**Purpose**: Shows a bar chart of the last 30 days of expenses on the dashboard.

```javascript
const Last30DaysExpenses = ({ data }) => {
  const [charData, setCharData] = useState([]);

  useEffect(() => {
    const result = prepareExpenseBarChartData(data);
    setCharData(result);
    return () => { };
  }, [data]);

  return (
    <div className="card col-span-1">
      <h5 ...>Last 30 Days Expenses</h5>
      <CustomBarChart data={charData} />
    </div>
  );
};
```
- Calls `prepareExpenseBarChartData` (from `helper.js`) to reshape the data, then passes it to `CustomBarChart`.
- `return () => { }` — An empty cleanup function. Not strictly necessary here, but a good habit to include one in `useEffect`.

---

### 📄 `RecentIncome.jsx`

**Purpose**: Shows up to 5 recent income transactions on the dashboard.

```javascript
{transactions?.slice(0, 5)?.map((item) => (
  <TransactionInfoCard
    key={item._id}
    title={item.source}
    icon={item.icon}
    date={moment(item.date).format("Do MMM YYYY")}
    amount={item.amount}
    type="income"
    hideDeleteBtn
  />
))}
```
- Nearly identical to `ExpenseTransactions` but `type="income"` and `title={item.source}`.

---

### 📄 `RecentIncomeWithChart.jsx`

**Purpose**: Shows the last 60 days of income as a donut chart, breaking it down by income source.

```javascript
const COLORS = ["#875CF5", "#FA2C37", "#FF6900", "#4f39f6"];

const RecentIncomeWithChart = ({ data, totalIncome }) => {
  const [charData, setCharData] = useState([]);

  const prepareCharData = () => {
    const dataArr = data?.map((item) => ({
      name: item?.source,
      amount: item?.amount,
    }));
    setCharData(dataArr);
  };

  useEffect(() => {
    prepareCharData();
    return () => { };
  }, [data]);
```
- Transforms the raw income data into the `{ name, amount }` format that `CustomPieChart` expects.
- `item?.source` — The income source name (e.g., "Freelance", "Salary").

---

### 📄 `AddExpenseForm.jsx`

**Purpose**: The form inside the "Add/Edit Expense" modal. A **controlled component** that tracks every field in React state.

```javascript
const AddExpenseForm = ({ onAddExpense, data }) => {
  const [expense, setExpense] = useState({
    category: data?.category || "",
    amount:   data?.amount   || "",
    date:     data?.date ? data.date.split("T")[0] : "",
    icon:     data?.icon     || "",
  });
```
- `data` — If the form is opened in "edit" mode, the existing expense's data is passed here. Otherwise it's `null`/`undefined`.
- `data?.category || ""` — If editing, pre-fill with existing data. If adding new, start empty.
- `data.date.split("T")[0]` — The date from the server looks like `"2025-04-10T00:00:00.000Z"`. `.split("T")[0]` takes only the `"2025-04-10"` part (the format HTML date inputs expect).

```javascript
  const handleChange = (key, value) => setExpense({ ...expense, [key]: value });
```
- `{ ...expense, [key]: value }` — The **spread operator** creates a new object with all existing fields from `expense`, then overrides just the one field identified by `key`.
- `[key]` — **Computed property name**: if `key` is `"amount"`, this becomes `{ ...expense, amount: value }`. This lets one function handle all fields instead of writing a separate handler for each.

```javascript
  return (
    <div>
      <EmojiPickerPopup icon={expense.icon} onSelect={(selectedIcon) => handleChange("icon", selectedIcon)} />
      <Input value={expense.category} onChange={({ target }) => handleChange("category", target.value)} label="Category" placeholder="Rent, Groceries,etc" type="text" />
      <Input value={expense.amount}   onChange={({ target }) => handleChange("amount", target.value)}   label="Amount"   placeholder="" type="number" />
      <Input value={expense.date}     onChange={({ target }) => handleChange("date", target.value)}     label="Date"     placeholder="" type="date"   />

      <div className="flex justify-end mt-6">
        <button className="add-btn add-btn-fill" type="button" onClick={() => onAddExpense(expense)}>
          {data ? "Update Expense" : "Add Expense"}
        </button>
      </div>
    </div>
  );
};
```
- Each `<Input>` component is given the current value from state and an `onChange` handler.
- `onChange={({ target }) => ...}` — Destructures the event object to get `target` (the input element) directly.
- `onClick={() => onAddExpense(expense)}` — When submitted, calls the parent's `onAddExpense` function with the entire form state object.
- `{data ? "Update Expense" : "Add Expense"}` — Button text changes based on whether we're editing or adding.

---

### 📄 `AddIncomeForm.jsx`

Identical structure to `AddExpenseForm`, but for income. Uses `source` instead of `category`.

---

### 📄 `ExpenseList.jsx`

**Purpose**: Displays all expense transactions in a two-column grid with edit and delete buttons.

```javascript
const ExpenseList = ({ transactions, onDelete, onDownload, hideHeader, onEdit }) => {
  return (
    <div className="card">
      {!hideHeader && (
        <div className="flex items-center justify-between">
          <h5 ...>Expense Log</h5>
          <button className="card-btn" onClick={onDownload}>
            <LuDownload className="text-base" /> Download
          </button>
        </div>
      )}
      <div className="grid grid-cols-1 md:grid-cols-2">
        {transactions?.map((expense) => (
          <TransactionInfoCard
            key={expense._id}
            title={expense.category}
            icon={expense.icon}
            date={moment(expense.date).format("Do MMM YYYY")}
            amount={expense.amount}
            type="expense"
            onDelete={() => onDelete(expense._id)}
            onEdit={() => onEdit(expense)}
          />
        ))}
      </div>
    </div>
  );
};
```
- `{!hideHeader && (...)}` — The header (title + download button) is hidden when `hideHeader` is true. On the Expense page, the header is moved outside and rendered separately with a search box.
- `grid-cols-1 md:grid-cols-2` — One column on mobile, two columns on medium+ screens.
- Note: NO `slice(0, 5)` here — this list shows ALL transactions.
- `onDelete={() => onDelete(expense._id)}` — Wraps the call in an arrow function so `expense._id` is captured correctly for each item in the loop.
- `onEdit={() => onEdit(expense)}` — Passes the entire expense object so the form can be pre-populated.

---

### 📄 `IncomeList.jsx`

Identical structure to `ExpenseList`, but for income (uses `income.source` as title).

---

### 📄 `ExpenseOverview.jsx`

**Purpose**: The header card on the Expense page showing a line chart of spending trends and an "Add Expense" button.

```javascript
const ExpenseOverview = ({ transactions, onExpenseIncome }) => {
  const [charData, setCharData] = useState([]);

  useEffect(() => {
    const result = prepareExpenseLineChartData(transactions);
    setCharData(result);
    return () => {};
  }, [transactions]);

  return (
    <div className="card">
      <div className="flex items-center justify-between">
        <div>
          <h5 ...>Expense Overview</h5>
          <p className="text-xs text-gray-600 ...">
            Track your spending trends over time ...
          </p>
        </div>
        <button className="add-btn" onClick={onExpenseIncome}>
          <LuPlus className="text-lg" /> Add Expense
        </button>
      </div>
      <div className="mt-10">
        <CustomLineChart data={charData} />
      </div>
    </div>
  );
};
```
- Uses `prepareExpenseLineChartData` (helper.js) to shape the data for the line chart.
- `onExpenseIncome` — Called when "Add Expense" is clicked; opens the modal in the parent page.

---

### 📄 `IncomeOverview.jsx`

Identical purpose to `ExpenseOverview` but for income, using a **bar chart** (`CustomBarChart`) instead of a line chart, and `prepareIncomeBarChartData`.

---

## 📄 9. Pages

Pages are the top-level screens that users navigate to. They orchestrate data fetching, state management, and render the appropriate layout + components.

---

### 📄 `Login.jsx`

**Purpose**: The login screen. Validates user input, calls the API, saves the token, and redirects.

```javascript
const Login = () => {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState(null);

  const { updateUser } = useContext(UserContext);
  const navigate = useNavigate();
```
- Three pieces of state: `email`, `password`, and `error` (for displaying validation messages).
- `updateUser` — From UserContext; saves the user info after successful login.
- `navigate` — For redirecting after login.

```javascript
  const handleLogin = async (e) => {
    e.preventDefault();
```
- `e.preventDefault()` — Prevents the default browser form submission (which would reload the page). We handle it with JavaScript instead.

```javascript
    if (!validateEmail(email)) {
      setError("Please enter a valid email address.");
      return;
    }
    if (!password) {
      setError("Please enter the password");
      return;
    }
    setError("");
```
- **Client-side validation** — Check inputs before wasting a server round-trip.
- `return` after `setError` — Stops the function from continuing to the API call.
- `setError("")` — Clears any previous error message if all validations pass.

```javascript
    try {
      const response = await axiosInstance.post(API_PATHS.AUTH.LOGIN, {
        email,
        password,
      });

      const { token, user } = response.data;

      if (token) {
        localStorage.setItem("token", token);
        updateUser(user);
        navigate("/dashboard");
      }
    } catch (error) {
      if (error.response && error.response.data.message) {
        setError(error.response.data.message);
      } else {
        setError("Something went wrong. Please try again");
      }
    }
  };
```
- `{ email, password }` — **Shorthand object property**: equivalent to `{ email: email, password: password }`.
- `const { token, user } = response.data` — **Destructuring**: pulls `token` and `user` out of the response.
- `localStorage.setItem("token", token)` — Saves the JWT token so it persists across page refreshes.
- `updateUser(user)` — Stores the user profile in global context.
- `navigate("/dashboard")` — Redirects to the dashboard.
- The `catch` block handles server-returned error messages (e.g., "Invalid credentials") or generic errors.

```javascript
  return (
    <AuthLayout bgImage="/bg.png">
      <div className="lg:w-[70%] h-3/4 md:h-full flex flex-col justify-center">
        <h3 ...>Welcome Back</h3>
        <p ...>Please enter your details to log in</p>

        <form onSubmit={handleLogin}>
          <Input value={email}    onChange={({ target }) => setEmail(target.value)}    label="Email Address" placeholder="..." type="text"     />
          <Input value={password} onChange={({ target }) => setPassword(target.value)} label="Password"      placeholder="..." type="password" />

          {error && <p className="text-red-500 text-xs pb-2.5">{error}</p>}

          <button type="submit" className="btn-primary">LOGIN</button>

          <p className="text-[13px] text-gray-300 mt-3">
            Don't have an account?{" "}
            <Link className="font-medium text-primary underline" to="/signUp">Sign Up</Link>
          </p>
        </form>
      </div>
    </AuthLayout>
  );
};
```
- `<form onSubmit={handleLogin}>` — The `onSubmit` event fires when the user presses Enter or clicks the submit button.
- `{error && <p>...</p>}` — Only shows the error paragraph if `error` is a non-empty string.
- `<Link to="/signUp">` — React Router's Link component for navigation without a full page reload.

---

### 📄 `SignUp.jsx`

**Purpose**: The registration screen. Similar to Login but also handles profile picture upload.

```javascript
const SignUp = () => {
  const [profilePic, setProfilePic] = useState(null);
  const [fullName, setFullname] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState(null);

  const { updateUser } = useContext(UserContext);
  const navigate = useNavigate();
```
- Four form fields. `profilePic` starts as `null` since the photo is optional.

```javascript
  const handleSignUp = async (e) => {
    e.preventDefault();

    let profileImageUrl = "";

    if (!fullName) { setError("Please enter your name"); return; }
    if (!validateEmail(email)) { setError("Please enter a valid email address"); return; }
    if (!password) { setError("Please enter the password"); return; }

    setError("");

    try {
      if (profilePic) {
        const imageUploadRes = await uploadImage(profilePic);
        profileImageUrl = imageUploadRes.imageUrl || "";
      }
```
- `let profileImageUrl = ""` — Starts empty. It will be populated only if the user selected a photo.
- `if (profilePic)` — Only uploads the image if one was selected. If not, `profileImageUrl` stays empty.
- `await uploadImage(profilePic)` — Calls our `uploadImage` utility, which returns `{ imageUrl: "..." }`.

```javascript
      const response = await axiosInstance.post(API_PATHS.AUTH.REGISTER, {
        fullName,
        email,
        password,
        profileImageUrl,
      });

      const { token, user } = response.data;

      if (token) {
        localStorage.setItem("token", token);
        updateUser(user);
        navigate("/dashboard");
      }
    } catch (error) {
      if (error.response && error.response.data.message) {
        setError(error.response.data.message);
      } else {
        setError("Something went wrong. Please try again");
      }
    }
  };
```
- Same pattern as Login: sends the registration data, saves the token, updates context, and navigates to dashboard.

```javascript
  return (
    <AuthLayout bgImage="/bg.png">
      <div className="lg:w-[100%] ...">
        <h3 ...>Create an Account</h3>
        <form onSubmit={handleSignUp}>
          <ProfilePhotoSelector image={profilePic} setImage={setProfilePic} />
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <Input value={fullName} onChange={({ target }) => setFullname(target.value)} label="Full Name" ... type="text" />
            <Input value={email}    onChange={({ target }) => setEmail(target.value)}    label="Email Address" ... type="text" />
            <div className="col-span-2">
              <Input value={password} onChange={({ target }) => setPassword(target.value)} label="Password" ... type="password" />
            </div>
          </div>
          {error && <p className="text-red-500 text-xs pb-2.5">{error}</p>}
          <button type="submit" className="btn-primary">SIGN UP</button>
          <p ...>Already have an account? <Link to="/login">Login</Link></p>
        </form>
      </div>
    </AuthLayout>
  );
};
```
- `col-span-2` — In the 2-column grid, the password field spans both columns (full width).

---

### 📄 `Home.jsx` (Dashboard)

**Purpose**: The central dashboard. Fetches all financial summary data from the backend and displays it across multiple cards and charts.

```javascript
const Home = () => {
  useUserAuth();

  const navigate = useNavigate();
  const [dashboardData, setDashboardData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [openBudgetModal, setOpenBudgetModal] = useState(false);
  const [budgetAmount, setBudgetAmount] = useState("");
```
- `useUserAuth()` — Calls our custom hook (the security guard) to verify the session on page load.
- `dashboardData` — Holds everything from the server: balance, income, expenses, transactions.
- `openBudgetModal` — Boolean controlling the "Set Monthly Budget" modal visibility.
- `budgetAmount` — The number the user types in the budget modal.

```javascript
  const fetchDashboardData = async () => {
    if (loading) return;
    setLoading(true);
    try {
      const response = await axiosInstance.get(`${API_PATHS.DASHBOARD.GET_DATA}`);
      if (response.data) {
        setDashboardData(response.data);
        setBudgetAmount(response.data.monthlyBudget || "");
      }
    } catch (error) {
      console.log("Something went wrong. Please try again. ", error);
    } finally {
      setLoading(false);
    }
  };
```
- `if (loading) return` — Prevents duplicate requests if this function is called while one is already in progress.
- `setBudgetAmount(response.data.monthlyBudget || "")` — Pre-fills the budget input with whatever the current budget is (so editing feels natural).

```javascript
  const handleUpdateBudget = async () => {
    try {
      const response = await axiosInstance.put(API_PATHS.AUTH.UPDATE_BUDGET, {
        monthlyBudget: Number(budgetAmount),
      });
      if (response.data) {
        toast.success("Budget updated successfully");
        setOpenBudgetModal(false);
        fetchDashboardData();
      }
    } catch (error) {
      console.error("Error updating budget:", error);
      toast.error(error.response?.data?.message || "Failed to update budget");
    }
  };
```
- `Number(budgetAmount)` — Converts the string from the input into a proper number for the API.
- `toast.success(...)` / `toast.error(...)` — Shows the notification popup via `react-hot-toast`.
- After saving, closes the modal and re-fetches dashboard data to reflect the new budget.

```javascript
  useEffect(() => {
    fetchDashboardData();
    return () => {};
  }, []);
```
- Fetches data once when the component mounts (the `[]` dependency array).

```javascript
  return (
    <DashboardLayout activeMenu="Dashboard">
      <div className="my-5 mx-auto">
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6">
          <InfoCard icon={<IoMdCard />}       label="Total balance" value={addThousandsSeparator(dashboardData?.totalBalance || 0)}  color="bg-primary"     />
          <InfoCard icon={<LuWalletMinimal />} label="Total Income"  value={addThousandsSeparator(dashboardData?.totalIncome || 0)}   color="bg-orange-500"  />
          <InfoCard icon={<LuHandCoins />}     label="Total Expense" value={addThousandsSeparator(dashboardData?.totalExpenses || 0)} color="bg-red-500"     />
        </div>
```
- `dashboardData?.totalBalance || 0` — Safe access with a fallback of 0 if data hasn't loaded yet.

```javascript
        <div className="grid grid-cols-1 md:grid-cols-2 gap-6 mt-6">
          <RecentTransactions transactions={dashboardData?.recentTransactions} onSeeMore={() => navigate("/expense")} />
          <AIInsights />
          <BudgetProgressCard totalExpenses={dashboardData?.totalExpenses || 0} monthlyBudget={dashboardData?.monthlyBudget || 0} onSetBudget={() => setOpenBudgetModal(true)} />
          <FinanceOverview totalBalance={dashboardData?.totalBalance || 0} totalIncome={dashboardData?.totalIncome || 0} totalExpense={dashboardData?.totalExpenses || 0} />
          <Last30DaysExpenses data={dashboardData?.last30DaysExpenses?.transactions || []} />
          <ExpenseTransactions transactions={dashboardData?.last30DaysExpenses?.transactions || []} onSeeMore={() => navigate("/expense")} />
          <RecentIncomeWithChart data={dashboardData?.last60DaysIncome?.transactions?.slice(0, 4) || []} totalIncome={dashboardData?.totalIncome || 0} />
          <ExpenseCategoryChart data={dashboardData?.last30DaysExpenses?.transactions || []} />
          <RecentIncome transactions={dashboardData?.last60DaysIncome?.transactions || []} onSeeMore={() => navigate("/income")} />
        </div>
```
- A 2-column grid of dashboard cards. Each card receives slices of the `dashboardData` object as props.
- `?.slice(0, 4)` — Takes only the first 4 income transactions for the donut chart (to keep it readable).

```javascript
        <Modal isOpen={openBudgetModal} onClose={() => setOpenBudgetModal(false)} title="Set Monthly Budget">
          <div className="space-y-4">
            <Input label="Monthly Budget Amount" placeholder="e.g. 5000" type="number" value={budgetAmount} onChange={(e) => setBudgetAmount(e.target.value)} />
            <div className="flex justify-end mt-6">
              <button className="add-btn add-btn-fill" onClick={handleUpdateBudget}>Save Budget</button>
            </div>
          </div>
        </Modal>
```
- Opens when the user clicks "Set Budget" / "Edit Budget" on `BudgetProgressCard`.

---

### 📄 `Expense.jsx`

**Purpose**: The full expense management page with search, add, edit, delete, and download functionality.

```javascript
const Expense = () => {
  useUserAuth();

  const [expenseData, setExpenseData] = useState([]);
  const [loading, setLoading] = useState(false);
  const [openDeleteAlert, setOpenDeleteAlert] = useState({ show: false, data: null });
  const [openAddExpenseModal, setOpenAddExpenseModal] = useState({ show: false, data: null });
  const [searchQuery, setSearchQuery] = useState("");
```
- `openDeleteAlert` — An object with `show` (boolean) and `data` (the ID to delete). Using an object allows us to store both the visibility flag and associated data together.
- `openAddExpenseModal` — Same pattern: `show` controls modal visibility, `data` holds the expense being edited (or `null` for "add" mode).
- `searchQuery` — The text the user types in the search box.

```javascript
  const filteredExpenseData = expenseData.filter((expense) => {
    return expense.category.toLowerCase().includes(searchQuery.toLowerCase());
  });
```
- **Client-side search**: filters the already-loaded data without calling the server.
- `.toLowerCase()` on both sides makes the search case-insensitive.

```javascript
  const handleAddOrUpdateExpense = async (expense) => {
    const { category, amount, date, icon } = expense;

    if (!category.trim()) { toast.error("Category is required"); return; }
    if (!amount || isNaN(amount) || Number(amount) <= 0) { toast.error("Amount should be a valid number greater than 0."); return; }
    if (!date) { toast.error("Date is required."); return; }

    try {
      if (openAddExpenseModal.data) {
        // Edit mode
        await axiosInstance.put(
          API_PATHS.EXPENSE.UPDATE_EXPENSE(openAddExpenseModal.data._id),
          { category, amount, date, icon }
        );
        toast.success("Expense updated successfully");
      } else {
        // Add mode
        await axiosInstance.post(API_PATHS.EXPENSE.ADD_EXPENSE, { category, amount, date, icon });
        toast.success("Expense added successfully");
      }
      setOpenAddExpenseModal({ show: false, data: null });
      fetchExpenseDetails();
    } catch (error) {
      console.error("Error adding/updating the Expense:", error.response?.data?.message || error.message);
    }
  };
```
- `category.trim()` — `.trim()` removes leading/trailing spaces. Prevents " " (a space) from passing validation.
- `isNaN(amount)` — Checks if the value is not a valid number.
- `if (openAddExpenseModal.data)` — If `data` is not null, we're in edit mode (use PUT). If null, we're adding (use POST).
- After a successful operation, closes the modal and re-fetches the list.

```javascript
  const deleteExpense = async (id) => {
    try {
      await axiosInstance.delete(API_PATHS.EXPENSE.DELETE_EXPENSE(id));
      setOpenDeleteAlert({ show: false, data: null });
      toast.success("Expense details deleted successfully");
      fetchExpenseDetails();
    } catch (error) {
      console.error("Error deleting expense", error?.response?.data?.message || error.message);
    }
  };
```
- Calls the DELETE endpoint with the expense ID. After success, closes the confirm dialog and refreshes the list.

```javascript
  const handleDownloadExpenseDetails = async () => {
    try {
      const response = await axiosInstance.get(API_PATHS.EXPENSE.DOWNLOAD_EXPENSE, {
        responseType: "blob",
      });

      const url = window.URL.createObjectURL(new Blob([response.data]));
      const link = document.createElement("a");
      link.href = url;
      link.setAttribute("download", "expense_details.xlsx");
      document.body.appendChild(link);
      link.click();
      link.parentNode.removeChild(link);
      window.URL.revokeObjectURL(url);
    } catch (error) {
      console.error("Error Downloading expense details:", error);
      toast.error("Failed to download expense details, Please try again");
    }
  };
```
- `responseType: "blob"` — Tells Axios to treat the response as raw binary data (a file), not JSON.
- `new Blob([response.data])` — Wraps the binary data into a Blob object.
- `window.URL.createObjectURL(...)` — Creates a temporary URL pointing to the Blob in memory.
- `document.createElement("a")` — Creates an invisible `<a>` (link) element.
- `link.setAttribute("download", "expense_details.xlsx")` — Tells the browser to download it with this filename instead of navigating.
- `document.body.appendChild(link); link.click()` — Appends the link to the page and simulates a click to trigger the download.
- `link.parentNode.removeChild(link)` — Cleans up the invisible link.
- `window.URL.revokeObjectURL(url)` — Releases the temporary URL from memory.

```javascript
  return (
    <DashboardLayout activeMenu="Expense">
      <div className="my-5 mx-auto">
        <div className="grid grid-cols-1 gap-6">
          <ExpenseOverview transactions={expenseData} onExpenseIncome={() => setOpenAddExpenseModal({ show: true, data: null })} />

          <div className="flex items-center justify-between gap-4 -mb-4">
            <h5 ...>Expense Log</h5>
            <div className="flex items-center gap-3">
              <input type="text" placeholder="Search category..." ... value={searchQuery} onChange={(e) => setSearchQuery(e.target.value)} />
              <button className="card-btn" onClick={handleDownloadExpenseDetails}>
                <LuDownload className="text-base" /> Download
              </button>
            </div>
          </div>

          <ExpenseList
            transactions={filteredExpenseData}
            onDelete={(id) => { setOpenDeleteAlert({ show: true, data: id }); }}
            onDownload={handleDownloadExpenseDetails}
            onEdit={(data) => setOpenAddExpenseModal({ show: true, data })}
            hideHeader
          />
        </div>

        <Modal isOpen={openAddExpenseModal.show} onClose={() => setOpenAddExpenseModal({ show: false, data: null })}
          title={openAddExpenseModal.data ? "Edit Expense" : "Add Expense"}>
          <AddExpenseForm onAddExpense={handleAddOrUpdateExpense} data={openAddExpenseModal.data} />
        </Modal>

        <Modal isOpen={openDeleteAlert.show} onClose={() => setOpenDeleteAlert({ show: false, data: null })} title="Delete Expense">
          <DeleteAlert content="Are you sure you want to delete this expense detail?" onDelete={() => deleteExpense(openDeleteAlert.data)} />
        </Modal>
      </div>
    </DashboardLayout>
  );
};
```
- `filteredExpenseData` — Passes the **filtered** (searched) list, not the raw one.
- `onDelete={(id) => { setOpenDeleteAlert({ show: true, data: id }) }}` — When delete is clicked on a card, opens the confirmation modal and stores the ID of the item to delete.
- `onEdit={(data) => setOpenAddExpenseModal({ show: true, data })}` — Opens the Add/Edit modal in edit mode, with the existing expense pre-loaded.
- `hideHeader` — The list header (title + download button) is hidden because we moved those controls outside the list component.
- `title={openAddExpenseModal.data ? "Edit Expense" : "Add Expense"}` — The modal title dynamically reflects the mode.

---

### 📄 `Income.jsx`

Identical in structure to `Expense.jsx`, but for income. The differences are:
- Uses `incomeData`, `openAddIncomeModal`, etc.
- Calls `API_PATHS.INCOME.*` endpoints.
- The search filters by `income.source` instead of `expense.category`.
- Uses `IncomeOverview`, `IncomeList`, `AddIncomeForm` components.
- Downloaded file is named `income_details.xlsx`.

---

*End of Documentation. Every file, every function, every line has been covered.*
