# Module 2a — Recurrence Relations and Strong Induction: The Landscape

> **The central shift of Module 2:**
> Module 1 was about *structure* — how data is mapped, grouped, and sliced. Module 2 is about *time* — how an optimal answer at step $i$ is built from optimal answers at steps before it.
>
> Dynamic Programming is not a data structure and it is not an algorithm. It is a mathematical proof technique applied to computation. Once you see it that way, every DP problem becomes the same problem wearing a different costume.

---

## The Core Idea in One Sentence

If you can prove that the optimal answer to a problem of size $i$ can be expressed entirely in terms of optimal answers to strictly smaller problems — and you trust those smaller answers completely — then you never need to search, backtrack, or guess.

You just look backwards, combine what you already know, and write down the result.

---

## Part 1: The Three Things You Must Define Before Writing Any Code

Every DP problem, without exception, requires three definitions written on paper before the code begins. If any of the three is wrong or vague, the code will be wrong even if it compiles.

---

### Definition 1 — The State

The state is a precise mathematical definition of what $f(i)$ represents.

Not "what index am I at" — but "what question is this subproblem answering?"

**Bad state definition:** "$f(i)$ is the answer for index $i$."  
**Good state definition:** "$f(i)$ is the maximum amount of money that can be collected from houses $0$ through $i$, subject to the no-adjacency constraint."

The difference is that the good definition tells you exactly what guarantee $f(i)$ carries. When you use it later in a recurrence, you know precisely what you're trusting.

For 2D problems, the state takes two parameters: $f(i, j)$ — the answer for the subproblem defined by row $i$ and column $j$.

---

### Definition 2 — The Recurrence Relation

The recurrence is an equation expressing $f(i)$ in terms of $f$ at strictly smaller indices.

$$f(i) = \text{some combination of } f(i-1), f(i-2), \ldots$$

This is the mathematical rule that propagates solutions forward. It must cover every possible case — and *only* the possible cases. If a physical constraint exists in the real world (an alarm system, an invalid character, a blocked cell), it must be encoded directly into this equation as a restriction on which terms are allowed.

The recurrence is where the problem's rules live. Not in the code. Here, in this equation.

---

### Definition 3 — The Base Cases

The base cases are the axioms of your proof — the smallest subproblems you can answer directly without further recursion.

$$f(0) = \text{known value}, \quad f(1) = \text{known value}$$

If the base cases are wrong, the entire structure built on top of them is wrong. Strong Induction guarantees that if the base cases are correct and the recurrence is correct, every value computed by the recurrence is correct.

> **The workflow:** State → Recurrence → Base cases → Code. In that order. Always.

---

## Part 2: Strong Induction — Why You Can Trust the Memory Cards

**Student:** I get that the formula looks backwards. But how do I know the values it looks back at are actually correct? What if $f(i-2)$ is wrong?

**Teacher:** This is exactly the right question, and the answer is Strong Induction.

Weak induction says: "If it's true for $n$, it's true for $n+1$."  
Strong induction says: "If it's true for *all* values up to $n$, it's true for $n+1$."

DP uses strong induction. Here's the structure:

**Base step:** Verify that $f(0)$ and $f(1)$ are correct by direct inspection. These are your axioms — you check them by hand.

**Inductive step:** Assume $f(0), f(1), \ldots, f(i-1)$ are all correct (this is the "strong" part — you assume the entire history is correct, not just the previous step). Then show that your recurrence produces the correct $f(i)$.

**Conclusion:** By strong induction, $f(i)$ is correct for all $i$.

This is why DP memory cards can be trusted completely. When the formula looks at $f(i-2)$, it is not trusting a guess — it is trusting a value that has already been proved correct by the same argument, applied to every step before it.

**Student:** So every time the loop adds a new entry to the DP array, it's completing one step of an inductive proof?

**Teacher:** Exactly. The loop is the proof. Each iteration is one application of the inductive step. The final value in the array is the conclusion of the proof.

---

## Part 3: Top-Down vs. Bottom-Up — Two Ways to Execute the Same Proof

The recurrence relation is the mathematics. The direction you compute it is the engineering.

### Top-Down (Memoization) — Start at the Answer, Recurse Down

You begin at $f(N)$ — the final target — and ask recursively for the smaller pieces you need. When a subproblem is solved for the first time, you cache the answer in a map. Every subsequent request for the same subproblem is answered instantly from the cache.

```
memo = {}

function f(i):
    if i in memo: return memo[i]       # cached — return immediately
    if i is a base case: return known value
    result = [recurrence using f(i-1), f(i-2), ...]
    memo[i] = result                   # cache before returning
    return result
```

**What it looks like mathematically:** You're building the proof tree top-down, pruning branches the moment a subproblem has already been proved.

**Pros:**
- Code looks almost identical to the recurrence relation — easy to write
- Only computes subproblems actually needed (sparse graphs benefit from this)

**Cons:**
- Recursion builds a call stack. Deep problems crash with stack overflow
- Slightly higher constant overhead per call

---

### Bottom-Up (Tabulation) — Start at the Base Cases, Build Up

