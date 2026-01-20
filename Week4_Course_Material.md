# Week 4: Data Persistence Evolution - JDBC to Spring Data JPA
**Course:** DEV 615 - Advanced Web Programming
**Duration:** ~5-6 Hours of Content
**Target Audience:** Postgraduate Students

---

## 📚 Module Overview
This week explores the critical layer of any enterprise application: **Data Persistence**. We will trace the history and evolution of database interaction in Java, starting from low-level JDBC to the automated magic of Spring Data JPA. Understanding the "hard way" (JDBC) is essential to appreciating (and debugging) the "easy way" (Hibernate/JPA).

### Learning Objectives (CLOs Covered: 2, 6)
- Implement raw **JDBC** connections and handle `SQLException`.
- Apply the **DAO (Data Access Object)** pattern to decouple business logic from persistence.
- Understand **ORM (Object-Relational Mapping)** concepts and the **Hibernate** lifecycle (Transient, Persistent, Detached).
- Configure **Spring ORM** to manage Hibernate Sessions and Transactions.
- Master **Spring Data JPA** repositories to eliminate boilerplate code.

---

## 📖 Part 1: The Foundation - JDBC & The DAO Pattern (1.5 Hours)

### 1.1 JDBC Architecture
**JDBC (Java Database Connectivity)** is the industry standard API for database-independent connectivity between the Java programming language and a wide range of databases.

**Core Components:**
1.  `DriverManager`: Manages a list of database drivers (e.g., MySQL, H2).
2.  `Connection`: Represents a session with the database. Expensive to create!
3.  `PreparedStatement`: Pre-compiled SQL. Protections against **SQL Injection**.
4.  `ResultSet`: A table of data representing a database result set.

### 1.2 The DAO Pattern
We never put SQL code inside a Controller or Service. We use the **Data Access Object (DAO)** pattern.
- **Interface**: Defines *what* we can do (`save`, `findById`).
- **Implementation**: Defines *how* we do it (JDBC, SQL).
- **Benefit**: We can switch the implementation (e.g., from MySQL to Oracle, or JDBC to Hibernate) without breaking the Service layer.

**The "Try-With-Resources" Block:**
Crucial for preventing memory leaks. Java 7+ feature.
```java
String sql = "INSERT INTO users (name, email) VALUES (?, ?)";
try (Connection conn = dataSource.getConnection();
     PreparedStatement ps = conn.prepareStatement(sql)) {
    ps.setString(1, user.getName());
    ps.executeUpdate();
} catch (SQLException e) {
    throw new RuntimeException("DB Error", e);
}
```

---

## 📖 Part 2: The Revolution - Hibernate & ORM (1.5 Hours)

### 2.1 What is ORM?
**Object-Relational Mapping** allows us to map a Java Class (`User`) directly to a Database Table (`users`).
- **Impedance Mismatch**: Java has inheritance, polymorphism, and references. SQL has tables, rows, and foreign keys. ORM bridges this gap.

### 2.2 Hibernate Core Concepts
Hibernate is the most popular ORM framework.
- **`@Entity`**: Marks a POJO as a database object.
- **`SessionFactory`**: Heavyweight, created once per app. A factory for Sessions.
- **`Session`**: Lightweight, not thread-safe. Represents a "Unit of Work". Wraps a JDBC Connection.
- **`Transaction`**: Atomic unit of work.

### 2.3 HQL (Hibernate Query Language)
Instead of selecting columns (`SELECT * FROM users`), we select Objects.
> `FROM User u WHERE u.age > 20`

### 2.4 Pagination (The "Hard" Stuff made Easy)
Implementing `LIMIT` and `OFFSET` in raw SQL varies by database (MySQL `LIMIT`, Oracle `ROWNUM`). Hibernate abstracts this:
```java
Query<User> query = session.createQuery("FROM User", User.class);
query.setFirstResult(0); // Offset
query.setMaxResults(10); // Page Size
List<User> users = query.list();
```

