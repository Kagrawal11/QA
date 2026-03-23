# 📘 Part 1: Core Java & OOPs (Q1–Q23)

---

## Q1. What is Java, JRE, JVM, and JDK?

**Java** is a high-level, object-oriented, platform-independent programming language developed by James Gosling at Sun Microsystems (1995). It follows **WORA** — Write Once, Run Anywhere.

| Component | Full Form | Role |
|-----------|-----------|------|
| **JDK** | Java Development Kit | Superset — includes JRE + dev tools (javac, jdb, javadoc) |
| **JRE** | Java Runtime Environment | JVM + class libraries — needed to **run** Java programs |
| **JVM** | Java Virtual Machine | Executes bytecode — platform-dependent engine |

**Flow:**
```
Source.java → javac (compiler in JDK) → Source.class (bytecode) → JVM (interprets/executes) → Output
```

**Key Differentiator Answer:**
> "JDK is for developers, JRE is for users, JVM is the engine. JDK ⊃ JRE ⊃ JVM. JVM itself is platform-dependent — there's a different JVM for Windows, Linux, Mac — but the bytecode it runs is platform-independent. That's how Java achieves WORA."

---

## Q2. Why is Java platform-independent, and why is it not 100% object-oriented?

**Platform Independence:**
Java source code is compiled into **bytecode** (.class), not native machine code. This bytecode runs on **any JVM**, regardless of OS.

```
Developer (Windows) → .class file → JVM (Linux) → Runs perfectly
```

**Why NOT 100% OOP:**
Java uses **8 primitive data types** that are NOT objects:
`byte, short, int, long, float, double, char, boolean`

These exist for **performance** — objects have overhead (heap allocation, GC). Primitives live on the **stack** and are faster.

**Other reasons:** `static` methods belong to the class (not objects), and `main()` is static.

> "If Java were 100% OOP, even `int x = 5;` would be an object on the heap. Java trades purity for performance with primitives. Languages like Smalltalk and Ruby are closer to 100% OOP."

---

## Q3. Explain OOPs concepts, the 4 pillars, and provide real-life examples.

### The 4 Pillars:

**1. Encapsulation** — Wrapping data + methods, hiding internal state
```java
public class BankAccount {
    private double balance; // hidden

    public void deposit(double amount) {  // controlled access
        if (amount > 0) this.balance += amount;
    }
    public double getBalance() { return balance; }
}
```
*Real-life:* ATM machine — you interact via buttons, internal cash mechanism is hidden.

**2. Inheritance** — Child class acquires properties of parent class
```java
class Vehicle { void start() { System.out.println("Starting..."); } }
class Car extends Vehicle { void playMusic() { System.out.println("Playing music"); } }
// Car inherits start() from Vehicle
```
*Real-life:* iPhone 15 inherits features from iPhone 14 and adds new ones.

**3. Polymorphism** — One name, many forms
```java
// Compile-time (Overloading)
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
}

// Runtime (Overriding)
class Animal { void sound() { System.out.println("Some sound"); } }
class Dog extends Animal { void sound() { System.out.println("Bark!"); } }

Animal a = new Dog();
a.sound(); // Output: Bark! (decided at runtime)
```
*Real-life:* A person is a student at school, a son at home, a customer at a shop — same person, different behavior.

**4. Abstraction** — Showing only essential features, hiding complexity
```java
abstract class Payment {
    abstract void pay(double amount); // WHAT to do
}
class UPI extends Payment {
    void pay(double amount) { /* HOW — UPI logic */ }
}
class CreditCard extends Payment {
    void pay(double amount) { /* HOW — Card logic */ }
}
```
*Real-life:* Car steering wheel — you turn it, you don't need to know the rack-and-pinion mechanism.

---

## Q4. What is a class, an object, and what is the size of an empty class?

- **Class:** Blueprint/template that defines properties (fields) and behaviors (methods).
- **Object:** Instance of a class — a real entity in memory.

```java
class Student { }                    // Class (blueprint)
Student s = new Student();           // Object (instance in heap)
```

**Size of an empty class object in Java:**
> In Java, even an empty class object occupies **16 bytes** (on 64-bit JVM with compressed oops). This includes:
> - **12 bytes** object header (Mark Word 8B + Klass Pointer 4B)
> - **4 bytes** padding (JVM aligns to 8-byte boundary)

> "Unlike C++ where an empty class is 1 byte, Java's object header carries metadata for GC, locking, and identity hashCode — that's why it's 16 bytes minimum."

