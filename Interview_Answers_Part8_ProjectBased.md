# 📁 Part 8: Project-Based Questions (Q151–Q168)

---

## Q151. Explain Architecture of Your Project

> **Tailor this to YOUR project. Here's a template for a Spring Boot + React full-stack project:**

```
                    ┌──────────────────────────────────────────┐
                    │          CLIENT LAYER (React.js)         │
                    │  Components → Pages → Axios HTTP Calls   │
                    └──────────────────┬───────────────────────┘
                                       │ REST API (JSON)
                    ┌──────────────────▼───────────────────────┐
                    │       API GATEWAY / LOAD BALANCER        │
                    │            (Nginx / AWS ALB)              │
                    └──────────────────┬───────────────────────┘
                                       │
                    ┌──────────────────▼───────────────────────┐
                    │     BACKEND (Spring Boot Application)     │
                    │                                           │
                    │  Controller → Service → Repository → DB  │
                    │       ↕            ↕          ↕          │
                    │  Validation    Business    JPA/Hibernate  │
                    │  DTO Mapping   Logic       SQL Generation │
                    │  Auth Filter   Caching                    │
                    └──────────────────┬───────────────────────┘
                                       │
                    ┌──────────────────▼───────────────────────┐
                    │         DATABASE (MySQL/PostgreSQL)       │
                    │  Users | Orders | Products | Payments     │
                    └──────────────────────────────────────────┘
```

**How to explain:**
> "Our project follows a **3-tier architecture** — Presentation (React), Business Logic (Spring Boot), and Data (MySQL). The frontend communicates with the backend via RESTful APIs. We use JWT for authentication, Spring Security for authorization, and JPA/Hibernate for ORM. The backend follows the Controller-Service-Repository pattern for clean separation of concerns."

---

## Q152. How Many APIs You Created?

> **Sample answer:**

"I created **15+ REST APIs** in the project, grouped by modules:

| Module | APIs | Methods |
|--------|------|---------|
| Authentication | Login, Register, Refresh Token | POST, POST, POST |
| User Management | Get Profile, Update Profile, Delete Account | GET, PUT, DELETE |
| Products | List, Search, Get by ID, Create, Update, Delete | GET, GET, GET, POST, PUT, DELETE |
| Orders | Place Order, Get Orders, Cancel Order | POST, GET, PUT |
| Payments | Initiate Payment, Verify Payment, Get Status | POST, POST, GET |

Each API follows REST conventions — proper HTTP methods, status codes, and JSON request/response bodies."

---

## Q153. What is Request-Response Flow?

```
User clicks "Place Order" button
    ↓
Frontend (React) sends:
    POST /api/orders
    Headers: { Authorization: "Bearer <JWT>", Content-Type: "application/json" }
    Body: { "productId": 5, "quantity": 2, "addressId": 10 }
    ↓
Backend receives request:
    1. JwtAuthFilter → validates JWT token, extracts userId
    2. DispatcherServlet → routes to OrderController
    3. OrderController.placeOrder(@RequestBody OrderDTO)
       → validates input (@Valid)
       → calls OrderService.createOrder(userId, dto)
    4. OrderService:
       → checks product stock (ProductRepository)
       → calculates total price
       → creates Order entity
       → saves via OrderRepository.save(order)
       → triggers PaymentService.initiatePayment()
       → publishes OrderEvent (async notification)
    5. OrderRepository → JPA → Hibernate generates:
       INSERT INTO orders (user_id, product_id, qty, total, status) VALUES (...)
    6. Response flows back:
       OrderController returns ResponseEntity<OrderResponse>(201, orderResponse)
    ↓
Frontend receives:
    HTTP 201 Created
    { "orderId": 1234, "status": "PENDING", "total": 5000.00 }
    → updates UI → shows success message
```

---

## Q154. What DB Tables You Designed?

```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,  -- BCrypt hashed
    role ENUM('USER', 'ADMIN') DEFAULT 'USER',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Products table
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10,2) NOT NULL,
    stock INT DEFAULT 0,
    category_id BIGINT REFERENCES categories(id)
);

-- Orders table
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT REFERENCES users(id),
    total DECIMAL(10,2) NOT NULL,
    status ENUM('PENDING','CONFIRMED','SHIPPED','DELIVERED','CANCELLED'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Order Items (Many-to-Many between Orders and Products)
CREATE TABLE order_items (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT REFERENCES orders(id) ON DELETE CASCADE,
    product_id BIGINT REFERENCES products(id),
    quantity INT NOT NULL,
    price DECIMAL(10,2) NOT NULL
);
```

---

## Q155. How You Handled Errors?

