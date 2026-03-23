# 💻 Part 10: .NET, C/C++, OS & SDLC (Q201–Q210)

---

## Q201. ASP.NET, ADO.NET, and Entity Framework

**ASP.NET:** Microsoft's web framework for building web applications and APIs.
- **ASP.NET MVC** — Model-View-Controller pattern (similar to Spring MVC)
- **ASP.NET Web API** — RESTful services (similar to Spring REST)
- **ASP.NET Core** — Cross-platform, modern version

**ADO.NET:** Low-level data access technology — equivalent of JDBC in .NET.
```csharp
// ADO.NET (like JDBC)
SqlConnection conn = new SqlConnection(connectionString);
conn.Open();
SqlCommand cmd = new SqlCommand("SELECT * FROM Users WHERE Id=@id", conn);
cmd.Parameters.AddWithValue("@id", userId);
SqlDataReader reader = cmd.ExecuteReader();
while (reader.Read()) {
    Console.WriteLine(reader["Name"]);
}
conn.Close();
```

**Entity Framework (EF):** ORM for .NET — equivalent of Hibernate in Java.
```csharp
// Entity Framework (like Hibernate)
public class User {
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

// DbContext (like SessionFactory)
using var context = new AppDbContext();
var users = context.Users.Where(u => u.Name == "Alice").ToList();  // LINQ query
context.Users.Add(new User { Name = "Bob", Email = "bob@test.com" });
context.SaveChanges();
```

| Java Equivalent | .NET Equivalent |
|----------------|-----------------|
| Spring Boot | ASP.NET Core |
| JDBC | ADO.NET |
| Hibernate/JPA | Entity Framework |
| Maven | NuGet |
| Tomcat | IIS / Kestrel |

---

## Q202. Java vs .NET & Architecture Types

| Feature | Java | .NET |
|---------|------|------|
| Language | Java | C#, VB.NET, F# |
| Runtime | JVM | CLR (Common Language Runtime) |
| Bytecode | .class (bytecode) | .dll (IL — Intermediate Language) |
| Platform | Cross-platform | Historically Windows; .NET Core is cross-platform |
| Open Source | Yes | .NET Core is open source |
| Web Framework | Spring Boot | ASP.NET Core |
| ORM | Hibernate | Entity Framework |
| Build Tool | Maven/Gradle | MSBuild/NuGet |
| IDE | IntelliJ/Eclipse | Visual Studio |

**Architecture Types:**
- **Monolithic** — Single deployable unit (traditional)
- **Microservices** — Independent services communicating via APIs
- **SOA** — Service-Oriented Architecture (SOAP/ESB)
- **Serverless** — FaaS (AWS Lambda, Azure Functions)
- **Event-Driven** — Components communicate via events (Kafka, RabbitMQ)

---

## Q203. CLR and CLS

**CLR (Common Language Runtime):** .NET's equivalent of JVM.
- Manages memory (Garbage Collection)
- Converts IL (Intermediate Language) to native code via JIT
- Provides exception handling, thread management, security

```
C# / VB.NET / F# → Compiler → IL (Intermediate Language) → CLR → JIT → Native Code
                                (like Java bytecode)         (like JVM)
```

**CLS (Common Language Specification):** Set of rules that all .NET languages must follow for interoperability. E.g., all public types must be CLS-compliant so C# code can call VB.NET libraries.

---

## Q204. WCF Pseudocode

**WCF (Windows Communication Foundation):** Framework for building service-oriented applications.

```csharp
// 1. Define Service Contract (Interface)
[ServiceContract]
public interface ICalculatorService {
    [OperationContract]
    int Add(int a, int b);
    
    [OperationContract]
    int Subtract(int a, int b);
}

// 2. Implement Service
public class CalculatorService : ICalculatorService {
    public int Add(int a, int b) { return a + b; }
    public int Subtract(int a, int b) { return a - b; }
}

// 3. Host Service
ServiceHost host = new ServiceHost(typeof(CalculatorService));
host.Open();
Console.WriteLine("Service running...");

// 4. Client Consumption
CalculatorServiceClient client = new CalculatorServiceClient();
int result = client.Add(5, 3);  // calls remote service
```

