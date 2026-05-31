# Module 2b — 1D Recurrence States: House Robber and Decode Ways

> **The question 1D DP is always asking:**
> Given that I know the optimal answer for every prefix up to $i-1$, what is the optimal answer for the prefix up to $i$?

---

## Problem 1: House Robber

### The Setup

You are walking down a street of $N$ houses. Each house $i$ contains $v_i$ dollars. The constraint: you cannot rob two adjacent houses — the alarm connects them. Formally:

> If you rob house $i$, you cannot rob house $i-1$ or house $i+1$.

**Goal:** Find the maximum total you can collect.

---

### Step 1 — Define the State

$$f(i) = \text{the maximum amount collectable from houses } 0 \text{ through } i$$

This is a precise guarantee. $f(i)$ doesn't say "the best path that includes house $i$." It says "the best possible outcome considering all houses up to and including position $i$, whether we rob house $i$ or not."

---

### Step 2 — Write the Recurrence

When you stand at house $i$, there are exactly two physical possibilities. No more, no less.

**Universe A — Skip house $i$:** You don't rob it. Your total is unchanged from the best you had at house $i-1$.

$$\text{Universe A} = f(i-1)$$

**Universe B — Rob house $i$:** You take $v_i$. But the alarm rule means you couldn't have robbed house $i-1$. So your base is the best total from house $i-2$.

$$\text{Universe B} = f(i-2) + v_i$$

There is no Universe C. Either you rob house $i$ or you don't. The recurrence is:

$$\boxed{f(i) = \max\bigl(f(i-1),\; f(i-2) + v_i\bigr)}$$

---

### Step 3 — Define the Base Cases

The recurrence needs $f(i-2)$ and $f(i-1)$. At $i=0$, $f(-2)$ and $f(-1)$ don't exist. We initialize directly:

$$f(0) = v_0 \qquad \text{(only one house — just rob it)}$$

$$f(1) = \max(v_0, v_1) \qquad \text{(two houses — rob the richer one)}$$

These are verified by hand. They need no formula to derive — they are the axioms.

---

### The Adjacency Rule Is Baked Into the Formula

**Student:** I understand the formula logically, but how does a computer "know" the alarm rule? It doesn't know what a house is.

**Teacher:** It doesn't need to. Look very carefully at where the $+ v_i$ appears in the recurrence:

$$f(i) = \max\bigl(\underbrace{f(i-1)}_{\text{no } +v_i},\; \underbrace{f(i-2) + v_i}_{\text{skips } i-1}\bigr)$$

The left option uses $f(i-1)$ — but there is no $+ v_i$ attached to it. You either use yesterday's history *or* you rob today's house. The formula never writes $f(i-1) + v_i$ because we deliberately left it out. We didn't give the computer the option to break the alarm rule. It is mathematically blind to that possibility.

The adjacency constraint is not enforced by an if-statement. It is enforced by the *structure of the equation itself*.

---

### Why the Formula Guarantees the Global Maximum

**Student:** How can looking back just two steps guarantee the globally optimal answer for 10,000 houses?

**Teacher:** Because $f(i-1)$ is not just the cash in the previous house. It is a **memory card** that says: *"I have already solved every house from 0 to $i-1$ optimally. The absolute maximum you can collect up to this exact point is this number."*

When you look at $f(i-1)$ and $f(i-2)$, you are not looking at two houses. You are looking at two complete, provably optimal solutions to smaller versions of the same problem. Strong induction guarantees they are correct. The $\max$ of two correct, exhaustive options is itself correct.

$f(i)$ doesn't look forward. It doesn't need to. The future will look *back* at $f(i)$ the same way $f(i)$ looks back at its predecessors.

---

### Why Skipping 2 Is Legal but Skipping 3 Is Never Optimal

**Skipping 2 houses** — consider `[10, 1, 1, 10]`:

| $i$ | $v_i$ | Universe A: $f(i-1)$ | Universe B: $f(i-2) + v_i$ | $f(i)$ |
|---|---|---|---|---|
| 0 | 10 | — | — | 10 |
| 1 | 1 | — | — | $\max(10, 1) = 10$ |
| 2 | 1 | $f(1) = 10$ | $f(0) + 1 = 11$ | **11** |
| 3 | 10 | $f(2) = 11$ | $f(1) + 10 = 20$ | **20** |