---

## Q5. Difference between Abstract class and Interface?

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Can have abstract + concrete | All abstract (pre-Java 8); default/static allowed (Java 8+) |
| Variables | Can have instance variables | Only `public static final` (constants) |
| Constructor | ✅ Yes | ❌ No |
| Inheritance | `extends` (single) | `implements` (multiple) |
| Access Modifiers | Any | Methods are `public` by default |
| When to use | IS-A + shared code | CAN-DO capability / contract |

```java
abstract class Animal {
    String name;
    Animal(String name) { this.name = name; }
    abstract void sound();
    void breathe() { System.out.println("Breathing..."); } // shared code
}

interface Swimmable {
    void swim(); // contract
}

class Dog extends Animal implements Swimmable {
    Dog(String n) { super(n); }
    void sound() { System.out.println("Bark"); }
    public void swim() { System.out.println("Dog paddling"); }
}
```

> "Use abstract class when classes share common state/code. Use interface when unrelated classes need the same capability — like `Serializable` can be implemented by any class."

---

## Q6. Explain the abstract keyword and provide examples.

The `abstract` keyword is used with **classes** and **methods** to define incomplete implementations.

**Rules:**
- Abstract **method**: No body, must be overridden by subclass.
- Abstract **class**: Cannot be instantiated; can have both abstract and concrete methods.
- If a class has even one abstract method → class MUST be abstract.
- A class can be abstract without any abstract methods (to prevent instantiation).

```java
abstract class Shape {
    abstract double area();        // no implementation
    void display() {               // concrete method
        System.out.println("Area: " + area());
    }
}

class Circle extends Shape {
    double radius;
    Circle(double r) { this.radius = r; }
    double area() { return Math.PI * radius * radius; }
}

// Shape s = new Shape();       // ❌ Compile error!
Shape s = new Circle(5);        // ✅ Polymorphism
s.display();                     // Output: Area: 78.54
```

---

## Q7. What are access modifiers/specifiers in Java?

| Modifier | Same Class | Same Package | Subclass (diff pkg) | World |
|----------|-----------|-------------|---------------------|-------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` (no keyword) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Memory trick:** `private < default < protected < public`

```java
public class Employee {
    private int salary;         // only this class
    int department;             // same package
    protected String name;      // same pkg + subclasses
    public int id;              // everywhere
}
```

> "In real projects, fields are `private`, getters/setters are `public` (encapsulation). `protected` is used in framework code where subclassing is expected."

---

## Q8. Types of inheritance, multiple inheritance, hybrid inheritance, and diamond problem.

**Types of Inheritance:**
```
1. Single:      A → B
2. Multilevel:  A → B → C
3. Hierarchical: A → B, A → C
4. Multiple:    A → C, B → C  (NOT supported via classes; supported via interfaces)
5. Hybrid:      Combination of above
```

**Diamond Problem:**
```
        A (greet())
       / \
      B   C    (both override greet())
       \ /
        D      — Which greet() does D inherit? AMBIGUOUS!
```

Java **avoids** this by NOT allowing multiple class inheritance. With interfaces (Java 8+), if both interfaces have default methods with same signature, the implementing class **must override** it:

```java
interface A { default void greet() { System.out.println("A"); } }
interface B { default void greet() { System.out.println("B"); } }

class C implements A, B {
    public void greet() {     // MUST override to resolve
        A.super.greet();      // explicitly choose
    }
}
```

---

## Q9. Method Overloading vs Method Overriding & Overriding Rules

| Feature | Overloading | Overriding |
|---------|------------|------------|
| Definition | Same method name, different parameters | Same method signature in parent & child |
| Binding | Compile-time (Static) | Runtime (Dynamic) |
| Where | Same class | Parent-child relationship |
| Return Type | Can differ | Must be same or covariant |
| Access | Can differ | Cannot be MORE restrictive |
| Static | Can overload static | Cannot override static (method hiding) |
| Private | Can overload private | Cannot override private |

**Overriding Rules:**
1. Method signature must be IDENTICAL
2. Return type: same or covariant (subtype)
3. Access modifier: same or WIDER (not narrower)
4. Cannot override `final`, `static`, `private` methods
5. Constructor cannot be overridden
6. Checked exceptions: child can throw same, narrower, or none — NOT broader

```java
class Parent {
    protected Number calculate() { return 42; }
}
class Child extends Parent {
    @Override
    public Integer calculate() { return 100; } // ✅ Wider access + covariant return
}
```

---

## Q10. Constructors in Java and Constructor Chaining

**Constructor:** Special method to initialize objects. Same name as class, no return type.

**Types:**
```java
class Student {
    String name; int age;

