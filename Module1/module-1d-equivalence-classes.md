# Module 1d — Equivalence Classes: Disjoint Set Union

> **The question this class is always asking:**
> Given a set of elements and a connectivity relation, what are the maximal groups of mutually connected elements?

---

## The Mathematical Setup

An **equivalence relation** $\sim$ on a set $S$ is a relation satisfying three properties:

| Property | Meaning |
|---|---|
| **Reflexive** | $a \sim a$ — every element is connected to itself |
| **Symmetric** | $a \sim b \implies b \sim a$ — connection is mutual |
| **Transitive** | $a \sim b$ and $b \sim c \implies a \sim c$ — connection propagates |

When a relation satisfies all three, it partitions $S$ into non-overlapping groups called **equivalence classes**. Every element belongs to exactly one class. Elements in the same class are all mutually connected; elements in different classes share no connection.

**DSU** (Disjoint Set Union, also called Union-Find) is the programmatic implementation of this partition. It maintains the classes dynamically as new connections are declared.

---

## Islands and Bridges

**Student:** Can we get a concrete model for this?

**Teacher:** Think of nodes as islands and edges as bridges. An equivalence class is an **archipelago** — a group of islands you can reach from any other island in the group by crossing bridges. Islands with no bridges form their own single-island archipelago.

The connectivity relation is: $A \sim B$ if there exists any sequence of bridges connecting $A$ to $B$. Transitivity is automatic — if you can reach $B$ from $A$, and $C$ from $B$, you can reach $C$ from $A$.

The two operations we need:

- **Union($A$, $B$):** Build a bridge between $A$ and $B$. If they were in different archipelagos, merge those archipelagos.
- **Find($A$):** Return the canonical representative (the "capital island") of $A$'s archipelago.

Two islands are in the same equivalence class if and only if `Find(A) == Find(B)`.

---

## The Data Structure

Each node stores a pointer to its **parent** — the next island up in its archipelago's internal tree. The root of the tree (the node pointing to itself) is the representative.

Initial state: every node is its own representative.

```
parent[i] = i    for all i
```

**Find:** Walk up the parent chain until you reach a node pointing to itself.

```
function Find(x):
    if parent[x] == x: return x
    return Find(parent[x])
```

**Union:** Find the roots of both nodes. If they differ, make one root point to the other.

```
function Union(x, y):
    rx = Find(x)
    ry = Find(y)
    if rx ≠ ry:
        parent[rx] = ry
```

---

## Why We Connect Roots, Not the Common Node

**Student:** If two sets overlap at a shared node, why not just connect at that node instead of finding the roots?

**Teacher:** Let's trace what happens. Say Set 1 is `{a, b, c, d, e}` with root `a`, and Set 2 is `{p, q, r, s, d}` with root `p`. The shared node is `d`.

If we connect at `d`, we get a chain: `s → p ... → d → ... → a`. Now `q` must walk: `q → p → d → a` to find its root. The tree grows tall and searches grow slow.

If we connect at the roots — making `p → a` — the structure stays flat. `q → p → a`. Much shorter. And `d` is untouched — it still points directly to `a`.

The rule is: **always merge at the roots**. This is what keeps the tree manageable.

---

## Two Optimizations

### Union by Rank

When merging two trees, attach the root of the *shorter* tree under the root of the *taller* tree. This keeps the overall tree height logarithmic.

```
if rank[rx] < rank[ry]:
    parent[rx] = ry
elif rank[rx] > rank[ry]:
    parent[ry] = rx
else:
    parent[ry] = rx
    rank[rx]++
```

### Path Compression

When executing `Find(x)`, every node visited on the way to the root is a potential shortcut. After finding the root, update each visited node to point directly to the root.

```
function Find(x):
    if parent[x] ≠ x:
        parent[x] = Find(parent[x])    # compress on the way back up
    return parent[x]
```

The first time you ask for `Find(q)` and walk `q → p → a`, path compression rewires `q` directly to `a`. The next call is $O(1)$.

Together, Union by Rank and Path Compression reduce DSU operations to nearly $O(1)$ amortized — formally $O(\alpha(N))$ where $\alpha$ is the inverse Ackermann function, which is effectively constant for any realistic input size.

---

## The Two Types of DSU Problems

**Student:** What does a DSU problem actually look like in practice?

**Teacher:** Every DSU problem is one of two types:

### Type 1 — Query: Are These in the Same Class?

You're given a stream of connections and a stream of queries. For each query, answer: are nodes $A$ and $B$ connected?

Algorithm: process all connections with `Union`. For each query, check `Find(A) == Find(B)`.

*Example:* Given a network of computers and connections, determine if two specific computers can communicate.

### Type 2 — Grouping: Reconstruct the Classes

You need to return the actual groups, not just answer yes/no queries.

Algorithm: process all connections with `Union`. Then iterate over all nodes and group them by their representative: build a map `Find(node) → [list of nodes in this class]`.

*Example:* Accounts Merge — group email addresses that transitively share an account. After running `Union` on all shared emails, collect all emails with the same root into one merged account.

---

## Tracing Accounts Merge

**Teacher:** Let's trace the key step. We have:

- Account 1: `[john@gmail, john_work@gmail]`  
- Account 2: `[john_work@gmail, john_spam@gmail]`

Initial state: each email is its own root.

After processing Account 1:  
`Union(john@gmail, john_work@gmail)` → `john_work → john@gmail`

After processing Account 2:  
`Union(john_work@gmail, john_spam@gmail)`

- `Find(john_work@gmail)` walks `john_work → john@gmail`. Root is `john@gmail`.
- `Find(john_spam@gmail)` returns `john_spam`. Root is itself.
- Connect roots: `john_spam → john@gmail`.

Final tree: `john_work → john@gmail ← john_spam`

All three have the same root. They're in the same equivalence class. When we build the output map, all three land under `john@gmail` and get merged into one account.

---

## The Complete Algorithm (Pseudocode)

```
# Build DSU
for each account:
    for each email in account (after the first):
        Union(account[0], email)

# Group by representative
groups = {}
for each email we've seen:
    root = Find(email)
    groups[root].append(email)

# Format output
result = []
for root, emails in groups:
    result.append([owner_of[root]] + sorted(emails))
```

---

## On Paper First

Before writing code for any equivalence class problem, write this down:

1. **State the relation:** "Two elements are in the same class if [condition]. This relation is reflexive, symmetric, and transitive."
2. **State the operation:** "For each [connection/overlap], call Union. For each [query/output], call Find."
3. **Identify the type:** "Am I answering queries (Type 1) or reconstructing groups (Type 2)?"

---

## Summary

| Property | Value |
|---|---|
| Mapping type | Partition into equivalence classes |
| Data structure | DSU / Union-Find (parent array) |
| Core operations | `Union(x, y)`, `Find(x)` |
| With optimizations | $O(\alpha(N)) \approx O(1)$ amortized per operation |
| Problem types | Query (same class?) or Grouping (reconstruct classes) |

> **The structural insight:** DSU doesn't move data. It never creates new arrays or scans through set members. It only maintains a parent pointer per node and connects roots. The equivalence classes emerge from the structure of those pointers, and Path Compression keeps the cost of reading them negligible.

---

*← Module 1c | Module 1e →*
