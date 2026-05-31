# Module 2c — 2D Transitions: Unique Paths and Longest Palindromic Substring

> **The shift from 1D to 2D:**
> In 1D DP, the state is a single index $i$ — a position along a line. In 2D DP, the state is a pair $(i, j)$ — a position in a grid, or a window within a sequence. The same three-step process applies: define the state, write the recurrence, establish base cases. The only new challenge is filling order — you must compute the table in the right sequence so every value you depend on is already available.

---

## Problem 1: Unique Paths

### The Setup

A robot starts at the top-left corner of an $M \times N$ grid (position $(0, 0)$). It must reach the bottom-right corner (position $(M-1, N-1)$). The only legal moves are: **down** (increase row by 1) or **right** (increase column by 1).

**Goal:** Count the total number of unique paths from start to finish.

---

### Step 1 — Define the State

$$f(i, j) = \text{the number of unique paths from } (0,0) \text{ to cell } (i, j)$$

---

### Step 2 — Write the Recurrence

**Student:** The robot can move down or right. But for the recurrence, we look *backwards* — which cells could the robot have come *from* to arrive at $(i, j)$?

**Teacher:** Exactly right. A robot at $(i, j)$ could only have arrived from two cells:
- $(i-1, j)$ — it was one row above and stepped **down**
- $(i, j-1)$ — it was one column to the left and stepped **right**

Every path that reaches $(i-1, j)$ and every path that reaches $(i, j-1)$ can be extended by one step to reach $(i, j)$. These paths are distinct — no overlap. So we add them:

$$\boxed{f(i, j) = f(i-1, j) + f(i, j-1)}$$

---

### Step 3 — Define the Base Cases

Any cell in the first row ($i = 0$) can only be reached by moving right from $(0, 0)$. There is exactly one path to every cell in the first row.

$$f(0, j) = 1 \quad \text{for all } j$$

Any cell in the first column ($j = 0$) can only be reached by moving down from $(0, 0)$. There is exactly one path to every cell in the first column.

$$f(i, 0) = 1 \quad \text{for all } i$$

These are your axioms — the edges of the grid. The interior fills itself from them.

---

### Filling the Table

With the recurrence and base cases defined, filling order is natural: process rows top to bottom, columns left to right. When you compute $f(i, j)$, both $f(i-1, j)$ (same column, row above) and $f(i, j-1)$ (same row, column to the left) are already computed.

For a $3 \times 3$ grid:

```
Base cases fill the edges:

f(0,0)=1  f(0,1)=1  f(0,2)=1
f(1,0)=1     ?         ?
f(2,0)=1     ?         ?

Recurrence fills the interior:

f(1,1) = f(0,1) + f(1,0) = 1 + 1 = 2
f(1,2) = f(0,2) + f(1,1) = 1 + 2 = 3
f(2,1) = f(1,1) + f(2,0) = 2 + 1 = 3
f(2,2) = f(1,2) + f(2,1) = 3 + 3 = 6
```

Final grid:

| | j=0 | j=1 | j=2 |
|---|---|---|---|
| **i=0** | 1 | 1 | 1 |
| **i=1** | 1 | 2 | 3 |
| **i=2** | 1 | 3 | **6** |

There are 6 unique paths from $(0,0)$ to $(2,2)$.

---

### The Combinatorial Closed Form

**Student:** This is a counting problem on a grid. Is there a direct formula?

**Teacher:** Yes — and it reveals what the recurrence is actually computing under the hood.

To travel from $(0,0)$ to $(M-1, N-1)$, you must take exactly $(M-1)$ down-steps and $(N-1)$ right-steps, in some order. The total path length is always $(M-1)+(N-1)$ steps. The only freedom is which of those steps are down-steps:

$$\text{Unique paths} = \binom{(M-1)+(N-1)}{M-1}$$

For a $3 \times 3$ grid: $\binom{4}{2} = 6$. Matches exactly.

The DP recurrence and the binomial formula are computing the same thing. The recurrence builds the answer iteratively via Pascal's triangle. The formula computes it directly. For grids without obstacles, the formula is $O(1)$. For grids *with* obstacles (blocked cells), the formula breaks — you need the DP because the combinatorial symmetry is destroyed.

---

### Space Optimization

The recurrence $f(i, j) = f(i-1, j) + f(i, j-1)$ only looks at the current row and the row above. You don't need the full $M \times N$ table:

```
dp = [1] * N        # initialize as if filling row 0

for i from 1 to M-1:
    for j from 1 to N-1:
        dp[j] = dp[j] + dp[j-1]
        # dp[j] already holds f(i-1, j) before update
        # dp[j-1] now holds f(i, j-1) — just updated

return dp[N-1]
```

Space: $O(N)$ instead of $O(MN)$.

---

## Problem 2: Longest Palindromic Substring

### The Setup

Given a string $S$ of length $N$, find the longest contiguous substring that is a palindrome.

A string $S[i \ldots j]$ is a palindrome if it reads the same forwards and backwards.

---

### Step 1 — Define the State

$$P(i, j) = \begin{cases} \text{true} & \text{if } S[i \ldots j] \text{ is a palindrome} \\ \text{false} & \text{otherwise} \end{cases}$$

The state is **boolean** — not a count or a maximum, but a truth value. The DP table is a grid of true/false entries. Once filled, we scan it for the longest span where $P(i, j) = \text{true}$.

---

### Step 2 — Write the Recurrence

A string $S[i \ldots j]$ is a palindrome if and only if:
1. The outer characters match: $S[i] = S[j]$
2. The inner substring is also a palindrome: $P(i+1, j-1) = \text{true}$

