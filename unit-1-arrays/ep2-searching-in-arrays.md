# Episode 02 — Searching in Arrays

> **ByteFrames · Unit 1 · Episode 2**
> *DSA, frame by frame.*

---

## One-frame summary

When data is **unsorted**, you have no choice but to look at every element — that's **linear search**, **O(n)**. When data is **sorted**, you can keep halving the problem — that's **binary search**, **O(log n)**. For 1,000,000 items: linear ≈ 1,000,000 checks, binary ≈ 20.

---

## 1. Why do we need search?

In our previous episode on arrays we learned how arrays store data and how to walk through them. The natural next question is: *given an array and a value, how do we find that value's position?*

Real situations:

- "Is this email in our user list?"
- "What index holds the contact named *Priya*?"
- "Did this person enter the right password?"
- "Is **47** somewhere inside `[3, 7, 18, 47, 92, 105]`?"

Without an algorithm, the computer has no clue where to look. We need a method.

**Punchline of this episode:** *how the data is arranged decides which algorithm we can use.* Sorted data unlocks something much faster than unsorted. We'll meet two algorithms, see exactly how each one moves through an array, and learn five super-useful binary-search variants.

---

## 2. Two strategies

### 2.1 Linear search (a.k.a. sequential search)

Walk through the array element by element. Compare each one to the target. Stop when you find a match or reach the end.

- Works on **any** array — sorted or not.
- Slow when the array is large.

### 2.2 Binary search

If the array is **sorted**, jump to the middle. If the middle equals the target, done. If the target is smaller, search the left half; if larger, search the right half. Repeat.

- **Requires a sorted array.**
- Each step throws away half the remaining work — incredibly fast.

### 2.3 Quick comparison

| | Linear | Binary |
|---|---|---|
| Sorted required? | No | **Yes** |
| Best case | O(1) | O(1) |
| Worst case | **O(n)** | **O(log n)** |
| Space | O(1) | O(1) iterative · O(log n) recursive |
| Implementation | Trivial | Slightly tricky (off-by-ones) |

For n = 1,000,000:
- Linear ≈ 1,000,000 comparisons in the worst case.
- Binary ≈ 20 comparisons in the worst case.

That's a **50,000× speedup** — and it's why sorting often pays for itself.

---

## 3. How they work

### 3.1 Linear search

The simplest approach: a for-loop that checks every element.

#### Code (Java)

```java
public static int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) {
            return i;            // found at index i
        }
    }
    return -1;                   // not found
}
```

#### Step-by-step trace

We'll search for **target = 27** in this array:

```
Index:    0   1   2   3   4   5   6   7
Value:  [ 3][ 7][ 9][12][22][27][41][53]
```

**Frame 1 — i = 0:**

```
Comparisons so far: 1

  ↓ cursor
[ 3][ 7][ 9][12][22][27][41][53]
  ↑
  arr[0] = 3   →   3 ≠ 27   →   move right
```

**Frame 2 — i = 1:**

```
Comparisons so far: 2

       ↓ cursor
[ 3][ 7][ 9][12][22][27][41][53]
       ↑
       arr[1] = 7   →   7 ≠ 27   →   move right
```

**Frame 3 — i = 2:**

```
Comparisons so far: 3

            ↓ cursor
[ 3][ 7][ 9][12][22][27][41][53]
            ↑
            arr[2] = 9   →   9 ≠ 27   →   move right
```

**Frame 4 — i = 3:**

```
Comparisons so far: 4

                 ↓ cursor
[ 3][ 7][ 9][12][22][27][41][53]
                 ↑
                 arr[3] = 12   →   12 ≠ 27   →   move right
```

**Frame 5 — i = 4:**

```
Comparisons so far: 5

                      ↓ cursor
[ 3][ 7][ 9][12][22][27][41][53]
                      ↑
                      arr[4] = 22   →   22 ≠ 27   →   move right
```

**Frame 6 — i = 5:**

```
Comparisons so far: 6

                           ↓ cursor
[ 3][ 7][ 9][12][22][27][41][53]
                           ↑
                           arr[5] = 27   →   MATCH ✓   →   return 5
```

**Result:** target 27 found at index 5 in **6 comparisons**.

#### Complexity analysis

