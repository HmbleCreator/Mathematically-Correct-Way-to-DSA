# Module 1b — Structural Isomorphism: Proving Bijections

> **The question this class is always asking:**
> Does a bijection $f: A \to B$ consistent with the sequence structure exist?

---

## The Mathematical Setup

**Student:** What does "consistent with the sequence" actually mean?

**Teacher:** It means the mapping must hold *at every position*, left to right, without contradiction. It's not enough to find a mapping that works for some characters — it must work for all of them simultaneously, and it must be a strict bijection: one-to-one in both directions.

Formally, we need a function $f$ such that:

$$\forall\, i,\; f(S[i]) = T[i]$$

And this $f$ must be **injective** (no two characters in $S$ map to the same character in $T$) and must have an **injective inverse** (no two characters in $T$ are claimed by the same character in $S$).

A function that satisfies both is a **bijection**. If either direction fails, the strings are not isomorphic.

---

## Why One Map Is Not Enough

**Student:** Can't I just keep one hash map from $S \to T$ and check for contradictions?

**Teacher:** Let's trace a case that breaks it. Take $S =$ `"bad"`, $T =$ `"bab"`.

Building the map left to right:

| $i$ | $S[i]$ | $T[i]$ | Map $S \to T$ | Contradiction? |
|---|---|---|---|---|
| 0 | `b` | `b` | `{b: b}` | No |
| 1 | `a` | `a` | `{b: b, a: a}` | No |
| 2 | `d` | `b` | `{b: b, a: a, d: b}` | No |

The single map sees no contradiction. It happily accepts `d → b`. But look at what we have: both `b` and `d` point to `b`. Two different inputs share the same output — injectivity is violated.

A one-way map only checks: *does this input already have a different output?* It never checks: *is this output already claimed by a different input?* That second check requires a second map.

---

## The Two-Map Invariant

To enforce a bijection, we maintain two maps simultaneously:

- **Map 1** ($S \to T$): no character in $S$ maps to two different characters in $T$
- **Map 2** ($T \to S$): no character in $T$ is claimed by two different characters in $S$

At each index $i$, before recording a new pair, we check both:

```
if map_ST[S[i]] exists and map_ST[S[i]] ≠ T[i]:
    return False          # S[i] already maps to something else

if map_TS[T[i]] exists and map_TS[T[i]] ≠ S[i]:
    return False          # T[i] is already claimed by someone else

record map_ST[S[i]] = T[i]
record map_TS[T[i]] = S[i]
```

If we reach the end without contradiction, a valid bijection exists.

---

## Tracing the Failure Case

**Student:** Let's trace `"foo"` and `"bar"` through both maps.

**Teacher:** Good. Let's go position by position:

| $i$ | $S[i]$ | $T[i]$ | Map $S \to T$ | Map $T \to S$ | Contradiction? |
|---|---|---|---|---|---|
| 0 | `f` | `b` | `{f: b}` | `{b: f}` | No |
| 1 | `o` | `a` | `{f: b, o: a}` | `{b: f, a: o}` | No |
| 2 | `o` | `r` | — | — | **Yes** |

At $i = 2$: Map 1 already has `o → a`. The new pair tries to set `o → r`. Contradiction. Return `False`.

The structural shape of `"foo"` (two repeated characters) cannot match `"bar"` (all distinct characters). The bijection cannot exist.

---

## Word Pattern: The Same Problem, Different Input Format

**Student:** Word Pattern looks different. The pattern is `"abba"` and the input is `"dog cat cat dog"`.

**Teacher:** It's identical mathematically. The only difference is parsing. Before the bijection check, you split the second string by spaces:

$$\text{"dog cat cat dog"} \;\longrightarrow\; [\text{"dog"},\, \text{"cat"},\, \text{"cat"},\, \text{"dog"}]$$

Now you're mapping characters to words instead of characters to characters. The two-map invariant is exactly the same.

One extra check: after splitting, if `len(pattern) ≠ len(words)`, return `False` immediately. You can't have a bijection between sets of different sizes.

| $i$ | pattern$[i]$ | word$[i]$ | Map $p \to w$ | Map $w \to p$ | Contradiction? |
|---|---|---|---|---|---|
| 0 | `a` | `"dog"` | `{a: dog}` | `{dog: a}` | No |
| 1 | `b` | `"cat"` | `{a: dog, b: cat}` | `{dog: a, cat: b}` | No |
| 2 | `b` | `"cat"` | unchanged | unchanged | No — consistent |
| 3 | `a` | `"dog"` | unchanged | unchanged | No — consistent |

Both maps hold. The pattern is isomorphic to the word sequence.

---

## On Paper First

Before writing code for any structural isomorphism problem, write this down:

1. **State the bijection:** "I need a function $f: \text{char set of } S \to \text{char set of } T$ such that $f(S[i]) = T[i]$ for all $i$."
2. **State the invariant:** "At every step, Map 1 must be internally consistent and Map 2 must be internally consistent."
3. **State the failure conditions:** "If Map 1 or Map 2 reports a contradiction at any index, return False."

The code is just a direct translation of those three lines.

---

## The Complete Algorithm (Pseudocode)

```
function isIsomorphic(S, T):
    if len(S) ≠ len(T): return False

    map_ST = {}
    map_TS = {}

    for i in range(len(S)):
        s, t = S[i], T[i]

        if s in map_ST and map_ST[s] ≠ t: return False
        if t in map_TS and map_TS[t] ≠ s: return False

        map_ST[s] = t
        map_TS[t] = s

    return True
```

For Word Pattern: replace `T[i]` with `words[i]` after splitting. Everything else is identical.

---

## Summary

| Property | Requirement |
|---|---|
| Mapping type | Bijection ($f$ injective + $f^{-1}$ injective) |
| Data structure | Two hash maps |
| Time complexity | $O(N)$ — single pass |
| Space complexity | $O(\|\Sigma\|)$ — size of the character/word set |

> **The structural insight:** Isomorphism is not about the characters themselves — it's about the *shape* of repetition. `"egg"` and `"add"` are isomorphic because their repetition pattern is identical: `xYY`. The bijection is how we formalize that shape.

---

*← Module 1a | Module 1c →*
