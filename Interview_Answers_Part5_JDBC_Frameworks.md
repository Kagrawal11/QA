# 🔌 Part 5: JDBC & Backend Frameworks (Q104–Q122)

---

## Q104. JDBC, Types of Drivers, Most Used

**JDBC (Java Database Connectivity)** — Java API for connecting to relational databases.

**4 Types of JDBC Drivers:**

| Type | Name | Mechanism | Speed |
|------|------|-----------|-------|
| Type 1 | JDBC-ODBC Bridge | Java → ODBC → DB | Slowest |
| Type 2 | Native-API | Java → Native lib → DB | Fast, platform-dependent |
| Type 3 | Network Protocol | Java → Middleware → DB | Flexible |
| **Type 4** | **Thin Driver** | **Java → DB directly** | **Fastest, most used** |

> "Type 4 (Thin) is used 99% of the time — `mysql-connector-java`, `ojdbc`, `postgresql` are all Type 4. Pure Java, no native libraries, platform-independent."

---

## Q105. JDBC Code — Connection, CRUD Operations

```java
import java.sql.*;

public class JDBCDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/mydb";
        String user = "root";
        String pass = "password";

        // 1. Load driver (optional since JDBC 4.0 — auto-loaded)
        // Class.forName("com.mysql.cj.jdbc.Driver");

        // 2. Get Connection
        try (Connection conn = DriverManager.getConnection(url, user, pass)) {

            // CREATE
            PreparedStatement ps = conn.prepareStatement(
                "INSERT INTO employees (name, salary) VALUES (?, ?)");
            ps.setString(1, "John");
            ps.setDouble(2, 75000);
            ps.executeUpdate();

            // READ
            PreparedStatement read = conn.prepareStatement("SELECT * FROM employees WHERE id = ?");
            read.setInt(1, 1);
            ResultSet rs = read.executeQuery();
            while (rs.next()) {
                System.out.println(rs.getString("name") + " → " + rs.getDouble("salary"));
            }

            // UPDATE
            PreparedStatement update = conn.prepareStatement(
                "UPDATE employees SET salary = ? WHERE name = ?");
            update.setDouble(1, 80000);
            update.setString(2, "John");
            update.executeUpdate();

            // DELETE
            PreparedStatement delete = conn.prepareStatement("DELETE FROM employees WHERE id = ?");
            delete.setInt(1, 1);
            delete.executeUpdate();

        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

---

## Q106. Statement vs PreparedStatement vs CallableStatement

| Feature | Statement | PreparedStatement | CallableStatement |
|---------|----------|-------------------|-------------------|
| Use | Simple, static SQL | Parameterized queries | Stored procedures |
| SQL Injection | ❌ Vulnerable | ✅ Safe (parameterized) | ✅ Safe |
| Precompiled | No | **Yes** (faster for repeated) | Yes |
| Parameters | No (`+` concatenation) | `?` placeholders | `?` (IN/OUT params) |

```java
// Statement — AVOID in production
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM users WHERE id=" + userId); // SQL injection risk!

// PreparedStatement — ALWAYS USE
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE id=?");
ps.setInt(1, userId);  // safe
ResultSet rs = ps.executeQuery();

