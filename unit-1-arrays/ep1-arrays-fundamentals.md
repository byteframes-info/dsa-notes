# Episode 01 — Arrays

> **ByteFrames · Unit 1 · Episode 1**
> *DSA, frame by frame.*

---

## One-frame summary

An **array** is a fixed-size, contiguous block of memory holding elements of the same type, accessed by integer index in **O(1)** time. It is the foundation of nearly every other data structure.

---

## 1. Why Do We Need Arrays?

### The problem (without arrays)

To store the marks of 1000 students, you would need 1000 named variables:

```java
int marks1 = 87;
int marks2 = 91;
int marks3 = 76;
// ... × 997 more
```

This is unmanageable. You cannot loop over them, cannot pass them around as a group, cannot sort or search them as a unit.

### The fix

```java
int[] marks = new int[1000];   // one name, holds 1000 ints
```

One declaration. One name. Iterable. Sortable. Done.

### What arrays give you

| Capability            | Why it matters                                                   |
|-----------------------|------------------------------------------------------------------|
| **Group data**        | Store any size collection under a single name.                   |
| **Instant access**    | Jump to any index in **O(1)** — no loop, just a formula.         |
| **Easy iteration**    | One `for` loop visits all elements.                              |
| **Foundation of DSA** | Stacks, queues, heaps, hash tables — all built on arrays.        |

---

## 2. What Is an Array?

A contiguous block of memory holding elements of the **same type**, addressed by an integer **index** starting from **0**.

### Visualization

```
Index:    [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
        +----+----+----+----+----+----+----+----+
arr  =  | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |
        +----+----+----+----+----+----+----+----+
Addr:    2000 2004 2008 200C 2010 2014 2018 201C   (hex, 4 bytes/int)
```

### Key properties

- **Fixed size** — declared once, does not grow on its own.
- **Zero-indexed** — first element is at index `0`, last at `n - 1`.
- **Same type** — all elements share the same data type.
- **Contiguous** — stored side by side in memory, no gaps.

### Declaration (Java)

```java
// Static initialization
int[] marks = {87, 91, 76, 88, 95};

// Dynamic allocation
int[] arr = new int[10];        // size 10, default 0
String[] names = new String[5]; // size 5, default null

// Length
int n = marks.length;           // 5
```

---

## 3. 1D Array Operations

### 3.1 Access — `O(1)`

Jump directly to any index using the address formula. The CPU computes the address in **one arithmetic step**:

```
address = base + index × sizeof(element)
```

For `arr[3]` with base = `0x2000`, int size = 4 bytes:

```
address = 0x2000 + 3 × 4 = 0x200C   →  fetch arr[3] = 8
```

```java
int value = arr[3];   // O(1)
```

**No loop. No search. Always constant time.**

---

### 3.2 Insert — `O(n)` worst case

To insert a value at index `i`, we **shift every element from index `i` onward one position to the right** to free up the slot, then place the new value.

We'll insert **`99` at index `2`** in an array of 8 elements that has one extra empty slot at the end (capacity = 9):

#### Initial state

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]  [8]
          +----+----+----+----+----+----+----+----+----+
arr =     | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |  _ |
          +----+----+----+----+----+----+----+----+----+
                       ▲                              empty
                       target index for the new 99
```

#### Step-by-step shift (right-to-left, to avoid overwriting unread data)

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]  [8]
          +----+----+----+----+----+----+----+----+----+
Step 1:   | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 | 26 |   ← copy arr[7] → arr[8]
Step 2:   | 12 |  5 | 31 |  8 | 47 | 19 |  3 |  3 | 26 |   ← copy arr[6] → arr[7]
Step 3:   | 12 |  5 | 31 |  8 | 47 | 19 | 19 |  3 | 26 |   ← copy arr[5] → arr[6]
Step 4:   | 12 |  5 | 31 |  8 | 47 | 47 | 19 |  3 | 26 |   ← copy arr[4] → arr[5]
Step 5:   | 12 |  5 | 31 |  8 |  8 | 47 | 19 |  3 | 26 |   ← copy arr[3] → arr[4]
Step 6:   | 12 |  5 | 31 | 31 |  8 | 47 | 19 |  3 | 26 |   ← copy arr[2] → arr[3]
          +----+----+----+----+----+----+----+----+----+
```

The slot at index `2` is now safely duplicated at index `3`. We can finally overwrite it:

#### Final placement

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]  [8]
          +----+----+----+----+----+----+----+----+----+
Step 7:   | 12 |  5 | 99 | 31 |  8 | 47 | 19 |  3 | 26 |   ← write 99 at arr[2]
          +----+----+----+----+----+----+----+----+----+
                       ▲
                       inserted
