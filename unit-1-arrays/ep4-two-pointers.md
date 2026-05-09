# Episode 04 — Two Pointers

> **ByteFrames · Unit 1 · Episode 4**
> *DSA, frame by frame.*

---

## One-frame summary

Two indices instead of one. Walk them through the array — sometimes from **opposite ends converging**, sometimes **same direction at different speeds**, sometimes **one scanning while the other tracks a partition wall**. Done right, this turns a lot of nested-loop **O(n²)** problems into a clean **O(n)** with **O(1)** extra space.

---

## 1. Why?

In our previous episodes on arrays we learned how to access them, how to search them, and how to grow them. The natural next question is: *what about problems where the answer isn't at one index, but lives in a **relationship** between two?*

Real questions:

- "Does this sorted array contain two numbers that sum to 9?"
- "Is this sequence a palindrome?"
- "Move all zeroes to the end without allocating a new array."
- "Remove duplicates from a sorted array, in place, and tell me the new length."

The naive way is two nested loops — for each `i`, scan every `j > i`. That's **O(n²)**. For n = 10,000 it's 100 million comparisons. For n = 1,000,000 it's a trillion.

The **two-pointer technique** exploits *structure* in the array — sortedness, or the freedom to mutate in place — to walk through the data only once with two indices. **O(n) time, O(1) space.** Often a 100×–1,000,000× improvement.

**Punchline:** when you see a problem that *feels* like nested loops, ask "can two indices walking the array carry enough state to solve this in a single pass?" If yes, you've got a two-pointer solution.

---

## 2. The three patterns

| Pattern | Pointer setup | Movement | Classic problem |
|---|---|---|---|
| **Opposite ends** | `i = 0`, `j = n−1` | Converge — `i++` or `j−−` based on a comparison | Two-sum on sorted, reverse, palindrome, container-with-most-water |
| **Fast / slow** | Both at index 0 | Same direction; `fast` advances every step, `slow` only when a condition holds | Remove duplicates, in-place compaction |
| **Partition wall** | `slow = 0` (write head), `fast = 0` (read head) | `fast` scans; `slow` advances only when `fast` finds a "keeper" | Move zeroes, Hoare partition (preview of quick sort) |

All three share the same DNA: **two indices, one pass, O(n) work, O(1) extra memory.**

---

## 3. How they work

### 3.1 Opposite ends — two-sum on a sorted array

**Problem:** given a sorted array and a target sum, return indices `(i, j)` with `arr[i] + arr[j] == target`, or `(−1, −1)` if no such pair exists.

**The idea.** If `arr[i] + arr[j]` is *too small*, we need a bigger left number → move `i` right. If it's *too big*, we need a smaller right number → move `j` left. Each step throws away exactly one possibility.

#### Code (Java)

```java
public static int[] twoSumSorted(int[] arr, int target) {
    int i = 0;
    int j = arr.length - 1;
    while (i < j) {
        int sum = arr[i] + arr[j];
        if (sum == target) {
            return new int[]{ i, j };
        } else if (sum < target) {
            i++;                  // need a bigger left
        } else {
            j--;                  // need a smaller right
        }
    }
    return new int[]{ -1, -1 };   // no such pair
}
```

#### Step-by-step trace

Search for **target = 17** in this sorted array:

```
Index:    0   1   2   3   4   5   6   7
Value:  [ 2][ 5][ 8][ 9][12][14][17][22]
```

**Frame 1 — i = 0, j = 7:**

```
 ↓i                                    ↓j
[ 2][ 5][ 8][ 9][12][14][17][22]
 arr[0] + arr[7] = 2 + 22 = 24   →   24 > 17   →   j--
```

**Frame 2 — i = 0, j = 6:**

```
 ↓i                               ↓j
[ 2][ 5][ 8][ 9][12][14][17][22]
 arr[0] + arr[6] = 2 + 17 = 19   →   19 > 17   →   j--
```

**Frame 3 — i = 0, j = 5:**

```
 ↓i                          ↓j
[ 2][ 5][ 8][ 9][12][14][17][22]
 arr[0] + arr[5] = 2 + 14 = 16   →   16 < 17   →   i++
```

**Frame 4 — i = 1, j = 5:**

```
      ↓i                     ↓j
[ 2][ 5][ 8][ 9][12][14][17][22]
 arr[1] + arr[5] = 5 + 14 = 19   →   19 > 17   →   j--
```

**Frame 5 — i = 1, j = 4:**

