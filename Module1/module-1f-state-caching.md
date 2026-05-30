# Module 1f — State Caching: Memoization and Tabulation

> **The question this class is always asking:**
> Has this subproblem already been solved? If yes, return the cached answer. If no, solve it, cache the answer, and return it.

---

## The Mathematical Setup

**Student:** What makes a problem a "state caching" problem?

**Teacher:** Two properties must hold simultaneously:

1. **Optimal substructure** — the solution to a problem can be expressed in terms of solutions to smaller versions of the same problem.
2. **Overlapping subproblems** — those smaller versions appear repeatedly across different branches of the computation.

When both hold, naive recursion recomputes the same subproblem exponentially many times. State caching — storing solved subproblems in a map — transforms that exponential work into linear work.

The map is a partial function:

$$\text{memo}: \text{subproblem parameters} \to \text{answer}$$

Initially empty. It fills lazily, entry by entry, as subproblems are solved for the first time.

---

## The Cost of Ignoring Overlap

Take the Fibonacci sequence: $F(n) = F(n-1) + F(n-2)$, with $F(0) = 0$, $F(1) = 1$.

To compute $F(5)$ naively:

```
F(5)
├── F(4)
│   ├── F(3)
│   │   ├── F(2) ← computed here
│   │   └── F(1)
│   └── F(2) ← recomputed
└── F(3) ← recomputed entirely
    ├── F(2) ← recomputed again
    └── F(1)
```

$F(3)$ is computed twice. $F(2)$ is computed three times. For $F(50)$, $F(3)$ is recomputed over five million times. The time complexity is $O(2^N)$.

The work is real. The tree is exponential. The answers are identical every time. This is pure waste.

---

## Top-Down: Memoization

Add a check before every computation: "Have I already solved this?"

```
memo = {}

function F(n):
    if n in memo: return memo[n]          # already solved
    if n <= 1: return n                    # base cases
    result = F(n-1) + F(n-2)
    memo[n] = result                       # cache before returning
    return result
```

Now the computation tree is pruned:

```
F(5)
├── F(4)
│   ├── F(3)
│   │   ├── F(2) ← computed, cached
│   │   └── F(1)
│   └── F(2) ← cache hit, O(1)
└── F(3) ← cache hit, O(1)
```

Every subproblem is solved exactly once. Time complexity: $O(N)$. Space: $O(N)$ for the memo map.

**Student:** The memo is the same kind of map we've been building all module?

**Teacher:** Exactly. It's a function $f: \mathbb{N} \to \mathbb{N}$ mapping each subproblem index to its answer. The difference from frequency counting or complement search is that we're constructing this map *lazily* — only filling in values we actually need, in the order the recursion demands them.

---

## Bottom-Up: Tabulation

Memoization is top-down: start at $F(N)$ and recurse downward. Tabulation is bottom-up: start at the base cases and build upward using a loop.

```
table = [0] * (n + 1)
table[0] = 0
table[1] = 1

for i from 2 to n:
    table[i] = table[i-1] + table[i-2]

return table[n]
```

No recursion. No function call overhead. We fill the table in order, guaranteeing that when we compute `table[i]`, the values it depends on (`table[i-1]` and `table[i-2]`) are already filled.

**Space optimization:** If the recurrence only looks back a fixed number of steps, you don't need the full table. For Fibonacci, you only need the last two values:

```
a, b = 0, 1
for i from 2 to n:
    a, b = b, a + b
return b
```

Space drops from $O(N)$ to $O(1)$.

---

## Choosing Top-Down vs. Bottom-Up

| | Memoization (Top-Down) | Tabulation (Bottom-Up) |
|---|---|---|
| Direction | Start at the target, recurse down | Start at base cases, build up |
| Code style | Recursive with a cache | Iterative with a table |
| Subproblems computed | Only the ones actually needed | All subproblems in order |
| Space | $O(N)$ for memo + call stack | $O(N)$ for table (reducible) |
| Best for | Sparse subproblem graphs | Dense, well-ordered dependencies |

**Student:** When do you prefer one over the other?

**Teacher:** Use top-down when you're not sure which subproblems you'll actually need — it's easier to reason about and avoids computing unnecessary entries. Use bottom-up when the ordering is obvious and you want to avoid recursion stack overhead. For most standard DP problems, bottom-up is slightly faster in practice. For problems with irregular subproblem graphs (like memoized DFS on a graph), top-down is much cleaner.

---

## The Three Questions Before You Write Any DP

1. **What is the state?** Define $f(i)$ (or $f(i, j)$ for 2D) precisely: what problem is this subproblem solving?

2. **What is the recurrence?** Express $f(i)$ in terms of strictly smaller subproblems. Write this as an equation before touching code.

3. **What are the base cases?** These are your axioms — the ground truth the induction builds on. If the base cases are wrong, the entire structure collapses.

For Fibonacci:
- State: $f(n)$ = the $n$-th Fibonacci number
- Recurrence: $f(n) = f(n-1) + f(n-2)$
- Base cases: $f(0) = 0$, $f(1) = 1$

For House Robber:
- State: $f(i)$ = maximum amount robbable from houses $0$ through $i$
- Recurrence: $f(i) = \max(f(i-1),\; f(i-2) + v_i)$
- Base cases: $f(0) = v_0$, $f(1) = \max(v_0, v_1)$

The code is a direct translation of these three lines. If you can write them precisely on paper, the implementation is mechanical.

---

## On Paper First

Before writing code for any state caching problem:

1. **Verify the two conditions:** Does this problem have optimal substructure? Do subproblems overlap?
2. **Define the state clearly:** "$f(i)$ is the [what?] for input of size [what?]."
3. **Write the recurrence:** $f(i) = \ldots$ in terms of $f(i-1)$, $f(i-2)$, etc.
4. **Write the base cases:** The smallest inputs you can answer directly.
5. **Choose direction:** Top-down (memo) or bottom-up (tabulation)?

---

## Summary

| Property | Value |
|---|---|
| Mapping type | Partial function (subproblem → answer), filled lazily |
| Data structure | Hash map (memo) or array (table) |
| Key property | Each subproblem solved exactly once |
| Time complexity | $O(\text{number of distinct subproblems} \times \text{cost per subproblem})$ |
| Space (memoization) | $O(\text{number of distinct subproblems})$ |
| Space (tabulation) | Reducible to the "lookback depth" of the recurrence |

> **The structural insight:** Memoization converts an exponential recursion tree into a DAG — a directed acyclic graph where each node is a subproblem and edges are dependencies. Each node is computed exactly once. The memo map is a cache of the DAG's computed values.

---

*← Module 1e | Module 1g →*
