# Module 1g — Vector Spaces: The Sliding Window

> **The question this class is always asking:**
> What is the optimal contiguous sub-vector of this array satisfying a given constraint on its scalar projection?

---

## Arrays as Vectors

An array of $N$ numbers is a vector in $\mathbb{R}^N$:

$$\mathbf{v} = [v_1,\; v_2,\; \ldots,\; v_N]$$

This isn't just notation — it changes how you think about problems. Hash map problems ask: *what relationships exist between elements?* Vector problems ask: *what properties does a contiguous slice of this space have?*

A **contiguous subarray** from index $\ell$ to $r$ is a sub-vector:

$$\mathbf{v}[\ell : r] = [v_\ell,\; v_{\ell+1},\; \ldots,\; v_r]$$

A **scalar projection** of this sub-vector is a single number derived from it — the sum, the maximum, the count of distinct elements, or any other aggregate. The sliding window technique efficiently maintains this projection as the window moves.

---

## Fixed-Size Windows

**Setup:** Given array $\mathbf{v}$ of length $N$ and window size $K$, compute a scalar projection for every contiguous subarray of length $K$.

**Naïve approach:** For each of the $N - K + 1$ windows, recompute the projection from scratch. This is $O(NK)$.

**The observation that changes everything:** When a window of size $K$ slides one step to the right, only two elements change:

$$\text{sum}[\ell+1 : r+1] = \text{sum}[\ell : r] - v_\ell + v_{r+1}$$

The $K - 2$ elements in the interior are unchanged. Instead of recomputing them, we perform one subtraction and one addition:

$$\text{sum}_{\text{new}} = \text{sum}_{\text{old}} - v_{\text{outgoing}} + v_{\text{incoming}}$$

This is $O(1)$ per slide, reducing the total to $O(N)$.

**Trace:** Array `[2, 1, 5, 1, 3, 2]`, $K = 3$, maximise window sum.

| Window | Outgoing | Incoming | Sum | Running Max |
|---|---|---|---|---|
| `[2, 1, 5]` | — | — | 8 | 8 |
| `[1, 5, 1]` | 2 | 1 | $8 - 2 + 1 = 7$ | 8 |
| `[5, 1, 3]` | 1 | 3 | $7 - 1 + 3 = 9$ | **9** |
| `[1, 3, 2]` | 5 | 2 | $9 - 5 + 2 = 6$ | 9 |

Answer: 9. Computed in a single pass with $O(1)$ work per step.

---

## Dynamic-Size Windows

**Student:** What if the problem doesn't give me a fixed window size?

**Teacher:** Then the window size is itself the thing you're solving for. You control two independent pointers: a left pointer $\ell$ and a right pointer $r$. The window is always $\mathbf{v}[\ell : r]$.

Two rules drive the pointers:

- **Expand** (move $r$ right): when the current window doesn't yet satisfy the constraint.
- **Shrink** (move $\ell$ right): when the constraint is satisfied, try to tighten the window and still satisfy it.

The key insight is that $\ell$ and $r$ **never move backwards**. Each pointer advances at most $N$ steps in total. The entire algorithm is $O(N)$ — not $O(N^2)$ as it might initially appear.

**Student:** Why is it $O(N)$ if we have two nested-looking loops?

**Teacher:** Because the two pointers are independent, not nested. In a nested loop, for each position of the outer pointer, the inner pointer might traverse the whole array. Here, $r$ only ever moves right, and $\ell$ only ever moves right. Each element enters the window once (when $r$ passes it) and leaves the window once (when $\ell$ passes it). Total work: $2N$ steps. $O(N)$.

---

## Tracing the Dynamic Window: Minimum Subarray Sum

**Problem:** Find the shortest contiguous subarray with sum $\geq \text{target}$.

Array: `[2, 1, 5, 2, 8]`, target: 7.

We track: current window sum, window length, best length so far.