    Student() { this("Unknown", 0); }              // Default → calls parameterized
    Student(String name) { this(name, 18); }        // Overloaded
    Student(String name, int age) {                  // Parameterized
        this.name = name;
        this.age = age;
    }
}
```

**Constructor Chaining:** One constructor calling another using `this()` (same class) or `super()` (parent class).

```java
class Animal {
    Animal() { System.out.println("Animal created"); }
}
class Dog extends Animal {
    Dog() {
        super();   // calls Animal() — MUST be first line
        System.out.println("Dog created");
    }
}
// new Dog() → Output: "Animal created" then "Dog created"
```

**Key Rules:**
- `this()` and `super()` must be the FIRST statement
- If you don't write `super()`, compiler inserts it automatically
- Constructor is NOT inherited

---

## Q11. Static keyword, static block, non-static block, and static variables

**Static** = belongs to the **class**, not to any instance.

```java
class Counter {
    static int count = 0;      // shared across ALL objects
    int id;

    static {                    // STATIC BLOCK — runs ONCE when class is loaded
        System.out.println("Static block executed");
        count = 0;
    }

    {                           // INSTANCE BLOCK — runs EVERY TIME before constructor
        count++;
        id = count;
        System.out.println("Instance block: id=" + id);
    }

    Counter() {
        System.out.println("Constructor: id=" + id);
    }

    static void showCount() {  // static method — no access to 'this'
        System.out.println("Total objects: " + count);
    }
}
```

**Execution Order:** `Static block → Instance block → Constructor`

| Static | Non-Static |
|--------|-----------|
| Loaded once with class | Created per object |
| Can't access instance members | Can access static members |
| Called via `ClassName.method()` | Called via `object.method()` |

---

## Q12. Final keyword and finalize() method

**`final` keyword — 3 uses:**

| Usage | Effect |
|-------|--------|
| `final variable` | Constant — value can't change |
| `final method` | Can't be overridden |
| `final class` | Can't be inherited |

```java
final class Constants {                          // Can't extend
    final double PI = 3.14159;                   // Can't reassign
    final void display() { /* Can't override */ }
}
```
*Examples:* `String`, `Integer`, wrapper classes are all `final` classes.

**`finalize()` method:**
- Called by **Garbage Collector** before destroying an object.
- Defined in `Object` class, can be overridden.
- **Deprecated since Java 9** — unpredictable and unreliable.
- Use **try-with-resources** or `Cleaner` API instead.

```java
@Override
protected void finalize() throws Throwable {
    System.out.println("Object garbage collected");
    super.finalize();
}
```

---

## Q13. What are Wrapper classes?

Wrapper classes convert **primitives → objects** (Boxing) and **objects → primitives** (Unboxing).

| Primitive | Wrapper |
|-----------|---------|
| int | Integer |
| char | Character |
| double | Double |
| boolean | Boolean |

```java
// Autoboxing (primitive → object)
int x = 10;
Integer obj = x;              // auto-boxed

// Unboxing (object → primitive)
Integer obj2 = Integer.valueOf(20);
int y = obj2;                 // auto-unboxed

// WHY needed?
List<Integer> list = new ArrayList<>();  // Collections need objects, not primitives!
list.add(42);                            // autoboxing happens here
```

**Why they exist:** Collections framework, Generics, `null` support, utility methods like `Integer.parseInt("123")`.

---

## Q14. Exception handling, hierarchy, NullPointerException, ClassCastException

**Exception Hierarchy:**
```
java.lang.Object
  └── Throwable
        ├── Error (OutOfMemoryError, StackOverflowError) — unrecoverable
        └── Exception
              ├── Checked (IOException, SQLException) — must handle at compile time
              └── RuntimeException (Unchecked)
                    ├── NullPointerException
                    ├── ClassCastException
                    ├── ArrayIndexOutOfBoundsException
                    └── ArithmeticException
```

```java
try {
    String s = null;
    s.length();                      // NullPointerException
} catch (NullPointerException e) {
    System.out.println("Null reference: " + e.getMessage());
} finally {
    System.out.println("Always executes"); // cleanup
}

