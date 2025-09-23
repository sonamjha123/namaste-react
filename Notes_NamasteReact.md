# NAMASTE REACT _ NOTES
-------------------------
## 1. Episode01: Inception

### Topics covered:
- cdn
- React-Hello world using createElement, createRoot and render


## CDN : Detailed explanation
### 🔹 1. What a CDN actually does

A **CDN is a network of servers** distributed around the world. Its main job is to **store cached copies** of static assets (HTML, CSS, JS, images, videos, etc.) from your website and deliver them to users from the **nearest server** to reduce latency.

### 🔹 2. When you type a website for the 1st time

Let’s say you type **example.com** into your browser for the very first time:

1. **DNS Lookup**
   Your browser asks: *“Where is example.com?”*
   The DNS is configured to point not to the origin server directly, but to the **CDN’s servers** (like Cloudflare, Akamai, AWS CloudFront).

2. **CDN Server Chosen (Nearest POP)**
   The CDN has many **Points of Presence (POP)** across the globe.
   Based on your location, the DNS routes your request to the **nearest POP** (for example, if you’re in Paris, it will connect to a CDN node in Paris, not in the US).

3. **Cache Check (Does the POP already have the copy?)**

   * If **yes** (someone nearby already visited this site recently):
     The CDN **serves the cached content immediately**.
   * If **no** (you are the first user in that region, or the cache expired):
     The CDN server will **fetch the content from the origin server** (the real website’s hosting server), store it (cache it), and then deliver it to you.

---

### 🔹 3. What happens next time?

* The next user in the same region asking for the same site will **get the cached copy directly from the CDN POP**.
* This means faster load times and less load on the original server.

---

### 🔹 4. Key Point — “Copy Creation”

👉 The CDN **doesn’t copy the whole site in advance everywhere**.

* It **creates local copies on demand** (when a user in a region requests content).
* Over time, as more people from different regions request the site, multiple CDN POPs build their own cached versions.

---

✅ **Example Flow (Your 1st visit to example.com in Paris):**

1. You type example.com → DNS → Paris CDN POP.
2. CDN POP in Paris checks cache → empty.
3. CDN fetches from origin server (say in US).
4. CDN stores a copy in Paris.
5. Serves you the page.
6. Next user in Paris gets it directly from Paris POP without going to US.

Perfect 👌 let’s dive deeper. I’ll split this into two parts:

1. **How CDNs handle dynamic content (APIs, user dashboards, etc.)**
2. **How a CDN knows the “original server” and how many servers are involved.**

---

## 🔹 1. CDN and Dynamic Content

Not everything on a website can be cached, e.g.:

* User-specific dashboards (e.g. your Gmail inbox)
* APIs that change frequently (stock prices, live scores)
* Payment transactions

So, how does a CDN handle these?

### ✅ Approaches:

* **Static caching**: Things like images, CSS, JS files, videos → cached for long durations.
* **Dynamic content pass-through**: For things that can’t be cached, CDN simply acts as a **smart proxy**.

  * Example: You log into Facebook. The CDN forwards your login request to the origin (real server), because this response is unique to you.
* **Edge computing / Edge caching**: Many CDNs (like Cloudflare Workers, AWS CloudFront Lambda\@Edge) now allow running logic at the **edge (nearest POP)**.

  * Example: For an API returning stock prices, instead of always hitting the origin, the CDN can cache data for 2–5 seconds at edge servers and refresh automatically.

👉 So, CDN **caches what’s cacheable** and **forwards what’s not**.

---

## 🔹 2. How does a CDN know the original server?

Every website using a CDN configures:

* The **origin server address** → usually an IP or domain name (e.g. `origin.example.com`).
* When you set up a CDN (like Cloudflare, Akamai, AWS CloudFront), you tell it:

  > "Hey CDN, my actual server lives at `123.45.67.89` (or AWS S3, or some hosting). Fetch from there if you don’t have cached content."

So:

* If cache is **hit** → serve from CDN POP.
* If cache is **miss** → go to **origin server**, fetch fresh data, store locally, then serve.

---

## 🔹 3. Is there only 1 origin server per country?

Not necessarily. There are a few deployment models:

### 🔸 (a) **Single Origin Server (basic setup)**

