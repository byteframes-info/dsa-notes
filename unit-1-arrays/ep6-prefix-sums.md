# Episode 06 — Prefix Sums

> **ByteFrames · Unit 1 · Episode 6**
> *DSA, frame by frame.*

---

## One-frame summary

Precompute one auxiliary array `P` where `P[i] = arr[0] + arr[1] + ... + arr[i]`. After that **O(n)** preprocessing pass, *any* range sum `arr[l..r]` answers in **O(1)** as `P[r] − P[l−1]`. The trick is to **trade a one-time preprocessing cost for unbounded cheap queries** — turning q range-sum queries from O(q·n) brute force into O(n + q) total. Generalises beautifully to 2D (rectangle sums), prefix XOR, prefix max, and many more.

---

## 1. Why?

In our previous episodes on two pointers and sliding window we saw single-pass tricks that exploit array structure. Sliding window is great when a window is moving forward; what about when the queries are *arbitrary*?

Real questions:

- "What's the sum of `arr[3..7]`?" Then `arr[1..9]`. Then `arr[5..6]`. Then `arr[0..n−1]`. Then…
- "How many users were active on days 50 through 80?" — daily active counts are a 1D array.
- "What's the total revenue inside this 7×7 region of a sales heatmap?"
- "Is index `i` an *equilibrium index* (sum to its left == sum to its right)?"

The naive answer to "sum of `arr[l..r]`" is to walk through `arr[l]`, `arr[l+1]`, …, `arr[r]` and add. That's **O(r − l + 1)** per query — up to **O(n) in the worst case**. For q queries that's **O(q·n)**. With q = 1,000,000 and n = 1,000,000 you're looking at a **trillion operations**.

**The trick.** Precompute *one* auxiliary array `P` once, in **O(n)** time, that stores **cumulative sums**. Every future range-sum query is then a single subtraction — **O(1)**. q queries cost **O(n + q)** total instead of **O(q·n)**. For the numbers above, the gap is 2,000,000 vs a trillion — roughly **half a million times faster.**

**Punchline:** when a problem hints at *"many range-sum queries on an immutable array"*, prefix sums are almost always the move.

---

## 2. The core idea

### 2.1 Definition (1D)

Given `arr[0..n−1]`, define the **prefix-sum array** `P` of length `n` by:

```
P[0] = arr[0]
P[i] = P[i − 1] + arr[i]    for i = 1, 2, ..., n − 1
```

Equivalently: `P[i]` is the sum of the first `i + 1` elements of `arr`.

### 2.2 The query formula

The sum of `arr[l..r]` (inclusive on both ends) is:

```
sum(l, r) = P[r] − P[l − 1]      if l > 0
sum(l, r) = P[r]                 if l == 0
```

That single edge case at `l == 0` is the common beginner pitfall. A clean way to dodge it: define a **virtual** `P[−1] = 0`, then the formula is uniformly `P[r] − P[l − 1]`. Most production code uses a slight variant of the prefix array (length `n + 1` with `P[0] = 0` and `P[i] = P[i − 1] + arr[i − 1]`) precisely to make this edge case disappear. We'll mention that variant in the panels; the inline definition above is friendlier to reason about visually.

### 2.3 Why this works (the invariant)

```
P[r]      = arr[0] + arr[1] + ... + arr[l − 1] + arr[l] + ... + arr[r]
P[l − 1]  = arr[0] + arr[1] + ... + arr[l − 1]
─────────────────────────────────────────────────────────────────────
P[r] − P[l − 1]  =                                arr[l] + ... + arr[r]
```

The two prefixes share the entire `arr[0..l−1]` portion. Subtracting kills it, leaving exactly the range we want. Whatever cumulative function you build (sum, XOR, count, product modulo a prime) the same telescoping works as long as the operation has an **inverse**.

---

## 3. How they work

### 3.1 Build the prefix array

#### Code (Java)

```java
public static int[] buildPrefix(int[] arr) {
    int n = arr.length;
    int[] P = new int[n];
    P[0] = arr[0];
    for (int i = 1; i < n; i++) {
        P[i] = P[i - 1] + arr[i];
    }
    return P;
}
```

One pass, **O(n) time, O(n) space**.