```
      ↓i                ↓j
[ 2][ 5][ 8][ 9][12][14][17][22]
 arr[1] + arr[4] = 5 + 12 = 17   →   MATCH ✓   →   return [1, 4]
```

**Result:** target 17 found at indices `[1, 4]` in **5 comparisons**. Nested loops would have done up to 28 comparisons on n = 8 (worst case `n(n−1)/2`).

#### Why this works (the invariant)

At every step, the answer (if it exists) lies somewhere inside the window `[i..j]`. When `arr[i] + arr[j] < target`, no `arr[i] + arr[k]` for `k <= j` can hit the target either (every `arr[k]` is `<= arr[j]`) — so `i` is safe to discard. Symmetric argument for the other branch. Each step is an O(1) decision that eliminates at least one element. After at most n steps the window is exhausted.

#### Complexity

- **Time:** O(n) — at most n iterations, each O(1).
- **Space:** O(1) — two indices, no extra structures.
- **Requires:** sorted array. If unsorted, sorting first is O(n log n) — still beats O(n²) past n ≈ 20.

---

### 3.2 Opposite ends — reverse, palindrome check

Same pointer pattern, different work happens at each step. **Reverse** swaps `arr[i]` with `arr[j]`. **Palindrome** compares them.

#### Code (Java)

```java
public static void reverse(int[] arr) {
    int i = 0, j = arr.length - 1;
    while (i < j) {
        int tmp = arr[i];
        arr[i] = arr[j];
        arr[j] = tmp;
        i++;
        j--;
    }
}

public static boolean isPalindrome(int[] arr) {
    int i = 0, j = arr.length - 1;
    while (i < j) {
        if (arr[i] != arr[j]) return false;
        i++;
        j--;
    }
    return true;
}
```

#### Step-by-step trace — reverse `[1, 2, 3, 4, 5]`

```
Frame 1 — i=0, j=4:
[ 1][ 2][ 3][ 4][ 5]   →   swap   →   [ 5][ 2][ 3][ 4][ 1]
  ↑               ↑
  i               j

Frame 2 — i=1, j=3:
[ 5][ 2][ 3][ 4][ 1]   →   swap   →   [ 5][ 4][ 3][ 2][ 1]
       ↑     ↑
       i     j

Frame 3 — i=2, j=2:
[ 5][ 4][ 3][ 2][ 1]   →   i ≮ j → loop exits
            ↑
           i,j

Result: [5, 4, 3, 2, 1]   →   ⌊n/2⌋ swaps, O(n) time, O(1) space.
```

For odd-length arrays, the middle element is its own mirror — no swap needed. For even-length arrays, every element gets swapped exactly once.

---

### 3.3 Fast / slow — remove duplicates from a sorted array

**Problem:** given a sorted array, remove duplicates *in place* and return the new length. The first `k` elements should hold the unique values in order.

**The idea.** Two pointers walk in the same direction:

- `slow` = where the next unique element should land (write head).
- `fast` = where we're currently reading (read head).

Whenever `arr[fast]` differs from `arr[slow]`, advance `slow` and copy `arr[fast]` there. Otherwise just keep scanning.

#### Code (Java)

```java
public static int removeDuplicates(int[] arr) {
    if (arr.length == 0) return 0;
    int slow = 0;
    for (int fast = 1; fast < arr.length; fast++) {
        if (arr[fast] != arr[slow]) {
            slow++;
            arr[slow] = arr[fast];
        }
    }
    return slow + 1;          // count of unique elements
}
```

#### Step-by-step trace

Input: `[1, 1, 2, 2, 2, 3, 4, 4, 5]` (size 9).