* Small websites often have just 1 main server (say in the US).
* The CDN fetches from that server when needed.
* Example: Your Paris CDN POP might fetch content directly from the US origin.

### 🔸 (b) **Multiple Origin Servers / Regional Servers**

* Big companies (Netflix, YouTube, Amazon) have **multiple data centers/origin servers** across regions.
* CDN knows which origin to fetch from based on **routing rules**.
* Example: Netflix has servers in almost every country; CDNs just direct you to the closest one.

### 🔸 (c) **Hybrid with Cloud Storage**

* Some sites don’t even have their own servers → they use cloud buckets (like AWS S3, GCP Storage).
* The CDN origin is just a bucket endpoint.

---

## 🔹 Putting it all together (Example: You visit `myshop.com`)

1. You type `myshop.com`.
2. DNS sends you to **nearest CDN POP**.
3. POP checks its cache.

   * If **static CSS/JS/image** → served from cache.
   * If **API call / logged-in page** → forwarded to origin (say AWS server in US).
4. CDN **knows origin** because the website owner configured it (e.g., `origin.myshop.com`).
5. If many users worldwide are requesting → CDN creates cached copies across multiple POPs.

---

⚡ **So the CDN is like a middleman with warehouses around the world.**

* It knows where the factory (origin server) is.
* It stocks frequently used products (cached files).
* If something is new/custom (dynamic request), it calls the factory directly.
Nice 👍 this is an important scenario. Let’s break down **what happens if the origin server goes down while using a CDN**.

---

## 🔹 1. Origin Server Goes Down — Two Cases

### ✅ Case A: Cached Content Exists

* If your CDN already has cached copies of your site’s files (HTML, CSS, JS, images, videos), it can still **serve them to users**.
* Example:

  * You go to a blog that uses Cloudflare.
  * The origin is down.
  * But Cloudflare still has your last articles cached, so it shows them.

This is called **“stale content serving”** or **“Always Online” mode**.

👉 Some CDNs (like Cloudflare, Akamai, Fastly) have a feature where:

* If the origin is unavailable, CDN keeps serving the cached version until the origin comes back.

---

### ❌ Case B: No Cached Content

* If the CDN **doesn’t have the file in cache** (maybe it’s new, or cache expired), then:

  * It will try to fetch from origin.
  * Since the origin is down, the request fails.
  * User sees an **error page (502/503/504 Bad Gateway or Origin Unreachable)**.

---

## 🔹 2. Dynamic vs Static in Downtime

* **Static parts** (images, JS, CSS, product pages) → can still work if cached.
* **Dynamic parts** (APIs, checkout page, login dashboard) → will break, since they depend on live origin.

---

## 🔹 3. CDN Features for Handling Downtime

1. **Stale While Revalidate**: Serve old content to users while retrying in background.
2. **Always Online Mode**: Show last cached version (Cloudflare does this).
3. **Origin Shielding / Failover Origins**:

   * You can configure multiple origin servers (backup origin).
   * If one is down, CDN routes traffic to another.
4. **Edge Workers (advanced)**: Some CDNs can even generate a fallback page at the edge if origin is dead.

---

## 🔹 4. Real Example

Let’s say your **e-commerce site** runs on a single server in the US, but you use Cloudflare CDN.

* If your server goes down, and a user in Paris visits a product page:

  * If the page was cached → Paris POP still shows it.
  * If user tries to **checkout (dynamic)** → fails, since it needs origin server.

---

✅ So, **a CDN is not a full backup of your website**, but it can keep static/deliverable parts alive until the origin recovers.
That’s why large companies often combine CDNs + **multi-region servers** + **failover strategies** for true high availability.

---
## CORS:

**CORS** is a **security feature** implemented by **browsers** that **controls how web pages can request resources from a different domain** than the one that served the web page.
Ah! You’re asking about the `crossorigin` attribute in your `<script>` tags. Let’s break it down carefully.

---

## 🔹 1. What **Cross-Origin** Means

“Cross-origin” refers to **resources loaded from a different domain (origin) than your webpage**.

* An **origin** is defined by **protocol + domain + port**.

  * Example:

    * `https://yourwebsite.com` → origin A
    * `https://unpkg.com` → origin B

If your page is on `https://yourwebsite.com` and it loads a script from `https://unpkg.com`, that’s a **cross-origin request**.

