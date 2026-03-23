# 📦 Part 3: Collections (Q64–Q79)

---

## Q64. How to iterate over a HashMap (all ways)

```java
Map<String, Integer> map = new HashMap<>();
map.put("Alice", 90); map.put("Bob", 85); map.put("Charlie", 92);

// 1. entrySet() + for-each (MOST USED)
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// 2. keySet()
for (String key : map.keySet()) {
    System.out.println(key + " = " + map.get(key));  // extra lookup
}

// 3. values() — only values
for (Integer val : map.values()) {
    System.out.println(val);
}

// 4. Iterator
Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, Integer> e = it.next();
    if (e.getValue() < 90) it.remove();  // safe removal!
}

// 5. forEach + Lambda (Java 8)
map.forEach((key, value) -> System.out.println(key + " → " + value));

// 6. Stream API (Java 8)
map.entrySet().stream()
   .filter(e -> e.getValue() > 85)
   .forEach(e -> System.out.println(e.getKey()));
```

---

## Q65. Why override hashCode() and equals()?

If you use **custom objects as HashMap keys**, you MUST override both. Otherwise, different objects with same data will map to different buckets.

**Contract:**
- If `a.equals(b)` is `true`, then `a.hashCode() == b.hashCode()` must be true.
- If hashCodes are equal, `equals()` may or may not be true (collision).

```java
class Employee {
    int id; String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Employee)) return false;
        Employee e = (Employee) o;
        return id == e.id && Objects.equals(name, e.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

// Without override: map.get(new Employee(1, "John")) returns null even if added!
// With override: works correctly because same id+name → same hashCode → equals matches
```

---

## Q66. Internal working of HashMap (put operation — step by step)

```
map.put("Alice", 90);
```

**Step-by-step:**
1. Calculate `hashCode()` of key `"Alice"` → e.g., 63560447
2. Apply internal hash: `hash = hashCode ^ (hashCode >>> 16)` — spreads bits
3. Calculate bucket index: `index = hash & (n-1)` where n = array capacity (default 16)
4. Go to `table[index]`:
   - If **empty** → create new `Node(hash, key, value, null)`, place here
   - If **occupied** → check each node:
     - If `hash matches AND key.equals(existingKey)` → **replace value** (update)
     - Else → **append** to linked list (collision)
     - If linked list length ≥ **8** AND table size ≥ **64** → convert to **Red-Black Tree**
5. Increment `size`. If `size > capacity * loadFactor (0.75)` → **rehash** (double capacity)

```
HashMap Internal Structure:
table[0]  → null
table[1]  → [Node: "Alice"=90] → null
table[2]  → [Node: "Bob"=85] → [Node: "Dave"=78] → null  (collision!)
table[3]  → null
...
table[15] → [Node: "Charlie"=92] → null
```

---

## Q67-Q68. Collision handling & multiple keys with same hash

**Collision:** Two different keys get same bucket index.

**Java 7:** Uses **Linked List** at each bucket (chaining)
**Java 8+:** Uses **Linked List** initially, converts to **Red-Black Tree** when list length ≥ 8 (treeify threshold). Converts back to list when ≤ 6 (untreeify threshold).

```
Before Treeification:
table[5] → A → B → C → D → E → F → G → H  (8 nodes — becomes Tree)

After Treeification:
table[5] →    D (root)
             / \
            B   F
           / \ / \
          A  C E  G
                   \
                    H

Linked List: O(n) lookup
Red-Black Tree: O(log n) lookup — huge improvement for collision-heavy buckets!
```

---

## Q69. Load Factor and Rehashing

**Load Factor** = threshold ratio that triggers resizing. Default = **0.75**

```
Capacity = 16, Load Factor = 0.75
Threshold = 16 × 0.75 = 12

When 13th element is added → REHASH:
1. Create new array of size 32 (double)
2. Recalculate index for ALL existing entries
3. Move entries to new positions
4. New threshold = 32 × 0.75 = 24
```

| Load Factor | Meaning |
|-------------|---------|
| Low (0.5) | More empty buckets, less collision, more memory |
| High (1.0) | More collisions, less memory, slower lookups |
| 0.75 | **Sweet spot** — good balance of time & space |

---

## Q70. HashMap vs ConcurrentHashMap

| HashMap | ConcurrentHashMap |
|---------|-------------------|
| NOT thread-safe | Thread-safe |
| Allows 1 null key, N null values | **NO null** key or value |
| Fail-fast iterator | Weakly consistent iterator |
| Entire map locked (if synchronized externally) | Segment-level/bucket-level locking |
| Fast (single-threaded) | Optimized for concurrent access |

```java
// ConcurrentHashMap — multiple threads can read/write simultaneously
ConcurrentHashMap<String, Integer> cmap = new ConcurrentHashMap<>();
cmap.put("A", 1);  // lock only on bucket containing "A"
cmap.put("B", 2);  // different bucket — no contention!

// Atomic operations
cmap.putIfAbsent("C", 3);
cmap.computeIfPresent("A", (k, v) -> v + 10);
```

---

## Q71. How to make HashMap thread-safe?

