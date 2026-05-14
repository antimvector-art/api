# API, Fetch & Axios Learning Notes — Day 1

---

# Table of Contents

1. HTTP Basics
2. Request vs Response
3. Fetch Fundamentals
4. Why `response.json()` is Needed
5. Streams & ReadableStream
6. Promises & Async/Await
7. Fetch Error Handling
8. Loading & Error States
9. GET Requests
10. POST Requests
11. Dynamic Form Requests
12. PATCH Requests
13. DELETE Requests
14. React StrictMode Double Calls
15. AbortController
16. CSRF Token
17. Axios Introduction
18. Fetch vs Axios
19. Important Mental Models
20. Complete Code Snapshots

---

# 1. HTTP Basics

HTTP = communication protocol between:

```text
Frontend  ↔  Backend
```

Frontend sends:

- Request

Backend sends:

- Response

---

# 2. Request vs Response

## Request

Frontend → Backend

Example:

```json
{
  "name": "Leo",
  "email": "leo@gmail.com"
}
```

---

## Response

Backend → Frontend

Example:

```json
{
  "id": 101,
  "name": "Leo",
  "email": "leo@gmail.com"
}
```

---

# 3. Fetch Fundamentals

## Basic Fetch Flow

```text
fetch()
   ↓
Response object
   ↓
response.json()
   ↓
Actual JS data
```

---

## Basic Fetch Example

```js
fetch("https://jsonplaceholder.typicode.com/users")
  .then((response) => {
    return response.json();
  })
  .then((data) => {
    console.log(data);
  })
  .catch((err) => {
    console.log(err);
  });
```

---

# 4. Why `response.json()` is Needed

# VERY IMPORTANT REALITY

`fetch()` gives:

```text
HTTP Response object
```

NOT actual data directly.

---

## The actual response body is:

```text
ReadableStream
```

because browser receives data as stream/chunks from network.

---

# WHY STREAM?

Because response can be HUGE.

Imagine:

- 5GB video
- massive JSON
- file download

Browser cannot always wait for everything first.

So data comes gradually as stream.

---

# NOW IMPORTANT PART

`response.json()`

When we do:

```js
response.json()
```

browser:

```text
Reads stream
   ↓
Collects chunks
   ↓
Converts text to JSON
   ↓
Returns JavaScript object
```

VERY important:

It returns a:

```text
Promise
```

NOT immediate JSON.

---

# Internal Lifecycle

```text
fetch()
   ↓
Promise<Response>
   ↓
response.json()
   ↓
Promise<Actual Data>
   ↓
Final JavaScript Object
```

---

# 5. Streams & ReadableStream

## Important Concept

Browser does NOT instantly receive complete response.

Data arrives in chunks.

```text
Server
   ↓
Chunk 1
Chunk 2
Chunk 3
   ↓
Browser combines them
```

This is why:

```js
response.json()
```

is asynchronous.

---

# 6. Promises & Async/Await

# Fetch Returns Promise

```js
const result = fetch(url);
```

`result` is NOT actual data.

It is:

```text
Promise
```

---

# Async/Await Mental Model

```js
const response = await fetch(url);
```

Means:

```text
Pause THIS async function
until Promise resolves
```

NOT pause whole JavaScript.

---

# Async/Await Version

```js
const fetchUsers = async () => {
  try {
    const response = await fetch(
      "https://jsonplaceholder.typicode.com/users"
    );

    const data = await response.json();

    console.log(data);

  } catch (err) {
    console.log(err);
  }
};
```

---

# 7. Fetch Error Handling

# VERY IMPORTANT

`fetch()` does NOT reject Promise for:

- 404
- 500
- 401
- 403

These are still valid HTTP responses.

---

# Fetch ONLY rejects for:

- network failure
- internet disconnected
- DNS failure
- CORS issues
- aborted requests

---

# Wrong Assumption

```text
404 → catch() runs automatically
```

❌ WRONG

---

# Correct Flow

```text
fetch()
   ↓
404 comes
   ↓
Promise STILL resolves
   ↓
response.ok = false
```

---

# Correct Error Handling

```js
const response = await fetch(url);

if (!response.ok) {
  throw new Error(`HTTP Error: ${response.status}`);
}
```

---

# VERY IMPORTANT CONTROL FLOW

When `throw new Error()` executes:

- current function execution immediately stops
- remaining code below it is skipped
- control jumps directly to nearest catch block

---

# Error Lifecycle

```text
response.ok === false
   ↓
throw new Error(...)
   ↓
remaining code skipped
   ↓
catch(err) executes
```

---

# 8. Loading & Error States

# Why Loading State Needed?

API requests are asynchronous.

Real APIs may take:

- 100ms
- 2 sec
- 10 sec

User needs feedback.

---

# Loading Lifecycle