#### Step-by-step trace

Build `P` for `arr = [3, 1, 4, 1, 5, 9, 2, 6]`:

```
Index:    0   1   2   3   4   5   6   7
arr:    [ 3][ 1][ 4][ 1][ 5][ 9][ 2][ 6]
```

**Frame 1 — i = 0 (base case):**

```
P[0] = arr[0] = 3

P:    [ 3][ ?][ ?][ ?][ ?][ ?][ ?][ ?]
```

**Frame 2 — i = 1:**

```
P[1] = P[0] + arr[1] = 3 + 1 = 4

P:    [ 3][ 4][ ?][ ?][ ?][ ?][ ?][ ?]
```

**Frame 3 — i = 2:**

```
P[2] = P[1] + arr[2] = 4 + 4 = 8

P:    [ 3][ 4][ 8][ ?][ ?][ ?][ ?][ ?]
```

**Frame 4 — i = 3:**

```
P[3] = P[2] + arr[3] = 8 + 1 = 9

P:    [ 3][ 4][ 8][ 9][ ?][ ?][ ?][ ?]
```

**Frame 5 — i = 4:**

```
P[4] = P[3] + arr[4] = 9 + 5 = 14

P:    [ 3][ 4][ 8][ 9][14][ ?][ ?][ ?]
```

**Frame 6 — i = 5:**

```
P[5] = P[4] + arr[5] = 14 + 9 = 23

P:    [ 3][ 4][ 8][ 9][14][23][ ?][ ?]
```

**Frame 7 — i = 6:**

```
P[6] = P[5] + arr[6] = 23 + 2 = 25

P:    [ 3][ 4][ 8][ 9][14][23][25][ ?]
```

**Frame 8 — i = 7:**

```
P[7] = P[6] + arr[7] = 25 + 6 = 31

P:    [ 3][ 4][ 8][ 9][14][23][25][31]
```

Total work: 1 assignment + 7 additions = 8 operations. **O(n).**

---

### 3.2 O(1) range-sum query

Once `P` is built, `sum(l, r) = P[r] − P[l − 1]` (with `l > 0`).

#### Code (Java)

```java
public static int rangeSum(int[] P, int l, int r) {
    if (l == 0) return P[r];
    return P[r] - P[l - 1];
}
```

#### Step-by-step trace

Using the `P` we just built, find `sum(arr[2..5])` = `arr[2] + arr[3] + arr[4] + arr[5]` = `4 + 1 + 5 + 9` = `19`.

```
Index:   0   1   2   3   4   5   6   7
arr:   [ 3][ 1][ 4][ 1][ 5][ 9][ 2][ 6]
P:     [ 3][ 4][ 8][ 9][14][23][25][31]
                 ↑              ↑
                 P[1]=4         P[5]=23
                 (sum of 0..1)  (sum of 0..5)

sum(2..5) = P[5] − P[2 − 1]
          = P[5] − P[1]
          = 23 − 4
          = 19 ✓
```

The two prefix entries share the `arr[0..1]` portion. Subtracting them kills the shared part, leaving exactly the queried range.

#### Visual intuition

```
P[5] covers:    [ 3][ 1][ 4][ 1][ 5][ 9]                       (sum = 23)
P[1] covers:    [ 3][ 1]                                       (sum =  4)
                  XX  XX  ←── these cancel ──→
                          [ 4][ 1][ 5][ 9]                     (sum = 19)
                           ─────── target ──────
```

Two array reads, one subtraction. **O(1)** per query, regardless of range length.

---

### 3.3 Build once, query forever — the immutable-array trick

Prefix sums shine when you have a **read-heavy** workload: build `P` once in O(n), then service unlimited queries in O(1) each.

| Workload | Naive (no prefix) | With prefix sums | Best when |
|---|---|---|---|
| 1 query | O(n) | O(n) build + O(1) query = O(n) | tied |
| 10 queries | O(10n) | O(n) + O(10) ≈ O(n) | prefix wins |
| n queries | O(n²) | O(n) + O(n) = O(n) | prefix wins by a factor of n |
| q queries | **O(q·n)** | **O(n + q)** | prefix always wins past q ≥ 1 |