| Step | Action | Window | Sum | Best length |
|---|---|---|---|---|
| $r = 0$ | Expand: eat 2 | `[2]` | 2 | $\infty$ |
| $r = 1$ | Expand: eat 1 | `[2,1]` | 3 | $\infty$ |
| $r = 2$ | Expand: eat 5 | `[2,1,5]` | 8 ≥ 7 ✓ | 3 |
| $\ell = 1$ | Shrink: drop 2 | `[1,5]` | 6 < 7 | 3 |
| $r = 3$ | Expand: eat 2 | `[1,5,2]` | 8 ≥ 7 ✓ | 3 |
| $\ell = 2$ | Shrink: drop 1 | `[5,2]` | 7 ≥ 7 ✓ | **2** |
| $\ell = 3$ | Shrink: drop 5 | `[2]` | 2 < 7 | 2 |
| $r = 4$ | Expand: eat 8 | `[2,8]` | 10 ≥ 7 ✓ | 2 |
| $\ell = 4$ | Shrink: drop 2 | `[8]` | 8 ≥ 7 ✓ | **1** |
| $\ell = 5$ | Shrink: drop 8 | `[]` | 0 < 7 | 1 |

Answer: 1 (the subarray `[8]`).

---

## The Caterpillar Mental Model

The dynamic window moves like a caterpillar. The right end (head) reaches forward and eats new elements when it's hungry. The left end (tail) releases elements when the caterpillar is full enough to try getting shorter.

The caterpillar never contracts backwards. It only moves forward. This is why it's $O(N)$.

---

## When the Constraint Involves a Frequency Map

Some window problems have constraints not on the sum but on the *distribution* of elements within the window. The window's internal frequency map must be maintained alongside the window itself.

**Example: Longest Repeating Character Replacement**

Constraint: `(window length) − (max frequency in window) ≤ K`

The left side is the number of characters you'd need to change to make the window all one character. If this exceeds $K$, the window is invalid.

```
freq = {}
max_freq = 0
ℓ = 0

for r in range(len(A)):
    freq[A[r]] = freq.get(A[r], 0) + 1
    max_freq = max(max_freq, freq[A[r]])

    if (r - ℓ + 1) - max_freq > K:
        freq[A[ℓ]] -= 1
        ℓ += 1

    best = max(best, r - ℓ + 1)
```

The frequency map is updated as elements enter and leave. The constraint is checked after each expansion.

---

## Choosing Window Type

| Problem asks for | Window type | Pointer rule |
|---|---|---|
| Projection over every window of size $K$ | Fixed | Advance both pointers together |
| Shortest window satisfying a sum constraint | Dynamic | Expand until satisfied; shrink to minimize |
| Longest window satisfying a distribution constraint | Dynamic | Expand greedily; shrink only when violated |

---

## On Paper First

Before writing code for any sliding window problem:

1. **Identify the projection:** "The window tracks [sum / max / frequency map / distinct count]."
2. **State the constraint:** "The window is valid when [projection satisfies condition]."
3. **Choose window type:** "Is the window size fixed or variable?"
4. **Write the update rule:** "When $r$ advances: [do X]. When $\ell$ advances: [undo X]."

The update rule is the key. If you can't write it in one line, reconsider what the projection is.

---

## Summary

| Property | Value |
|---|---|
| Data model | Array as vector in $\mathbb{R}^N$ |
| Subproblem | Scalar projection over sub-vector $\mathbf{v}[\ell : r]$ |
| Fixed window | $O(N)$ via incremental update |
| Dynamic window | $O(N)$ because each element enters and exits exactly once |
| Space | $O(1)$ for sum/max; $O(|\Sigma|)$ for frequency map |

> **The structural insight:** The sliding window works because contiguous sub-vectors share most of their elements. Instead of recomputing a projection from scratch, you maintain it incrementally — adding what entered, removing what left. The efficiency comes from never moving backwards.

---

*← Module 1f | Module 2 →*