```text
Component renders
   ↓
API starts
   ↓
loading = true
   ↓
Loading UI visible
   ↓
API completes
   ↓
loading = false
   ↓
Final UI visible
```

---

# finally Block

```js
finally {
  setLoading(false);
}
```

`finally` ALWAYS runs.

Whether:

- success
- failure
- thrown error

---

# Error State

```js
const [error, setError] = useState("");
```

Used to show:

```text
User-friendly error UI
```

instead of only console errors.

---

# 9. GET Requests

## GET Meaning

```text
Fetch/Read data
```

---

# GET Example

```js
const response = await fetch(GETURL);

const data = await response.json();

setUsers(data);
```

---

# GET is Usually Idempotent

Meaning:

```text
Calling multiple times
usually does not change server state
```

---

# 10. POST Requests

# POST Meaning

```text
Create/send data
```

---

# POST Request Structure

```js
fetch(POSTURL, {
  method: "POST",

  headers: {
    "Content-Type": "application/json",
  },

  body: JSON.stringify({
    name: "leo",
    email: "leo@gmail.com",
  }),
});
```

---

# VERY IMPORTANT

We cannot directly send JS object to backend.

So:

```js
JSON.stringify()
```

converts:

```text
JS Object
   ↓
JSON String
```

---

# Content-Type Header

```js
"Content-Type": "application/json"
```

Means:

```text
"Backend, upcoming body is JSON"
```

---

# Serialization vs Deserialization

## Serialization

```text
JS Object → JSON String
```

Example:

```js
JSON.stringify()
```

---

## Deserialization

```text
JSON String → JS Object
```

Example:

```js
response.json()
```

---

# 11. Dynamic Form Requests

# Important Shift

Earlier:

```text
API on component mount
```

Now:

```text
API on user action
```

---

# Controlled Inputs

```js
value={name}
onChange={(e) => setName(e.target.value)}
```

Means:

```text
React state controls input value
```

---

# preventDefault()

```js
e.preventDefault();
```

Prevents browser default form reload.

---

# Form Lifecycle

```text
User types
   ↓
React state updates
   ↓
Submit clicked
   ↓
POST request sent
   ↓
Backend responds
   ↓
UI rerenders
```

---

# 12. PATCH Requests

# PATCH Meaning

```text
Partial update
```

Example:

```js
fetch("/users/1", {
  method: "PATCH",
  body: JSON.stringify({
    name: "Updated Leo"
  })
});
```

---

# REST URL Pattern

| Operation | URL |
|---|---|
| Get all users | `/users` |
| Get one user | `/users/1` |
| Create user | `/users` |
| Update user | `/users/1` |
| Delete user | `/users/1` |

---

# 13. DELETE Requests

# DELETE Meaning

```text
Delete resource
```

---

# DELETE Example

```js
fetch("/users/1", {
  method: "DELETE"
});
```

---

# Common DELETE Response

```text
204 No Content
```

Meaning:

```text
Operation successful
BUT
No response body returned
```

---

# 14. React StrictMode Double Calls

# VERY IMPORTANT

In development mode:

```text
useEffect runs twice
```

because React StrictMode intentionally stress-tests side effects.

---

# Dangerous Scenario

```text
POST request inside useEffect
   ↓
useEffect runs twice
   ↓
POST sent twice
   ↓
Duplicate records created
```

---

# Idempotent vs Non-Idempotent

## Idempotent

```text
Running multiple times gives same result
```

Example:

```text
GET requests
```

---

## Non-Idempotent

```text
Multiple calls change server state repeatedly
```

Example:

```text
POST requests
```

---

# 15. AbortController

# Real Problem

```text
Component mounted
   ↓
API starts
   ↓
User leaves page
   ↓
Component destroyed
   ↓
API finishes later
   ↓
setState on destroyed component
```

Bad.

---

# AbortController Solution

```js
const controller = new AbortController();
```

Think:

```text
Remote control for request
```

---

# Connecting Fetch

```js
fetch(url, {
  signal: controller.signal,
});
```

Means:

```text
This request can now be cancelled
```

---

# Cancelling Request

```js
controller.abort();
```

Immediately stops fetch request.

---

# Cleanup Function

```js
return () => {
  controller.abort();
};
```

Runs when component unmounts.

---

# Abort Flow

```text
Request started
   ↓
Component unmounts
   ↓
cleanup function runs
   ↓
abort()
   ↓
Request cancelled
```

---

# AbortError

Cancelled fetch throws:

```text
AbortError
```

---

# 16. CSRF Token

# What is CSRF?

Cross Site Request Forgery.

---

# Real Problem

Browser automatically sends cookies.

Attacker website can secretly trigger:

```text
POST request to bank.com
```

Browser automatically attaches user cookies.

Server thinks:

```text
Real logged-in user made request
```

---

# CSRF Token Solution

Server gives frontend:

