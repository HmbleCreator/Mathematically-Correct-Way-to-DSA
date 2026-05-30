# Module 1e — Frequency Counting: Functions into ℕ

> **The question this class is always asking:**
> What is the distribution of elements across this data? For each element $e$, what is $f(e) = |\{i : A[i] = e\}|$?

---

## The Mathematical Setup

A frequency map is a function:

$$f: \text{elements} \to \mathbb{N}$$

where $f(e)$ is the number of times element $e$ appears in the input. Unlike complement search (which asks a binary membership question) or bijection checking (which enforces structural constraints), frequency counting constructs a full description of the distribution.

The range of $f$ is a subset of $\mathbb{N}$. The codomain is all natural numbers. Querying the range — asking what counts appear, which element has the highest count, whether any element's count exceeds a threshold — is how you solve frequency-based problems.

**Student:** Is a frequency map injective?

**Teacher:** Almost never. Multiple elements can have the same count (e.g., both `A` and `B` appear 3 times). And it's not surjective either — not every natural number need appear as a count. It's a general function: valid, well-defined, but with no special structural properties. That's fine. The power is in what you can query from its range.

---

## Building the Map: One Pass

Reading left to right, update the map at each step:

```
freq = {}
for element in A:
    freq[element] = freq.get(element, 0) + 1
```

This constructs $f$ in $O(N)$ time. Every element is visited once. The map after one full pass is the complete distribution.

---

## Problem Class 1 — Threshold Queries

**Example: Majority Element.** Find the element appearing more than $\lfloor N/2 \rfloor$ times.

After building $f$, scan the range: which key has $f(e) > N/2$?

$$\exists\, e : f(e) > \lfloor N/2 \rfloor$$

Because the majority element (if it exists) must appear more than half the time, at most one such element can exist. The query is $O(|\Sigma|)$ where $\Sigma$ is the alphabet size.

**Student:** Can we do better than building the full map?

**Teacher:** Yes. Boyer-Moore Voting is an $O(N)$ time, $O(1)$ space algorithm that doesn't build a map at all. It maintains a single candidate and a count, treating the array as a sequence of votes. But it only works for the majority element — elements appearing more than $N/2$ times. For general frequency queries, you need the full map.

---

## Problem Class 2 — Equivalence by Frequency Signature

**Example: Group Anagrams.** Group strings that are anagrams of each other.

Two strings are anagrams if and only if they have identical character frequency maps. So the key insight is:

$$\text{strings } s_1 \sim s_2 \iff f_{s_1} = f_{s_2}$$

This defines an equivalence relation. To group by it, we need a canonical form for each equivalence class — something we can use as a hash map key.

Two canonical forms work:

1. **Sorted string:** Sort the characters of each string alphabetically. `"eat"`, `"tea"`, `"ate"` all become `"aet"`. This is $O(K \log K)$ per string where $K$ is the string length.

2. **Frequency tuple:** Represent the frequency map as a fixed-length tuple (one slot per possible character). `"eat"` becomes `(1, 0, 0, 1, 1, 0, ...)`. This is $O(K)$ per string but uses more space.

Both are normalizers: functions $\pi: \text{strings} \to \text{canonical form}$ that map every member of an equivalence class to the same representative.

The algorithm becomes:

$$\pi(s) \to \text{canonical key} \quad \Longrightarrow \quad \text{group}[\pi(s)].append(s)$$

This is a composition of two mappings:
- $\pi$: string → canonical form (normalizer)  
- group: canonical form → list of strings (accumulator)

**Student:** That's the same structure as the equivalence class problems in Module 1d.

**Teacher:** Exactly. DSU groups elements by dynamic connectivity. Frequency-based grouping groups elements by a static property (the frequency signature). The mathematical operation is the same — partition into equivalence classes — but the mechanism is different. DSU uses Union-Find; frequency grouping uses a normalizer.

---

## Problem Class 3 — Sliding Frequency (Window Constraints)

Some problems require a frequency map over a moving window, not the whole array. The map must be updated as the window slides:

- **Element enters the window:** `freq[element]++`
- **Element leaves the window:** `freq[element]--` (remove if count drops to 0)

The constraint on the window is typically expressed in terms of the frequency map's properties:

**Example: Longest Repeating Character Replacement.** Find the longest subarray where you can change at most $K$ characters to make all characters the same.

The key observation:

$$\text{characters to change} = \text{window length} - \max_{e}\, f(e) \leq K$$

$\max f(e)$ is the frequency of the most common character in the current window. If the window length minus this maximum exceeds $K$, the window is invalid — shrink it.

You maintain:
- A frequency map over the current window
- A running maximum of the map's values

This is $O(N)$ because each element enters and leaves the window exactly once.

---

## On Paper First

Before writing code for any frequency counting problem:

1. **State the function:** "I'm constructing $f: \text{elements} \to \mathbb{N}$ where $f(e)$ is the count of $e$ in [scope: full array / current window]."
2. **State the query:** "I'm asking: which element has $f(e) > \text{threshold}$? / which elements share the same $f$-signature? / what is $\max f(e)$ for the current window?"
3. **State the scope:** "Is the frequency map over the full input (static) or a moving window (dynamic)?"

---

## Summary

| Property | Value |
|---|---|
| Mapping type | General function into $\mathbb{N}$ |
| Data structure | Hash map (element → count) |
| Core operation | Build in $O(N)$; query in $O(1)$ per element |
| Problem subtypes | Threshold query / Equivalence grouping / Window frequency |

> **The structural insight:** A frequency map is a complete description of the distribution of your data. Any problem that asks "how many?", "which appears most?", or "are these the same structure?" is asking a question about this distribution. Build the map first, then query it.

---

*← Module 1d | Module 1f →*
