# Spring Boot — Complete Concept Notes
### Everything You Need to Know, Explained Simply

> **How to read this document:**
> Every concept is explained in plain English first, then with code.
> Read it top to bottom the first time — concepts build on each other.
> Come back to individual sections as reference when coding.

---

## Table of Contents

**Part 1 — The Foundation**
1. [Spring vs Spring Boot — What is the Difference?](#1-spring-vs-spring-boot)
2. [IoC — Inversion of Control](#2-ioc-inversion-of-control)
3. [DI — Dependency Injection](#3-di-dependency-injection)
4. [The Spring Container](#4-the-spring-container)
5. [Beans — What They Are and How They Work](#5-beans)
6. [Bean Lifecycle](#6-bean-lifecycle)
7. [Annotations — The Complete Reference](#7-annotations)

**Part 2 — Building the Application**

8. [Auto-Configuration — Spring Boot's Superpower](#8-auto-configuration)
9. [Application Properties & Profiles](#9-properties-and-profiles)
10. [The Layered Architecture — Why It Exists](#10-layered-architecture)
11. [Entity & ORM Deep Dive](#11-entity-and-orm)
12. [Repository & Spring Data JPA Deep Dive](#12-repository-deep-dive)
13. [Service Layer — Best Practices](#13-service-layer)
14. [Controller & REST Design](#14-controller-and-rest)
15. [DTO Pattern & Mapping](#15-dto-pattern)
16. [Validation — Complete Guide](#16-validation)
17. [Exception Handling — Complete Guide](#17-exception-handling)

**Part 3 — Security**

18. [Spring Security — How It Actually Works](#18-spring-security)
19. [JWT — Complete Explanation](#19-jwt)
20. [Role-Based Access Control](#20-role-based-access)

**Part 4 — Advanced Concepts**

21. [Transactions — @Transactional Deep Dive](#21-transactions)
22. [Pagination & Sorting](#22-pagination-and-sorting)
23. [CORS](#23-cors)
24. [Logging](#24-logging)
25. [Bean Scopes](#25-bean-scopes)
26. [Spring Profiles](#26-spring-profiles)
27. [Actuator & Monitoring](#27-actuator)

**Part 5 — Putting It All Together**

28. [Complete App Build — Step by Step](#28-complete-app-build)
29. [Best Practices Consolidated](#29-best-practices)
30. [Common Mistakes Reference](#30-common-mistakes)

---

# Part 1 — The Foundation

---

## 1. Spring vs Spring Boot

### What is the Spring Framework?

The Spring Framework is a massive collection of tools, utilities, and conventions for building Java applications. It solves real problems:
- How do objects find and use each other without being tightly coupled?
- How do you manage database connections safely?
- How do you handle security consistently across the entire app?

The problem with the original Spring Framework: **you had to configure everything manually** — XML files, Java config classes, setting up web servers, wiring every component together. A simple "Hello World" REST API took 50+ lines of configuration before you wrote a single line of business logic.

### What is Spring Boot?

Spring Boot is an **opinionated wrapper** on top of Spring Framework. It makes assumptions about what you need and pre-configures most things automatically.

```
Spring Framework  =  The toolbox (all the tools, you assemble them)
Spring Boot       =  The pre-built workshop (tools already arranged, just use them)
```

**The three things Spring Boot adds on top of Spring Framework:**

| Feature | What It Does |
|---------|-------------|
| **Auto-configuration** | Detects what's on your classpath and configures it automatically |
| **Starter POMs** | Bundles related dependencies so you add one line instead of ten |
| **Embedded Server** | Tomcat is bundled — no need to deploy to an external server |

### The Key Difference in Practice

```java
// Old Spring — you had to write this XML (web.xml + applicationContext.xml + servlet config)
// <servlet>
//   <servlet-name>dispatcher</servlet-name>
//   <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
// </servlet>
// ... 30 more lines

// Spring Boot — this is ALL you need
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

Spring Boot reads `@SpringBootApplication`, detects you have `spring-boot-starter-web` on the classpath, and automatically:
- Starts an embedded Tomcat server on port 8080
- Sets up JSON serialization/deserialization
- Configures Spring MVC dispatcher
- Sets up error handling

**Rule:** Spring Boot is not a replacement for Spring — it IS Spring, just with intelligent defaults.

---

## 2. IoC — Inversion of Control

### The Problem Without IoC

Imagine you are building a `UserController`. It needs a `UserService`. Without any framework, you would write:

```java
public class UserController {
    // You create the dependency yourself
    private UserService userService = new UserServiceImpl();
}
```

Problems with this:
1. `UserController` is **tightly coupled** to `UserServiceImpl`. If you create a `MockUserService` for testing, you cannot swap it in.
2. If `UserServiceImpl` needs its own dependencies, `UserController` has to create those too.
3. If you need `UserService` in 10 different places, you create 10 different instances (or manage a singleton yourself).

### What is IoC?

**Inversion of Control** means: instead of YOUR code creating and managing objects, you hand that responsibility OVER to a framework (Spring).

```
Without IoC: You control object creation → new UserService()
With IoC:    Spring controls object creation → Spring creates UserService and gives it to you
```

The word "inversion" refers to this flip: the **control of creating objects** is inverted — it moves from your code to the framework.

### The Hollywood Principle

IoC is often called the "Hollywood Principle":
> "Don't call us — we'll call you."

You don't call `new UserService()`. You tell Spring "I need a UserService here" and Spring calls (injects) it into your class at the right time.

### Why Does This Matter?

```java
// WITHOUT IoC — tightly coupled, cannot test, cannot swap
public class UserController {
    private UserService service = new UserServiceImpl(
        new UserRepository(
            new DatabaseConnection("localhost", "root", "password")
        )
    );
}

// WITH IoC — loose coupling, testable, swappable
public class UserController {
    private final UserService service; // Spring injects this

    public UserController(UserService service) {
        this.service = service;
    }
}
// Now you can inject MockUserService in tests — UserController doesn't care which implementation it gets
```

---

## 3. DI — Dependency Injection

**Dependency Injection (DI)** is the mechanism Spring uses to implement IoC. It is the HOW of IoC.

```
IoC  = the concept (give control to the framework)
DI   = the technique (the framework injects dependencies into your classes)
```

### What is a Dependency?

A dependency is any object your class needs to do its job.

```java
public class UserService {
    private UserRepository repository;  // UserRepository is a dependency of UserService
    private PasswordEncoder encoder;    // PasswordEncoder is a dependency of UserService
}
```

### Three Ways to Inject Dependencies

Spring supports three injection styles. **Constructor Injection is the correct choice in all modern Spring applications.**

#### 1. Constructor Injection (PREFERRED ✅)

```java
@Service
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    // Spring sees this constructor and automatically provides the arguments
    public UserServiceImpl(UserRepository userRepository,
                           PasswordEncoder passwordEncoder) {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }
}

// With Lombok @RequiredArgsConstructor — same thing, less code
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;    // 'final' = required dependency
    private final PasswordEncoder passwordEncoder;  // Lombok generates the constructor
}
```

**Why Constructor Injection is Best:**
- Dependencies are `final` — they cannot be accidentally changed after creation
- The class cannot be created without all its dependencies — no `NullPointerException` surprises
- Instantly reveals if a class has too many dependencies (too many constructor params = class needs splitting)
- Works naturally without Spring in unit tests — just `new UserServiceImpl(mockRepo, mockEncoder)`

#### 2. Field Injection (AVOID ❌)

```java
@Service
public class UserServiceImpl {

    @Autowired  // Spring injects directly into the field via reflection
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;
}
```

**Why to avoid it:**
- Dependencies are not `final` — they can be accidentally reassigned
- Cannot write unit tests without a Spring context — you cannot manually set `private` fields
- Hides dependencies — you cannot tell what a class needs by looking at its constructor
- IntelliJ warns you about this with: "Field injection is not recommended"

#### 3. Setter Injection (USE ONLY FOR OPTIONAL DEPENDENCIES)

```java
@Service
public class UserServiceImpl {

    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

Only use setter injection when a dependency is truly optional and has a default value.

### How Spring Knows What to Inject

When Spring creates a `UserController` and sees it needs a `UserService`:
1. Spring looks in its container for a bean of type `UserService`
2. It finds `UserServiceImpl` (because `UserServiceImpl` is annotated `@Service`, which makes it a bean)
3. Spring injects `UserServiceImpl` into `UserController`

```java
// Spring finds UserServiceImpl because of @Service
@Service
public class UserServiceImpl implements UserService { ... }

// Spring injects it here because UserController declares UserService as a dependency
@RestController
public class UserController {
    private final UserService userService; // Spring injects UserServiceImpl here
}
```

**The rule:** the type declared in the constructor/field must match a bean in the Spring container. If Spring finds 0 beans of that type → exception. If Spring finds 2+ beans of that type → exception (use `@Primary` or `@Qualifier` to resolve).

---

## 4. The Spring Container

### What is the Spring Container?

The Spring Container (also called the **Application Context**) is a big, in-memory registry that:
1. **Creates** all your beans at startup
2. **Stores** them
3. **Injects** them wherever needed
4. **Manages** their entire lifecycle

Think of it as a factory + warehouse for objects. You describe what you need; the container manufactures, stores, and delivers it.

```
Your code says: "I need a UserService"
Container says: "I have one. Here it is." (injects it)

Your code never writes: new UserService()
The container writes that new statement once, keeps the object, and hands it out on demand.
```

### How the Container Starts

When your `main()` method runs `SpringApplication.run(MyApp.class, args)`:

```
1. Spring reads @SpringBootApplication
2. @ComponentScan scans all packages under MyApp's package
3. Spring finds all classes annotated with @Component, @Service, @Repository, @Controller
4. Spring creates instances of all those classes (beans)
5. Spring wires dependencies — injects each bean into wherever it is needed
6. Embedded Tomcat starts
7. App is ready to accept requests
```

### ApplicationContext vs BeanFactory

```
BeanFactory     = the basic container (creates beans on demand — lazy)
ApplicationContext = BeanFactory + extra features (creates beans at startup, event publishing, i18n)
```

In Spring Boot, you always use `ApplicationContext`. `BeanFactory` is low-level and rarely used directly.

### Accessing the Container Programmatically

You rarely need to do this, but it is possible:

```java
@Component
public class SomeClass implements ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.context = ctx;
    }

    public void doSomething() {
        // Manually get a bean from the container
        UserService service = context.getBean(UserService.class);
    }
}
```

**Rule:** Never manually call `getBean()` in normal application code. Always use constructor injection. `getBean()` is for special infrastructure scenarios.

---

## 5. Beans

### What is a Bean?

A **Bean** is simply any object that is created and managed by the Spring Container. That is the entire definition.

```
Regular Java object  =  you create it with 'new', you manage it, you destroy it
Spring Bean          =  Spring creates it, Spring manages it, Spring destroys it
```

### How to Make Something a Bean

**Method 1: Stereotype Annotations** (the standard way — use this)

```java
@Component      // generic Spring-managed component
@Service        // business logic layer (same as @Component + semantic meaning)
@Repository     // data access layer (same as @Component + exception translation)
@Controller     // web layer — handles HTTP (same as @Component + MVC registration)
@RestController // @Controller + @ResponseBody (auto-serialize return values to JSON)
```

All four annotations (`@Service`, `@Repository`, `@Controller`, `@RestController`) are technically `@Component` with extra meaning. Spring treats them all the same way — it creates a bean. The different names are for readability and tooling.

```java
@Service
public class UserServiceImpl implements UserService {
    // Spring sees @Service and creates a bean of type UserServiceImpl
    // It is also registered as type UserService (the interface)
}
```

**Method 2: @Bean Method** (for third-party classes you cannot annotate)

```java
@Configuration
public class AppConfig {

    // You cannot put @Component on BCryptPasswordEncoder (it is a library class)
    // So you create a @Bean method in a @Configuration class
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    // Now Spring can inject PasswordEncoder anywhere in your app
}
```

**Rule:** Use stereotype annotations (`@Service`, `@Repository`, etc.) for your own classes. Use `@Bean` methods for library/third-party classes.

### Bean Naming

Every bean has a name. By default, the name is the class name with lowercase first letter.

```java
@Service
public class UserServiceImpl { }
// Bean name: "userServiceImpl"

@Bean
public PasswordEncoder passwordEncoder() { ... }
// Bean name: "passwordEncoder" (the method name)
```

You can override the name:

```java
@Service("myUserService")
public class UserServiceImpl { }

@Bean("encoder")
public PasswordEncoder passwordEncoder() { ... }
```

### @Primary and @Qualifier

When Spring finds multiple beans of the same type, it does not know which to inject:

```java
@Service
public class EmailNotificationService implements NotificationService { }

@Service
public class SmsNotificationService implements NotificationService { }

// Spring has TWO NotificationService beans — which one to inject into UserService?
```

**Solution 1: @Primary** — marks one bean as the default

```java
@Service
@Primary  // this one is injected by default when NotificationService is requested
public class EmailNotificationService implements NotificationService { }

@Service
public class SmsNotificationService implements NotificationService { }
```

**Solution 2: @Qualifier** — explicitly name which bean to inject

```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl {

    @Qualifier("smsNotificationService")  // inject the SMS one specifically
    private final NotificationService notificationService;
}
```

---

## 6. Bean Lifecycle

Every Spring bean goes through a defined lifecycle from creation to destruction.

```
Spring Container Starts
        ↓
1. Bean Definition Loaded
   (Spring reads @Service, @Component, @Bean annotations)
        ↓
2. Bean Instantiated
   (Spring calls the constructor — dependencies are injected)
        ↓
3. @PostConstruct runs
   (custom initialization code runs AFTER construction)
        ↓
4. Bean is READY — in use by the application
        ↓
5. Spring Container Shuts Down
        ↓
6. @PreDestroy runs
   (custom cleanup code runs BEFORE bean is destroyed)
        ↓
7. Bean is Destroyed
```

### @PostConstruct — Run Code After Bean is Created

```java
@Service
@RequiredArgsConstructor
public class CacheService {

    private final UserRepository userRepository;
    private Map<Long, User> cache;

    // Runs ONCE after the bean is fully constructed and all dependencies are injected
    // Perfect for: loading data into cache, validating config, opening connections
    @PostConstruct
    public void init() {
        // userRepository is already injected here — safe to use
        cache = new HashMap<>();
        userRepository.findAll().forEach(user -> cache.put(user.getId(), user));
        System.out.println("Cache loaded with " + cache.size() + " users");
    }
}
```

**Why not put this in the constructor?**
Because at construction time, other dependencies may not yet be injected. `@PostConstruct` guarantees all injection is complete.

### @PreDestroy — Run Code Before Bean is Destroyed

```java
@Service
public class ConnectionPoolService {

    private Connection connection;

    @PostConstruct
    public void openConnection() {
        connection = openDatabaseConnection();
    }

    // Runs ONCE before the Spring container shuts down
    // Perfect for: closing connections, flushing caches, releasing resources
    @PreDestroy
    public void closeConnection() {
        if (connection != null) {
            connection.close();
        }
    }
}
```

### InitializingBean and DisposableBean (older approach)

```java
// Old way — avoid in new code, use @PostConstruct and @PreDestroy instead
@Service
public class OldStyleService implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        // equivalent to @PostConstruct
    }

    @Override
    public void destroy() throws Exception {
        // equivalent to @PreDestroy
    }
}
```

---

## 7. Annotations

Annotations are Java's way of attaching metadata to classes, methods, and fields. Spring reads these annotations at startup and uses them to configure your application. No code is generated — Spring uses reflection to read them at runtime.

### Core Spring Annotations

```java
// ── BEAN CREATION ─────────────────────────────────────────────────────────────
@Component          // Generic bean. Use when no specific layer applies.
@Service            // Business logic layer bean. No technical difference to @Component.
@Repository         // Data layer bean. Adds DB exception translation.
@Controller         // Web layer bean. Registers with Spring MVC dispatcher.
@RestController     // @Controller + @ResponseBody. Every method returns JSON automatically.
@Configuration      // Class that contains @Bean methods.
@Bean               // Method that produces a Spring-managed bean.

// ── INJECTION ────────────────────────────────────────────────────────────────
@Autowired          // Marks injection point. Not needed on constructors in Spring 4+.
@Qualifier("name")  // Specifies which bean to inject when multiple exist.
@Primary            // Marks the default bean when multiple of same type exist.
@Value("${key}")    // Injects a value from application.properties.

// ── SCANNING & CONFIG ────────────────────────────────────────────────────────
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
@ComponentScan          // Tells Spring where to look for @Component-annotated classes.
@EnableAutoConfiguration // Triggers auto-configuration based on classpath.
@PropertySource         // Load additional .properties files.
@Import                 // Import another @Configuration class.

// ── LIFECYCLE ────────────────────────────────────────────────────────────────
@PostConstruct          // Run after bean is fully created and injected.
@PreDestroy             // Run before bean is destroyed.

// ── SCOPE ────────────────────────────────────────────────────────────────────
@Scope("singleton")     // One instance per container (default).
@Scope("prototype")     // New instance every time the bean is requested.
@RequestScope           // One instance per HTTP request.
@SessionScope           // One instance per HTTP session.
```

### Spring MVC / Web Annotations

```java
// ── CONTROLLER MAPPING ───────────────────────────────────────────────────────
@RequestMapping("/path")    // Base URL. Can go on class (prefix) or method.
@GetMapping("/path")        // HTTP GET shorthand.
@PostMapping("/path")       // HTTP POST shorthand.
@PutMapping("/path")        // HTTP PUT shorthand.
@PatchMapping("/path")      // HTTP PATCH shorthand.
@DeleteMapping("/path")     // HTTP DELETE shorthand.

// ── METHOD PARAMETERS ────────────────────────────────────────────────────────
@PathVariable               // Extract {id} from URL path.
@RequestParam               // Extract ?name=value from query string.
@RequestBody                // Parse JSON body into Java object.
@RequestHeader              // Read an HTTP header value.
@ResponseBody               // Serialize return value to JSON (included in @RestController).
@ResponseStatus(HttpStatus.CREATED)  // Set HTTP status code on the method.

// ── CROSS CUTTING ────────────────────────────────────────────────────────────
@CrossOrigin                // Allow cross-origin requests on this controller/method.
```

### JPA / Persistence Annotations

```java
// ── ENTITY DEFINITION ───────────────────────────────────────────────────────
@Entity                         // This class is a DB table.
@Table(name = "users")          // Specify table name.
@Id                             // This field is the primary key.
@GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment.
@Column(name="col", nullable=false, unique=true, length=100)  // Column config.
@Transient                      // This field is NOT a DB column — ignored by Hibernate.

// ── RELATIONSHIPS ────────────────────────────────────────────────────────────
@OneToOne                       // One-to-one relationship.
@OneToMany(mappedBy="field")    // One parent, many children.
@ManyToOne                      // Many children, one parent.
@ManyToMany                     // Both sides can have multiple.
@JoinColumn(name="fk_col")      // Specifies the FK column on the owning side.
@JoinTable                      // For @ManyToMany — specifies the join table.

// ── INHERITANCE ──────────────────────────────────────────────────────────────
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)  // All subclasses in one table.
@DiscriminatorColumn(name="type")  // Column that distinguishes subclasses.

// ── LIFECYCLE CALLBACKS ──────────────────────────────────────────────────────
@PrePersist     // Before INSERT.
@PreUpdate      // Before UPDATE.
@PreRemove      // Before DELETE.
@PostLoad       // After entity is loaded from DB.
```

### Spring Security Annotations

```java
@EnableWebSecurity          // Enable Spring Security (auto in Spring Boot).
@EnableMethodSecurity       // Enable @PreAuthorize/@PostAuthorize on methods.
@PreAuthorize("hasRole('ADMIN')")   // Method only runs if expression is true.
@PostAuthorize("...")               // Check AFTER method returns.
@Secured("ROLE_ADMIN")      // Simple role-based access (less flexible than @PreAuthorize).
```

### Transaction Annotations

```java
@Transactional                          // Wrap method in a transaction.
@Transactional(readOnly = true)         // Read-only transaction (better performance).
@Transactional(rollbackFor = Exception.class)  // Rollback on checked exceptions too.
@Transactional(propagation = Propagation.REQUIRES_NEW)  // Always create a new transaction.
@EnableTransactionManagement            // Enable transaction support (auto in Spring Boot).
```

---

# Part 2 — Building the Application

---

## 8. Auto-Configuration

### What Auto-Configuration Does

Auto-configuration is Spring Boot detecting what libraries are on your classpath and configuring them automatically — without any action from you.

```
You add mysql-connector-j to pom.xml
          ↓
Spring Boot detects: "MySQL driver is present"
          ↓
Spring Boot auto-configures: DataSource, JPA, HikariCP connection pool
          ↓
You just write application.properties with DB credentials — nothing else
```

### How It Works Internally

Spring Boot ships with hundreds of `@Configuration` classes inside `spring-boot-autoconfigure.jar`. Each one has a condition:

```java
// This is INSIDE Spring Boot's source (you don't write this — Spring Boot does)
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })  // only runs IF these classes are on classpath
@ConditionalOnMissingBean(DataSource.class)  // only runs IF you haven't already defined your own DataSource
public class DataSourceAutoConfiguration {

    @Bean
    public DataSource dataSource() {
        // Creates a HikariCP connection pool using your application.properties values
    }
}
```

The `@ConditionalOn...` annotations are the key — they ensure auto-config only applies when relevant and never overrides your own beans.

### Key Conditions

| Annotation | Meaning |
|------------|---------|
| `@ConditionalOnClass` | Only apply if this class is on the classpath |
| `@ConditionalOnMissingBean` | Only apply if you have NOT defined your own bean of this type |
| `@ConditionalOnProperty` | Only apply if a property exists/has a specific value |
| `@ConditionalOnBean` | Only apply if another specific bean exists |

### Seeing What Was Auto-Configured

Add this to `application.properties` and check the startup logs:

```properties
logging.level.org.springframework.boot.autoconfigure=DEBUG
```

You will see lines like:
```
DataSourceAutoConfiguration matched: @ConditionalOnClass found mysql-connector-j
SecurityAutoConfiguration matched: @ConditionalOnClass found spring-security-core
```

### Overriding Auto-Configuration

Because of `@ConditionalOnMissingBean`, you can override any auto-configuration simply by defining your own bean:

```java
// You define your own DataSource
@Bean
public DataSource dataSource() {
    // Spring Boot's auto-config sees YOUR DataSource exists → skips auto-configuring one
    HikariDataSource ds = new HikariDataSource();
    ds.setMaximumPoolSize(50);  // custom pool size
    return ds;
}
```

**Rule:** You almost never need to override auto-configuration. Only do it for edge cases.

---

## 9. Properties and Profiles

### application.properties vs application.yml

Spring Boot reads configuration from `src/main/resources/application.properties` (or `application.yml` — both work, `.properties` is clearer for beginners).

```properties
# application.properties — KEY=VALUE format, simple and explicit
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
```

```yaml
# application.yml — indented YAML format, cleaner for complex configs
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
```

**Rule:** Use `application.properties` — it is simpler and less error-prone (YAML is whitespace-sensitive).

### Reading Properties in Code with @Value

```java
@Component
public class JwtUtil {

    // Reads 'app.jwt.secret' from application.properties
    // The 'defaultValue' part after ':' is used if the property doesn't exist
    @Value("${app.jwt.secret:default-secret}")
    private String secret;

    @Value("${app.jwt.expiration:86400000}")
    private long expiration;

    @Value("${server.port}")
    private int port;
}
```

### Reading Properties with @ConfigurationProperties (Better for Groups)

When you have a group of related properties, use `@ConfigurationProperties` instead of multiple `@Value`:

```properties
# application.properties
app.jwt.secret=my-secret-key-at-least-32-characters-long
app.jwt.expiration=86400000
app.jwt.refresh-expiration=604800000
app.jwt.issuer=my-app
```

```java
// Automatically binds all app.jwt.* properties to this class
@ConfigurationProperties(prefix = "app.jwt")
@Component
@Data  // Lombok — generates getters/setters needed for binding
public class JwtProperties {
    private String secret;
    private long expiration;
    private long refreshExpiration;  // app.jwt.refresh-expiration → camelCase auto-converted
    private String issuer;
}

// Usage — inject like any other bean
@Service
@RequiredArgsConstructor
public class JwtUtil {
    private final JwtProperties jwtProperties;  // all JWT config in one object

    public String getSecret() {
        return jwtProperties.getSecret();
    }
}
```

**Rule:** Use `@Value` for 1-2 isolated properties. Use `@ConfigurationProperties` for a group of related properties — it is cleaner, type-safe, and supports IDE auto-completion.

---

## 10. Layered Architecture

### Why Have Layers?

Every non-trivial application has multiple concerns that should be kept separate:

```
HTTP communication    ← that is the Controller's job
Business logic        ← that is the Service's job
Data access           ← that is the Repository's job
Data shape in DB      ← that is the Entity's job
Data shape in API     ← that is the DTO's job
```

### The Dependency Rule

Dependencies flow **downward** only. Upper layers depend on lower layers. Lower layers never depend on upper layers.

```
Controller
    │ depends on
    ↓
Service (interface)
    │ depends on
    ↓
Repository (interface)
    │ depends on
    ↓
Entity (plain class)
```

A `Repository` should never import anything from `Controller`. A `Service` should never import from `Controller`. If you find yourself doing this, your design is wrong.

### What Belongs Where

**Controller must:**
- Accept HTTP requests
- Extract data from request (path variables, request body, query params)
- Call exactly ONE service method
- Return a response with the correct HTTP status code

**Controller must NOT:**
- Contain any business logic
- Directly call a repository
- Know anything about the database

**Service must:**
- Contain ALL business rules ("email must be unique", "user must be active to login")
- Orchestrate calls to one or more repositories
- Transform data between layers

**Service must NOT:**
- Know what HTTP is (no `HttpServletRequest`, no `@RequestMapping`)
- Return entities directly — always convert to DTOs before returning

**Repository must:**
- Query the database
- Return entities (not DTOs)

**Repository must NOT:**
- Contain business logic
- Know about HTTP

**Entity must:**
- Represent the database table structure
- Contain lifecycle callbacks (`@PrePersist`, `@PreUpdate`)
- Contain helper methods for the entity itself

**Entity must NOT:**
- Know about HTTP
- Know about business rules
- Contain service calls or repository calls

---

## 11. Entity and ORM Deep Dive

### What is ORM?

ORM = Object Relational Mapping. The "impedance mismatch" problem: Java works with objects and inheritance; relational databases work with tables, rows, and foreign keys. ORM bridges this gap.

```
Java World             ←→     Database World
class User             ←→     table users
User instance          ←→     row in users
String name field      ←→     VARCHAR(100) name column
Long id field          ←→     BIGINT id column (PRIMARY KEY)
List<Order> orders     ←→     orders table with user_id FK
```

### Hibernate as the JPA Provider

JPA (Java Persistence API) is a **specification** — a set of interfaces and rules.
Hibernate is an **implementation** of JPA — it is the actual engine that generates and executes SQL.

```
You write:  repository.save(user)
JPA says:   "OK, I need to persist this entity"
Hibernate:  "I will generate: INSERT INTO users (name, email) VALUES (?, ?)"
Hibernate:  Executes SQL against MySQL
```

You code against JPA interfaces (so your code is portable). Hibernate does the actual work.

### Entity Mapping — Complete Example

```java
package tech.csm.demo.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(
    name = "users",
    // Table-level constraints — indexes for performance and uniqueness at DB level
    uniqueConstraints = {
        @UniqueConstraint(columnNames = "email", name = "uk_users_email")
    },
    indexes = {
        @Index(columnList = "email", name = "idx_users_email")
    }
)
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
@EqualsAndHashCode(exclude = {"orders"})  // exclude collections from equals/hashCode
@ToString(exclude = {"orders", "password"})  // exclude sensitive/circular fields
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "full_name", nullable = false, length = 100)
    private String name;

    @Column(nullable = false, length = 150)
    private String email;

    @Column(nullable = false)
    private String password;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private Role role = Role.USER;  // default value

    @Column(nullable = false)
    private Boolean active = true;

    // @Lob for large text fields
    @Lob
    @Column(columnDefinition = "TEXT")
    private String bio;

    @Column(updatable = false)
    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    // ── RELATIONSHIPS ─────────────────────────────────────────────────────────

    // One User has many Orders
    @OneToMany(
        mappedBy = "user",           // field in Order that has the FK
        cascade = CascadeType.ALL,   // operations on User cascade to Orders
        fetch = FetchType.LAZY,      // don't load orders unless accessed
        orphanRemoval = true         // if Order removed from list, delete it from DB
    )
    @Builder.Default
    private List<Order> orders = new ArrayList<>();

    // One User has one Profile
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private UserProfile profile;

    // ── LIFECYCLE CALLBACKS ───────────────────────────────────────────────────
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // ── HELPER METHODS (entity-level only — no business logic) ────────────────
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);  // maintain bidirectional consistency
    }

    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }

    // ── NESTED ENUM ───────────────────────────────────────────────────────────
    public enum Role {
        USER, ADMIN, MODERATOR, SUPPORT
    }
}
```

### Cascade Types — What They Mean

| CascadeType | Meaning |
|-------------|---------|
| `PERSIST` | Saving parent also saves children |
| `MERGE` | Updating parent also updates children |
| `REMOVE` | Deleting parent also deletes children |
| `REFRESH` | Refreshing parent also refreshes children |
| `DETACH` | Detaching parent also detaches children |
| `ALL` | All of the above combined |

```java
// CascadeType.ALL means:
User user = new User();
Order order = new Order();
user.getOrders().add(order);

repository.save(user);      // PERSIST cascades → order is also saved
repository.delete(user);    // REMOVE cascades → all user's orders are also deleted
```

### FetchType — LAZY vs EAGER

```
LAZY  = "Don't load related data until I actually access it"
EAGER = "Always load related data immediately, even if I don't need it"
```

```java
// LAZY example (correct)
User user = userRepository.findById(1L).get();
// At this point: only user data is loaded. Orders are NOT in memory.

List<Order> orders = user.getOrders();
// NOW: Hibernate fires a second SQL query to load orders
// SELECT * FROM orders WHERE user_id = 1

// EAGER example (problematic)
// SELECT * FROM users JOIN orders ON orders.user_id = users.id WHERE users.id = 1
// Orders loaded immediately even if you never use them
```

**Rule: Always use `FetchType.LAZY` for collections (`@OneToMany`, `@ManyToMany`).** EAGER on collections causes severe performance problems — every user query loads all their orders, even when you only need the user's name.

### The N+1 Query Problem

This is the most common ORM performance problem:

```java
// You load 100 users
List<User> users = userRepository.findAll();
// SQL: SELECT * FROM users   (1 query)

// Then you iterate and access their orders
for (User user : users) {
    System.out.println(user.getOrders().size()); // triggers 1 query per user
}
// SQL: SELECT * FROM orders WHERE user_id = 1
// SQL: SELECT * FROM orders WHERE user_id = 2
// ... 100 more queries
// Total: 1 + 100 = 101 queries for what should be 2 queries
```

**Solutions:**

```java
// Solution 1: JOIN FETCH in JPQL — load everything in one query
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(Long id);

// Solution 2: @EntityGraph — declarative fetch (Spring Data JPA)
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();

// Solution 3: Projections — only select the columns you need
public interface UserNameOnly {
    String getName();
    String getEmail();
}
List<UserNameOnly> findAllProjectedBy();  // SELECT name, email FROM users (no join at all)
```

---

## 12. Repository Deep Dive

### The Repository Pattern

The Repository pattern provides a collection-like abstraction over data access. Your Service thinks it is working with a collection of entities — it does not know or care about SQL.

```java
// Service sees this (abstract, collection-like)
userRepository.findAll()         // like a List.getAll()
userRepository.save(user)        // like a List.add()
userRepository.deleteById(1L)    // like a List.remove()
userRepository.findById(1L)      // like a List.get()

// Hibernate/JPA translates to this (actual SQL)
// SELECT * FROM users
// INSERT INTO users ...
// DELETE FROM users WHERE id = 1
// SELECT * FROM users WHERE id = 1
```

### JpaRepository Hierarchy

```
Repository                          (marker interface, no methods)
    └── CrudRepository              (basic CRUD: save, findById, findAll, delete)
            └── PagingAndSortingRepository  (adds pagination and sorting)
                    └── JpaRepository       (adds flush, saveAndFlush, batch ops)
```

You extend `JpaRepository` and get everything from all four levels.

### Optional — Handling Nullable Results

`findById()` and derived `findBy*` methods return `Optional<T>`, not `T`. This forces you to handle the case where the record does not exist.

```java
// Optional<T> is a container that may or may not hold a value

// Getting the value — 4 ways:
Optional<User> opt = userRepository.findById(5L);

// 1. orElseThrow — throw exception if empty (MOST COMMON in Spring apps)
User user = opt.orElseThrow(() -> new ResourceNotFoundException("User", "id", 5L));

// 2. orElse — return a default value if empty
User user = opt.orElse(new User()); // returns empty User if not found

// 3. orElseGet — compute default lazily
User user = opt.orElseGet(() -> createDefaultUser());

// 4. isPresent / isEmpty — check first
if (opt.isPresent()) {
    User user = opt.get();
}

// 5. ifPresent — only execute if value exists
opt.ifPresent(user -> sendWelcomeEmail(user));
```

**Rule:** Never call `opt.get()` without first checking `isPresent()` or using `orElseThrow`. Calling `.get()` on an empty Optional throws `NoSuchElementException`.

### Custom Queries — Three Approaches

#### Approach 1: Derived Query Methods (for simple queries)

```java
// Spring reads the method name and generates the SQL
List<User> findByNameAndActiveTrue(String name);
// SELECT * FROM users WHERE name = ? AND active = true

List<User> findByEmailContainingIgnoreCaseOrderByCreatedAtDesc(String email);
// SELECT * FROM users WHERE UPPER(email) LIKE UPPER(%?%) ORDER BY created_at DESC
```

Limit: method names get long and unreadable for complex queries.

#### Approach 2: @Query with JPQL (for complex queries)

```java
// JPQL uses Entity class names and field names, NOT table/column names
@Query("SELECT u FROM User u WHERE u.role = :role AND u.active = true ORDER BY u.createdAt DESC")
List<User> findActiveUsersByRole(@Param("role") User.Role role);

// With pagination
@Query("SELECT u FROM User u WHERE u.active = true")
Page<User> findAllActive(Pageable pageable);

// Modifying query (UPDATE/DELETE)
@Modifying
@Transactional
@Query("UPDATE User u SET u.active = false WHERE u.id = :id")
int deactivateUser(@Param("id") Long id);  // returns number of rows affected
```

#### Approach 3: @Query with Native SQL (for DB-specific queries)

```java
// Use when JPQL cannot express what you need, or for performance tuning
@Query(
    value = "SELECT * FROM users WHERE DATE(created_at) = CURDATE()",
    nativeQuery = true
)
List<User> findUsersCreatedToday();

// Native with pagination (must also provide countQuery)
@Query(
    value = "SELECT * FROM users WHERE active = 1",
    countQuery = "SELECT COUNT(*) FROM users WHERE active = 1",
    nativeQuery = true
)
Page<User> findActiveUsersNative(Pageable pageable);
```

**Rule priority:** Use derived methods for simple queries → JPQL for complex queries → Native SQL only when necessary.

### Projections — Fetching Only What You Need

```java
// Interface-based projection — only load specific columns
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
}

// In repository
List<UserSummary> findAllProjectedBy();
// Generates: SELECT id, name, email FROM users  (not SELECT *)

// Closed projection with aliased expressions
public interface UserStats {
    String getEmail();
    @Value("#{target.name + ' (' + target.role + ')'}")
    String getLabel();
}
```

---

## 13. Service Layer

### The Contract: Interface First

Always define a service interface before the implementation. This is not optional — it is how you write testable, loosely-coupled code.

```java
// Step 1: Define the contract (what this service can do)
public interface ProductService {
    ProductResponseDTO createProduct(ProductRequestDTO dto);
    ProductResponseDTO getProductById(Long id);
    Page<ProductResponseDTO> getProducts(Pageable pageable);
    ProductResponseDTO updateProduct(Long id, ProductRequestDTO dto);
    void deleteProduct(Long id);
    List<ProductResponseDTO> getProductsByCategory(String category);
    boolean isProductAvailable(Long id, int quantity);
}

// Step 2: Implement the contract (HOW it does it)
@Service
@Slf4j
@RequiredArgsConstructor
@Transactional  // applies @Transactional to all methods by default
public class ProductServiceImpl implements ProductService {

    private final ProductRepository productRepository;

    // Override class-level @Transactional for read operations
    @Override
    @Transactional(readOnly = true)
    public ProductResponseDTO getProductById(Long id) {
        log.debug("Fetching product with id: {}", id);
        return productRepository.findById(id)
                .map(ProductResponseDTO::fromEntity)
                .orElseThrow(() -> new ResourceNotFoundException("Product", "id", id));
    }

    // Class-level @Transactional applies here (write operation)
    @Override
    public ProductResponseDTO createProduct(ProductRequestDTO dto) {
        log.info("Creating new product: {}", dto.getName());
        // business rule
        if (productRepository.existsByName(dto.getName())) {
            throw new IllegalArgumentException("Product already exists: " + dto.getName());
        }
        Product product = Product.builder()
                .name(dto.getName())
                .price(dto.getPrice())
                .stock(dto.getStock())
                .build();
        return ProductResponseDTO.fromEntity(productRepository.save(product));
    }
}
```

### Transaction Propagation

When one `@Transactional` method calls another, what happens to the transaction?

```java
@Service
public class OrderServiceImpl {

    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;

    @Transactional  // starts Transaction A
    public Order placeOrder(OrderRequestDTO dto) {
        Order order = orderRepository.save(buildOrder(dto));  // runs in Transaction A

        // inventoryService.reduceStock() is also @Transactional
        // With REQUIRED (default): joins Transaction A — same transaction
        inventoryService.reduceStock(dto.getProductId(), dto.getQuantity());

        return order;
        // Transaction A commits HERE — both order save AND stock reduction commit together
        // If anything throws, BOTH are rolled back
    }
}
```

| Propagation | Meaning |
|-------------|---------|
| `REQUIRED` | Join existing transaction. If none, create a new one. **(Default)** |
| `REQUIRES_NEW` | Always create a new transaction. Suspend the current one. |
| `SUPPORTS` | Join if there is one. Run without transaction if there is none. |
| `NOT_SUPPORTED` | Always run without a transaction. Suspend current one. |
| `MANDATORY` | Must have an existing transaction. Throws if there is none. |
| `NEVER` | Must NOT have an existing transaction. Throws if there is one. |
| `NESTED` | Run within a nested transaction (savepoint) if one exists. |

```java
// REQUIRES_NEW example — log the attempt EVEN IF the order fails
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logOrderAttempt(String userId, String action) {
    // This runs in its OWN transaction
    // Even if the parent transaction (placeOrder) rolls back, this commit stands
    auditRepository.save(new AuditLog(userId, action, LocalDateTime.now()));
}
```

### When Does @Transactional Rollback?

**By default:** `@Transactional` only rolls back on `RuntimeException` (unchecked exceptions) and `Error`.

```java
@Transactional
public void doSomething() throws IOException {  // checked exception
    repository.save(entity);
    throw new IOException("File not found");   // does NOT roll back by default!
}

// Fix: explicitly declare which exceptions trigger rollback
@Transactional(rollbackFor = {IOException.class, Exception.class})
public void doSomething() throws IOException {
    repository.save(entity);
    throw new IOException("File not found");  // NOW rolls back
}

// Or use RuntimeException (extends RuntimeException, always rolls back)
@Transactional
public void doSomething() {
    repository.save(entity);
    throw new RuntimeException("Always rolls back");  // rolls back
}
```

---

## 14. Controller and REST Design

### REST Principles

REST (Representational State Transfer) is an architectural style for building APIs. Your API is RESTful when it follows these conventions:

| Principle | Meaning | Example |
|-----------|---------|---------|
| **Resources** | URLs represent nouns, not verbs | `/api/users` not `/api/getUsers` |
| **HTTP methods** | Method indicates action | GET=read, POST=create, PUT=replace, PATCH=update, DELETE=remove |
| **Stateless** | Each request carries all needed info | No sessions — JWT carries identity |
| **Consistent responses** | Same structure always | Always return JSON with same error shape |
| **Versioning** | Version your API | `/api/v1/users` not `/api/users` |

### URL Design Rules

```
✅ CORRECT — nouns, hierarchical, lowercase, hyphens
GET     /api/v1/users                   (get all users)
GET     /api/v1/users/5                 (get user with id 5)
POST    /api/v1/users                   (create a user)
PUT     /api/v1/users/5                 (replace user with id 5)
PATCH   /api/v1/users/5                 (partially update user 5)
DELETE  /api/v1/users/5                 (delete user 5)

GET     /api/v1/users/5/orders          (get orders belonging to user 5)
GET     /api/v1/users/5/orders/10       (get order 10 of user 5)
POST    /api/v1/users/5/orders          (create order for user 5)

❌ WRONG — verbs in URLs (the HTTP method is already the verb)
GET     /api/v1/getUsers
GET     /api/v1/getUserById?id=5
POST    /api/v1/createUser
DELETE  /api/v1/deleteUser/5
POST    /api/v1/users/update            (PUT /api/v1/users/5 is correct)
```

### Controller Best Practices

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
@Tag(name = "Users", description = "User management endpoints")  // Swagger/OpenAPI
public class UserController {

    private final UserService userService;

    // ── Consistent: always return ResponseEntity<> ────────────────────────────

    // GET /api/v1/users?page=0&size=10&sort=name,asc
    @GetMapping
    public ResponseEntity<Page<UserResponseDTO>> getUsers(
            @PageableDefault(size = 10, sort = "id") Pageable pageable) {
        // @PageableDefault sets sensible defaults — client doesn't need to send all params
        return ResponseEntity.ok(userService.getUsers(pageable));
    }

    // GET /api/v1/users/5
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDTO> getUserById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.getUserById(id));
    }

    // POST /api/v1/users → 201 Created
    @PostMapping
    public ResponseEntity<UserResponseDTO> createUser(
            @Valid @RequestBody UserRequestDTO dto) {
        UserResponseDTO created = userService.createUser(dto);
        // Optionally include Location header pointing to new resource
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();
        return ResponseEntity.created(location).body(created);
    }

    // PUT /api/v1/users/5
    @PutMapping("/{id}")
    public ResponseEntity<UserResponseDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UserRequestDTO dto) {
        return ResponseEntity.ok(userService.updateUser(id, dto));
    }

    // PATCH /api/v1/users/5 — partial update (only send the fields you want to change)
    @PatchMapping("/{id}")
    public ResponseEntity<UserResponseDTO> patchUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        return ResponseEntity.ok(userService.patchUser(id, updates));
    }

    // DELETE /api/v1/users/5 → 204 No Content
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
}
```

### @PageableDefault — Clean Pagination in Controllers

```java
// Spring Boot can auto-resolve Pageable from query params:
// GET /api/v1/users?page=0&size=10&sort=name,asc&sort=createdAt,desc

@GetMapping
public ResponseEntity<Page<UserResponseDTO>> getUsers(
        @PageableDefault(
            size = 10,          // default page size
            sort = "createdAt", // default sort field
            direction = Sort.Direction.DESC  // default sort direction
        ) Pageable pageable) {
    return ResponseEntity.ok(userService.getUsers(pageable));
}
```

---

## 15. DTO Pattern and Mapping

### Why DTOs?

The Entity is your internal database representation. The DTO is your external API contract. These should be separate because:

1. **Security**: Entity has `password` field — you never want that in a response
2. **Shape**: API response may combine fields from multiple entities
3. **Versioning**: API shape can change without changing the DB schema
4. **Validation**: Validation rules on input are different from DB constraints

### Standard DTO Naming Convention

```
UserRequestDTO   → what client SENDS to create/update
UserResponseDTO  → what server SENDS BACK to client
UserSummaryDTO   → lightweight version for lists/search results
UserDetailDTO    → full version with all relations for a detail view
AuthRequestDTO   → login input
AuthResponseDTO  → login output (token)
```

### The fromEntity Pattern

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserResponseDTO {
    private Long id;
    private String name;
    private String email;
    private String role;
    private LocalDateTime createdAt;

    // Static method on the DTO itself — keeps mapping code co-located with the DTO
    public static UserResponseDTO fromEntity(User user) {
        return UserResponseDTO.builder()
                .id(user.getId())
                .name(user.getName())
                .email(user.getEmail())
                .role(user.getRole().name())
                .createdAt(user.getCreatedAt())
                .build();
    }

    // For lists — maps a list of entities to a list of DTOs
    public static List<UserResponseDTO> fromEntityList(List<User> users) {
        return users.stream()
                .map(UserResponseDTO::fromEntity)
                .collect(Collectors.toList());
    }
}
```

### The toEntity Pattern (on RequestDTO)

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserRequestDTO {

    @NotBlank
    @Size(min = 2, max = 100)
    private String name;

    @NotBlank
    @Email
    private String email;

    @NotBlank
    @Size(min = 8)
    private String password;

    // Convert to entity for persistence
    // Note: caller must hash the password before passing it to the entity
    public User toEntity(String hashedPassword) {
        return User.builder()
                .name(this.name)
                .email(this.email)
                .password(hashedPassword)
                .role(User.Role.USER)
                .build();
    }
}
```

### When to Use MapStruct (for complex projects)

For large projects with many entities, writing `fromEntity` methods manually becomes repetitive. MapStruct is an annotation processor that generates the mapping code for you:

```java
// You write this mapper interface
@Mapper(componentModel = "spring")  // makes it a Spring bean
public interface UserMapper {
    UserResponseDTO toDto(User user);
    User toEntity(UserRequestDTO dto);
    List<UserResponseDTO> toDtoList(List<User> users);
}

// MapStruct generates the implementation at compile time — no reflection, no runtime overhead
// Usage in service:
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserMapper userMapper;  // inject the generated mapper

    public UserResponseDTO createUser(UserRequestDTO dto) {
        User user = userMapper.toEntity(dto);
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        return userMapper.toDto(userRepository.save(user));
    }
}
```

**Rule:** For small-medium projects, use `fromEntity()` static methods. For large projects with 10+ entities, use MapStruct.

---

## 16. Validation

### How Validation Works

When you annotate a controller method parameter with `@Valid`:
1. Spring intercepts the call before the method runs
2. Spring passes the DTO to the Jakarta Validator
3. Validator checks every annotated field
4. If any check fails → `MethodArgumentNotValidException` is thrown
5. Your `GlobalExceptionHandler` catches it and returns a 400 response

This all happens automatically — your controller method only runs if all validations pass.

### Field-Level Annotations

```java
@Data
public class ProductRequestDTO {

    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 200, message = "Name must be 2–200 characters")
    private String name;

    @NotNull(message = "Price is required")
    @DecimalMin(value = "0.01", message = "Price must be greater than 0")
    @DecimalMax(value = "999999.99", message = "Price is too high")
    @Digits(integer = 6, fraction = 2, message = "Price format: up to 6 digits and 2 decimals")
    private BigDecimal price;

    @NotNull(message = "Stock is required")
    @Min(value = 0, message = "Stock cannot be negative")
    @Max(value = 100000, message = "Stock too large")
    private Integer stock;

    @NotBlank(message = "Category is required")
    private String category;

    @Size(max = 2000, message = "Description too long")
    private String description;  // optional, but limited to 2000 chars if provided

    @NotNull(message = "Available status required")
    private Boolean available;

    @Pattern(regexp = "^[A-Z]{2,5}-\\d{4}$", message = "SKU must match format: AB-1234")
    private String sku;  // optional custom format validation

    @Email(message = "Supplier email must be valid")
    private String supplierEmail;  // optional

    @Future(message = "Expiry must be in the future")
    private LocalDate expiryDate;  // optional
}
```

### Class-Level Validation (Cross-Field Validation)

When one field's validity depends on another field:

```java
// Step 1: Create the custom annotation
@Target(ElementType.TYPE)  // applies to the class, not a field
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = PasswordMatchValidator.class)
public @interface PasswordsMatch {
    String message() default "Passwords do not match";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Step 2: Create the validator
public class PasswordMatchValidator implements ConstraintValidator<PasswordsMatch, RegisterRequestDTO> {

    @Override
    public boolean isValid(RegisterRequestDTO dto, ConstraintValidatorContext context) {
        if (dto.getPassword() == null || dto.getConfirmPassword() == null) {
            return true;  // let @NotBlank handle null checks
        }
        return dto.getPassword().equals(dto.getConfirmPassword());
    }
}

// Step 3: Use on your DTO class
@PasswordsMatch(message = "Password and confirm password must match")
@Data
public class RegisterRequestDTO {

    @NotBlank
    @Size(min = 8)
    private String password;

    @NotBlank
    private String confirmPassword;
}
```

### Validation Groups — Apply Different Rules in Different Contexts

```java
// Define groups as marker interfaces
public interface OnCreate {}
public interface OnUpdate {}

// Use groups on fields
@Data
public class UserRequestDTO {

    @NotBlank(groups = OnCreate.class)  // required when creating
    @Size(min = 8, groups = {OnCreate.class, OnUpdate.class})
    private String password;

    @NotBlank(groups = {OnCreate.class, OnUpdate.class})
    private String name;
}

// Specify which group to validate in the controller
@PostMapping       // CREATE
public ResponseEntity<?> create(@Validated(OnCreate.class) @RequestBody UserRequestDTO dto) { ... }

@PutMapping("/{id}")  // UPDATE
public ResponseEntity<?> update(@PathVariable Long id,
                                 @Validated(OnUpdate.class) @RequestBody UserRequestDTO dto) { ... }
```

### Manual Validation in Service

Sometimes you need to validate business rules that cannot be expressed with annotations:

```java
@Service
public class UserServiceImpl {

    public UserResponseDTO createUser(UserRequestDTO dto) {
        // Business validation — cannot be done with annotations
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new IllegalArgumentException("Email already registered");
        }

        // Domain-specific rule
        if (dto.getAge() != null && dto.getAge() < 18) {
            throw new IllegalArgumentException("Users must be 18 or older");
        }

        // ... rest of creation logic
    }
}
```

---

## 17. Exception Handling

### The Exception Hierarchy in Spring

```
Throwable
├── Error           (OutOfMemoryError etc. — do NOT catch these)
└── Exception
    ├── IOException (checked — must declare or catch)
    ├── SQLException (checked)
    └── RuntimeException (unchecked — Spring handles by default)
        ├── IllegalArgumentException     (your business validation errors)
        ├── IllegalStateException        (invalid state errors)
        ├── NullPointerException
        ├── ResourceNotFoundException    (your custom — extends RuntimeException)
        └── UsernameNotFoundException    (Spring Security)
```

**Rule:** Always use `RuntimeException` (unchecked) for business errors in Spring apps. It keeps your code clean (no `throws` declarations everywhere) and Spring rolls back transactions on it automatically.

### Custom Exception Hierarchy (for larger projects)

```java
// Base exception for your application
public class AppException extends RuntimeException {
    private final int statusCode;
    private final String errorCode;  // machine-readable code for the client

    public AppException(String message, int statusCode, String errorCode) {
        super(message);
        this.statusCode = statusCode;
        this.errorCode = errorCode;
    }

    // Getters
    public int getStatusCode() { return statusCode; }
    public String getErrorCode() { return errorCode; }
}

// Specific exceptions
public class ResourceNotFoundException extends AppException {
    public ResourceNotFoundException(String resource, String field, Object value) {
        super(resource + " not found with " + field + ": " + value, 404, "RESOURCE_NOT_FOUND");
    }
}

public class DuplicateResourceException extends AppException {
    public DuplicateResourceException(String message) {
        super(message, 409, "DUPLICATE_RESOURCE");
    }
}

public class UnauthorizedException extends AppException {
    public UnauthorizedException(String message) {
        super(message, 401, "UNAUTHORIZED");
    }
}

public class ForbiddenException extends AppException {
    public ForbiddenException(String message) {
        super(message, 403, "FORBIDDEN");
    }
}
```

### Complete GlobalExceptionHandler

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // ── Standard error response ───────────────────────────────────────────────
    @Data
    @AllArgsConstructor
    public static class ErrorResponse {
        private LocalDateTime timestamp;
        private int status;
        private String error;
        private String errorCode;
        private String message;
        private String path;
    }

    // ── 404 Not Found ─────────────────────────────────────────────────────────
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());
        return ResponseEntity.status(404)
                .body(new ErrorResponse(LocalDateTime.now(), 404,
                        "Not Found", ex.getErrorCode(),
                        ex.getMessage(), request.getRequestURI()));
    }

    // ── 409 Conflict ──────────────────────────────────────────────────────────
    @ExceptionHandler({DuplicateResourceException.class, IllegalArgumentException.class})
    public ResponseEntity<ErrorResponse> handleConflict(
            RuntimeException ex, HttpServletRequest request) {
        return ResponseEntity.status(409)
                .body(new ErrorResponse(LocalDateTime.now(), 409,
                        "Conflict", "CONFLICT",
                        ex.getMessage(), request.getRequestURI()));
    }

    // ── 400 Validation errors ─────────────────────────────────────────────────
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, Object>> handleValidation(
            MethodArgumentNotValidException ex) {
        Map<String, String> fieldErrors = new LinkedHashMap<>();
        ex.getBindingResult().getFieldErrors()
                .forEach(err -> fieldErrors.put(err.getField(), err.getDefaultMessage()));

        return ResponseEntity.badRequest().body(Map.of(
                "timestamp", LocalDateTime.now(),
                "status", 400,
                "error", "Validation Failed",
                "fieldErrors", fieldErrors
        ));
    }

    // ── 400 Type mismatch (e.g. string passed for Long id) ───────────────────
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(
            MethodArgumentTypeMismatchException ex, HttpServletRequest request) {
        String message = "Parameter '" + ex.getName() + "' must be of type " +
                ex.getRequiredType().getSimpleName();
        return ResponseEntity.badRequest()
                .body(new ErrorResponse(LocalDateTime.now(), 400,
                        "Bad Request", "TYPE_MISMATCH",
                        message, request.getRequestURI()));
    }

    // ── 401 Authentication failure ────────────────────────────────────────────
    @ExceptionHandler(BadCredentialsException.class)
    public ResponseEntity<ErrorResponse> handleBadCredentials(
            BadCredentialsException ex, HttpServletRequest request) {
        return ResponseEntity.status(401)
                .body(new ErrorResponse(LocalDateTime.now(), 401,
                        "Unauthorized", "INVALID_CREDENTIALS",
                        "Invalid email or password", request.getRequestURI()));
    }

    // ── 403 Access Denied ─────────────────────────────────────────────────────
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(
            AccessDeniedException ex, HttpServletRequest request) {
        return ResponseEntity.status(403)
                .body(new ErrorResponse(LocalDateTime.now(), 403,
                        "Forbidden", "ACCESS_DENIED",
                        "You do not have permission to perform this action",
                        request.getRequestURI()));
    }

    // ── 500 Catch-all ─────────────────────────────────────────────────────────
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleAll(
            Exception ex, HttpServletRequest request) {
        log.error("Unhandled exception on {}: {}", request.getRequestURI(), ex.getMessage(), ex);
        return ResponseEntity.internalServerError()
                .body(new ErrorResponse(LocalDateTime.now(), 500,
                        "Internal Server Error", "INTERNAL_ERROR",
                        "An unexpected error occurred", request.getRequestURI()));
    }
}
```

---

# Part 3 — Security

---

## 18. Spring Security — How It Actually Works

### The Security Filter Chain

Spring Security works as a chain of **Servlet Filters** that intercept every HTTP request before it reaches your controller. Think of it as a gauntlet — the request must pass through every filter before reaching the controller.

```
HTTP Request
    ↓
Filter 1: CorsFilter             (handles CORS headers)
    ↓
Filter 2: SecurityContextPersistenceFilter  (loads SecurityContext)
    ↓
Filter 3: JwtAuthFilter (YOUR CUSTOM FILTER) (validates JWT, sets auth)
    ↓
Filter 4: UsernamePasswordAuthenticationFilter  (Spring's default, we bypass this)
    ↓
Filter 5: ExceptionTranslationFilter  (converts security exceptions to HTTP responses)
    ↓
Filter 6: FilterSecurityInterceptor  (checks access rules)
    ↓
DispatcherServlet
    ↓
Your Controller
```

### SecurityContextHolder — The Auth Storage

During a request, Spring Security stores the authenticated user's information in `SecurityContextHolder`. It uses a `ThreadLocal` — each thread (each request) gets its own storage.

```java
// After JwtAuthFilter sets auth, any part of your code can read who is logged in:

// Method 1: Via SecurityContextHolder
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();             // the email (our 'username')
Collection<GrantedAuthority> roles = auth.getAuthorities();

// Method 2: Via @AuthenticationPrincipal in Controller (cleaner)
@GetMapping("/me")
public ResponseEntity<UserResponseDTO> getCurrentUser(
        @AuthenticationPrincipal UserDetails userDetails) {
    // userDetails is injected by Spring from the SecurityContext
    String email = userDetails.getUsername();
    return ResponseEntity.ok(userService.getUserByEmail(email));
}
```

### Authentication vs Authorization

```
Authentication = verifying IDENTITY ("Who are you?")
                 → Login: is this email/password combination valid?
                 → JWT: is this token valid and not expired?

Authorization  = verifying PERMISSIONS ("What can you do?")
                 → Does this user have the ADMIN role?
                 → Is this user allowed to access this specific resource?
```

Spring Security handles both. Authentication happens in the filter chain (JwtAuthFilter + `authenticationManager`). Authorization happens after — in the URL rules (`authorizeHttpRequests`) and method rules (`@PreAuthorize`).

### Authentication Internals — What Happens During Login

```
AuthController.login() calls authenticationManager.authenticate(token)
    ↓
AuthenticationManager delegates to DaoAuthenticationProvider
    ↓
DaoAuthenticationProvider calls UserDetailsService.loadUserByUsername(email)
    ↓
UserDetailsService loads User from DB, returns UserDetails
    ↓
DaoAuthenticationProvider uses PasswordEncoder.matches(rawPassword, hashedPassword)
    ↓
If match: Authentication object is created and returned
If no match: BadCredentialsException is thrown
    ↓
AuthController generates JWT from the UserDetails
    ↓
JWT returned to client
```

### PasswordEncoder — Why BCrypt

```java
// Passwords are ALWAYS stored as hashes — never plain text
PasswordEncoder encoder = new BCryptPasswordEncoder();

// Hashing a password (happens on registration)
String raw = "myPassword123";
String hashed = encoder.encode(raw);
// hashed = "$2a$10$EXAMPLEHASHSHOULDBELONGERTHAN50chars..."
// The hash is different every time even for the same password (BCrypt uses a random salt)

// Checking a password (happens on login)
boolean matches = encoder.matches(raw, hashed);  // true
boolean matches2 = encoder.matches("wrongPass", hashed);  // false

// You never decrypt the hash — you re-hash the attempt and compare
```

**Why BCrypt and not MD5/SHA?**
- BCrypt is deliberately slow (configurable cost factor) — makes brute-force impractical
- BCrypt includes a random salt in the hash — same password produces different hashes
- MD5 and SHA are fast and unslated — rainbow table attacks crack them in seconds
- BCrypt `strength=10` (default) means `2^10 = 1024` iterations per hash

---

## 19. JWT — Complete Explanation

### Why JWT for REST APIs?

HTTP is stateless — each request is independent. Traditional web apps use sessions:
- Server stores session in memory or DB
- Client gets a session cookie
- Server looks up session on every request

Problems with sessions:
- Does not scale — if you have 10 servers, which server has the session?
- Memory-intensive — millions of sessions in memory
- Requires sticky sessions or a shared session store (Redis)

JWT solves this:
- Server stores **nothing** — all state is in the token
- Client sends the token with every request
- Server validates the signature — no lookup needed
- Stateless — any server can validate any token

### JWT Structure — Decoded

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiJ1c2VyQGV4YW1wbGUuY29tIiwicm9sZXMiOiJbUk9MRV9VU0VSXSIsImlhdCI6MTcwNTI5NTgwMCwiZXhwIjoxNzA1MzgyMjAwfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Part 1 (Header — Base64 decoded):
{
  "alg": "HS256",   // algorithm used to sign
  "typ": "JWT"
}

Part 2 (Payload — Base64 decoded):
{
  "sub": "user@example.com",   // subject = who this token is for
  "roles": "[ROLE_USER]",      // custom claim — roles embedded in token
  "iat": 1705295800,           // issued at (Unix timestamp)
  "exp": 1705382200            // expires at (Unix timestamp)
}

Part 3 (Signature):
HMACSHA256(
    base64Url(header) + "." + base64Url(payload),
    secret
)
```

The payload is Base64URL encoded — NOT encrypted. Anyone can decode and read it. **Never put sensitive data (passwords, SSNs, credit cards) in JWT claims.**

The signature IS cryptographically protected. Without the secret key, you cannot produce a valid signature. Tampering with the payload produces a different signature → server rejects it.

### Token Refresh Pattern

Access tokens should be short-lived (15 minutes to 24 hours). When they expire, the user should not have to log in again. The refresh token pattern solves this:

```java
// Two tokens:
// Access Token  — short-lived (15-60 min), used for API calls
// Refresh Token — long-lived (7-30 days), used ONLY to get a new access token

@PostMapping("/login")
public ResponseEntity<AuthResponseDTO> login(@RequestBody AuthRequestDTO dto) {
    // ... authenticate ...
    String accessToken = jwtUtil.generateAccessToken(userDetails);   // expires in 15 min
    String refreshToken = jwtUtil.generateRefreshToken(userDetails); // expires in 7 days

    return ResponseEntity.ok(AuthResponseDTO.builder()
            .accessToken(accessToken)
            .refreshToken(refreshToken)
            .expiresIn(900000L)   // 15 minutes in ms
            .build());
}

// Client calls this when access token expires
@PostMapping("/refresh")
public ResponseEntity<AuthResponseDTO> refresh(@RequestBody Map<String, String> body) {
    String refreshToken = body.get("refreshToken");
    // Validate refresh token, extract user, issue new access token
    if (jwtUtil.isTokenValid(refreshToken)) {
        String email = jwtUtil.extractUsername(refreshToken);
        UserDetails userDetails = userDetailsService.loadUserByUsername(email);
        String newAccessToken = jwtUtil.generateAccessToken(userDetails);
        return ResponseEntity.ok(AuthResponseDTO.builder()
                .accessToken(newAccessToken)
                .expiresIn(900000L)
                .build());
    }
    throw new UnauthorizedException("Invalid or expired refresh token");
}
```

---

## 20. Role-Based Access Control

### Understanding Roles and Authorities

```
Role       = a named group of permissions ("ADMIN", "USER", "MODERATOR")
Authority  = a specific permission ("READ_USERS", "DELETE_ORDERS")

In Spring Security:
- Role    = authority with the "ROLE_" prefix
- "ADMIN" role = authority string "ROLE_ADMIN"
```

```java
// In UserDetailsServiceImpl — how authorities are set:
.authorities(List.of(new SimpleGrantedAuthority("ROLE_" + user.getRole().name())))
// User.Role.ADMIN → "ROLE_ADMIN"
// User.Role.USER  → "ROLE_USER"
```

### Access Control Methods

**Method 1: URL-based rules in SecurityConfig**

```java
.authorizeHttpRequests(auth -> auth
    .requestMatchers("/api/v1/auth/**").permitAll()
    .requestMatchers(HttpMethod.GET, "/api/v1/products/**").permitAll()  // public read
    .requestMatchers("/api/v1/admin/**").hasRole("ADMIN")               // admin only
    .requestMatchers("/api/v1/users/**").hasAnyRole("USER", "ADMIN")    // multiple roles
    .anyRequest().authenticated()
)
```

**Method 2: @PreAuthorize on methods (preferred for granular control)**

```java
// Role check
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { }

// Multiple roles
@PreAuthorize("hasAnyRole('ADMIN', 'MODERATOR')")
public void flagContent(Long id) { }

// Check the logged-in user's email matches the resource owner
@PreAuthorize("authentication.name == #email")
public UserResponseDTO getUserByEmail(String email) { }

// Combine conditions
@PreAuthorize("hasRole('ADMIN') or authentication.name == #dto.email")
public UserResponseDTO updateUser(Long id, UserRequestDTO dto) { }

// Check AFTER method runs — can inspect the return value
@PostAuthorize("returnObject.email == authentication.name or hasRole('ADMIN')")
public UserResponseDTO getUserById(Long id) { }
```

**Method 3: Programmatic check in Service**

```java
@Service
public class UserServiceImpl {

    public UserResponseDTO updateUser(Long id, UserRequestDTO dto) {
        // Programmatic security check
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        String currentUserEmail = auth.getName();
        boolean isAdmin = auth.getAuthorities().stream()
                .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));

        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User", "id", id));

        // Only allow if current user is admin OR is updating their own account
        if (!isAdmin && !user.getEmail().equals(currentUserEmail)) {
            throw new ForbiddenException("Cannot update another user's account");
        }

        // ... proceed with update
    }
}
```

---

# Part 4 — Advanced Concepts

---

## 21. Transactions

### The ACID Properties

A database transaction is a unit of work that satisfies ACID:

| Property | Meaning |
|----------|---------|
| **Atomicity** | All operations succeed, or all are rolled back — no partial updates |
| **Consistency** | Database moves from one valid state to another valid state |
| **Isolation** | Concurrent transactions do not interfere with each other |
| **Durability** | Committed data is permanently saved even if the server crashes |

### How @Transactional Works — Internally

When you annotate a method with `@Transactional`, Spring creates a **proxy** object wrapping your service:

```
You write:
    @Service
    public class UserServiceImpl {
        @Transactional
        public void createUser(UserRequestDTO dto) { ... }
    }

Spring creates a proxy at runtime:
    class UserServiceImpl$$SpringCGLIBProxy extends UserServiceImpl {
        @Override
        public void createUser(UserRequestDTO dto) {
            // Spring adds this before your code:
            transactionManager.beginTransaction();
            try {
                super.createUser(dto);   // your actual code runs here
                transactionManager.commit();
            } catch (RuntimeException e) {
                transactionManager.rollback();
                throw e;
            }
        }
    }
```

This proxy is what Spring injects, not your class directly. This is why `@Transactional` on `private` methods does NOT work — the proxy cannot override private methods.

### Common @Transactional Mistakes

```java
// ❌ MISTAKE 1: @Transactional on private method — does NOT work
@Service
public class UserServiceImpl {

    @Transactional  // IGNORED — proxy cannot intercept private methods
    private void internalCreate(User user) {
        userRepository.save(user);
    }
}

// ❌ MISTAKE 2: Calling @Transactional method from within the same class
@Service
public class UserServiceImpl {

    public void outerMethod() {
        this.innerMethod();  // calls directly on 'this', not through the proxy
    }

    @Transactional  // IGNORED — 'this.innerMethod()' bypasses the proxy
    public void innerMethod() { ... }
}

// ✅ FIX: Separate classes, or self-injection (last resort)

// ❌ MISTAKE 3: Catching exceptions inside @Transactional prevents rollback
@Transactional
public void doSomething() {
    try {
        userRepository.save(user);
        throw new RuntimeException("error");
    } catch (Exception e) {
        log.error("Error occurred"); // exception swallowed → transaction COMMITS with corrupt data
    }
}

// ✅ FIX: Rethrow or do not catch inside transactional methods
@Transactional
public void doSomething() {
    userRepository.save(user);
    // throw the exception — let it propagate up to trigger rollback
    throw new RuntimeException("error");
}
```

---

## 22. Pagination and Sorting

### Pageable — The Core Type

`Pageable` is an interface that carries page number, page size, and sort instructions.

```java
// Ways to create a Pageable:
Pageable p1 = PageRequest.of(0, 10);                     // page 0, size 10, no sort
Pageable p2 = PageRequest.of(0, 10, Sort.by("name"));    // sorted by name ASC
Pageable p3 = PageRequest.of(0, 10, Sort.by("name").descending()); // sorted DESC
Pageable p4 = PageRequest.of(0, 10,
    Sort.by(Sort.Order.desc("createdAt"), Sort.Order.asc("name"))
);  // multi-column sort

// Spring can auto-build Pageable from URL params:
// GET /users?page=0&size=10&sort=name,desc&sort=createdAt,asc
```

### Page<T> — What the Response Contains

```java
Page<User> page = userRepository.findAll(PageRequest.of(0, 10));

page.getContent()       // List<User> — the data for this page
page.getTotalElements() // long — total rows in DB matching the query
page.getTotalPages()    // int — how many pages exist
page.getNumber()        // int — current page index (0-based)
page.getSize()          // int — page size
page.isFirst()          // boolean — is this page 0?
page.isLast()           // boolean — is there no next page?
page.hasNext()          // boolean
page.hasPrevious()      // boolean
page.getSort()          // Sort — what sort was applied
```

### Slice<T> — Lighter Alternative

```java
// Page counts total — requires extra COUNT(*) query
// Slice does NOT count — just knows if there is a next page
// Use Slice for infinite scroll / "load more" patterns

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Slice<User> findByActiveTrue(Pageable pageable);
}

Slice<User> slice = userRepository.findByActiveTrue(pageable);
slice.hasNext();     // true/false — just checks if more data exists
slice.getContent();  // List<User> — the data
// No totalElements, no totalPages — faster query
```

---

## 23. CORS

### Same-Origin Policy

Browsers enforce the Same-Origin Policy (SOP): JavaScript can only make requests to the same origin as the page it is running on.

```
Origin = scheme + domain + port
http://localhost:3000  ≠  http://localhost:8080   (different ports)
https://myapp.com      ≠  https://api.myapp.com  (different subdomain)
http://myapp.com       ≠  https://myapp.com      (different scheme)
```

### Preflight Request

For non-simple requests (any request with `Authorization` header, or `Content-Type: application/json`), the browser sends a **preflight** `OPTIONS` request first:

```
Browser: OPTIONS /api/v1/users
         Origin: http://localhost:3000
         Access-Control-Request-Method: POST
         Access-Control-Request-Headers: Authorization, Content-Type

Server:  Access-Control-Allow-Origin: http://localhost:3000
         Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
         Access-Control-Allow-Headers: Authorization, Content-Type
         Access-Control-Max-Age: 3600  (cache this preflight for 1 hour)

Browser: OK, now send the actual POST request
```

If the preflight fails, the browser never sends the actual request — Postman always succeeds because it does not enforce SOP.

---

## 24. Logging

### SLF4J — The Logging Facade

SLF4J (Simple Logging Facade for Java) is an API. Logback (Spring Boot's default) is the implementation behind it. You always code against SLF4J — the actual logger can be swapped without changing your code.

```java
// Two ways to get the logger:

// Method 1: Lombok @Slf4j (preferred — zero boilerplate)
@Slf4j
@Service
public class UserService {
    // Lombok injects: private static final Logger log = LoggerFactory.getLogger(UserService.class);
}

// Method 2: Manual declaration
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserService {
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
}
```

### Structured Logging — What to Log

```java
@Service
@Slf4j
public class UserServiceImpl {

    @Transactional
    public UserResponseDTO createUser(UserRequestDTO dto) {
        // 1. Log entry for significant operations at DEBUG
        log.debug("createUser called with email: {}", dto.getEmail());

        // 2. Log business rule violations at WARN
        if (userRepository.existsByEmail(dto.getEmail())) {
            log.warn("Registration attempt with existing email: {}", dto.getEmail());
            throw new DuplicateResourceException("Email already registered");
        }

        User saved = userRepository.save(buildUser(dto));

        // 3. Log significant events at INFO
        log.info("User created successfully. id={}, email={}", saved.getId(), saved.getEmail());

        return UserResponseDTO.fromEntity(saved);
    }

    public void deleteUser(Long id) {
        if (!userRepository.existsById(id)) {
            // 4. Log not-found at DEBUG (expected scenario, not an error)
            log.debug("Delete requested for non-existent user id: {}", id);
            throw new ResourceNotFoundException("User", "id", id);
        }
        userRepository.deleteById(id);
        log.info("User deleted. id={}", id);  // audit trail
    }

    private void sendEmail(String email) {
        try {
            emailService.send(email);
        } catch (Exception e) {
            // 5. Log unexpected errors at ERROR with exception stack trace
            log.error("Failed to send welcome email to: {}. Error: {}", email, e.getMessage(), e);
            // Note: passing 'e' as last arg causes SLF4J to print the full stack trace
        }
    }
}
```

### What NOT to Log

```java
// ❌ Never log sensitive data
log.info("User logged in with password: {}", password);  // NEVER
log.debug("JWT token: {}", token);                        // NEVER
log.info("Credit card: {}", cardNumber);                  // NEVER

// ❌ Never use string concatenation in log calls
log.debug("Processing user: " + user.toString());  // String built even if DEBUG is off

// ✅ Always use {} placeholders — lazy evaluation
log.debug("Processing user: {}", user.getId());  // String only built if DEBUG level is active
```

---

## 25. Bean Scopes

### Singleton — The Default

```java
// One instance per Spring container — shared across all requests
// This is the DEFAULT if you do not specify a scope
@Service  // same as @Service + @Scope("singleton")
public class UserService { }

// All these injections point to the SAME object:
@RestController
public class UserController {
    @Autowired UserService service;  // same object
}
@RestController
public class OrderController {
    @Autowired UserService service;  // same object as UserController's service
}
```

**Implication:** Singleton beans must be **stateless** (no instance variables that change per request). If you store request-specific data in an instance variable of a singleton, concurrent requests will corrupt each other's data.

```java
// ❌ WRONG — storing state in a singleton
@Service
public class UserServiceImpl {
    private User currentUser;  // DANGER: shared across all requests!

    public UserResponseDTO processRequest() {
        // Request 1 sets currentUser = UserA
        // Request 2 sets currentUser = UserB (overwrites Request 1's data!)
        // Request 1 continues and now sees UserB — corrupt!
    }
}

// ✅ CORRECT — stateless singleton, data passed as parameters
@Service
public class UserServiceImpl {
    public UserResponseDTO processRequest(UserRequestDTO dto) {
        // dto is a method parameter — local to this stack frame
        // Each request has its own copy on the stack
    }
}
```

### Prototype — New Instance Per Request

```java
// A new instance is created every time this bean is requested
@Component
@Scope("prototype")
public class ReportGenerator {
    // Can safely hold state — each caller gets their own instance
    private final List<String> lines = new ArrayList<>();

    public void addLine(String line) { lines.add(line); }
    public String generate() { return String.join("\n", lines); }
}

// Usage — must use ApplicationContext or @Lookup to get a new instance:
@Service
public class ReportService {
    @Autowired
    private ApplicationContext context;

    public String generateReport() {
        // Gets a NEW ReportGenerator instance each time
        ReportGenerator generator = context.getBean(ReportGenerator.class);
        generator.addLine("Line 1");
        return generator.generate();
    }
}
```

### Request Scope — One Per HTTP Request

```java
// New instance for each HTTP request, destroyed when the request ends
@Component
@RequestScope  // same as @Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestContext {
    private String correlationId;
    private Long authenticatedUserId;
    // Can safely store per-request data here
}
```

---

## 26. Spring Profiles

### What Profiles Do

Profiles let you have different configurations for different environments:

```
dev     → local MySQL, DEBUG logging, show SQL
test    → H2 in-memory DB, minimal logging
prod    → RDS MySQL, WARN logging, no show-SQL
```

### Setting Up Profile-Specific Properties

```
src/main/resources/
    application.properties          ← always loaded (common config)
    application-dev.properties      ← loaded when 'dev' profile is active
    application-test.properties     ← loaded when 'test' profile is active
    application-prod.properties     ← loaded when 'prod' profile is active
```

```properties
# application.properties (common)
spring.application.name=student-service
app.jwt.expiration=86400000

# application-dev.properties (development)
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/student_db_dev
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
logging.level.tech.csm=DEBUG

# application-prod.properties (production)
server.port=8443
spring.datasource.url=jdbc:mysql://${DB_HOST}:3306/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASS}
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=false
logging.level.root=WARN
```

### Activating a Profile

```properties
# In application.properties
spring.profiles.active=dev
```

```bash
# Via command line (overrides application.properties)
java -jar app.jar --spring.profiles.active=prod

# Via environment variable
SPRING_PROFILES_ACTIVE=prod java -jar app.jar
```

### @Profile on Beans

```java
// This bean only exists when 'dev' profile is active
@Configuration
@Profile("dev")
public class DevDataSeeder {

    @Bean
    @PostConstruct
    public void seedData() {
        // Insert test data into DB — only runs in dev
    }
}

// This bean only exists when 'prod' profile is active
@Configuration
@Profile("prod")
public class ProdEmailConfig {

    @Bean
    public JavaMailSender mailSender() {
        // Real SMTP config
    }
}
```

---

## 27. Actuator

Spring Boot Actuator adds production-ready monitoring endpoints to your application.

```xml
<!-- Add to pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# Expose specific endpoints
management.endpoints.web.exposure.include=health,info,metrics,env,beans
management.endpoint.health.show-details=always
management.server.port=8081  # separate port for actuator (security)
```

### Key Actuator Endpoints

| Endpoint | URL | What It Shows |
|----------|-----|---------------|
| Health | `/actuator/health` | App health status, DB connection, disk space |
| Info | `/actuator/info` | App version, description (from `application.properties`) |
| Metrics | `/actuator/metrics` | JVM memory, CPU, HTTP request counts, response times |
| Beans | `/actuator/beans` | All Spring beans in the context |
| Env | `/actuator/env` | All environment properties |
| Mappings | `/actuator/mappings` | All URL mappings / endpoints in the app |
| Loggers | `/actuator/loggers` | View and change log levels at runtime |

**Rule:** Never expose all actuator endpoints on the same port as your API in production. Use `management.server.port` to separate them, or secure them with Spring Security.

---

# Part 5 — Putting It All Together

---

## 28. Complete App Build — Step by Step

This section shows the exact sequence to build any Spring Boot application from scratch. Follow this order every time.

### Phase 1: Setup (Do This Once)

**Step 1: Create the project**
```
Go to: https://start.spring.io
Select: Maven, Java 17+, Spring Boot 3.3.x
Dependencies: Spring Web, Spring Data JPA, Spring Security,
              MySQL Driver, Lombok, Validation
Download and open in IntelliJ
```

**Step 2: Set up the database**
```sql
-- In MySQL Workbench or terminal
CREATE DATABASE student_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'yourpassword';
GRANT ALL PRIVILEGES ON student_db.* TO 'appuser'@'localhost';
FLUSH PRIVILEGES;
```

**Step 3: Configure application.properties**
```properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/student_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=appuser
spring.datasource.password=yourpassword
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
app.jwt.secret=your-secret-key-that-is-at-least-32-characters-long
app.jwt.expiration=86400000
logging.level.tech.csm=DEBUG
```

**Step 4: Add pom.xml JJWT dependency**
```xml
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
```

### Phase 2: Entity Layer

**Step 5: Create your entity class**

For each table you need:
1. Create class in `entity/` package
2. Add `@Entity`, `@Table`, `@Id`, `@GeneratedValue`
3. Add all fields with appropriate `@Column` annotations
4. Add `@PrePersist` and `@PreUpdate` for audit fields
5. Use Lombok: `@Data`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor`

```java
@Entity
@Table(name = "students")
@Data @Builder @NoArgsConstructor @AllArgsConstructor
@EqualsAndHashCode(exclude = {"courses"})
@ToString(exclude = {"courses"})
public class Student {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true, length = 150)
    private String email;

    @Column(nullable = false)
    private String password;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private Role role = Role.STUDENT;

    @Column(updatable = false)
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist protected void onCreate() { createdAt = updatedAt = LocalDateTime.now(); }
    @PreUpdate  protected void onUpdate() { updatedAt = LocalDateTime.now(); }

    public enum Role { STUDENT, ADMIN }
}
```

### Phase 3: Repository Layer

**Step 6: Create repository interface**

For each entity:
1. Create interface in `repository/` package
2. Extend `JpaRepository<Entity, IdType>`
3. Add custom query methods as needed

```java
@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    Optional<Student> findByEmail(String email);
    boolean existsByEmail(String email);
    List<Student> findByRole(Student.Role role);
    Page<Student> findAll(Pageable pageable);
}
```

### Phase 4: DTO Layer

**Step 7: Create DTOs**

For each entity:
1. Create `EntityRequestDTO` in `dto/` package (for input)
2. Create `EntityResponseDTO` in `dto/` package (for output)
3. Add validation annotations on the request DTO
4. Add `fromEntity()` static method on the response DTO
5. Create `AuthRequestDTO` and `AuthResponseDTO`

### Phase 5: Service Layer

**Step 8: Create service interface and implementation**

1. Create `EntityService` interface in `service/` package — list all method signatures
2. Create `EntityServiceImpl` class in `service/` package
3. Annotate class: `@Service`, `@Slf4j`, `@RequiredArgsConstructor`
4. Add `@Transactional(readOnly = true)` on all read methods
5. Add `@Transactional` on all write methods
6. Implement business rules (existence checks, permission checks)
7. Always convert entities to DTOs before returning

### Phase 6: Copy Security Package

**Step 9: Copy the security package exactly**

Copy these four files (they are the same in every project — only `UserRepository` reference changes):
- `UserDetailsServiceImpl.java` — change the repository and entity type
- `JwtUtil.java` — copy as-is
- `JwtAuthFilter.java` — copy as-is
- `SecurityConfig.java` — copy as-is, update URL rules for your app

```java
// Only thing to change in UserDetailsServiceImpl:
@Service
@RequiredArgsConstructor
public class UserDetailsServiceImpl implements UserDetailsService {

    private final StudentRepository studentRepository;  // ← change this

    @Override
    public UserDetails loadUserByUsername(String email) throws UsernameNotFoundException {
        var student = studentRepository.findByEmail(email)  // ← and this
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + email));

        return User.builder()
                .username(student.getEmail())
                .password(student.getPassword())
                .authorities(List.of(new SimpleGrantedAuthority(
                        "ROLE_" + student.getRole().name())))
                .build();
    }
}
```

### Phase 7: Copy Exception Package

**Step 10: Copy exception classes**

Copy exactly as-is — they are reusable across all projects:
- `ResourceNotFoundException.java`
- `GlobalExceptionHandler.java`

### Phase 8: Controller Layer

**Step 11: Create controllers**

1. Create `EntityController` in `controller/` package
2. Annotate: `@RestController`, `@RequestMapping("/api/v1/entities")`, `@RequiredArgsConstructor`
3. Inject the service interface (NOT the impl)
4. Implement all CRUD endpoints with correct HTTP methods and status codes
5. Create `AuthController` — always the same, copy and adjust

### Phase 9: CORS and Config

**Step 12: Add CorsConfig**

```java
@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOrigin("http://localhost:3000");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        config.setAllowCredentials(true);
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

### Phase 10: Test

**Step 13: Test with Postman**

```
1. POST /api/v1/auth/register   → { name, email, password }       → expect 200
2. POST /api/v1/auth/login      → { email, password }             → expect 200 + token
3. GET  /api/v1/students        → Header: Authorization: Bearer <token> → expect 200
4. POST /api/v1/students        → Header + Body                   → expect 201
5. GET  /api/v1/students/1      → Header                          → expect 200
6. PUT  /api/v1/students/1      → Header + Body                   → expect 200
7. DELETE /api/v1/students/1    → Header (admin token)            → expect 204

Test error cases:
8. GET /api/v1/students/9999    → Header                          → expect 404
9. POST /api/v1/auth/login with wrong password                    → expect 401
10. DELETE without ADMIN role                                     → expect 403
```

---

## 29. Best Practices Consolidated

### Code Organization

```
✅ One public class per file, file name = class name
✅ Package by layer: entity, repository, dto, service, controller, security, exception, config
✅ All classes in the same root package as @SpringBootApplication or sub-packages
✅ Interface names without I prefix (UserService not IUserService)
✅ Implementation names with Impl suffix (UserServiceImpl)
✅ DTO names with DTO suffix (UserResponseDTO, UserRequestDTO)
```

### Naming Conventions

```
Classes:    PascalCase    →  UserServiceImpl, ProductController
Methods:    camelCase     →  getUserById, createProduct
Variables:  camelCase     →  userRepository, totalCount
Constants:  UPPER_SNAKE   →  MAX_RETRY_ATTEMPTS, DEFAULT_PAGE_SIZE
Packages:   lowercase     →  tech.csm.demo.service
DB tables:  snake_case    →  user_profiles, order_items
DB columns: snake_case    →  created_at, user_id
URLs:       kebab-case    →  /api/v1/user-profiles
```

### Entity Rules

```
✅ Always use @Enumerated(EnumType.STRING) — never store ordinals
✅ Always add createdAt and updatedAt with @PrePersist/@PreUpdate
✅ Always use FetchType.LAZY for collections
✅ Use @Column(nullable = false) — enforce NOT NULL at JPA level, not just DB
✅ Exclude collections from @EqualsAndHashCode and @ToString
✅ Use Long (not int) for IDs — int overflows at ~2 billion
✅ Use LocalDateTime (not Date or Timestamp) — modern Java time API
```

### Service Rules

```
✅ Always use @Transactional on write methods
✅ Always use @Transactional(readOnly = true) on read methods
✅ Validate business rules in service, not controller
✅ Always convert entity to DTO before returning from service
✅ Log all significant operations at INFO, debug details at DEBUG
✅ Throw specific exceptions (ResourceNotFoundException) not generic RuntimeException
✅ Depend on the Repository interface (JpaRepository), not a specific implementation
```

### Controller Rules

```
✅ No business logic in controllers — only routing and delegation
✅ Always use @Valid on @RequestBody parameters
✅ Always return ResponseEntity<> with explicit status codes
✅ Use correct HTTP status codes: 201 for create, 204 for delete
✅ Inject service interface, not implementation class
✅ Use @PathVariable for IDs, @RequestParam for optional filters
✅ Version your API: /api/v1/...
```

### Security Rules

```
✅ Never store plain-text passwords — always BCrypt
✅ JWT secret must be 32+ characters
✅ Use STATELESS session policy — no sessions/cookies
✅ Permit only /auth/** and /public/** without token
✅ Add /error to permitAll() — Spring's internal error redirect
✅ Use @PreAuthorize for method-level access control
✅ Set appropriate JWT expiry (15 min-24 hours for access tokens)
✅ Never put sensitive data (passwords, SSNs) in JWT claims
```

### Performance Rules

```
✅ Always use FetchType.LAZY on collections
✅ Use @Transactional(readOnly = true) for all read operations
✅ Use pagination — never return unbounded lists from APIs
✅ Add database indexes on columns used in WHERE clauses and JOINs
✅ Use @Column(nullable = false) — DB-level constraints improve query planning
✅ Use projections when you only need a few columns from a large entity
✅ Set HikariCP connection pool size appropriately (default is 10)
```

---

## 30. Common Mistakes Reference

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Field injection (`@Autowired` on field) | Works but untestable, IntelliJ warning | Switch to constructor injection |
| Business logic in Controller | Fat controllers, hard to test | Move to Service layer |
| Returning Entity instead of DTO | Password/internal fields exposed | Create and use ResponseDTO |
| Missing `@Transactional` on write | Partial saves on exception | Add `@Transactional` to service write methods |
| `@Transactional` on private method | Transaction silently ignored | Only use on public methods |
| EAGER fetch on collections | N+1 queries, slow performance | Use LAZY + @EntityGraph where needed |
| Storing plain-text password | Critical security breach | Always use `passwordEncoder.encode()` |
| JWT secret under 32 chars | `WeakKeyException` at startup | Increase secret length |
| No `@Valid` on controller params | Invalid data reaches service | Add `@Valid` before `@RequestBody` |
| No GlobalExceptionHandler | Stack traces returned to client | Add `@RestControllerAdvice` class |
| Missing CORS config | 403/Network Error in browser | Add `CorsConfig` bean |
| `Optional.get()` without check | `NoSuchElementException` | Use `orElseThrow()` instead |
| Committing credentials to Git | Security breach | Use environment variables in production |
| Catching exceptions in `@Transactional` | Transaction commits with bad data | Let exceptions propagate |
| `ddl-auto=create` in production | All data wiped on restart | Use `validate` or `none` in prod |
| No pagination on list endpoints | Out-of-memory on large datasets | Always paginate `findAll()` |
| Enum stored as ordinal | DB has meaningless numbers (0,1,2) | Use `@Enumerated(EnumType.STRING)` |
| String concat in log calls | Performance overhead even when log level is off | Use `{}` placeholders |
| Missing `@SpringBootApplication` | App fails to start | Check main class annotation |
| Classes outside root package | Beans not found by `@ComponentScan` | Move inside the root package |

---

## Summary: The Mental Model

```
When a request hits your app, think of it as going through 5 gates:

Gate 1 — CORS Filter
  "Is this request from an allowed origin?"

Gate 2 — JWT Filter (JwtAuthFilter)
  "Does this request have a valid token?"
  "Who is this person?"

Gate 3 — Spring Security Authorization
  "Is this person ALLOWED to access this URL?"
  (based on rules in SecurityConfig)

Gate 4 — Controller
  "Parse the request, validate it, call the right service method"

Gate 5 — Service
  "Apply business rules, use repository, return DTO"

At every gate, if something is wrong:
  - Gate 1 failure → CORS error in browser
  - Gate 2 failure → 401 Unauthorized
  - Gate 3 failure → 403 Forbidden
  - Gate 4 failure → 400 Bad Request (validation) or 404/409 (business)
  - Gate 5 failure → 500 Internal Server Error (unexpected)
```

---

*tech.csm Spring Boot Training · Complete Concept Notes*