At $i=3$, the formula chose $f(1) + 10 = 20$. $f(1)$ itself came from house 0. The physical path is house 0 → house 3. Two houses were skipped. The formula allowed this naturally — not because it knows about skipping, but because $f(1) + 10 > f(2)$.

**Skipping 3 houses** — say a robber takes house $A$, skips $B$, $C$, $D$, then takes house $E$.

Look at house $C$. Its left neighbor is $B$ — not robbed. Its right neighbor is $D$ — not robbed. No alarm reaches $C$. Since $v_C \geq 0$, the robber could take house $C$ for free. A plan that skips $C$ is *valid* but cannot be *optimal* — it leaves guaranteed non-negative money uncollected.

The formula prevents this not through an explicit rule, but through coverage. The recurrence visits every house and at each step absorbs any "free money" sitting in an unguarded position. It is mathematically impossible to finish the loop with a 3-house gap intact — the formula would have exploited it.

---

### Parallel Universes, Not Parallel Loops

**Student:** It feels like the algorithm runs two separate paths — one that robbed house $i$ and one that didn't. Are there two loops?

**Teacher:** One loop. Two lookups. The "parallel universes" don't exist as active computation threads. They exist as two already-computed entries in the DP table — entries that were solved and written down in previous iterations.

At step $i$, you do two table reads and one comparison. That's it. The entire history of every possible valid path is compressed into these two numbers. The branching that would cost $O(2^N)$ naively is collapsed into $O(1)$ work per step because every subproblem was solved exactly once and stored.

---

### Bottom-Up Implementation

```
function houseRobber(v):
    N = len(v)
    if N == 0: return 0
    if N == 1: return v[0]

    prev2 = v[0]               # f(0)
    prev1 = max(v[0], v[1])    # f(1)

    for i from 2 to N-1:
        curr = max(prev1, prev2 + v[i])
        prev2 = prev1
        prev1 = curr

    return prev1
```

Space: $O(1)$ — we only keep the last two values because the recurrence looks back exactly two steps.

---

## Problem 2: Decode Ways

### The Setup

The mapping: $A=1, B=2, \ldots, Z=26$. Given a string of digit characters, count the total number of valid ways to decode the entire string.

This is the **counting** flavor of 1D DP. Instead of competing for a maximum, valid decodings cooperate — they add together.

---

### Step 1 — Define the State

$$f(i) = \text{the number of valid ways to decode the prefix } S[0 \ldots i-1]$$

Note: $f(i)$ counts complete decodings of the first $i$ characters. Every digit in the prefix must be consumed — no digits left floating, no digits skipped.

---

### Step 2 — Write the Recurrence

At position $i$, there are two ways the last decode step could have arrived here:

**Single-digit step:** The last character $S[i-1]$ was decoded alone. This is valid if $S[i-1] \in \{1, \ldots, 9\}$ (not `'0'`). Every valid decoding of the prefix $S[0 \ldots i-2]$ can be extended by this one character.

$$\text{if } S[i-1] \neq \text{'0'}: \quad +f(i-1)$$

**Double-digit step:** The last two characters $S[i-2]\,S[i-1]$ were decoded together. This is valid if the two-digit number is between 10 and 26. Every valid decoding of $S[0 \ldots i-3]$ can be extended.

$$\text{if } 10 \leq \text{int}(S[i-2:i]) \leq 26: \quad +f(i-2)$$

The full recurrence (with firewalls):

$$\boxed{f(i) = [\text{single valid}] \cdot f(i-1) \;+\; [\text{double valid}] \cdot f(i-2)}$$

where the brackets are 1 if the condition holds, 0 otherwise.

---

### Step 3 — Define the Base Cases

$$f(0) = 1 \qquad \text{(the empty prefix — one way to decode nothing)}$$

$$f(1) = \begin{cases} 0 & \text{if } S[0] = \text{'0'} \\ 1 & \text{otherwise} \end{cases}$$

$f(0) = 1$ is the standard sentinel. It acts as the multiplicative identity — when a valid single or double step reaches all the way back to the beginning, it contributes exactly 1 path, not 0.

---

### The Zero Firewall

**Student:** What actually happens when the algorithm hits a `'0'`?