> "WCF is .NET's equivalent of Java RMI/EJB. Modern replacement is ASP.NET Core Web API + gRPC."

---

## Q205. Garbage Collector Generations

The GC organizes objects by **age** into generations for efficient collection:

```
┌──────────────────────────────────────────────┐
│ Generation 0 (Gen 0) — Short-lived objects   │  ← Most GC runs here
│ (newly created, small, temporary)            │     (fastest, most frequent)
├──────────────────────────────────────────────┤
│ Generation 1 (Gen 1) — Medium-lived objects  │  ← Buffer between Gen0 & Gen2
│ (survived Gen 0 collection)                  │
├──────────────────────────────────────────────┤
│ Generation 2 (Gen 2) — Long-lived objects    │  ← Full GC (expensive, rare)
│ (static data, caches, survived Gen 1)        │
└──────────────────────────────────────────────┘
```

**Process:**
1. New object → Gen 0
2. Gen 0 full → GC runs on Gen 0 → survivors promoted to Gen 1
3. Gen 1 full → GC runs on Gen 0 + Gen 1 → survivors → Gen 2
4. Gen 2 full → Full GC (all generations) — expensive!

**Java Equivalent:** Young Generation (Eden + Survivor) = Gen 0+1, Old Generation = Gen 2.

---

## Q206. Structure vs Union, Class vs Structure

**Structure vs Union (C/C++):**

| Feature | Structure | Union |
|---------|----------|-------|
| Memory | Sum of ALL members | Size of LARGEST member |
| Access | All members at once | Only ONE member at a time |
| Memory sharing | No | Yes (shared memory) |

```c
struct Employee {      // 4 + 20 = 24 bytes (approx, with padding)
    int id;            // 4 bytes
    char name[20];     // 20 bytes
};

union Data {           // 20 bytes (size of largest member)
    int id;            // 4 bytes  ─┐
    char name[20];     // 20 bytes ─┘ shared memory!
};
```

