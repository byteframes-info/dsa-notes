# Episode 07 — Difference Arrays

> **ByteFrames · Unit 1 · Episode 7**
> *DSA, frame by frame.*

---

## One-frame summary

The **dual of prefix sums.** A difference array `D` stores *deltas* between consecutive elements. The killer trick: a range update `arr[l..r] += v` becomes **two single-cell touches** — `D[l] += v` and `D[r+1] −= v`. **O(1) per range update**, regardless of range size. After all updates, one O(n) prefix-sum pass over `D` reconstructs the final array. For k range updates that's **O(k + n)** total vs **O(k·n)** brute force.

---

## 1. Why?

In our previous episode on prefix sums we got **O(1) range-sum *queries*** at the cost of an O(n) preprocessing pass. The *dual* problem flips the workload:

- "Add 5 to every element from index 2 to index 7."
- "Decrement values in `arr[100..999]` by 1."
- "On day 12 through day 18, every flight gains 50 bookings."
- "Stamp this 4×4 sub-grid with +3."

The naive way to apply each range update is to walk the range and bump every cell. **O(r − l + 1)** per update — up to O(n) worst case. For k updates that's **O(k·n)**. With k = 1,000,000 updates on an array of n = 1,000,000 you're looking at a trillion operations, just to apply the updates. *Reading* anything from the array hasn't even started yet.

**The trick.** Instead of writing into every affected cell, store the *change at the boundaries*:

- `D[l] += v` says *"starting at index l, the running total goes up by v."*
- `D[r+1] −= v` says *"starting at index r+1, that bump should stop."*

That's two cells touched per range update — **O(1)**. Once all updates are queued, a single prefix-sum pass over `D` "spreads" each delta forward and stops it at the right boundary, reconstructing the final array in **O(n)**. Total: **O(k + n)**.

**Punchline:** when a problem hints at *"many range updates on an array, then I'll read the final state"*, difference arrays are the dual move to prefix sums.

---

## 2. The core idea

### 2.1 Definition (1D)

Two equivalent framings depending on whether you start from an array or from zero:

**Framing A — derive D from arr.** Given `arr[0..n−1]`, define:

```
D[0] = arr[0]
D[i] = arr[i] − arr[i − 1]      for i = 1, 2, ..., n − 1
```

Then `arr[i]` can be recovered from `D` by prefix-summing: `arr[i] = D[0] + D[1] + ... + D[i]`. This is just *the inverse of building a prefix-sum array.* If P was the integral of arr, D is the *derivative*.

**Framing B — D as a delta accumulator.** Start with `D` of length `n` initialised to all zeros (this represents `arr` also being all zeros). Each *range update* on the conceptual `arr` becomes two single-cell writes on `D`:

```
range update arr[l..r] += v:
  D[l]   += v
  if r + 1 < n:  D[r + 1] −= v
```

After all updates, reconstruct `arr` by prefix-summing `D`. The killer use case is Framing B — that's where the O(1) per update payoff lives.

### 2.2 The dual table — prefix sums vs difference arrays