You begin at the base cases — the axioms — and fill in the DP table iteratively, left to right, guaranteeing each entry is ready before it's needed.

```
dp = array of size N+1
dp[0] = base case 0
dp[1] = base case 1

for i from 2 to N:
    dp[i] = [recurrence using dp[i-1], dp[i-2], ...]

return dp[N]
```

**What it looks like mathematically:** You're executing the inductive proof step by step, in order, from the axioms to the conclusion.

**Pros:**
- No recursion — no stack overflow
- Allows space optimization (if the recurrence only looks back $k$ steps, you only need $k$ variables, not the full table)
- Slightly faster in practice — no function call overhead

**Cons:**
- Computes all subproblems, including ones that might not be needed
- Ordering can be tricky for 2D problems (you must fill in the right order)

---

### The Space Optimization

When a recurrence only looks back a fixed number of steps, you don't need to keep the entire table. You only need a window of the last $k$ values.

For any recurrence of the form $f(i) = g(f(i-1), f(i-2))$:

```
# Instead of dp[i], just keep two variables
prev2 = f(0)
prev1 = f(1)

for i from 2 to N:
    curr = recurrence using prev1 and prev2
    prev2 = prev1
    prev1 = curr

return prev1
```

Space drops from $O(N)$ to $O(1)$. This is almost always worth doing once the bottom-up solution is working.

---

### Side-by-Side Comparison

| | Top-Down (Memoization) | Bottom-Up (Tabulation) |
|---|---|---|
| Starting point | $f(N)$ — the target | $f(0)$, $f(1)$ — the axioms |
| Execution | Recursive | Iterative |
| Subproblems computed | Only what's needed | All, in order |
| Stack risk | Yes (deep recursion = overflow) | No |
| Space optimization | Harder | Natural — just shrink the window |
| Interview note | Good starting point | Interviewers expect this as the follow-up |

> **The pro tip:** In interviews, write top-down first — it's faster to get right and looks exactly like the recurrence. Then convert to bottom-up when asked to optimize. They are mathematically identical; the only difference is direction and data structure.

---

## Part 4: What Makes DP Different From Greedy

**Student:** Greedy algorithms also make local choices. What's the difference?

**Teacher:** A greedy algorithm makes the locally best choice at each step and commits to it — it never reconsiders. This works when the locally best choice is always globally optimal (like always taking the heavier coin when making change with certain denominations).

DP doesn't commit to any single choice. It computes *all* options at each step and stores all of them. The "winner" only emerges at the end when you read the final entry.

The reason DP needs to store all options is that a locally suboptimal choice at step $i$ can be part of the globally optimal solution overall. House Robber is the perfect example — skipping a large house now might unlock two even larger houses later. Greedy would grab the large house and pay the price. DP keeps both timelines alive and lets the math decide at the end.

**The formal condition for greedy to work:** The problem must have the *greedy choice property* — the locally optimal choice at each step is provably part of a globally optimal solution. DP requires no such guarantee — it works even when this property fails.

---

## Part 5: The Counting vs. Optimizing Distinction

There are two fundamentally different flavors of DP recurrence:

**Optimization DP:** Find the maximum or minimum value.

$$f(i) = \max\bigl(f(i-1),\; f(i-2) + v_i\bigr)$$

At each step, the parallel universes *compete*. One wins. The other is discarded (but its history lives on in the DP table for future use).

**Counting DP:** Find the total number of ways.

$$f(i) = f(i-1) + f(i-2)$$

At each step, the parallel universes *cooperate*. Both contribute. Their counts are added together because every valid path from the left is a distinct valid path overall.

The structure of the recurrence — $\max$ vs. $+$ — is not a coding choice. It's the direct mathematical consequence of what you're being asked to compute.

---

## The Landscape: Module 2 Problem Classes

| Module | Problem | DP type | State dimension | Recurrence structure |
|---|---|---|---|---|
| **2b** | House Robber | Optimization | 1D | $f(i) = \max(f(i-1),\, f(i-2) + v_i)$ |
| **2b** | Decode Ways | Counting | 1D | $f(i) = f(i-1) + f(i-2)$ with firewalls |
| **2c** | Unique Paths | Counting | 2D | $f(i,j) = f(i-1,j) + f(i,j-1)$ |
| **2c** | Longest Palindromic Substring | Boolean | 2D | $P(i,j) = (S_i = S_j) \land P(i+1,j-1)$ |

---

## The Workflow (Every Time)

```
1. Define the state
   └── f(i) = "the [what?] for the subproblem of size [what?]"
       └── Be precise. Vague states produce wrong recurrences.

2. Write the recurrence
   └── Express f(i) in terms of strictly smaller subproblems.
       └── Encode every real-world constraint as a restriction here.

3. Define the base cases
   └── The smallest subproblems you can answer directly.
       └── Verify them by hand — they are your axioms.

4. Choose direction
   └── Top-down if subproblem graph is sparse or ordering is unclear.
       └── Bottom-up if ordering is obvious and you want to optimize space.

5. Translate to code
   └── The code is a direct transcription of steps 1–4.
       └── If the code is complex, the recurrence wasn't clear enough.
```

---

*Module 2b →*
