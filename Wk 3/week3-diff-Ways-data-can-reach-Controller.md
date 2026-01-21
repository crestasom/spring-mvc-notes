## 3. Different Ways Data Can Reach the Controller (and Why They Exist)

In a web application, **not all data comes from HTML forms**.

Data can live in different parts of an **HTTP request**, such as:
- query string
- URL path
- request body
- headers
- cookies
- session
- raw input/output streams

Spring MVC supports **multiple ways to access data** because each source serves a **different purpose**.

Before Spring MVC, most of this was handled manually using **Servlet APIs**.  
Spring MVC builds on top of that, but still keeps those low-level options available when needed.

---

## 3.1 Low-level Access: `HttpServletRequest` / `HttpServletResponse`

### Where this fits in the Spring MVC flow

```

Browser
↓
HTTP Request
↓
Servlet Container (Tomcat)
↓
HttpServletRequest / HttpServletResponse
↓
DispatcherServlet
↓
Controller

````

Before Spring MVC does anything, the request already exists as:
- `HttpServletRequest`
- `HttpServletResponse`

Spring MVC is built **on top of the Servlet API**, not instead of it.

---

### Context: Sometimes you need full control

```java
@GetMapping("/raw")
public void raw(HttpServletRequest request,
                HttpServletResponse response) throws IOException {

    String name = request.getParameter("fullName");
    response.getWriter().write(name);
}
````

### What this code is doing

* Reads raw request data using `getParameter`
* Writes directly to the HTTP response
* Bypasses Spring’s abstraction layers

This style looks very similar to **pure Servlet code**.

---

### How this was done before Spring MVC (Servlet / Spring Core)

Before Spring MVC:

* Controllers *were* Servlets
* You always worked at this level

```java
protected void doGet(HttpServletRequest request,
                     HttpServletResponse response)
        throws IOException {

    String name = request.getParameter("fullName");
    response.getWriter().write(name);
}
```

Spring Core did **not** change this.
It only helped with dependency injection, not web request handling.

---

### Why Spring still allows low-level access

Spring MVC does **not forbid** low-level access because:

* some tasks cannot be abstracted cleanly
* backward compatibility matters
* advanced use cases need raw control

Spring prefers higher-level APIs, but does not block you when you need full control.

---

### When low-level access is commonly used

#### 1️⃣ Reading session manually

##### Context

Session data is stored **server-side** and associated with a user.

```java
HttpSession session = request.getSession();
session.setAttribute("userId", 10);
```

##### Why not higher-level APIs?

* legacy applications rely heavily on session access
* some session usage is highly custom
* Spring cannot always guess how session data should be handled

---

#### 2️⃣ Streaming files

##### Context

Large files should **not** be loaded fully into memory.

```java
OutputStream out = response.getOutputStream();
InputStream in = new FileInputStream(file);

IOUtils.copy(in, out);
```

##### Why low-level is better here

* direct control over streams
* better performance
* avoids buffering surprises
* essential for large downloads/uploads

---

#### 3️⃣ Legacy code

Older Java web applications were built using:

* `HttpServlet`
* `doGet()` / `doPost()`

Spring MVC integrates with this style instead of breaking it.

This allows:

* gradual migration
* reuse of existing systems

---

### Why Spring MVC tries to move you away from low-level access

#### 1️⃣ Verbose code

**Low-level**

```java
String email = request.getParameter("email");
```

**Spring MVC**

```java
@RequestParam String email
```

Less code means:

* fewer bugs
* better readability

---

#### 2️⃣ Tight coupling to the Servlet API

Low-level controllers depend directly on:

* Servlet API
* Tomcat / Jetty
* HTTP-specific details

This makes:

* reuse harder
* refactoring harder
* testing harder

Spring MVC reduces this coupling.

---

#### 3️⃣ Harder to test

With `HttpServletRequest`, tests require:

* mock request
* mock response
* mock session
* mock streams

This makes unit tests heavy and fragile.

Spring MVC annotations:

* are easier to mock
* allow simpler tests
* support cleaner unit testing

---

#### 4️⃣ Mixing responsibilities

Low-level code mixes:

* request parsing
* business logic
* response writing

Spring MVC prefers:

* controllers → coordination
* services → business logic
* views → rendering

This separation improves maintainability.

---

## 3.2 `@RequestParam`: Small, Simple Inputs

### Context

Used when data is:

* small
* independent
* usually part of the query string or form fields

```java
@GetMapping("/search")
public String search(@RequestParam String q,
                     @RequestParam(defaultValue = "1") int page) {
    return "results";
}
```
Note: Recommended to use in GET but can be used in other HTTP methods like POST for lesser data units [ amount 255 characters]

### New technical words

* **Query string**: Data after `?` in the URL
  Example: `?q=spring&page=1`
* **defaultValue**: Value used when the parameter is missing

---

### How this compares to Spring Core / Servlets

Before Spring MVC:

```java
String q = request.getParameter("q");
int page = Integer.parseInt(request.getParameter("page"));
```

Now:

* parsing
* default values
* error handling

are handled by Spring.

---

### When used

* filters
* pagination
* toggles
* search inputs

---

## 3.3 `@PathVariable`: Data Inside the URL

### Context

URLs often identify **resources**.

```java
@GetMapping("/users/{id}")
public String user(@PathVariable long id) {
    return "user";
}
```

### New technical word

* **Path variable**: A value embedded directly in the URL path

---

### Why this exists

Before Spring MVC:

```java
String path = request.getPathInfo();
// manually parse the ID
```

Spring MVC:

* extracts values automatically
* enforces clean URLs

---

### Why it is used

* clean, readable URLs
* REST-style design
* clear resource identification

---

## 3.4 `@RequestBody`: JSON and APIs

### Context

Modern frontends (React, Angular, mobile apps) send **JSON**, not form data.

```java
@PostMapping("/api/users")
public void create(@RequestBody UserForm form) {
}
```

### New technical words

* **Request body**: The main content of the HTTP request
* **JSON**: Text format for structured data
* **Message converter**: Component that converts JSON ↔ Java objects

---

### Comparison with older approaches

Before Spring MVC:

* manually read input stream
* manually parse JSON

```java
InputStream in = request.getInputStream();
// manual JSON parsing
```

Spring MVC:

* automatically converts JSON into Java objects

---

### Important difference

| `@ModelAttribute`   | `@RequestBody`   |
| ------------------- | ---------------- |
| Form / query params | JSON / XML body  |
| Browser forms       | APIs             |
| URL-encoded data    | Raw request body |

---

## 3.5 `@RequestHeader`: Metadata About the Request

```java
@GetMapping("/info")
public String info(@RequestHeader("User-Agent") String agent) {
    return agent;
}
```

### New technical word

* **HTTP header**: Metadata sent with a request (auth info, browser info)

---

### Before Spring MVC

```java
String agent = request.getHeader("User-Agent");
```

Spring MVC:

* injects headers directly into method parameters
* avoids low-level access

---

### When used

* authentication tokens
* content negotiation
* analytics
* debugging

---

## 3.6 `@CookieValue`: Data Stored in the Browser

```java
@GetMapping("/theme")
public String theme(@CookieValue(defaultValue = "light") String theme) {
    return theme;
}
```

### New technical word

* **Cookie**: Small key–value data stored in the browser and sent with requests

---

### Before Spring MVC

```java
Cookie[] cookies = request.getCookies();
// manually search for cookie
```

Spring MVC:

* extracts cookie values directly
* handles defaults safely

---

### When used

* user preferences
* session identifiers
* tracking information

---

## Key takeaway

Spring MVC supports **multiple ways to access request data** because:

* HTTP requests carry data in different places
* not all use cases are the same
* backward compatibility matters

Spring MVC **guides you toward cleaner abstractions**, but still allows raw access when necessary.

The goal is:

> use high-level annotations by default
> drop to low-level APIs only when you truly need them
