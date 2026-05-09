# Episode 05 — Sliding Window

> **ByteFrames · Unit 1 · Episode 5**
> *DSA, frame by frame.*

---

## One-frame summary

A *window* is a sub-range of an array or string between two indices, **left** and **right**. The window **slides** forward — sometimes a fixed size (move both edges in lockstep), sometimes variable (right expands, left shrinks based on a condition). The trick: instead of recomputing window stats from scratch, **update them incrementally** as the window slides — turning a lot of O(n·k) or O(n²) problems into a clean **O(n)** with O(1) extra space.

---

## 1. Why?

In our previous episode on two pointers we saw that two indices can carry enough state to solve pair-sum problems in O(n). Sliding window is a close cousin: same two-index idea, but now the indices define a *range* whose contents we track over time.

Real questions:

- "What's the maximum sum of any 5 consecutive elements?"
- "What's the longest substring with no repeating characters?"
- "What's the smallest subarray whose sum ≥ a given target?"
- "How many distinct values appear in every window of size k as I slide across the array?"

The naive way is to enumerate every window and recompute its stats from scratch: **O(n·k)** for fixed size, **O(n²)** for variable size. Both feel slow.

**The trick.** When the window moves by 1, *most* of its contents stay the same — one element joins on the right, and (for fixed-size) one leaves on the left. Update the window's summary *incrementally* using just those two changes. That's the **O(1) update** that turns the whole algorithm into O(n).

**Punchline:** when a problem says *"consider every window"* or *"consider every contiguous subarray with property X"*, ask "can I maintain my answer as the window slides, instead of recomputing it from zero each time?" If yes, sliding window beats brute force.

---

## 2. The two flavors

| Flavor | Window size | When right moves | When left moves | Classic problem |
|---|---|---|---|---|
| **Fixed-size** | exactly `k` | every step | every step (lockstep) | max sum of k consecutive, average of k, max in every window |
| **Variable-size** | grows / shrinks | when invariant still holds | when invariant breaks (until restored) | longest without repeats, smallest subarray sum ≥ target, longest with ≤ k distinct |

Both share the DNA: **two indices defining a range, incremental summary updates, one pass, O(n) work, O(1) or O(k) space.**

---

## 3. How they work

### 3.1 Fixed-size window — max sum of k consecutive elements

**Problem:** given an array and a positive integer `k`, find the maximum sum of any `k` consecutive elements.

**The naive way.** For each starting index `i`, sum `arr[i..i+k-1]`. That's `n − k + 1` windows times `k` additions each — **O(n·k)**.

**The sliding-window way.** Compute the first window's sum once. Then slide: as the right edge moves to include `arr[right]`, the left edge moves to exclude `arr[left]`. Update with two arithmetic operations: `sum += arr[right] − arr[left]; left++`. That's **O(1) per step → O(n) total.**

#### Code (Java)

```java
public static int maxSumOfK(int[] arr, int k) {
    if (arr.length < k) return -1;
    int sum = 0;
    for (int i = 0; i < k; i++) sum += arr[i];          // first window
    int max = sum;
    for (int right = k; right < arr.length; right++) {
        sum += arr[right] - arr[right - k];             // slide by 1
        if (sum > max) max = sum;
    }
    return max;
}
```

#### Step-by-step trace

Find max sum of **k = 3** in `[2, 1, 5, 1, 3, 2]`:

```
Index:    0   1   2   3   4   5
Value:  [ 2][ 1][ 5][ 1][ 3][ 2]
```

**Frame 1 — initial window [0..2]:**

```
[ 2][ 1][ 5][ 1][ 3][ 2]
 └────window────┘
 sum = 2 + 1 + 5 = 8        max = 8
```

**Frame 2 — slide right: drop arr[0]=2, add arr[3]=1:**

```
[ 2][ 1][ 5][ 1][ 3][ 2]
      └────window────┘
 sum += arr[3] − arr[0] = 1 − 2 = −1   →   sum = 8 − 1 = 7
 max = max(8, 7) = 8
```

**Frame 3 — slide right: drop arr[1]=1, add arr[4]=3:**

