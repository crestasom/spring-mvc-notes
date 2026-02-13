# Week 3: Spring MVC, REST Services, and Validation

## Learning Objectives
By the end of this week, you will be able to:
1.  Understand the Role of Spring MVC in modern web development.
2.  Build RESTful APIs using `@RestController`.
3.  Handle JSON data using `@RequestBody`.
4.  Implement Validation logic manually.
5.  Implement Validation using Spring Bean Validation (JSR-380).

---

## 1. Spring MVC & REST Architecture

### Traditional MVC vs REST
- **Traditional MVC**: Controller returns a View (HTML).
- **REST**: Controller returns Data (JSON/XML). The "View" is often a separate JavaScript application (React, Angular) or mobile app.

### @Controller vs @RestController
- `@Controller`: Used for returning views (Thymeleaf templates).
- `@RestController`: A convenience annotation that adds `@Controller` and `@ResponseBody`. It indicates that the return value of methods should be bound to the web response body.

---

## 2. Handling Data with Spring Boot

### @RequestBody
To accept JSON data from a client, we use `@RequestBody`. Spring (via Jackson) automatically deserializes the JSON into a Java Object.

```java
@PostMapping("/api/users")
public String createUser(@RequestBody UserDTO user) {
    // user object is populated from JSON
    return "User saved";
}
```

---

## 3. Validation

Validation is crucial for data integrity. We will explore two approaches.

### Approach 1: Manual Validation (The "Hard Way")
You manually check every field in your Service or Controller.
**Pros**: Full control, no external dependencies.
**Cons**: Verbose, repetitive, mixes business logic with validation logic.

```java
if (user.getName() == null || user.getName().isEmpty()) {
    throw new IllegalArgumentException("Name is required");
}
```

### Approach 2: Bean Validation (The "Spring Way")
Using JSR-380 (Jakarta Bean Validation) annotations.
**Pros**: Declarative, clean, reusable.
**Cons**: Requires learning annotations.

```java
public class UserDTO {
    @NotBlank(message = "Name is required")
    private String name;
    
    @Email
    private String email;
}

// In Controller
public String create(@Valid @RequestBody UserDTO user) {
    // ...
}
```

---

## Lab Activity: Employee Registration System

We will build an Employee Registration feature in our `ecommerce-demo` project using **Spring Boot**.

### Scenario
We need an API to register employees. The client (a web page) will send JSON data. We need to validate this data.

### Step 1: Manual Validation
First, we will implement it the "hard way" to feel the pain of writing boilerplate code.

### Step 2: Spring Validation
Then, we will refactor it to use `@Valid` and standard annotations.

### Lab Instructions
1.  Open `ecommerce-demo`.
2.  Add dependencies (if not present).
3.  Create `EmployeeDTO`.
4.  Create `EmployeeAPIController`.
5.  Create `EmployeeService`.
6.  Test using `stresstest` or `curl`.

---