```
Frame 1 — slow=0, fast=1:
[ 1][ 1][ 2][ 2][ 2][ 3][ 4][ 4][ 5]
  ↑w   ↑r
  arr[1] = 1 == arr[0] = 1   →   no copy, fast++

Frame 2 — slow=0, fast=2:
[ 1][ 1][ 2][ 2][ 2][ 3][ 4][ 4][ 5]
  ↑w        ↑r
  arr[2] = 2 ≠ arr[0] = 1    →   slow++, arr[1] = 2

Frame 3 — slow=1, fast=3:
[ 1][ 2][ 2][ 2][ 2][ 3][ 4][ 4][ 5]
       ↑w        ↑r
  arr[3] = 2 == arr[1] = 2   →   no copy, fast++

Frame 4 — slow=1, fast=4:
[ 1][ 2][ 2][ 2][ 2][ 3][ 4][ 4][ 5]
       ↑w             ↑r
  arr[4] = 2 == arr[1] = 2   →   no copy, fast++

Frame 5 — slow=1, fast=5:
[ 1][ 2][ 2][ 2][ 2][ 3][ 4][ 4][ 5]
       ↑w                  ↑r
  arr[5] = 3 ≠ arr[1] = 2   →   slow++, arr[2] = 3

Frame 6 — slow=2, fast=6:
[ 1][ 2][ 3][ 2][ 2][ 3][ 4][ 4][ 5]
            ↑w                  ↑r
  arr[6] = 4 ≠ arr[2] = 3   →   slow++, arr[3] = 4

Frame 7 — slow=3, fast=7:
[ 1][ 2][ 3][ 4][ 2][ 3][ 4][ 4][ 5]
                 ↑w                  ↑r
  arr[7] = 4 == arr[3] = 4  →   no copy, fast++

Frame 8 — slow=3, fast=8:
[ 1][ 2][ 3][ 4][ 2][ 3][ 4][ 4][ 5]
                 ↑w                       ↑r
  arr[8] = 5 ≠ arr[3] = 4   →   slow++, arr[4] = 5

End: slow = 4, fast exhausted   →   return slow + 1 = 5

Final array: [1, 2, 3, 4, 5, _, _, _, _]   (the first 5 slots are the answer)
```

Five unique values — the first five slots hold them. Trailing slots still contain old data; the function's contract is "the first k slots are valid."

#### Complexity

- **Time:** O(n) — `fast` advances exactly n − 1 times.
- **Space:** O(1) — two indices, all work in place.
- **Comparisons + copies:** at most n − 1 of each.

The naive solution copies into a hash set then back — O(n) time but O(n) extra space, and you lose the sorted order unless you re-sort. The two-pointer approach is strictly better when the input is sorted.

---

### 3.4 Partition wall — move zeroes to end

**Problem:** given an array, move all zeroes to the end while preserving the relative order of non-zero elements. Do it in place.

> **Heads-up:** this problem gets a fuller, dedicated treatment elsewhere in this unit (Dutch National Flag generalises it to 3-way partitioning). Here it shows up as a clean illustration of the partition-wall pattern.

**The idea.** Same shape as fast/slow with an interpretation tweak:

- `slow` = next slot for a non-zero value (write head).
- `fast` = current scan position (read head).

When `arr[fast]` is non-zero, copy it to `arr[slow]` and advance both. When `arr[fast]` is zero, advance only `fast`. After the scan, fill the tail from `slow..n−1` with zeroes.

#### Code (Java)

```java
public static void moveZeroes(int[] arr) {
    int slow = 0;
    for (int fast = 0; fast < arr.length; fast++) {
        if (arr[fast] != 0) {
            arr[slow] = arr[fast];
            slow++;
        }
    }
    while (slow < arr.length) {
        arr[slow] = 0;
        slow++;
    }
}
```

#### Step-by-step trace (compressed)

Input: `[0, 1, 0, 3, 12]`.

```
Frame 1: slow=0, fast=0  →  arr[0]=0, skip                              [0, 1, 0, 3, 12]
Frame 2: slow=0, fast=1  →  arr[1]=1 ≠ 0   →   arr[0]=1, slow=1         [1, 1, 0, 3, 12]
Frame 3: slow=1, fast=2  →  arr[2]=0, skip                              [1, 1, 0, 3, 12]
Frame 4: slow=1, fast=3  →  arr[3]=3 ≠ 0   →   arr[1]=3, slow=2         [1, 3, 0, 3, 12]
Frame 5: slow=2, fast=4  →  arr[4]=12 ≠ 0  →   arr[2]=12, slow=3        [1, 3, 12, 3, 12]

Tail-fill (slow..n−1):
slow=3  →  arr[3] = 0                                                    [1, 3, 12, 0, 12]
slow=4  →  arr[4] = 0                                                    [1, 3, 12, 0,  0]
```

Final: `[1, 3, 12, 0, 0]`. O(n) time, O(1) space, relative order of non-zeroes preserved.

#### The mental picture

```
After the scan loop ends, the array splits into two regions:

  ┌─── kept (in original order) ───┐ ┌─── stale data ───┐
  [ 1 ][ 3 ][ 12]                    [ 3 ][ 12]
  └────── slow → ─────────┘
        (wall: anything to the LEFT of slow is finalised
         non-zero data; anything to the RIGHT will be
         overwritten with zeroes)
```