**Class vs Structure (C#/.NET):**

| Feature | Class | Struct |
|---------|-------|--------|
| Type | Reference type (Heap) | Value type (Stack) |
| Default | null | default values |
| Inheritance | ✅ Yes | ❌ No |
| Garbage Collected | Yes | No (stack-managed) |
| Use for | Complex, large objects | Small, lightweight data (Point, Color) |

---

## Q207. Thread vs Process

| Feature | Process | Thread |
|---------|---------|--------|
| Definition | Independent program in execution | Smallest unit of execution within a process |
| Memory | Own memory space (isolated) | Shares memory with parent process |
| Creation | Heavyweight (slow) | Lightweight (fast) |
| Communication | IPC (pipes, sockets, shared memory) | Direct — shared variables |
| Crash impact | Process crash doesn't affect others | Thread crash can crash the process |
| Context switch | Expensive | Cheap |

```
Process (Chrome Browser)
├── Thread 1: UI rendering
├── Thread 2: Network requests
├── Thread 3: JavaScript execution
├── Thread 4: Audio/Video playback
└── Thread 5: Garbage collection

Each Chrome TAB = separate Process (isolation for security/stability)
Threads within a tab share memory (faster communication)
```

---

## Q208. Scheduling Algorithms & Booting Process

**CPU Scheduling Algorithms:**

| Algorithm | Type | Description |
|-----------|------|-------------|
| **FCFS** | Non-preemptive | First Come First Served — simple, can cause convoy effect |
| **SJF** | Non-preemptive | Shortest Job First — optimal avg wait, starvation possible |
| **SRTF** | Preemptive | Shortest Remaining Time First — preemptive SJF |
| **Round Robin** | Preemptive | Time quantum per process — fair, used in time-sharing OS |
| **Priority** | Both | Higher priority runs first — starvation solved by aging |
| **Multilevel Queue** | Both | Multiple queues with different algorithms |

**Booting Process:**
```
1. Power ON → BIOS/UEFI (Basic Input/Output System)
2. POST (Power-On Self Test) — checks hardware (RAM, CPU, disk)
3. BIOS finds bootable device (HDD/SSD/USB)
4. MBR (Master Boot Record) / GPT loaded — first 512 bytes of disk
5. Boot Loader (GRUB/Windows Boot Manager) loaded
6. Operating System Kernel loaded into RAM
7. Kernel initializes drivers, file systems, memory management
8. Init/SystemD starts system services
9. Login screen / Desktop environment displayed
```

---

## Q209. SDLC Life Cycle & Phase Worked On

**SDLC (Software Development Life Cycle):**
```
1. Requirement Analysis → Gather & document requirements (BRD, SRS)
2. System Design         → Architecture, DB design, tech stack selection
3. Implementation        → Actual coding (this is where developers work)
4. Testing               → Unit, Integration, System, UAT testing
5. Deployment            → Release to production (CI/CD pipeline)
6. Maintenance           → Bug fixes, enhancements, monitoring
```

**SDLC Models:**
| Model | Best For |
|-------|----------|
| **Waterfall** | Fixed requirements, small projects |
| **Agile (Scrum)** | Changing requirements, iterative development |
| **V-Model** | High-reliability systems (medical, aerospace) |
| **Spiral** | Large, complex projects with risk analysis |
| **DevOps** | Continuous delivery, modern web apps |

**Sample Answer:**
> "I worked primarily in the **Implementation and Testing phases** using **Agile Scrum** methodology. We had 2-week sprints, daily standups, sprint planning, and retrospectives. I was involved in coding backend APIs, writing unit tests (JUnit/Mockito), and participating in code reviews. I also contributed to the system design phase for the module I owned."

---

## Q210. How to Deploy and Maintain a Project?

**Deployment Process (Modern CI/CD Pipeline):**
```
Developer pushes code → GitHub/GitLab
    ↓
CI Pipeline (Jenkins/GitHub Actions):
    1. Build (Maven: mvn clean package)
    2. Run unit tests (mvn test)
    3. Code quality check (SonarQube)
    4. Build Docker image
    5. Push image to Container Registry
    ↓
CD Pipeline:
    6. Deploy to Staging environment
    7. Run integration/smoke tests
    8. Manual approval (for production)
    9. Deploy to Production (Kubernetes/AWS ECS)
    10. Health check verification
    ↓
Monitoring & Maintenance:
    - Logs: ELK Stack (Elasticsearch + Logstash + Kibana)
    - Metrics: Prometheus + Grafana
    - Alerts: PagerDuty / Slack notifications
    - APM: New Relic / Datadog (performance monitoring)
```

**Maintenance Activities:**
- **Bug fixes** — triage, reproduce, fix, test, deploy
- **Performance tuning** — query optimization, caching, load testing
- **Security patches** — dependency updates, vulnerability scanning
- **Feature enhancements** — new requirements from stakeholders
- **Database maintenance** — backups, index optimization, schema migrations
- **Scaling** — horizontal scaling based on traffic patterns

```yaml
# Sample GitHub Actions CI/CD pipeline
name: CI/CD Pipeline
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-java@v3
        with: { java-version: '17' }
      - run: mvn clean test
      - run: mvn package -DskipTests
      - run: docker build -t myapp:latest .
      - run: docker push myregistry/myapp:latest
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: kubectl apply -f k8s/deployment.yaml
```

> "In my project, we used Jenkins for CI/CD. Every push to `main` triggered a pipeline that built the JAR, ran tests, created a Docker image, and deployed to AWS EC2. We monitored using CloudWatch and received Slack alerts for errors."