// CallableStatement — for stored procedures
CallableStatement cs = conn.prepareCall("{call get_salary(?, ?)}");
cs.setInt(1, empId);
cs.registerOutParameter(2, Types.DOUBLE);
cs.execute();
double salary = cs.getDouble(2);
```

---

## Q107. JDBC vs ODBC, JDBC vs Hibernate

| JDBC | ODBC |
|------|------|
| Java-specific | Language-independent (C-based) |
| Platform-independent | Platform-dependent |
| Object-oriented | Procedural |

| JDBC | Hibernate |
|------|-----------|
| Write SQL manually | Generates SQL from objects |
| Low-level, verbose | High-level ORM, less code |
| No caching | L1 + L2 cache built-in |
| Result → manual mapping | Auto maps to POJOs |
| Database-dependent SQL | HQL — database-independent |

---

## Q108. Hibernate, JPA, and Annotations

**JPA** = specification (interface). **Hibernate** = implementation (vendor).

```java
@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "emp_name", nullable = false)
    private String name;

    @Column
    private double salary;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "dept_id")
    private Department department;

    @OneToMany(mappedBy = "employee", cascade = CascadeType.ALL)
    private List<Address> addresses;
}
```

**Key Annotations:** `@Entity`, `@Table`, `@Id`, `@GeneratedValue`, `@Column`, `@OneToMany`, `@ManyToOne`, `@ManyToMany`, `@Transient`, `@Temporal`.

---

## Q109. SessionFactory, hbm, Hibernate Template

| Component | Role |
|-----------|------|
| **SessionFactory** | Heavyweight factory — creates Session objects. ONE per database. Thread-safe. |
| **Session** | Lightweight — represents one unit of work. NOT thread-safe. |
| **hbm files** | XML mapping files (e.g., `Employee.hbm.xml`) — maps class to table. Now replaced by annotations. |
| **HibernateTemplate** | Spring helper that simplifies Session management — handles open/close/exception. |

```java
// SessionFactory creation
SessionFactory sf = new Configuration().configure("hibernate.cfg.xml")
                                       .addAnnotatedClass(Employee.class)
                                       .buildSessionFactory();

// Using Session
Session session = sf.openSession();
Transaction tx = session.beginTransaction();
session.save(new Employee("Alice", 70000));  // INSERT
Employee emp = session.get(Employee.class, 1L);  // SELECT
tx.commit();
session.close();
```

---

## Q110. Spring MVC Flow Diagram & Spring Beans

```
Client Request → DispatcherServlet → HandlerMapping → Controller
                                                          ↓
                                                     Service Layer
                                                          ↓
                                                     DAO / Repository
                                                          ↓
                                                       Database
                                                          ↑
           ViewResolver ← Model ← Controller ← Service ←─┘
                ↓
           JSP/Thymeleaf → Response to Client
```

**Spring Beans** = objects managed by Spring IoC container. Configured via annotations or XML.

```java
@Controller
public class UserController {
    @Autowired
    private UserService userService;  // Spring injects the bean

    @GetMapping("/users")
    public String getUsers(Model model) {
        model.addAttribute("users", userService.getAllUsers());
        return "userList";  // ViewResolver maps to userList.jsp
    }
}

@Service
public class UserService {
    @Autowired
    private UserRepository userRepo;

    public List<User> getAllUsers() { return userRepo.findAll(); }
}
```

---

## Q111. Spring vs Spring Boot

| Spring | Spring Boot |
|--------|-------------|
| Manual configuration (XML/Java) | Auto-configuration |
| Need to set up server (Tomcat) | Embedded server included |
| Complex dependency setup | Starter dependencies |
| No opinionated defaults | Convention over configuration |
| `web.xml`, `dispatcher-servlet.xml` | `application.properties` only |
| WAR deployment | JAR with embedded server |

```java
// Spring Boot — entire app in one class!
@SpringBootApplication  // = @Configuration + @ComponentScan + @EnableAutoConfiguration
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

---

## Q112. Dependency Injection

**DI** = pattern where objects receive dependencies from outside rather than creating them.

```java
// WITHOUT DI — tightly coupled
class OrderService {
    private PaymentGateway pg = new StripePayment();  // hardcoded
}

// WITH DI — loosely coupled
@Service
class OrderService {
    private final PaymentGateway pg;

    @Autowired
    OrderService(PaymentGateway pg) { this.pg = pg; }  // injected by Spring
}
```

**Types:** Constructor Injection (recommended), Setter Injection, Field Injection (`@Autowired`, least preferred).

---

## Q113. Maven

**Maven** = build automation + dependency management tool.

```xml
<!-- pom.xml — Project Object Model -->
<project>
    <groupId>com.example</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0.0</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>3.2.0</version>
        </dependency>
    </dependencies>
</project>
```

