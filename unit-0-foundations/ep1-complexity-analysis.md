# Episode 01 — Time & Space Complexity

> **ByteFrames · Unit 0 · Episode 1**
> *DSA, frame by frame.*

---

## One-frame summary

**Complexity analysis** answers two questions about an algorithm: *how much time does it take* and *how much memory does it use*, expressed as a function of the input size `n`. We use **asymptotic notations** — Big O (worst case), Big Omega (best case), Big Theta (tight bound) — to describe how that cost grows as `n` grows.

The whole point: pick algorithms whose cost grows **slowly** as data grows.

---

## 1. Why Complexity Matters

You cannot judge an algorithm by how fast it runs on **your** machine with **your** small input. What matters is how it scales when the input gets large.

### The brutal truth in numbers

Suppose your CPU does **1 billion operations per second**. Here is how long an algorithm takes on input size `n`, depending on its complexity:

| n         | O(log n) | O(n)        | O(n log n)  | O(n²)              | O(2ⁿ)               |
|-----------|----------|-------------|-------------|--------------------|---------------------|
| 10        | < 1 µs   | 10 ns       | 33 ns       | 100 ns             | 1 µs                |
| 100       | < 1 µs   | 100 ns      | 664 ns      | 10 µs              | ~10¹³ years (!)     |
| 1,000     | < 1 µs   | 1 µs        | 10 µs       | 1 ms               | far past heat death |
| 10,000    | < 1 µs   | 10 µs       | 133 µs      | 100 ms             | impossible          |
| 1,000,000 | < 1 µs   | 1 ms        | 20 ms       | **~16 minutes**    | impossible          |

> **The takeaway:** at `n = 1,000,000`, an O(n) algorithm finishes in 1 ms while an O(n²) algorithm takes 16 minutes. Same machine. Same problem. Different algorithm.

### Why we don't just "test it"

Testing on small inputs hides the problem. An O(n²) algorithm feels instant on `n = 100`. It collapses on `n = 1,000,000`. Complexity analysis lets us **predict** the collapse before we hit production data.

---

## 2. Time Complexity

**Time complexity** = how the **number of basic operations** an algorithm performs grows as the input size grows.

We don't measure in seconds — seconds depend on hardware, language, compiler. We count **operations** (comparisons, assignments, arithmetic) and express the count as a function of `n`.

### Example

```java
int sum = 0;                           // 1 op
for (int i = 0; i < n; i++) {          // runs n times
    sum += arr[i];                     // 1 op per iteration
}
return sum;                            // 1 op
```

Total operations: `1 + n + 1 = n + 2` → **O(n)**.

We drop the `+ 2` and the constant factor because as `n` grows, the `n` term dominates everything else.

---

## 3. Space Complexity

**Space complexity** = how the **extra memory** an algorithm uses grows as the input size grows.

Note: we count **extra** (auxiliary) memory, *not* the input itself.

### Example — O(1) space

```java
int sum = 0;
for (int i = 0; i < n; i++) sum += arr[i];
return sum;
```

Uses only one variable (`sum`) regardless of `n`. **O(1) space**.

### Example — O(n) space

```java
int[] doubled = new int[n];
for (int i = 0; i < n; i++) doubled[i] = arr[i] * 2;
return doubled;
```

Allocates a new array of size `n`. **O(n) space**.

### Hidden space — recursion call stack

```java
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);     // recursive call
}
```

Looks like O(1) space, but each recursive call adds a stack frame. `factorial(n)` builds `n` stack frames before unwinding → **O(n) space**.

> **Always count recursion stack depth as part of space complexity.**

---

## 4. The Three Asymptotic Notations

These three describe **how a function grows** as `n` grows. They are bounds, not exact counts.

### 4.1 Big O — `O(f(n))` — Upper Bound (Worst Case)

> "The algorithm's cost grows **no faster than** `f(n)`."

This is the **worst-case guarantee**. If we say a function is O(n²), it means the cost will not exceed some constant times n² for large enough n.

```
Cost
  ^
  |              c · f(n)        ← upper bound
  |          .  /
  |        .   /
  |      .    / actual cost
  |    .  ___/
  |  ._-/
  |_/____________________________→ n
```

**Used for:** worst-case guarantees. The most common notation in interviews and practice.

### 4.2 Big Omega — `Ω(f(n))` — Lower Bound (Best Case)

> "The algorithm's cost grows **at least as fast as** `f(n)`."

