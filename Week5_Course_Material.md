# Week 5: Advanced Web Security, Forms & Performance Optimization
**Course:** DEV 615 - Advanced Web Programming
**Duration:** ~6 Hours (Lecture + Lab)
**Target:** Postgraduate Students (Advanced Implementation & Analytical Skills)

---

## 1. Deep Dive: Advanced HTML5 Form Features
Modern web applications rely on rich client-side interactions. HTML5 introduced several semantic elements that reduce the need for heavy JavaScript.

### 1.1 Beyond Simple Inputs
*   **`<datalist>`**: Provides "autocomplete" functionality without extra JS libraries.
    ```html
    <label for="browser">Choose your browser:</label>
    <input list="browsers" name="browser" id="browser">
    <datalist id="browsers">
      <option value="Edge">
      <option value="Firefox">
      <option value="Chrome">
      <option value="Opera">
    </datalist>
    ```
*   **`<output>`**: Represents the result of a calculation.
    ```html
    <form oninput="x.value=parseInt(a.value)+parseInt(b.value)">
      0 <input type="range" id="a" value="50"> 100 +
      <input type="number" id="b" value="50"> =
      <output name="x" for="a b"></output>
    </form>
    ```
*   **Native Client-Side Validation**:
    *   `pattern`: Regex matching.
    *   `title`: Tooltip shown on validation failure.
    *   `required`: Boolean attribute.
    ```html
    <input type="text" pattern="[A-Z]{3}" title="Three letter country code" required>
    ```

*   **Form Data API & AJAX**: Modern apps don't refresh the whole page.
    ```javascript
    const form = document.querySelector('form');
    form.addEventListener('submit', (e) => {
        e.preventDefault();
        const formData = new FormData(form);
        fetch('/api/submit', {
            method: 'POST',
            body: formData,
            headers: { 'Authorization': `Bearer ${token}` }
        }).then(res => res.json()).then(data => console.log(data));
    });
    ```

---

## 2. Advanced Security Architecture
Web security is about defense-in-depth. We move from simple "Login/Logout" to a comprehensive security posture.

### 2.1 The Spring Security Filter Chain (Internal Mechanics)
Spring Security is a chain of Servlet Filters. Each filter has a specific responsibility.
1.  **`SecurityContextPersistenceFilter`**: Loads/Stores `SecurityContext` between requests.
2.  **`LogoutFilter`**: Handles logout logic.
3.  **`UsernamePasswordAuthenticationFilter`**: Authenticates credentials for form-login.
4.  **`DefaultLoginPageGeneratingFilter` / `DefaultLogoutPageGeneratingFilter`**: Generates default UI if none provided.
5.  **`FilterSecurityInterceptor`**: Final authority; decides if the request can proceed to the Controller.

### 2.2 Password Security: Salting & Hashing
Never store plain text passwords. Use **BCrypt** with a high "Cost" factor.
*   **BCrypt Principles**:
    *   **Salt**: A random string added to the password before hashing to prevent rainbow table attacks.
    *   **Cost**: The computational effort required (Default is 10).
```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // Higher cost = slower hashing = more secure
}
```

### 2.3 Advanced Authorization: RBAC (Role-Based Access Control)
Using Annotations to secure methods at the Service layer:
```java
@Service
public class EmployeeService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteEmployee(Long id) {
        // Logic
    }

    @PreAuthorize("hasAnyRole('ADMIN', 'MANAGER')")
    public void updateSalary(Long id, Double amount) {
        // Logic
    }
}
```

### 2.4 JWT vs. Session
| Feature | Session-Based (Traditional) | Token-Based (JWT) |
| :--- | :--- | :--- |
| **Storage** | Server-side (Memory/DB) | Client-side (LocalStorage/Cookies) |
| **Scalability** | Hard (Requires session stickiness/Redis) | Easy (Stateless - any server can verify) |
| **Overhead** | High memory usage on server | High bandwidth (Token sent with every request) |
| **Logout** | Simple (Delete session on server) | Complex (Requires blacklisting or short expiry) |

### 2.5 Detailed JWT Implementation (Java Example)
**Token Generation**:
```java
String token = Jwts.builder()
    .setSubject(user.getUsername())
    .claim("roles", user.getAuthorities())
    .setIssuedAt(new Date())
    .setExpiration(new Date(System.currentTimeMillis() + 3600000)) // 1 hour
    .signWith(SignatureAlgorithm.HS512, secretKey)
    .compact();
```