```

That's **6 shifts + 1 write = 7 operations** to insert at index `2` in an 8-element array.

> **Why right-to-left?** If we shifted left-to-right, copying `arr[2] → arr[3]` first would *destroy* the original `arr[3]` before we got to move it. Walking from the end toward the target keeps every value safe until it has been copied.

```java
// Insert value v at index i (n = new logical size after insert)
for (int j = n - 1; j > i; j--) {
    arr[j] = arr[j - 1];   // shift right
}
arr[i] = v;
```

#### Why it's O(n)

| Where you insert     | Shifts needed | Complexity |
|----------------------|---------------|------------|
| At the end           | 0             | **O(1)**   |
| In the middle        | ~ n / 2       | **O(n)**   |
| At index 0 (front)   | all n         | **O(n)** worst case |

---

### 3.3 Delete — `O(n)` worst case

To delete the element at index `i`, we **overwrite each cell with the value from its right neighbour**, walking left-to-right, then clear the last slot.

We'll delete **`arr[2]` (value `31`)** from `[12, 5, 31, 8, 47, 19, 3, 26]`:

#### Initial state

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
arr =     | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |
          +----+----+----+----+----+----+----+----+
                       ▲
                       delete target
```

#### Step-by-step shift (left-to-right, each cell overwritten by its right neighbour)

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
Step 1:   | 12 |  5 |  8 |  8 | 47 | 19 |  3 | 26 |   ← copy arr[3] → arr[2]   (31 is overwritten by 8)
Step 2:   | 12 |  5 |  8 | 47 | 47 | 19 |  3 | 26 |   ← copy arr[4] → arr[3]
Step 3:   | 12 |  5 |  8 | 47 | 19 | 19 |  3 | 26 |   ← copy arr[5] → arr[4]
Step 4:   | 12 |  5 |  8 | 47 | 19 |  3 |  3 | 26 |   ← copy arr[6] → arr[5]
Step 5:   | 12 |  5 |  8 | 47 | 19 |  3 | 26 | 26 |   ← copy arr[7] → arr[6]
          +----+----+----+----+----+----+----+----+
```

After step 5, the value `26` is duplicated at indices `6` and `7`. The last slot is now stale data — we clear it:

#### Final cleanup

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
Step 6:   | 12 |  5 |  8 | 47 | 19 |  3 | 26 |  _ |   ← clear arr[7]
          +----+----+----+----+----+----+----+----+
                                                ▲
                                                set to 0 / null
```

The element `31` is gone, everything to its right has slid down by one position, and the logical size shrinks from **8 → 7**.

> **Why left-to-right?** We're moving values *left* (to lower indices). If we walked right-to-left, copying `arr[7] → arr[6]` first would clobber `arr[6]` before we could move *its* value into `arr[5]`. Walking left-to-right preserves every value until it has been read.

```java
// Delete element at index i (n = current logical size)
for (int j = i; j < n - 1; j++) {
    arr[j] = arr[j + 1];   // shift left
}
arr[n - 1] = 0;            // clear last slot
```

#### Why it's O(n)

| Where you delete     | Shifts needed | Complexity |
|----------------------|---------------|------------|
| From the end         | 0             | **O(1)**   |
| From the middle      | ~ n / 2       | **O(n)**   |
| From index 0 (front) | n − 1         | **O(n)** worst case |

---

### 3.4 Search — `O(n)` worst case (linear search)

Walk the array left-to-right, compare each element with the target, and stop on the first match. If you reach the end without a match, the target isn't there.

We'll search for **`target = 47`** in `[12, 5, 31, 8, 47, 19, 3, 26]`:

#### The array

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
arr =     | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |
          +----+----+----+----+----+----+----+----+
```

#### Step-by-step trace (cursor moves through each index)

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
i = 0:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   compare:  12 ≠ 47   keep going
            ▲

i = 1:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   compare:   5 ≠ 47   keep going
                 ▲

i = 2:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   compare:  31 ≠ 47   keep going
                      ▲

i = 3:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   compare:   8 ≠ 47   keep going
                           ▲

i = 4:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   compare:  47 = 47   ✓ FOUND, return 4
                                ▲
          +----+----+----+----+----+----+----+----+
```

That's **5 comparisons** before the match. Indices `5`, `6`, `7` are never visited because we exit early.

```java
public int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;
    }
    return -1;   // not found
}
```

#### Three scenarios

| Scenario        | Where the target is        | Comparisons | Complexity  |
|-----------------|----------------------------|-------------|-------------|
| **Best case**   | At index `0` (very first)  | 1           | **O(1)**    |
| **Average**     | Somewhere in the middle    | ~ n / 2     | **O(n)**    |
| **Worst case**  | At the end, or not present | n           | **O(n)**    |