---

## 🔹 2. Why `crossorigin` is Needed

Browsers enforce **CORS (Cross-Origin Resource Sharing)** rules for security.

The `crossorigin` attribute on `<script>` tells the browser:

1. **It’s okay to request this script from a different origin.**
2. **How to handle credentials (cookies, HTTP auth) when fetching it.**

It can have **3 modes**:

| Value             | Meaning                                                                                         |
| ----------------- | ----------------------------------------------------------------------------------------------- |
| `anonymous`       | Send request **without credentials** (default if you just write `crossorigin`).                 |
| `use-credentials` | Send request **with credentials** (cookies, HTTP auth, client certs).                           |
| omitted           | Browser may block certain operations (like reading script for integrity check) if cross-origin. |

---

### 🔹 3. Example in Your Case

```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
```

* Your page is on `https://mywebsite.com`.
* Script is on `https://unpkg.com`.
* `crossorigin` allows the browser to **fetch React safely** from unpkg.
* This is especially needed if you are also using **Subresource Integrity (SRI)**:

```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.development.js" integrity="sha384-XXXX"></script>
```

Without `crossorigin`, SRI might fail because the browser cannot verify the hash for cross-origin scripts.

---

## 🔹 4. Key Points

* **Cross-origin scripts** are scripts loaded from another domain.
* `crossorigin` ensures the browser fetches them safely and can apply security checks.
* Usually, for CDN scripts like React/ReactDOM, you use `crossorigin="anonymous"`.
* If the script requires cookies for access, use `crossorigin="use-credentials"` (rare for public CDNs).
---

### Real-World Analogy

Imagine you’re on website `websiteA.com`, and it tries to fetch data from `websiteB.com`.  
CORS is like a **security guard** who checks if `websiteB.com` allows `websiteA.com` to access its data.

---

### 💡 Why Do We Need CORS?

To **prevent malicious websites** from accessing sensitive data on another site where you're logged in (like your bank).

---

### 🔄 Same-Origin vs Cross-Origin

- ✅ **Same-origin** request:  
  `websiteA.com/page` → `websiteA.com/api/data` → No problem.

- ❌ **Cross-origin** request:  
  `websiteA.com` → `api.websiteB.com` → Needs permission via CORS headers.

---

### 🛡️ How Does CORS Work?

When your browser makes a **cross-origin request**, it:

1. Sends a **preflight request** (if needed) using the HTTP `OPTIONS` method.
2. The server (like `websiteB.com`) responds with headers like:

```http
Access-Control-Allow-Origin: https://websiteA.com
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type
```

3. If allowed, the browser proceeds with the actual request.

---

### ⚙️ Common CORS Headers

| Header | Purpose |
|--------|---------|
| `Access-Control-Allow-Origin` | Specifies which origin is allowed to access. (`*` allows all) |
| `Access-Control-Allow-Methods` | Allowed HTTP methods (GET, POST, etc.) |
| `Access-Control-Allow-Headers` | Headers allowed in request |
| `Access-Control-Allow-Credentials` | Allow cookies/auth to be sent |

---

### ⚠️ Common Issues Developers Face

- Getting a **CORS error** in the browser console (e.g., `No 'Access-Control-Allow-Origin' header`).
- Not handling **preflight requests** on the server.
- Forgetting to allow **credentials** (cookies, tokens) if needed.

---

### 🛠️ How to Fix CORS Issues

- **Client (Frontend)**: You *can't* fix CORS on the frontend; the **server must set the right headers**.
- **Server (Backend)**: Use middleware or server config to allow certain origins.

**Example in Express.js:**
```js
const cors = require('cors');
app.use(cors({ origin: 'https://websiteA.com', credentials: true }));
```
------------------------------------------------------------------------------------------------------

## What exactly is React : createElement, createRoot, render 

Here’s an **optimized version** of your notes for GitHub — clean, well-structured, and ready for a repository. I preserved all your content but organized it neatly with comments and formatting for clarity.
---
# React Basic Example Notes

## index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <title>This is the title of the webpage!</title>
    <link rel="stylesheet" href="index.css">
  </head>
  <body>
    <!-- Root div where React will render -->
    <div id="root"></div>

    <!-- React & ReactDOM CDN -->
    <script
      crossorigin
      src="https://unpkg.com/react@18/umd/react.development.js"
    ></script>
    <script
      crossorigin
      src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"
    ></script>

    <!-- Main React script -->
    <script src="index.js"></script>
  </body>
