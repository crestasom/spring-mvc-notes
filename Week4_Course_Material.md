# Week 4: Data Persistence Evolution - JDBC to Spring Data JPA
**Course:** DEV 615 - Advanced Web Programming
**Outline:** JDBC, Hibernate, ORM Mappings, Spring Data JPA, Logging, and Transactions.

---

## 1. JDBC (Java Database Connectivity)

### 1.1 Introduction
JDBC is the standard Java API for database-independent connectivity. It provides methods to query and update data in a database using SQL. While lower-level than ORMs, it offers the highest performance and total control over SQL.

### 1.2 Sample Code: JDBC CRUD
```java
public class JdbcExample {
    private static final String URL = "jdbc:mysql://localhost:3306/demo_db";
    private static final String USER = "root";
    private static final String PASS = "password";

    public void saveUser(String name, String email) {
        String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
        try (Connection conn = DriverManager.getConnection(URL, USER, PASS);
             PreparedStatement ps = conn.prepareStatement(sql)) {
            
            ps.setString(1, name);
            ps.setString(2, email);
            ps.executeUpdate();
            System.out.println("User saved successfully!");
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public void fetchUsers() {
        String sql = "SELECT * FROM users";
        try (Connection conn = DriverManager.getConnection(URL, USER, PASS);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            while (rs.next()) {
                System.out.println("ID: " + rs.getInt("id") + ", Name: " + rs.getString("name"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 2. Hibernate Basics

### 2.1 Hibernate vs JDBC
| Feature | JDBC | Hibernate (ORM) |
| :--- | :--- | :--- |
| **SQL** | Manual SQL writing (Select *...) | Generates SQL automatically |
| **Mapping** | Manual mapping (rs.getString -> obj) | Automatic mapping via Annotations |
| **Boilerplate** | High (Try-catch, closing resources) | Low (Handled by Session) |
| **Caching** | No built-in caching | 1st and 2nd level caching |
| **Portability** | DB specific SQL | DB independent (Dialects) |

### 2.2 Simple CRUD in Hibernate
```java
@Entity
@Table(name = "employees")
public class Employee {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}

// Service Logic
public void hibernateCrud(Employee emp) {
    Session session = sessionFactory.openSession();
    Transaction tx = session.beginTransaction();
    
    // Create
    session.save(emp); 
    
    // Read
    Employee loaded = session.get(Employee.class, 1L);
    
    // Update
    loaded.setName("Updated Name");
    session.update(loaded);
    
    // Delete
    session.delete(loaded);
    
    tx.commit();
    session.close();
}
```

---

## 3. ORM Mappings

### 3.1 One-to-One
One user has exactly one profile.
```java
@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}
```

**Equivalent SQL Schema**:
```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    profile_id BIGINT UNIQUE,
    FOREIGN KEY (profile_id) REFERENCES user_profile(id)
);
```

### 3.2 Many-to-One / One-to-Many
Many employees belong to one department.
```java
@Entity
public class Employee {
    @ManyToOne
    @JoinColumn(name = "dept_id")
    private Department department;
}

@Entity
public class Department {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
}
```

**Equivalent SQL Schema**:
```sql
CREATE TABLE department (
    id BIGINT PRIMARY KEY
);

CREATE TABLE employees (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    dept_id BIGINT,
    FOREIGN KEY (dept_id) REFERENCES department(id)
);
```

### 3.3 Many-to-Many
Many students take many courses.
```java
@Entity
public class Student {
    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses;
}
```

**Equivalent SQL Schema**:
```sql
CREATE TABLE student (id BIGINT PRIMARY KEY);
CREATE TABLE course (id BIGINT PRIMARY KEY);

-- Join Table
CREATE TABLE student_course (
    student_id BIGINT,
    course_id BIGINT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES student(id),
    FOREIGN KEY (course_id) REFERENCES course(id)
);
```

### 3.4 Fetching Strategies: Lazy vs Eager
How data is loaded from the database impacts performance significantly.

#### 1. Lazy Loading (Default for `@OneToMany`, `@ManyToMany`)
Data is loaded only when you actually access the collection or proxy field.
- **Proxy**: Hibernate creates a "placeholder" object (Proxy) instead of the real object.
- **N+1 Problem**: If you fetch N parents and then access a lazy collection for each, you'll trigger N additional queries.
```java
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
private List<Employee> employees;
```

#### 2. Eager Loading (Default for `@ManyToOne`, `@OneToOne`)
Data is loaded immediately with the parent entity using a JOIN.
```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "dept_id")
private Department department;
```

### 4.4 Practical Example: N+1 Problem & Solution
**The Problem**:
```java
// This triggers 1 query for all employees
List<Employee> employees = repository.findAll(); 