```text
Secret random token
```

Frontend sends token with request.

Example:

```js
headers: {
  "X-CSRF-Token": csrfToken
}
```

---

# Why This Works?

Malicious website:

```text
Cannot access real CSRF token
```

So forged request fails validation.

---

# CSRF Mental Model

Cookie proves:

```text
Who user is
```

CSRF token proves:

```text
Request truly came from trusted frontend
```

---

# 17. Axios Introduction

# What is Axios?

```text
Promise-based HTTP client library
```

Used to simplify HTTP requests.

---

# Axios Mental Model

```text
Higher-level abstraction over HTTP requests
```

---

# Axios GET Example

```js
const response = await axios.get(GETURL);

setUsers(response.data);
```

---

# 18. Fetch vs Axios

| Feature | Fetch | Axios |
|---|---|---|
| Built into browser | ✅ | ❌ |
| Auto JSON parsing | ❌ | ✅ |
| Auto HTTP error throw | ❌ | ✅ |
| response.json() needed | ✅ | ❌ |
| Cleaner POST syntax | ❌ | ✅ |
| Interceptors | ❌ | ✅ |
| Centralized config | Limited | Strong |

---

# Axios Automatically Parses JSON

## Fetch

```js
const data = await response.json();
```

---

## Axios

```js
response.data
```

Already parsed.

---

# Axios Automatically Throws HTTP Errors

## Fetch

Need:

```js
if (!response.ok)
```

---

## Axios

404/500 automatically go to:

```js
catch()
```

---

# Axios POST Syntax

```js
axios.post(url, data, config)
```

---

# Headers in Axios

```js
axios.post(
  "/users",

  {
    name: "Leo"
  },

  {
    headers: {
      Authorization: "Bearer xyz"
    }
  }
);
```

---

# Axios Generic Config Pattern

```js
axios({
  method: "POST",
  url: "/users",
  headers: {},
  data: {}
});
```

Very common in companies.

---

# 19. Important Mental Models

# Frontend UI = Visual Representation of State

```text
State changes
   ↓
UI rerenders
```

---

# APIs Are Asynchronous

```text
Request starts
Response comes later
```

So initial render happens BEFORE data arrives.

---

# Fetch vs Axios Philosophy

## Fetch

```text
Low-level browser primitive
```

---

## Axios

```text
Enterprise-friendly abstraction
```

---

# 20. Complete Code Snapshots

# Complete GET Request Component

```jsx
import { useEffect, useState } from "react";

const Users = () => {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);

        const response = await fetch(
          "https://jsonplaceholder.typicode.com/users"
        );

        if (!response.ok) {
          throw new Error(`Http Error: ${response.status}`);
        }

        const data = await response.json();

        setUsers(data);

      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, []);

  return (
    <>
      <h2>Users</h2>

      {loading && <p>Loading...</p>}

      {error && <p>Error: {error}</p>}

      <ol>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ol>
    </>
  );
};

export default Users;
```

---

# Complete POST Request Component

```jsx
import { useState } from "react";

const Users = () => {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [createdUser, setCreatedUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState("");

  const POSTURL =
    "https://jsonplaceholder.typicode.com/users";

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      setLoading(true);
      setError("");

      const response = await fetch(POSTURL, {
        method: "POST",

        headers: {
          "Content-Type": "application/json",
        },

        body: JSON.stringify({
          name,
          email,
        }),
      });

      if (!response.ok) {
        throw new Error(`HTTP Error: ${response.status}`);
      }

      const data = await response.json();

      setCreatedUser(data);

      setName("");
      setEmail("");

    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <>
      <h2>Create User</h2>

      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Enter name"
          value={name}
          onChange={(e) => setName(e.target.value)}
        />

        <br />
        <br />

        <input
          type="email"
          placeholder="Enter email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
        />

        <br />
        <br />

        <button type="submit">
          Create User
        </button>
      </form>

      {loading && <p>Creating user...</p>}

      {error && <p>Error: {error}</p>}

      {
        createdUser && (
          <>
            <h3>User Created Successfully</h3>

            <p>Name: {createdUser.name}</p>
            <p>Email: {createdUser.email}</p>
            <p>Id: {createdUser.id}</p>
          </>
        )
      }
    </>
  );
};

export default Users;
```

---

# Final Takeaway

You now understand:

✅ request/response lifecycle
✅ fetch internals
✅ response.json()
✅ streams
✅ Promise flow
✅ async/await
✅ loading/error states
✅ response.ok
✅ GET/POST/PATCH/DELETE
✅ JSON.stringify()
✅ serialization/deserialization
✅ form-driven requests
✅ StrictMode duplicate calls
✅ idempotency
✅ AbortController basics
✅ CSRF fundamentals
✅ axios foundations
✅ fetch vs axios

This is already a VERY strong foundation for real-world frontend API work.