</html>
```

---

## index.js

```javascript
/*
Structure to be rendered:

<div id="parent">
    <div id="child">
        <h1>This is h1 tag</h1>
        <h2>This is h2 tag</h2>
    </div>
</div>

ReactElement => Object => HTML element (browser understands this)
*/

// Creating parent div with child and headings
const parent = React.createElement(
  "div",
  { id: "parent" },
  React.createElement(
    "div",
    { id: "child" },
    [
      React.createElement("h1", {}, "This is h1 tag"),
      React.createElement("h2", {}, "This is h2 tag")
    ]
  )
);

// Rendering the parent element
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(parent);

// Create a separate heading element and render (optional)
/*
const heading = React.createElement("h1", { id: "heading" }, "Namaste React");
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(heading);
*/
```

---

### ✅ Notes / Key Points

1. **React.createElement** creates a React element (JavaScript object) that React converts to actual DOM elements in the browser.
2. **Rendering**: `ReactDOM.createRoot(...).render(element)` is used to render React elements.
3. **Cross-origin CDN scripts**: Required for loading React from another domain safely.
4. You can create multiple elements and render them as children using **arrays**.
5. Optional headings can be rendered separately by uncommenting the last block.

#### Note: if you have h1 tags above or below div id = root then u can use react only in that div and rest h1 tags will not be replaced
Exactly ✅ — that’s how **React works with the DOM**. Let me explain clearly.

---

### 🔹 Key Idea

React **only controls the DOM elements inside the root container** you specify.

Example:

```html
<body>
  <h1>This is outside React</h1>
  <div id="root"></div>
  <h1>This is also outside React</h1>
</body>
```

```javascript
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(React.createElement("h1", {}, "This is rendered by React"));
```

* React will **only render inside `<div id="root">`**.
* The two `<h1>` tags outside of `root` will **not be replaced or affected**.

---

### 🔹 Why this matters

1. **Scoped rendering**: React manages **only the elements inside its root container**.
2. **Non-intrusive**: You can mix React with normal HTML without overwriting other content.
3. **Multiple roots**: You can have **multiple React roots** in a single page:

```html
<div id="root1"></div>
<div id="root2"></div>
```

```javascript
ReactDOM.createRoot(document.getElementById("root1"))
  .render(React.createElement("h1", {}, "Hello from root1"));

ReactDOM.createRoot(document.getElementById("root2"))
  .render(React.createElement("h1", {}, "Hello from root2"));
```

* Each root is **independent** and React doesn’t touch elements outside its container.

---------------------------------------------------------------------------------

# 2. Episode 02: Igniting our App
### Topics covered:
- npm, parcel
- nodemodules
- Bundlers
- React app file structure
---
###  📦 NPM & Bundlers (Quick Revision Notes)
#### 🔹 NPM Basics
- [x] **NPM** = Node Package Manager (manages dependencies & packages).  
- [x] Initialize project → `npm init`.  
- [x] Dependencies:
  - Normal dependencies → required in **production**.  
  - Dev dependencies → required in **development only** (`-D` flag).  
- [x] `package-lock.json` → locks exact package versions.  
- [x] Versioning:
  - `^` → minor + patch updates allowed.  
  - `~` → only patch updates allowed.  

---

#### 🔹 Bundlers
- [x] **Bundler** = tool that prepares code for production.  
- [x] Examples: **Webpack, Parcel, Vite**.  
- [x] **Job of bundlers**:
  - Bundle multiple files.  
  - Minify & optimize code.  
  - Code splitting.  
  - Differential bundling (support older browsers).  
  - Tree shaking (remove unused code).  
  - Image optimization.  

---

#### 🔹 Create React App (CRA)
- CRA internally uses **Webpack**.  
- We use **Parcel** → simpler & zero-config.  

---

#### 🔹 Using Parcel
- Install:  
  ```bash
  npm install -D parcel

	•	Run:

npx parcel index.html


	•	Build for production:

npx parcel build index.html


	•	Features:
	•	✅ Dev server with HMR (Hot Module Replacement).
	•	✅ File watching (C++ algorithm).
	•	✅ Caching (.parcel-cache) → faster rebuilds.
	•	✅ Consistent hashing for caching.
	•	✅ Better error suggestions.
	•	✅ HTTPS hosting support.
	•	⚠️ Cache & dist folders → auto-regenerated → add to .gitignore.
	•	Configure Browserlist in package.json to specify supported browsers.

⸻

🔹 React Setup with Parcel
	•	Install React:

npm install react react-dom


	•	Import in index.js:

import React from "react";
import ReactDOM from "react-dom/client";


	•	Common error:

[parcel]: Browser scripts cannot have imports/exports

✅ Fix → Add type="module" in <script> inside index.html.

⸻

🔹 Why Not CDN for React?
	•	Managing React via npm (node_modules) is easier, version-controlled, and consistent.

⸻

🔹 References
	•	Creating your own Create-React-App.
	•	Parcel Docs
	•	BrowsersList

⸻

👉 Quick Mental Model:
NPM manages dependencies → Bundler (Parcel) optimizes & ships code → React runs on top with proper imports & setup.

---

Do you also want me to add a **section with command cheat sheet** (all npm/parcel commands in one place for fast recall)?
⸻-------------------------

#### Refer Chapter 02 - Assignment - Igniting our App - for more questions
## 1.🔹 What are Parcel and Webpack?

Both **Parcel** and **Webpack** are **JavaScript bundlers** (also called module bundlers).
They take your application’s source code (JS, CSS, HTML, images, etc.), process it, and output optimized files that can run efficiently in the browser.

* **Why we need bundlers**

  * Modern JS apps use **modules** (`import/export`), which browsers don’t handle natively for all cases (especially older browsers).
  * They **combine many small files** into fewer/larger bundles to reduce network requests.
  * Handle **transpilation** (e.g., Babel, TypeScript → JS).
  * Handle **asset optimization** (minification, tree shaking, code splitting).
  * Provide **development features** like Hot Module Replacement (HMR), live reloading, and dev servers.

---

### 🔹 Webpack

* One of the most popular & widely adopted bundlers.
* Highly **configurable** but has a **steep learning curve**.
* Uses a config file (`webpack.config.js`) to define entry, loaders, plugins, and output.
* Strong ecosystem with many **plugins and loaders**.
* Great for **large-scale, complex applications** where fine-grained control is needed.

✅ **Pros**

* Huge community and ecosystem.
* Extremely flexible & customizable.
* Works well with advanced optimizations (tree shaking, code splitting).
* Mature and proven in production.

❌ **Cons**

* Complex configuration.
* Slower build times compared to newer tools.

---

### 🔹 Parcel

* A newer bundler known for **zero-config setup**.
* Automatically detects project setup (no `webpack.config.js` required for basics).
* Built-in support for **JS, TypeScript, CSS, HTML, images, etc.**
* Has **fast build times** with caching and parallel processing.
* Great choice for **small-to-medium projects** or when you want quick setup.

✅ **Pros**

* Zero-config (works out-of-the-box).
* Faster builds (thanks to multi-core + caching).
* Simpler learning curve.
* Built-in features (HMR, Babel, TypeScript, etc.).

❌ **Cons**

* Smaller ecosystem compared to Webpack.
* Less flexible for very advanced/custom setups.
* Not as widely adopted in enterprise.

---

### 🔹 Parcel vs Webpack – Quick Comparison

| Feature           | Webpack ⚡                             | Parcel 🚀                            |
| ----------------- | ------------------------------------- | ------------------------------------ |
| Config required?  | Yes (detailed)                        | No (zero-config by default)          |
| Learning curve    | High                                  | Low                                  |
| Build speed       | Slower (needs optimization)           | Faster (multi-thread + caching)      |
| Ecosystem/plugins | Huge                                  | Smaller                              |
| Best for          | Large-scale apps needing custom setup | Small/medium apps, quick prototypes  |
| Popularity        | Very high                             | Growing, but less used in enterprise |

---

### 🔹 Which is Better?

* **Use Webpack** if:

  * You need **full control** over the build process.
  * You’re working on a **large enterprise application**.
  * You need access to a **mature plugin ecosystem**.

* **Use Parcel** if:

  * You want a **fast setup** with minimal config.
  * You’re building a **small-to-medium project** or **prototype**.
  * You prefer simplicity and faster build times.

