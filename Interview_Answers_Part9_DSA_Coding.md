# 🧠 Part 9: Data Structures, Algorithms & Coding (Q169–Q200)

---

## Q169. Code a Sorting Algorithm

```java
// BUBBLE SORT — Simple, O(n²)
public static void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                swapped = true;
            }
        }
        if (!swapped) break;  // optimization: already sorted
    }
}

// QUICK SORT — Efficient, O(n log n) average
public static void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i]; arr[i] = arr[j]; arr[j] = temp;
        }
    }
    int temp = arr[i + 1]; arr[i + 1] = arr[high]; arr[high] = temp;
    return i + 1;
}

// MERGE SORT — Stable, O(n log n) guaranteed
public static void mergeSort(int[] arr, int l, int r) {
    if (l < r) {
        int mid = (l + r) / 2;
        mergeSort(arr, l, mid);
        mergeSort(arr, mid + 1, r);
        merge(arr, l, mid, r);
    }
}

private static void merge(int[] arr, int l, int mid, int r) {
    int[] left = Arrays.copyOfRange(arr, l, mid + 1);
    int[] right = Arrays.copyOfRange(arr, mid + 1, r + 1);
    int i = 0, j = 0, k = l;
    while (i < left.length && j < right.length)
        arr[k++] = (left[i] <= right[j]) ? left[i++] : right[j++];
    while (i < left.length) arr[k++] = left[i++];
    while (j < right.length) arr[k++] = right[j++];
}
```

---

## Q170. Count Frequency of Characters in a String

```java
public static void charFrequency(String str) {
    // Method 1: HashMap
    Map<Character, Integer> freq = new LinkedHashMap<>();
    for (char c : str.toCharArray()) {
        freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    freq.forEach((k, v) -> System.out.println(k + " → " + v));

    // Method 2: Java 8 Streams
    str.chars()
       .mapToObj(c -> (char) c)
       .collect(Collectors.groupingBy(c -> c, Collectors.counting()))
       .forEach((k, v) -> System.out.println(k + " → " + v));
}
// Input: "hello" → h→1, e→1, l→2, o→1
```

---

## Q171. Check Prime Without Modulus Operator

```java
public static boolean isPrime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; i++) {
        // Instead of n % i == 0, use subtraction
        int temp = n;
        while (temp > 0) {
            temp -= i;
        }
        if (temp == 0) return false;  // divisible → not prime
    }
    return true;
}

// Alternative: use division
public static boolean isPrimeAlt(int n) {
    if (n <= 1) return false;
    for (int i = 2; i * i <= n; i++) {
        if (n / i * i == n) return false;  // if n/i is exact → divisible
    }
    return true;
}
```

---

## Q172. Number of Digits After Decimal Point

```java
public static int digitsAfterDecimal(double num) {
    String s = String.valueOf(num);
    int dotIndex = s.indexOf('.');
    if (dotIndex == -1) return 0;
    // Remove trailing zeros
    String decimal = s.substring(dotIndex + 1).replaceAll("0+$", "");
    return decimal.isEmpty() ? 0 : decimal.length();
}
// Input: 3.14159 → 5, Input: 10.0 → 0
```

---

## Q173. Merge Two Sorted Arrays

```java
public static int[] mergeSorted(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) {
        if (a[i] <= b[j]) result[k++] = a[i++];
        else result[k++] = b[j++];
    }
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
// Input: [1,3,5] + [2,4,6] → [1,2,3,4,5,6]  — O(n+m) time
```

---

## Q174. Print Pythagorean Triplets

```java
public static void pythagoreanTriplets(int limit) {
    for (int a = 1; a <= limit; a++) {
        for (int b = a; b <= limit; b++) {
            int cSquared = a * a + b * b;
            int c = (int) Math.sqrt(cSquared);
            if (c * c == cSquared && c <= limit) {
                System.out.println(a + ", " + b + ", " + c);
            }
        }
    }
}
// Output: 3,4,5 | 5,12,13 | 6,8,10 | 8,15,17 | ...
```

---

## Q175. Reverse a Number

```java
public static int reverse(int num) {
    int reversed = 0;
    while (num != 0) {
        reversed = reversed * 10 + num % 10;
        num /= 10;
    }
    return reversed;
}
// Input: 12345 → Output: 54321
// Input: -123  → Output: -321
```

---

## Q176. Print Factors of a Number

```java
public static void printFactors(int n) {
    System.out.print("Factors of " + n + ": ");
    for (int i = 1; i <= n; i++) {
        if (n % i == 0) System.out.print(i + " ");
    }
    // Optimized: only check up to sqrt(n)
    System.out.println();
    for (int i = 1; i * i <= n; i++) {
        if (n % i == 0) {
            System.out.print(i + " ");
            if (i != n / i) System.out.print(n / i + " ");
        }
    }
}
// Input: 12 → 1 2 3 4 6 12
```

---

