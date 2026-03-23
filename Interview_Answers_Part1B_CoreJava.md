# 📘 Part 1B: Core Java & OOPs (Q24–Q46)

---

## Q24. Set vs List, Map/HashMap vs HashTable

**Set vs List:**

| List | Set |
|------|-----|
| Ordered (insertion order) | Unordered (except LinkedHashSet, TreeSet) |
| Allows duplicates | NO duplicates |
| Index-based access `get(i)` | No index-based access |
| `ArrayList, LinkedList` | `HashSet, TreeSet, LinkedHashSet` |

**HashMap vs HashTable:**

| HashMap | HashTable |
|---------|-----------|
| NOT synchronized | Synchronized |
| Allows **one null key**, multiple null values | **NO null** key or value |
| Faster | Slower |
| `Iterator` (fail-fast) | `Enumerator` (fail-safe) |
| Java 1.2 | Legacy Java 1.0 |

> "In modern Java, replace HashTable with `ConcurrentHashMap` — better concurrency with segment-level locking."

---

## Q25. Serialization with code

**Serialization** = converting an object's state to a **byte stream** (for storage/transmission).
**Deserialization** = reconstructing the object from byte stream.

```java
import java.io.*;

class Employee implements Serializable {
    private static final long serialVersionUID = 1L;  // version control
    String name;
    transient String password;  // excluded from serialization
    int age;

    Employee(String n, String p, int a) { name=n; password=p; age=a; }
}

// SERIALIZE
Employee emp = new Employee("John", "secret123", 30);
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("emp.ser"));
oos.writeObject(emp);
oos.close();

// DESERIALIZE
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("emp.ser"));
Employee e = (Employee) ois.readObject();
ois.close();
System.out.println(e.name);      // "John"
System.out.println(e.password);  // null (transient)
```

**Real-world use:** Saving session data, sending objects over network (RMI), caching, Hibernate second-level cache.

---

## Q26. What are Java Beans?

A **JavaBean** is a reusable component class that follows conventions:
1. **Private fields** with public **getters/setters**
2. **No-arg constructor**
3. Implements `Serializable`

```java
public class StudentBean implements Serializable {
    private String name;
    private int rollNo;

    public StudentBean() {}  // no-arg constructor

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public int getRollNo() { return rollNo; }
    public void setRollNo(int rollNo) { this.rollNo = rollNo; }
}
```

> "JavaBeans are the foundation of Spring beans, JPA entities, and JSP's `<jsp:useBean>`. They enable frameworks to manipulate objects via reflection."

---

## Q27. Design Patterns & Singleton Class

**Design Pattern:** Reusable solution to a commonly occurring problem in software design.

**Categories:** Creational, Structural, Behavioral

**Singleton Pattern** — Only ONE instance of a class exists throughout the application.

```java
// Thread-safe Singleton (Bill Pugh's approach — best practice)
public class DatabaseConnection {
    private DatabaseConnection() {}  // private constructor

    private static class Holder {
        private static final DatabaseConnection INSTANCE = new DatabaseConnection();
    }

    public static DatabaseConnection getInstance() {
        return Holder.INSTANCE;  // loaded lazily, thread-safe
    }
}

// Usage
DatabaseConnection db1 = DatabaseConnection.getInstance();
DatabaseConnection db2 = DatabaseConnection.getInstance();
System.out.println(db1 == db2);  // true — same instance
```

**Ways to break Singleton:** Reflection, Serialization, Cloning
**Prevention:** Use `enum` Singleton (recommended by Joshua Bloch):
```java
public enum Singleton {
    INSTANCE;
    public void doSomething() { /* ... */ }
}
```

---

## Q28. What is JOC?

**JOC = Java Optimizer Compiler** — part of JVM that optimizes bytecode during execution.

The **JIT (Just-In-Time) Compiler** is the main JOC component:
- Converts frequently executed bytecode ("hot spots") into native machine code
- Uses **method inlining, loop unrolling, dead code elimination**
- The HotSpot JVM uses **C1 (client)** and **C2 (server)** compilers

```
Bytecode → Interpreter (slow) → JIT Compiler detects hot code → Native code (fast)
```

> "JIT compilation is why Java performance is competitive with C++ — frequently called methods get compiled to native code and cached."

---