---

#### 🔹 Interview Tips

1. **Explain why bundlers exist** (optimization, compatibility, developer experience).
2. **Highlight trade-offs** → Webpack = flexibility + ecosystem, Parcel = speed + simplicity.
3. **Mention real-world use cases**:

   * Webpack → React, Angular, Vue enterprise apps.
   * Parcel → quick prototypes, personal projects, small teams.
4. Stay updated: alternatives like **Vite, esbuild, and Rollup** are also common in interviews.

---
### Explain Bundler Key Concepts: Asset Optimization, Development Features, 
##### 🔹 Asset Optimization

**Minification** → Removes unnecessary characters (spaces, comments, long variable names) to make files smaller and load faster.

**Tree Shaking** → Removes unused/dead code from the final bundle (e.g., unused functions).

**Code Splitting** → Splits code into smaller chunks that are loaded on demand (improves initial load time).

##### 🔹 Development Features

**Hot Module Replacement (HMR)** → Updates only the changed module in the browser without refreshing the entire page → faster dev feedback.

**Live Reloading** → Refreshes the whole page automatically when you change code.

**Dev Server** → A local server that serves your app in development, handles fast rebuilds, live reload/HMR, and often supports proxying APIs.
In React, `componentDidMount` is a lifecycle method that is called after a component has been rendered to the DOM. Making API calls in `componentDidMount` is a common pattern for several reasons:

------
### 📦 Difference: `package.json` vs `package-lock.json`
#### 🔹 `package.json`

* Describes the **project metadata** and its **dependencies**.
* Lists dependencies with **version ranges** (e.g., `"^1.2.3"`, `"~4.5.6"`).
* Used by developers to know which packages the project needs.
* Example:

  ```json
  {
    "name": "my-app",
    "version": "1.0.0",
    "dependencies": {
      "react": "^18.2.0"
    }
  }
  ```

---

#### 🔹 `package-lock.json`

* Auto-generated file when installing dependencies.
* Locks down the **exact versions** of every installed package (including nested dependencies).
* Ensures **consistent installs across different machines**.
* Should always be committed to version control.
* Example snippet:

  ```json
  {
    "name": "my-app",
    "lockfileVersion": 2,
    "dependencies": {
      "react": {
        "version": "18.2.0",
        "resolved": "https://registry.npmjs.org/react/-/react-18.2.0.tgz",
        "integrity": "sha512-..."
      }
    }
  }
  ```

---

### 🔹 Quick Comparison

| Feature                    | `package.json`          | `package-lock.json`        |
| -------------------------- | ----------------------- | -------------------------- |
| Purpose                    | Metadata + dependencies | Lock exact dependency tree |
| Maintained by              | Developer               | npm (auto-generated)       |
| Dependency versions        | Ranges allowed          | Exact versions             |
| Human readability          | Yes                     | No (machine-focused)       |
| Commit to version control? | ✅ Yes                   | ✅ Yes                      |

---

# 🎯 Interview-Style Questions

1. **Q:** What is the difference between `package.json` and `package-lock.json`?
   **A:** `package.json` defines project metadata and dependency ranges, while `package-lock.json` locks exact versions for consistent installs across machines.

2. **Q:** If you delete `package-lock.json` and reinstall, what happens?
   **A:** npm will generate a new lock file, possibly with different dependency versions (within allowed ranges), which can cause subtle bugs.

3. **Q:** Should you commit `package-lock.json` to Git? Why?
   **A:** Yes, because it ensures all team members and CI/CD pipelines install identical dependency versions.

4. **Q:** Which file do you manually edit, and which one is auto-generated?
   **A:** Developers edit `package.json`; `package-lock.json` is auto-generated by npm.
---
### What is node modules
**Definition**

- **node_modules/ is a directory created by npm or yarn when you install dependencies for a Node.js project.
It stores all installed packages (and their sub-dependencies) listed in package.json.**

**🎯 Interview-Style Questions**

- **Q: What is the purpose of the node_modules folder**?
A: It stores all installed dependencies and their nested dependencies needed for the project to run.

- **Q: Why is node_modules not committed to Git?**
A: Because it can be regenerated from package.json + package-lock.json, and committing it would bloat the repo.