```
[ 2][ 1][ 5][ 1][ 3][ 2]
           └────window────┘
 sum += arr[4] − arr[1] = 3 − 1 = 2    →   sum = 7 + 2 = 9
 max = max(8, 9) = 9
```

**Frame 4 — slide right: drop arr[2]=5, add arr[5]=2:**

```
[ 2][ 1][ 5][ 1][ 3][ 2]
                └────window────┘
 sum += arr[5] − arr[2] = 2 − 5 = −3   →   sum = 9 − 3 = 6
 max = max(9, 6) = 9
```

**Result:** max sum = **9**, found at window `[2..4]`. Total work: 3 (initial sum) + 3 (slides × 2 ops each) = 9 arithmetic operations. Brute force on this input would have done 4 windows × 3 elements = 12 operations. The gap blows up as n and k grow — for n = 1,000,000 and k = 100, brute force does 100M ops; sliding window does 1M.

#### Why this is O(n)

Each element enters the window exactly once (when `right` passes over it) and leaves exactly once (when `left` passes over it). At most 2n operations total → **O(n) time, O(1) space.**

---

### 3.2 The prefix → window transition (the trick that runs through all of this)

A useful mental model: a sliding window is what you get when you take a prefix-sum approach and **reuse work between adjacent windows** instead of recomputing from scratch.

```
Brute force:                      Sliding window:
  for each window starting at i:   compute sum of window [0..k-1]
    sum = 0                        for right = k..n-1:
    for j = i..i+k-1:                sum += arr[right] - arr[right-k]
      sum += arr[j]                  update best

  Inner loop runs k times.        Inner work is O(1) per step.
  Total: O(n · k).                Total: O(n).
```

The two only differ in *what they remember between iterations*. Brute force forgets everything and rebuilds. Sliding window keeps the running sum and updates it with a single `add-new − drop-old` operation.

The same reuse idea generalises to other window stats: a count of distinct elements (use a frequency map), a max (use a monotonic deque), a sum-of-squares, etc. **As long as you can update the stat incrementally on +1 element / −1 element, sliding window applies.**

---

### 3.3 Variable-size window — expand right, shrink left

When the window size depends on a *condition* rather than being fixed, the pattern shifts:

1. **Expand right** every step — always advance `right`.
2. **Shrink left** until the window's invariant is restored.
3. After each step, take the answer (current size, current sum, etc.) and update the running best.

Two main shapes:

- **Find the longest window with property X.** Expand right; shrink left only if the window violates X. Track max window size seen.
- **Find the shortest window with property X.** Expand right; once X is satisfied, shrink left as far as you can while keeping X. Track min window size seen.

The shrink loop runs at most n times **across the whole algorithm** — each element can leave the window only once. So the total work is bounded by 2n → **O(n) time**, even though there's a nested loop in the code.

---

### 3.4 Variable-size — longest substring without repeating characters

**Problem:** given a string, find the length of the longest substring with no repeated character.

**The idea.** Maintain a window with a `seen` set. Expand right by adding the new char. If the new char is already in `seen`, shrink left (advance `left` and remove chars from `seen`) until the duplicate is gone. Track the max window size at every step.

#### Code (Java)

```java
public static int longestWithoutRepeats(String s) {
    Set<Character> seen = new HashSet<>();
    int left = 0, max = 0;
    for (int right = 0; right < s.length(); right++) {
        while (seen.contains(s.charAt(right))) {
            seen.remove(s.charAt(left));
            left++;
        }
        seen.add(s.charAt(right));
        max = Math.max(max, right - left + 1);
    }
    return max;
}
```

#### Step-by-step trace

Input: `"abcabcbb"`.