| Operation | **Prefix sum** (immutable, range *reads*) | **Difference array** (lazy range *updates*) |
|---|---|---|
| Build cost | O(n) | O(n) from arr, or O(1) from zeros |
| Point read `arr[i]` | O(1) (it's just arr[i]) | O(n) (or maintain a running prefix) |
| **Range read** `sum(l, r)` | **O(1)** ← killer feature | O(n) |
| Point update `arr[i] += v` | O(n) (rebuild from i) | O(1) (just `D[i] += v; D[i+1] −= v`) |
| **Range update** `arr[l..r] += v` | O(n) (touch every cell) | **O(1)** ← killer feature |
| Use when… | many range *queries*, array immutable | many range *updates*, then read final state |

Same telescoping idea, mirror image. Prefix sums optimise the *read* side; difference arrays optimise the *write* side. They are inverses of each other.

### 2.3 Why this works (the invariant)

A range update `arr[l..r] += v` adds v to every cell from l to r. In delta-space, that's a **+v jump at index l** and a **−v jump at index r+1** (because right after r, the running total returns to where it was). The "running total" is exactly the prefix sum of D — reconstructing the array is just integrating the deltas.

```
Lazy update: arr[2..5] += 3, arr[1..3] += 5

D before:  [ 0][ 0][ 0][ 0][ 0][ 0][ 0][ 0]   (n = 8)
                ↑                ↑
update 1:    D[2] += 3       D[6] −= 3
                ↑       ↑
update 2:    D[1] += 5    D[4] −= 5

D after:   [ 0][ 5][ 3][ 0][−5][ 0][−3][ 0]

Reconstruct (prefix-sum over D):
  arr[0] = 0
  arr[1] = 0 + 5 = 5
  arr[2] = 5 + 3 = 8
  arr[3] = 8 + 0 = 8
  arr[4] = 8 + (−5) = 3
  arr[5] = 3 + 0 = 3
  arr[6] = 3 + (−3) = 0
  arr[7] = 0 + 0 = 0

Final arr = [0, 5, 8, 8, 3, 3, 0, 0]

Verify by hand:
  start:           [0, 0, 0, 0, 0, 0, 0, 0]
  arr[2..5] += 3 → [0, 0, 3, 3, 3, 3, 0, 0]
  arr[1..3] += 5 → [0, 5, 8, 8, 3, 3, 0, 0]   ✓
```

Two updates touched only **four cells of D**. A naive approach would have touched 4 cells (range 2..5) + 3 cells (range 1..3) = 7 cells. The win is small here because the ranges are tiny; for ranges of length 100,000 the win is 2 vs 100,000.

---

## 3. How they work

### 3.1 Build D from arr

#### Code (Java)

```java
public static int[] buildDiff(int[] arr) {
    int n = arr.length;
    int[] D = new int[n];
    D[0] = arr[0];
    for (int i = 1; i < n; i++) {
        D[i] = arr[i] - arr[i - 1];
    }
    return D;
}
```

One pass, **O(n) time, O(n) space**.

#### Step-by-step trace

Build `D` for `arr = [3, 5, 8, 6, 9, 2]`:

```
Index:    0   1   2   3   4   5
arr:    [ 3][ 5][ 8][ 6][ 9][ 2]
```

```
D[0] = arr[0]              = 3
D[1] = arr[1] − arr[0] = 5 − 3 = 2
D[2] = arr[2] − arr[1] = 8 − 5 = 3
D[3] = arr[3] − arr[2] = 6 − 8 = −2
D[4] = arr[4] − arr[3] = 9 − 6 = 3
D[5] = arr[5] − arr[4] = 2 − 9 = −7

D:    [ 3][ 2][ 3][−2][ 3][−7]
```

Reconstructing arr from D is the prefix-sum we already know:

```
arr[0] = D[0]                 = 3
arr[1] = arr[0] + D[1] = 3 + 2 = 5
arr[2] = arr[1] + D[2] = 5 + 3 = 8
arr[3] = arr[2] + D[3] = 8 − 2 = 6
arr[4] = arr[3] + D[4] = 6 + 3 = 9
arr[5] = arr[4] + D[5] = 9 − 7 = 2          ✓
```

This perfectly mirrors what we did building a prefix-sum array — *integral and derivative are inverses.*

---

### 3.2 The killer use case — O(1) range update

Most of the time you don't build D from a known arr. You start with `D` zeroed out (representing arr also being zeros, or any baseline) and apply range updates lazily.

#### Code (Java)

```java
public static void rangeUpdate(int[] D, int l, int r, int v) {
    D[l] += v;
    if (r + 1 < D.length) {
        D[r + 1] -= v;
    }
}
```

Two array writes. Done. **O(1) per update**, regardless of how big the range is.

#### Step-by-step trace

Start: `arr = [0, 0, 0, 0, 0, 0, 0, 0]` (n = 8). Apply three range updates:

1. `arr[2..5] += 3`
2. `arr[1..3] += 5`
3. `arr[0..7] += 1` (full array)

**Frame 0 — D initialised to all zeros:**

```
Index:    0   1   2   3   4   5   6   7
D:      [ 0][ 0][ 0][ 0][ 0][ 0][ 0][ 0]
```

**Frame 1 — apply `arr[2..5] += 3`:**

```
D[2] += 3      →  D[2] = 3
D[6] −= 3      →  D[6] = −3

         ↓+3                    ↓−3
D:      [ 0][ 0][ 3][ 0][ 0][ 0][−3][ 0]

Cost: 2 cell touches, regardless of range length.
```

**Frame 2 — apply `arr[1..3] += 5`:**

```
D[1] += 5      →  D[1] = 5
D[4] −= 5      →  D[4] = −5

      ↓+5            ↓−5
D:      [ 0][ 5][ 3][ 0][−5][ 0][−3][ 0]
```

**Frame 3 — apply `arr[0..7] += 1` (full array):**

```
D[0] += 1      →  D[0] = 1
r + 1 == 8 == n   →  skip the second touch (no D[8] to write to)

 ↓+1
D:      [ 1][ 5][ 3][ 0][−5][ 0][−3][ 0]
```

Three updates total, **5 cells touched** (instead of 4 + 3 + 8 = 15 in brute force). For larger ranges the gap explodes.

---

### 3.3 Reconstruction — prefix-sum over D

Once all updates are queued in `D`, **a single prefix-sum pass** rebuilds the final array.

#### Code (Java)

```java
public static int[] reconstruct(int[] D) {
    int n = D.length;
    int[] arr = new int[n];
    arr[0] = D[0];
    for (int i = 1; i < n; i++) {
        arr[i] = arr[i - 1] + D[i];
    }
    return arr;
}
```

Looks identical to `buildPrefix` from our previous episode on prefix sums — because it *is* the same operation. **O(n) time, O(n) space.**

#### Step-by-step trace

Reconstruct `arr` from the final D state above (`D = [1, 5, 3, 0, −5, 0, −3, 0]`):

```
arr[0] = D[0]                  =  1
arr[1] = arr[0] + D[1] =  1 + 5 =  6
arr[2] = arr[1] + D[2] =  6 + 3 =  9
arr[3] = arr[2] + D[3] =  9 + 0 =  9
arr[4] = arr[3] + D[4] =  9 − 5 =  4
arr[5] = arr[4] + D[5] =  4 + 0 =  4
arr[6] = arr[5] + D[6] =  4 − 3 =  1
arr[7] = arr[6] + D[7] =  1 + 0 =  1

Final arr = [1, 6, 9, 9, 4, 4, 1, 1]
```

Verify by hand-applying the three updates:

```
start:                    [0, 0, 0, 0, 0, 0, 0, 0]
arr[2..5] += 3        →   [0, 0, 3, 3, 3, 3, 0, 0]
arr[1..3] += 5        →   [0, 5, 8, 8, 3, 3, 0, 0]
arr[0..7] += 1        →   [1, 6, 9, 9, 4, 4, 1, 1]   ✓
```

Total work for three range updates and one read of the final state:

```
brute force:    (4) + (3) + (8) = 15 cell writes      →  15 ops
diff array:     2 + 2 + 1 (touches) + 8 (reconstruction) = 13 ops

Tied for tiny ranges. Now imagine ranges of length 1,000,000 each:
brute force:    1,000,000 × 3 = 3,000,000 ops
diff array:     5 (touches) + n (reconstruction) ≈ n ops

That's the difference between O(k·n) and O(k + n).
```

---

### 3.4 The 2D version — rectangle updates

Same telescoping idea, lifted into the plane. Given a grid `arr[R][C]`, define a 2D difference grid `D[R+1][C+1]` (one extra sentinel row + column to dodge the boundary edge case).

A rectangle update `arr[r1..r2][c1..c2] += v` becomes **four single-cell touches** on D:

```
D[r1]   [c1]    += v
D[r1]   [c2 + 1] −= v
D[r2 + 1][c1]    −= v
D[r2 + 1][c2 + 1] += v
```

After all rectangle updates, reconstruct `arr` with a 2D prefix-sum pass over D.

The four touches mirror the four-corner inclusion-exclusion query from our previous episode on prefix sums. Visually:

```
  Goal: bump this rectangle by +v

  ┌───────────────────────┐
  │     A           B     │
  │     ┌───────────┐     │     A = D[r1][c1]      += v   (top-left,  positive)
  │     │           │     │     B = D[r1][c2+1]    −= v   (top-right, negative)
  │     │  rectangle│     │     C = D[r2+1][c1]    −= v   (bot-left,  negative)
  │     │           │     │     D = D[r2+1][c2+1]  += v   (bot-right, positive)
  │     └───────────┘     │
  │     C           D     │
  └───────────────────────┘
```

Four cells touched per rectangle update — **O(1)**, regardless of rectangle size. Reconstruction is then a 2D prefix sum: O(R · C).

#### Worked example

Start: 4×4 grid of zeros. Apply two rectangle updates:

1. `arr[1..2][1..2] += 5`
2. `arr[0..3][0..1] += 2`

**D after update 1** (D has size 5×5):

```
        j=0  j=1  j=2  j=3  j=4
i=0   [  0][  0][  0][  0][  0]
i=1   [  0][  5][  0][ −5][  0]
i=2   [  0][  0][  0][  0][  0]
i=3   [  0][ −5][  0][  5][  0]
i=4   [  0][  0][  0][  0][  0]
```

**D after update 2:**

```
        j=0  j=1  j=2  j=3  j=4
i=0   [  2][  0][ −2][  0][  0]
i=1   [  0][  5][  0][ −5][  0]
i=2   [  0][  0][  0][  0][  0]
i=3   [  0][ −5][  0][  5][  0]
i=4   [ −2][  0][  2][  0][  0]
```

Reconstructing `arr` via 2D prefix sum gives the final grid where every cell reflects the cumulative effect of both rectangle updates. Two updates × 4 touches = **8 cell writes** on D, vs `(2 × 2) + (4 × 2) = 12` cell writes brute force on this tiny grid. For 1000×1000 rectangles the gap explodes.

---

### 3.5 Decision tree — which dual to use

```
What's the dominant workload?
├── many RANGE QUERIES (sum of arr[l..r]), array IMMUTABLE
│       → Prefix sums (covered in our previous episode).
│         O(n) build, O(1) per query, O(n + q) total.
│
├── many RANGE UPDATES (arr[l..r] += v), then ONE read of final state
│       → Difference arrays.
│         O(1) per update, O(n) one-time reconstruction, O(k + n) total.
│
├── BOTH range queries AND range updates, frequently interleaved
│       → Neither pure prefix sum nor pure diff array fits cleanly.
│         Use a SEGMENT TREE WITH LAZY PROPAGATION (deferred to advanced
│         topics). Both directions in O(log n) per op.
│
└── Frequent POINT UPDATES + RANGE QUERIES
        → Fenwick / Binary Indexed Tree (deferred to advanced topics).
          Both ops in O(log n).
```

**Mental shortcut.** When the question is *"many range updates, then read the final array,"* difference arrays are the move. The reconstruction is paid once, no matter how many updates queued up.

---

### 3.6 A quick-reference table of classic problems

| Problem | Key idea | Time | Space |
|---|---|---|---|
| Apply k range increments, then read arr | D += v at boundaries, prefix-sum once | O(k + n) | O(n) |
| Flight bookings / corporate-flight-bookings family | Each booking is `arr[l..r] += seats`; final = prefix-sum | O(k + n) | O(n) |
| Car pooling — does the bus exceed capacity? | Each trip is `passengers[from..to-1] += people`; check max ≤ cap | O(k + n) | O(n) |
| Range increment then point query | Maintain D; point read = prefix-sum up to i (or reconstruct once) | O(k + n) | O(n) |
| Range XOR (toggle a flag region) | Replace `+`/`−` with `⊕`; reconstruction is prefix-XOR | O(k + n) | O(n) |
| 2D rectangle stamps | 4-corner update on D; reconstruct via 2D prefix | O(k + R·C) | O(R·C) |
| Range increment with multiple final point reads | One reconstruction, then O(1) per read | O(k + n + q) | O(n) |
| "Number of times each index gets covered by ranges" | Treat each range as `+1` increment; final D-reconstructed array gives counts | O(k + n) | O(n) |

The flight-bookings family is the most-asked interview shape: every booking adds `seats` to a range of days; you want the final per-day total. Difference array makes each booking O(1).

---

## 4. Real World

- **Flight / hotel / car-share bookings.** Each booking adds capacity to a range of days. Final per-day occupancy comes from a single reconstruction.
- **Range-painting and range-marking systems.** Vector-graphics fills, GIS map overlays, calendar-busy-block computations all benefit when "marks" are stored as boundary deltas.
- **Range-update sensor arrays.** "Increase fan speed for hours 5–10" applied to a 24-hour schedule array is a range update; the final hourly values come from one prefix-sum sweep.
- **Compiler optimisations.** Loop transformations like *strength reduction* convert per-iteration delta updates into one big prefix-sum after the loop.
- **Image processing — stamp operations.** Rectangle-stamp filters (paint, blur masks, ROI brightness adjustments) over an integral image.
- **Bookkeeping / accounting.** Lazy *journal-and-replay* postings: each transaction is an O(1) delta entry; the final ledger state is computed by replaying.
- **Game design.** Buffs and debuffs that affect a range of levels or a region of the map: stored as boundary deltas, applied on read.
- **Computational geometry.** *Sweep-line* algorithms over interval events use the same boundary-delta trick to compute coverage, max-overlap, etc.

The pattern: **whenever updates are range-shaped and the final state matters more than intermediate states, difference arrays turn linear-per-update work into constant-per-update.**

---

## 5. Key Takeaways

- **Difference array** `D[i]` stores the *delta* between consecutive elements: `D[0] = arr[0]`, `D[i] = arr[i] − arr[i − 1]`. It's the inverse of a prefix-sum array.
- **The killer use case:** a range update `arr[l..r] += v` becomes **two single-cell touches** — `D[l] += v` and `D[r + 1] −= v`. **O(1) per update**, regardless of range size.
- **Reconstruction.** After all updates, one O(n) prefix-sum pass over D rebuilds the final arr. That pass is *literally* the same code as `buildPrefix` from our previous episode on prefix sums.
- **Workload payoff.** k range updates cost **O(k + n)** with diff arrays vs **O(k·n)** brute force.
- **The dual table.** Prefix sums optimise *range reads*; difference arrays optimise *range updates*. They are inverses — same telescoping trick, mirror image.
- **2D extension.** Rectangle updates become **four-corner touches** on a `(R + 1) × (C + 1)` diff grid: positive at `(r1, c1)` and `(r2+1, c2+1)`, negative at `(r1, c2+1)` and `(r2+1, c1)`. Reconstruction is a 2D prefix sum.
- **Generalises beyond `+`.** Any operation with an inverse: range XOR (inverse is XOR), modular range product, etc.
- **When neither dual fits.** If reads and writes are *interleaved* and both ranged, neither pure prefix nor pure diff is enough — segment trees with lazy propagation become the right tool (deferred to advanced topics).
- **Mental shortcut.** When you see *"k range updates, then read the final state,"* difference arrays are the dual move to prefix sums.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