> **Note:** if the array is **sorted**, you can use **binary search** in `O(log n)` — covered in Unit 2.

---

### 3.5 Traverse — `O(n)`

Visit every element exactly once. This powers printing, sum, average, min/max, filter — anything that needs to look at all the data.

We'll compute **sum** and **max** while traversing `[12, 5, 31, 8, 47, 19, 3, 26]`:

#### The array

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
arr =     | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |
          +----+----+----+----+----+----+----+----+
```

#### Step-by-step trace (running sum and max after each visit)

```
            [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
          +----+----+----+----+----+----+----+----+
i = 0:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum =  12   max =  12
            ▲

i = 1:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum =  17   max =  12
                 ▲

i = 2:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum =  48   max =  31    ← new max
                      ▲

i = 3:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum =  56   max =  31
                           ▲

i = 4:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum = 103   max =  47    ← new max
                                ▲

i = 5:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum = 122   max =  47
                                     ▲

i = 6:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum = 125   max =  47
                                          ▲

i = 7:    | 12 |  5 | 31 |  8 | 47 | 19 |  3 | 26 |   sum = 151   max =  47
                                               ▲
          +----+----+----+----+----+----+----+----+
```

Final answer: **sum = 151**, **max = 47**, after touching all 8 elements.

```java
int sum = 0, max = arr[0];
for (int i = 0; i < arr.length; i++) {
    sum += arr[i];
    if (arr[i] > max) max = arr[i];
}
```

Always **O(n)** — there is no way to compute a result that depends on every element while looking at fewer than `n` of them.

---

### 3.6 1D Operations — Complexity Table

| Operation       | Best  | Worst | Reason                                  |
|-----------------|-------|-------|-----------------------------------------|
| Access by index | O(1)  | O(1)  | Direct address formula                  |
| Insert at end   | O(1)  | O(1)  | No shifting                             |
| Insert at i     | O(1)  | O(n)  | Shift `n - i` elements right            |
| Delete at end   | O(1)  | O(1)  | Just clear the last slot                |
| Delete at i     | O(1)  | O(n)  | Shift `n - i - 1` elements left         |
| Linear search   | O(1)  | O(n)  | Scan until match                        |
| Traverse        | O(n)  | O(n)  | Visit every element                     |

---

## 4. 2D Arrays

A **2D array is an array of arrays** — a grid with **rows and columns**, accessed with two indices: `arr[row][col]`.

### Visualization (3 × 4 grid)

```
              col 0  col 1  col 2  col 3
            +------+------+------+------+
   row 0    |   1  |   2  |   3  |   4  |
            +------+------+------+------+
   row 1    |   5  |   6  |   7  |   8  |
            +------+------+------+------+
   row 2    |   9  |  10  |  11  |  12  |
            +------+------+------+------+

grid[1][2]  =  7
```

### Memory layout — row-major order

A 2D array is **stored flat** in memory: row 0 first, then row 1, then row 2, etc.

```
Memory:  [1][2][3][4]  [5][6][7][8]  [9][10][11][12]
         └─ row 0  ─┘  └─ row 1  ─┘  └─  row 2   ─┘
```

### Address formula

```
address = base + (row × numCols + col) × sizeof(element)
```

For `grid[1][2]` with `numCols = 4`, base = `0x3000`, size = 4:

```
address = 0x3000 + (1 × 4 + 2) × 4
        = 0x3000 + 24
        = 0x3018   →  value = 7
```

### Declaration (Java)

```java
// Literal initialization
int[][] grid = {
    {1, 2, 3, 4},     // row 0
    {5, 6, 7, 8},     // row 1
    {9, 10, 11, 12}   // row 2
};

// Dynamic
int[][] mat = new int[3][4];   // 3 rows, 4 cols, default 0

// Access
int val = grid[1][2];          // 7
grid[2][3] = 99;               // set

// Dimensions
int rows = grid.length;        // 3
int cols = grid[0].length;     // 4
```

### Traversing a 2D array

```java
for (int r = 0; r < grid.length; r++) {
    for (int c = 0; c < grid[r].length; c++) {
        System.out.print(grid[r][c] + " ");
    }
    System.out.println();
}
```

Two nested loops → `O(rows × cols)` = `O(n × m)`.

### Common uses

- **Image pixels** — `pixel[y][x] = color`
- **Game boards** — chess, tic-tac-toe, minesweeper
- **Spreadsheets** — Excel cell B3 is `grid[3][1]`
- **ML matrices** — neural network weights, dot products

---

## 5. 3D Arrays

A **3D array is an array of 2D arrays** — a cube, or a stack of grids (layers). Accessed with three indices: `arr[layer][row][col]`.

### Visualization

```
       Layer 0           Layer 1           Layer 2
       +--+--+--+        +--+--+--+        +--+--+--+
       | 1| 2| 3|        |10|11|12|        |19|20|21|
       +--+--+--+        +--+--+--+        +--+--+--+
       | 4| 5| 6|        |13|14|15|        |22|23|24|
       +--+--+--+        +--+--+--+        +--+--+--+
       | 7| 8| 9|        |16|17|18|        |25|26|27|
       +--+--+--+        +--+--+--+        +--+--+--+

cube[1][2][0] = 16
```

### Address formula

```
address = base + (layer × R × C + row × C + col) × sizeof(element)
```

Where `R` = rows per layer, `C` = cols per row.

### Declaration (Java)

```java
int[][][] cube = new int[3][3][3];   // 3 layers × 3 rows × 3 cols

cube[1][2][0] = 42;                  // set
int v = cube[0][1][2];               // get

// Dimensions
int layers = cube.length;            // 3
int rows   = cube[0].length;         // 3
int cols   = cube[0][0].length;      // 3
```

### Traversing a 3D array

```java
for (int l = 0; l < cube.length; l++) {
    for (int r = 0; r < cube[l].length; r++) {
        for (int c = 0; c < cube[l][r].length; c++) {
            System.out.print(cube[l][r][c] + " ");
        }
    }
}
```

Three nested loops → `O(layers × rows × cols)` = `O(n × m × p)`.

### Common uses

- **3D game worlds** — `voxel[x][y][z]` (Minecraft block grid)
- **Color images** — `image[y][x][channel]` (RGB has 3 channels)
- **Medical imaging** — MRI/CT scans: `scan[slice][row][col]`
- **Video** — `frames[t][y][x]` (time × height × width)

---

## 6. Real-World Examples

| Where                | What is the array?                              |
|----------------------|-------------------------------------------------|
| Netflix watchlist    | Array of show IDs                               |
| Spotify song         | Array of audio samples (44,100 floats / sec)    |
| Instagram feed       | Ordered array of post objects                   |
| 1080p screen         | 2D array of pixels (1920 × 1080)                |
| Chess board          | 2D `int[8][8]` of piece codes                   |
| DNA genome           | Char array of A/T/G/C (≈ 3 billion elements)    |
| Airline seat map     | 2D boolean array `seats[row][col]`              |
| MRI scan             | 3D array of voxels (slices × height × width)    |
| Neural network image | 3D array (height × width × channels)            |

---

## 7. Master Complexity Table

| Operation         | 1D Array  | 2D Array      | 3D Array          |
|-------------------|-----------|---------------|-------------------|
| Access            | O(1)      | O(1)          | O(1)              |
| Insert at end     | O(1)      | N/A*          | N/A*              |
| Insert at middle  | O(n)      | O(n × m)**    | O(n × m × p)**    |
| Delete at middle  | O(n)      | O(n × m)**    | O(n × m × p)**    |
| Linear search     | O(n)      | O(n × m)      | O(n × m × p)      |
| Traverse all      | O(n)      | O(n × m)      | O(n × m × p)      |
| Space             | O(n)      | O(n × m)      | O(n × m × p)      |

*Fixed-shape multi-dimensional arrays do not generally support arbitrary insertion at a position; you usually allocate a new shape.
**If implemented as flat row-major storage with shifting.

---

## 8. Quick Revision Bullets

> Last-minute skim before the exam. Go.

- An **array** = contiguous, fixed-size, same-type, zero-indexed.
- **Access is O(1)** because the address is a formula, not a search.
- **Memory formula:**
  - 1D: `base + i × size`
  - 2D: `base + (r × C + c) × size`
  - 3D: `base + (l × R × C + r × C + c) × size`
- **Row-major** = rows stored back-to-back in memory (Java, C, Python default).
- **Insert at index `i`** → shift `n - i` elements right → **O(n)**.
- **Delete at index `i`** → shift `n - i - 1` elements left, clear last slot → **O(n)**.
- **Insert / delete at the end** is **O(1)**.
- **Linear search** is **O(n)**; binary search is faster but needs a **sorted** array.
- **Traversal** is always **O(n)** — touches every element.
- **2D = grid**, accessed by `arr[row][col]`.
- **3D = cube**, accessed by `arr[layer][row][col]`.
- All multi-dimensional arrays are **stored flat** under the hood.
- Fixed size is the main weakness → solved later by **dynamic arrays / ArrayList**.
- Arrays are everywhere: pixels, audio samples, feeds, DNA, ML tensors.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