**Teacher:** The zero is not an error — it's a firewall that the formula evaluates honestly.

Take the string `"30"`. At position $i=2$ (the `'0'`):

- **Single digit:** Is `'0'` between 1 and 9? No. Contributes $0 \cdot f(1) = 0$.
- **Double digit:** Is `30` between 10 and 26? No. Contributes $0 \cdot f(0) = 0$.

$f(2) = 0 + 0 = 0$.

The memory card records zero valid decodings. Not an error — a mathematical truth. There are genuinely zero ways to decode `"30"` in the A–Z mapping.

When a subsequent character tries to extend from this position, it multiplies by $f(2) = 0$ and also gets 0. The broken bridge propagates forward automatically. No special error handling needed — the arithmetic does it.

---

### Tracing `"226"`

| $i$ | Characters | Single check | Double check | $f(i)$ | Meaning |
|---|---|---|---|---|---|
| 0 | (empty) | — | — | 1 | sentinel |
| 1 | `"2"` | `'2'` valid | — | 1 | one way: `[2]` |
| 2 | `"22"` | `'2'` valid → $+f(1)=1$ | `22` ≤ 26 → $+f(0)=1$ | 2 | `[2,2]` or `[22]` |
| 3 | `"226"` | `'6'` valid → $+f(2)=2$ | `26` ≤ 26 → $+f(1)=1$ | **3** | `[2,2,6]`, `[22,6]`, `[2,26]` |

Three valid decodings. The formula counted all three paths without ever constructing or storing them.

---

### The Critical Distinction: Counting Paths vs. Storing Paths

**Student:** I see 1 way for 1, 1 way for 5, 1 way for 15 — that feels like 3 ways.

**Teacher:** You're counting valid *pieces*, not valid *complete decodings*. The rule is that every digit in the string must be consumed, left to right, in a single continuous read. You cannot pick pieces from different positions simultaneously.

For `"152"`:

| Complete decoding | Path | Valid? |
|---|---|---|
| 1 → 5 → 2 | A, E, B | Yes |
| 15 → 2 | O, B | Yes |
| 1 → 52 | A, ? | No — 52 > 26 |

That's 2 complete decodings. The DP records 2, not 3. The pieces you identified (1, 5, 15, 2) are *possible steps* — some lead to complete paths, some dead-end. The DP counts only paths that successfully consume the entire string.

---

### Bottom-Up Implementation

```
function decodeWays(S):
    N = len(S)
    if N == 0 or S[0] == '0': return 0

    prev2 = 1              # f(0)
    prev1 = 1              # f(1) — S[0] != '0' already checked

    for i from 2 to N:
        curr = 0
        if S[i-1] != '0':
            curr += prev1                          # single-digit step
        two_digit = int(S[i-2:i])
        if 10 <= two_digit <= 26:
            curr += prev2                          # double-digit step
        prev2 = prev1
        prev1 = curr

    return prev1
```

Time: $O(N)$. Space: $O(1)$.

---

### Side-by-Side: House Robber vs. Decode Ways

| | House Robber | Decode Ways |
|---|---|---|
| DP type | Optimization | Counting |
| Recurrence combinator | $\max$ — universes compete | $+$ — universes cooperate |
| Constraint | Adjacency (no $f(i-1) + v_i$) | Validity (zero firewall) |
| State | Max money up to $i$ | Number of valid paths to $i$ |
| Lookback depth | 2 | 2 |
| Space-optimized | 2 variables | 2 variables |

> **The structural insight:** Both problems look back exactly two steps. Both use two variables in the optimized form. The difference is entirely in what you're computing — max vs. count — which determines whether the two options compete ($\max$) or cooperate ($+$).

---

### On Paper First

For any 1D DP problem:

1. **Define $f(i)$** with full precision — what guarantee does this number carry?
2. **Enumerate all possible last steps** — how could the optimal solution have arrived at position $i$? These become the terms in the recurrence.
3. **Encode constraints as restrictions** on which terms are allowed — not as if-statements after the fact.
4. **Set base cases** — the smallest $i$ values you can answer directly. Verify by hand.
5. **Identify the combinator** — are you maximizing (use $\max$) or counting (use $+$)?

---

*← Module 2a | Module 2c →*
