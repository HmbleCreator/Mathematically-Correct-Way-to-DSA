# Module 1a — Mappings: The Full Landscape

> **Before you write a single line of code, you need to ask a different question.**
> Not *"which data structure do I use?"* — but *"what are the mathematical properties of this data, and what does my problem actually need from a mapping?"*
>
> When you see it this way, a hash map stops being a memory trick and starts being a function. And once it's a function, all of mathematics is available to reason about it.

---

## What is a Mapping?

**Student:** I keep hearing "just use a hash map." Is there a real reason, or is it pattern-matching?

**Teacher:** There's a very precise reason. A hash map is a function:

$$f: A \to B$$

$A$ is the set of keys (the **domain**). $B$ is the set of values (the **codomain**). The entries you've actually inserted — those are the **range**, a subset of $B$.

When you write `map[key] = value`, you're defining one assignment of that function. The question to ask before you reach for one is: *what mathematical property does my problem require from this function?*

---

## The Three Terms You Must Know

These three terms define the *type signature* of any function. Confusing them leads to bugs that look like logic errors but are actually violations of a mathematical contract.

| Term | What it means | Example: $f(x) = x^2$ |
|---|---|---|
| **Domain** | The set of valid inputs | $\mathbb{R}$ |
| **Codomain** | The declared output space | $\mathbb{R}$ |
| **Range (Image)** | The outputs actually produced | $[0, \infty)$ |

The formal relationship is always:

$$\text{Im}(f) \subseteq \text{Codomain}(f)$$

**Student:** Why does the codomain even exist if the range is what actually comes out?

**Teacher:** Because a function's *type declaration* and its *actual behavior* are two different things. When you declare a function returns an integer, that's the codomain. What integers it actually returns — that's the range. The gap between them isn't an error; it's information. A function $f: \mathbb{Z} \to \mathbb{Z}$ whose range is only even numbers is telling you something structural about $f$.

> **Key rule:** The range is always a subset of the codomain. They are equal *only* when the function is surjective (onto) — meaning every output in the codomain is actually hit by some input.

---

## The Four Structural Properties

Every mapping falls into one of these four shapes. Identifying which shape your problem requires tells you what constraints to enforce — before you think about code.

---

### 1. Injective (One-to-One)

> No two inputs map to the same output.

$$a \neq b \implies f(a) \neq f(b)$$

Each element of the domain has a *unique* image. The function never "collides."

**Real-world example:** Passport numbers. No two people share one. If you know the number, you can trace it back to exactly one person.

**DSA example:** A hash map tracking `value → index` in a Two Sum problem. Each value you've seen maps to a unique index.

---

### 2. Surjective (Onto)

> Every element of the codomain is hit by at least one input.

$$\forall\, b \in B,\; \exists\, a \in A : f(a) = b$$

Nothing in the declared output space is left unused. The range fills the entire codomain.

**Real-world example:** A gym where every locker is currently being used by at least one member.

**DSA example:** Rare in algorithmic problems. Surjectivity matters when you're *constructing* a mapping and need to prove nothing in the target set is unreachable.

---

### 3. Bijective (Perfect Pairing)

> Both injective *and* surjective. A perfect one-to-one correspondence.

$$f \text{ is bijective} \iff f \text{ is injective} \land f \text{ is surjective}$$

Every input maps to a unique output, and every output is claimed by exactly one input. The function is *invertible* — you can run it backwards.

**Real-world example:** DNS. Each domain name maps to exactly one IP address, and each IP address maps to exactly one canonical domain.

**DSA example:** Isomorphic Strings. You're proving that a valid bijection exists between the character sets of two strings. This is why you need *two* hash maps — one for each direction.

---

### 4. Neither (General Function)

> Valid inputs, valid outputs, but no uniqueness or coverage guarantees.

A function doesn't have to be injective or surjective to be a function. Most hash functions in practice are neither — they deliberately collapse a large domain onto a small codomain. A frequency map is neither injective (many elements can have count = 3) nor surjective (not every natural number need appear as a count).

---

## Putting It Together

**Student:** So when I see a new problem, I first figure out which of these four my mapping needs to be?

**Teacher:** Exactly. And here's the key insight: once you know the required structural property, the data structure isn't a *decision* anymore — it's a *consequence*.

- A bijection requires enforcing uniqueness in *both directions*. Two maps.
- A frequency count produces a function into $\mathbb{N}$. One map with integer values.
- A complement search defines a set $S$ and asks membership queries. One map (or set) with O(1) lookup.

You don't choose the structure. The math hands it to you.

---

## The Landscape of Mapping Problems

Every hash map problem in DSA is asking one of four questions about a mapping:

| Class | The question being asked | What you're enforcing |
|---|---|---|
| **Structural Isomorphism** | Does a bijection $f: A \to B$ consistent with the sequence exist? | Injectivity in both directions |
| **Complement Search** | For each $x \in A$, does $(\text{target} - x) \in S$? | Set membership |
| **Frequency Counting** | What is $f(e) = \|\{i : A[i] = e\}\|$ for each element? | A function into $\mathbb{N}$ |
| **State Caching** | Has this subproblem already been solved? | Partial function completion |

Each of these gets its own module. In the next sections, we solve one problem class at a time — first on paper, then in code.

---

## A Note on Arrays

Before we move to problem classes, one more piece of the landscape: arrays are not just hash maps with integer keys. They are something geometrically richer.

An array of $N$ numbers is a **vector in $\mathbb{R}^N$**:

$$v = [v_1,\; v_2,\; \ldots,\; v_N]$$

A contiguous subarray (a "window") is a sub-vector. The sum of a window is a scalar projection of that sub-vector. When you slide a window of size $k$ across the array, you're not doing repeated addition from scratch — you're performing an incremental update:

$$\text{sum}_{\text{new}} = \text{sum}_{\text{old}} - v_{\text{outgoing}} + v_{\text{incoming}}$$

This geometric view — arrays as vectors, windows as projections — is the foundation of the **Sliding Window** technique. It belongs to the same module because arrays and hash maps are both implementations of the same mathematical idea: a function from an index (or key) to a value.

The sliding window will be covered in its own section once we've worked through the four mapping problem classes.

---

## What's Next

This module is organized as one section per problem class:

- **Module 1b** — Structural Isomorphism: proving bijections (Isomorphic Strings, Word Pattern)
- **Module 1c** — Complement Search: set membership queries (Two Sum, 3Sum, 4Sum)
- **Module 1d** — Frequency Counting: functions into $\mathbb{N}$ (Majority Element, Group Anagrams)
- **Module 1e** — State Caching: memoization and tabulation (Fibonacci, Climbing Stairs)
- **Module 1f** — Vector Space and Sliding Window (Minimum Subarray Sum, Longest Repeating Replacement)

For each section, the workflow is the same:

1. State the mathematical question the problem is asking.
2. Identify the required structural property of the mapping.
3. Derive the data structure as a consequence.
4. Write the invariant or transition on paper.
5. Translate to code last.

> *The code is just the final translation. The real work happens before you open an editor.*

---

*Module 1b →*