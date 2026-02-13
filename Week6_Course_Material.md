# Week 6: Software Testing & BDD in Spring Boot
**Course:** DEV 615 - Advanced Web Programming
**Duration:** ~6 Hours (Lecture + Lab)
**Target:** Postgraduate Students

---

## 1. Overview of Software Testing
Software testing is a critical part of the SDLC. In modern Java applications, we follow the **Testing Pyramid** strategy:
*   **Unit Tests (Base):** Test individual components (Services, Mappers) in isolation.
*   **Integration Tests (Middle):** Test interaction between components (Controller to Service, Repository to DB).
*   **End-to-End (E2E) / BDD (Top):** Test the entire user flow from the perspective of the user.

---

## 2. Unit Testing with JUnit 5 & Mockito
JUnit 5 is the standard for Java testing, and Mockito allows us to "mock" dependencies to isolate the unit.

### 2.1 The `@ExtendWith(MockitoExtension.class)`
Instead of loading the full Spring context, we use Mockito to inject dependencies.
```java
@ExtendWith(MockitoExtension.class)
public class ProductServiceTest {
    @Mock
    private ProductRepository repository;
    @InjectMocks
    private ProductService service;
}
```

### 2.2 Mocking vs. Spying
*   **Mock:** A complete "fake" implementation. Returns `null` or default values unless configured.
*   **Spy:** A "partial" mock. Calls real methods unless a specific behavior is stubbed.

---

## 3. Web Layer Testing with `MockMvc`
Testing Controllers without starting a full Servlet Container (like Tomcat) is faster and more reliable.

### 3.1 `@WebMvcTest`
Focuses only on the Web layer (Spring MVC, Security, Interceptors).
```java
@WebMvcTest(ProductController.class)
public class ProductControllerTest {
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private ProductService productService;
}
```

---

## 4. Behavior Driven Development (BDD) with Cucumber
BDD bridges the gap between technical and non-technical stakeholders using **Gherkin** syntax (`Given`, `When`, `Then`).

### 4.1 Feature Files
Written in human-readable language.
```gherkin
Feature: User Login
  Scenario: Success
    Given the user is on login page
    When the user enters valid credentials
    Then the user redirected to dashboard
```

### 4.2 Step Definitions
Java code that executes the logic for each Gherkin step.

---

## 5. Lab Activity: Implementing Test Cases
1.  **Task 1:** Write Unit Tests for the `UserService` including success and failure scenarios (e.g., user not found).
2.  **Task 2:** Implement a Cucumber feature for "Product Search" with at least 2 scenarios.
3.  **Task 3:** Perform a "Dry Run" using the `CucumberTestRunner`.

---

## 6. Best Practices
*   **F.I.R.S.T Principles:** Fast, Independent, Repeatable, Self-Validating, Timely.
*   **TDD (Test Driven Development):** Write the test *before* the implementation.
*   **Wait for Red:** Only write implementation code once you have a failing test.
