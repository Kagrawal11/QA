# 🧪 Part 7: Output / Tricky Questions (Q143–Q150)

---

## Q143. Static Block Execution Order

```java
class Parent {
    static { System.out.println("1. Parent static block"); }
    { System.out.println("4. Parent instance block"); }
    Parent() { System.out.println("5. Parent constructor"); }
}

class Child extends Parent {
    static { System.out.println("2. Child static block"); }
    { System.out.println("6. Child instance block"); }
    Child() { System.out.println("7. Child constructor"); }
}

public class Main {
    static { System.out.println("0. Main static block"); }
    public static void main(String[] args) {
        System.out.println("3. Main method");
        new Child();
        System.out.println("--- Second object ---");
        new Child();  // static blocks DON'T run again
    }
}
```

**Output:**
```
0. Main static block
1. Parent static block
2. Child static block
3. Main method
4. Parent instance block
5. Parent constructor
6. Child instance block
7. Child constructor
--- Second object ---
4. Parent instance block
5. Parent constructor
6. Child instance block
7. Child constructor
```

**Rule:** Static blocks run ONCE when class is loaded. Instance blocks + constructors run EVERY time. Parent before child. Static → Instance → Constructor.

---

## Q144. Constructor + Inheritance Output

```java
class A {
    A()        { System.out.println("A()"); }
    A(int x)   { System.out.println("A(int): " + x); }
}

class B extends A {
    B()        { System.out.println("B()"); }
    B(int x)   { super(x); System.out.println("B(int): " + x); }
}

class C extends B {
    C()        { this(100); System.out.println("C()"); }
    C(int x)   { super(x); System.out.println("C(int): " + x); }
}

public class Main {
    public static void main(String[] args) {
        new C();
    }
}
```

**Output:**
```
A(int): 100
B(int): 100
C(int): 100
C()
```

**Why?** `C()` calls `this(100)` → `C(100)` calls `super(100)` → `B(100)` calls `super(100)` → `A(100)`. Then constructors complete in reverse order.

---

## Q145. Exception + Finally Output

```java
// Case 1: Normal
try {
    System.out.println("try");
    // no exception
} catch (Exception e) {
    System.out.println("catch");
} finally {
    System.out.println("finally");
}
// Output: try, finally

// Case 2: Exception thrown
try {
    System.out.println("try");
    int x = 10 / 0;
    System.out.println("after exception");  // NEVER executes
} catch (ArithmeticException e) {
    System.out.println("catch");
} finally {
    System.out.println("finally");
}
// Output: try, catch, finally

// Case 3: Return in try — finally STILL runs
public static int test() {
    try {
        return 1;
    } finally {
        System.out.println("finally");  // runs before return!
    }
}
// Output: finally, then returns 1

// Case 4: Return in both try and finally
public static int tricky() {
    try {
        return 1;
    } finally {
        return 2;  // ⚠️ OVERRIDES try's return!
    }
}
// Returns: 2 (finally's return wins — BAD practice, avoid!)

// Case 5: System.exit() — finally does NOT run
try {
    System.out.println("try");
    System.exit(0);  // JVM shuts down
} finally {
    System.out.println("finally");  // NEVER executes!
}
// Output: try
```

**Key Rule:** `finally` ALWAYS runs, except when `System.exit()` is called or JVM crashes.

---

## Q146. String Pool Output

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");
String s4 = new String("Hello");
String s5 = s3.intern();  // returns pool reference

System.out.println(s1 == s2);      // true  (same pool reference)
System.out.println(s1 == s3);      // false (pool vs heap)
System.out.println(s3 == s4);      // false (different heap objects)
System.out.println(s1 == s5);      // true  (intern() returns pool ref)
System.out.println(s1.equals(s3)); // true  (same content)

// String concatenation
String a = "Hello";
String b = "Hel" + "lo";       // compiler optimizes to "Hello"
System.out.println(a == b);     // true (both point to pool "Hello")

String c = "Hel";
String d = c + "lo";            // runtime concatenation → new object
System.out.println(a == d);     // false!
System.out.println(a == d.intern()); // true
```

---

## Q147. Overriding vs Overloading Output

```java
class Animal {
    void sound() { System.out.println("Animal sound"); }
    void eat(String food) { System.out.println("Animal eats " + food); }
}

class Dog extends Animal {
    @Override
    void sound() { System.out.println("Bark"); }          // OVERRIDING (runtime)
    void eat(int quantity) { System.out.println("Dog eats " + quantity); } // OVERLOADING (compile-time)
}

Animal a = new Dog();
a.sound();          // "Bark"           — runtime polymorphism (Dog's version)
a.eat("bone");      // "Animal eats bone" — Animal's eat(String)
// a.eat(5);        // ❌ COMPILE ERROR! Reference type is Animal, which has no eat(int)

Dog d = new Dog();
d.eat("bone");      // "Animal eats bone" — inherited
d.eat(5);           // "Dog eats 5"       — overloaded
```

---

## Q148. String == vs .equals() Output

```java
String a = "Java";
String b = "Java";
String c = new String("Java");
String d = new String("Java");

System.out.println(a == b);         // true  — same String Pool reference
System.out.println(a == c);         // false — pool vs heap
System.out.println(c == d);         // false — different heap objects
System.out.println(a.equals(b));    // true  — same content
System.out.println(a.equals(c));    // true  — same content
System.out.println(c.equals(d));    // true  — same content

// Integer caching trap:
Integer x = 127, y = 127;
System.out.println(x == y);        // true  — cached range [-128, 127]

Integer p = 128, q = 128;
System.out.println(p == q);        // false — outside cache!
System.out.println(p.equals(q));   // true  — same value
```

---

## Q149. try-catch-finally Output Cases

```java
// Case: Exception in catch block
try {
    throw new RuntimeException("original");
} catch (Exception e) {
    System.out.println("catch: " + e.getMessage());  // "catch: original"
    throw new RuntimeException("from catch");         // re-thrown
} finally {
    System.out.println("finally");                    // runs before exception propagates
}
// Output: catch: original → finally → then RuntimeException("from catch") propagates

// Case: Multiple catch blocks
try {
    int[] arr = new int[3];
    arr[5] = 10;
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Array error");    // this one catches it
} catch (Exception e) {
    System.out.println("General error");  // skipped
} finally {
    System.out.println("finally");
}
// Output: Array error, finally

// Case: finally with return value manipulation
public static int method() {
    int x = 10;
    try {
        return x;       // x=10 is "saved" for return
    } finally {
        x = 20;         // modifies x, but return value already saved
    }
}
// Returns: 10 (not 20!) — for primitives, the return value is already captured
```

---

## Q150. Array Index Out of Bounds Scenario

```java
// Java
int[] arr = new int[4];     // valid indices: 0, 1, 2, 3
arr[5] = 10;                // ArrayIndexOutOfBoundsException at RUNTIME

// This is a RuntimeException (unchecked) — compiles fine, fails at runtime

try {
    int[] a = {10, 20, 30};
    System.out.println(a[3]);              // ArrayIndexOutOfBoundsException
} catch (ArrayIndexOutOfBoundsException e) {
    System.out.println("Index " + e.getMessage() + " is out of bounds");
}
// Output: Index 3 is out of bounds

// C/C++ equivalent: int arr[4]; arr[5]=10;
// In C/C++: UNDEFINED BEHAVIOR — no exception! May overwrite adjacent memory,
// corrupt data, crash, or appear to work. This is a BUFFER OVERFLOW vulnerability.
```

> "Java throws a clear exception with the invalid index. C/C++ gives undefined behavior — a major source of security vulnerabilities like buffer overflow attacks."