## Q177. Palindrome Check Without String Functions

```java
// Number palindrome
public static boolean isPalindromeNum(int n) {
    int original = n, reversed = 0;
    while (n > 0) {
        reversed = reversed * 10 + n % 10;
        n /= 10;
    }
    return original == reversed;
}

// String palindrome WITHOUT string functions
public static boolean isPalindromeStr(String s) {
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        if (chars[left] != chars[right]) return false;
        left++;
        right--;
    }
    return true;
}

// Advanced: Ignore case and non-alphanumeric
public static boolean isPalindromeAdvanced(String s) {
    char[] c = s.toCharArray();
    int l = 0, r = c.length - 1;
    while (l < r) {
        while (l < r && !Character.isLetterOrDigit(c[l])) l++;
        while (l < r && !Character.isLetterOrDigit(c[r])) r--;
        if (Character.toLowerCase(c[l]) != Character.toLowerCase(c[r])) return false;
        l++; r--;
    }
    return true;
}
// "A man, a plan, a canal: Panama" → true
```

---

## Q178. Highest Digit in a Number

```java
public static int highestDigit(int n) {
    n = Math.abs(n);
    int max = 0;
    while (n > 0) {
        int digit = n % 10;
        if (digit > max) max = digit;
        if (max == 9) return 9;  // optimization: can't be higher
        n /= 10;
    }
    return max;
}
// Input: 38527 → Output: 8
```

---

## Q179. Add/Delete Node in Linked List

```java
class Node {
    int data;
    Node next;
    Node(int d) { data = d; next = null; }
}

class SinglyLinkedList {
    Node head;

    // Add at beginning — O(1)
    void addFirst(int data) {
        Node newNode = new Node(data);
        newNode.next = head;
        head = newNode;
    }

    // Add at end — O(n)
    void addLast(int data) {
        Node newNode = new Node(data);
        if (head == null) { head = newNode; return; }
        Node curr = head;
        while (curr.next != null) curr = curr.next;
        curr.next = newNode;
    }

    // Delete by value — O(n)
    void delete(int data) {
        if (head == null) return;
        if (head.data == data) { head = head.next; return; }
        Node curr = head;
        while (curr.next != null && curr.next.data != data) curr = curr.next;
        if (curr.next != null) curr.next = curr.next.next;
    }

    // Print list
    void print() {
        Node curr = head;
        while (curr != null) {
            System.out.print(curr.data + " → ");
            curr = curr.next;
        }
        System.out.println("null");
    }
}

// Doubly Linked List Node
class DNode {
    int data;
    DNode prev, next;
    DNode(int d) { data = d; prev = null; next = null; }
}

// Add to doubly linked list
void addDLL(int data) {
    DNode newNode = new DNode(data);
    if (head == null) { head = newNode; return; }
    DNode curr = head;
    while (curr.next != null) curr = curr.next;
    curr.next = newNode;
    newNode.prev = curr;
}

// Delete from doubly linked list
void deleteDLL(DNode node) {
    if (node.prev != null) node.prev.next = node.next;
    else head = node.next;  // deleting head
    if (node.next != null) node.next.prev = node.prev;
}
```

---

## Q180. Count Special Characters, Alphabets, Numbers

```java
public static void countCharTypes(String str) {
    int alpha = 0, digits = 0, special = 0, spaces = 0;
    for (char c : str.toCharArray()) {
        if (Character.isLetter(c)) alpha++;
        else if (Character.isDigit(c)) digits++;
        else if (c == ' ') spaces++;
        else special++;
    }
    System.out.println("Alphabets: " + alpha + ", Digits: " + digits
        + ", Special: " + special + ", Spaces: " + spaces);
}
// Input: "Hello World! 123@#" → Alphabets:10, Digits:3, Special:3, Spaces:2
```

---

## Q181. Print First N Elements of Array

```java
public static void printFirstN(int[] arr, int n) {
    if (n > arr.length) {
        System.out.println("N exceeds array length!");
        n = arr.length;
    }
    for (int i = 0; i < n; i++) {
        System.out.print(arr[i] + " ");
    }
}
// Java 8: Arrays.stream(arr).limit(n).forEach(System.out::println);
```

---

## Q182. Find All Pairs with Given Sum

```java
public static void findPairs(int[] arr, int targetSum) {
    // Method 1: HashSet — O(n) time, O(n) space
    Set<Integer> seen = new HashSet<>();
    for (int num : arr) {
        int complement = targetSum - num;
        if (seen.contains(complement)) {
            System.out.println("(" + complement + ", " + num + ")");
        }
        seen.add(num);
    }
}
// Input: arr=[1,5,7,2,3,8], sum=10 → (3,7), (2,8)

// Method 2: Sort + Two Pointers — O(n log n), O(1) space
public static void findPairsSort(int[] arr, int target) {
    Arrays.sort(arr);
    int left = 0, right = arr.length - 1;
    while (left < right) {
        int sum = arr[left] + arr[right];
        if (sum == target) {
            System.out.println("(" + arr[left] + ", " + arr[right] + ")");
            left++; right--;
        } else if (sum < target) left++;
        else right--;
    }
}
```