This is the **best-case floor**. The cost cannot drop below some constant times f(n) for large enough n.

**Example:** linear search has Ω(1) — best case is finding the target at index 0 in 1 comparison.

### 4.3 Big Theta — `Θ(f(n))` — Tight Bound

> "The algorithm's cost grows **exactly as fast as** `f(n)`."

When best case and worst case have the same growth rate, we use Θ. The cost is sandwiched between two constant multiples of f(n).

**Example:** traversing an array to compute the sum is Θ(n) — best, average, and worst all visit every element.

### 4.4 Quick comparison

| Notation | Meaning                | Sentence                                     |
|----------|------------------------|----------------------------------------------|
| O(f(n))  | upper bound (≤)        | "grows **no faster than** f(n)"              |
| Ω(f(n))  | lower bound (≥)        | "grows **at least as fast as** f(n)"         |
| Θ(f(n))  | tight bound (=)        | "grows **exactly like** f(n)"                |

### 4.5 The same algorithm in all three

Linear search on an unsorted array of size `n`:

| Case          | Complexity | When it happens                              |
|---------------|------------|----------------------------------------------|
| Best case     | Ω(1)       | Target is at index 0                         |
| Average case  | Θ(n)       | Target is somewhere in the middle on average |
| Worst case    | O(n)       | Target is at the end, or absent              |

> **In daily use, "complexity" almost always means Big O (worst case).** Whenever someone says "this is O(n)", they mean the worst case.

---

## 5. Common Growth Classes

These are the names you will see again and again. Sorted from fastest (best) to slowest (worst):

| Class         | Name             | Example algorithm                             |
|---------------|------------------|-----------------------------------------------|
| O(1)          | Constant         | Array access by index                         |
| O(log n)      | Logarithmic      | Binary search                                 |
| O(n)          | Linear           | Linear search, single loop, traversal         |
| O(n log n)    | Linearithmic     | Merge sort, quick sort (avg), heap sort       |
| O(n²)         | Quadratic        | Bubble sort, nested loop on same array        |
| O(n³)         | Cubic            | Naive matrix multiplication                   |
| O(2ⁿ)         | Exponential      | Naive recursive Fibonacci, brute-force subsets |
| O(n!)         | Factorial        | Brute-force traveling salesman, permutations  |

### Growth visualization

```
Cost (operations)
    ^
    |                                 O(n!) O(2^n)
    |                                  /    /
    |                                 /    /
    |                                /    /        O(n^2)
    |                               /    /        /
    |                              /    /        /
    |                             /    /        /
    |                            /    /        /        O(n log n)
    |                           /    /        /        /  O(n)
    |                          /    /        /        / /
    |                         /    /        /        //
    |                        /    /       _/_____   //          O(log n)
    |                       /    /     __/      \__//______
    |                      /    /  __/                \____   O(1)
    |____________________/_____/_/_______________________________
                                                                n
```

### Numerical comparison

For `n = 1,000,000`:

| Class         | Operations           | Real-world feel        |
|---------------|----------------------|------------------------|
| O(1)          | 1                    | instant                |
| O(log n)      | ~20                  | instant                |
| O(n)          | 1,000,000            | a few milliseconds     |
| O(n log n)    | ~20,000,000          | tens of ms             |
| O(n²)         | 1,000,000,000,000    | ~16 minutes            |
| O(2ⁿ)         | 10^301,030           | longer than the universe |

---

## 6. How to Analyze Code

A few mechanical rules will get you 90% of the way.

### Rule 1 — A single loop over `n` items is O(n)

```java
for (int i = 0; i < n; i++) {
    doSomething();         // O(1) inside
}
// Total: O(n)
```