- **Q: Can you delete node_modules? What happens?**
A: Yes. It can be deleted anytime and restored by running npm install or yarn install.

- **Q: Why does node_modules sometimes get very large?**
A: Due to nested dependencies and multiple versions of the same package required by different libraries.

----------------
# Episode 03: Laying the foundation
## Topics covered 
- JSX, React Components
- class-based vs functional components


Here’s your optimized .md version for GitHub notes. I’ve kept everything you provided, added a few missing points, and structured it cleanly for readability.

⸻

React Basics Notes

📦 Package.json Scripts

"scripts": {
  "start": "parcel index.html",
  "build": "parcel build index.html",
  "test": "echo \"Error: no test specified\" && exit 1"
}

	•	To start the project:

npm run start   # or simply npm start


	•	To build the project:

npm run build



⸻

⚛️ React Elements
	•	React elements are equivalent to DOM elements.
	•	React.createElement → returns an object, when rendered to the DOM, it becomes an HTML element.
	•	Example:

const heading = React.createElement("h1", { id: "heading" }, "Namaste React");
console.log(heading); // Object



⸻

🖼️ ReactDOM
	•	Since we deal with browsers, we use:

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(heading);



⸻

💡 JSX (JavaScript XML)
	•	JSX is created by Facebook developers.
	•	JSX is not part of React. It’s just syntactic sugar that makes development easier.
	•	JSX is not HTML inside JS, but an HTML-like syntax.
	•	Example:

const jsxHeading = <h1 id="jsx-heading">This is JSX heading</h1>;
console.log(jsxHeading); // Object



⸻

🔑 Difference: HTML vs JSX
	•	HTML:

const heading = React.createElement("h1", { id: "heading" }, "Namaste React");


	•	JSX:

const jsxHeading = <h1 id="jsx-heading">This is JSX heading</h1>;



⸻

🚀 How JSX Works
	•	JavaScript Engine understands ECMAScript (ES6), but browsers do not understand JSX.
	•	Parcel → bundles code.
	•	Babel → transpiles JSX → React.createElement.
	•	Flow:

JSX → Babel → React.createElement → React Element (Object) → HTML Element


	•	Babel is an open-source JavaScript compiler.
	•	It doesn’t just convert but also transpiles (down-levels code).

⸻

✅ JSX Rules
	•	JSX attributes use camelCase.
	•	Single-line JSX is valid without wrapping.
	•	Multi-line JSX must be wrapped inside parentheses ():

const Title = () => (
  <h1 className="heading">Namaste React</h1>
);



⸻

🛠️ Recommended VSCode Extensions
	•	Prettier - Code Formatter
	•	Bracket Pair Colorizer
	•	ESLint
	•	Better Comments

⸻

🌟 Advantages of JSX
	•	Makes code more readable and declarative.
	•	Prevents injection attacks by sanitizing expressions.
	•	Allows embedding JavaScript inside {}.

⸻

🧩 React Components
	•	Class-based Components
	•	Functional Components → functions that return JSX or React Elements.
```
Example: Component Composition

import React from "react";
import ReactDOM from "react-dom/client";

const Title = () => (
  <h1 className="heading">Namaste React</h1>
);

const HeadingComponent = () => (
  <div>
    <Title />
    <h2>This is a Namaste React Functional Component</h2>
    <h3>This is a Namaste React Functional Component</h3>
  </div>
);

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<HeadingComponent />);

export default HeadingComponent;

```
⸻

🧱 Component Composition
	•	Using one component inside another is called Component Composition.
	•	Example: Rendering <Title /> inside <HeadingComponent />.

⸻

🔮 Superpowers of JSX
	•	You can write any JavaScript expression inside {}.

const name = "React";
const element = <h1>Hello, {name}!</h1>;


	•	JSX automatically sanitizes data, preventing XSS attacks.

⸻

⚡ Ways to Call a Component
```
const HeadingComponent = () => (
  <div>
    {Title()}
    <Title />
    <Title></Title>
    <h2>This is a Namaste React Functional Component</h2>
    <h3>This is a Namaste React Functional Component</h3>
  </div>
);

```
⸻

📌 ToDo
	•	Deep dive into Babel:
	•	How it transpiles JSX to JS
	•	Plugins and presets
	•	Role in modern React apps

⸻

