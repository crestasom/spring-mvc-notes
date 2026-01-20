# Week 3: Advanced Spring MVC, REST APIs, and Validation
**Course:** DEV 615 - Advanced Web Programming
**Duration:** ~5-6 Hours of Content
**Target Audience:** Postgraduate Students

---

## 📚 Module Overview
This week bridges the gap between traditional server-side rendering and modern API-driven development. We move beyond basic `Controller` logic to explore **Data Binding**, **Validation**, and **RESTful Web Services**. We will understand how Spring MVC handles the "Model" in depth and how to build services that communicate via JSON rather than HTML.

### Learning Objectives (CLOs Covered: 1, 2, 6)
- Understand the internal workflow of Spring MVC's Data Binding.
- Implement robust server-side validation using Jakarta Bean Validation (Hibernate Validator).
- Differentiate between SOAP and REST architectures.
- Build RESTful APIs using `RestController` and `ResponseEntity`.
- Handle JSON serialization and deserialization with Jackson.

---

## 📖 Part 1: Advanced Spring MVC & Data Binding (1.5 Hours)

### 1.1 The Spring MVC Request Lifecycle (Deep Dive)
While we know the `DispatcherServlet` is the front controller, let's analyze the **HandlerAdapter** and **ArgumentResolver** phase.

1.  **Request Arrival**: `DispatcherServlet` receives the HTTP Request.
2.  **Handler Mapping**: Determines *which* controller method to call.
3.  **Handler Adapter**: This is key. It invokes the method. But how does it get the arguments?
    *   *Argument Resolvers* inspect the method signature.
    *   If it sees `HttpServletRequest`, it passes the raw request.
    *   If it sees a POJO (e.g., `Employee`), it triggers **Data Binding**.

### 1.2 Data Binding Explained
**Data Binding** is the process of converting HTTP request parameters (strings) into Java Objects. Spring uses the `WebDataBinder`.

```java
// The "Magic" happens here
@PostMapping("/register")
public String register(@ModelAttribute Employee employee) { ... }
```

**How it works:**
1.  Spring instantiates `Employee`.
2.  it looks for request parameters matching field names (`name`, `email`).
3.  Type Conversion: Converts "25" (String) to `int age`.
4.  The populated object is injected into the method.

### 1.3 Validation (The Guard Rails)
Data binding happens *before* validation. We must validate the bound object before using it.
Spring integrates with **Jakarta Bean Validation (JSR-380)**.

**Key Annotations:**
- `@NotNull`: Must not be null.
- `@NotEmpty`: Not null, length > 0.
- `@NotBlank`: Not null, trimmed length > 0.
- `@Size(min=x, max=y)`: String or Collection size.
- `@Min(x) / @Max(y)`: Numeric bounds.
- `@Email`: Valid email format.

**Implementation Logic:**
You must pair validation annotations in the DTO with `@Valid` in the Controller.

```java
// DTO
public class Employee {
    @NotBlank(message = "Name is required")
    private String name;
}

// Controller
public String submit(@Valid @ModelAttribute Employee emp, BindingResult result) {
    if (result.hasErrors()) {
        return "formView"; // Return to form with errors
    }
    return "successView";
}
```
*Note: `BindingResult` MUST immediately follow the `@Valid` object argument.*

---

## 📖 Part 2: Web Services & REST (2 Hours)

### 2.1 Evolution: SOAP vs. REST
Historically, web services used **SOAP (Simple Object Access Protocol)**.
- **SOAP**: XML-based, rigid, strict standards (WSDL), transport agnostic (can use SMTP, HTTP). Heavyweight.
- **REST (Representational State Transfer)**: Architectural style, not a protocol. Uses standard HTTP methods (GET, POST, PUT, DELETE). Lightweight, usually JSON.

| Feature | SOAP | REST |
| :--- | :--- | :--- |
| **Protocol** | XML only | HTTP (JSON, XML, Text) |
| **State** | Stateful (often) | Stateless |
| **Caching** | Difficult | Built-in HTTP Caching |
| **Usage** | Enterprise / Legacy Banking | Modern Web / Mobile APIs |

