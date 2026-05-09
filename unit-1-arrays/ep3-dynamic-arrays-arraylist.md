# Episode 03 — Dynamic Arrays / ArrayList

> **ByteFrames · Unit 1 · Episode 3**
> *DSA, frame by frame.*

---

## One-frame summary

A plain array has a **fixed size** — once it's full, you're stuck. A **dynamic array** (Java's `ArrayList`) keeps a **bigger backing array** behind the scenes and *grows* it whenever it fills up. The trick: when full, **allocate a new array twice as big and copy everything over**. Most appends are **O(1)**; the occasional resize is **O(n)** — but spread across all appends, the average is still **O(1)**. That's called **amortized O(1)**.

---

## 1. Why do we need dynamic arrays?

In our previous episodes on arrays we saw the rules of contiguous memory: an array's length is **decided when you create it** and can never change. That gives us O(1) random access, but it forces an awkward question:

> *"How big should I make my array?"*

Real situations:

- A chat app loads messages one by one — you don't know the count in advance.
- A user adds rows to a spreadsheet — could be 5, could be 5 million.
- A web crawler discovers links — every page found adds more URLs to visit.
- A search reads results from a database — the count comes back with the data.

Two bad workarounds developers used to fall into:

| Workaround | Problem |
|---|---|
| Allocate a "huge" array up front, e.g. `new int[1_000_000]` | Wastes memory most of the time; still breaks when the actual count is 1,000,001. |
| Allocate exactly enough each time and re-copy on every add | Each add becomes O(n) — appending a million items is O(n²). |

**Punchline of this episode:** we want the **convenience of "just keep adding"** without paying O(n) on every add. The dynamic array gives us exactly that — by growing *geometrically* (doubling the backing buffer) instead of *arithmetically* (one slot at a time).

---

## 2. The core idea — capacity vs size

A dynamic array carries **two numbers**, not one:

| Term | Meaning |
|---|---|
| **size** | How many elements the user has actually added. (`list.size()` in Java.) |
| **capacity** | How big the underlying array currently is. The "shelf space" available before another resize is needed. |

**Always:** `size <= capacity`.

```
Backing array (capacity = 8):
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │    │    │    │    │
└────┴────┴────┴────┴────┴────┴────┴────┘
  0    1    2    3    4    5    6    7

  size = 4   ←── only the first 4 slots are "in use"
  capacity = 8
  free slots = capacity − size = 4
```

The user only ever sees indices `0..size-1`. The empty slots `4..7` exist in memory but are off-limits to the user — they're the runway for future appends.

Java's `ArrayList` does **not** expose `capacity` directly (the field is internal). It exposes `size()` and `isEmpty()`. That's deliberate — capacity is an implementation detail, but understanding it is the whole point of this episode.

---

## 3. How dynamic arrays work

### 3.1 Append (the headline operation)

Appending an element is simple **as long as there's room**:

```java
public void add(int x) {
    if (size == capacity) {
        resize();                 // grow the backing array first
    }
    arr[size] = x;
    size++;
}
```

Two cases happen in real life:

- **Fast path** (most appends): `size < capacity` → just write to `arr[size]` and bump `size`. **O(1)**.
- **Slow path** (occasional): `size == capacity` → allocate a new bigger array, copy every existing element across, then do the write. **O(n)** for that single operation.

#### Step-by-step trace — appending into spare capacity

Starting state: `size=4, capacity=8`.

```
Frame 1 — initial state:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ ·· │ ·· │ ·· │ ·· │   capacity = 8
└────┴────┴────┴────┴────┴────┴────┴────┘   size     = 4
  0    1    2    3    4    5    6    7

Call: list.add(50)
  size (4) < capacity (8)   →   no resize needed   →   FAST PATH
```

```
Frame 2 — after writing arr[size]:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ ·· │ ·· │ ·· │
└────┴────┴────┴────┴────┴────┴────┴────┘
  0    1    2    3    4    5    6    7
                       ↑
                       new element written at index size = 4

  size++   →   size = 5
  capacity = 8 (unchanged)
  cost     = O(1)
```

That's the cheap append. Most appends in real code look like this.

---

### 3.2 The resize — what actually happens when full

#### The growth strategy: **doubling**

When `size == capacity` and another `add` arrives, the dynamic array:

1. Allocates a brand-new backing array of size `2 * capacity` (Java's `ArrayList` actually uses `oldCapacity + (oldCapacity >> 1)` ≈ **1.5×**, but conceptually it's still geometric — we'll use 2× for clean math).
2. **Copies all `size` existing elements** from the old array into the new one.
3. Updates the internal reference to point at the new array. The old array is now garbage.
4. Performs the actual write that triggered all this.

```java
private void resize() {
    int newCap = capacity * 2;
    int[] bigger = new int[newCap];
    for (int i = 0; i < size; i++) {       // ← this is the O(n) cost
        bigger[i] = arr[i];
    }
    arr = bigger;
    capacity = newCap;
}
```

#### Step-by-step trace — append that triggers a resize

Starting state: `size=4, capacity=4` (full).

```
Frame 1 — backing array is FULL:
┌────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │     capacity = 4
└────┴────┴────┴────┘     size     = 4   (no spare slots!)
  0    1    2    3

Call: list.add(50)
  size (4) == capacity (4)   →   RESIZE FIRST
```

```
Frame 2 — allocate new array of capacity 8 (doubling):
old: ┌────┬────┬────┬────┐
     │ 10 │ 20 │ 30 │ 40 │       capacity_old = 4
     └────┴────┴────┴────┘
       0    1    2    3

new: ┌────┬────┬────┬────┬────┬────┬────┬────┐
     │ ·· │ ·· │ ·· │ ·· │ ·· │ ·· │ ·· │ ·· │   capacity_new = 8
     └────┴────┴────┴────┴────┴────┴────┴────┘
       0    1    2    3    4    5    6    7
```

```
Frame 3 — copy every existing element across (this is the O(n) part):
old: ┌────┬────┬────┬────┐
     │ 10 │ 20 │ 30 │ 40 │
     └─┬──┴─┬──┴─┬──┴─┬──┘
       │    │    │    │
       ▼    ▼    ▼    ▼      ← n copies (here n = 4)
     ┌────┬────┬────┬────┬────┬────┬────┬────┐
new: │ 10 │ 20 │ 30 │ 40 │ ·· │ ·· │ ·· │ ·· │
     └────┴────┴────┴────┴────┴────┴────┴────┘
       0    1    2    3    4    5    6    7
```

```
Frame 4 — drop the old array, point at the new one, then do the write:
                         (old array → garbage collector)

     ┌────┬────┬────┬────┬────┬────┬────┬────┐
arr: │ 10 │ 20 │ 30 │ 40 │ 50 │ ·· │ ·· │ ·· │     capacity = 8
     └────┴────┴────┴────┴────┴────┴────┴────┘     size     = 5
       0    1    2    3    4    5    6    7
                              ↑
                              the add(50) finally lands

  cost of THIS append = O(n)   (the n copies during resize)
```

The point: **the resize is rare**, but when it hits, it's expensive. The next 4 appends will all be O(1) again, until size hits 8 — then we double to 16, copy 8 elements once, and back to fast appends.

#### Why doubling and not "just add 1 more"?

| Growth rule | Cost of n appends | Why |
|---|---|---|
| Add 1 slot every time it fills | **O(n²)** | Every append from the 2nd onward triggers a copy. Copies: 1 + 2 + 3 + … + n = n(n+1)/2. |
| Add a fixed chunk (e.g. +10) | **O(n²)** | Same shape, smaller constant. Still quadratic. |
| **Double the capacity** | **O(n) total** → **O(1) amortized** per append | Resizes are spaced exponentially apart, so total copy work is bounded by 2n. |

**Doubling is the secret.** It turns a potential O(n²) catastrophe into linear total work.

#### Doubling growth table

Watch how few resizes you actually pay for, even at huge sizes:

| Append # that triggers a resize | Old capacity | New capacity | Elements copied |
|---|---|---|---|
| 1 | 0 | 1 | 0 |
| 2 | 1 | 2 | 1 |
| 3 | 2 | 4 | 2 |
| 5 | 4 | 8 | 4 |
| 9 | 8 | 16 | 8 |
| 17 | 16 | 32 | 16 |
| 33 | 32 | 64 | 32 |
| 65 | 64 | 128 | 64 |
| … | … | … | … |
| 524,289 | 524,288 | 1,048,576 | 524,288 |

Total copies after 1,048,576 appends = `1 + 2 + 4 + 8 + … + 524,288 ≈ 2 × 1,048,576 = ~2n`. That's **O(n) total work for n appends** — about 2 unit-cost operations per append on average.

---

### 3.3 Amortized analysis — why "average O(1)" really holds

"Amortized" means *averaged over a long sequence of operations*. A single resize is expensive, but the cost is "paid back" by the cheap appends around it.

#### The accountant argument (informal)

Imagine charging **3 units** per append:
- 1 unit pays for the actual write.
- 2 units go into a "savings jar" attached to that element.

When the array doubles from capacity `c` to `2c`, we need to copy `c` elements. But **every one of those c elements has 2 units saved**, accumulated since the last resize. Total savings = `2c` — exactly enough to fund the `c` copies, with the leftover paying for the *next* resize.

So no matter how long you append, the per-append cost stays **bounded by 3** — i.e. **O(1) amortized**.

#### The cost-of-n-appends argument (formal-ish)

Let's count the actual work to do n appends starting from capacity 1:

```
n appends, doubling whenever full:

  copies during resize at append #2   :    1
  copies during resize at append #3   :    2
  copies during resize at append #5   :    4
  copies during resize at append #9   :    8
  copies during resize at append #17  :   16
  …
  copies during the LAST resize       :  ≤ n

Total copies   ≤ 1 + 2 + 4 + 8 + … + n
               = 2n − 1               (geometric series)
               ≤ 2n                   →   O(n)

Plus n writes (one per append)        →   n        →   O(n)

Grand total work over n appends       →   O(n)
Average per append                    →   O(1)
```

That's the formal version of "amortized O(1) append".

#### What "amortized O(1)" does NOT mean

- It does **not** mean every append is O(1). The resize ones are still O(n).
- It does **not** make ArrayList safe for **hard real-time** systems where every individual operation must finish in bounded time. A robot arm's control loop should NOT be appending to an `ArrayList` mid-tick.
- It **does** mean: across many appends, the total work — and the average — behaves like O(1) per append.

---

### 3.4 Other operations on a dynamic array

Once you grasp resize, the rest is just "the array operations from our previous episode, on the front portion of size `size`."

| Operation | Cost | Notes |
|---|---|---|
| `get(i)` | **O(1)** | Just `arr[i]`. Indexing into the backing array is direct. |
| `set(i, x)` | **O(1)** | Same — direct write. No size change. |
| `add(x)` (append at end) | **O(1) amortized** | Most appends are O(1); occasional O(n) resize. |
| `add(i, x)` (insert at index i) | **O(n)** | Must shift `size − i` elements right. Plus possible resize. |
| `remove(i)` | **O(n)** | Must shift `size − i − 1` elements left. Capacity does NOT shrink in Java's `ArrayList`. |
| `contains(x)` / `indexOf(x)` | **O(n)** | Linear scan (the array isn't sorted). |
| `size()` | **O(1)** | Just returns the stored counter. |

#### Insert in the middle — quick trace

`list = [10, 20, 40, 50]`, `size=4`, `capacity=8`. Call `list.add(2, 30)` — insert 30 at index 2.

```
Frame 1 — before:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 40 │ 50 │ ·· │ ·· │ ·· │ ·· │   size = 4
└────┴────┴────┴────┴────┴────┴────┴────┘
  0    1    2    3    4    5    6    7
            ↑ insert here

Frame 2 — shift indices 2..3 one slot right:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ ·· │ 40 │ 50 │ ·· │ ·· │ ·· │
└────┴────┴────┴────┴────┴────┴────┴────┘
  0    1    2    3    4    5    6    7
            ▲ ── shift ── ▲

Frame 3 — write 30 into the freed slot, bump size:
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │ ·· │ ·· │ ·· │   size = 5
└────┴────┴────┴────┴────┴────┴────┴────┘
  0    1    2    3    4    5    6    7

Total work: 2 shifts + 1 write   →   O(n) in the worst case (insert at index 0 = shift everything).
```

Insert/remove in the middle are exactly the array shifts we saw in our previous episode on arrays — `ArrayList` does not magically make them faster.

---

### 3.5 Java's `ArrayList` in practice

```java
import java.util.ArrayList;

ArrayList<Integer> list = new ArrayList<>();    // capacity ~10 by default
list.add(10);                                   // size=1
list.add(20);                                   // size=2
list.add(30);                                   // size=3

list.get(1);          // → 20      O(1)
list.set(1, 99);      // size unchanged, slot 1 = 99
list.add(1, 200);     // insert 200 at index 1, shifts right   O(n)
list.remove(0);       // remove index 0, shifts left           O(n)
list.size();          // → number of stored elements           O(1)
list.isEmpty();       // → false
list.contains(99);    // linear scan                           O(n)
```

#### Pre-sizing — a free perf win

If you know roughly how many elements you'll add, **tell the constructor**:

```java
ArrayList<Integer> list = new ArrayList<>(1_000_000);   // initial capacity hint
for (int i = 0; i < 1_000_000; i++) {
    list.add(i);                                        // zero resizes
}
```

vs.

```java
ArrayList<Integer> list = new ArrayList<>();            // starts at 10
for (int i = 0; i < 1_000_000; i++) {
    list.add(i);                                        // ~20 resizes happen
}
```

Both are correct. Both are O(n) total. But the first one **never copies a single element** because there's never a resize. The second one ends up copying ~2,000,000 ints across its lifetime. For hot loops, the hint matters.

#### Shrinking — `trimToSize()`

If you finished filling an `ArrayList` and the capacity is much bigger than the size, you can reclaim the slack:

```java
list.trimToSize();   // shrinks the backing array down to exactly size
```

`ArrayList` does **not** auto-shrink on `remove`. If you build up to a million elements then remove 999,000, the backing array is still capacity ~1,000,000 until you call `trimToSize()`. That's a memory leak shape worth knowing.

---

### 3.6 Classic gotchas

- **`capacity ≠ size`.** A capacity-100 ArrayList with one element stored is `size() == 1`, not 100. Looping `for (int i = 0; i < arr.length; i++)` on the backing array would walk over 99 garbage slots.
- **Hidden copy cost on resize.** `add` looks O(1) but every doubling triggers a full copy. For latency-sensitive code, pre-size or use a different structure.
- **Insert / remove in the middle is O(n).** `ArrayList` is a *random-access* structure, not a fast-mutate-anywhere structure. (Linked lists fix that — Unit 3.)
- **Remove does not shrink capacity.** Memory stays held until `trimToSize()` or the object is dropped.
- **Iterator invalidation.** Modifying an ArrayList while iterating it (other than via `iterator.remove()`) throws `ConcurrentModificationException`. Resize moves the backing array — any held reference to the old one becomes stale.
- **Boxing overhead.** `ArrayList<Integer>` stores `Integer` objects, not raw `int`s. For hot numeric loops, prefer `int[]` or a primitive-collection library.
- **Array doubling vs Java's 1.5×.** Real `ArrayList` grows by ~50% (`oldCap + (oldCap >> 1)`), not 2×. The amortized analysis still works the same way — *any* geometric growth factor `> 1` gives O(1) amortized appends; the constant differs.

---

### 3.7 Decision tree — array vs ArrayList?

```
Do I know the exact size up front, and is it fixed?
├── Yes  →  Use a plain array (int[], Object[]).  Less overhead, no boxing.
└── No   →  Will the size change at runtime?
            ├── Yes (mostly appending at the end)
            │       →  ArrayList.  O(1) amortized append.
            ├── Yes (lots of inserts/removes in the middle/front)
            │       →  See Unit 3 (LinkedList) — O(1) at the ends, O(1) at a known node.
            └── Yes (lots of "is this in the collection?" lookups)
                    →  See Unit 5 (HashSet/HashMap) — O(1) average lookup.
```

---

## 4. Real World

- **Java collections framework.** `ArrayList`, `Vector`, `Stack` are all dynamic arrays under the hood.
- **C++ `std::vector`.** Same idea — geometric growth, amortized O(1) push_back.
- **Python `list`.** A dynamic array of pointers. Growth factor ~1.125 after 8 elements (CPython implementation detail) but the amortized O(1) story is identical.
- **JavaScript arrays.** Engines like V8 internally use dynamic arrays for "fast" element storage modes.
- **Game engines** loading entities, particles, or projectiles per frame — known peak counts → `Reserve(N)` (C++) / `EnsureCapacity(N)` (C#) / `ArrayList(N)` (Java) avoids mid-frame resizes that would cause hitches.
- **Logging frameworks.** Buffering log entries before flush — append-heavy workload, dynamic array is a perfect fit.
- **Database query result builders.** Rows arrive one at a time; the count is known only when the query ends.
- **Parsers / lexers.** Tokens are produced sequentially as a file is read.

The pattern: **dynamic arrays shine when the workload is "lots of appends, mostly random reads, occasional resize tolerated."**

---

## 5. Key Takeaways

- A **dynamic array** carries two numbers: **size** (in-use elements) and **capacity** (allocated slots). Always `size <= capacity`.
- **Append fast path:** spare capacity exists → write and bump size → **O(1)**.
- **Append slow path:** array is full → allocate a bigger array (typically 2× or 1.5×), copy all `n` elements, then write → **O(n)** for that single append.
- **Geometric growth** (doubling, or any factor > 1) is what turns repeated appends into **O(1) amortized** instead of **O(n²)**.
- The accountant view: charge 3 units per append; 2 saved per element → enough credit to fund every future doubling. Total work for n appends = O(n).
- **Java's `ArrayList`** uses ~1.5× growth. Same amortized O(1) story, different constant.
- **Other operations:** `get`/`set`/`size` are **O(1)**; **insert / remove in the middle** are **O(n)** (shifting).
- **Pre-size** when the rough count is known (`new ArrayList<>(N)`) — avoids resizes entirely.
- **`remove` does NOT shrink capacity** in Java's `ArrayList`. Use `trimToSize()` to reclaim.
- **Capacity is an internal detail.** `ArrayList.size()` is the user-facing length. Don't loop over the backing array — only the first `size` slots are valid.
- Amortized O(1) ≠ guaranteed O(1). For hard real-time, pre-size or pick a different structure.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
