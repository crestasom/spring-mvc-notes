## 6. Server-side Validation: Trust Nothing from the Browser

Server-side validation is **non-negotiable** in any real application.  
Even if the UI looks perfect, the server must assume that **every request could be malformed or malicious**.

---

### Why validation is needed

Client-side validation (HTML5, JavaScript):

- can be bypassed (disabled JS, custom HTTP clients, curl, Postman)
- is meant only for user convenience
- provides no security guarantees

**Server-side validation is mandatory** because:
- the server is the final authority
- invalid data can corrupt databases
- business rules must be enforced consistently

---

## How validation was done before Spring MVC (Servlet / Spring Core)

Before Spring MVC, validation logic was written manually inside controllers or servlets.

```java
String fullName = request.getParameter("fullName");

if (fullName == null || fullName.trim().isEmpty()) {
    // manual error handling
}

String email = request.getParameter("email");
if (!email.contains("@")) {
    // manual validation
}
````

Problems with this approach:

* validation rules scattered across code
* difficult to reuse
* inconsistent error messages
* tightly coupled to request handling

Spring MVC solves this by **centralizing validation rules in the model**.

---

## Validation Annotations (Declarative Validation)

```java
public class UserForm {

    @NotBlank
    private String fullName;

    @Email
    private String email;

    @NotNull
    private LocalDate dob;
}
```

### What this class represents now

This class is no longer just a data holder.
It now also describes **what valid data looks like**.

---

### New technical words (simple definitions)

* **Bean Validation**
  A standard Java framework (Jakarta Validation) for validating objects.

* **Constraint annotation**
  An annotation that defines a validation rule for a field.

Examples:

* `@NotNull` → value must not be `null`
* `@NotBlank` → must contain text
* `@Email` → must match email format

---

### Why this approach is powerful

Advantages:

* validation rules live next to the data
* rules are reusable
* consistent behavior across controllers
* easy to read and maintain

Validation becomes **declarative**, not procedural.

---

## Triggering Validation in Spring MVC

Validation annotations alone do nothing.
Spring must be told **when to validate**.

---

### Controller with validation

```java
@PostMapping("/users")
public String submit(@Valid @ModelAttribute UserForm form,
                     BindingResult result) {

    if (result.hasErrors()) {
        return "form";
    }
    return "success";
}
```

---

### New technical words

* **@Valid**
  Tells Spring: “Validate this object after binding.”

* **BindingResult**
  An object that stores:

  * validation errors
  * binding errors
  * type conversion errors

Important rule:

> `BindingResult` must come **immediately after** the validated argument.

---

## What happens internally during validation (Step-by-step)

This happens **after data binding and type conversion**.

---

### Step 1 — Data binding completes

Spring finishes:

* creating the object
* binding request parameters
* converting data types (for example, date strings)

---

### Step 2 — Validation is triggered

Spring sees `@Valid` and:

* locates the Bean Validation provider (Hibernate Validator by default)
* runs validation rules on the object

---

### Step 3 — Each constraint is evaluated

For each field:

* `@NotBlank` checks if the string is empty
* `@Email` checks email format
* `@NotNull` checks for null values

---

### Step 4 — Errors are collected

If any rule fails:

* an error is created
* error is linked to a specific field
* error is added to `BindingResult`

At this point, the controller is **not yet called**.

---

### Step 5 — Controller method executes

Spring now calls:

```java
submit(form, result)
```

The controller decides:

* whether to return the form again
* or proceed with business logic

---

## Why `BindingResult` is critical

Without `BindingResult`:

* Spring throws an exception
* request fails immediately

With `BindingResult`:

* errors can be handled gracefully
* form can be redisplayed with messages
* user experience improves

---

## Common Validation Example (Realistic)

```java
public class UserForm {

    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 50)
    private String fullName;

    @Email(message = "Invalid email format")
    @NotBlank
    private String email;

    @Past(message = "Date of birth must be in the past")
    private LocalDate dob;
}
```

This shows:

* multiple constraints per field
* business rules embedded in the model

---

## Comparison: Old vs Spring MVC Validation

| Aspect      | Old Approach | Spring MVC |
| ----------- | ------------ | ---------- |
| Location    | Controller   | Model      |
| Reusability | Low          | High       |
| Readability | Low          | High       |
| Consistency | Hard         | Easy       |
| Testing     | Hard         | Easy       |

---

## 7. How Everything Connects (End-to-End Flow)

Spring MVC follows a **strict, ordered pipeline**.

---

### End-to-end request flow

```
Browser submits form (POST)
        ↓
Request parameters created
        ↓
DispatcherServlet receives request
        ↓
Controller selected
        ↓
Data binding creates object
        ↓
Type conversion (dates, numbers)
        ↓
Validation executed
        ↓
Errors collected (if any)
        ↓
Controller decides response
        ↓
View rendered or redirect sent
```

---

## Where each concept fits

* **Data binding**
  Turns request data into Java objects

* **Formatter / Converter**
  Translates strings into types

* **Validator**
  Checks if the object is valid

* **BindingResult**
  Stores problems found during binding or validation

---

## Mental Model (Easy to Remember)

* **Controller** → traffic police
* **@ModelAttribute / @RequestBody** → data carrier
* **Binder / Formatter** → translator
* **Validator** → rule checker
* **View** → final output

---

## Final takeaway

Validation is not an optional feature — it is a **core responsibility of the server**.

Spring MVC:

* makes validation declarative
* integrates it into the request lifecycle
* keeps controllers clean
* enforces consistent business rules

Everything works together as a single, predictable pipeline.