Caveat: if the array is **mutable**, every update invalidates `P` and rebuilding is O(n). For a mutable array with frequent updates *and* range queries, you'll want a different structure — segment trees or Fenwick trees, both deferred to the advanced topics.

---

### 3.4 The 2D version — rectangle sums

Same idea, two dimensions. Given a grid `arr[0..R−1][0..C−1]`, define a **2D prefix-sum grid** `P` of size `(R + 1) × (C + 1)` with:

```
P[0][j] = 0  for all j
P[i][0] = 0  for all i
P[i][j] = P[i − 1][j] + P[i][j − 1] − P[i − 1][j − 1] + arr[i − 1][j − 1]
```

Each `P[i][j]` is the sum of the submatrix `arr[0..i−1][0..j−1]`.

**Why the `+ P[i−1][j−1]`?** Because `P[i−1][j]` and `P[i][j−1]` both include `P[i−1][j−1]` — so adding them double-counts that overlap. Subtracting `P[i−1][j−1]` corrects it. This is **inclusion-exclusion** in one dimension; the rectangle query below is the same trick in two.

#### Rectangle range query

The sum of the rectangle from `(r1, c1)` to `(r2, c2)` (inclusive) is:

```
sum = P[r2+1][c2+1]
    − P[r1]   [c2+1]
    − P[r2+1][c1]
    + P[r1]   [c1]
```

That's **four reads, three arithmetic ops** — **O(1)** regardless of rectangle size.

#### Worked example

Grid:

```
        c=0  c=1  c=2  c=3
r=0   [  3][  1][  4][  1]
r=1   [  5][  9][  2][  6]
r=2   [  5][  3][  5][  8]
r=3   [  9][  7][  9][  3]
```

Build the prefix grid `P` (size 5 × 5, with the extra zero-row and zero-column on top and left):

```
        j=0  j=1  j=2  j=3  j=4
i=0   [  0][  0][  0][  0][  0]
i=1   [  0][  3][  4][  8][  9]
i=2   [  0][  8][ 18][ 24][ 31]
i=3   [  0][ 13][ 26][ 37][ 52]
i=4   [  0][ 22][ 42][ 62][ 80]
```

Query: sum of the rectangle from `(r1=1, c1=1)` to `(r2=2, c2=2)` — the four cells `9, 2, 3, 5`:

```
sum = P[3][3] − P[1][3] − P[3][1] + P[1][1]
    = 37     − 8       − 13      + 3
    = 19
```

Check: `9 + 2 + 3 + 5 = 19` ✓.

#### Why inclusion-exclusion (the four-corners picture)

```
   want this rectangle:     A − B − C + D    (Venn-style)

   ┌─────────────────┐      A = full prefix to (r2, c2)
   │A      ┌──────┐  │      B = strip ABOVE the target
   │       │      │  │      C = strip LEFT  of the target
   │       │      │  │      D = the corner counted twice in B + C
   │       └──────┘  │
   │                 │      Subtracting B and C removes both strips,
   │     target sits │      but D was inside both strips — we subtracted
   │     in there    │      it twice, so we add it back once.
   └─────────────────┘
```

Same telescoping as 1D, lifted into the plane. Three rectangles' worth of cancellation, one corner correction. Constant time, beautiful.

---

### 3.5 Decision tree — when to reach for prefix sums

```
Is the input an array (or 2D grid) and largely IMMUTABLE?
└── Yes → Will the workload run MANY range-sum (or range-XOR /
          range-product / range-count) queries on it?
          ├── Yes → Prefix sums.
          │         1D: O(n) build, O(1) query, total O(n + q).
          │         2D: O(R·C) build, O(1) query, total O(R·C + q).
          └── No  → If just one or two queries, naive iteration is fine.
                    No setup cost.

Is the array MUTABLE (frequent point updates)?
└── Yes → Prefix sums are the wrong tool: every update is O(n) to
          rebuild. Reach for a segment tree or Fenwick tree (deferred
          to the advanced-topics section).
```

**Mental shortcut.** When the question is *"give me the sum (or count, or XOR) of arr[l..r] — repeatedly"* and the array doesn't change, prefix sums are the move. The build cost is paid once; queries are free.

---

### 3.6 A quick-reference table of classic problems