- **Best case:** target is at index 0 → 1 comparison → **O(1)**. (Don't count on it.)
- **Worst case:** target is at the last index, or not present → **n comparisons** → **O(n)**.
- **Average case:** ~n/2 comparisons → still **O(n)** (we drop the constant).
- **Space:** **O(1)** — no extra memory beyond a loop counter.

#### When to use linear search

- The array is unsorted.
- The array is small (say, n ≤ 100). Simplicity wins.
- You only need to search once, so sorting first wouldn't pay off.

---

### 3.2 Binary search (iterative)

**Prerequisite:** the array MUST be sorted in ascending order. We'll see what breaks if it isn't (spoiler: nonsense answers).

#### The idea

Keep three index pointers: `lo`, `mid`, `hi`. Compare `arr[mid]` to the target. Three outcomes:

- `arr[mid] == target` → **found**, return `mid`.
- `arr[mid] < target` → target must be in the right half → set `lo = mid + 1`.
- `arr[mid] > target` → target must be in the left half → set `hi = mid - 1`.

Repeat until found or `lo > hi` (window empty → not found).

#### Code (Java)

```java
public static int binarySearch(int[] arr, int target) {
    int lo = 0;
    int hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;          // safe from int overflow
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return -1;
}
```

#### Beginner traps to know

- **Trap #1 — overflow:** `int mid = (lo + hi) / 2;` can overflow when `lo + hi > Integer.MAX_VALUE`. Use `lo + (hi - lo) / 2` — same result, safe.
- **Trap #2 — loop condition:** the loop must be `lo <= hi`, NOT `lo < hi`. When `lo == hi` there's still one element to check.
- **Trap #3 — `mid + 1` and `mid - 1`:** without these, the window can stop shrinking and the loop becomes infinite.

#### Step-by-step trace

We'll search for **target = 47** in a sorted array of 16 elements:

```
Index:    0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
Value:  [ 3][ 7][ 9][12][15][18][22][27][30][35][41][47][53][68][82][91]
```

**Frame 1 — lo = 0, hi = 15, mid = 7:**

```
Comparisons so far: 1

[ 3][ 7][ 9][12][15][18][22][27][30][35][41][47][53][68][82][91]
 └lo──────────────[mid▼]──────────────────────────────────────hi┘

arr[7] = 27   →   27 < 47   →   target is in the RIGHT half   →   lo = mid + 1 = 8
```

**Frame 2 — lo = 8, hi = 15, mid = 11:**

```
Comparisons so far: 2

░░  ░░  ░░  ░░  ░░  ░░  ░░  ░░  [30][35][41][47][53][68][82][91]
(left half eliminated — won't be looked at again)
                                 └lo──────[mid▼]──────────────hi┘

arr[11] = 47   →   MATCH ✓   →   return 11
```

**Result:** target 47 found at index 11 in **2 comparisons**.

For 16 elements, binary search needs at most `⌈log₂(16)⌉ = 4` comparisons regardless of where the target sits. Linear search would need up to 16.

#### Why it's O(log n)

Every comparison cuts the search space in half. Starting from n elements, after k halvings the space is `n / 2^k`. We're done when `n / 2^k ≤ 1`, i.e. `k = log₂(n)`.

| n | Worst-case comparisons (binary) | Worst-case comparisons (linear) |
|---|---|---|
| 16 | 4 | 16 |
| 1,000 | 10 | 1,000 |
| 1,000,000 | 20 | 1,000,000 |
| 1,000,000,000 | 30 | 1,000,000,000 |

That's the magic of `log`.

---

### 3.3 Binary search (recursive)

Same algorithm, written so the function calls itself on a smaller window instead of using a `while` loop.

> **Heads-up on recursion:** if you're new to it, the key idea is *"a function that calls itself with a smaller version of the problem."* We'll go deeper when we hit Trees in Unit 6. For now, just see how the same binary-search logic re-shapes when we let the function recurse.

#### Code (Java)

```java
public static int binarySearchRec(int[] arr, int target, int lo, int hi) {
    if (lo > hi) {
        return -1;                                                  // base case: window empty
    }
    int mid = lo + (hi - lo) / 2;
    if (arr[mid] == target) {
        return mid;                                                 // base case: found
    } else if (arr[mid] < target) {
        return binarySearchRec(arr, target, mid + 1, hi);           // recurse RIGHT
    } else {
        return binarySearchRec(arr, target, lo, mid - 1);           // recurse LEFT
    }
}

// Initial call:
//   int idx = binarySearchRec(arr, 47, 0, arr.length - 1);
```

The body looks similar to the iterative version. Instead of updating `lo` / `hi` and looping, it calls itself with the new window.

#### How memory works during recursion

When a function calls itself, each call gets its own slot in memory called a **stack frame**. The frames stack up — the **top** frame is the one currently running, and the frames below are paused, waiting for their child to return.

Each frame remembers its own local variables (`lo`, `hi`, `mid`, etc.) and the spot it should resume from once the child returns.

#### Step-by-step trace with call stack

We'll trace `binarySearchRec(arr, 47, 0, 15)` on the same 16-element array.

**Step 1 — `bsRec(lo=0, hi=15)` is called.**

```
Call Stack (top = currently running):
┌────────────────────────────────────────────────────┐
│  bsRec(lo=0, hi=15)   mid=7   running              │  Frame 1
└────────────────────────────────────────────────────┘
        ↑ stack depth = 1

Array view (Frame 1's window = entire array):
[ 3][ 7][ 9][12][15][18][22][27][30][35][41][47][53][68][82][91]
 └lo──────────────[mid▼]──────────────────────────────────────hi┘

Compare arr[7]=27 with 47   →   27 < 47   →   recurse RIGHT
→ Frame 1 calls bsRec(lo=8, hi=15) and pauses, waiting for the result.
```

**Step 2 — Frame 1 calls into `bsRec(lo=8, hi=15)`.** A new frame stacks on top.

```
Call Stack (memory growing downward):
┌────────────────────────────────────────────────────┐
│  bsRec(lo=8, hi=15)   mid=11  running              │  Frame 2
├────────────────────────────────────────────────────┤
│  bsRec(lo=0, hi=15)   mid=7   paused               │  Frame 1
└────────────────────────────────────────────────────┘
        ↑ stack depth = 2

Array view (Frame 2's window only — left half is "frozen" by Frame 1):
░░  ░░  ░░  ░░  ░░  ░░  ░░  ░░  [30][35][41][47][53][68][82][91]
                                 └lo──────[mid▼]──────────────hi┘

Compare arr[11]=47 with 47   →   MATCH ✓   →   Frame 2 returns 11.
```

**Step 3 — Frame 2 returns 11. It pops off the stack.**

```
Call Stack (Frame 2 popped — memory shrinking):
┌────────────────────────────────────────────────────┐
│  bsRec(lo=0, hi=15)   receives 11 → returns 11     │  Frame 1
└────────────────────────────────────────────────────┘
        ↑ stack depth = 1

The 11 bubbles down to Frame 1's caller — the original `binarySearchRec(arr, 47, 0, 15)`
call returns 11.
```

**Step 4 — Frame 1 returns 11. Recursion done.**

```
Call Stack:
┌────────────────────────────────────────────────────┐
│            (empty — recursion complete)            │
└────────────────────────────────────────────────────┘

Final answer: 11
```

#### Memory cost of recursion

Each frame holds the function's local variables (`lo`, `hi`, `mid`, the return address, etc.) — roughly a few dozen bytes per frame. The maximum stack depth equals the maximum number of halvings: `⌈log₂(n)⌉`.

| n | Max stack depth | Approx memory used |
|---|---|---|
| 16 | 4 | ~256 bytes |
| 1,024 | 10 | ~640 bytes |
| 1,000,000 | 20 | ~1.3 KB |
| 1,000,000,000 | 30 | ~2 KB |

So recursive binary search is **O(log n) time, O(log n) space**. The iterative version is **O(log n) time, O(1) space** — it doesn't add stack frames.

For binary search this difference is tiny (log(n) frames is barely anything). But for some recursive algorithms the depth gets much larger and stack space matters — worth knowing now.

---

### 3.4 Variants — same skeleton, one tweak

Real-world arrays often have **duplicates**, or you don't want just *any* match — you want the *first* match, or *where target would slot in if you inserted it*. These variants of binary search handle those cases.

**The trick that runs through all of them:** *don't return immediately on a match. Save the index, then keep searching the appropriate half.*

We'll use this 16-element sorted array with duplicates for every variant:

```
Index:   0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
Value: [ 3][ 7][ 9][12][15][18][22][27][27][27][27][35][41][47][53][82]
```

The value **27** appears at indices **7, 8, 9, 10**. That's where the variants disagree on what to return.

---

#### 3.4.1 Standard binary search

**Definition:** return *any* index where `arr[i] == target`. Stops on the first match.

**Behavior:** already shown in section 3.2. The Java code returns the moment `arr[mid] == target`.

**Trace (target = 27):**

```
Frame 1: lo=0, hi=15, mid=7   →   arr[7] = 27   →   MATCH ✓   →   return 7
```

**Result:** 7. (It happens to be the leftmost 27 here, but only because mid landed there. With a different array length, standard binary search could return 8, 9, or 10 — any matching index is fair game.)

---

#### 3.4.2 First occurrence

**Definition:** return the *leftmost* index where `arr[i] == target`. Useful when duplicates exist and you specifically want the first one.

**Code (Java):**

```java
public static int firstOccurrence(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1, result = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) {
            result = mid;            // remember it
            hi = mid - 1;            // but keep searching LEFT for an earlier match
        } else if (arr[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return result;
}
```

**Trace (target = 27):**

```
Frame 1: lo=0, hi=15, mid=7   →   arr[7] = 27   →   match, save 7,  search LEFT   →   hi = 6
Frame 2: lo=0, hi=6,  mid=3   →   arr[3] = 12   →   12 < 27,         search RIGHT  →   lo = 4
Frame 3: lo=4, hi=6,  mid=5   →   arr[5] = 18   →   18 < 27,         search RIGHT  →   lo = 6
Frame 4: lo=6, hi=6,  mid=6   →   arr[6] = 22   →   22 < 27,         search RIGHT  →   lo = 7
                                  lo > hi   →   loop ends   →   return saved result = 7
```

**Result:** **7**. (Guaranteed leftmost — even if mid had landed elsewhere, the "search left after match" loop would still walk down to 7.)

---

#### 3.4.3 Last occurrence

**Definition:** return the *rightmost* index where `arr[i] == target`. Same idea as first occurrence, but search RIGHT after a match.

**Code (Java):**

```java
public static int lastOccurrence(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1, result = -1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) {
            result = mid;            // remember it
            lo = mid + 1;            // but keep searching RIGHT for a later match
        } else if (arr[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return result;
}
```

**Trace (target = 27):**

```
Frame 1: lo=0,  hi=15, mid=7   →   arr[7] = 27    →   match, save 7,  search RIGHT  →   lo = 8
Frame 2: lo=8,  hi=15, mid=11  →   arr[11] = 35   →   35 > 27,         search LEFT  →   hi = 10
Frame 3: lo=8,  hi=10, mid=9   →   arr[9] = 27    →   match, save 9,  search RIGHT  →   lo = 10
Frame 4: lo=10, hi=10, mid=10  →   arr[10] = 27   →   match, save 10, search RIGHT  →   lo = 11
                                   lo > hi   →   loop ends   →   return saved result = 10
```

**Result:** **10**.

---

#### 3.4.4 Lower bound

**Definition:** smallest index `i` such that `arr[i] >= target`. In plain English: *"if I were to insert target while keeping the array sorted, what's the leftmost index it could go at?"*

**Code (Java):**

```java
public static int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;          // note: hi = length, not length - 1
    while (lo < hi) {                      // note: < not <=
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) {
            lo = mid + 1;
        } else {
            hi = mid;                      // mid might be the answer — keep it in the window
        }
    }
    return lo;
}
```

**Examples on our array:**

- `lowerBound(arr, 27)` = **7**. First index where value ≥ 27.
- `lowerBound(arr, 26)` = **7**. (26 isn't present; 27 is the first value ≥ 26.)
- `lowerBound(arr, 100)` = **16** = `arr.length` (target is larger than every element).

---

#### 3.4.5 Upper bound

**Definition:** smallest index `i` such that `arr[i] > target`.

**Code (Java):**

```java
public static int upperBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    return lo;
}
```

**Example:** `upperBound(arr, 27)` = **11**. First index where value is *strictly greater* than 27.

---

#### 3.4.6 Side-by-side — Standard vs First Occurrence on the same input

```
Array:    [ 3][ 7][ 9][12][15][18][22][27][27][27][27][35][41][47][53][82]
Target:   27

────────────────────────────────────────────────────────────────────────
Standard binary search:
  Step 1: mid=7  →  arr[7] = 27  →  MATCH  →  return 7
  Total: 1 comparison.
────────────────────────────────────────────────────────────────────────
First occurrence:
  Step 1: mid=7  →  match, save 7, hi=6   ← does NOT return!
  Step 2: mid=3  →  12 < 27, lo=4
  Step 3: mid=5  →  18 < 27, lo=6
  Step 4: mid=6  →  22 < 27, lo=7
  lo > hi → return saved result = 7
  Total: 4 comparisons, but guaranteed to be the leftmost 27.
────────────────────────────────────────────────────────────────────────
```

Variants trade a few extra comparisons (still O(log n)) for a guarantee about *which* match index they return.

---

#### 3.4.7 Bonus combo — counting occurrences

A nice combo: how many times does `target` appear in the sorted array?

```java
int count = upperBound(arr, target) - lowerBound(arr, target);
```

For target = 27 in our array: `11 - 7 = 4`. ✓

Two binary searches, total `O(log n)` time — far better than scanning the whole array.

---

### 3.5 Decision tree — which search should I reach for?

```
Is the array sorted?
├── No   →  Linear search (only choice).  O(n).
└── Yes  →  Is n large enough to matter?  (rough threshold: n > 50)
            ├── No   →  Linear is fine. Simpler code.
            └── Yes  →  Need first / last / lower bound / upper bound / count?
                        ├── No   →  Standard binary search.   O(log n).
                        └── Yes  →  Use the matching variant. Still O(log n).
```

If the data is mostly unsorted but you'll search it many times → consider sorting once (`O(n log n)`) and binary-searching repeatedly (each `O(log n)`). Worth it past roughly `log(n)` searches.

---

## 4. Real World

- **Phone book / contact list** — sorted by name → binary search finds anyone in ~20 steps even with millions of contacts.
- **Database B-tree indexes** — under the hood of every SQL `WHERE id = ...` query, there's a binary-search-flavored algorithm walking a tree of sorted keys.
- **Git** — looking up a commit by its SHA prefix.
- **IDE "go to file"** — autocomplete uses lower-bound to find files starting with the prefix you're typing.
- **Compiler symbol tables** — when the compiler needs to resolve a variable name, it searches the table fast.
- **Spell-checkers** — sorted dictionary + binary search per word.
- **Network routing tables** — match an IP address to its destination using a sorted prefix list.
- **Calendar / scheduling apps** — finding the next free slot uses lower-bound on a sorted list of busy intervals.

The pattern: **whenever data is sorted (or sortable once and queried often), binary search and its variants are the go-to.**

---

## 5. Key Takeaways

- **Linear search:** O(n) worst-case, O(1) space. Works on any array. Use for unsorted or small data.
- **Binary search:** O(log n) worst-case, O(1) iterative / O(log n) recursive space. **Requires sorted array.**
- **Halving = log = the secret of fast search.** For n = 1,000,000 → 1,000,000 vs ~20 comparisons.
- **Beginner traps:**
  - `mid = (lo + hi) / 2` can overflow → use `lo + (hi - lo) / 2`.
  - Loop condition is `lo <= hi`, NOT `lo < hi`.
  - Always do `mid + 1` and `mid - 1` when shrinking — otherwise the window can stop shrinking and the loop becomes infinite.
- **Variants of binary search** (same skeleton, one tweak):
  - **First occurrence** — keep searching LEFT after a match.
  - **Last occurrence** — keep searching RIGHT after a match.
  - **Lower bound** — smallest index where `arr[i] >= target`.
  - **Upper bound** — smallest index where `arr[i] > target`.
  - **Count of target** = `upperBound - lowerBound`.
- **Recursive binary search** uses **O(log n) stack memory** (one frame per halving). Iterative uses **O(1)**. For binary search the difference is tiny, but the recursion pattern matters because we'll see it again in trees and graphs.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