// ClassCastException
Object obj = "Hello";
Integer num = (Integer) obj;         // ClassCastException at runtime!
```

**Best Practices:**
- Catch specific exceptions, not generic `Exception`
- Use `finally` for cleanup (or try-with-resources)
- Don't swallow exceptions with empty catch blocks
- Use custom exceptions for business logic

```java
class InsufficientFundsException extends Exception {
    public InsufficientFundsException(String msg) { super(msg); }
}
```

---

## Q15. Difference between String, StringBuffer, and StringBuilder

| Feature | String | StringBuffer | StringBuilder |
|---------|--------|-------------|---------------|
| Mutability | **Immutable** | Mutable | Mutable |
| Thread-safe | Yes (immutable) | Yes (synchronized) | **No** |
| Performance | Slow for concatenation | Moderate | **Fastest** |
| Storage | String Pool + Heap | Heap | Heap |
| When to use | Few modifications | Multi-threaded + modifications | Single-threaded + modifications |

```java
// String — creates NEW object each time
String s = "Hello";
s = s + " World";    // Original "Hello" abandoned, new object created

// StringBuilder — modifies same object
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World"); // Same object modified — no waste!

// StringBuffer — same as StringBuilder but thread-safe
StringBuffer sbuf = new StringBuffer("Hello");
sbuf.append(" World"); // Synchronized — slower but safe
```

> "In interviews I always say: for loops with concatenation, NEVER use String. Use StringBuilder. If shared across threads, use StringBuffer. String is for constants."

---

## Q16. Why is String immutable? How many objects are created?

**Why Immutable:**
1. **String Pool:** JVM reuses strings — mutation would affect all references
2. **Security:** DB URLs, passwords stored as strings — shouldn't change
3. **Thread Safety:** Immutable = inherently thread-safe
4. **Caching hashCode:** String caches its hash — used extensively in HashMap

**Object Count Analysis:**
```java
String s1 = "Hello";           // 1 object in String Pool
String s2 = "Hello";           // 0 new objects — reuses from pool
String s3 = new String("Hello"); // 1 new object on Heap (pool already has "Hello")
String s4 = new String("World"); // 2 objects: "World" in pool + new on heap

String s5 = "Hi" + "There";   // Compiler optimizes to "HiThere" — 1 pool object
String a = "Hi";
String s6 = a + "There";      // Creates StringBuilder internally — 1 heap object
```

```
s1 ──→ ┌──────────┐
s2 ──→ │ "Hello"  │  ← String Pool (shared)
       └──────────┘
s3 ──→ ┌──────────┐ ──→ Pool "Hello"
       │ Heap Obj │
       └──────────┘
```

---

## Q17. Multithreading — creating threads and thread lifecycle

**Two Ways to Create a Thread:**

```java
// Way 1: Extend Thread
class MyThread extends Thread {
    public void run() { System.out.println("Thread: " + getName()); }
}

// Way 2: Implement Runnable (PREFERRED — allows extending another class)
class MyRunnable implements Runnable {
    public void run() { System.out.println("Runnable thread"); }
}

// Usage
new MyThread().start();
new Thread(new MyRunnable()).start();
new Thread(() -> System.out.println("Lambda thread")).start(); // Java 8
```

**Thread Lifecycle:**
```
NEW → (start()) → RUNNABLE → (scheduler) → RUNNING → (complete) → TERMINATED
                      ↑                        ↓
                      └── BLOCKED/WAITING ←────┘  (sleep/wait/lock)
```

| State | Description |
|-------|-------------|
| NEW | Thread created, not started |
| RUNNABLE | Ready to run, waiting for CPU |
| RUNNING | Executing run() |
| BLOCKED/WAITING | Waiting for lock, sleep(), wait() |
| TERMINATED | Execution completed or exception |

---

## Q18. What happens if you call .start() twice?

```java
Thread t = new Thread(() -> System.out.println("Running"));
t.start();    // ✅ First call — thread starts normally
t.start();    // ❌ Throws IllegalThreadStateException!
```

**Why?** A thread can only transition from `NEW → RUNNABLE` once. After `start()`, the thread is no longer in `NEW` state. Calling `start()` again is illegal.

> "If you need the same task to run again, create a NEW Thread object. A Thread object is single-use — like a match stick, you can't re-light it."

---

## Q19. Object Lock vs. Class Lock

| Feature | Object Lock | Class Lock |
|---------|------------|------------|
| Scope | Per instance | Per class (shared by all instances) |
| Acquired on | `this` (instance) | `ClassName.class` |
| Use case | Instance methods | Static synchronized methods |

```java
class Printer {
    // Object lock — different objects can run simultaneously
    synchronized void printInstance(String msg) {
        System.out.println(msg);
    }