| Problem | Key idea | Time | Space |
|---|---|---|---|
| Range sum of `arr[l..r]` (immutable) | Build P, query `P[r] − P[l−1]` | O(n) build, O(1) query | O(n) |
| Equilibrium index | For each i, check `prefix(i−1) == total − prefix(i)` | O(n) | O(n) or O(1) |
| Rectangle sum on a grid | 2D prefix + inclusion-exclusion | O(R·C) build, O(1) query | O(R·C) |
| Range XOR | Same skeleton, replace `+` with `⊕`, `−` with `⊕` (XOR is its own inverse) | O(n) build, O(1) query | O(n) |
| Count subarrays with sum K | Prefix sum + hash map of seen prefix counts | O(n) | O(n) |
| Maximum subarray sum (Kadane variant via prefix) | Track running min prefix; answer = max(P[i] − min P seen so far) | O(n) | O(1) |
| Number of "good" subarrays (sum divisible by K) | Group prefixes by `P[i] mod K`; count pairs per bucket | O(n) | O(K) |
| Subarray with given sum (positive ints only) | Two-pointer on prefix is equivalent to sliding window | O(n) | O(1) |

The hash-map combo (the "count subarrays with sum K" pattern) is one of the most-asked interview problems. Worth recognising on sight: *prefix sums + a map of "how many prefixes had this value before me"*.

---

## 4. Real World

- **OLAP / data-warehouse aggregations.** Pre-aggregated cube cells are 1D / 2D / N-D prefix sums under different names. Slicing a region of a cube is a constant-time inclusion-exclusion query.
- **Image processing — integral images.** The 2D prefix sum is called an *integral image* in computer vision. The Viola-Jones face detector's feature evaluation runs in constant time per Haar feature, *because* of integral images.
- **GPU and rendering.** Summed-area tables (SAT) for box blurs, depth-of-field, ambient occlusion — same 2D prefix-sum trick, computed on the GPU.
- **Time-series analytics.** Pre-computing cumulative event counts lets you answer "how many events happened between t1 and t2" in O(1).
- **Game design.** Loot tables and weighted random sampling: build a prefix-sum array of weights, binary-search for a random target → O(log n) sampling.
- **Compilers / linkers.** Section-offset tables in object files are essentially prefix sums of section sizes.
- **Bioinformatics.** GC-content over a sliding genomic window is a prefix-sum lookup.
- **Networking.** Cumulative byte counters (TCP sequence numbers, byte-range HTTP requests) leverage the same telescoping.

The pattern: **whenever you have read-heavy aggregate queries on essentially-static data, prefix sums turn linear queries into constant-time lookups.**

---

## 5. Key Takeaways

- **Prefix-sum array** `P[i] = arr[0] + arr[1] + ... + arr[i]` — built in **O(n)** with a single pass.
- **Range-sum query** `sum(l, r) = P[r] − P[l − 1]` (use `P[r]` directly when `l == 0`). **O(1)** per query.
- **Workload payoff.** q range-sum queries cost **O(n + q)** with prefix sums vs **O(q·n)** brute force. The break-even is **q ≥ 1** — prefix sums almost always win.
- **The invariant.** Two prefix entries share their first `l` elements. Subtracting them telescopes the shared portion away, leaving exactly the queried range.
- **2D extension.** A `(R + 1) × (C + 1)` prefix grid lets you answer rectangle sums in **O(1)** via inclusion-exclusion: `P[r2+1][c2+1] − P[r1][c2+1] − P[r2+1][c1] + P[r1][c1]`.
- **Off-by-one alert.** The `l == 0` edge case in 1D queries trips up most beginners. Either special-case it, or use the *(n + 1)*-length variant with `P[0] = 0` to dodge it entirely.
- **Generalises beyond `+`.** Any operation with an inverse works: prefix XOR (inverse is XOR itself), prefix product modulo a prime (inverse is modular inverse), prefix count of a flag.
- **Mutability matters.** Prefix sums shine on immutable arrays. If point updates happen often, a different structure (segment tree, Fenwick tree) is the right call.
- **Cousin technique.** A *difference array* is the dual: O(1) range *updates* + O(n) reconstruction. Same telescoping idea, mirror image of what we just covered.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