### 2.2 Building REST Services in Spring
Spring 4.0 introduced `@RestController`, which combines `@Controller` and `@ResponseBody`.

**The `@ResponseBody` Annotation:**
Tells Spring: "Do not interpret the return value as a View Name. Write it directly to the Body of the HTTP Response."
Spring uses `HttpMessageConverters` (usually Jackson) to serialize Objects to JSON.

**The `@RequestBody` Annotation:**
Tells Spring: "Read the Body of the HTTP Request and deserialize it into this Object."

### 2.3 The `ResponseEntity` Wrapper
For APIs, returning just the object isn't enough. We need control over the **HTTP Status Code** and **Headers**.
`ResponseEntity<T>` is a wrapper that gives us this control.

```java
@PostMapping("/api/employees")
public ResponseEntity<?> createEmployee(@RequestBody Employee employee) {
    if (invalid(employee)) {
        return ResponseEntity.badRequest().body("Invalid Data"); // 400 Bad Request
    }
    service.save(employee);
    return ResponseEntity.status(HttpStatus.CREATED).body(employee); // 201 Created
}
```

---

## 🗣️ Discussion Topic (30 Minutes)

**Topic:** "Dependency Injection in the Request Scope vs Singleton Scope"

**Prompt:**
> "How is dependency injection invoked when a client's request is captured by a controller then passed to services? Most Spring Beans (Controllers, Services) are Singletons by default. What happens if multiple users hit the Controller at the exact same time? Does the Controller's member variable get overwritten?"

**Instructor Answer Key / Talking Points:**
1.  **Singleton Nature**: Explain that there is only *one* instance of the Controller.
2.  **Thread Safety**: Spring Controllers should be **stateless**. We should *not* store request-specific data (like `currentUser` or `formInput`) in instance fields of the Controller.
3.  **Method Scope**: Request data is passed through method arguments (`HttpServletRequest`, DTOs). Each thread gets its own stack frame, keeping local variables isolated.
4.  **Request Scope Beans**: Mention `@Scope("request")` for rare cases where we *need* a bean tied to an HTTP request lifecycle (e.g., a Shopping Cart).

---

## 🧪 Laboratory Activity: "Hybrid Employee Registration" (1.5 - 2 Hours)

### Scenario
We are building a modern registration flow. Instead of a traditional page reload, the client (Browser) will send data as **JSON** to a **REST API**. The server will validate it and return a JSON response. The client will then decide whether to show an error or redirect to a summary page.

### Requirements
1.  **Backend API**:
    - Build a REST endpoint `POST /api/employees`.
    - Accept JSON payload.
    - Validate fields: Name (Non-empty), Email (Valid format), Contact (Size 10-15).
    - If valid: Save to an in-memory list and return HTTP 200 + Message.
    - If invalid: Return HTTP 400 + List of error messages.
2.  **Frontend**:
    - `register.html`: Standard HTML form.
    - **JavaScript**: Intercept the 'submit' event. Construct a JSON object. Use `fetch()` to call the API.
    - Handle the response:
        - *Success*: Redirect window to `/employees/summary`.
        - *Error*: Display error messages above the form dynamically.
3.  **Tools**:
    - Use Postman or `curl` to test the API independently before writing the JavaScript.

### Step-by-Step Guide
1.  **Add Dependency**: Ensure `spring-boot-starter-validation` is in `pom.xml`.
2.  **DTO**: Create `EmployeeDTO` with annotations (`@NotBlank`, `@Email`).
3.  **Controller**:
    - Create `EmployeeAPIController`. Use `@RestController`.
    - Create `EmployeeViewController`. Use `@Controller` for serving HTML.
4.  **JS Logic**:
    ```javascript
    // Concept code
    const data = { name: document.getElementById('name').value, ... };
    fetch('/api/employees', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data)
    }).then(response => ...)
    ```

### Deliverables
- A short report with:
    - UML Sequence Diagram (Form -> JS -> Controller -> Service).
    - Screenshot of **Network Tab** showing the JSON payload.
    - Screenshot of Postman request/response.