    // Class lock — ALL objects blocked until lock released
    static synchronized void printClass(String msg) {
        System.out.println(msg);
    }

    void demo() {
        synchronized(this) { /* object lock */ }
        synchronized(Printer.class) { /* class lock */ }
    }
}
```

---

## Q20. Difference between == and .equals()

| `==` | `.equals()` |
|------|------------|
| Compares **references** (memory address) | Compares **content/value** |
| Works on primitives and objects | Only works on objects |
| Cannot be overridden | Can be overridden |

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

s1 == s2;      // true  — same pool reference
s1 == s3;      // false — different memory locations
s1.equals(s3); // true  — same content

Integer a = 127, b = 127;
a == b;        // true  — Integer cache (-128 to 127)
Integer c = 128, d = 128;
c == d;        // false — outside cache range!
c.equals(d);   // true  — same value
```

> "Always use `.equals()` for object comparison. The `==` on Integer cache gotcha is a classic interview trick question."

---

## Q21. Transient and Volatile keywords

**Transient:** Excludes a field from **serialization**.
```java
class User implements Serializable {
    String username;
    transient String password;  // NOT saved during serialization
}
// After deserialization, password = null
```

**Volatile:** Ensures a variable is read from **main memory**, not CPU cache. Guarantees **visibility** across threads.
```java
class SharedFlag {
    volatile boolean running = true;  // all threads see latest value

    void stop() { running = false; }  // change visible to other threads immediately
    void run() {
        while (running) { /* work */ }  // without volatile, may loop forever!
    }
}
```

| Transient | Volatile |
|-----------|---------|
| Related to Serialization | Related to Multithreading |
| Skips field during serialization | Reads/writes from main memory |
| Default value after deserialization | Ensures visibility across threads |

---

## Q22. Collections framework, Collection hierarchy, and Iterators

**Hierarchy:**
```
                    Iterable
                       │
                   Collection
                  /    |     \
               List   Set    Queue
              / |       |  \      \
       ArrayList  HashSet TreeSet  PriorityQueue
       LinkedList LinkedHashSet    Deque
       Vector
       
            Map (separate hierarchy)
           / |  \
    HashMap  TreeMap  HashTable
    LinkedHashMap     ConcurrentHashMap
```

**Iterators:**
```java
List<String> list = Arrays.asList("A", "B", "C");

// 1. Iterator
Iterator<String> it = list.iterator();
while(it.hasNext()) System.out.println(it.next());

// 2. ListIterator (bidirectional — only for List)
ListIterator<String> lit = list.listIterator();
while(lit.hasNext()) System.out.println(lit.next());
while(lit.hasPrevious()) System.out.println(lit.previous());

// 3. For-each
for(String s : list) System.out.println(s);

// 4. forEach + Lambda (Java 8)
list.forEach(s -> System.out.println(s));

// 5. Spliterator (parallel processing)
list.spliterator().forEachRemaining(System.out::println);
```

**ConcurrentModificationException:**
```java
// WRONG — modifying while iterating
for(String s : list) { if(s.equals("B")) list.remove(s); } // ❌ CME!

// CORRECT — use Iterator.remove()
Iterator<String> it = list.iterator();
while(it.hasNext()) { if(it.next().equals("B")) it.remove(); } // ✅
```

---

## Q23. Array vs ArrayList, ArrayList vs LinkedList, ArrayList vs Vector

**Array vs ArrayList:**

| Array | ArrayList |
|-------|-----------|
| Fixed size | Dynamic size |
| Can hold primitives | Only objects (autoboxing) |
| `arr[i]` | `list.get(i)` |
| No built-in methods | Rich API (add, remove, sort) |

**ArrayList vs LinkedList:**

| ArrayList | LinkedList |
|-----------|-----------|
| Dynamic array internally | Doubly linked list |
| **O(1)** random access | **O(n)** random access |
| **O(n)** insert/delete middle | **O(1)** insert/delete (if position known) |
| Better for **read-heavy** | Better for **write-heavy** |
| Less memory | More memory (prev + next pointers) |

**ArrayList vs Vector:**

| ArrayList | Vector |
|-----------|--------|
| NOT synchronized | Synchronized (thread-safe) |
| Faster | Slower |
| Grows by 50% | Grows by 100% |
| Introduced Java 1.2 | Legacy (Java 1.0) |

> "In modern code, we never use Vector. If thread safety is needed, use `Collections.synchronizedList()` or `CopyOnWriteArrayList`."