```java
// 1. Global Exception Handler (@ControllerAdvice)
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(
            new ErrorResponse(404, ex.getMessage(), LocalDateTime.now()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        String errors = ex.getBindingResult().getFieldErrors().stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .collect(Collectors.joining(", "));
        return ResponseEntity.badRequest().body(new ErrorResponse(400, errors, LocalDateTime.now()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        return ResponseEntity.status(500).body(
            new ErrorResponse(500, "Internal server error", LocalDateTime.now()));
    }
}

// 2. Custom Exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, Long id) {
        super(resource + " not found with id: " + id);
    }
}

// 3. Service layer throws meaningful exceptions
public Order getOrder(Long id) {
    return orderRepo.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("Order", id));
}
```

---

## Q156. Any Optimization You Did?

> "Yes, I did several optimizations:

1. **N+1 Query Problem** — Used `@EntityGraph` and `JOIN FETCH` in JPQL to load related entities in one query instead of N+1 separate queries.

2. **Database Indexing** — Added indexes on frequently searched columns (`email`, `product_name`, `order_status`) — reduced query time by 60%.

3. **Pagination** — Instead of loading all records, used `Pageable` in Spring Data:
```java
Page<Product> products = productRepo.findAll(PageRequest.of(0, 20, Sort.by("price")));
```

4. **Caching** — Used `@Cacheable` for product catalog (rarely changes):
```java
@Cacheable("products")
public List<Product> getAllProducts() { return productRepo.findAll(); }
```

5. **Lazy Loading** — Changed `@ManyToOne(fetch = FetchType.LAZY)` to avoid loading unnecessary related data.

6. **Connection Pooling** — Configured HikariCP with optimal pool size."

---

## Q157-Q158. What If API Fails? What Improvements?

**If API Fails:**
- **Retry mechanism** with exponential backoff for transient failures
- **Circuit Breaker** (Resilience4j) — stops calling failing service, returns fallback
- **Graceful error responses** — 500 with meaningful message, not stack traces
- **Logging & Monitoring** — log exceptions with correlation IDs, alerting via ELK/Grafana
- **Fallback** — return cached data or default response
- **Dead Letter Queue** — for async operations, failed messages go to DLQ for replay

**Possible Improvements:**
- Add **Redis cache** for session management and frequently accessed data
- Implement **message queue** (RabbitMQ/Kafka) for async processing
- Add **API rate limiting** to prevent abuse
- Implement **CQRS** — separate read/write models for performance
- Add **comprehensive test coverage** (unit + integration + e2e)
- **Containerize** with Docker + orchestrate with Kubernetes

---

## Q159-Q164. Project Architecture Step-by-Step, Schema, Tech Stack Choices, Toughest Bug

**Why this tech stack?**
> "Spring Boot — rapid development, auto-config, embedded Tomcat, massive ecosystem. React — component-based UI, virtual DOM performance, large community. MySQL — ACID compliant, reliable, great for transactional data. JWT — stateless auth, works with microservices."

**Toughest bug:**
> "We had a **race condition** in the order placement flow. Two users could order the last item simultaneously — both saw stock=1, both placed orders. Fixed it using **database-level pessimistic locking**: `SELECT ... FOR UPDATE` on the product row. Also added application-level check with `@Version` for optimistic locking."

**How I debugged it:**
> "First, I couldn't reproduce it locally. Added detailed logging with timestamps. Then used **JMeter** to simulate concurrent requests. Saw interleaved logs showing both threads reading same stock value. Root cause: missing transaction isolation. Fixed with `@Transactional(isolation = Isolation.SERIALIZABLE)` on the order method."

---

## Q165-Q168. Scaling, Database Down Scenarios

**How to Scale for 1 Lakh Users?**
```
1. Horizontal Scaling    → Multiple server instances behind load balancer
2. Database Replication  → Master (writes) + Read Replicas (reads)
3. Caching Layer         → Redis for sessions, product catalog, frequently accessed data
4. CDN                   → Static assets (images, CSS, JS) served from CDN edge servers
5. Message Queue         → Async processing (order confirmation emails, notifications)
6. Database Sharding     → Split data across multiple DBs by user region/ID range
7. Auto-scaling          → AWS Auto Scaling Groups — add servers based on CPU/traffic
8. API Gateway           → Rate limiting, request routing, caching at gateway level
9. Microservices         → Split monolith into independent services (User, Order, Payment)
10. Connection Pooling   → Optimize DB connections (HikariCP max-pool-size tuning)
```

**What If Database Goes Down?**
- **Read Replica failover** — promote replica to master
- **Database backups** — restore from latest backup (daily automated)
- **Circuit breaker** — return cached responses, queue writes for retry
- **Multi-AZ deployment** — AWS RDS Multi-AZ automatic failover
- **Graceful degradation** — show "Service temporarily unavailable" instead of crash
- **Health checks** — monitoring + alerts (PagerDuty/Datadog) for immediate response
