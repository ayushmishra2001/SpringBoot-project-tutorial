# Spring Boot — Complete CRUD Application Blueprint
### Step-by-Step Tutorial with Code Templates
**Stack:** Spring Boot 4.x · Spring Security 6 · Hibernate / JPA · JWT Authentication · MySQL 8 · REST API · Maven

---

## Table of Contents

1. [Introduction – What is Spring Boot?](#1-introduction)
2. [The Tech Stack Explained](#2-tech-stack)
3. [Project Setup & Directory Structure](#3-project-setup)
4. [pom.xml — The Dependency File](#4-pomxml)
5. [application.properties — The Config File](#5-applicationproperties)
6. [Entity Layer — Your Database Table in Java](#6-entity-layer)
7. [Repository Layer — Talking to the Database](#7-repository-layer)
8. [DTO Layer — What You Send & Receive](#8-dto-layer)
9. [Service Layer — The Business Logic](#9-service-layer)
10. [Controller Layer — The REST API Endpoints](#10-controller-layer)
11. [Exception Handling — Dealing with Errors](#11-exception-handling)
12. [Spring Security Configuration](#12-spring-security)
13. [JWT Utility Class — Token Generator](#13-jwt-utility)
14. [JWT Filter — Token Validator](#14-jwt-filter)
15. [Authentication Controller — Login & Register](#15-auth-controller)
16. [The Full Request Flow — End to End](#16-full-request-flow)
17. [Quick-Reference Templates](#17-quick-reference)
18. [Advanced Essentials — CORS, Pagination, ApiResponse, Logging & Pitfalls](#18-advanced-essentials)

---

## 1. Introduction

Think of building a web application like building a house. Normally you would have to order bricks, cement, wood, wires, pipes — everything separately and then figure out how they all fit together. **Spring Boot is like a pre-fabricated house kit.** Everything comes bundled together, pre-configured, and you just customize the rooms.

Spring Boot is a Java framework that lets you build web applications and REST APIs very quickly. It sits on top of the larger Spring Framework and removes most of the painful configuration work.

> **🔑 Key Idea — Spring Boot's Golden Rule**
> "Convention over Configuration." Spring Boot makes sensible default decisions for you. You only write code when you want to CHANGE the default behaviour. This means 80% of every project is almost identical — you are just changing the entity names and business logic.

### What is CRUD?

| Letter | Operation | HTTP Method |
|--------|-----------|-------------|
| C | **Create** — Add a new record | POST |
| R | **Read** — Fetch one or all records | GET |
| U | **Update** — Modify a record | PUT / PATCH |
| D | **Delete** — Remove a record | DELETE |

Every application you will ever build — a student portal, a shopping app, a hospital system — is fundamentally a CRUD application.

### What does a Spring Boot application do?

1. Starts an embedded Tomcat web server on a port (usually `8080`)
2. Waits for HTTP requests (from Postman, a browser, a mobile app)
3. Processes the request through layers: **Security → Controller → Service → Database**
4. Returns a JSON response

> **📌 Analogy — The Restaurant Model**
> - Customer (Frontend / Postman) → places an order (HTTP request)
> - Bouncer at the door (Spring Security) → checks your ID (JWT validation)
> - Waiter (Controller) → takes the order, passes to kitchen
> - Head Chef (Service) → decides what to cook and how
> - Pantry Staff (Repository) → fetches raw ingredients
> - Storage Room (MySQL Database) → where all data physically lives

---

## 2. Tech Stack

| Technology | What It Does |
|------------|-------------|
| **Spring Boot 4.x** | The main framework — wires everything together, auto-configuration, embedded Tomcat |
| **Spring MVC** | Handles HTTP routing — maps `/api/users` to a Java method automatically |
| **Spring Data JPA** | Database access — auto-generates SQL queries from method names |
| **Hibernate** | ORM engine — converts Java objects ↔ database rows (you never write SQL for basic CRUD) |
| **Spring Security 6** | Guards your API — authentication (who are you?) and authorization (what can you do?) |
| **JWT** | JSON Web Token — a signed ticket that proves identity without sessions or cookies |
| **MySQL 8** | Relational database — stores data in tables with rows and columns |
| **Maven** | Build tool — manages dependencies, compiles code, packages the app into a JAR |
| **Lombok** | Reduces boilerplate — auto-generates getters, setters, constructors, builders |
| **Jakarta Validation** | `@NotNull`, `@Email`, `@Size` etc. — enforce input rules declaratively on DTOs |

> **🔑 How They Work Together**
> 1. Request hits Spring Security → JWT is validated
> 2. If valid, passes to Spring MVC Controller
> 3. Controller calls Service (business logic)
> 4. Service calls Repository (Spring Data JPA)
> 5. Repository uses Hibernate to run SQL on MySQL
> 6. Data flows back up and is serialized to JSON

---

## 3. Project Setup

### Step 1: Generate at start.spring.io

Go to **https://start.spring.io** and configure:
- Project: **Maven** | Language: **Java** | Spring Boot: **3.3.x**
- Group: `tech.csm` | Artifact: `your-app-name`
- Packaging: **Jar** | Java: **17 or 21**

**Add these dependencies:**
- Spring Web
- Spring Data JPA
- Spring Security
- MySQL Driver
- Lombok
- Validation

### Step 2: The Standard Directory Structure

**Always follow this layout — Spring relies on it to scan and wire your classes.**

```
src/
  main/
    java/
      tech/csm/yourapp/
        YourAppApplication.java         // main entry point — NEVER modify

        entity/                         // Java classes mapped to DB tables
          User.java
          Product.java

        repository/                     // Interfaces for DB access (Spring Data JPA)
          UserRepository.java
          ProductRepository.java

        dto/                            // Data Transfer Objects (request & response)
          UserRequestDTO.java
          UserResponseDTO.java
          AuthRequestDTO.java
          AuthResponseDTO.java

        service/                        // Business logic layer
          UserService.java              // interface
          UserServiceImpl.java          // implementation

        controller/                     // REST endpoints (HTTP in / JSON out)
          UserController.java
          AuthController.java

        security/                       // ALL security code lives here
          SecurityConfig.java
          JwtUtil.java
          JwtAuthFilter.java
          UserDetailsServiceImpl.java

        exception/                      // Custom exceptions + global error handler
          ResourceNotFoundException.java
          GlobalExceptionHandler.java

        config/                         // Other config beans (CORS, etc.)
          CorsConfig.java

    resources/
      application.properties            // All configuration

  test/                                 // Unit & integration tests
```

> **📌 Rule — Why Layers? (Separation of Concerns)**
> - **Controller** — handles HTTP only (routing, status codes)
> - **Service** — handles business logic only (rules, decisions)
> - **Repository** — handles database only (queries, saves)
> - **Entity** — represents a DB table (data shape only, no logic)
>
> Benefit: Swapping MySQL for PostgreSQL only changes the config — no Service or Controller code changes.

---

## 4. pom.xml

The shopping list for your project. Maven reads this and downloads every library needed.

**FREQUENCY:** 1 per project — created once, only change `groupId` and `artifactId`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
           https://maven.apache.org/xsd/maven-4.0.0.xsd">

  <modelVersion>4.0.0</modelVersion>

  <!-- ✏ CHANGE these 2 values for every new project -->
  <groupId>tech.csm</groupId>
  <artifactId>student-service</artifactId>
  <version>1.0.0</version>

  <!-- Spring Boot parent — manages all dependency versions automatically -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.0</version>
  </parent>

  <properties>
    <java.version>17</java.version>
  </properties>

  <dependencies>

    <!-- Spring Web — enables @RestController, @GetMapping, JSON serialization -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Data JPA — repository pattern, Hibernate ORM, auto SQL -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Spring Security — authentication and authorization framework -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- Validation — @NotNull, @Email, @Size, @Pattern etc. on DTOs -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- MySQL Driver — JDBC connector to MySQL 8 database -->
    <dependency>
      <groupId>com.mysql</groupId>
      <artifactId>mysql-connector-j</artifactId>
      <scope>runtime</scope>
    </dependency>

    <!-- Lombok — compile-time code generator (getters, setters, builders) -->
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>

    <!-- JJWT — library for creating and validating JWT tokens -->
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-api</artifactId>
      <version>0.12.3</version>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-impl</artifactId>
      <version>0.12.3</version>
      <scope>runtime</scope>
    </dependency>
    <dependency>
      <groupId>io.jsonwebtoken</groupId>
      <artifactId>jjwt-jackson</artifactId>
      <version>0.12.3</version>
      <scope>runtime</scope>
    </dependency>

    <!-- Spring Boot Test — JUnit 5, Mockito, MockMvc -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <!-- Exclude Lombok from the final JAR — it is compile-time only -->
          <excludes>
            <exclude>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
            </exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

**Rules:**
- The `<parent>` block is what makes this "Spring Boot" — never remove it
- Never manually specify versions for Spring dependencies — the parent manages them
- JJWT is the only library requiring manual versioning
- Lombok must be excluded from the build plugin — compile-time only
- After adding a dependency: right-click → Maven → Reload Project in IntelliJ

---

## 5. application.properties

The settings panel of your application. Spring Boot reads this automatically at startup.

**FREQUENCY:** 1 per project — change DB name, JWT secret, and port per project.

```properties
# ============================================================
#  SERVER CONFIG
# ============================================================
server.port=8080

# ============================================================
#  DATABASE CONFIG  (✏ change DB name for each new project)
# ============================================================
spring.datasource.url=jdbc:mysql://localhost:3306/student_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ============================================================
#  JPA / HIBERNATE CONFIG
# ============================================================
# create      = drop + recreate tables on startup     (early development)
# update      = only adds new columns/tables, no drops (mid-development)
# validate    = checks schema matches entities          (staging)
# none        = does nothing                            (production)
spring.jpa.hibernate.ddl-auto=update

# Shows SQL Hibernate sends to MySQL — useful for debugging
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# ============================================================
#  JWT CONFIG  (✏ change the secret for each project)
# ============================================================
# Secret MUST be at least 32 characters long for HS256 algorithm
app.jwt.secret=my-super-secret-key-must-be-at-least-32-chars-long

# Token expiry in milliseconds  (86400000 = 24 hours)
app.jwt.expiration=86400000

# ============================================================
#  LOGGING
# ============================================================
logging.level.tech.csm=DEBUG
logging.level.org.springframework.security=DEBUG
```

**Rules:**
- Use `ddl-auto=update` during development, `none` in production (use Flyway/Liquibase for schema migrations)
- Never commit your real database password to GitHub
- JWT secret must be **at least 32 characters** — shorter throws `WeakKeyException` at startup
- Turn off `show-sql` in production — massive log output and performance impact
- Use `@Value("${app.jwt.secret}")` in Java classes to read values from this file

---

## 6. Entity Layer

An Entity is a Java class that directly represents a MySQL table. Every field = a column. Hibernate reads this and automatically creates or updates the table based on `ddl-auto`.

**FREQUENCY:** 1 per DB table — mandatory. Most projects have 3–10 entities.

> **🔑 What is ORM?**
> Without ORM: you write SQL → `INSERT INTO users (name, email) VALUES (?, ?)`
> With ORM (Hibernate): you call `repository.save(user)` and Hibernate writes the SQL for you.

```java
package tech.csm.studentservice.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDateTime;

// @Entity — tells Hibernate: 'this class is a database table'
@Entity

// @Table — specifies the exact MySQL table name
// If omitted, Hibernate uses the class name
@Table(name = "users")

// Lombok — auto-generates boilerplate at compile time
@Data            // → getters, setters, toString, equals, hashCode
@NoArgsConstructor  // → new User()
@AllArgsConstructor // → new User(id, name, email, password, role, ...)
@Builder         // → User.builder().name("John").email("j@j.com").build()
public class User {

    // @Id — PRIMARY KEY column
    // @GeneratedValue(IDENTITY) — uses MySQL AUTO_INCREMENT
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    // nullable=false → NOT NULL in MySQL
    // length=100     → VARCHAR(100)
    @Column(nullable = false, length = 100)
    private String name;

    // unique=true → UNIQUE constraint
    @Column(nullable = false, unique = true, length = 150)
    private String email;

    // NEVER store plain-text passwords — always store BCrypt hash
    @Column(nullable = false)
    private String password;

    // @Enumerated(STRING) → stores 'ADMIN' or 'USER' as text
    // Without this, Hibernate stores ordinal numbers (0, 1) — avoid that
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role;

    // updatable=false → written once on INSERT, never changed
    @Column(updatable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    // @PrePersist — Hibernate calls this just BEFORE a new row is INSERTED
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    // @PreUpdate — called just BEFORE an existing row is UPDATED
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // Inner enum — values map to allowed strings in the 'role' column
    public enum Role {
        USER, ADMIN, MODERATOR
    }
}
```

### Entity Annotation Reference

| Annotation | Purpose |
|------------|---------|
| `@Entity` | Marks class as a DB table. MANDATORY. |
| `@Table(name=...)` | Specify exact table name. Optional — defaults to class name. |
| `@Id` | Marks the primary key. Every entity MUST have exactly one `@Id`. |
| `@GeneratedValue(IDENTITY)` | Auto-increments the ID using MySQL's `AUTO_INCREMENT`. |
| `@Column` | Customise: `nullable`, `unique`, `length`, `name`, `updatable`, `insertable`. |
| `@Enumerated(EnumType.STRING)` | Stores enum as text (`USER`/`ADMIN`) not number (`0`/`1`). |
| `@OneToMany` | One parent has many children. e.g. one User → many Orders. |
| `@ManyToOne` | Many children belong to one parent. e.g. many Orders → one User. |
| `@ManyToMany` | Both sides can have multiple of each other. e.g. Student ↔ Course. |
| `@JoinColumn` | Specifies the foreign key column name on the owning side. |
| `@PrePersist` | Lifecycle callback — runs before INSERT. Used for `createdAt`. |
| `@PreUpdate` | Lifecycle callback — runs before UPDATE. Used for `updatedAt`. |

### Relationship Mapping Template

```java
// In Department.java  (the 'one' side — the parent)
@OneToMany(
    mappedBy = "department",        // field name in Employee that holds the FK
    cascade = CascadeType.ALL,      // deleting Dept also deletes all its Employees
    fetch = FetchType.LAZY          // don't load employees from DB until accessed
)
private List<Employee> employees = new ArrayList<>();

// In Employee.java  (the 'many' side — the child, holds the FK column)
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "department_id", nullable = false)
private Department department;

// Rule: @JoinColumn (FK column) always lives on the @ManyToOne side
// Rule: mappedBy value = the exact field name in the child class
// Rule: always use FetchType.LAZY — EAGER causes N+1 query problems
```

---

## 7. Repository Layer

An interface — you write NO implementation. Spring Data JPA reads method names and generates SQL at startup.

**FREQUENCY:** 1 per Entity — mandatory.

> **🔑 Spring Data JPA Magic**
> You declare: `Optional<User> findByEmail(String email);`
> Spring generates: `SELECT * FROM users WHERE email = ?`

```java
package tech.csm.studentservice.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Repository;
import tech.csm.studentservice.entity.User;
import java.util.Optional;
import java.util.List;

// JpaRepository<User, Long>:
//   User = the Entity this repository manages
//   Long = data type of the primary key
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // ── DERIVED QUERIES (method name = SQL) ──────────────────────────────────

    // SELECT * FROM users WHERE email = ?
    Optional<User> findByEmail(String email);

    // SELECT * FROM users WHERE role = ?
    List<User> findByRole(User.Role role);

    // SELECT * FROM users WHERE name LIKE %?%
    List<User> findByNameContainingIgnoreCase(String name);

    // SELECT CASE WHEN COUNT(*) > 0 THEN TRUE ELSE FALSE WHERE email = ?
    boolean existsByEmail(String email);

    // ── PAGINATION QUERY ─────────────────────────────────────────────────────
    Page<User> findByRole(User.Role role, Pageable pageable);

    // ── CUSTOM JPQL QUERY ────────────────────────────────────────────────────
    // Uses Entity CLASS names (User), not table names (users)
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.role = :role")
    Optional<User> findByEmailAndRole(String email, User.Role role);

    // ── NATIVE SQL QUERY ─────────────────────────────────────────────────────
    // nativeQuery=true lets you write raw MySQL SQL
    @Query(value = "SELECT * FROM users WHERE email LIKE %:keyword%",
           nativeQuery = true)
    List<User> searchByEmail(String keyword);
}
```

### Built-in JpaRepository Methods (Free — No Code Needed)

| Method | What It Does |
|--------|-------------|
| `save(entity)` | INSERT if new, UPDATE if existing |
| `findById(id)` | SELECT WHERE id=? → returns `Optional<T>` |
| `findAll()` | SELECT * FROM table |
| `findAll(Pageable)` | SELECT with pagination and sorting |
| `existsById(id)` | Returns true/false — does row exist? |
| `deleteById(id)` | DELETE WHERE id=? |
| `count()` | SELECT COUNT(*) |
| `saveAll(list)` | Batch INSERT/UPDATE |

### Method Naming Convention

| Method Name | Generated SQL |
|-------------|---------------|
| `findBy{Field}` | `WHERE field = ?` |
| `findBy{Field1}And{Field2}` | `WHERE field1 = ? AND field2 = ?` |
| `findByNameContaining` | `WHERE name LIKE %?%` |
| `findByNameStartingWith` | `WHERE name LIKE ?%` |
| `findByAgeGreaterThan` | `WHERE age > ?` |
| `findByAgeLessThanEqual` | `WHERE age <= ?` |
| `findByActiveTrue` | `WHERE active = true` |
| `findAllByOrderByNameAsc` | `ORDER BY name ASC` |
| `countBy{Field}` | `SELECT COUNT(*) WHERE field = ?` |
| `deleteBy{Field}` | `DELETE WHERE field = ?` |

---

## 8. DTO Layer

A DTO (Data Transfer Object) defines exactly what data enters and exits your API. **Never expose your Entity directly to the outside world.**

**FREQUENCY:** 1 Request DTO + 1 Response DTO per entity — always.

> **🔑 Why DTOs? Three Reasons**
> 1. **SECURITY** — control which fields are exposed (never expose passwords)
> 2. **VALIDATION** — put `@NotNull`, `@Email` etc. on DTOs, not on Entities
> 3. **DECOUPLING** — your API shape can evolve independently of your DB shape

### Request DTO

```java
package tech.csm.studentservice.dto;

import jakarta.validation.constraints.*;
import lombok.*;

// Defines what the client MUST send to create/update a User
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserRequestDTO {

    // @NotBlank = not null AND not empty AND not whitespace
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2–100 characters")
    private String name;

    // @Email validates format: must contain @ and domain
    @NotBlank(message = "Email is required")
    @Email(message = "Must be a valid email address")
    private String email;

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
}
```

### Response DTO

```java
package tech.csm.studentservice.dto;

import lombok.*;
import tech.csm.studentservice.entity.User;
import java.time.LocalDateTime;

// Defines EXACTLY what the API returns — notice: NO password field
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class UserResponseDTO {

    private Long id;
    private String name;
    private String email;
    private String role;
    private LocalDateTime createdAt;

    // Static factory method — the STANDARD pattern to convert Entity → DTO
    public static UserResponseDTO fromEntity(User user) {
        return UserResponseDTO.builder()
                .id(user.getId())
                .name(user.getName())
                .email(user.getEmail())
                .role(user.getRole().name())
                .createdAt(user.getCreatedAt())
                .build();
    }
}
```

### Auth DTOs

```java
// AuthRequestDTO.java — used for POST /api/v1/auth/login
@Data @NoArgsConstructor @AllArgsConstructor
public class AuthRequestDTO {
    @NotBlank private String email;
    @NotBlank private String password;
}

// AuthResponseDTO.java — returned after successful login
@Data @NoArgsConstructor @AllArgsConstructor @Builder
public class AuthResponseDTO {
    private String token;           // the JWT string
    private String type = "Bearer"; // always 'Bearer' for JWT
    private String email;
    private String role;
    private long expiresIn;         // milliseconds until expiry
}
```

### Validation Annotation Reference

| Annotation | Rule |
|------------|------|
| `@NotNull` | Cannot be null (but CAN be empty string) |
| `@NotBlank` | Cannot be null, empty, or whitespace — **preferred for all String fields** |
| `@NotEmpty` | Cannot be null or empty collection/string |
| `@Email` | Must be valid email format |
| `@Size(min, max)` | String length or Collection size must be within range |
| `@Min(value)` | Number must be >= value |
| `@Max(value)` | Number must be <= value |
| `@Pattern(regexp)` | String must match the given regex |
| `@Positive` | Number must be > 0 |
| `@Future` | Date/time must be in the future |
| `@Past` | Date/time must be in the past |

---

## 9. Service Layer

All business rules live here. The Controller should have zero logic — only routing.

**FREQUENCY:** 1 interface + 1 implementation per feature — always. Most coding time is spent here.

> **📌 Rule — Always Use Interface + Impl Pattern**
> 1. `UserService.java` — interface (method signatures only, no code)
> 2. `UserServiceImpl.java` — implementation (actual logic)
>
> The Controller depends on the interface, not the implementation. This allows mocking in unit tests.

### Service Interface

```java
package tech.csm.studentservice.service;

import tech.csm.studentservice.dto.*;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import java.util.List;

// Interface: defines WHAT this service can do — no method bodies
public interface UserService {

    UserResponseDTO createUser(UserRequestDTO requestDTO);

    UserResponseDTO getUserById(Long id);

    List<UserResponseDTO> getAllUsers();

    Page<UserResponseDTO> getUsersPaged(Pageable pageable);

    UserResponseDTO updateUser(Long id, UserRequestDTO requestDTO);

    void deleteUser(Long id);

    boolean emailExists(String email);
}
```

### Service Implementation

```java
package tech.csm.studentservice.service;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import tech.csm.studentservice.dto.*;
import tech.csm.studentservice.entity.User;
import tech.csm.studentservice.exception.ResourceNotFoundException;
import tech.csm.studentservice.repository.UserRepository;
import java.util.List;
import java.util.stream.Collectors;

// @Service — Spring creates one instance (Singleton) and manages it
// @Slf4j (Lombok) — injects a 'log' field for logging
// @RequiredArgsConstructor (Lombok) — generates constructor for all 'final' fields
@Service
@Slf4j
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

    // Constructor injection — preferred over @Autowired field injection
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    // @Transactional — wraps in a DB transaction
    // If an exception occurs mid-method, ALL DB changes are rolled back
    // Always use on methods that WRITE to the database
    @Override
    @Transactional
    public UserResponseDTO createUser(UserRequestDTO requestDTO) {
        log.debug("Creating user with email: {}", requestDTO.getEmail());

        // Business Rule: Email must be unique
        if (userRepository.existsByEmail(requestDTO.getEmail())) {
            throw new IllegalArgumentException(
                    "Email already registered: " + requestDTO.getEmail());
        }

        // Build entity from DTO — never pass DTOs directly to repository
        User user = User.builder()
                .name(requestDTO.getName())
                .email(requestDTO.getEmail())
                // Always hash passwords with BCrypt — NEVER store plain text
                .password(passwordEncoder.encode(requestDTO.getPassword()))
                .role(User.Role.USER)  // default role
                .build();

        User saved = userRepository.save(user);
        log.info("User created with id: {}", saved.getId());
        return UserResponseDTO.fromEntity(saved);
    }

    // readOnly=true — Hibernate skips dirty checking → better read performance
    @Override
    @Transactional(readOnly = true)
    public UserResponseDTO getUserById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() ->
                        new ResourceNotFoundException("User", "id", id));
        return UserResponseDTO.fromEntity(user);
    }

    @Override
    @Transactional(readOnly = true)
    public List<UserResponseDTO> getAllUsers() {
        // Stream: fetch list → convert each entity to DTO → collect as list
        return userRepository.findAll()
                .stream()
                .map(UserResponseDTO::fromEntity)
                .collect(Collectors.toList());
    }

    @Override
    @Transactional(readOnly = true)
    public Page<UserResponseDTO> getUsersPaged(Pageable pageable) {
        // Page.map() converts each entity in the page to a DTO efficiently
        return userRepository.findAll(pageable).map(UserResponseDTO::fromEntity);
    }

    @Override
    @Transactional
    public UserResponseDTO updateUser(Long id, UserRequestDTO requestDTO) {
        User user = userRepository.findById(id)
                .orElseThrow(() ->
                        new ResourceNotFoundException("User", "id", id));

        user.setName(requestDTO.getName());
        user.setEmail(requestDTO.getEmail());

        // Only update password if a new one is provided
        if (requestDTO.getPassword() != null && !requestDTO.getPassword().isBlank()) {
            user.setPassword(passwordEncoder.encode(requestDTO.getPassword()));
        }

        // save() on an entity with an existing id performs UPDATE, not INSERT
        return UserResponseDTO.fromEntity(userRepository.save(user));
    }

    @Override
    @Transactional
    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User", "id", id);
        }
        userRepository.deleteById(id);
        log.info("User deleted with id: {}", id);
    }

    @Override
    public boolean emailExists(String email) {
        return userRepository.existsByEmail(email);
    }
}
```

---

## 10. Controller Layer

The front door of your application. Receives HTTP requests, calls Service, returns JSON. **Zero business logic here.**

**FREQUENCY:** 1 per feature/entity + 1 AuthController — always.

> **🔑 ResponseEntity**
> Wraps your data AND the HTTP status code together.
> - `ResponseEntity.ok(data)` → 200 OK
> - `ResponseEntity.status(201).body(d)` → 201 Created
> - `ResponseEntity.noContent().build()` → 204 No Content

```java
package tech.csm.studentservice.controller;

import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;
import tech.csm.studentservice.dto.*;
import tech.csm.studentservice.service.UserService;
import java.util.List;

// @RestController = @Controller + @ResponseBody
// @ResponseBody = auto-serialize all return values to JSON
@RestController

// Base URL for ALL endpoints in this class
@RequestMapping("/api/v1/users")

@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    // ── CREATE  POST /api/v1/users ────────────────────────────────────────────
    // @RequestBody — reads JSON body and converts to UserRequestDTO
    // @Valid       — triggers validation annotations on the DTO
    @PostMapping
    public ResponseEntity<UserResponseDTO> createUser(
            @Valid @RequestBody UserRequestDTO dto) {
        return ResponseEntity
                .status(HttpStatus.CREATED)     // 201
                .body(userService.createUser(dto));
    }

    // ── READ ONE  GET /api/v1/users/5 ────────────────────────────────────────
    // @PathVariable — extracts {id} from the URL path
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDTO> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    // ── READ ALL  GET /api/v1/users ───────────────────────────────────────────
    @GetMapping
    public ResponseEntity<List<UserResponseDTO>> getAllUsers() {
        return ResponseEntity.ok(userService.getAllUsers());
    }

    // ── READ PAGED  GET /api/v1/users/paged?page=0&size=10&sortBy=name ───────
    @GetMapping("/paged")
    public ResponseEntity<Page<UserResponseDTO>> getUsersPaged(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy) {
        PageRequest pageRequest = PageRequest.of(page, size, Sort.by(sortBy));
        return ResponseEntity.ok(userService.getUsersPaged(pageRequest));
    }

    // ── UPDATE  PUT /api/v1/users/5 ───────────────────────────────────────────
    @PutMapping("/{id}")
    public ResponseEntity<UserResponseDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserRequestDTO dto) {
        return ResponseEntity.ok(userService.updateUser(id, dto));
    }

    // ── DELETE  DELETE /api/v1/users/5 ───────────────────────────────────────
    // @PreAuthorize — method only executes if condition is true
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build(); // 204
    }
}
```

### HTTP Method & Annotation Reference

| Annotation | HTTP Method | Purpose |
|------------|-------------|---------|
| `@GetMapping` | GET | Fetch data. No request body. |
| `@PostMapping` | POST | Create. Data in body. Returns 201. |
| `@PutMapping` | PUT | Replace entire resource. Full object in body. |
| `@PatchMapping` | PATCH | Partial update. Send only changed fields. |
| `@DeleteMapping` | DELETE | Remove resource. Returns 204. |
| `@PathVariable` | — | Extract value from URL path: `/users/{id}` |
| `@RequestParam` | — | Extract from query string: `/users?name=John` |
| `@RequestBody` | — | Parse JSON request body into Java object |
| `@Valid` | — | Trigger Jakarta Validation on the DTO |
| `@PreAuthorize` | — | Role-based access at method level |

### HTTP Status Codes

| Code | Meaning | When to Use |
|------|---------|-------------|
| 200 OK | Success, data returned | GET, PUT responses |
| 201 Created | Resource created | POST create responses |
| 204 No Content | Success, nothing to return | DELETE responses |
| 400 Bad Request | Invalid data from client | Validation failure |
| 401 Unauthorized | Not authenticated | Missing/invalid JWT |
| 403 Forbidden | Authenticated but no permission | Wrong role |
| 404 Not Found | Resource doesn't exist | — |
| 409 Conflict | Duplicate data | Duplicate email |
| 500 Server Error | Unexpected server error | — |

---

## 11. Exception Handling

Without this, Spring returns ugly HTML error pages. Always return clean, structured JSON.

### Custom Exception

```java
package tech.csm.studentservice.exception;

// RuntimeException — unchecked, no try-catch required everywhere
public class ResourceNotFoundException extends RuntimeException {

    // new ResourceNotFoundException("User", "id", 5)
    // produces: "User not found with id : '5'"
    public ResourceNotFoundException(String resourceName,
                                     String fieldName,
                                     Object fieldValue) {
        super(String.format("%s not found with %s : '%s'",
                resourceName, fieldName, fieldValue));
    }
}
```

### Global Exception Handler

```java
package tech.csm.studentservice.exception;

import org.springframework.http.*;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.*;
import java.time.LocalDateTime;
import java.util.*;

// @RestControllerAdvice — intercepts ALL exceptions from ALL controllers
@RestControllerAdvice
public class GlobalExceptionHandler {

    // Standard shape for all error responses
    private record ErrorResponse(
        LocalDateTime timestamp,
        int status,
        String error,
        String message,
        String path) {}

    // ── 404 Resource Not Found ────────────────────────────────────────────────
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex,
            jakarta.servlet.http.HttpServletRequest request) {

        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ErrorResponse(LocalDateTime.now(), 404,
                        "Not Found", ex.getMessage(), request.getRequestURI()));
    }

    // ── 400 Validation Failed (triggered by @Valid on DTOs) ──────────────────
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidation(
            MethodArgumentNotValidException ex) {

        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getAllErrors().forEach(error -> {
            String field = ((FieldError) error).getField();
            fieldErrors.put(field, error.getDefaultMessage());
        });

        Map<String, Object> body = new LinkedHashMap<>();
        body.put("timestamp", LocalDateTime.now());
        body.put("status", 400);
        body.put("error", "Validation Failed");
        body.put("fieldErrors", fieldErrors);
        return ResponseEntity.badRequest().body(body);
    }

    // ── 409 Conflict (duplicate email, etc.) ─────────────────────────────────
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleConflict(
            IllegalArgumentException ex,
            jakarta.servlet.http.HttpServletRequest request) {

        return ResponseEntity.status(HttpStatus.CONFLICT)
                .body(new ErrorResponse(LocalDateTime.now(), 409,
                        "Conflict", ex.getMessage(), request.getRequestURI()));
    }

    // ── 401 Bad Credentials (wrong password at login) ────────────────────────
    @ExceptionHandler(BadCredentialsException.class)
    public ResponseEntity<ErrorResponse> handleBadCredentials(
            BadCredentialsException ex,
            jakarta.servlet.http.HttpServletRequest request) {

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                .body(new ErrorResponse(LocalDateTime.now(), 401,
                        "Unauthorized", "Invalid email or password",
                        request.getRequestURI()));
    }

    // ── 500 Catch-All ─────────────────────────────────────────────────────────
    // NEVER expose stack trace or internal details to the client
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAll(
            Exception ex,
            jakarta.servlet.http.HttpServletRequest request) {

        return ResponseEntity.internalServerError()
                .body(new ErrorResponse(LocalDateTime.now(), 500,
                        "Internal Server Error",
                        "An unexpected error occurred",
                        request.getRequestURI()));
    }
}
```

> **📌 Exception Handling Rules**
> 1. ALWAYS use `GlobalExceptionHandler` — never let stack traces reach the client
> 2. Use `ResourceNotFoundException` for all "record not found" cases
> 3. Use `IllegalArgumentException` for duplicate/conflict cases (409)
> 4. Always handle `BadCredentialsException` → 401 (not 500)
> 5. The catch-all must say "unexpected error" — never expose internals
> 6. 400/409 = client did something wrong | 500 = server did something wrong

---

## 12. Spring Security

Guards every endpoint. Adding Spring Security to `pom.xml` locks down the entire application by default. `SecurityConfig` defines what is public and what requires a JWT.

> **🔑 Spring Security 6 — Important Change**
> Old (Spring Boot 2.x): extend `WebSecurityConfigurerAdapter`.
> New (Spring Boot 3.x): use `@Bean` methods returning `SecurityFilterChain`.
> `WebSecurityConfigurerAdapter` was **removed** in Security 6.

### UserDetailsService Implementation

```java
package tech.csm.studentservice.security;

import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.*;
import org.springframework.stereotype.Service;
import tech.csm.studentservice.repository.UserRepository;
import java.util.List;

// Bridge between Spring Security and your UserRepository
@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final UserRepository userRepository;

    // Spring Security calls this when authenticating a user
    @Override
    public UserDetails loadUserByUsername(String email)
            throws UsernameNotFoundException {

        var user = userRepository.findByEmail(email)
                .orElseThrow(() -> new UsernameNotFoundException(
                        "User not found: " + email));

        // Authorities MUST be prefixed with 'ROLE_' for hasRole() to work
        // e.g. Role.ADMIN → authority string 'ROLE_ADMIN'
        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getEmail())
                .password(user.getPassword())   // already BCrypt hashed
                .authorities(List.of(new SimpleGrantedAuthority(
                        "ROLE_" + user.getRole().name())))
                .build();
    }
}
```

### Security Configuration

```java
package tech.csm.studentservice.security;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.*;
import org.springframework.security.authentication.*;
import org.springframework.security.config.annotation.authentication.configuration.*;
import org.springframework.security.config.annotation.method.configuration.*;
import org.springframework.security.config.annotation.web.builders.*;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

// @EnableMethodSecurity — activates @PreAuthorize on controller/service methods
@Configuration
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthFilter jwtAuthFilter;
    private final UserDetailsServiceImpl userDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            // Disable CSRF — not needed for stateless JWT REST APIs
            .csrf(csrf -> csrf.disable())

            // URL-based access rules — first match wins
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/v1/auth/**").permitAll()   // login, register
                .requestMatchers("/api/v1/public/**").permitAll() // public endpoints
                .requestMatchers("/error").permitAll()            // Spring error page
                .anyRequest().authenticated()                     // everything else = JWT required
            )

            // STATELESS — no sessions or cookies. JWT handles state.
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )

            .userDetailsService(userDetailsService)

            // Insert JWT filter BEFORE Spring's default auth filter
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    // BCrypt — one-way hash with auto-salt. Always use this for passwords.
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    // Required in AuthController to trigger the login/authenticate flow
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
}
```

### Security Rules Reference

| Rule | Meaning |
|------|---------|
| `.permitAll()` | Anyone can access — no token required |
| `.authenticated()` | Must have a valid JWT token — any role |
| `.hasRole('ADMIN')` | Must have `ROLE_ADMIN` authority |
| `.hasAnyRole('A','B')` | Must have either role |
| `@PreAuthorize` | Method-level check — more granular than URL rules |
| `STATELESS` | No sessions. Every request carries its own JWT. |
| `csrf.disable()` | Safe for REST APIs — CSRF only needed for session-cookie apps |
| `BCryptPasswordEncoder` | Industry standard. Default strength 10. Never use MD5 or SHA. |

---

## 13. JWT Utility

> **🔑 Anatomy of a JWT**
> Format: `header.payload.signature` (three Base64-encoded parts joined by dots)
> - **Header** — algorithm: HS256, type: JWT
> - **Payload** — claims: `sub` (email), roles, `iat` (issued at), `exp` (expiry)
> - **Signature** — `HMAC-SHA256(header + '.' + payload, secretKey)`
>
> The payload is Base64-encoded, **NOT encrypted** — anyone can decode it.
> The signature can ONLY be produced by the server that knows the secret key.

```java
package tech.csm.studentservice.security;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;
import javax.crypto.SecretKey;
import java.nio.charset.StandardCharsets;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {

    // Reads values from application.properties at startup
    @Value("${app.jwt.secret}")
    private String secretKey;

    @Value("${app.jwt.expiration}")
    private long jwtExpiration;

    // ── TOKEN GENERATION ──────────────────────────────────────────────────────
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        // Embed roles in token — avoids DB lookup per request
        claims.put("roles", userDetails.getAuthorities().toString());
        return buildToken(claims, userDetails.getUsername(), jwtExpiration);
    }

    private String buildToken(Map<String, Object> claims, String subject, long expiration) {
        return Jwts.builder()
                .claims(claims)                // custom payload
                .subject(subject)               // 'sub' = user's email
                .issuedAt(new Date())           // 'iat' = creation time
                .expiration(new Date(
                        System.currentTimeMillis() + expiration))  // 'exp'
                .signWith(getSigningKey())      // sign with HMAC-SHA256
                .compact();                    // build the final string
    }

    // ── TOKEN VALIDATION ──────────────────────────────────────────────────────
    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername())
               && !isTokenExpired(token);
    }

    // ── CLAIM EXTRACTION ──────────────────────────────────────────────────────
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    // Generic: extract any claim using a function reference
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        return claimsResolver.apply(extractAllClaims(token));
    }

    // Parse and verify token signature — throws JwtException if invalid
    private Claims extractAllClaims(String token) {
        return Jwts.parser()
                .verifyWith(getSigningKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    // Convert the plain string secret to a cryptographic key
    // Keys.hmacShaKeyFor validates the key length is sufficient for HS256
    private SecretKey getSigningKey() {
        return Keys.hmacShaKeyFor(
                secretKey.getBytes(StandardCharsets.UTF_8));
    }
}
```

> **⚠️ Critical JWT Rules**
> 1. Secret must be at least 32 characters (256 bits) — shorter throws `WeakKeyException`
> 2. Use `getBytes(UTF_8)` — not Base64 decode — with a plain string secret
> 3. Never put secrets in code or Git — always read from `application.properties`
> 4. The JWT payload is NOT encrypted — never put passwords or sensitive data in claims

---

## 14. JWT Filter

Runs on **every** incoming request. Validates the JWT before any controller is called.

> **🔑 How the Filter Works**
> 1. Request arrives with `Authorization: Bearer eyJhbGci...`
> 2. Filter reads header, extracts token (removes `Bearer ` prefix, 7 chars)
> 3. Extracts email from token via `jwtUtil.extractUsername()`
> 4. Loads user from DB via `UserDetailsService`
> 5. Validates: signature valid? not expired? matches this user?
> 6. If valid → sets authentication in `SecurityContextHolder`
> 7. Request continues to Controller
> 8. If invalid → authentication NOT set → Spring Security returns 401

```java
package tech.csm.studentservice.security;

import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.lang.NonNull;
import org.springframework.security.authentication.*;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;
import java.io.IOException;

// OncePerRequestFilter — exactly ONE execution per HTTP request
@Component
@RequiredArgsConstructor
public class JwtAuthFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;
    private final UserDetailsServiceImpl userDetailsService;

    @Override
    protected void doFilterInternal(
            @NonNull HttpServletRequest request,
            @NonNull HttpServletResponse response,
            @NonNull FilterChain filterChain) throws ServletException, IOException {

        // ── Step 1: Read and validate Authorization header ────────────────────
        final String authHeader = request.getHeader("Authorization");

        // No header or doesn't start with 'Bearer ' → skip JWT processing
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        // Remove 'Bearer ' prefix (7 characters)
        final String jwt = authHeader.substring(7);

        // ── Step 2: Extract username from token ───────────────────────────────
        final String userEmail;
        try {
            userEmail = jwtUtil.extractUsername(jwt);
        } catch (JwtException e) {
            // Token is malformed or signature invalid — pass through
            // Spring Security returns 401 for protected endpoints
            filterChain.doFilter(request, response);
            return;
        }

        // ── Step 3: Validate and set authentication ───────────────────────────
        if (userEmail != null &&
            SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails =
                    userDetailsService.loadUserByUsername(userEmail);

            if (jwtUtil.isTokenValid(jwt, userDetails)) {

                var authToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,                          // credentials null post-auth
                        userDetails.getAuthorities()   // roles for @PreAuthorize
                );
                authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request));

                // Tell Spring Security: 'this user is authenticated for this request'
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        // ── Step 4: Continue to next filter / controller ──────────────────────
        filterChain.doFilter(request, response);
    }
}
```

---

## 15. Auth Controller

Handles the two public endpoints every application needs.

```java
package tech.csm.studentservice.controller;

import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.security.authentication.*;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.web.bind.annotation.*;
import tech.csm.studentservice.dto.*;
import tech.csm.studentservice.security.*;
import tech.csm.studentservice.service.UserService;

@RestController
@RequestMapping("/api/v1/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationManager authenticationManager;
    private final UserDetailsServiceImpl userDetailsService;
    private final JwtUtil jwtUtil;
    private final UserService userService;

    // ── REGISTER  POST /api/v1/auth/register ──────────────────────────────────
    // PUBLIC endpoint — marked permitAll() in SecurityConfig
    @PostMapping("/register")
    public ResponseEntity<UserResponseDTO> register(
            @Valid @RequestBody UserRequestDTO request) {
        return ResponseEntity.ok(userService.createUser(request));
    }

    // ── LOGIN  POST /api/v1/auth/login ────────────────────────────────────────
    @PostMapping("/login")
    public ResponseEntity<AuthResponseDTO> login(
            @Valid @RequestBody AuthRequestDTO request) {

        // authenticate() triggers the full Spring Security auth flow:
        //   1. Calls UserDetailsService.loadUserByUsername(email)
        //   2. Compares password with stored BCrypt hash
        //   3. Throws BadCredentialsException if wrong → 401
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getEmail(), request.getPassword())
        );

        // Execution only reaches here if credentials were valid
        UserDetails userDetails =
                userDetailsService.loadUserByUsername(request.getEmail());

        String token = jwtUtil.generateToken(userDetails);

        return ResponseEntity.ok(AuthResponseDTO.builder()
                .token(token)
                .type("Bearer")
                .email(request.getEmail())
                .role(userDetails.getAuthorities().iterator().next().getAuthority())
                .expiresIn(86400000L)
                .build());
    }
}
```

### Login Flow — Step by Step

1. Client sends `POST /api/v1/auth/login` with `{ email, password }`
2. `JwtAuthFilter` runs — no token in header, passes through
3. `AuthController.login()` is called
4. `authenticationManager.authenticate()` is called
5. Spring calls `UserDetailsService.loadUserByUsername(email)` to fetch user from DB
6. BCrypt compares submitted password against stored hash
7. Match → success. No match → `BadCredentialsException` → 401
8. `jwtUtil.generateToken()` creates signed JWT with email and roles
9. JWT returned in `AuthResponseDTO`
10. Client sends JWT in every future request: `Authorization: Bearer <token>`

---

## 16. Full Request Flow

**Trace of: `GET /api/v1/users/5` with a valid JWT token**

```
REQUEST:  GET /api/v1/users/5
HEADER:   Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

Step 1 ─ JwtAuthFilter (runs first, before any controller)
         Reads 'Authorization: Bearer <token>' header
         Calls jwtUtil.extractUsername(token) → 'user@example.com'
         Loads user from DB via UserDetailsService
         Calls jwtUtil.isTokenValid(token, userDetails)
         Valid → sets SecurityContextHolder authentication
         ↓
Step 2 ─ Spring Security Authorization Check
         Rule: .anyRequest().authenticated() in SecurityConfig
         User IS authenticated (set in Step 1) → ALLOW
         ↓
Step 3 ─ UserController.getUserById(5) called
         @GetMapping("/{id}") matches URL
         @PathVariable extracts id = 5
         Calls userService.getUserById(5)
         ↓
Step 4 ─ UserServiceImpl.getUserById(5)
         Calls userRepository.findById(5)
         Not found → throws ResourceNotFoundException → 404
         Found → converts entity to UserResponseDTO
         ↓
Step 5 ─ UserRepository.findById(5)
         Hibernate generates: SELECT * FROM users WHERE id = 5
         Executes against MySQL → returns Optional<User>
         ↓
RESPONSE: 200 OK
{
  "id": 5,
  "name": "John Doe",
  "email": "john@example.com",
  "role": "USER",
  "createdAt": "2024-01-15T10:30:00"
}
```

### Expired Token Scenario

```
Step 1 ─ JwtAuthFilter calls jwtUtil.isTokenValid(token, userDetails)
Step 2 ─ isTokenValid() calls isTokenExpired(token)
Step 3 ─ extractExpiration(token).before(new Date()) returns TRUE (expired)
Step 4 ─ isTokenValid() returns FALSE
Step 5 ─ Authentication NOT set in SecurityContextHolder
Step 6 ─ Spring Security sees unauthenticated request to protected endpoint
Step 7 ─ Returns 401 Unauthorized automatically

Client must call POST /api/v1/auth/login again to get a new token.
```

---

## 17. Quick Reference

### Main Application Entry Point

```java
package tech.csm.studentservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

// @SpringBootApplication = @Configuration + @EnableAutoConfiguration + @ComponentScan
// Bootstraps the entire Spring context and starts embedded Tomcat
@SpringBootApplication
public class StudentServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(StudentServiceApplication.class, args);
    }
}
```

### Running the Application

```bash
# Method 1: IntelliJ IDE
# Right-click StudentServiceApplication.java → Run 'main()'

# Method 2: Maven
mvn spring-boot:run

# Method 3: Build and run JAR
mvn clean package
java -jar target/student-service-1.0.0.jar

# App starts on: http://localhost:8080
```

### New Project Checklist

1. Generate at `start.spring.io` with required dependencies
2. Edit `pom.xml` — change `groupId` and `artifactId`
3. Edit `application.properties` — change DB name and JWT secret
4. Create MySQL database: `CREATE DATABASE your_db;`
5. Create Entity class for each table
6. Create Repository interface per entity
7. Create Request + Response DTOs per entity
8. Create Service interface + Impl per entity
9. Create Controller per entity + `AuthController`
10. Copy security package: `SecurityConfig`, `JwtUtil`, `JwtAuthFilter`, `UserDetailsServiceImpl`
11. Copy exception package: `ResourceNotFoundException`, `GlobalExceptionHandler`
12. Add `CorsConfig` (Chapter 18)
13. Test with Postman: Register → Login → Use token on protected endpoints

### Postman Testing Guide

| Action | Request |
|--------|---------|
| Register | `POST /api/v1/auth/register` — Body: `{ name, email, password }` |
| Login | `POST /api/v1/auth/login` — Body: `{ email, password }` → copy token from response |
| Get all | `GET /api/v1/users` — Header: `Authorization: Bearer <token>` |
| Get one | `GET /api/v1/users/5` — Header: `Authorization: Bearer <token>` |
| Create | `POST /api/v1/users` — Header + Body: full JSON object |
| Update | `PUT /api/v1/users/5` — Header + Body: full JSON object |
| Delete | `DELETE /api/v1/users/5` — Header: Bearer `<admin-token>` |
| Paged | `GET /api/v1/users/paged?page=0&size=5&sortBy=name` — Header required |

### Component Frequency Reference

| Component | Frequency |
|-----------|-----------|
| Application Entry Point | 1 per project — never modified |
| `pom.xml` | 1 per project — change 3-4 values only |
| `application.properties` | 1 per project — change DB name and secret |
| Entity class | 1 per DB table — always created fresh |
| Repository interface | 1 per Entity — always created fresh |
| Service interface + impl | 1 pair per entity — always created fresh |
| Controller | 1 per feature/entity — always created fresh |
| DTOs (Request + Response) | 1 pair per entity — always created fresh |
| `SecurityConfig` | 1 per project — copy template, URL rules change |
| `JwtUtil` | 1 per project — **COPY AS-IS, zero changes** |
| `JwtAuthFilter` | 1 per project — **COPY AS-IS, zero changes** |
| `UserDetailsServiceImpl` | 1 per project — copy, minor field changes |
| `AuthController` | 1 per project — copy, minor changes |
| `GlobalExceptionHandler` | 1 per project — copy, add handlers as needed |
| `ResourceNotFoundException` | 1 per project — **COPY AS-IS** |
| `CorsConfig` | 1 per project — copy, change allowed origins |

---

## 18. Advanced Essentials

### 18.1 CORS Configuration

CORS (Cross-Origin Resource Sharing) is a browser security mechanism. When your React/Angular frontend (e.g. `localhost:3000`) calls your Spring Boot API (`localhost:8080`), the browser **blocks the request** because the ports differ. Without CORS config, your frontend can never reach your API.

> **🔑 What is CORS?**
> Same Origin = same scheme + domain + port.
> A browser blocks cross-origin requests unless the **server** explicitly says "I allow requests from this origin."
> CORS adds response headers like: `Access-Control-Allow-Origin: http://localhost:3000`
> **Postman ignores CORS entirely** — it is a browser-only restriction.

```java
package tech.csm.studentservice.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.*;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();

        // ✏ Change these to match your frontend URL in production
        config.addAllowedOrigin("http://localhost:3000");   // React default
        config.addAllowedOrigin("http://localhost:4200");   // Angular default
        config.addAllowedOrigin("http://localhost:5173");   // Vite/Vue default

        // Allow the Authorization header (needed for JWT Bearer tokens)
        config.addAllowedHeader("*");

        // Allow all HTTP methods: GET, POST, PUT, DELETE, PATCH, OPTIONS
        config.addAllowedMethod("*");

        // Allow credentials (cookies, Authorization headers)
        config.setAllowCredentials(true);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);

        return new CorsFilter(source);
    }
}
```

Also add this to the `authorizeHttpRequests` block in `SecurityConfig` to allow browser preflight requests:

```java
// Add this line inside .authorizeHttpRequests(auth -> auth ... )
.requestMatchers(HttpMethod.OPTIONS, "/**").permitAll()  // CORS preflight
```

---

### 18.2 Pagination

Returns data in pages instead of all at once. Returning 50,000 users in one response is a performance disaster.

```java
// ── 1. Repository — Spring Data provides this for free ───────────────────────
// Page<User> findAll(Pageable pageable);  ← already inherited from JpaRepository

// Custom paged query:
Page<User> findByRole(User.Role role, Pageable pageable);


// ── 2. Service ────────────────────────────────────────────────────────────────
public Page<UserResponseDTO> getUsersPaged(Pageable pageable) {
    // Page.map() converts every entity in the page to a DTO efficiently
    return userRepository.findAll(pageable)
            .map(UserResponseDTO::fromEntity);
}


// ── 3. Controller ─────────────────────────────────────────────────────────────
@GetMapping("/paged")
public ResponseEntity<Page<UserResponseDTO>> getUsersPaged(
        @RequestParam(defaultValue = "0")    int page,
        @RequestParam(defaultValue = "10")   int size,
        @RequestParam(defaultValue = "id")   String sortBy,
        @RequestParam(defaultValue = "asc")  String direction) {

    Sort sort = direction.equalsIgnoreCase("desc")
            ? Sort.by(sortBy).descending()
            : Sort.by(sortBy).ascending();

    PageRequest pageRequest = PageRequest.of(page, size, sort);
    return ResponseEntity.ok(userService.getUsersPaged(pageRequest));
}


// ── 4. Response shape — Spring's Page<T> automatically includes ───────────────
// {
//   "content": [ { ...user1 }, { ...user2 } ],   actual data for this page
//   "totalElements": 250,                          total rows in DB
//   "totalPages": 25,                              how many pages exist
//   "number": 0,                                   current page index (0-based)
//   "size": 10,                                    items per page
//   "first": true,                                 is this the first page?
//   "last": false                                  is this the last page?
// }

// Sample URL:
// GET /api/v1/users/paged?page=2&size=5&sortBy=name&direction=asc
```

---

### 18.3 Standardised API Response Wrapper

A standard envelope so the client always sees the same shape regardless of success or failure.

```java
// ApiResponse.java
package tech.csm.studentservice.dto;

import lombok.*;
import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class ApiResponse<T> {

    private boolean success;
    private String message;
    private T data;               // the actual payload — generic type
    private LocalDateTime timestamp;

    // Use for all successful responses
    public static <T> ApiResponse<T> success(String message, T data) {
        return ApiResponse.<T>builder()
                .success(true)
                .message(message)
                .data(data)
                .timestamp(LocalDateTime.now())
                .build();
    }

    // Use for error responses (in GlobalExceptionHandler)
    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
                .success(false)
                .message(message)
                .data(null)
                .timestamp(LocalDateTime.now())
                .build();
    }
}


// Example controller usage:
@PostMapping
public ResponseEntity<ApiResponse<UserResponseDTO>> createUser(
        @Valid @RequestBody UserRequestDTO dto) {
    UserResponseDTO created = userService.createUser(dto);
    return ResponseEntity
            .status(HttpStatus.CREATED)
            .body(ApiResponse.success("User created successfully", created));
}

// Response will look like:
// {
//   "success": true,
//   "message": "User created successfully",
//   "data": { "id": 1, "name": "John", "email": "j@j.com", "role": "USER" },
//   "timestamp": "2024-01-15T10:30:00"
// }
```

---

### 18.4 Logging with @Slf4j

`System.out.println` is not appropriate for production. Use `@Slf4j` (Lombok) for proper logging.

```java
@Slf4j   // Lombok injects a 'log' field into the class
@Service
public class UserServiceImpl implements UserService {

    public UserResponseDTO createUser(UserRequestDTO dto) {

        // Log levels — choose based on importance:
        log.trace("Entering createUser method");            // finest detail
        log.debug("Creating user: {}", dto.getEmail());    // dev debugging
        log.info("User created with id: {}", saved.getId()); // normal events
        log.warn("Approaching rate limit for: {}", dto.getEmail()); // warnings
        log.error("Failed to create user", exception);     // actual errors

        // {} is a placeholder — filled lazily (no string concat if level is OFF)
        // ALWAYS use {} placeholders, never String concatenation in log calls
        log.debug("Processing request for: {}", dto.getEmail());  // CORRECT
        log.debug("Processing: " + dto.getEmail());               // WRONG
    }
}
```

```properties
# Control log levels in application.properties:
logging.level.tech.csm=DEBUG          # see all DEBUG+ from your packages
logging.level.root=WARN               # only WARN+ from all libraries
logging.level.org.hibernate.SQL=DEBUG # see every SQL statement
```

| Level | When to Use |
|-------|------------|
| TRACE | Method entry/exit — finest grain. Never use in production. |
| DEBUG | Development debugging. Turn off in production. |
| INFO  | Normal application events (user registered, order placed). Keep in production. |
| WARN  | Unexpected but recoverable. Investigate later. |
| ERROR | Something failed that needs attention. Always log the exception object. |

---

### 18.5 Common Errors and How to Fix Them

| Error | Cause & Fix |
|-------|------------|
| **401 on every request** | JWT filter not extracting token correctly. Check `Authorization: Bearer <token>` header has a space. Check JWT secret is >= 32 chars. |
| **403 on ADMIN endpoint** | User has role `ADMIN` in DB but authority string is missing `ROLE_` prefix. Verify `UserDetailsServiceImpl` prepends `"ROLE_"` before the role name. |
| **LazyInitializationException** | Accessed a `@OneToMany` list outside a transaction. Add `@Transactional` to your service method. |
| **BeanCreationException (circular dependency)** | Bean A needs Bean B and Bean B needs Bean A. Use `@Lazy` on one injection, or restructure to break the cycle. |
| **Validation error not caught** | `GlobalExceptionHandler` is missing the `MethodArgumentNotValidException` handler, or `@Valid` is missing in the controller. |
| **Password stored as plain text** | Forgot to call `passwordEncoder.encode(dto.getPassword())` in service. Check `UserServiceImpl.createUser()`. |
| **Table not created on startup** | Check `ddl-auto=update` in properties. Check DB name in URL matches an existing schema. Check DB credentials. |
| **CORS error in browser** | `CorsConfig` bean missing, or the frontend origin is not in the allowed origins list. Check browser DevTools → Network tab. |
| **N+1 Query Problem** | Using `FetchType.EAGER` on `@OneToMany` causes 1 SELECT per parent row. Use `LAZY` fetch + `@Transactional`. |
| **WeakKeyException at startup** | `app.jwt.secret` is under 32 characters. Make it longer. |

---

## ✅ Final Summary — The 7 Golden Rules

1. **Controller → Service → Repository → Entity** — never skip or reverse layers
2. **Entities go INTO the DB. DTOs come OUT of your API.** Never mix them.
3. **All business logic lives in Service.** Controllers are routing only.
4. **`@Transactional` on every Service method that writes to the DB.**
5. **`JwtUtil` and `JwtAuthFilter` are always copied as-is** — they are infrastructure.
6. **`BCryptPasswordEncoder` is MANDATORY** — never store plain-text passwords.
7. **Add CORS config before connecting any frontend** — browsers block without it.

---

*tech.csm Spring Boot Training · Module Reference Guide*