## Q29. Explain EJB (Enterprise JavaBeans)

**EJB** = server-side component model for building large-scale, distributed enterprise applications.

**Types:**
| Type | Purpose |
|------|---------|
| **Session Bean** | Business logic (Stateful, Stateless, Singleton) |
| **Message-Driven Bean (MDB)** | Asynchronous messaging (JMS) |
| **Entity Bean** | Database persistence (replaced by JPA) |

```java
@Stateless
public class OrderService {
    @PersistenceContext
    private EntityManager em;

    public void placeOrder(Order order) {
        em.persist(order);
    }
}
```

**EJB Container provides:** Transaction management, Security, Concurrency, Remoting, Dependency Injection.

> "EJB was heavy in older versions. Modern alternatives include Spring Framework, which provides similar services with less complexity."

---

## Q30. Working of RMI (Remote Method Invocation)

**RMI** allows a Java object on one JVM to invoke methods on an object in another JVM.

```
Client JVM                          Server JVM
┌─────────┐   Network Call    ┌─────────────┐
│  Client  │ ──────────────→  │ Remote Object│
│  (Stub)  │ ←──────────────  │  (Skeleton)  │
└─────────┘   Response        └─────────────┘
       ↕                              ↕
   RMI Registry (rmiregistry) — name lookup
```

**Steps:**
1. Define remote interface extending `Remote`
2. Implement the interface extending `UnicastRemoteObject`
3. Register with `rmiregistry`
4. Client looks up remote object and calls methods

```java
// Remote Interface
public interface Calculator extends Remote {
    int add(int a, int b) throws RemoteException;
}

// Implementation
public class CalculatorImpl extends UnicastRemoteObject implements Calculator {
    public int add(int a, int b) { return a + b; }
}

// Server — bind
Naming.rebind("rmi://localhost/Calculator", new CalculatorImpl());

// Client — lookup & call
Calculator calc = (Calculator) Naming.lookup("rmi://localhost/Calculator");
System.out.println(calc.add(5, 3));  // 8 — called on remote JVM!
```

---

## Q31. Difference between RMI & EJB

| RMI | EJB |
|-----|-----|
| Low-level remoting framework | High-level enterprise component model |
| Manual transaction management | Container-managed transactions |
| No built-in security | Declarative security |
| Tightly coupled | Loosely coupled |
| Only Java-to-Java | Can integrate with other systems |
| No connection pooling | Built-in connection pooling |

> "RMI is like making a raw phone call. EJB is like using a corporate phone system with call forwarding, recording, and security — the container handles everything."

---

## Q32. What are JAR Files?

**JAR (Java ARchive)** = compressed package (ZIP format) containing `.class` files, metadata, and resources.

```bash
# Create JAR
jar cf myapp.jar -C classes/ .

# Create executable JAR
jar cfe myapp.jar com.Main -C classes/ .

# Run JAR
java -jar myapp.jar

# View contents
jar tf myapp.jar
```

**MANIFEST.MF** (inside META-INF/):
```
Main-Class: com.Main
Class-Path: lib/dependency.jar
```

**Types:** JAR (library), WAR (web app), EAR (enterprise app containing WARs + JARs)

---

## Q33. Can you write main method two times in one class?

**Yes — via overloading!** But only `public static void main(String[] args)` is the entry point.

```java
public class Test {
    public static void main(String[] args) {       // Entry point ✅
        System.out.println("String[] args version");
        main(10);  // calling overloaded main
    }

    public static void main(int x) {                // Overloaded ✅
        System.out.println("int version: " + x);
    }

    // public static void main(String[] args) {}   // ❌ Duplicate — compile error
}
```

> "You can overload main (different parameters), but you cannot have two main methods with the SAME signature."

---

## Q34. How to track how many objects are created?

Use a **static counter** — shared across all instances:

```java
public class MyClass {
    private static int objectCount = 0;

    public MyClass() {
        objectCount++;
        System.out.println("Object #" + objectCount + " created");
    }

    public static int getObjectCount() {
        return objectCount;
    }
}

// Usage
new MyClass();  // Object #1 created
new MyClass();  // Object #2 created
new MyClass();  // Object #3 created
System.out.println("Total: " + MyClass.getObjectCount());  // Total: 3
```