That "wall" between finalised and unfinalised regions is what gives this pattern its name.

---

### 3.5 Decision tree — when to reach for two pointers

```
Is the input an array (or string)?
└── Yes → Does the problem hint at "find a pair / triple / window"?
          ├── Yes → Is the array sorted (or sortable)?
          │         ├── Yes → OPPOSITE-ENDS converging is the first thing to try.
          │         └── No  → Sort first if allowed; otherwise look for a
          │                   hash-set alternative (covered later in the
          │                   hashing unit).
          └── No  → Does the problem ask for in-place rearrangement /
                    deduplication / partitioning?
                    ├── Yes → FAST/SLOW or PARTITION-WALL is the first
                    │         thing to try.
                    └── No  → Probably not a two-pointer problem. Consider
                              sliding window, prefix sums, or just iterating.
```

Mental shortcut: **two pointers love sorted arrays and in-place rearrangements.** Whenever you see one of those, ask "can two indices walking the array carry enough state to solve this in one pass?" before reaching for a hash map or a nested loop.

---

### 3.6 A quick-reference table of classic problems

| Problem | Pattern | Time | Space | One-liner |
|---|---|---|---|---|
| Two-sum on sorted array | Opposite ends | O(n) | O(1) | Converge from both ends; move based on `sum vs target`. |
| Reverse array in place | Opposite ends | O(n) | O(1) | Swap `arr[i]` and `arr[j]`, then `i++, j−−`. |
| Valid palindrome | Opposite ends | O(n) | O(1) | Compare `arr[i]` and `arr[j]`; bail on mismatch. |
| Container-with-most-water | Opposite ends | O(n) | O(1) | Track best area; move the pointer with the **smaller** height. |
| Three-sum (sum to target) | Outer loop + opposite ends | O(n²) | O(1) | Fix one element, two-pointer the rest. |
| Remove duplicates (sorted) | Fast / slow | O(n) | O(1) | `slow` writes; advance only when `arr[fast] ≠ arr[slow]`. |
| Move zeroes to end | Partition wall | O(n) | O(1) | `slow` collects non-zeroes; tail-fill with zeroes. |
| Sort 0s/1s/2s (Dutch flag) | 3-way partition | O(n) | O(1) | Three pointers; full episode dedicated to this elsewhere in this unit. |

---

## 4. Real World

- **Database query optimisation.** Joining two sorted relations is a *merge join* — opposite-ends-style walks over the two tables.
- **Merge step of merge sort.** Two pointers walk two sorted halves, picking the smaller front element each time.
- **String / DNA sequence alignment.** Palindrome checks, pair-matching, and many bioinformatics primitives are two-pointer scans.
- **In-place compaction in memory managers and garbage collectors.** Live objects are copied past dead ones using a slow/fast pointer pair — the same pattern as `removeDuplicates`.
- **Network packet reassembly.** Two indices over a circular buffer track read/write head separation.
- **Streaming deduplication.** Ad servers, log pipelines, message queues all use slow/fast variants over windowed data.
- **Image processing.** Convolution and erosion / dilation kernels often boil down to two-pointer scans across rows.

The pattern: **whenever the data has structure (sortedness, ordering, in-place mutability) that lets two indices carry enough state to solve the problem in one pass, two pointers wins.**

---

## 5. Key Takeaways

- **Two pointers** = two indices walking an array, exploiting structure to reach **O(n) time** and **O(1) space**.
- **Three patterns to recognise:**
  - **Opposite ends** — `i = 0`, `j = n−1`, converge. Use for sorted-pair-sum, reverse, palindrome, container-with-most-water.
  - **Fast / slow** — both forward, `fast` every step, `slow` on condition. Use for in-place dedup and compaction.
  - **Partition wall** — `slow` is the write head, `fast` is the read head. Use for move-zeroes, Hoare partition, multi-way partitions.
- **The invariant.** At every step, eliminate possibilities you can prove won't contain the answer, without ever revisiting an index. That's why total work stays O(n).
- **Sortedness is the unlock for opposite ends.** If unsorted but you can sort first, that's O(n log n) — still beats O(n²) past n ≈ 20.
- **Mental shortcut.** When a problem *feels* like nested loops on an array, ask "two indices, one pass?" before writing the inner loop.
- **Common cousins.** Sliding window and prefix sums build on the same single-pass idea, with different flavours of state-tracking; both have their own episodes in this unit.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