---

## Q183. Power of a Number

```java
// Iterative
public static long power(int base, int exp) {
    long result = 1;
    for (int i = 0; i < exp; i++) result *= base;
    return result;
}

// Efficient: O(log n) — Fast Exponentiation
public static long fastPower(long base, int exp) {
    long result = 1;
    while (exp > 0) {
        if (exp % 2 == 1) result *= base;
        base *= base;
        exp /= 2;
    }
    return result;
}
// fastPower(2, 10) → 1024
```

---

## Q184. Pascal's Triangle

```java
public static void pascalTriangle(int rows) {
    for (int i = 0; i < rows; i++) {
        int val = 1;
        // Print leading spaces for pyramid shape
        for (int s = 0; s < rows - i - 1; s++) System.out.print(" ");
        for (int j = 0; j <= i; j++) {
            System.out.print(val + " ");
            val = val * (i - j) / (j + 1);
        }
        System.out.println();
    }
}
/*  Output (5 rows):
        1
       1 1
      1 2 1
     1 3 3 1
    1 4 6 4 1
*/
```

---

## Q185. Swap Two Numbers

```java
// Method 1: Using temp
int temp = a; a = b; b = temp;

// Method 2: Without temp (arithmetic)
a = a + b;    // a = 15 (5+10)
b = a - b;    // b = 5  (15-10)
a = a - b;    // a = 10 (15-5)

// Method 3: XOR (best — no overflow risk)
a = a ^ b;
b = a ^ b;
a = a ^ b;
```

---

## Q186. printf() Without Semicolon (C/C++ trick)

```c
// In C/C++:
if (printf("Hello World\n")) { }    // printf inside if condition
// or
while (!printf("Hello World\n")) { } // printf inside while
// or
switch (printf("Hello World\n")) { } // printf inside switch
```

> "This is a C/C++ trick. `printf` returns the number of characters printed (non-zero = true). In Java, this isn't applicable — `System.out.println()` returns void."

---

## Q187. int arr[4]; arr[5]=10; — Output?

```c
// C/C++: UNDEFINED BEHAVIOR
int arr[4];   // indices 0-3
arr[5] = 10;  // writing beyond array bounds!
// May: crash (segfault), corrupt memory, work silently, or anything else
// This is a BUFFER OVERFLOW — a major security vulnerability

// Java equivalent:
int[] arr = new int[4];
arr[5] = 10;  // throws ArrayIndexOutOfBoundsException (safe!)
```

---

## Q188-Q192. Print Odd/Even, Max Element, Reverse String, Check Anagram, Count Vowels

```java
// Print odd/even
for (int i = 1; i <= 20; i++)
    System.out.println(i + " is " + (i % 2 == 0 ? "Even" : "Odd"));

// Separate odd/even using streams
IntStream.rangeClosed(1, 20)
    .filter(n -> n % 2 == 0)
    .forEach(n -> System.out.print(n + " "));

// Max element
public static int findMax(int[] arr) {
    int max = arr[0];
    for (int num : arr) if (num > max) max = num;
    return max;
}
// Or: Arrays.stream(arr).max().getAsInt();

// Reverse string WITHOUT function
public static String reverseString(String str) {
    char[] chars = str.toCharArray();
    int l = 0, r = chars.length - 1;
    while (l < r) {
        char temp = chars[l];
        chars[l] = chars[r];
        chars[r] = temp;
        l++; r--;
    }
    return new String(chars);
}

// Check Anagram
public static boolean isAnagram(String s1, String s2) {
    if (s1.length() != s2.length()) return false;
    int[] freq = new int[256];
    for (char c : s1.toLowerCase().toCharArray()) freq[c]++;
    for (char c : s2.toLowerCase().toCharArray()) freq[c]--;
    for (int f : freq) if (f != 0) return false;
    return true;
}
// "listen", "silent" → true

// Count vowels/consonants
public static void countVowelsConsonants(String str) {
    int v = 0, c = 0;
    for (char ch : str.toLowerCase().toCharArray()) {
        if (Character.isLetter(ch)) {
            if ("aeiou".indexOf(ch) >= 0) v++;
            else c++;
        }
    }
    System.out.println("Vowels: " + v + ", Consonants: " + c);
}
```

---

## Q193. Sum of Digits

```java
public static int sumOfDigits(int n) {
    n = Math.abs(n);
    int sum = 0;
    while (n > 0) {
        sum += n % 10;
        n /= 10;
    }
    return sum;
}
// Input: 1234 → 1+2+3+4 = 10

// Recursive version
public static int sumRecursive(int n) {
    return n == 0 ? 0 : n % 10 + sumRecursive(n / 10);
}
```