### Rule 2 — Nested loops multiply

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < n; j++) {
        doSomething();
    }
}
// Total: O(n × n) = O(n^2)
```

If the inner loop runs `m` times instead of `n`:

```java
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) { ... }
}
// Total: O(n × m)
```

### Rule 3 — Halving the input each step is O(log n)

```java
while (n > 1) {
    n = n / 2;             // halves each iteration
}
// Total: O(log n)
```

This is the binary-search pattern: **whenever you cut the problem in half, you get a log n term**.

### Rule 4 — Sequential blocks add (then drop the smaller one)

```java
for (int i = 0; i < n; i++) { ... }    // O(n)
for (int i = 0; i < n; i++) {          // O(n^2)
    for (int j = 0; j < n; j++) { ... }
}
// Total: O(n + n^2) = O(n^2)   ← keep the dominant term
```

### Rule 5 — Drop constants

```java
for (int i = 0; i < n; i++) { ... }    // O(n)
for (int i = 0; i < n; i++) { ... }    // O(n)
for (int i = 0; i < n; i++) { ... }    // O(n)
// Total: O(3n) = O(n)   ← constants don't matter asymptotically
```

`O(3n)` and `O(n)` are the same. `O(n / 2)` is also `O(n)`. Constants vanish.

### Rule 6 — Drop lower-order terms

`O(n² + n + 100)` simplifies to **O(n²)**. Only the term that grows fastest matters as `n → ∞`.

### Rule 7 — Recursion: count the call tree

```java
int fib(int n) {
    if (n < 2) return n;
    return fib(n - 1) + fib(n - 2);    // 2 recursive calls
}
```

Each call branches into 2 calls, depth is `n`. Total calls ≈ `2ⁿ`. **O(2ⁿ)** time.

```java
int factorial(int n) {
    if (n == 0) return 1;
    return n * factorial(n - 1);       // 1 recursive call, depth n
}
```

One call per level, depth `n`. **O(n)** time, **O(n)** stack space.

---

## 7. Common Beginner Mistakes

- **Forgetting the call stack** in recursion. A recursive function with no extra arrays still uses **O(depth)** space.
- **Confusing best and worst case.** "It's O(1) in the best case" is technically Ω(1). Big O is *upper bound*, so saying "best case O(n)" is correct but weak — you usually want Big Omega.
- **Counting input size as space.** Input doesn't count. We measure *extra* memory.
- **Hidden costs in library calls.** `String s = a + b + c + d;` in a loop can be O(n²) silently.
- **Comparing same-Big-O algorithms.** Two O(n) algorithms can differ 10× in real speed because of hidden constants. Big O ignores constants by design.
- **Over-optimizing.** O(n log n) usually beats O(n²), but only past a certain `n`. For tiny inputs, the simpler algorithm wins.
- **Confusing log bases.** `log₂ n`, `log₁₀ n`, `ln n` — they all differ by a constant factor, so they're all just **O(log n)**.

---

## 8. Master Reference Table

| Algorithm / Operation                  | Time          | Space        |
|----------------------------------------|---------------|--------------|
| Array access by index                  | O(1)          | O(1)         |
| Linear search                          | O(n)          | O(1)         |
| Binary search (sorted array)           | O(log n)      | O(1)         |
| Bubble / Insertion / Selection sort    | O(n²)         | O(1)         |
| Merge sort                             | O(n log n)    | O(n)         |
| Quick sort (average)                   | O(n log n)    | O(log n)     |
| Quick sort (worst)                     | O(n²)         | O(n)         |
| Heap sort                              | O(n log n)    | O(1)         |
| Hash table lookup (average)            | O(1)          | O(n)         |
| BFS / DFS on graph                     | O(V + E)      | O(V)         |
| Dijkstra (with min-heap)               | O((V+E) log V)| O(V)         |
| Recursive Fibonacci (naive)            | O(2ⁿ)         | O(n)         |
| Memoized Fibonacci                     | O(n)          | O(n)         |

(You'll meet each of these in later episodes — don't worry about memorizing now.)

---

## 9. Quick Revision Bullets

> Last-minute skim. Go.

- **Time complexity** = how operation count grows with `n`.
- **Space complexity** = how extra memory grows with `n` (input doesn't count, recursion stack does).
- **Big O (`O`)** = upper bound (worst case).
- **Big Omega (`Ω`)** = lower bound (best case).
- **Big Theta (`Θ`)** = tight bound (best == worst growth).
- In daily use, "complexity" = **Big O** (worst case).
- Sorted by speed: **O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(2ⁿ) < O(n!)**.
- **Single loop = O(n).** Nested loop = **O(n²)**.
- **Halving = O(log n).** Anytime you divide the problem by 2, expect a log.
- **Drop constants:** O(3n) = O(n).
- **Drop lower terms:** O(n² + n) = O(n²).
- For `n = 10⁶`: O(n) is milliseconds, O(n²) is **16 minutes**.
- O(2ⁿ) and O(n!) are basically "do not use" for any n above ~30.
- **Recursion → count the call tree** for time, **stack depth** for space.
- Two algorithms with the same Big O can still differ in real-world speed — constants matter in practice.
- **Always state the case:** "This is O(n²) **worst case**" is clearer than just "O(n²)".

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