**Lifecycle:** `validate → compile → test → package → verify → install → deploy`

---

## Q114. RESTful API

**REST** = Representational State Transfer — architectural style for networked apps.

**Principles:** Stateless, Client-Server, Cacheable, Uniform Interface, Layered System.

```java
@RestController
@RequestMapping("/api/employees")
public class EmployeeController {

    @GetMapping              // GET    /api/employees
    public List<Employee> getAll() { return service.findAll(); }

    @GetMapping("/{id}")     // GET    /api/employees/1
    public Employee getById(@PathVariable Long id) { return service.findById(id); }

    @PostMapping             // POST   /api/employees
    public Employee create(@RequestBody Employee emp) { return service.save(emp); }

    @PutMapping("/{id}")     // PUT    /api/employees/1
    public Employee update(@PathVariable Long id, @RequestBody Employee emp) { ... }

    @DeleteMapping("/{id}")  // DELETE /api/employees/1
    public void delete(@PathVariable Long id) { service.delete(id); }
}
```

| HTTP Method | Operation | Idempotent |
|------------|-----------|-----------|
| GET | Read | Yes |
| POST | Create | No |
| PUT | Update (full) | Yes |
| PATCH | Update (partial) | No |
| DELETE | Delete | Yes |

---

## Q115. DAO and CRUD

**DAO (Data Access Object)** = design pattern that separates data access logic from business logic.

```java
// DAO Interface
public interface EmployeeDAO {
    Employee findById(Long id);           // READ
    List<Employee> findAll();             // READ
    void save(Employee emp);              // CREATE
    void update(Employee emp);            // UPDATE
    void delete(Long id);                 // DELETE
}

// DAO Implementation
@Repository
public class EmployeeDAOImpl implements EmployeeDAO {
    @Autowired private JdbcTemplate jdbc;

    public Employee findById(Long id) {
        return jdbc.queryForObject("SELECT * FROM employees WHERE id=?",
            new BeanPropertyRowMapper<>(Employee.class), id);
    }
    // ... other CRUD methods
}
```

---

## Q116. Spring MVC vs Struts

| Spring MVC | Struts |
|-----------|--------|
| Flexible, modular | Monolithic |
| POJO-based controllers | Action classes (coupled to framework) |
| Annotation-driven | XML-heavy configuration |
| Easy testing (mockable) | Hard to test |
| Active community | Declining, legacy |
| Integrates with anything | Harder integration |

---

## Q117. Struts Flow & Struts-Spring-Hibernate

```
Request → web.xml → FilterDispatcher → struts.xml → Action Class
             → Business Logic → JSP (View) → Response
```

**Integration:** Struts handles web layer, Spring manages beans + DI, Hibernate handles DB.

---

## Q118-Q122. Connection Pooling, Batch Processing, Transaction Management, Auto-commit, DAO Pattern

**Connection Pooling:** Pre-created pool of DB connections reused across requests.
```properties
# application.properties (HikariCP — default in Spring Boot)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
```

**Batch Processing:**
```java
PreparedStatement ps = conn.prepareStatement("INSERT INTO logs VALUES (?, ?)");
for (int i = 0; i < 1000; i++) {
    ps.setInt(1, i);
    ps.setString(2, "Log " + i);
    ps.addBatch();
    if (i % 100 == 0) ps.executeBatch();  // flush every 100
}
ps.executeBatch();
```

**Transaction Management:**
```java
@Transactional  // Spring manages begin/commit/rollback
public void transferMoney(Long from, Long to, double amount) {
    accountRepo.debit(from, amount);
    accountRepo.credit(to, amount);
    // If exception → auto rollback
}
```

**Auto-commit:** By default, JDBC auto-commits every statement. Disable for transactions:
```java
conn.setAutoCommit(false);
// execute multiple statements
conn.commit();   // or conn.rollback();
```

**DAO Pattern:** Abstracts and encapsulates all data access. Separates persistence logic from business logic. Allows swapping DB implementation without changing business code.