### 2.6 Modern Architecture: OAuth2 & OpenID Connect (OIDC)
For enterprise-level apps, we use Centralized Auth Servers (Keycloak, Okta, Auth0).
*   **Concepts**:
    *   **Resource Owner**: The User.
    *   **Client**: The Web/Mobile App.
    *   **Authorization Server**: Issues tokens.
    *   **Resource Server**: The API (e.g., our Spring Boot App).
*   **Flow (Authorization Code)**:
    1.  User redirects to Auth Server.
    2.  User logs in and consents.
    3.  Auth Server sends an **Authorization Code** back to Client.
    4.  Client exchanges Code for an **Access Token** on the backend.
    5.  Client uses Token to access protected API.

---

## 3. Vulnerability Analysis: OWASP Top 10
Understanding how to break code is the first step to securing it.

### 3.1 SQL Injection (A03:2021-Injection)
*   **Vulnerable Code**:
    ```java
    String query = "SELECT * FROM users WHERE username = '" + username + "'";
    Statement statement = connection.createStatement();
    ResultSet rs = statement.executeQuery(query); // Attacker can input: admin' OR '1'='1
    ```
*   **Secure Code (Prepared Statement)**:
    ```java
    String query = "SELECT * FROM users WHERE username = ?";
    PreparedStatement pstmt = connection.prepareStatement(query);
    pstmt.setString(1, username); // Safe parameter binding
    ResultSet rs = pstmt.executeQuery();
    ```

### 3.2 Cross-Site Scripting (XSS)
*   **The Threat**: Injecting malicious scripts (JS) into pages seen by other users.
*   **Mitigation**: Use Thymeleaf/JSP escape tags, implement Content Security Policy (CSP) headers.

---

## 4. Performance Optimization & Monitoring
High-scale applications must be fast and stable.

### 4.1 Connection Pooling Tuning (HikariCP)
```properties
# Optimal configurations for production
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.connection-timeout=20000
spring.datasource.hikari.max-lifetime=1200000
```
**Why limit pool size?** Contrary to intuition, a massive pool size (e.g., 500) creates high context-switching overhead on the DB CPU.

### 4.2 Caching Strategies
*   **L1 Cache (Session)**: Hibernate stores objects in the current session.
*   **L2 Cache (SessionFactory)**: Shared across sessions using providers like **Ehcache**.
    ```xml
    <!-- ehcache.xml example -->
    <cache name="org.example.model.Product"
           maxEntriesLocalHeap="1000"
           eternal="false"
           timeToIdleSeconds="300"
           timeToLiveSeconds="600">
    </cache>
    ```

### 4.2 Concurrency Control: Optimistic Locking
When multiple users edit the same record, we need to prevent "data loss".
*   **Implementation**: Add `@Version` column.
```java
@Entity
public class Product {
    @Id @GeneratedValue
    private Long id;
    
    @Version
    private Long version; // Hibernate checks this on every UPDATE
}
```
If two users try to update the same version, the second one gets an `OptimisticLockException`.

### 4.3 Monitoring with Spring Boot Actuator
Actuator provides production-ready features to help you monitor and manage your application.
*   **Endpoints**:
    *   `/actuator/health`: System health.
    *   `/actuator/metrics`: Specific metrics (JVM, Tomacat, JDBC).
    *   `/actuator/prometheus`: Data for Grafana dashboards.
*   **Config**:
    ```properties
    management.endpoints.web.exposure.include=health,metrics,info,env
    ```

### 4.4 Memory Analysis Protocols
1.  **Identify**: High memory usage or `OutOfMemoryError`.
2.  **Capture**: `jmap -dump:format=b,file=heap.bin <pid>`
3.  **Analyze**: Open `heap.bin` in **Eclipse MAT**. Look for "Shallow Heap" (self memory) vs "Retained Heap" (memory held by dependencies).

---

## 5. Lab Activity: Advanced Implementation
Students will create a **Security Dashboard** that:
1.  Authenticates users via JWT.
2.  Uses an Optimized Database connection pool.
3.  Implements **AOP-based Logging** to track query performance.

**Bonus Challenge**: Implement a "Memory Leak" intentionally and use VisualVM to find it.
