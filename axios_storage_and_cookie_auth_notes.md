# Axios, Interceptors, Storage and Cookies Notes

# Axios Interceptors

Interceptors are middleware-like functions that run automatically:
- before request goes
- after response comes

They help centralize common API logic.

## Common Uses

### Request Interceptor
- Add auth token
- Add headers
- Logging
- Modify request config

### Response Interceptor
- Handle errors globally
- Handle 401 unauthorized
- Token refresh logic
- Transform response
- Redirect user

---

# Why Storage is Needed

Backend sends token after login.

Example:

```json
{
  "token": "abc123"
}
```

Frontend must store this token somewhere because React state disappears after refresh.

Storage helps preserve data even after:
- refresh
- tab changes
- reopening browser

---

# Types of Storage

1. localStorage
2. sessionStorage
3. Cookies

---

# localStorage

## Store Data

```js
localStorage.setItem("token", "abc123");
```

## Get Data

```js
localStorage.getItem("token");
```

## Remove Data

```js
localStorage.removeItem("token");
```

## Characteristics

Data survives:
- refresh
- browser close
- system restart

Until manually removed.

---

# sessionStorage

Similar to localStorage but removed when tab/browser closes.

## Characteristics

Data survives:
- refresh

Data does NOT survive:
- tab close
- browser close

---

# Important Difference

| Storage | JS Can Access? | Survives Browser Close? |
|---|---|---|
| localStorage | Yes | Yes |
| sessionStorage | Yes | No |

Main difference:
- persistence/lifetime
- not security

---

# JSON.stringify and JSON.parse

Storage can only store strings.

Wrong:

```js
localStorage.setItem("user", {
  name: "Leo"
});
```

Correct:

```js
localStorage.setItem(
  "user",
  JSON.stringify({ name: "Leo" })
);
```

Retrieve:

```js
const user = JSON.parse(
  localStorage.getItem("user")
);
```

---

# Interceptor + Storage Flow

```txt
Backend gives token
        ↓
Frontend stores token
        ↓
Axios interceptor reads token
        ↓
Adds token to every request
```

Example:

```js
const token = localStorage.getItem("token");

config.headers.Authorization = `Bearer ${token}`;
```

---

# Security Concern With localStorage

Any JavaScript running on page can access localStorage.

Example:

```js
localStorage.getItem("token");
```

If malicious script runs on website, token may get stolen.

This is related to:

# XSS (Cross Site Scripting)

sessionStorage also has same accessibility issue.

---

# Cookies

Cookies are small pieces of data stored by browser.

Unlike localStorage/sessionStorage, cookies are deeply connected with:
- backend
- HTTP requests
- authentication

---

# localStorage Auth Flow

Frontend manually attaches token.

```js
const token = localStorage.getItem("token");

axios.get("/profile", {
  headers: {
    Authorization: `Bearer ${token}`
  }
});
```

Frontend handles token.

---

# Cookie Auth Flow

Backend sends:

```http
Set-Cookie: token=abc123
```

Browser stores it automatically.

Future requests automatically include:

```http
Cookie: token=abc123
```

Browser handles token.

---

# HttpOnly Cookies

Special secure cookies.

JavaScript cannot access them.

This helps protect token from malicious scripts.

---

# Final Comparison

| Storage Type | JS Can Access? | Survives Browser Close? |
|---|---|---|
| localStorage | Yes | Yes |
| sessionStorage | Yes