> "Static variable belongs to the class, so it's shared. Each constructor call increments it. In multi-threaded environments, use `AtomicInteger` instead."

---

## Q35. Write a function to compute average and how to call it

```java
public class MathUtils {

    // Method 1: Fixed parameters
    public static double average(int a, int b, int c) {
        return (a + b + c) / 3.0;
    }

    // Method 2: Varargs (variable arguments)
    public static double average(int... numbers) {
        if (numbers.length == 0) throw new IllegalArgumentException("No numbers");
        int sum = 0;
        for (int n : numbers) sum += n;
        return (double) sum / numbers.length;
    }

    // Method 3: Using Java 8 Streams
    public static double averageStream(int[] arr) {
        return java.util.Arrays.stream(arr).average().orElse(0.0);
    }

    public static void main(String[] args) {
        System.out.println(average(10, 20, 30));           // 20.0
        System.out.println(average(10, 20, 30, 40, 50));   // 30.0
        System.out.println(averageStream(new int[]{5,10})); // 7.5
    }
}
```

---

## Q36. What is ClassLoader? Types?

**ClassLoader** loads `.class` files into JVM memory. It follows **Delegation Hierarchy**.

```
Bootstrap ClassLoader (C/C++) — loads rt.jar (core Java API)
         ↑
Extension ClassLoader — loads ext/*.jar (javax.*)
         ↑
Application ClassLoader — loads classpath classes (your code)
         ↑
Custom ClassLoader — user-defined (e.g., hot-deploy servers)
```

**Principles:**
1. **Delegation:** Child asks parent first. Only loads if parent can't.
2. **Visibility:** Child can see parent's classes, not vice versa.
3. **Uniqueness:** A class is loaded only once per classloader.

```java
// Check which classloader loaded a class
System.out.println(String.class.getClassLoader());      // null (Bootstrap)
System.out.println(MyClass.class.getClassLoader());     // AppClassLoader
```

---

## Q37. Memory Model in Java

```
JVM Memory
├── Heap (shared across threads)
│   ├── Young Generation (Eden + Survivor S0/S1)  ← new objects
│   └── Old Generation (Tenured)                   ← long-lived objects
│
├── Stack (per thread)
│   ├── Local variables, method frames
│   └── Primitive values, references
│
├── Method Area / Metaspace (Java 8+)
│   ├── Class metadata, static variables
│   └── Constant pool
│
├── PC Register (per thread) — current instruction address
│
└── Native Method Stack (per thread) — for native (C/C++) methods
```

| Area | Stores | Thread Safety |
|------|--------|---------------|
| Stack | Local vars, method calls | Thread-private |
| Heap | Objects, arrays | Shared — needs synchronization |
| Metaspace | Class metadata | Shared |
| String Pool | Interned strings | Part of heap (Java 7+) |

---

## Q38. Pass by Value vs Reference in Java

**Java is ALWAYS pass by value.** But the "value" of an object variable is the **reference** (memory address).

```java
// Primitives — pure pass by value
void change(int x) { x = 100; }
int a = 5;
change(a);
System.out.println(a);  // 5 — unchanged!

// Objects — value of reference is copied
void modify(StringBuilder sb) {
    sb.append(" World");  // modifies same object ✅
}
void reassign(StringBuilder sb) {
    sb = new StringBuilder("New");  // points to NEW object, original unchanged
}

StringBuilder s = new StringBuilder("Hello");
modify(s);
System.out.println(s);  // "Hello World" — modified via shared reference

reassign(s);
System.out.println(s);  // "Hello World" — reassignment didn't affect original
```

> "Java copies the reference value (like copying a remote control). You can change the TV channel (modify object), but buying a new remote (reassigning) doesn't affect the original remote."

---

## Q39–Q40. Can we override static methods? Can we overload main?

**Can we override static methods? NO — it's method hiding.**
```java
class Parent { static void greet() { System.out.println("Parent"); } }
class Child extends Parent { static void greet() { System.out.println("Child"); } }

Parent p = new Child();
p.greet();  // "Parent" — NOT polymorphic! Resolved at compile time.
```

**Can we overload main? YES.**
```java
public class Test {
    public static void main(String[] args) { System.out.println("Entry"); }
    public static void main(int x) { System.out.println("Overloaded: " + x); }
    public static void main(String arg) { System.out.println("Single: " + arg); }
}
// Only main(String[] args) is the entry point. Others are regular static methods.
```