```
right=0, ch='a':  not in seen. add 'a'.        window [0..0] = "a"      max=1
right=1, ch='b':  not in seen. add 'b'.        window [0..1] = "ab"     max=2
right=2, ch='c':  not in seen. add 'c'.        window [0..2] = "abc"    max=3
right=3, ch='a':  IN SEEN. shrink:
                    left=0: remove 'a', left=1. seen={'b','c'}.
                  add 'a'.                     window [1..3] = "bca"    max=3
right=4, ch='b':  IN SEEN. shrink:
                    left=1: remove 'b', left=2. seen={'c','a'}.
                  add 'b'.                     window [2..4] = "cab"    max=3
right=5, ch='c':  IN SEEN. shrink:
                    left=2: remove 'c', left=3. seen={'a','b'}.
                  add 'c'.                     window [3..5] = "abc"    max=3
right=6, ch='b':  IN SEEN. shrink:
                    left=3: remove 'a', left=4. seen={'b','c'}.
                    left=4: remove 'b', left=5. seen={'c'}.
                  add 'b'.                     window [5..6] = "cb"     max=3
right=7, ch='b':  IN SEEN. shrink:
                    left=5: remove 'c', left=6. seen={'b'}.
                    left=6: remove 'b', left=7. seen={}.
                  add 'b'.                     window [7..7] = "b"      max=3

return max = 3.
```

**Result:** longest substring without repeats = **3** ("abc", "bca", "cab" — multiple windows tie).

#### Why this is O(n)

`right` advances exactly n times. `left` advances at most n times *total* across all the shrink loops combined. So the total work is bounded by 2n → **O(n) time**, **O(min(n, alphabet))** space for the set.

---

### 3.5 Variable-size — smallest subarray with sum ≥ target

**Problem:** given a positive-integer array and a target, find the *smallest* contiguous subarray whose sum is ≥ target. Return its length, or 0 if no such subarray exists.

**The idea.** Expand right while `sum < target`. Once `sum ≥ target`, shrink left as far as possible while the invariant holds, recording the window size at each shrink step. Track the min window size seen.

#### Code (Java)

```java
public static int minSubarraySum(int[] arr, int target) {
    int left = 0, sum = 0, min = Integer.MAX_VALUE;
    for (int right = 0; right < arr.length; right++) {
        sum += arr[right];
        while (sum >= target) {
            min = Math.min(min, right - left + 1);
            sum -= arr[left];
            left++;
        }
    }
    return min == Integer.MAX_VALUE ? 0 : min;
}
```

#### Step-by-step trace

Input: `arr = [2, 3, 1, 2, 4, 3]`, `target = 7`.

```
right=0, sum=2.   sum < 7.   window [0..0].
right=1, sum=5.   sum < 7.   window [0..1].
right=2, sum=6.   sum < 7.   window [0..2].
right=3, sum=8.   sum ≥ 7.   min = 4 (window [0..3]).
                  shrink: sum −= arr[0]=2 → sum=6, left=1. sum < 7.
right=4, sum=10.  sum ≥ 7.   min = min(4, 4) = 4 (window [1..4]).
                  shrink: sum −= arr[1]=3 → sum=7, left=2. min = 3 (window [2..4]).
                  shrink: sum −= arr[2]=1 → sum=6, left=3. sum < 7.
right=5, sum=9.   sum ≥ 7.   min = min(3, 3) = 3 (window [3..5]).
                  shrink: sum −= arr[3]=2 → sum=7, left=4. min = 2 (window [4..5]).
                  shrink: sum −= arr[4]=4 → sum=3, left=5. sum < 7.

return min = 2.
```

**Result:** smallest subarray sum ≥ 7 has length **2** (the subarray `[4, 3]` at indices 4..5).

#### Why "shortest" works the same as "longest"

Same skeleton (expand right, shrink left), different condition for shrinking. **Longest** shrinks only when the invariant breaks. **Shortest** shrinks while the invariant *holds*, recording the window size each time. Either way: every element enters once, every element leaves at most once → O(n).

---

### 3.6 Decision tree — when to reach for sliding window