---

## 📖 Part 3: The Integration - Spring ORM & Transaction Management (1 Hour)

### 3.1 Managing the Session
Manually opening and closing Hibernate sessions (`session.openSession()`, `tx.commit()`) is verbose and error-prone.
**Spring ORM** provides `LocalSessionFactoryBean` to configure Hibernate in the Spring Context.

### 3.2 Declarative Transaction Management
Instead of `try-catch-rollback`, we uses **AOP (Aspect Oriented Programming)**.
- **`@Transactional`**: Annotation that tells Spring: "Wrap this method in a transaction. Commit if successful. Rollback if a RuntimeException occurs."

```java
@Service
@Transactional // All methods read/write transactional
public class UserService {
    @Autowired
    private SessionFactory sessionFactory;
    
    public void register(User user) {
        sessionFactory.getCurrentSession().save(user);
        // No commit() call needed!
    }
}
```

---

## 📖 Part 4: The Modern Standard - Spring Data JPA (1 Hour)

### 4.1 "Boilerplate, Begone!"
Even with Spring ORM, we write the same DAO methods (`persist`, `find`) over and over.
**Spring Data JPA** solves this by creating the implementation *for* us at runtime.

### 4.2 JpaRepository
We simply define an interface:
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Magic Method: Spring generates the SQL automatically!
    List<User> findByNameContaining(String name, Pageable pageable);
}
```
That's it. No class. No SQL. No HQL.

---

## 🗣️ Discussion Topic (30 Minutes)

**Topic:** "The Cost of Abstraction: JDBC vs. Hibernate"

**Prompt:**
> "Hibernate makes life easy, but it adds overhead. Identify at least two scenarios where you might still prefer raw JDBC over Hibernate/JPA in a modern Spring Boot application. Also, what is the 'N+1 Select Problem' in ORM?"

**Instructor Answer Key:**
1.  **Batch Processing**: Inserting 1 million rows is faster in JDBC batching than hydrating 1 million Entities.
2.  **Complex Reporting**: Highly complex analytical queries with specific SQL optimizations are hard to model in HQL.
3.  **N+1 Problem**: Fetching a list of `Orders` (1 query), then for each order, Hibernate lazily fetches the `Customer` (N queries). Result: 1 + N queries instead of 1 JOIN.

---

## 🧪 Laboratory Activity: "The Persistence Evolution" (2 Hours)

**Scenario**: You will implement the 'save user' and 'search user' features four times to physically feel the difference in developer experience.

### Lab A: Raw JDBC (`demo-servlet`)
1.  Add `mysql-connector` to `pom.xml`.
2.  In `UserDaoImpl`, use `DriverManager` to connect.
3.  Write raw SQL: `INSERT INTO users...`.
4.  **Observe**: Check the amount of `try-catch` blocks required.

### Lab B: Hibernate Core (`demo-servlet` Extended)
1.  Add `hibernate-core`. Create `hibernate.cfg.xml`.
2.  Create `HibernateUserDaoImpl`.
3.  Use `session.save(user)`.
4.  Implement **Pagination**: Retrieve only records 5-10.

### Lab C: Spring ORM (`demo-spring-annot`)
1.  Add `spring-orm`.
2.  Configure `HibernateTransactionManager` in `AppConfig`.
3.  Add `@Transactional` to `UserService`.
4.  **Observe**: No manual transaction handling code in Java.

### Lab D: Spring Data JPA (`demo-spring-boot`)
1.  Add `spring-boot-starter-data-jpa`.
2.  **Delete** the DAO implementation entirely.
3.  Create `UserRepository` interface extending `JpaRepository`.
4.  Use `PageRequest` to handle pagination effortlessly.

### Deliverables
- A table comparing Lines of Code (LOC) for the `save()` method across the 4 implementations.
- Screenshots of the Pagination result from Lab B or D.