$$\boxed{P(i, j) = \bigl(S[i] = S[j]\bigr) \;\land\; P(i+1,\, j-1)}$$

---

### Step 3 — Define the Base Cases

**Single characters** are always palindromes:

$$P(i, i) = \text{true} \quad \text{for all } i$$

**Adjacent pairs** are palindromes if the two characters match:

$$P(i, i+1) = \bigl(S[i] = S[i+1]\bigr)$$

---

### The Critical Filling Order

**Student:** In Unique Paths, we filled row by row, top to bottom. Can we do the same here?

**Teacher:** No — and this is the trickiest part of this problem. Look at the recurrence:

$$P(i, j) \text{ depends on } P(i+1, j-1)$$

$P(i+1, j-1)$ has a *larger* row index ($i+1 > i$) and a *smaller* column index ($j-1 < j$). If we fill row by row (increasing $i$), then when we try to compute $P(i, j)$, we haven't computed $P(i+1, j-1)$ yet — it's in a row we haven't reached.

The correct filling order is by **substring length** — fill all substrings of length 1 first, then length 2, then length 3, and so on. When we compute a substring of length $\ell$, all substrings of length $\ell - 2$ (which is what $P(i+1, j-1)$ represents) are already computed.

```
# Fill by increasing length
for length from 1 to N:
    for i from 0 to N - length:
        j = i + length - 1

        if length == 1:
            P[i][j] = true
        elif length == 2:
            P[i][j] = (S[i] == S[j])
        else:
            P[i][j] = (S[i] == S[j]) and P[i+1][j-1]

        if P[i][j] and length > best_length:
            best_start = i
            best_length = length
```

---

### Tracing `"babad"`

Base cases (length 1): all single characters are true.

```
P[0][0]=T  P[1][1]=T  P[2][2]=T  P[3][3]=T  P[4][4]=T
```

Length 2 — adjacent pairs:

| Pair | $S[i]=S[j]$? | $P(i,j)$ |
|---|---|---|
| `"ba"` (0,1) | b ≠ a | false |
| `"ab"` (1,2) | a ≠ b | false |
| `"ba"` (2,3) | b ≠ a | false |
| `"ad"` (3,4) | a ≠ d | false |

Length 3:

| Substring | $S[i]=S[j]$? | $P(i+1,j-1)$ | $P(i,j)$ |
|---|---|---|---|
| `"bab"` (0,2) | b = b ✓ | $P(1,1)$ = true | **true** |
| `"aba"` (1,3) | a = a ✓ | $P(2,2)$ = true | **true** |
| `"bad"` (2,4) | b ≠ d | — | false |

Length 4 and 5: checking these yields no additional palindromes longer than 3.

**Result:** Both `"bab"` and `"aba"` are valid length-3 palindromes. Either is a correct answer.

---

### Why the Filling Order Matters

**Student:** What goes wrong if I fill row by row instead of by length?

**Teacher:** Let's say you're filling row by row and you reach $P(0, 4)$ (the full string `"babad"`).

The recurrence says: $P(0, 4) = (S[0] = S[4]) \land P(1, 3)$.

We need $P(1, 3)$. In row-by-row order, row 1 hasn't been filled yet. We'd be reading an uninitialized cell — garbage data. The wrong filling order silently produces wrong answers.

The dependency graph of the recurrence dictates the filling order. Before writing any 2D DP, draw the arrow: "$P(i,j)$ depends on $P(\_, \_)$." Then ask: in what order must I fill the table so the arrow always points to an already-computed cell?

---

### Comparing the Two 2D Problems

| | Unique Paths | Longest Palindromic Substring |
|---|---|---|
| State type | Integer (count) | Boolean (is/isn't) |
| Recurrence | Adds two neighbors | Logical AND of outer match + inner state |
| Base cases | First row and column | Diagonal (length 1) + sub-diagonal (length 2) |
| Filling order | Row by row, left to right | By substring length (diagonal sweep) |
| What you extract | Bottom-right cell | Longest span where state = true |
| Time complexity | $O(MN)$ | $O(N^2)$ |
| Space complexity | $O(N)$ optimized | $O(N^2)$ (table needed for backtracking) |

---

### On Paper First

For any 2D DP problem:

1. **Define $f(i, j)$** — what problem does this cell solve? What type is the answer (integer, boolean, etc.)?
2. **Write the recurrence** — which other cells does $f(i, j)$ depend on? Draw the dependency arrow.
3. **Define base cases** — which cells are filled directly without the recurrence?
4. **Determine filling order** — the dependency arrow must always point to an already-computed cell. The filling order is the topological sort of the dependency graph.
5. **Determine what to extract** — is the answer in a specific cell (Unique Paths: bottom-right), or do you scan the whole table (Palindrome: longest true span)?

---

### Summary

| | 1D DP (Module 2b) | 2D DP (Module 2c) |
|---|---|---|
| State dimension | Single index $i$ | Index pair $(i, j)$ |
| Table shape | Array of length $N$ | Grid of size $M \times N$ |
| Filling direction | Left to right | Depends on dependencies |
| New challenge | Encoding constraints | Getting the filling order right |
| Space optimization | Keep last $k$ values | Keep last $k$ rows |

> **The structural insight:** 2D DP is not fundamentally harder than 1D DP. The state, recurrence, and base case definitions work exactly the same way. The one new skill is reading the dependency arrows in the recurrence and deriving the correct filling order from them. Get that right, and the rest is mechanical.

---

*← Module 2b | Module 3 →*
