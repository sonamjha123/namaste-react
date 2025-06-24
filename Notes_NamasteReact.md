## CORS:

**CORS** is a **security feature** implemented by **browsers** that **controls how web pages can request resources from a different domain** than the one that served the web page.

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


In React, `componentDidMount` is a lifecycle method that is called after a component has been rendered to the DOM. Making API calls in `componentDidMount` is a common pattern for several reasons:

### 1. Ensures the component is mounted

By making API calls in `componentDidMount`, you ensure that the component has been rendered and is mounted in the DOM. This is important because some APIs may require access to the DOM or its elements.

### 2. Prevents unnecessary API calls

If you make API calls in the `constructor` or `render` method, they may be called multiple times, leading to unnecessary requests. `componentDidMount` is only called once, after the initial render, so you can be sure that the API call is only made when the component is first mounted.

### 3. Allows for async data fetching

`componentDidMount` is a good place to fetch data asynchronously, as it allows the component to render initially with a loading state, and then update with the fetched data when it becomes available.

### 4. Is a good practice for separation of concerns

Making API calls in `componentDidMount` helps to separate the concerns of your component. The `render` method should only be concerned with rendering the component, while `componentDidMount` can handle any necessary side effects, such as API calls.

Here's an example:
```jsx
import React, { Component } from 'react';

class MyComponent extends Component {
  state = {
    data: null,
  };

  componentDidMount() {
    fetch('https://api.example.com/data')
      .then(response => response.json())
      .then(data => this.setState({ data }));
  }

  render() {
    if (!this.state.data) {
      return <div>Loading...</div>;
    }

    return <div>Data: {this.state.data}</div>;
  }
}
```

---

## 🔷 **What is a Higher Order Component (HOC)?**

A **Higher Order Component (HOC)** is a function that **takes a component and returns a new component** with additional functionality.

👉 **It's similar to higher-order functions in JavaScript.**
👉 HOCs are commonly used for **code reuse, logic abstraction, and prop manipulation**.

---

## 🔷 **Basic Syntax:**

```jsx
const higherOrderComponent = (WrappedComponent) => {
  return (props) => {
    // Add logic here
    return <WrappedComponent {...props} />;
  };
};
```

---

## 🔷 **Simple Example: Logging Props**

```jsx
import React from 'react';

// HOC definition
const withLogger = (WrappedComponent) => {
  return (props) => {
    console.log('Props:', props);  // Side-effect: logging props
    return <WrappedComponent {...props} />;
  };
};

// Regular Component
const Hello = ({ name }) => {
  return <h1>Hello, {name}!</h1>;
};

// Enhanced Component
const HelloWithLogger = withLogger(Hello);

// Usage
export default function App() {
  return <HelloWithLogger name="John" />;
}
```

---

## 🔷 **Example 2: Authentication Check HOC**

```jsx
const withAuth = (WrappedComponent) => {
  return (props) => {
    const isAuthenticated = false; // Simulated auth check
    if (!isAuthenticated) {
      return <p>Please log in to continue.</p>;
    }
    return <WrappedComponent {...props} />;
  };
};

const Dashboard = () => <h2>Dashboard Page</h2>;

const ProtectedDashboard = withAuth(Dashboard);

export default function App() {
  return <ProtectedDashboard />;
}
```

---

## 🔷 **When to use HOC in Real Projects (for interview answer):**

✅ Reusing logic (like authentication, logging, fetching data)
✅ Injecting props conditionally
✅ Handling permissions or roles
✅ Implementing cross-cutting concerns like theming, tracking, error boundaries, etc.

---

## 🔷 **Important Notes (for interviews):**

| Point                              | Explanation                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| 🔹 Display Name                    | Set `displayName` on HOCs for better debugging.              |
| 🔹 Don't mutate                    | Avoid changing the original component’s behavior directly.   |
| 🔹 Ref forwarding                  | HOCs may block refs—use `React.forwardRef()` if needed.      |
| 🔹 Compose multiple HOCs carefully | Example: `export default withRouter(withAuth(MyComponent));` |

---

Sure! Let's break this down clearly.

---

## **1. Pure Functions**

A **pure function** is a function that:

* **Given the same inputs, always returns the same output.**
* **Does not cause any side effects** (like modifying global variables, DOM manipulation, API calls, etc.).

### **Example of Pure Function:**

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(2, 3)); // Always returns 5
console.log(add(2, 3)); // Still returns 5
```

✅ **Why is this pure?**

* Output depends only on the inputs (`a` and `b`).
* No modification of external state or variables.
* No side effects.

---

## **2. Impure Functions**

An **impure function** is a function that:

* **May return different results for the same inputs**, or
* **Causes side effects** (like logging to console, modifying global variables, changing DOM, etc.).

### **Example of Impure Function:**

```javascript
let count = 0;

function increment() {
  count++;
  return count;
}

console.log(increment()); // 1
console.log(increment()); // 2 (same input, but different result)
```

✅ **Why is this impure?**

* It modifies the external variable `count`.
* Output changes even though no arguments are passed.
* Side effect: modifies global state.

---

### **Another Impure Function Example:**

```javascript
function greet(name) {
  console.log("Hello " + name);
}

greet("Alice");
```

✅ **Why impure?**

* Causes a side effect: logs to the console.
* Its behavior affects the outside world.

---

## **Summary Table:**

| **Aspect**        | **Pure Function** | **Impure Function**            |
| ----------------- | ----------------- | ------------------------------ |
| Output depends on | Inputs only       | Inputs + external factors      |
| Side effects      | None              | Yes (e.g., modify global vars) |
| Predictable       | Always            | Not always                     |
| Example           | `add(a, b)`       | `increment()`, `console.log()` |

---