---

## Q41. What is a Marker Interface?

A **marker/tag interface** has **no methods or fields**. It signals the JVM/framework that a class has a special property.

**Examples:**
- `Serializable` — marks class as serializable
- `Cloneable` — marks class as cloneable
- `Remote` — marks class for RMI

```java
public interface Serializable {}  // empty!

class Student implements Serializable {  // JVM now knows to allow serialization
    String name;
}
```

> "Since Java 5, annotations (`@Entity`, `@Deprecated`) have largely replaced marker interfaces. But `Serializable` and `Cloneable` remain for backward compatibility."

---

## Q42. Cloneable Interface

```java
class Address implements Cloneable {
    String city;
    Address(String c) { city = c; }
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

Address a1 = new Address("Mumbai");
Address a2 = (Address) a1.clone();  // creates copy
a2.city = "Delhi";
System.out.println(a1.city);  // "Mumbai" — a1 unaffected (for primitives/immutables)
```

If you DON'T implement `Cloneable`, calling `clone()` throws `CloneNotSupportedException`.

---

## Q43. Deep Copy vs Shallow Copy

| Shallow Copy | Deep Copy |
|-------------|-----------|
| Copies field values (references shared) | Copies everything recursively |
| Changes in nested objects reflect in both | Completely independent copies |
| `Object.clone()` default | Must implement manually |

```java
class Employee implements Cloneable {
    String name;
    Address address;  // reference type

    // SHALLOW COPY — address is shared
    Employee shallowCopy() throws CloneNotSupportedException {
        return (Employee) super.clone();
    }

    // DEEP COPY — address is also cloned
    Employee deepCopy() throws CloneNotSupportedException {
        Employee copy = (Employee) super.clone();
        copy.address = (Address) address.clone();  // clone nested object
        return copy;
    }
}
```

---

## Q44. Reflection API

**Reflection** allows inspecting and modifying class structure, methods, fields at **runtime**.

```java
Class<?> clazz = Class.forName("com.example.Employee");

// Get all methods
Method[] methods = clazz.getDeclaredMethods();
for (Method m : methods) System.out.println(m.getName());

// Create instance dynamically
Object obj = clazz.getDeclaredConstructor().newInstance();

// Access private field
Field field = clazz.getDeclaredField("salary");
field.setAccessible(true);  // bypass private
field.set(obj, 50000);

// Invoke method
Method m = clazz.getMethod("getName");
String name = (String) m.invoke(obj);
```

**Used by:** Spring (DI), Hibernate (mapping), JUnit, Serialization frameworks.

> "Reflection is powerful but slow — it bypasses compile-time checks. Use it only when necessary (frameworks, testing)."

---

## Q45. Enum in Java

**Enum** = fixed set of constants. More powerful than C/C++ enums — they're full classes.

```java
public enum Status {
    PENDING("Pending Review"),
    APPROVED("Approved"),
    REJECTED("Rejected");

    private final String description;

    Status(String desc) { this.description = desc; }

    public String getDescription() { return description; }
}

// Usage
Status s = Status.APPROVED;
System.out.println(s.getDescription());  // "Approved"

// Switch
switch(s) {
    case APPROVED -> System.out.println("Go ahead!");
    case REJECTED -> System.out.println("Try again.");
}

// Enum implements Comparable, Serializable by default
// Can implement interfaces, have abstract methods
```

---

## Q46. Optional Class (Java 8)

**Optional** = container that may or may not hold a value. Eliminates `NullPointerException`.

```java
// WITHOUT Optional — NPE risk
String name = getUserFromDB(id);  // might return null
System.out.println(name.toUpperCase());  // 💥 NPE if null!

// WITH Optional — safe
Optional<String> name = Optional.ofNullable(getUserFromDB(id));

// Safe operations
name.ifPresent(n -> System.out.println(n.toUpperCase()));
String result = name.orElse("Unknown");
String result2 = name.orElseThrow(() -> new RuntimeException("Not found"));

// Chaining
Optional<String> upper = name.map(String::toUpperCase);
Optional<String> filtered = name.filter(n -> n.length() > 3);
```

> "Never use `Optional.get()` without `isPresent()`. And never use Optional as a method parameter — only as return type."
