# Module 1a — Set Theory and Mappings: The Full Landscape

> **Before you write a single line of code, you need to ask a different question.**
> Not *"which data structure do I use?"* — but *"what are the mathematical properties of this data, and what transformation does my problem require?"*
>
> When you approach DSA this way, it stops looking like a collection of coding tricks and starts looking like the geometric interpretability of embeddings, or the logic behind graph-based retrieval engines. The mathematics was always there. You were just taught to skip it.

---

## What This Module Covers

Module 1 is built on three mathematical foundations that underlie hash tables, arrays, and union-find:

| Foundation | Mathematical object | DSA manifestation |
|---|---|---|
| **Set theory and mappings** | Functions $f: A \to B$ | Hash maps, dictionaries |
| **Equivalence relations** | Partitions of a set | Disjoint Set Union (DSU) |
| **Vector spaces** | Vectors in $\mathbb{R}^N$ | Arrays, sliding window |

These aren't three separate topics — they're three ways of asking the same question: *what is the structure of this data, and what operation does this problem need?*

This file (1a) is the landscape. Every subsequent section in Module 1 is a deep-dive into one problem class that lives on this map.

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

## Part 4: Equivalence Relations and Partitions

**Student:** Hash maps handle functions from elements to values. But some problems aren't about values at all — they're about grouping. Like: these users are the same person, those computers are in the same network. How does that fit?

**Teacher:** That's a different mathematical object entirely — an **equivalence relation**.

A relation $\sim$ on a set $S$ is an equivalence relation if it satisfies three properties:

| Property | Formal statement | Plain meaning |
|---|---|---|
| **Reflexive** | $a \sim a$ | Every element is related to itself |
| **Symmetric** | $a \sim b \implies b \sim a$ | The relation goes both ways |
| **Transitive** | $a \sim b \land b \sim c \implies a \sim c$ | Relatedness propagates through chains |

When these three hold, the relation **partitions** $S$ into non-overlapping groups called **equivalence classes**. Every element belongs to exactly one class. Elements inside a class are all mutually related; elements in different classes share no relation.

**Student:** Why does transitivity matter so much?

**Teacher:** Because it's what makes grouping global instead of local. If john@gmail.com appears in Account 1 alongside john_work@gmail.com, and john_work@gmail.com appears in Account 2 alongside john_spam@gmail.com — then by transitivity, all three belong to the same person, even though john@gmail.com and john_spam@gmail.com never appeared together directly. Without transitivity, you'd only merge pairwise. With it, you merge entire connected components.

### Disjoint Set Union (DSU) — The Implementation

**Disjoint Set Union** (also called Union-Find) is the programmatic implementation of equivalence class partitioning. It maintains the partition dynamically as new connections are declared.

Two operations define it:

- **Union($a$, $b$):** Declare $a \sim b$. If they're in different classes, merge those classes.
- **Find($a$):** Return the canonical representative of $a$'s equivalence class.

Two elements are in the same class if and only if `Find(a) == Find(b)`.

**Why it belongs in Module 1:** DSU is not a graph algorithm — it's a set theory algorithm. You're partitioning a set according to a relation. The "graph" interpretation (nodes and edges) is just one domain where this partition problem arises. The same structure appears in merging accounts, detecting redundant connections, and grouping images by content. The common thread is always: *partition this set by this equivalence relation*.

**Student:** So DSU is for problems where the question is "are these two things in the same group?"

**Teacher:** Exactly. And it's faster than anything you could build by scanning arrays. With path compression and union by rank, each operation costs effectively $O(1)$ — formally $O(\alpha(N))$ where $\alpha$ is the inverse Ackermann function, a value that never exceeds 4 for any input you'll ever see in practice.

---

## Putting It Together

**Student:** So when I see a new problem, I first figure out which of these four my mapping needs to be?

**Teacher:** Exactly. And here's the key insight: once you know the required structural property, the data structure isn't a *decision* anymore — it's a *consequence*.

- A bijection requires enforcing uniqueness in *both directions*. Two maps.
- A frequency count produces a function into $\mathbb{N}$. One map with integer values.
- A complement search defines a set $S$ and asks membership queries. One map (or set) with O(1) lookup.

You don't choose the structure. The math hands it to you.

---

## Part 5: Arrays as Vectors

Before we move to problem classes, one more piece of the landscape: arrays are not just hash maps with integer keys. They are something geometrically richer.

An array of $N$ numbers is a **vector in $\mathbb{R}^N$**:

$$v = [v_1,\; v_2,\; \ldots,\; v_N]$$

A contiguous subarray (a "window") is a sub-vector. The sum of a window is a scalar projection of that sub-vector. When you slide a window of size $k$ across the array, you're not doing repeated addition from scratch — you're performing an incremental update:

$$\text{sum}_{\text{new}} = \text{sum}_{\text{old}} - v_{\text{outgoing}} + v_{\text{incoming}}$$

For a dynamic window, two pointers $\ell$ and $r$ define the sub-vector $\mathbf{v}[\ell:r]$. They never move backwards. Each element enters the window once and exits once. Total work: $O(N)$ regardless of how the window expands and contracts.

This geometric view — arrays as vectors, windows as projections — is the foundation of the **Sliding Window** technique. It belongs to the same module because arrays and hash maps are both implementations of the same mathematical idea: a function from an index (or key) to a value.

The sliding window will be covered in its own section once we've worked through the four mapping problem classes.

---

## Part 6: The Full Landscape

With all three foundations in place, here is every problem class in Module 1:

| Module | Class | Mathematical question | Required property | Data structure |
|---|---|---|---|---|
| **1b** | Structural Isomorphism | Does a bijection $f: A \to B$ consistent with the sequence exist? | Bijection (two-way injectivity) | Two hash maps |
| **1c** | Complement Search | For each $x$, does $(\text{target} - x) \in S$? | Set membership | Hash map or set |
| **1d** | Equivalence Classes | What are the maximal groups under this equivalence relation? | Partition by equivalence | DSU (Union-Find) |
| **1e** | Frequency Counting | What is $f(e) = \|\{i : A[i] = e\}\|$ for each $e$? | Function into $\mathbb{N}$ | Hash map |
| **1f** | State Caching | Has this subproblem already been solved? | Partial function, filled lazily | Hash map (memo) or array (table) |
| **1g** | Vector / Sliding Window | What is the optimal sub-vector satisfying a constraint on its projection? | Incremental projection | Two pointers |

Each of these gets its own module. In the next sections, we solve one problem class at a time — first on paper, then in code.

---

## What's Next

This module is organized as one section per problem class:

- **Module 1b** — Structural Isomorphism: proving bijections (Isomorphic Strings, Word Pattern)
- **Module 1c** — Complement Search: set membership queries (Two Sum, 3Sum, 4Sum)
- **Module 1d** — Equivalence Classes: partitioning by relation (Accounts Merge, Redundant Connection)
- **Module 1e** — Frequency Counting: functions into $\mathbb{N}$ (Majority Element, Group Anagrams)
- **Module 1f** — State Caching: memoization and tabulation (Fibonacci, Climbing Stairs)
- **Module 1g** — Vector Space and Sliding Window (Minimum Subarray Sum, Longest Repeating Character Replacement)

For each section, the workflow is the same:

1. State the mathematical question the problem is asking.
2. Identify the required structural property of the mapping.
3. Derive the data structure as a consequence.
4. Write the invariant or transition on paper.
5. Translate to code last.

> *The code is just the final translation. The real work happens before you open an editor.*

---

*Module 1b →*