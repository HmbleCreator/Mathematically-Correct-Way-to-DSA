# Module 1c — Complement Search: Set Membership Queries

> **The question this class is always asking:**
> For each element $x$ I'm holding, does its complement $(\text{target} - x)$ already exist in the set of elements I've seen?

---

## The Mathematical Setup

**Student:** I know the Two Sum problem. You use a hash map. But what is it actually doing mathematically?

**Teacher:** You're maintaining a set $S$ of values you've already encountered. At each step, the question is purely about membership:

$$\text{Does } (\text{target} - x) \in S?$$

That's it. The hash map is just the implementation of $S$ that gives you $O(1)$ membership queries. The algorithm itself is a single sweep over the input, asking that membership question at each position.

---

## Two Sum: Building It From Scratch

**Student:** Let's trace it. Array is `[2, 7, 11, 15]`, target is `9`.

**Teacher:** We'll be explicit about the set at each step.

| $i$ | $x$ | Need $(\text{target} - x)$ | $S$ before this step | Found? | Action |
|---|---|---|---|---|---|
| 0 | 2 | 7 | $\emptyset$ | No | Add 2 to $S$ |
| 1 | 7 | 2 | $\{2\}$ | **Yes** | Return $(i_2, i_7)$ |

At $i = 1$, we hold 7. We need $9 - 7 = 2$. We check $S$: is $2 \in S$? Yes. We found our pair.

Notice what we didn't do: we didn't scan forward through the rest of the array. We looked backwards into $S$, which we've been building as we go. One pass, $O(N)$.

**Student:** But we're not just storing values in $S$ — we're storing `value → index`. Why?

**Teacher:** Because the problem asks for indices, not values. The set $S$ becomes a function:

$$f: \text{value} \to \text{index}$$

We extend the set to a map so that when we find a match, we can immediately return both indices. The membership query is the same; the codomain is richer.

---

## The "History Book" Mental Model

Think of the map as a history book. As you walk through the array left to right, you write down every number you've seen and where you saw it. When you pick up a new number, you open the book and ask: "Have I seen the complement of this before?"

If yes — you're done. You have both parties of the pair.  
If no — you write the current number into the book and keep walking.

This works because if the pair $(x, y)$ exists with $x + y = \text{target}$, then at the moment you encounter the later of the two, the earlier one is already in the book.

---

## Generalizing: kSum as Reduction

**Student:** Does this extend to 3Sum and 4Sum?

**Teacher:** Yes, and the extension is a mathematical reduction. For 3Sum:

$$x + y + z = \text{target}$$

Lock in $x$ by iterating over it. Your equation becomes:

$$y + z = \text{target} - x$$

This is exactly Two Sum with a new target. You've reduced 3Sum to Two Sum inside a loop.

$$\underbrace{x}_{\text{outer loop}} + \underbrace{y + z}_{\text{Two Sum engine}} = \text{target}$$

For 4Sum, lock in two variables:

$$\underbrace{w + x}_{\text{two outer loops}} + \underbrace{y + z}_{\text{Two Sum engine}} = \text{target}$$

The pattern is: **kSum = one outer loop + (k−1)Sum**. The recursion bottoms out at Two Sum.

---

## Complexity Analysis

| Problem | Reduction | Time Complexity |
|---|---|---|
| Two Sum | Direct set query | $O(N)$ |
| 3Sum | One loop + Two Sum | $O(N^2)$ |
| 4Sum | Two loops + Two Sum | $O(N^3)$ |
| kSum | $(k-2)$ loops + Two Sum | $O(N^{k-1})$ |

**Student:** Is there a way to bring 3Sum below $O(N^2)$?

**Teacher:** For finding the actual elements, $O(N^2)$ is the known optimal for 3Sum. You can bring kSum down to $O(N^{\lceil k/2 \rceil})$ using a meet-in-the-middle technique — you split the sum into two halves, compute all possible sums for each half, and then do a set intersection — but this trades time for space and is rarely required in standard problem sets.

---

## The Two Pointer Alternative

**Student:** I've seen a different approach for 3Sum that doesn't use a hash map at all.

**Teacher:** That's the Two Pointer technique, and it's worth understanding because it handles duplicates more cleanly. The trade-off is that it requires sorting first.

**The precondition:** Sort the array. Now the problem has a geometric structure — smaller numbers are on the left, larger on the right.

For Two Sum on a sorted array:
- Place a left pointer $\ell$ at the start and a right pointer $r$ at the end.
- If $A[\ell] + A[r] = \text{target}$: found a pair.
- If $A[\ell] + A[r] < \text{target}$: the sum is too small. Move $\ell$ right (increase the smaller number).
- If $A[\ell] + A[r] > \text{target}$: the sum is too large. Move $r$ left (decrease the larger number).

Every move eliminates an entire slice of the search space — not just a single pair. This is the mathematical reason it's $O(N)$: you never revisit a position.

**Handling duplicates in sorted arrays:** After finding a match, skip over consecutive equal values before moving the pointers. Because the array is sorted, duplicates are always adjacent:

```
while ℓ < r and A[ℓ] == A[ℓ + 1]: ℓ++
while ℓ < r and A[r] == A[r - 1]: r--
```

This guarantees every pair in the output is unique without needing a separate deduplication step.

---

## Hash Map vs. Two Pointer: When to Use Which

| | Hash Map | Two Pointer |
|---|---|---|
| Precondition | None | Array must be sorted |
| Duplicates | Requires extra logic | Handled naturally |
| Time | $O(N)$ | $O(N)$ after $O(N \log N)$ sort |
| Space | $O(N)$ for the map | $O(1)$ extra |
| Best for | Unsorted data, index queries | Sorted data, unique combinations |

---

## On Paper First

Before writing code for any complement search problem, write this down:

1. **State the membership question:** "At each step, I'm asking: does $(\text{target} - x) \in S$?"
2. **State what $S$ maps to:** "Values only (set)? Values to indices (map)? Values to counts?"
3. **State the reduction if $k > 2$:** "Lock in $k-2$ variables with outer loops. Inner problem is Two Sum."

---

## Summary

| Property | Value |
|---|---|
| Mapping type | General function (set membership query) |
| Data structure | Hash set or hash map |
| Core operation | $O(1)$ membership: does $(\text{target} - x) \in S$? |
| Generalization | kSum = $(k-2)$ nested loops + Two Sum |

> **The structural insight:** You're not "finding a pair." You're building a set of seen values and asking whether the complement of your current element has already appeared. The hash map is an implementation detail. The mathematical operation is set membership.

---

*← Module 1b | Module 1d →*