```java
// Method 1: Collections.synchronizedMap (locks entire map)
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// Method 2: ConcurrentHashMap (RECOMMENDED — fine-grained locking)
ConcurrentHashMap<String, Integer> concMap = new ConcurrentHashMap<>();

// Method 3: Hashtable (legacy — avoid)
Hashtable<String, Integer> ht = new Hashtable<>();
```

> "Always prefer ConcurrentHashMap. synchronizedMap locks the entire map for every operation. ConcurrentHashMap allows concurrent reads and segment-level writes."

---

## Q72. LinkedHashMap — what and when to use

**LinkedHashMap** = HashMap + maintains **insertion order** (doubly-linked list).

```java
LinkedHashMap<String, Integer> lhm = new LinkedHashMap<>();
lhm.put("C", 3); lhm.put("A", 1); lhm.put("B", 2);
System.out.println(lhm);  // {C=3, A=1, B=2} — insertion order preserved!

// Access-order mode (useful for LRU Cache!)
LinkedHashMap<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);
accessOrder.put("A", 1); accessOrder.put("B", 2); accessOrder.put("C", 3);
accessOrder.get("A");  // A moves to end
System.out.println(accessOrder);  // {B=2, C=3, A=1}

// LRU Cache implementation
LinkedHashMap<String, Integer> lruCache = new LinkedHashMap<>(16, 0.75f, true) {
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > 3;  // max 3 entries
    }
};
```

---

## Q73. TreeMap and how it works internally

**TreeMap** = sorted map based on **Red-Black Tree** (self-balancing BST).

```java
TreeMap<String, Integer> tm = new TreeMap<>();
tm.put("Banana", 2); tm.put("Apple", 1); tm.put("Cherry", 3);
System.out.println(tm);  // {Apple=1, Banana=2, Cherry=3} — sorted by key!

// Navigation methods
tm.firstKey();          // "Apple"
tm.lastKey();           // "Cherry"
tm.lowerKey("Banana");  // "Apple" — strictly less
tm.subMap("Apple", "Cherry");  // {Apple=1, Banana=2}

// Custom comparator
TreeMap<String, Integer> desc = new TreeMap<>(Comparator.reverseOrder());
```

| Feature | Complexity |
|---------|-----------|
| put/get/remove | O(log n) |
| Null keys | NOT allowed |
| Thread-safe | No |

---

## Q74. HashMap Tree conversion (Java 8)

When a bucket's linked list reaches **8 nodes** AND total capacity ≥ **64**, it converts to a **Red-Black Tree** for O(log n) lookups instead of O(n).

When tree shrinks to **6 nodes** or fewer, it converts back to linked list (un-treeify).

**Constants in HashMap source:**
```java
static final int TREEIFY_THRESHOLD = 8;
static final int UNTREEIFY_THRESHOLD = 6;
static final int MIN_TREEIFY_CAPACITY = 64;
```

---

## Q75. Load Factor & Initial Capacity

```java
// Default: capacity=16, loadFactor=0.75
HashMap<String, Integer> map1 = new HashMap<>();

// Custom: capacity=32, loadFactor=0.5
HashMap<String, Integer> map2 = new HashMap<>(32, 0.5f);
```

**Best practice:** If you know the expected size, set initial capacity = `expectedSize / 0.75 + 1` to avoid rehashing.

```java
// Expecting 100 entries
HashMap<String, Integer> map = new HashMap<>(134);  // 100/0.75 ≈ 134
```

---

## Q76. HashSet vs TreeSet

| HashSet | TreeSet |
|---------|---------|
| Backed by HashMap | Backed by TreeMap |
| Unordered | **Sorted** (natural or custom) |
| O(1) add/remove/contains | O(log n) |
| Allows ONE null | NO nulls |
| Uses hashCode()/equals() | Uses compareTo()/Comparator |

---

## Q77. PriorityQueue

**PriorityQueue** = Queue where elements are ordered by **priority** (min-heap by default).

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.add(30); minHeap.add(10); minHeap.add(20);
System.out.println(minHeap.poll());  // 10 (smallest first)
System.out.println(minHeap.poll());  // 20

// Max-Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.add(30); maxHeap.add(10); maxHeap.add(20);
System.out.println(maxHeap.poll());  // 30 (largest first)
```

**Use case:** Task scheduling, Dijkstra's algorithm, finding top-K elements.

---

## Q78-Q79. Making Collections Thread-Safe

```java
// 1. Synchronized wrappers
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// 2. Concurrent collections (PREFERRED)
ConcurrentHashMap<String, Integer> cmap = new ConcurrentHashMap<>();
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();  // read-heavy
CopyOnWriteArraySet<String> cowSet = new CopyOnWriteArraySet<>();
ConcurrentLinkedQueue<String> clq = new ConcurrentLinkedQueue<>();
BlockingQueue<String> bq = new LinkedBlockingQueue<>();  // producer-consumer

// 3. Unmodifiable (immutable)
List<String> immutable = Collections.unmodifiableList(list);
List<String> jdk9 = List.of("A", "B", "C");  // Java 9+
```

> "CopyOnWriteArrayList creates a new copy on every write — great for read-heavy, write-rare scenarios like listener lists."