```
Is the input an array (or string)?
└── Yes → Does the problem ask for a property over a contiguous subarray
          (sum, max, count, distinct count, longest, shortest, average)?
          ├── Yes → Is the subarray size FIXED (= k)?
          │         ├── Yes → Fixed-size sliding window. Initial sum,
          │         │         then add-new-drop-old per step. O(n).
          │         └── No  → Is the property monotone in window size
          │                   (e.g., adding to the window can only INCREASE
          │                   the count of distinct chars)?
          │                   ├── Yes → Variable-size sliding window.
          │                   │         Expand right; shrink left until
          │                   │         invariant restored. O(n).
          │                   └── No  → Probably need prefix sums + hashing
          │                             (covered later in this unit), or a
          │                             different technique entirely.
          └── No  → Probably not a sliding-window problem. Consider two
                    pointers, prefix sums, or just iterating.
```

**Mental shortcut.** When you see *"consider every window"* or *"every contiguous subarray with property X"* → think sliding window first. The prefix→window transition (turning recompute-each-window into incremental update) is the move.

---

### 3.7 A quick-reference table of classic problems

| Problem | Flavor | Window invariant | Time | Space |
|---|---|---|---|---|
| Max sum of k consecutive | Fixed | size = k | O(n) | O(1) |
| Average of k consecutive | Fixed | size = k | O(n) | O(1) |
| Max in every window of size k | Fixed | size = k | O(n) | O(k) (monotonic deque) |
| Longest substring without repeats | Variable | no duplicate char | O(n) | O(min(n, alphabet)) |
| Longest with at most k distinct chars | Variable | distinct count ≤ k | O(n) | O(k) |
| Smallest subarray sum ≥ target | Variable | sum ≥ target | O(n) | O(1) |
| Substring containing all chars of T | Variable | every count in T satisfied | O(n + m) | O(m) |
| Permutation of T inside S | Variable | size = len(T), counts match | O(n + m) | O(m) |

---

## 4. Real World

- **Network protocols (TCP).** The receive window is *literally* a sliding window — tracks bytes received but not yet acknowledged.
- **Rate limiting.** Token-bucket and sliding-window rate limiters track requests inside a moving time window: "no more than 100 requests in any 60-second window."
- **Streaming analytics.** "Average price over the last 5 minutes" — every metric pipeline uses sliding-window aggregations.
- **Audio / video signal processing.** Moving averages, low-pass filters, and FFT windowing all slide a fixed-size window over a sample stream.
- **Sensor data smoothing.** IoT readings, fitness trackers, weather stations — all run sliding-window smoothers to denoise their inputs.
- **Typeahead / search-as-you-type.** "Did the query change in the last 200 ms?" is a sliding-window check on user input.
- **Anomaly detection.** "Did the error rate in the last 60 seconds exceed 3× the baseline?" — sliding window over event timestamps.

The pattern: **whenever the question is about a contiguous range of recent data and the answer can be maintained incrementally as the range moves, sliding window wins.**

---

## 5. Key Takeaways

- A **sliding window** is a sub-range `[left..right]` of an array or string, with summary state maintained incrementally as the window slides.
- **Fixed-size** (size = k): initial sum, then `sum += arr[right] − arr[left]; left++` per step. O(n).
- **Variable-size**: expand `right` every step, shrink `left` when an invariant breaks (or while it holds, for the "shortest" variant). O(n) because each element enters and leaves the window at most once.
- **The prefix → window transition** is the trick: recomputing each window from scratch is O(n·k); incremental update is O(1) per step → O(n) total.
- **Two-flavor signal.** Problem says fixed `k` → fixed-size window. Problem says *"longest"* / *"shortest"* with a condition → variable-size window.
- **Generalises beyond sums.** Any stat that updates in O(1) on +1 element / −1 element works: count of distinct, max (with a monotonic deque), sum-of-squares, etc.
- **Mental shortcut.** When you see *"every contiguous subarray with property X"*, think sliding window before brute force.
- **Cousin techniques.** Two pointers (covered in our previous episode) and prefix sums (covered later in this unit) share the same single-pass DNA — different shapes of state.

---

*ByteFrames · DSA, frame by frame.*

**Connect with us:**
- ▶️ — [`@Byte.Frames`](https://www.youtube.com/channel/UCxt5WocT6YYbQZ6qunkHxqg)
- 📷 — [`@byte.framess`](https://www.instagram.com/byte.framess)
- 📧 — [`byteframes.info@gmail.com`](mailto:byteframes.info@gmail.com)