for (Employee emp : employees) {
    // This triggers 1 query for EACH employee's department (N queries)
    System.out.println(emp.getDepartment().getName());
}
```
**The Solution (JOIN FETCH)**:
```java
@Query("SELECT e FROM Employee e JOIN FETCH e.department")
List<Employee> findAllWithDepartments();
```

#### Why the `FETCH` word is critical:
- **Normal `JOIN`**: Hibernate joins the table in SQL but **only selects columns for the main entity** (`Employee`). When you access `emp.getDepartment()`, it still thinks the collection is lazy and might trigger another query if not initialized.
- **`JOIN FETCH`**: Hibernate joins the table **and selects all columns for BOTH entities**. It then populates the `Department` object inside each `Employee` immediately. This is the ultimate fix for N+1.

---

## 5. Hibernate Caching
Caching reduces database hits by storing objects in memory.

### 5.1 First-Level Cache (Session Level)
**Example**:
```java
Session session = sessionFactory.openSession();
Employee emp1 = session.get(Employee.class, 1L); // Hits DB
Employee emp2 = session.get(Employee.class, 1L); // Returns from Cache (No SQL)
session.close();
```

### 5.2 Second-Level Cache (SessionFactory Level)
- **Scope**: Shared across multiple sessions.
- **Behavior**: Optional (needs providers like Ehcache). Useful for data that doesn't change frequently.

### 5.3 Query Cache
- **Scope**: Specifically caches results of specific queries (SQL + Params).

### 5.4 Persistence Context: Flush vs Commit
The Persistence Context is Hibernate's internal store of "managed" entities.
- **Save is not immediate**: When you call `session.save(user)`, Hibernate just adds it to the context. It doesn't hit the DB immediately.
- **Flush**: Synchronizes the context with the DB (sends SQL). Happens automatically before queries or at the end of a transaction.
- **Commit**: Makes the changes permanent in the DB.

```java
entityManager.persist(user); // Entity is managed, but no SQL yet
entityManager.flush();       // SQL INSERT is sent to DB
// ... Transaction finishes and commits ...
```

---

## 6. Spring Data JPA
Spring Data JPA adds a layer on top of JPA to reduce boilerplate code to zero for standard operations.

```java
public interface EmployeeRepository extends JpaRepository<Employee, Long> {
    // Custom finder method based on property name (SELECT * FROM employees WHERE name LIKE %name%)
    List<Employee> findByNameContaining(String name);
    
    // Pagination & Sorting
    Page<Employee> findAll(Pageable pageable);
    
    // Native query
    @Query(value = "SELECT * FROM employees WHERE email = ?1", nativeQuery = true)
    Employee findByEmailAddress(String email);
}
```

### 6.1 Native vs JPQL Queries
- **JPQL (@Query)**: Target entities and properties. Database independent.
  - `SELECT e FROM Employee e WHERE e.name = :name`
- **Native Query (`nativeQuery = true`)**: Target tables and columns. Database specific (uses raw SQL).
  - `SELECT * FROM employees WHERE emp_name = :name`

### 6.2 Subqueries in JPQL
Useful for fetching main items based on attributes of related fields.
**Example: Fetch users who have a specific profile type.**
```java
@Query("SELECT u FROM User u WHERE u.profile.id IN " +
       "(SELECT p.id FROM UserProfile p WHERE p.type = 'PREMIUM')")
List<User> findPremiumUsers();
```


---

## 6. Logging and Error Handling

### 6.1 Logging (SLF4J + Logback)
Never use `System.out.println`. Use a logger.
```java
@Slf4j
@Service
public class PaymentService {
    public void process() {
        log.info("Starting payment...");
        try {
            // logic
        } catch (Exception e) {
            log.error("Payment failed: {}", e.getMessage());
        }
    }
}
```

### 6.2 Global Error Handling
Use `@ControllerAdvice` to handle DB errors globally.
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<String> handleNotFound(EntityNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

## 7. Transaction Management
Transactions ensure **ACID** properties. In Spring, we use `@Transactional`.

### 7.1 Propagation & Isolation
- **Propagation**: Defines how transactions relate to each other (e.g., `REQUIRED`, `REQUIRES_NEW`).
- **Isolation**: Defines the data visibility between concurrent transactions (e.g., `READ_COMMITTED`, `SERIALIZABLE`).

```java
@Service
public class OrderService {
    @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.READ_COMMITTED)
    public void placeOrder(Order order) {
        inventoryService.updateCount(order);
        paymentService.chargeUser(order);
        // If either fails, the whole transaction rolls back!
    }
}
```

### 7.2 Internal Mechanics: How it Works
1. **Proxy Pattern**: When you call a `@Transactional` method, you are actually calling a method on a **Spring-generated Proxy** object.
2. **AOP (Aspect Oriented Programming)**: The proxy starts a transaction before your method runs and commits/rolls back after it finishes.
3. **Rollback Rules**:
   - **Runtime Exceptions** (Unchecked): Triggers automatic rollback by default.
   - **Checked Exceptions**: Does **NOT** trigger rollback unless specified (e.g., `@Transactional(rollbackFor = Exception.class)`).

