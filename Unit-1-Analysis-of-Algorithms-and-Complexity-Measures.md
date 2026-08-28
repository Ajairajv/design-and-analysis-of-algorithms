# Unit 1 — Analysis of Algorithms and Complexity Measures

**Course:** CSE 408 — Design and Analysis of Algorithms
**Textbook:** *Introduction to the Design and Analysis of Algorithms* — Anany Levitin
**Reference:** *Introduction to Algorithms* — Cormen, Leiserson, Rivest, Stein (CLRS)

---

## Syllabus covered in this file

> **Analysis of Algorithms and Complexity Measures:** Time and Space Complexity, Complexity Analysis of Insertion Sort and Merge Sort, Analysis of Iterative Algorithms, Analysis of Recursive Algorithms: Substitution Method, Recursion Tree Method and Master Method. **Divide-and-Conquer Technique:** Strassen's Matrix Multiplication. **Order Statistics:** Quick select, k-th Smallest and k-th Largest Element.

---

## Table of Contents

1. [What is an Algorithm?](#1-what-is-an-algorithm)
2. [Why Analyse Algorithms at All?](#2-why-analyse-algorithms-at-all)
3. [Time Complexity and Space Complexity](#3-time-complexity-and-space-complexity)
4. [Best, Worst and Average Case](#4-best-worst-and-average-case)
5. [Asymptotic Notations](#5-asymptotic-notations)
6. [Growth Rates and the Rules of Simplification](#6-growth-rates-and-the-rules-of-simplification)
7. [Analysis of Iterative Algorithms](#7-analysis-of-iterative-algorithms)
8. [Insertion Sort — Complete Analysis](#8-insertion-sort--complete-analysis)
9. [Merge Sort — Complete Analysis](#9-merge-sort--complete-analysis)
10. [Analysis of Recursive Algorithms — Writing the Recurrence](#10-analysis-of-recursive-algorithms--writing-the-recurrence)
11. [Substitution Method](#11-substitution-method)
12. [Recursion Tree Method](#12-recursion-tree-method)
13. [Master Method (Master Theorem)](#13-master-method-master-theorem)
14. [Divide-and-Conquer Technique](#14-divide-and-conquer-technique)
15. [Strassen's Matrix Multiplication](#15-strassens-matrix-multiplication)
16. [Order Statistics — Quick Select, k-th Smallest, k-th Largest](#16-order-statistics--quick-select-k-th-smallest-k-th-largest)
17. [Practice Questions with Full Solutions](#17-practice-questions-with-full-solutions)
18. [One-Page Revision Sheet](#18-one-page-revision-sheet)

---

# 1. What is an Algorithm?

### Definition

> An **algorithm** is a finite sequence of unambiguous instructions that takes a set of values as **input** and produces a set of values as **output**, solving a well-specified computational problem in a finite amount of time.

Break that definition into five properties. Examiners ask this as a 2-mark or 5-mark question, so learn the names.

| Property | Meaning | Example of a violation |
|---|---|---|
| **Input** | Zero or more externally supplied quantities | — |
| **Output** | At least one quantity is produced, and it is *correct* for every legal input | A "sort" that only works on positive numbers |
| **Definiteness** | Every instruction is clear and unambiguous | "Add a suitable number to x" — which number? |
| **Finiteness** | Terminates after a finite number of steps for every input | `while (x != 0) x = x - 2;` with `x = 5` never ends |
| **Effectiveness** | Every operation is basic enough to be carried out exactly, in principle with pencil and paper | "Guess the answer" is not effective |

### Algorithm vs Program vs Pseudocode

- **Algorithm** — language-independent logic, written in English + pseudocode.
- **Program** — an algorithm expressed in a programming language so a machine can run it.
- **Pseudocode** — a half-way notation using `for`, `while`, `if`, assignment, but no declarations or I/O plumbing.

**Example — the same algorithm in three forms.**

*English:* "Look at every element one by one; if it equals the key, report its position; if you reach the end, report failure."

*Pseudocode:*

```
LINEAR-SEARCH(A[0..n-1], key)
1  for i <- 0 to n-1
2      if A[i] = key
3          return i
4  return -1
```

*C++:*

```cpp
int linearSearch(const vector<int>& A, int key) {
    for (int i = 0; i < (int)A.size(); ++i)
        if (A[i] == key) return i;
    return -1;
}
```

All three are the *same* algorithm. Analysis is done on the algorithm, never on one particular program.

### The three questions we always ask

1. **Is it correct?** Does it give the right answer on *every* valid input, not just the ones you tried?
2. **How fast is it?** -> **time complexity**
3. **How much memory does it use?** -> **space complexity**

Unit 1 is almost entirely about questions 2 and 3.

---

# 2. Why Analyse Algorithms at All?

A natural objection: *"Why not just run the program and time it with a stopwatch?"*

Because measured time depends on things that have nothing to do with the algorithm:

- the **machine** (gaming laptop vs old lab PC),
- the **compiler** and its optimisation flags,
- the **language** (C++ vs Python is often a 50x difference),
- what **else** is running at that moment,
- the **particular input** you happened to test with.

Analysis strips all of that away and asks a machine-independent question:

> **How does the running time grow as the input size grows?**

That growth rate is the fingerprint of the algorithm. It survives changes of machine, language and compiler.

### The number that actually matters

Two algorithms for the same problem:

- Algorithm A takes `0.0001 * n^2` seconds
- Algorithm B takes `10 * n * log2(n)` seconds

B looks terrible — its constant is 100 000 times larger. Let us tabulate.

| n | A: 0.0001 n² | B: 10 n log₂ n | Winner |
|---:|---:|---:|:--|
| 100 | 1 s | 6 645 s ≈ 1.8 hours | **A** |
| 10 000 | 10 000 s ≈ 2.8 h | 1 328 771 s ≈ 15 days | **A** |
| 10⁶ | 10⁸ s ≈ **3.2 years** | 1.99 × 10⁸ s ≈ 6.3 years | A (barely) |
| 10⁸ | 10¹² s ≈ **31 700 years** | 2.66 × 10¹⁰ s ≈ 843 years | **B** |
| 10¹⁰ | 10¹⁶ s ≈ 317 **million** years | 3.3 × 10¹² s ≈ 105 000 years | **B** |

Constants dominate for small `n`, but the **growth rate always wins in the end**. n² eventually crushes n log n no matter how favourable the constants are. That is the entire justification for asymptotic analysis.

### The table you should memorise

Assume the machine performs 10⁹ useful operations per second.

| n | log₂ n | n | n log₂ n | n² | n³ | 2ⁿ |
|---:|---:|---:|---:|---:|---:|---:|
| 10 | ~0 | ~0 | ~0 | ~0 | 1 μs | 1 μs |
| 100 | ~0 | ~0 | ~0 | 10 μs | 1 ms | 4 × 10¹³ years |
| 1 000 | ~0 | 1 μs | 10 μs | 1 ms | 1 s | — |
| 10⁵ | ~0 | 0.1 ms | 1.7 ms | 10 s | 31 years | — |
| 10⁶ | ~0 | 1 ms | 20 ms | 17 min | 31 700 years | — |
| 10⁸ | ~0 | 0.1 s | 2.7 s | 115 days | — | — |

**Practical rule for coding tests:** with a 1-second limit you can afford roughly **10⁸ simple operations**.

| Constraint on n | Complexity you must aim for |
|---|---|
| n ≤ 12 | O(n!) is fine |
| n ≤ 25 | O(2ⁿ) |
| n ≤ 500 | O(n³) |
| n ≤ 5 000 | O(n²) |
| n ≤ 10⁶ | O(n log n) |
| n ≤ 10⁸ | O(n) |
| n astronomically large | O(log n) or O(1) |

---

# 3. Time Complexity and Space Complexity

## 3.1 The model of computation — the RAM model

Before we can count anything, we must agree on what "one step" means. We use the **RAM (Random Access Machine)** model:

1. Instructions execute **one at a time** — no parallelism.
2. Each **primitive operation** takes **constant** time:
   - arithmetic `+ - * / %`
   - comparison `< > == !=`
   - assignment `x = y`
   - array indexing `A[i]` (constant, because memory is *random access*)
   - control: function call/return, jump
3. Each memory word holds an integer of about `log n` bits — big enough to index the input, so arithmetic on indices is O(1).

**What the model deliberately ignores:** caches, memory latency, pipelining, disk I/O. It is a simplification, but a stunningly effective one — asymptotic predictions from the RAM model match reality remarkably well.

## 3.2 Input size — `n`

"Input size" must be defined per problem:

| Problem | Natural size measure |
|---|---|
| Sorting / searching an array | number of elements `n` |
| Matrix operations | dimension `n` of an n×n matrix (the input holds n² numbers) |
| Graph algorithms | two parameters: vertices `V` and edges `E` |
| String matching | text length `n`, pattern length `m` |
| Primality test of integer N | **number of bits** `b = log₂ N`, not N itself |
| Polynomial evaluation | degree of the polynomial |

> ⚠️ **Classic trap.** Testing whether `N` is prime by trial division up to `√N` costs O(√N) operations. That is *not* polynomial in the input size, because the input is only `b = log₂ N` bits long and `√N = 2^(b/2)` — exponential in `b`. This distinction is why primality testing was historically hard.

## 3.3 Time complexity

> **Time complexity** `T(n)` is the number of primitive operations executed by an algorithm, expressed as a function of the input size `n`.

We never measure it in seconds. We count **basic operations**.

### The basic operation

The **basic operation** is the one that contributes most to total running time — usually the one in the innermost loop.

| Algorithm | Basic operation |
|---|---|
| Sorting | key **comparison** |
| Searching | key **comparison** |
| Matrix multiplication | **multiplication** of two numbers |
| Graph traversal | visiting an **edge** |
| Polynomial evaluation | **multiplication** |

If the basic operation runs `C(n)` times and costs `c_op`, then

```
T(n) ≈ c_op × C(n)
```

`c_op` is a machine-dependent constant, so we study only `C(n)`. **This is why constants get thrown away.**

### Two ways to count — both examinable

**Method A — line-by-line / tabular (CLRS style).** Give each line a cost and a count, multiply, add.

**Method B — summation (Levitin style).** Write the number of basic-operation executions as a sum, then evaluate it in closed form.

Both appear with worked examples in Section 7.

## 3.4 Space complexity

> **Space complexity** `S(n)` is the total memory an algorithm needs, as a function of input size `n`.

It splits into two parts:

```
S(n) = Input space (fixed part) + Auxiliary space (variable part)
```

- **Input space / fixed part** — memory for the input plus constants and simple variables.
- **Auxiliary space / variable part** — the *extra* working memory: temporary arrays, dynamic structures, and **the recursion call stack**.

When someone asks "what is the space complexity of merge sort?", they almost always mean **auxiliary space**.

> 🔑 **Never forget the recursion stack.** A recursive function `d` levels deep uses Θ(d) stack space even if it allocates no arrays. Quick Sort's worst-case space is O(n) purely because of stack depth.

### Space of common algorithms

| Algorithm | Auxiliary space | Reason |
|---|---|---|
| Linear search | **O(1)** | just a loop counter |
| Binary search (iterative) | **O(1)** | `low`, `high`, `mid` |
| Binary search (recursive) | **O(log n)** | stack depth = number of halvings |
| Insertion sort | **O(1)** | one `key` variable — sorts *in place* |
| Bubble / selection sort | **O(1)** | in place |
| Merge sort (array) | **O(n)** | temporary merge buffer (plus O(log n) stack, absorbed) |
| Quick sort | **O(log n)** avg, **O(n)** worst | recursion stack only; partition is in place |
| Counting sort | **O(k)** | the count array of size k |
| Naive recursive Fibonacci | **O(n)** | deepest chain of calls is n |

### The time–space trade-off

You can often buy speed with memory, or memory with speed.

**Example 1 — Fibonacci.**
- Plain recursion: **O(2ⁿ)** time, O(n) space.
- Memoised recursion: **O(n)** time, O(n) space — we spent an array to kill exponential time.
- Iterative with two variables: **O(n)** time, **O(1)** space.

**Example 2 — detecting duplicates.**
- Nested loops: O(n²) time, O(1) space.
- Sort then scan: O(n log n) time, O(1) extra space.
- Hash set: **O(n) time**, but **O(n) space**.

No universally right answer — it depends on which resource is scarce.

## 3.5 Full worked example — time *and* space together

```cpp
int sumOfSquares(int n) {         // cost   times
    int total = 0;                //  c1      1
    for (int i = 1; i <= n; ++i)  //  c2      n+1   (the test runs n+1 times)
        total += i * i;           //  c3      n
    return total;                 //  c4      1
}
```

**Time:**

```
T(n) = c1·1 + c2·(n+1) + c3·n + c4·1
     = (c2 + c3)·n + (c1 + c2 + c4)
     = a·n + b            for constants a, b
     = Θ(n)
```

**Space:** variables `total`, `i`, `n` -> 3 words -> **Θ(1)** auxiliary space.

Notice how `c1..c4` collapsed into `a·n + b`. This always happens, which is exactly why we jump straight to asymptotic notation.

---

# 4. Best, Worst and Average Case

For many algorithms the running time depends not only on the *size* of the input but on the *particular* input of that size.

Linear search on `A = [7, 3, 9, 4, 1]`, n = 5:

- searching for `7` -> 1 comparison
- searching for `1` -> 5 comparisons
- searching for `100` -> 5 comparisons, then failure

So `T(5)` is not a single number. Hence three separate analyses.

## 4.1 Definitions

| Case | Definition | Usually written with |
|---|---|---|
| **Worst case** `W(n)` | Maximum number of basic operations over **all** inputs of size n | Big-O / Θ |
| **Best case** `B(n)` | Minimum number of basic operations over all inputs of size n | Ω / Θ |
| **Average case** `A(n)` | Expected number, over a stated **probability distribution** of inputs | Θ |

## 4.2 Worked example — average case of linear search

**Assumptions** (an average-case analysis is meaningless without stated assumptions):

1. The probability that the key **is present** is `p`, where 0 ≤ p ≤ 1.
2. If present, it is **equally likely** to be at any of the n positions.

If the key is found at position `i` (1-indexed) we made `i` comparisons, with probability `p/n`.
If the key is absent we made `n` comparisons, with probability `1 − p`.

```
A(n) = Σ(i=1..n) i · (p/n)  +  n · (1 − p)
     = (p/n) · [ n(n+1)/2 ] +  n(1 − p)
     = p(n+1)/2 + n(1 − p)
```

**Sanity checks:**

- `p = 1` (always present): `A(n) = (n+1)/2` — on average we scan half the array. ✔
- `p = 0` (never present): `A(n) = n` — we always scan everything. ✔
- `p = 0.5`: `A(n) = (n+1)/4 + n/2 ≈ 3n/4`.

In every case `A(n) = Θ(n)`.

## 4.3 Which case should we report?

- **Worst case is the default.** It is a *guarantee*: the algorithm will never be slower. Critical for real-time systems, medical devices, air-traffic control.
- **Average case** is more realistic but needs a probability model that may not match reality.
- **Best case** is nearly useless alone — any algorithm can get lucky — but it *is* informative when it differs sharply from the worst case. Insertion sort's O(n) best case is exactly why it is used to finish off nearly-sorted data.

> ⚠️ **Extremely common misconception:** "Big-O means worst case and Ω means best case." **FALSE.** Case analysis and asymptotic notation are two independent axes. You can say "the *best* case of insertion sort is Θ(n)". O/Ω/Θ bound *a function*; best/worst/average choose *which function* you are bounding. Any of the three cases can be bounded by any of the notations.

## 4.4 Case table for the algorithms you must know

| Algorithm | Best | Average | Worst | Aux. space |
|---|---|---|---|---|
| Linear search | Θ(1) | Θ(n) | Θ(n) | Θ(1) |
| Binary search | Θ(1) | Θ(log n) | Θ(log n) | Θ(1) iterative |
| Insertion sort | **Θ(n)** | Θ(n²) | Θ(n²) | Θ(1) |
| Selection sort | Θ(n²) | Θ(n²) | Θ(n²) | Θ(1) |
| Bubble sort (optimised) | **Θ(n)** | Θ(n²) | Θ(n²) | Θ(1) |
| **Merge sort** | Θ(n log n) | Θ(n log n) | **Θ(n log n)** | Θ(n) |
| Quick sort | Θ(n log n) | Θ(n log n) | **Θ(n²)** | Θ(log n) avg |
| Heap sort | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(1) |
| **Quick select** | Θ(n) | **Θ(n)** | **Θ(n²)** | Θ(1) iterative |

---

# 5. Asymptotic Notations

Asymptotic notation is a precise language for saying "this function grows about this fast, ignoring constants and small inputs".

Everything below is about **sets of functions**. When we write `f(n) = O(g(n))` we really mean `f(n) ∈ O(g(n))` — the equals sign is a historical abuse of notation.

## 5.1 Big-O — asymptotic **upper** bound ("at most", ≤)

> **Definition.** `f(n) = O(g(n))` if there exist **positive constants** `c` and `n₀` such that
> `0 ≤ f(n) ≤ c · g(n)` **for all n ≥ n₀**.

In words: beyond some point `n₀`, `f` never rises above a constant multiple of `g`.

**Picture:** `c·g(n)` is a ceiling that `f(n)` slips under from `n₀` onwards and never escapes.

### Worked proof 1 — `3n + 2 = O(n)`

We must produce `c` and `n₀`.

```
3n + 2 ≤ 3n + 2n = 5n     whenever 2 ≤ 2n, i.e. n ≥ 1
```

So with **c = 5, n₀ = 1** we have `3n + 2 ≤ 5n` for all n ≥ 1. ∎

Alternatively **c = 4, n₀ = 2**: `3n + 2 ≤ 4n ⟺ 2 ≤ n`. Also valid — **the constants are not unique**, you only need to exhibit *one* pair.

### Worked proof 2 — `2n² + 3n + 1 = O(n²)`

For n ≥ 1 we have `n ≤ n²` and `1 ≤ n²`, so

```
2n² + 3n + 1 ≤ 2n² + 3n² + 1n² = 6n²
```

Take **c = 6, n₀ = 1**. ∎

### Worked proof 3 — `n² ≠ O(n)` (a disproof)

Suppose `n² ≤ c·n` for all n ≥ n₀. Divide by n (positive): `n ≤ c` for all n ≥ n₀. But `n` is unbounded, so choosing `n = max(n₀, c) + 1` breaks the inequality. Contradiction — no such `c` exists. ∎

**This is the standard technique for disproofs: assume the bound, derive "n ≤ constant", contradiction.**

### More examples

| Statement | True? | Why |
|---|---|---|
| `5n + 3 = O(n)` | ✔ | c = 8, n₀ = 1 |
| `5n + 3 = O(n²)` | ✔ | O is an *upper* bound; loose bounds are legal |
| `5n + 3 = O(n³)` | ✔ | even looser, still true |
| `n² = O(n log n)` | ✘ | n²/(n log n) = n/log n -> ∞ |
| `1000000 = O(1)` | ✔ | any constant is O(1) |
| `log₂ n = O(log₁₀ n)` | ✔ | bases differ by a constant factor only |
| `2^(n+1) = O(2ⁿ)` | ✔ | 2^(n+1) = 2·2ⁿ, so c = 2 |
| `2^(2n) = O(2ⁿ)` | ✘ | 2^(2n) = (2ⁿ)², ratio 2ⁿ -> ∞ |

> ⚠️ **`2^(n+1) = O(2ⁿ)` but `2^(2n) ≠ O(2ⁿ)`.** Adding to the exponent is a constant factor; multiplying the exponent is not. This is a favourite MCQ.

## 5.2 Big-Omega (Ω) — asymptotic **lower** bound ("at least", ≥)

> **Definition.** `f(n) = Ω(g(n))` if there exist positive constants `c` and `n₀` such that
> `0 ≤ c · g(n) ≤ f(n)` for all n ≥ n₀.

### Worked proof — `3n + 2 = Ω(n)`

`3n + 2 ≥ 3n` for all n ≥ 1, so **c = 3, n₀ = 1**. ∎

### Worked proof — `n²/2 − 3n = Ω(n²)`

We want `c·n² ≤ n²/2 − 3n`. Try `c = 1/4`:

```
n²/4 ≤ n²/2 − 3n
⟺ 3n ≤ n²/2 − n²/4 = n²/4
⟺ 12 ≤ n
```

So **c = 1/4, n₀ = 12**. ∎

**Interpretation:** Ω gives a performance *guarantee from below* — the algorithm will take **at least** this long. Saying "sorting by comparison is Ω(n log n)" means *no* comparison sort can beat n log n.

## 5.3 Theta (Θ) — **tight** bound ("exactly", =)

> **Definition.** `f(n) = Θ(g(n))` if there exist positive constants `c₁`, `c₂`, `n₀` such that
> `0 ≤ c₁·g(n) ≤ f(n) ≤ c₂·g(n)` for all n ≥ n₀.

### The theorem you should quote

> **`f(n) = Θ(g(n))` if and only if `f(n) = O(g(n))` AND `f(n) = Ω(g(n))`.**

This is how Θ proofs are done: prove the upper bound, prove the lower bound, conclude.

### Worked proof — `(1/2)n² − 3n = Θ(n²)`

*Upper (O):* `(1/2)n² − 3n ≤ (1/2)n²` for all n ≥ 1 -> `c₂ = 1/2`.
*Lower (Ω):* shown above -> `c₁ = 1/4`, n₀ = 12.

Together: `(1/4)n² ≤ (1/2)n² − 3n ≤ (1/2)n²` for all n ≥ 12. Hence Θ(n²) with c₁ = 1/4, c₂ = 1/2, n₀ = 12. ∎

### Worked proof — `6n³ ≠ Θ(n²)`

Assume `6n³ ≤ c₂n²`. Then `n ≤ c₂/6`, false for large n. So the O part fails, hence not Θ. ∎

## 5.4 little-o (o) — **strict** upper bound ("strictly less than", <)

> **Definition.** `f(n) = o(g(n))` if for **every** positive constant `c` there exists `n₀` such that
> `0 ≤ f(n) < c·g(n)` for all n ≥ n₀.

Equivalently, and far easier to use:

```
f(n) = o(g(n))   ⟺   lim (n→∞) f(n)/g(n) = 0
```

The difference from Big-O: in O, the bound must hold for *some* c; in little-o, it must hold for *every* c, however tiny. That forces `f` to become **negligible** compared to `g`.

| Statement | True? | Limit test |
|---|---|---|
| `2n = o(n²)` | ✔ | 2n/n² = 2/n -> 0 |
| `2n² = o(n²)` | ✘ | 2n²/n² = 2 -> 2, not 0 |
| `n log n = o(n²)` | ✔ | log n / n -> 0 |
| `n = o(n log n)` | ✔ | 1/log n -> 0 |
| `100n = o(n)` | ✘ | ratio -> 100 |

## 5.5 little-omega (ω) — **strict** lower bound ("strictly greater than", >)

> **Definition.** `f(n) = ω(g(n))` if for every positive constant `c` there exists `n₀` with `0 ≤ c·g(n) < f(n)` for all n ≥ n₀.

```
f(n) = ω(g(n))   ⟺   lim (n→∞) f(n)/g(n) = ∞   ⟺   g(n) = o(f(n))
```

Examples: `n² = ω(n)` ✔, `n² = ω(n²)` ✘, `2ⁿ = ω(n^100)` ✔.

## 5.6 The analogy that makes all five click

| Notation | Number analogy | Meaning |
|---|---|---|
| `f = O(g)` | `a ≤ b` | f grows no faster than g |
| `f = Ω(g)` | `a ≥ b` | f grows no slower than g |
| `f = Θ(g)` | `a = b` | same growth rate |
| `f = o(g)` | `a < b` | f grows strictly slower |
| `f = ω(g)` | `a > b` | f grows strictly faster |

The analogy is imperfect in one important way: for real numbers, **any** two are comparable, but two functions need not be. `f(n) = n` and `g(n) = n^(1+sin n)` satisfy none of the five relations, because `g` oscillates between `n⁰` and `n²` forever.

## 5.7 The limit method — your fastest tool

To compare `f` and `g`, compute `L = lim(n→∞) f(n)/g(n)`:

| L | Conclusion |
|---|---|
| `L = 0` | `f = o(g)`, hence `f = O(g)`, and **not** Θ |
| `0 < L < ∞` | `f = Θ(g)` (and both O and Ω) |
| `L = ∞` | `f = ω(g)`, hence `f = Ω(g)`, and **not** O |

Use L'Hôpital's rule when you hit ∞/∞.

### Worked example — is `log n = O(√n)`?

```
lim (log n)/(√n)   →  ∞/∞, apply L'Hôpital
= lim (1/n) / (1/(2√n))
= lim 2√n / n
= lim 2/√n
= 0
```

L = 0, so `log n = o(√n)`, and therefore certainly `log n = O(√n)`. Logarithms lose to *every* positive power of n.

### Worked example — compare `n!` and `2ⁿ`

```
n!/2ⁿ = (1·2·3·…·n)/(2·2·2·…·2) = (1/2)·(2/2)·(3/2)·(4/2)·…·(n/2)
```

From the third factor onward every term is ≥ 3/2, so the product diverges. `n! = ω(2ⁿ)` — factorial beats exponential.

## 5.8 Properties of asymptotic notation (frequently asked)

| Property | O | Ω | Θ | o | ω |
|---|---|---|---|---|---|
| **Reflexive** `f = X(f)` | ✔ | ✔ | ✔ | ✘ | ✘ |
| **Symmetric** `f=X(g) ⟹ g=X(f)` | ✘ | ✘ | ✔ | ✘ | ✘ |
| **Transitive** `f=X(g), g=X(h) ⟹ f=X(h)` | ✔ | ✔ | ✔ | ✔ | ✔ |
| **Transpose symmetry** | `f=O(g) ⟺ g=Ω(f)` | | | `f=o(g) ⟺ g=ω(f)` | |

**Other useful rules:**

1. **Sum rule:** `O(f) + O(g) = O(max(f, g))`. Sequential blocks -> keep the bigger one.
   *Example:* a Θ(n log n) sort followed by a Θ(n) scan is Θ(n log n).
2. **Product rule:** `O(f) × O(g) = O(f·g)`. Nested loops -> multiply.
   *Example:* an O(n) loop containing an O(log n) binary search is O(n log n).
3. **Constant rule:** `O(c·f) = O(f)` for any constant c > 0.
4. **Polynomial rule:** if `f(n) = a_k n^k + … + a_1 n + a_0` with `a_k > 0`, then `f(n) = Θ(n^k)`.
   Only the highest-degree term matters, and its coefficient does not.
5. `max(f(n), g(n)) = Θ(f(n) + g(n))`.

### Proof of the polynomial rule (short version)

Let `f(n) = a_k n^k + … + a_0`, `a_k > 0`. Divide by `n^k`:

```
f(n)/n^k = a_k + a_{k-1}/n + … + a_0/n^k  →  a_k   as n → ∞
```

The limit is `a_k`, a finite non-zero constant, so by the limit method `f(n) = Θ(n^k)`. ∎

---

# 6. Growth Rates and the Rules of Simplification

## 6.1 The standard hierarchy (memorise the order)

From slowest-growing (best) to fastest-growing (worst):

```
1  <  log log n  <  log n  <  √n  <  n  <  n log n  <  n²  <  n³  <  2ⁿ  <  3ⁿ  <  n!  <  nⁿ
```

| Class | Name | Typical algorithm |
|---|---|---|
| **O(1)** | constant | array access, hash lookup, `push`/`pop` |
| **O(log log n)** | double logarithmic | interpolation search on uniform data |
| **O(log n)** | logarithmic | binary search, balanced-BST operations, `gcd` |
| **O(√n)** | root | trial-division primality, some number theory |
| **O(n)** | linear | linear search, one pass over an array, **quickselect (avg)** |
| **O(n log n)** | linearithmic | **merge sort**, heap sort, FFT, building a suffix array |
| **O(n²)** | quadratic | **insertion sort**, bubble sort, simple all-pairs comparisons |
| **O(n³)** | cubic | naive matrix multiply, Floyd–Warshall |
| **O(n^2.81)** | — | **Strassen's matrix multiplication** |
| **O(2ⁿ)** | exponential | subsets, naive TSP by brute force |
| **O(n!)** | factorial | permutations, naive travelling salesman |

Everything up to `O(n^k)` is **polynomial** = "tractable". Anything from `2ⁿ` up is **exponential** = "intractable for large n".

## 6.2 Why log base does not matter

```
log_a n = log_b n / log_b a
```

`1/log_b a` is a constant, so `log₂ n`, `log₁₀ n` and `ln n` differ by constant factors only:

```
Θ(log₂ n) = Θ(log₁₀ n) = Θ(ln n) = Θ(log n)
```

We simply write `log n` and never mention the base — **inside asymptotic notation**. Outside it (e.g. "the tree has log₂ n levels") the base matters.

> ⚠️ But the base **does** matter in an exponent: `2ⁿ` and `3ⁿ` are *not* the same, because `3ⁿ/2ⁿ = (3/2)ⁿ -> ∞`.

## 6.3 Useful mathematical identities

You will need these constantly.

**Arithmetic series**

```
1 + 2 + 3 + … + n = n(n+1)/2 = Θ(n²)
```

**Sum of squares**

```
1² + 2² + … + n² = n(n+1)(2n+1)/6 = Θ(n³)
```

**Geometric series (r ≠ 1)**

```
1 + r + r² + … + r^n = (r^(n+1) − 1)/(r − 1)
```

- If `r < 1`: the sum converges to `1/(1−r)` = **Θ(1)** — dominated by the *first* term.
- If `r > 1`: the sum is **Θ(r^n)** — dominated by the *last* term.
- If `r = 1`: the sum is `n+1` = Θ(n).

**This single fact explains most of the recursion-tree analyses in Section 12.**

**Harmonic series**

```
H(n) = 1 + 1/2 + 1/3 + … + 1/n = ln n + γ ≈ Θ(log n),  γ ≈ 0.5772
```

**Logarithm rules**

```
log(ab) = log a + log b
log(a/b) = log a − log b
log(a^b) = b·log a
a^(log_b c) = c^(log_b a)          ← very handy in Master-Theorem work
log_b(a) = log(a)/log(b)
2^(log₂ n) = n
```

**Stirling's approximation**

```
n! ≈ √(2πn) · (n/e)^n     ⟹     log(n!) = Θ(n log n)
```

That last identity is why comparison-based sorting has an Ω(n log n) lower bound.

## 6.4 How to simplify an expression to its Θ class — recipe

1. Drop all **constant multipliers**: `7n² -> n²`.
2. Drop all **lower-order terms**: `n² + n + 5 -> n²`.
3. For **sums**, keep only the fastest-growing term.
4. For **products**, keep everything: `n · log n` stays `n log n`.

### Practice — simplify each

| Expression | Θ class | Reason |
|---|---|---|
| `5n³ + 200n² + 1000` | Θ(n³) | highest power |
| `n + n log n` | Θ(n log n) | n log n dominates n |
| `2ⁿ + n^100` | Θ(2ⁿ) | exponential beats every polynomial |
| `log(n²)` | Θ(log n) | `= 2 log n` |
| `log(n!)` | Θ(n log n) | Stirling |
| `n/1000 + 10⁶` | Θ(n) | constant factor and additive constant drop |
| `√n + log n` | Θ(√n) | any positive power of n beats log |
| `3^(log₂ n)` | Θ(n^1.585) | `= n^(log₂ 3)`, and log₂3 ≈ 1.585 |
| `n^(1/log n)` | Θ(1) | `= 2^(log n / log n) = 2` |
---

# 7. Analysis of Iterative Algorithms

## 7.1 The general procedure (Levitin's five steps)

1. Decide on the parameter `n` that measures **input size**.
2. Identify the **basic operation** (usually the innermost statement).
3. Check whether the count depends only on `n`, or also on the specific input. If it depends on the input, do **worst / best / average** separately.
4. Set up a **sum** expressing the number of times the basic operation executes.
5. **Evaluate the sum** in closed form, or at least establish its order of growth.

## 7.2 Rules of thumb you can apply instantly

| Loop shape | Iterations | Cost |
|---|---|---|
| `for (i=0; i<n; i++)` | n | Θ(n) |
| `for (i=0; i<n; i+=k)` (k constant) | n/k | Θ(n) |
| `for (i=1; i<=n; i*=2)` | log₂ n | Θ(log n) |
| `for (i=n; i>=1; i/=2)` | log₂ n | Θ(log n) |
| `for (i=2; i<n; i=i*i)` | log log n | Θ(log log n) |
| `for (i=0; i*i<n; i++)` | √n | Θ(√n) |
| two independent loops, one after the other | n + m | Θ(n + m) |
| two nested loops, both to n | n·n | Θ(n²) |
| inner loop bounded by outer index | n(n+1)/2 | Θ(n²) |

> **The key discipline: an increment (`i++`) gives a linear count; a multiplication (`i*=2`) gives a logarithmic count.**

## 7.3 Example 1 — Maximum element (input-independent count)

```cpp
int maxElement(int A[], int n) {
    int maxVal = A[0];               // 1 assignment
    for (int i = 1; i < n; ++i)      // loop runs for i = 1..n-1
        if (A[i] > maxVal)           // BASIC OPERATION: comparison
            maxVal = A[i];
    return maxVal;
}
```

**Step 1.** Input size = `n`.
**Step 2.** Basic operation = the comparison `A[i] > maxVal`.
**Step 3.** It executes once per iteration regardless of the data — **no case distinction needed**.
**Step 4.**

```
C(n) = Σ (i = 1 to n-1) 1
```

**Step 5.**

```
C(n) = (n − 1) − 1 + 1 = n − 1 = Θ(n)
```

**Space:** Θ(1).

> **Sum shortcut:** `Σ(i = a to b) 1 = b − a + 1`. Memorise it; half of iterative analysis is this one formula.

## 7.4 Example 2 — Element uniqueness (worst case, triangular sum)

```cpp
bool uniqueElements(int A[], int n) {
    for (int i = 0; i < n - 1; ++i)
        for (int j = i + 1; j < n; ++j)
            if (A[i] == A[j])        // BASIC OPERATION
                return false;
    return true;
}
```

**Worst case:** no duplicates at all (or the only duplicate is the last pair), so we never return early.

```
C_worst(n) = Σ(i=0 .. n-2) Σ(j=i+1 .. n-1) 1
           = Σ(i=0 .. n-2) [ (n-1) − (i+1) + 1 ]
           = Σ(i=0 .. n-2) (n − 1 − i)
```

Substitute `k = n − 1 − i`. As `i` runs 0..n−2, `k` runs n−1 down to 1:

```
           = Σ(k=1 .. n-1) k
           = (n−1)n/2
           = Θ(n²)
```

**Best case:** `A[0] == A[1]` -> 1 comparison -> Θ(1).

So: best Θ(1), worst Θ(n²), and the answer is "O(n²)" if you must give one bound.

## 7.5 Example 3 — Matrix multiplication (three nested loops)

```cpp
void matMul(int A[][N], int B[][N], int C[][N], int n) {
    for (int i = 0; i < n; ++i)
        for (int j = 0; j < n; ++j) {
            C[i][j] = 0;
            for (int k = 0; k < n; ++k)
                C[i][j] += A[i][k] * B[k][j];   // BASIC OPERATION: multiply
        }
}
```

```
M(n) = Σ(i=0..n-1) Σ(j=0..n-1) Σ(k=0..n-1) 1 = n · n · n = n³ = Θ(n³)
```

Additions: also n³. Total operations ≈ 2n³, still **Θ(n³)**.

**Space:** Θ(n²) for the output matrix, Θ(1) auxiliary beyond that.

> **This Θ(n³) is exactly what Strassen's algorithm attacks in Section 15.**

## 7.6 Example 4 — The logarithmic loop

```cpp
int countHalvings(int n) {
    int count = 0;
    for (int i = 1; i <= n; i *= 2)   // 1, 2, 4, 8, 16, ...
        count++;
    return count;
}
```

`i` takes values `2⁰, 2¹, 2², …, 2^k` and the loop stops when `2^k > n`:

```
2^k ≤ n  ⟹  k ≤ log₂ n  ⟹  number of iterations = ⌊log₂ n⌋ + 1 = Θ(log n)
```

**Check with n = 16:** i = 1, 2, 4, 8, 16 -> 5 iterations = ⌊log₂16⌋ + 1 = 4 + 1 = 5. ✔

**Variation — what if `i *= 3`?** Then it is `log₃ n`, still **Θ(log n)** — the base is a constant factor.

## 7.7 Example 5 — Nested loops where the inner depends on the outer

```cpp
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= i; ++j)
        cout << "*";                 // BASIC OPERATION
```

```
C(n) = Σ(i=1..n) Σ(j=1..i) 1 = Σ(i=1..n) i = n(n+1)/2 = Θ(n²)
```

**Trace for n = 4:** 1 + 2 + 3 + 4 = 10 stars = 4·5/2 ✔

> ⚠️ **Do not be fooled** into calling this Θ(n) just because the inner loop is "shorter". Half of n² is still Θ(n²).

## 7.8 Example 6 — Mixed linear and logarithmic

```cpp
for (int i = 1; i <= n; ++i)          // n times
    for (int j = 1; j <= n; j *= 2)   // log n times
        doWork();                     // BASIC OPERATION
```

The inner loop count does not depend on `i`, so simply multiply:

```
C(n) = n × ⌊log₂ n + 1⌋ = Θ(n log n)
```

**Now change the inner bound to `i`:**

```cpp
for (int i = 1; i <= n; ++i)
    for (int j = 1; j <= i; j *= 2)
        doWork();
```

```
C(n) = Σ(i=1..n) (⌊log₂ i⌋ + 1) ≈ log(n!) + n = Θ(n log n)
```

using `log(n!) = Θ(n log n)` from Stirling. Same class, but the reasoning is different — an examiner may ask for exactly this.

## 7.9 Example 7 — A loop that looks quadratic but is linear

```cpp
int i = 0, j = 0, count = 0;
while (i < n) {
    while (j < n && someCondition(j)) { j++; count++; }
    i++;
}
```

`j` is **never reset**. Across the whole run it can only advance from 0 to n, so the inner loop body executes at most `n` times *in total*. Total work = Θ(n), not Θ(n²).

> **This is amortised reasoning** — the same idea powers the two-pointer technique, the sliding window, and monotonic stacks (Unit 5).

## 7.10 Example 8 — Selection sort, counted exactly

```cpp
void selectionSort(int A[], int n) {
    for (int i = 0; i < n - 1; ++i) {
        int minIdx = i;
        for (int j = i + 1; j < n; ++j)
            if (A[j] < A[minIdx])          // BASIC OPERATION
                minIdx = j;
        swap(A[i], A[minIdx]);
    }
}
```

**Comparisons:**

```
C(n) = Σ(i=0..n-2) Σ(j=i+1..n-1) 1 = Σ(i=0..n-2)(n − 1 − i) = n(n−1)/2 = Θ(n²)
```

**Swaps:** exactly `n − 1` = Θ(n).

**Key property:** the comparison count is **the same for every input** — best = average = worst = Θ(n²). Selection sort cannot exploit sorted input, unlike insertion sort. Its one virtue is the minimal number of swaps, which matters when a swap is expensive (large records).

## 7.11 Example 9 — Binary search, iterative

```cpp
int binarySearch(int A[], int n, int key) {
    int low = 0, high = n - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;   // avoids integer overflow
        if (A[mid] == key) return mid;      // BASIC OPERATION (comparison)
        else if (A[mid] < key) low = mid + 1;
        else high = mid - 1;
    }
    return -1;
}
```

Each iteration halves the search interval: n -> n/2 -> n/4 -> … -> 1.

After `k` iterations the size is `n/2^k`. We stop when `n/2^k = 1`:

```
2^k = n  ⟹  k = log₂ n
```

**Worst case** `W(n) = ⌊log₂ n⌋ + 1 = Θ(log n)`.
**Best case** `B(n) = 1` (key sits at the middle) = Θ(1).
**Space** Θ(1) iterative, Θ(log n) recursive.

**Trace — search 23 in `[2, 5, 8, 12, 16, 23, 38, 56, 72, 91]`, n = 10:**

| Step | low | high | mid | A[mid] | Action |
|---|---|---|---|---|---|
| 1 | 0 | 9 | 4 | 16 | 16 < 23 -> low = 5 |
| 2 | 5 | 9 | 7 | 56 | 56 > 23 -> high = 6 |
| 3 | 5 | 6 | 5 | 23 | **found at index 5** |

3 comparisons instead of the 6 that linear search would need. For n = 10⁶, binary search needs 20 comparisons where linear search needs 500 000 on average.

## 7.12 Summary of the technique

```
1. Identify n
2. Identify the basic operation
3. Worst / best / average needed?
4. Write the summation
5. Close the sum, take the Θ class
```

---

# 8. Insertion Sort — Complete Analysis

## 8.1 The idea

Exactly how you sort a hand of playing cards. Keep the left part of the array sorted. Pick the next card, slide it leftwards past every larger card, and drop it into its slot. Repeat.

At the start of pass `i`, `A[0..i-1]` is already sorted. We insert `A[i]` into that sorted prefix.

## 8.2 The algorithm

**Pseudocode (CLRS, 1-indexed):**

```
INSERTION-SORT(A, n)
1  for j = 2 to n
2      key = A[j]
3      // insert A[j] into the sorted sequence A[1..j-1]
4      i = j - 1
5      while i > 0 and A[i] > key
6          A[i+1] = A[i]
7          i = i - 1
8      A[i+1] = key
```

**C++ (0-indexed, runnable):**

```cpp
void insertionSort(int A[], int n) {
    for (int j = 1; j < n; ++j) {
        int key = A[j];
        int i = j - 1;
        while (i >= 0 && A[i] > key) {   // BASIC OPERATION: comparison
            A[i + 1] = A[i];             // shift right
            --i;
        }
        A[i + 1] = key;                  // drop key into place
    }
}
```

## 8.3 Full trace on `A = [5, 2, 4, 6, 1, 3]`, n = 6

Sorted prefix shown in **bold**.

| Pass j | key | Array before the pass | Shifts | Array after the pass | Comparisons |
|---|---|---|---|---|---|
| j=1 | 2 | [**5**, 2, 4, 6, 1, 3] | 5 moves right | [**2, 5**, 4, 6, 1, 3] | 1 |
| j=2 | 4 | [**2, 5**, 4, 6, 1, 3] | 5 moves right | [**2, 4, 5**, 6, 1, 3] | 2 |
| j=3 | 6 | [**2, 4, 5**, 6, 1, 3] | none | [**2, 4, 5, 6**, 1, 3] | 1 |
| j=4 | 1 | [**2, 4, 5, 6**, 1, 3] | 6,5,4,2 all move | [**1, 2, 4, 5, 6**, 3] | 4 |
| j=5 | 3 | [**1, 2, 4, 5, 6**, 3] | 6,5,4 move | [**1, 2, 3, 4, 5, 6**] | 4 |

**Total comparisons = 1 + 2 + 1 + 4 + 4 = 12.**
Final array: `[1, 2, 3, 4, 5, 6]` ✔

### Step-by-step detail of pass j = 4 (key = 1)

Array is `[2, 4, 5, 6, | 1, 3]`, key = 1, i starts at 3.

```
i=3: A[3]=6 > 1  -> shift: [2,4,5,6,6,3]   i=2
i=2: A[2]=5 > 1  -> shift: [2,4,5,5,6,3]   i=1
i=1: A[1]=4 > 1  -> shift: [2,4,4,5,6,3]   i=0
i=0: A[0]=2 > 1  -> shift: [2,2,4,5,6,3]   i=-1
i=-1: loop ends (i < 0)
place key: A[0] = 1 -> [1,2,4,5,6,3]
```

4 comparisons, 4 shifts.

## 8.4 Loop invariant and correctness proof

> **Loop invariant.** At the start of each iteration of the `for` loop of line 1, the subarray `A[1..j-1]` consists of the elements originally in `A[1..j-1]`, but in sorted order.

Correctness needs three parts — **initialisation, maintenance, termination**. Learn this three-part shape; every loop-invariant proof in the course uses it.

**Initialisation.** Before the first iteration, `j = 2`, so `A[1..1]` is a single element — trivially sorted, and it is the original element. ✔

**Maintenance.** The `while` loop moves `A[j-1], A[j-2], …` one position right until it finds the correct slot for `key`, then inserts it. Every element moved is strictly greater than `key`, and every element left in place is ≤ `key`. So after the body, `A[1..j]` is sorted and holds exactly the original elements of `A[1..j]`. Incrementing `j` re-establishes the invariant for the next iteration. ✔

**Termination.** The loop ends when `j = n + 1`. Substituting into the invariant: `A[1..n]` is sorted and holds the original elements — i.e. the whole array is sorted. **That is exactly the specification, so the algorithm is correct.** ∎

## 8.5 Time complexity — line-by-line (the CLRS table)

Let `t_j` = the number of times the `while` test in line 5 executes for that value of `j`.

| Line | Cost | Times |
|---|---|---|
| 1 `for j = 2 to n` | c₁ | n |
| 2 `key = A[j]` | c₂ | n − 1 |
| 4 `i = j − 1` | c₄ | n − 1 |
| 5 `while i>0 and A[i]>key` | c₅ | Σ(j=2..n) t_j |
| 6 `A[i+1] = A[i]` | c₆ | Σ(j=2..n) (t_j − 1) |
| 7 `i = i − 1` | c₇ | Σ(j=2..n) (t_j − 1) |
| 8 `A[i+1] = key` | c₈ | n − 1 |

```
T(n) = c₁n + c₂(n−1) + c₄(n−1) + c₅ Σ t_j + c₆ Σ(t_j −1) + c₇ Σ(t_j −1) + c₈(n−1)
```

Everything now hinges on the values of `t_j`.

### Best case — array already sorted, e.g. `[1, 2, 3, 4, 5]`

`A[i] > key` is false immediately, so `t_j = 1` for every j. Then `Σ t_j = n − 1` and `Σ(t_j − 1) = 0`:

```
T(n) = c₁n + (c₂ + c₄ + c₅ + c₈)(n − 1)
     = a·n + b
     = Θ(n)                     ← LINEAR
```

**Comparisons: exactly n − 1. Shifts: 0.**

### Worst case — array in reverse order, e.g. `[5, 4, 3, 2, 1]`

Every `key` must travel all the way to the front, so `t_j = j` for every j.

```
Σ(j=2..n) j = [ n(n+1)/2 ] − 1
Σ(j=2..n) (j − 1) = n(n−1)/2
```

Substituting:

```
T(n) = c₁n + (c₂+c₄+c₈)(n−1) + c₅[n(n+1)/2 − 1] + (c₆+c₇)[n(n−1)/2]
     = a·n² + b·n + c
     = Θ(n²)                    ← QUADRATIC
```

**Comparisons: n(n−1)/2. Shifts: n(n−1)/2.**

*Check with n = 5, reverse sorted:* 5·4/2 = 10 comparisons.
Trace `[5,4,3,2,1]`: pass1 = 1, pass2 = 2, pass3 = 3, pass4 = 4 -> total 10 ✔

### Average case — random permutation

On average, `key` is compared with about half of the sorted prefix, so `t_j ≈ j/2`:

```
Σ(j=2..n) j/2 ≈ n²/4
T(n) ≈ a·n²/4 + … = Θ(n²)      ← still QUADRATIC
```

The average case is only about **twice as fast** as the worst case — the same order of growth. Being "usually lucky" does not save insertion sort.

## 8.6 Summary table

| Measure | Best (sorted) | Average (random) | Worst (reverse) |
|---|---|---|---|
| Comparisons | n − 1 | ≈ n²/4 | n(n−1)/2 |
| Shifts | 0 | ≈ n²/4 | n(n−1)/2 |
| Time | **Θ(n)** | **Θ(n²)** | **Θ(n²)** |
| Auxiliary space | Θ(1) | Θ(1) | Θ(1) |

## 8.7 Properties of insertion sort

| Property | Value | Why it matters |
|---|---|---|
| **In place** | ✔ Θ(1) extra memory | can sort a huge array without a second buffer |
| **Stable** | ✔ | equal keys keep their original relative order — because the test is `A[i] > key`, strictly greater, so we never jump over an equal element |
| **Adaptive** | ✔ | nearly-sorted input runs in nearly linear time |
| **Online** | ✔ | can sort a stream — insert each new element as it arrives, no need to know n in advance |

> ⚠️ **Stability depends on `>` vs `>=`.** If you write `while (i >= 0 && A[i] >= key)` the sort becomes **unstable** — it will shift past equal elements and reverse their order. Interview favourite.

## 8.8 Why insertion sort is genuinely used

Despite Θ(n²), real libraries use it:

1. **Small arrays.** For n ≤ ~16 its tiny constant factor beats merge sort and quick sort. Production sorts (e.g. `std::sort`'s introsort, Timsort in Python/Java) recurse down to a small threshold and then finish with insertion sort.
2. **Nearly sorted data.** If every element is at most `k` positions from its final spot, insertion sort runs in O(nk) — linear when k is a constant.
3. **Streaming input.** It is the natural online sort.
4. **Simplicity.** Few lines, no recursion, no extra memory, easy to verify.

## 8.9 Comparison with its quadratic cousins

| | Insertion sort | Selection sort | Bubble sort |
|---|---|---|---|
| Best case | **Θ(n)** | Θ(n²) | Θ(n) (with early-exit flag) |
| Worst case | Θ(n²) | Θ(n²) | Θ(n²) |
| Swaps/writes (worst) | Θ(n²) | **Θ(n)** | Θ(n²) |
| Stable | ✔ | ✘ (standard version) | ✔ |
| Adaptive | ✔ | ✘ | ✔ |
| Best used when | nearly sorted, small n | writes are expensive | teaching only |

---

# 9. Merge Sort — Complete Analysis

## 9.1 The idea — divide and conquer in its purest form

| Phase | What happens |
|---|---|
| **Divide** | Split the array of n elements into two halves of n/2 each |
| **Conquer** | Sort each half **recursively** (base case: one element is already sorted) |
| **Combine** | **Merge** the two sorted halves into one sorted array |

All the real work happens in the *combine* step — the opposite of quick sort, where the work is in the *divide*.

## 9.2 The algorithm

```
MERGE-SORT(A, p, r)
1  if p < r                       // more than one element
2      q = ⌊(p + r)/2⌋            // midpoint
3      MERGE-SORT(A, p, q)        // sort left half
4      MERGE-SORT(A, q+1, r)      // sort right half
5      MERGE(A, p, q, r)          // combine
```

```
MERGE(A, p, q, r)
 1  n1 = q − p + 1;  n2 = r − q
 2  create arrays L[1..n1+1], R[1..n2+1]
 3  copy A[p..q]   into L[1..n1]
 4  copy A[q+1..r] into R[1..n2]
 5  L[n1+1] = ∞;  R[n2+1] = ∞          // sentinels: avoid "ran out" checks
 6  i = 1; j = 1
 7  for k = p to r
 8      if L[i] ≤ R[j]                 // ≤ keeps the sort STABLE
 9          A[k] = L[i];  i = i + 1
10      else
11          A[k] = R[j];  j = j + 1
```

**Runnable C++:**

```cpp
void merge(vector<int>& A, int p, int q, int r) {
    vector<int> L(A.begin() + p, A.begin() + q + 1);   // left half
    vector<int> R(A.begin() + q + 1, A.begin() + r + 1); // right half
    int i = 0, j = 0, k = p;
    while (i < (int)L.size() && j < (int)R.size())
        A[k++] = (L[i] <= R[j]) ? L[i++] : R[j++];     // <= gives stability
    while (i < (int)L.size()) A[k++] = L[i++];         // drain leftovers
    while (j < (int)R.size()) A[k++] = R[j++];
}

void mergeSort(vector<int>& A, int p, int r) {
    if (p >= r) return;                // base case: 0 or 1 element
    int q = p + (r - p) / 2;
    mergeSort(A, p, q);
    mergeSort(A, q + 1, r);
    merge(A, p, q, r);
}
```

## 9.3 Trace of MERGE on two sorted halves

Merge `L = [2, 4, 5, 7]` and `R = [1, 2, 3, 6]` into `A[0..7]`.

| k | L[i] | R[j] | Comparison | Take | A so far |
|---|---|---|---|---|---|
| 0 | 2 | 1 | 2 ≤ 1? no | R -> 1 | [1] |
| 1 | 2 | 2 | 2 ≤ 2? **yes** | L -> 2 | [1,2] |
| 2 | 4 | 2 | 4 ≤ 2? no | R -> 2 | [1,2,2] |
| 3 | 4 | 3 | 4 ≤ 3? no | R -> 3 | [1,2,2,3] |
| 4 | 4 | 6 | 4 ≤ 6? yes | L -> 4 | [1,2,2,3,4] |
| 5 | 5 | 6 | 5 ≤ 6? yes | L -> 5 | [1,2,2,3,4,5] |
| 6 | 7 | 6 | 7 ≤ 6? no | R -> 6 | [1,2,2,3,4,5,6] |
| 7 | 7 | — | R exhausted | L -> 7 | [1,2,2,3,4,5,6,7] |

7 comparisons for n = 8 elements — consistent with the bound `n − 1`.

> **Note step k = 1:** when `L[i] == R[j]` we take from **L**, the left (earlier) half. That single choice is what makes merge sort **stable**.

## 9.4 Full trace of MERGE-SORT on `A = [38, 27, 43, 3, 9, 82, 10]`

**Divide phase (top-down):**

```
                    [38, 27, 43, 3, 9, 82, 10]
                    /                        \
        [38, 27, 43, 3]                    [9, 82, 10]
          /        \                        /       \
    [38, 27]     [43, 3]              [9, 82]      [10]
     /    \       /    \               /    \
  [38]   [27]  [43]   [3]           [9]   [82]
```

**Conquer / merge phase (bottom-up):**

```
  [38]   [27]  ->  [27, 38]
  [43]   [3]   ->  [3, 43]
  [27,38] [3,43] -> [3, 27, 38, 43]

  [9]  [82]    ->  [9, 82]
  [9,82] [10]  ->  [9, 10, 82]

  [3,27,38,43]  +  [9,10,82]  ->  [3, 9, 10, 27, 38, 43, 82]
```

**Final: `[3, 9, 10, 27, 38, 43, 82]`** ✔

## 9.5 Analysis of MERGE

For a merge of `n` total elements:

- copying into L and R: `n` moves
- the main loop runs exactly `r − p + 1 = n` times
- each iteration does 1 comparison and 1 assignment

Comparisons are between `⌈n/2⌉` (one half exhausts immediately) and `n − 1` (halves interleave). Either way:

```
MERGE runs in Θ(n) time and uses Θ(n) extra space.
```

## 9.6 The recurrence

Let `T(n)` be the time to merge-sort n elements.

| Step | Cost |
|---|---|
| Divide (compute q) | Θ(1) |
| Conquer (two subproblems of size n/2) | 2T(n/2) |
| Combine (merge) | Θ(n) |

```
        ⎧ Θ(1)                    if n = 1
T(n) =  ⎨
        ⎩ 2T(n/2) + Θ(n)          if n > 1
```

Or in the exact-constant form: `T(n) = 2T(n/2) + cn`, `T(1) = c`.

## 9.7 Solving it three ways

### (a) Recursion tree

At level `i` there are `2^i` nodes, each of size `n/2^i`, each doing `c·n/2^i` merge work:

```
cost at level i = 2^i × c(n/2^i) = cn        ← the SAME at every level
```

Number of levels: sizes go n, n/2, n/4, …, 1, so there are `log₂ n + 1` levels.

```
T(n) = cn × (log₂ n + 1) = cn log₂ n + cn = Θ(n log n)
```

Picture for n = 8:

```
level 0:                  cn                   = 8c   (1 node × 8)
level 1:          cn/2  +  cn/2                = 8c   (2 nodes × 4)
level 2:     cn/4 + cn/4 + cn/4 + cn/4         = 8c   (4 nodes × 2)
level 3:  c+c+c+c+c+c+c+c                      = 8c   (8 nodes × 1)
                                          ------------
                       4 levels × 8c  =  32c  =  cn log₂n + cn  ✔
```

### (b) Master theorem

`a = 2`, `b = 2`, `f(n) = Θ(n)`. Then `n^(log_b a) = n^(log₂ 2) = n¹ = n`.

`f(n) = Θ(n^(log_b a)) = Θ(n)` -> **Case 2** -> `T(n) = Θ(n^(log_b a) · log n) = Θ(n log n)`. ∎

### (c) Substitution (backward, assuming n = 2^k)

```
T(n) = 2T(n/2) + cn
     = 2[2T(n/4) + cn/2] + cn        = 4T(n/4) + 2cn
     = 4[2T(n/8) + cn/4] + 2cn       = 8T(n/8) + 3cn
     …
     = 2^k T(n/2^k) + k·cn
```

Stop when `n/2^k = 1`, i.e. `k = log₂ n`, `2^k = n`:

```
T(n) = n·T(1) + cn log₂ n = cn + cn log₂ n = Θ(n log n)  ∎
```

**Three independent methods, same answer.** That is exactly the kind of cross-check an exam answer should show.

## 9.8 Why merge sort's best case is also Θ(n log n)

Insertion sort adapts to sorted input; merge sort does not. Even if the array is already sorted, the algorithm still:

- splits all the way down to single elements (log n levels, unconditional),
- merges back up, touching all n elements at every level.

The recursion structure is **input-independent**, so

```
Best = Average = Worst = Θ(n log n)
```

This predictability is merge sort's biggest selling point over quick sort, which degrades to Θ(n²) on bad inputs.

## 9.9 Space complexity

- **Merge buffer:** Θ(n) — the dominant cost.
- **Recursion stack:** Θ(log n).
- **Total auxiliary:** Θ(n).

A clever implementation reuses one shared buffer of size n instead of allocating fresh `L`/`R` arrays at every call, which keeps it Θ(n) rather than "Θ(n) per level".

> **Linked lists change everything.** On a linked list, merging can be done by relinking pointers with **O(1)** extra space. Merge sort is therefore *the* sort of choice for linked lists, whereas quick sort (which needs random access) is a poor fit.

## 9.10 Properties of merge sort

| Property | Value | Note |
|---|---|---|
| Time (all cases) | **Θ(n log n)** | guaranteed, input-independent |
| Space | **Θ(n)** | its main weakness for arrays |
| **Stable** | ✔ | thanks to `L[i] <= R[j]` |
| In place | ✘ (arrays) / ✔ (lists) | |
| Parallelisable | ✔ | the two recursive calls are independent |
| **External sorting** | ✔ | the standard method for data too big for RAM: sort chunks, then k-way merge |
| Adaptive | ✘ | plain merge sort ignores existing order (Timsort fixes this) |

## 9.11 Insertion sort vs merge sort — the head-to-head

| Criterion | Insertion sort | Merge sort |
|---|---|---|
| Worst-case time | Θ(n²) | **Θ(n log n)** |
| Best-case time | **Θ(n)** | Θ(n log n) |
| Auxiliary space | **Θ(1)** | Θ(n) |
| Stable | ✔ | ✔ |
| Approach | incremental (iterative) | divide and conquer (recursive) |
| Good for | small or nearly-sorted n | large n, linked lists, external data |
| Constant factor | small | larger |

**Concrete numbers.** Suppose insertion sort costs `8n²` and merge sort costs `64 n log₂ n` operations.

- n = 8: IS = 512, MS = 1536 -> **insertion sort wins**
- n = 64: IS = 32 768, MS = 24 576 -> **merge sort wins**
- n = 10⁶: IS = 8 × 10¹² , MS ≈ 1.3 × 10⁹ -> **merge sort wins by ~6000×**

The crossover point is why real libraries use a **hybrid**: merge sort (or quick sort) down to a chunk of ~16–32 elements, then insertion sort. Java's `Arrays.sort` and Python's `sorted` (Timsort) both do exactly this.
---

# 10. Analysis of Recursive Algorithms — Writing the Recurrence

## 10.1 What a recurrence is

> A **recurrence relation** is an equation that describes a function in terms of its value on smaller inputs, together with one or more **base cases**.

Example: `T(n) = T(n−1) + 1`, `T(1) = 1`. Its solution is `T(n) = n`.

For recursive algorithms, `T(n)` = running time on input of size n. The recurrence writes that time as "time for the recursive calls + time for the work done outside them".

## 10.2 The general recipe

```
T(n) = (cost of dividing)
     + (sum of T(sizes of the subproblems))
     + (cost of combining)
```

with a base case `T(small) = Θ(1)`.

**Ask three questions about the code:**

1. **How many** recursive calls are made? -> the coefficient `a`
2. **How much smaller** is each subproblem? -> `n/b` or `n − 1`
3. **How much non-recursive work** is there? -> `f(n)`

## 10.3 Ten worked recurrence set-ups

### (1) Factorial

```cpp
int fact(int n) {
    if (n <= 1) return 1;      // Θ(1)
    return n * fact(n - 1);    // one call on n-1, plus one multiply
}
```

```
T(n) = T(n−1) + Θ(1),   T(1) = Θ(1)     ⟹  T(n) = Θ(n)
```

### (2) Naive Fibonacci

```cpp
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);   // TWO calls
}
```

```
T(n) = T(n−1) + T(n−2) + Θ(1)   ⟹  T(n) = Θ(φⁿ) ≈ Θ(1.618ⁿ)  — exponential
```

(The recursion tree has ≈ `fib(n)` leaves, and `fib(n) ≈ φⁿ/√5`.)

### (3) Binary search (recursive)

```cpp
int bs(int A[], int lo, int hi, int key) {
    if (lo > hi) return -1;
    int mid = lo + (hi - lo) / 2;
    if (A[mid] == key) return mid;
    if (A[mid] < key) return bs(A, mid + 1, hi, key);   // ONE call, half size
    return bs(A, lo, mid - 1, key);
}
```

```
T(n) = T(n/2) + Θ(1)   ⟹  T(n) = Θ(log n)
```

### (4) Merge sort

```
T(n) = 2T(n/2) + Θ(n)   ⟹  Θ(n log n)
```

### (5) Quick sort — best case (balanced split)

```
T(n) = 2T(n/2) + Θ(n)   ⟹  Θ(n log n)
```

### (6) Quick sort — worst case (pivot is always the extreme)

```
T(n) = T(n−1) + T(0) + Θ(n) = T(n−1) + Θ(n)   ⟹  Θ(n²)
```

### (7) Quick sort — a fixed 1:9 unbalanced split

```
T(n) = T(n/10) + T(9n/10) + Θ(n)   ⟹  still Θ(n log n)
```

*Any constant-fraction split gives n log n* — the tree depth changes from log₂n to log_{10/9}n, a constant factor.

### (8) Quick select (average case)

```
T(n) = T(n/2) + Θ(n)   ⟹  Θ(n)
```

Note the contrast with merge sort: **one** recursive call instead of two, and the total collapses from n log n to n.

### (9) Strassen's matrix multiplication

```
T(n) = 7T(n/2) + Θ(n²)   ⟹  Θ(n^log₂7) = Θ(n^2.807…)
```

### (10) Tower of Hanoi

```
T(n) = 2T(n−1) + 1,  T(1) = 1   ⟹  T(n) = 2ⁿ − 1 = Θ(2ⁿ)
```

## 10.4 Conventions that keep recurrences readable

1. **Ignore floors and ceilings.** Write `T(n/2)` instead of `T(⌊n/2⌋) + T(⌈n/2⌉)`. It never changes the asymptotic answer for the recurrences in this course.
2. **Ignore the base case** in the asymptotics. `T(n) = Θ(1)` for small constant n is assumed and is usually omitted.
3. **Boundary conditions matter for exact solutions**, not for Θ-classes.

## 10.5 The three solution methods

| Method | How it works | Best when |
|---|---|---|
| **Substitution** | *Guess* the answer, then *prove* it by induction | You already suspect the answer; you need a rigorous proof |
| **Recursion tree** | Draw the tree, sum the cost level by level | You need to *find* the guess; uneven splits |
| **Master method** | Plug into a formula | The recurrence has the form `aT(n/b) + f(n)` |

**Standard exam strategy:** use the recursion tree to *guess*, then substitution to *prove*, or the master theorem if the form fits.

---

# 11. Substitution Method

## 11.1 The two steps

1. **Guess** the form of the solution.
2. **Prove** it by mathematical induction: find the constants that make it work.

The method proves an upper bound (O), a lower bound (Ω), or both (Θ). It is the only method that gives full rigour, which is why exam questions say "solve **using the substitution method**" when they want proof.

## 11.2 Example 1 — `T(n) = 2T(n/2) + n`, show `T(n) = O(n log n)`

**Guess:** `T(n) ≤ c·n·log₂ n` for some constant `c > 0` and all `n ≥ n₀`.

**Inductive step.** Assume it holds for all smaller values, in particular for `n/2`:

```
T(n/2) ≤ c(n/2) log₂(n/2)
```

Substitute into the recurrence:

```
T(n) =  2T(n/2) + n
     ≤  2 · [ c(n/2) log₂(n/2) ] + n
     =  cn · log₂(n/2) + n
     =  cn · (log₂ n − log₂ 2) + n
     =  cn log₂ n − cn + n
     ≤  cn log₂ n            provided that  −cn + n ≤ 0,  i.e.  c ≥ 1
```

**Base case.** We cannot use n = 1, because `c·1·log₂1 = 0` while `T(1) > 0`. Standard fix: start the induction at `n = 2`.

```
T(2) = 2T(1) + 2 = 2c₁ + 2.  Need 2c₁ + 2 ≤ c·2·log₂2 = 2c  ⟹  c ≥ c₁ + 1
```

So choosing `c = max(1, c₁ + 1)` and `n₀ = 2` completes the proof: **T(n) = O(n log n)**. ∎

> 🔑 **The base-case trick.** Asymptotic notation only cares about `n ≥ n₀`, so you may *choose* `n₀` and ignore the awkward small values. Always mention this — examiners look for it.

## 11.3 Example 2 — `T(n) = T(n/2) + 1`, show `T(n) = O(log n)`

**Guess:** `T(n) ≤ c log₂ n` for `n ≥ 2`.

```
T(n) =  T(n/2) + 1
     ≤  c log₂(n/2) + 1
     =  c log₂ n − c + 1
     ≤  c log₂ n          provided  −c + 1 ≤ 0,  i.e.  c ≥ 1
```

**Base:** `T(2) = T(1) + 1 = c₁ + 1`. Need `c₁ + 1 ≤ c log₂2 = c`, so take `c = c₁ + 1`. ∎

## 11.4 Example 3 — the subtlety of subtracting a lower-order term

**Recurrence:** `T(n) = T(n/2) + T(n/2) + 1 = 2T(n/2) + 1`. Show `T(n) = O(n)`.

**Naive attempt.** Guess `T(n) ≤ cn`:

```
T(n) ≤ 2·c(n/2) + 1 = cn + 1
```

But `cn + 1 ≰ cn`. **The induction fails** — even though the guess is correct!

**Fix: strengthen the hypothesis by subtracting a lower-order term.** Guess `T(n) ≤ cn − d` for constants c, d > 0:

```
T(n) ≤ 2[c(n/2) − d] + 1
     =  cn − 2d + 1
     ≤  cn − d          provided  −2d + 1 ≤ −d,  i.e.  d ≥ 1
```

Now the induction goes through with `d = 1`. **T(n) = O(n)**. ∎

> ⚠️ This is the single most examinable subtlety of the substitution method: *a stronger hypothesis can be easier to prove.* If your induction leaves a stray additive constant, subtract a lower-order term.

## 11.5 Example 4 — a wrong proof to learn from

**Claim (false):** `T(n) = 2T(n/2) + n` is `O(n)`.

**Bogus proof:** assume `T(n/2) ≤ c(n/2)`. Then

```
T(n) ≤ 2·c(n/2) + n = cn + n = (c + 1)n = O(n)   ✘ WRONG
```

**Why it is wrong:** the inductive hypothesis was `T(n) ≤ cn` with a *fixed* c. We derived `T(n) ≤ (c+1)n`, which is **not** `≤ cn`. The constant must stay **exactly the same** — you may not let it grow, or you could "prove" that every function is O(n).

> 🔑 **Rule: the induction must reproduce the hypothesis with the identical constant.** Landing on `(c+1)n`, `cn + 1`, or `c(n+1)` all mean *failure*.

## 11.6 Example 5 — changing variables

**Recurrence:** `T(n) = 2T(√n) + log₂ n`.

Substitute `m = log₂ n` (so `n = 2^m` and `√n = 2^(m/2)`):

```
T(2^m) = 2T(2^(m/2)) + m
```

Rename `S(m) = T(2^m)`:

```
S(m) = 2S(m/2) + m      ← this is merge sort's recurrence!
S(m) = Θ(m log m)
```

Change back, `m = log n`:

```
T(n) = Θ(log n · log log n)
```

∎

## 11.7 Example 6 — subtract-and-conquer

**Recurrence:** `T(n) = T(n−1) + n`, `T(1) = 1`. Guess `T(n) = O(n²)`, i.e. `T(n) ≤ cn²`.

```
T(n) ≤ c(n−1)² + n
     =  cn² − 2cn + c + n
     =  cn² − (2c − 1)n + c
     ≤  cn²              provided  (2c − 1)n ≥ c,  true for c ≥ 1 and n ≥ 1
```

∎ (The exact solution is `T(n) = n(n+1)/2 = Θ(n²)`.)

## 11.8 Backward substitution (iteration / expansion)

Strictly this is a separate technique, but exams often lump it under "substitution". You unroll the recurrence until you see the pattern.

### Example — `T(n) = T(n−1) + n`, `T(1) = 1`

```
T(n) = T(n−1) + n
     = [T(n−2) + (n−1)] + n
     = [T(n−3) + (n−2)] + (n−1) + n
     …
     = T(1) + 2 + 3 + … + n
     = 1 + (2 + 3 + … + n)
     = n(n+1)/2 = Θ(n²)
```

### Example — `T(n) = 2T(n−1) + 1`, `T(1) = 1` (Tower of Hanoi)

```
T(n) = 2T(n−1) + 1
     = 2[2T(n−2) + 1] + 1        = 4T(n−2) + 2 + 1
     = 4[2T(n−3) + 1] + 3        = 8T(n−3) + 4 + 2 + 1
     …
     = 2^k T(n−k) + (2^k − 1)
```

Put `k = n − 1` so that `T(n−k) = T(1) = 1`:

```
T(n) = 2^(n−1)·1 + 2^(n−1) − 1 = 2ⁿ − 1 = Θ(2ⁿ)
```

**Verify:** n = 3 -> 2³ − 1 = 7 moves — the classic Hanoi answer. ✔

### Example — `T(n) = T(n/2) + n`, `T(1) = 1`

```
T(n) = n + T(n/2)
     = n + n/2 + T(n/4)
     = n + n/2 + n/4 + … + 1
     = n(1 + 1/2 + 1/4 + …) < 2n
     = Θ(n)
```

A decreasing geometric series is dominated by its **first** term. Compare with merge sort, where every level costs the same.

## 11.9 Common recurrences you should recognise instantly

| Recurrence | Solution | Where it appears |
|---|---|---|
| `T(n) = T(n−1) + 1` | Θ(n) | linear recursion, factorial |
| `T(n) = T(n−1) + n` | Θ(n²) | quick sort worst case |
| `T(n) = T(n−1) + log n` | Θ(n log n) | — |
| `T(n) = 2T(n−1) + 1` | Θ(2ⁿ) | Tower of Hanoi |
| `T(n) = T(n/2) + 1` | Θ(log n) | binary search |
| `T(n) = T(n/2) + n` | Θ(n) | **quick select (avg)** |
| `T(n) = 2T(n/2) + 1` | Θ(n) | tree traversal, tree size |
| `T(n) = 2T(n/2) + n` | Θ(n log n) | **merge sort** |
| `T(n) = 2T(n/2) + n²` | Θ(n²) | — |
| `T(n) = 3T(n/2) + n` | Θ(n^1.585) | Karatsuba-style |
| `T(n) = 4T(n/2) + n` | Θ(n²) | naive matrix multiply (D&C) |
| `T(n) = 7T(n/2) + n²` | Θ(n^2.807) | **Strassen** |
| `T(n) = T(√n) + 1` | Θ(log log n) | — |

---

# 12. Recursion Tree Method

## 12.1 The idea

Draw the recursion as a tree.

- Each **node** = one subproblem, labelled with the **non-recursive cost** `f(size)` it contributes.
- Each node's **children** = its recursive calls.
- **Sum the costs level by level**, then sum the levels.

The recursion tree is the best method for *discovering* the answer. It is not, by itself, a proof — use substitution to verify (or the master theorem if it applies).

## 12.2 The five questions to answer for any tree

1. What is the **cost of a single node** at level i?
2. How many **nodes** are at level i?
3. What is the **total cost of level i**?
4. What is the **height** of the tree (index of the last level)?
5. What is the **cost of the leaves**, and does the sum over levels converge, stay flat, or grow?

## 12.3 Example 1 — `T(n) = 2T(n/2) + n` (merge sort)

```
level 0:                       n                              total = n
                          /         \
level 1:              n/2            n/2                      total = n
                     /   \          /   \
level 2:          n/4    n/4     n/4    n/4                    total = n
                  ...    ...     ...    ...
level i:      2^i nodes, each of size n/2^i                    total = n
                  ...
level log n:  n leaves, each costing T(1)=Θ(1)                 total = n
```

| Question | Answer |
|---|---|
| Cost of a node at level i | `n/2^i` |
| Number of nodes at level i | `2^i` |
| **Total cost of level i** | `2^i × n/2^i = n` — **constant across levels** |
| Height | subproblem size at level i is `n/2^i`; it hits 1 when `i = log₂ n` |
| Number of levels | `log₂ n + 1` |

```
T(n) = Σ(i=0 .. log₂n) n = n(log₂ n + 1) = Θ(n log n)
```

## 12.4 Example 2 — `T(n) = 2T(n/2) + n²` (top-heavy)

| Level | Nodes | Size | Cost per node | Level total |
|---|---|---|---|---|
| 0 | 1 | n | n² | n² |
| 1 | 2 | n/2 | (n/2)² = n²/4 | **n²/2** |
| 2 | 4 | n/4 | n²/16 | **n²/4** |
| i | 2^i | n/2^i | n²/4^i | **n²/2^i** |

```
T(n) = Σ(i=0 .. log n) n²/2^i = n² Σ (1/2)^i < n² · 2 = Θ(n²)
```

**Decreasing geometric series -> the ROOT dominates.** Total = Θ(root cost) = Θ(n²).

## 12.5 Example 3 — `T(n) = 4T(n/2) + n` (bottom-heavy)

| Level | Nodes | Size | Cost per node | Level total |
|---|---|---|---|---|
| 0 | 1 | n | n | n |
| 1 | 4 | n/2 | n/2 | **2n** |
| 2 | 16 | n/4 | n/4 | **4n** |
| i | 4^i | n/2^i | n/2^i | **2^i · n** |

Height = log₂ n, and the number of leaves is `4^(log₂ n) = n^(log₂ 4) = n²`.

```
T(n) = Σ(i=0 .. log₂n − 1) 2^i·n  +  Θ(n²)
     = n(2^(log₂ n) − 1) + Θ(n²)
     = n(n − 1) + Θ(n²)
     = Θ(n²)
```

**Increasing geometric series -> the LEAVES dominate.** Total = Θ(leaf cost) = Θ(n²).

## 12.6 The three shapes — the single most useful summary

| Shape of level costs | Dominated by | Result |
|---|---|---|
| **Decreasing** geometric (root heaviest) | root | `T(n) = Θ(f(n))` |
| **Equal** at every level | all levels equally | `T(n) = Θ(f(n) · log n)` |
| **Increasing** geometric (leaves heaviest) | leaves | `T(n) = Θ(number of leaves) = Θ(n^log_b a)` |

**This is exactly the master theorem, drawn as a picture.** Cases 3, 2 and 1 respectively.

## 12.7 Example 4 — an unequal split: `T(n) = T(n/3) + T(2n/3) + n`

```
                          n                            total = n
                       /     \
                    n/3       2n/3                     total = n
                   /   \      /    \
                n/9   2n/9  2n/9  4n/9                 total = n
                  ...
```

Every **full** level costs exactly `n`. But the branches now have **different depths**:

- shortest path (always divide by 3): `n -> n/3 -> n/9 -> … -> 1`, length `log₃ n`
- longest path (always take 2/3): `n -> 2n/3 -> 4n/9 -> … -> 1`, length `log_{3/2} n`

Once the shallow branches bottom out, later levels cost *less* than n. So:

```
Lower bound: T(n) ≥ n · log₃ n = Ω(n log n)
Upper bound: T(n) ≤ n · log_{3/2} n = O(n log n)
```

Since `log₃ n` and `log_{3/2} n` differ only by a constant factor, **T(n) = Θ(n log n)**.

**Verify by substitution.** Guess `T(n) ≤ cn log n`:

```
T(n) ≤ c(n/3)log(n/3) + c(2n/3)log(2n/3) + n
     = cn log n − cn[ (1/3)log 3 + (2/3)log(3/2) ] + n
     = cn log n − cn(log 3 − 2/3) + n
     ≤ cn log n          provided  c ≥ 1/(log₂3 − 2/3) ≈ 1.06
```

∎ (Using log base 2: log₂3 ≈ 1.585, so log₂3 − 2/3 ≈ 0.918.)

> **Moral: any split into constant fractions gives Θ(n log n).** A 99:1 split is still n log n — just with a bigger constant. This is why quick sort is fast in practice.

## 12.8 Example 5 — `T(n) = T(n/2) + n` again, as a tree

```
level 0:   n
level 1:   n/2
level 2:   n/4
...
level i:   n/2^i
```

Only **one** node per level (a path, not a branching tree):

```
T(n) = n + n/2 + n/4 + … = n·Σ(1/2)^i ≤ 2n = Θ(n)
```

**Compare with merge sort:** the only difference is `2T` vs `T`, and the answer changes from Θ(n log n) to Θ(n). **This single fact is why quick select is linear while quick sort is n log n.** Do not let it pass by.

## 12.9 Example 6 — `T(n) = 3T(n/4) + n²`

| Level | Nodes | Size | Cost per node | Level total |
|---|---|---|---|---|
| 0 | 1 | n | n² | n² |
| 1 | 3 | n/4 | n²/16 | (3/16)n² |
| 2 | 9 | n/16 | n²/256 | (3/16)²n² |
| i | 3^i | n/4^i | n²/16^i | **(3/16)^i n²** |

Ratio `3/16 < 1`, a decreasing geometric series:

```
T(n) ≤ n² Σ(i=0..∞) (3/16)^i = n² · 1/(1 − 3/16) = (16/13)n² = O(n²)
```

And clearly `T(n) ≥ n²` (root alone). Hence **Θ(n²)**.

Leaf count check: `3^(log₄ n) = n^(log₄ 3) = n^0.792`, far smaller than n². Root dominates. ✔

## 12.10 Drawing tips

- Label nodes with the **cost**, not the subproblem size, and you will not confuse yourself.
- Always write the **general level i** row before summing.
- Compute the **height** from the *size* recurrence: `n/b^i = 1 -> i = log_b n`.
- Compute the **leaf count** as `a^(log_b n) = n^(log_b a)`.
- Decide which of the three shapes you have, then quote the corresponding result.

---

# 13. Master Method (Master Theorem)

## 13.1 Statement

The master theorem solves recurrences of the form

```
T(n) = a·T(n/b) + f(n)        where a ≥ 1, b > 1 are constants
                              and f(n) is asymptotically positive
```

Interpretation: a problem of size n is split into **a** subproblems of size **n/b**, and `f(n)` is the cost of splitting plus combining.

Define the **watershed function**

```
n^(log_b a)     ← "the cost of all the leaves"
```

and compare `f(n)` with it.

> ### Case 1 — leaves dominate
> If `f(n) = O(n^(log_b a − ε))` for some constant **ε > 0**, then
> **`T(n) = Θ(n^(log_b a))`**
>
> ### Case 2 — balanced
> If `f(n) = Θ(n^(log_b a))`, then
> **`T(n) = Θ(n^(log_b a) · log n)`**
>
> ### Case 3 — root dominates
> If `f(n) = Ω(n^(log_b a + ε))` for some ε > 0, **AND** the regularity condition
> `a·f(n/b) ≤ c·f(n)` holds for some `c < 1` and all sufficiently large n, then
> **`T(n) = Θ(f(n))`**

## 13.2 How to apply it — a checklist

```
1. Read off a, b, f(n).
2. Compute  log_b a  and the watershed  n^(log_b a).
3. Compare f(n) with n^(log_b a):
     f smaller by a POLYNOMIAL factor  -> Case 1 -> Θ(n^(log_b a))
     f the same order                  -> Case 2 -> Θ(n^(log_b a) log n)
     f larger by a POLYNOMIAL factor   -> Case 3 -> check regularity -> Θ(f(n))
4. If none applies, the master theorem is silent — use a tree or substitution.
```

> 🔑 **"Polynomially smaller/larger" is the crux.** `f(n)` must differ from `n^(log_b a)` by a factor of `n^ε` for some fixed ε > 0. A logarithmic factor is **not** enough, and that is exactly what creates the gaps.

## 13.3 Worked examples — Case 1

### Example 1.1 — `T(n) = 8T(n/2) + n²`

`a = 8, b = 2, f(n) = n²`. `log₂ 8 = 3`, watershed `n³`.

Is `n² = O(n^(3−ε))`? Take ε = 1: `n² = O(n²)` ✔

**Case 1 -> `T(n) = Θ(n³)`.**

### Example 1.2 — `T(n) = 4T(n/2) + n`

`a = 4, b = 2, f(n) = n`. `log₂ 4 = 2`, watershed `n²`.

`n = O(n^(2−ε))` with ε = 1 ✔

**Case 1 -> `T(n) = Θ(n²)`.** (This is the naive divide-and-conquer matrix multiply.)

### Example 1.3 — `T(n) = 2T(n/2) + 1`

`a = 2, b = 2, f(n) = 1`. `log₂2 = 1`, watershed `n`.

`1 = O(n^(1−ε))` with ε = 0.5 ✔

**Case 1 -> `T(n) = Θ(n)`.** (Counting nodes in a binary tree.)

### Example 1.4 — `T(n) = 9T(n/3) + n`

`a = 9, b = 3`, `log₃9 = 2`, watershed `n²`. `f(n) = n = O(n^(2−1))` ✔

**Case 1 -> `T(n) = Θ(n²)`.**

## 13.4 Worked examples — Case 2

### Example 2.1 — `T(n) = 2T(n/2) + n` (merge sort)

`log₂2 = 1`, watershed `n`. `f(n) = n = Θ(n)` ✔

**Case 2 -> `T(n) = Θ(n log n)`.**

### Example 2.2 — `T(n) = T(n/2) + 1` (binary search)

`a = 1, b = 2`, `log₂1 = 0`, watershed `n⁰ = 1`. `f(n) = 1 = Θ(1)` ✔

**Case 2 -> `T(n) = Θ(1 · log n) = Θ(log n)`.**

### Example 2.3 — `T(n) = 16T(n/4) + n²`

`log₄16 = 2`, watershed `n²`. `f(n) = n² = Θ(n²)` ✔

**Case 2 -> `T(n) = Θ(n² log n)`.**

## 13.5 Worked examples — Case 3

### Example 3.1 — `T(n) = 3T(n/4) + n log n`

`a = 3, b = 4`, `log₄3 ≈ 0.793`, watershed `n^0.793`.

Is `n log n = Ω(n^(0.793 + ε))`? Take ε = 0.2: `n log n = Ω(n^0.993)` ✔ (since n log n grows faster than n).

**Regularity check:** need `a f(n/b) ≤ c f(n)` for some c < 1:

```
3 · (n/4) log(n/4) ≤ (3/4) n log n     for large n,  with c = 3/4 < 1 ✔
```

**Case 3 -> `T(n) = Θ(n log n)`.**

### Example 3.2 — `T(n) = 2T(n/2) + n²`

`log₂2 = 1`, watershed `n`. `n² = Ω(n^(1+1))` ✔
Regularity: `2(n/2)² = n²/2 ≤ (1/2)n²` with c = 1/2 ✔

**Case 3 -> `T(n) = Θ(n²)`.**

### Example 3.3 — `T(n) = 4T(n/2) + n³`

`log₂4 = 2`, watershed `n²`. `n³ = Ω(n^(2+1))` ✔
Regularity: `4(n/2)³ = n³/2 ≤ (1/2)n³` ✔

**Case 3 -> `T(n) = Θ(n³)`.**

## 13.6 When the master theorem FAILS — the gaps

### Gap A — between Case 2 and Case 3: `T(n) = 2T(n/2) + n log n`

`log₂2 = 1`, watershed `n`. Is `f(n) = n log n` polynomially larger than `n`?

```
(n log n)/n = log n
```

`log n` grows slower than **every** `n^ε`, so there is **no ε > 0** with `n log n = Ω(n^(1+ε))`. Case 3 does not apply. It is not Θ(n) either, so Case 2 fails too.

**The master theorem is silent. Solve it with a recursion tree:**

| Level | Nodes | Size | Cost per node | Level total |
|---|---|---|---|---|
| i | 2^i | n/2^i | (n/2^i)·log(n/2^i) | n·(log n − i) |

```
T(n) = Σ(i=0 .. log n) n(log n − i)
     = n Σ(k=0 .. log n) k              (substituting k = log n − i)
     = n · (log n)(log n + 1)/2
     = Θ(n log² n)
```

**Answer: Θ(n log² n).** Memorise this one — it is the classic "master theorem fails" exam question.

### Gap B — Case 1 boundary: `T(n) = 2T(n/2) + n/log n`

Watershed `n`. Is `n/log n = O(n^(1−ε))`? No — dividing by log n is not a polynomial reduction. **Silent.** (The true answer is Θ(n log log n).)

### Gap C — regularity fails: `T(n) = T(n/2) + n(2 − cos n)`

`f(n)` oscillates, so no constant c < 1 satisfies regularity for all large n. **Silent.**

### Gap D — non-polynomial f: `T(n) = 2T(n/2) + 2ⁿ`

`f(n) = 2ⁿ` is not polynomially related in the required sense; regularity `2·2^(n/2) ≤ c·2ⁿ` actually holds here, so Case 3 does apply and gives Θ(2ⁿ) — but many similar recurrences do not. Always run the regularity check.

### Gap E — `a` or `b` not constant: `T(n) = nT(n/2) + n`

`a = n` is **not a constant**. The master theorem does not apply at all.

## 13.7 The Extended / Generalised Master Theorem (very handy)

For recurrences of the shape

```
T(n) = a T(n/b) + Θ(n^k log^p n)      with a ≥ 1, b > 1, k ≥ 0, p real
```

Compare `k` with `log_b a`:

| Condition | Result |
|---|---|
| `log_b a > k` | `T(n) = Θ(n^(log_b a))` |
| `log_b a = k` and `p > −1` | `T(n) = Θ(n^k log^(p+1) n)` |
| `log_b a = k` and `p = −1` | `T(n) = Θ(n^k log log n)` |
| `log_b a = k` and `p < −1` | `T(n) = Θ(n^k)` |
| `log_b a < k` and `p ≥ 0` | `T(n) = Θ(n^k log^p n)` |
| `log_b a < k` and `p < 0` | `T(n) = Θ(n^k)` |

**Re-solve Gap A with it:** `T(n) = 2T(n/2) + n log n` -> `a=2, b=2, k=1, p=1`. `log₂2 = 1 = k`, and `p = 1 > −1`, so

```
T(n) = Θ(n¹ log^(1+1) n) = Θ(n log² n)   ✔ matches the tree
```

**Re-solve Gap B:** `n/log n = n log^(−1) n` -> `k = 1, p = −1`, `log₂2 = 1 = k`, `p = −1`:

```
T(n) = Θ(n log log n)   ✔
```

The extended form covers both gaps in one line. Learn it — it saves a lot of tree-drawing.

## 13.8 The Master Theorem for subtract-and-conquer

For recurrences with **subtraction** instead of division:

```
T(n) = a T(n − b) + f(n),   where a > 0, b > 0, f(n) = O(n^k)
```

| Condition | Result |
|---|---|
| `a < 1` | `T(n) = O(n^k)` |
| `a = 1` | `T(n) = O(n^(k+1))` |
| `a > 1` | `T(n) = O(n^k · a^(n/b))` |

**Examples:**

- `T(n) = T(n−1) + 1` -> a = 1, k = 0 -> `O(n^1) = O(n)` ✔
- `T(n) = T(n−1) + n` -> a = 1, k = 1 -> `O(n²)` ✔
- `T(n) = 2T(n−1) + 1` -> a = 2, b = 1, k = 0 -> `O(2ⁿ)` ✔ (Hanoi)
- `T(n) = 3T(n−1) + n` -> a = 3, b = 1, k = 1 -> `O(n·3ⁿ)`

## 13.9 Quick-reference table of master-method applications

| Recurrence | a | b | f(n) | log_b a | Case | Answer |
|---|---|---|---|---|---|---|
| `T(n)=T(n/2)+1` | 1 | 2 | 1 | 0 | 2 | Θ(log n) |
| `T(n)=T(n/2)+n` | 1 | 2 | n | 0 | 3 | Θ(n) |
| `T(n)=2T(n/2)+1` | 2 | 2 | 1 | 1 | 1 | Θ(n) |
| `T(n)=2T(n/2)+n` | 2 | 2 | n | 1 | 2 | Θ(n log n) |
| `T(n)=2T(n/2)+n²` | 2 | 2 | n² | 1 | 3 | Θ(n²) |
| `T(n)=2T(n/2)+n log n` | 2 | 2 | n log n | 1 | — | Θ(n log² n) (extended) |
| `T(n)=3T(n/2)+n` | 3 | 2 | n | 1.585 | 1 | Θ(n^1.585) |
| `T(n)=4T(n/2)+n` | 4 | 2 | n | 2 | 1 | Θ(n²) |
| `T(n)=4T(n/2)+n²` | 4 | 2 | n² | 2 | 2 | Θ(n² log n) |
| `T(n)=4T(n/2)+n³` | 4 | 2 | n³ | 2 | 3 | Θ(n³) |
| `T(n)=7T(n/2)+n²` | 7 | 2 | n² | 2.807 | 1 | **Θ(n^2.807)** Strassen |
| `T(n)=8T(n/2)+n²` | 8 | 2 | n² | 3 | 1 | Θ(n³) |
| `T(n)=9T(n/3)+n` | 9 | 3 | n | 2 | 1 | Θ(n²) |
| `T(n)=16T(n/4)+n²` | 16 | 4 | n² | 2 | 2 | Θ(n² log n) |
---

# 14. Divide-and-Conquer Technique

## 14.1 The three steps

> **Divide-and-conquer** solves a problem by breaking it into smaller instances **of the same problem**, solving those recursively, and combining their solutions.

| Step | Meaning |
|---|---|
| **1. Divide** | Break the problem of size n into `a` subproblems, each of size `n/b` |
| **2. Conquer** | Solve each subproblem **recursively**. If the size is small enough, solve it directly (the base case) |
| **3. Combine** | Merge the subproblem answers into an answer for the original problem |

The generic running time is therefore

```
T(n) = a·T(n/b) + D(n) + C(n)
```

where `D(n)` is the divide cost and `C(n)` the combine cost. This is exactly the master-theorem shape, which is why Sections 13 and 14 belong together.

## 14.2 The zoo of divide-and-conquer algorithms

| Algorithm | a | b | Work outside recursion | Result |
|---|---|---|---|---|
| Binary search | 1 | 2 | Θ(1) | Θ(log n) |
| **Merge sort** | 2 | 2 | Θ(n) merge | Θ(n log n) |
| Quick sort (avg) | 2 | 2 | Θ(n) partition | Θ(n log n) |
| **Quick select (avg)** | **1** | 2 | Θ(n) partition | **Θ(n)** |
| Max–min finding | 2 | 2 | Θ(1) | Θ(n) |
| Naive D&C matrix mult. | 8 | 2 | Θ(n²) additions | Θ(n³) |
| **Strassen** | **7** | 2 | Θ(n²) additions | **Θ(n^2.807)** |
| Karatsuba integer mult. | 3 | 2 | Θ(n) | Θ(n^1.585) |
| Closest pair of points | 2 | 2 | Θ(n) strip check | Θ(n log n) |
| Tower of Hanoi | 2 | (n−1) | Θ(1) | Θ(2ⁿ) |
| Convex hull (Quickhull) | 2 | 2 | Θ(n) | Θ(n log n) avg |

## 14.3 Advantages and disadvantages

**Advantages**

1. **Reduces complexity** — n² problems often become n log n.
2. **Naturally parallel** — the subproblems are independent, so they map straight onto multiple cores.
3. **Cache-friendly** — subproblems eventually fit in cache, giving speedups the RAM model does not even predict.
4. **Clean correctness proofs** by strong induction on n.

**Disadvantages**

1. **Recursion overhead** — function calls and stack frames cost time and memory.
2. **Extra space** — merge sort needs Θ(n); recursion needs Θ(depth).
3. **Not always a win** — if the combine step is expensive, you can end up slower.
4. **Recomputation** — if subproblems *overlap*, plain divide-and-conquer redoes work exponentially (that is precisely what dynamic programming, Unit 3, fixes).

> **D&C vs DP in one line:** divide-and-conquer is for **independent** subproblems; dynamic programming is for **overlapping** ones.

## 14.4 A short example — max and min in one pass

**Naive:** scan for max (n−1 comparisons), scan for min (n−1) -> `2n − 2`.

**Divide and conquer:** split, find (max, min) of each half, then 2 comparisons to combine.

```
T(n) = 2T(n/2) + 2,   T(2) = 1
```

Solving: `T(n) = 3n/2 − 2` comparisons — **25 % fewer**. Same Θ(n) class, better constant. A useful reminder that constants are not always worthless.

---

# 15. Strassen's Matrix Multiplication

## 15.1 The problem

Given two n×n matrices A and B, compute `C = A × B`, where

```
C[i][j] = Σ(k=1..n) A[i][k] · B[k][j]
```

**Standard algorithm:** three nested loops -> `n³` multiplications and `n³ − n²` additions -> **Θ(n³)**.

For n = 1000 that is 10⁹ multiplications. Can we do better?

## 15.2 First attempt — plain divide and conquer

Assume n is a power of 2. Split each matrix into four (n/2)×(n/2) blocks:

```
A = ⎡ A11  A12 ⎤     B = ⎡ B11  B12 ⎤     C = ⎡ C11  C12 ⎤
    ⎣ A21  A22 ⎦         ⎣ B21  B22 ⎦         ⎣ C21  C22 ⎦
```

Block matrix multiplication gives

```
C11 = A11·B11 + A12·B21
C12 = A11·B12 + A12·B22
C21 = A21·B11 + A22·B21
C22 = A21·B12 + A22·B22
```

Count: **8 multiplications** of (n/2)×(n/2) matrices and **4 additions** of (n/2)×(n/2) matrices (each addition costs Θ(n²)).

```
T(n) = 8T(n/2) + Θ(n²)
```

Master theorem: `a = 8, b = 2, log₂8 = 3`, watershed `n³`. `f(n) = n² = O(n^(3−1))` -> **Case 1** ->

```
T(n) = Θ(n³)
```

**No improvement at all.** The bottleneck is the **8** multiplications. Strassen's insight: trade multiplications for additions.

## 15.3 Strassen's seven products

Compute these **seven** products of (n/2)×(n/2) matrices (note: only 7, not 8):

```
M1 = (A11 + A22) × (B11 + B22)
M2 = (A21 + A22) ×  B11
M3 =  A11        × (B12 − B22)
M4 =  A22        × (B21 − B11)
M5 = (A11 + A12) ×  B22
M6 = (A21 − A11) × (B11 + B12)
M7 = (A12 − A22) × (B21 + B22)
```

Then assemble C using **additions and subtractions only**:

```
C11 = M1 + M4 − M5 + M7
C12 = M3 + M5
C21 = M2 + M4
C22 = M1 − M2 + M3 + M6
```

**Cost:** 7 multiplications + 18 additions/subtractions of (n/2)×(n/2) matrices (10 to build the operands, 8 to assemble C).

## 15.4 Verification of C11 (do this once and the magic disappears)

```
M1 + M4 − M5 + M7
= (A11+A22)(B11+B22) + A22(B21−B11) − (A11+A12)B22 + (A12−A22)(B21+B22)
```

Expand each term:

```
(A11+A22)(B11+B22) = A11B11 + A11B22 + A22B11 + A22B22
A22(B21−B11)       = A22B21 − A22B11
−(A11+A12)B22      = −A11B22 − A12B22
(A12−A22)(B21+B22) = A12B21 + A12B22 − A22B21 − A22B22
```

Add them all and cancel:

```
  A11B11 + A11B22 + A22B11 + A22B22
+          A22B21 − A22B11
−          A11B22          − A12B22
+ A12B21 + A12B22 − A22B21 − A22B22
------------------------------------
= A11B11 + A12B21     ✔  which is exactly C11
```

Every pairing cancels except the two wanted terms. The other three blocks verify the same way.

> ⚠️ **Matrix multiplication is not commutative**, so the *order* of factors in each M-term matters. `A11 × (B12 − B22)` is not the same as `(B12 − B22) × A11`. Write them exactly as given.

## 15.5 Fully worked numerical example (2×2, so each block is a scalar)

```
A = ⎡ 1  2 ⎤        B = ⎡ 5  6 ⎤
    ⎣ 3  4 ⎦            ⎣ 7  8 ⎦
```

**Standard method (for checking):**

```
C11 = 1·5 + 2·7 = 19        C12 = 1·6 + 2·8 = 22
C21 = 3·5 + 4·7 = 43        C22 = 3·6 + 4·8 = 50
```

**Strassen:** A11=1, A12=2, A21=3, A22=4; B11=5, B12=6, B21=7, B22=8.

| Product | Expression | Arithmetic | Value |
|---|---|---|---|
| M1 | (A11+A22)(B11+B22) | (1+4)(5+8) = 5×13 | **65** |
| M2 | (A21+A22)·B11 | (3+4)×5 = 7×5 | **35** |
| M3 | A11·(B12−B22) | 1×(6−8) | **−2** |
| M4 | A22·(B21−B11) | 4×(7−5) = 4×2 | **8** |
| M5 | (A11+A12)·B22 | (1+2)×8 = 3×8 | **24** |
| M6 | (A21−A11)(B11+B12) | (3−1)(5+6) = 2×11 | **22** |
| M7 | (A12−A22)(B21+B22) | (2−4)(7+8) = −2×15 | **−30** |

Assemble:

```
C11 = M1 + M4 − M5 + M7 = 65 + 8 − 24 − 30 = 19   ✔
C12 = M3 + M5           = −2 + 24            = 22   ✔
C21 = M2 + M4           = 35 + 8             = 43   ✔
C22 = M1 − M2 + M3 + M6 = 65 − 35 − 2 + 22   = 50   ✔
```

```
C = ⎡ 19  22 ⎤
    ⎣ 43  50 ⎦
```

Matches the standard method exactly. **7 multiplications instead of 8.**

## 15.6 Complexity analysis

```
T(n) = 7·T(n/2) + Θ(n²)
```

- **7** recursive multiplications of half-sized matrices -> `a = 7`
- each of size n/2 -> `b = 2`
- 18 matrix additions, each Θ(n²) -> `f(n) = Θ(n²)`

**Master theorem:** `log₂ 7 ≈ 2.807355`, watershed `n^2.807`.

Is `f(n) = n² = O(n^(2.807 − ε))`? Take ε = 0.8: `n² = O(n^2.007)` ✔ -> **Case 1**.

```
T(n) = Θ(n^(log₂ 7)) = Θ(n^2.8074) ≈ Θ(n^2.81)
```

**Compare with Θ(n³):**

| n | n³ | n^2.807 | Speedup |
|---:|---:|---:|---:|
| 100 | 10⁶ | 4.1 × 10⁵ | 2.4× |
| 1 000 | 10⁹ | 2.6 × 10⁸ | 3.8× |
| 10 000 | 10¹² | 1.7 × 10¹¹ | 6.0× |
| 10⁶ | 10¹⁸ | 6.9 × 10¹⁶ | 14× |

The advantage grows without bound, but slowly.

## 15.7 Practical caveats (examiners like these)

1. **Larger constant factor.** 18 matrix additions vs the standard algorithm's 4. Strassen only pays off above a **crossover point** typically around n ≈ 64–128 in real implementations.
2. **Extra memory.** The seven M-matrices and the operand sums need Θ(n²) additional storage per level — the standard algorithm needs almost none.
3. **Numerical stability.** All those subtractions cause cancellation, so floating-point error is larger than the standard algorithm's. This is the main reason numerical-computing libraries (LAPACK/BLAS) usually stick with the n³ method.
4. **n must be a power of 2.** If it is not, **pad** the matrices with zero rows and columns up to the next power of 2. Padding at most doubles n, which changes the constant but not the Θ class.
5. **Not cache-optimal by default** — a well-tuned blocked n³ implementation can beat a naive Strassen implementation on real hardware.

## 15.8 Where Strassen sits in history

| Year | Author | Exponent ω | Complexity |
|---|---|---|---|
| — | Standard | 3 | Θ(n³) |
| 1969 | **Strassen** | **2.807** | Θ(n^2.807) |
| 1990 | Coppersmith–Winograd | 2.376 | Θ(n^2.376) |
| 2020s | Alman, Vassilevska Williams et al. | ≈ 2.371 | galactic — constants far too large for real use |
| lower bound | — | ≥ 2 | you must at least read n² inputs |

The exact value of ω is one of the great open problems of theoretical computer science. Strassen matters historically because it was the **first** proof that n³ is not optimal — it broke a barrier everyone assumed was solid.

## 15.9 Implementation sketch

```cpp
// n must be a power of 2; matrices are n x n
Matrix strassen(const Matrix& A, const Matrix& B, int n) {
    if (n <= THRESHOLD)                  // e.g. 64 — below this, plain O(n^3) wins
        return standardMultiply(A, B, n);

    int h = n / 2;
    Matrix A11, A12, A21, A22, B11, B12, B21, B22;
    split(A, A11, A12, A21, A22);        // Θ(n^2)
    split(B, B11, B12, B21, B22);

    Matrix M1 = strassen(add(A11, A22), add(B11, B22), h);
    Matrix M2 = strassen(add(A21, A22), B11,            h);
    Matrix M3 = strassen(A11,           sub(B12, B22),  h);
    Matrix M4 = strassen(A22,           sub(B21, B11),  h);
    Matrix M5 = strassen(add(A11, A12), B22,            h);
    Matrix M6 = strassen(sub(A21, A11), add(B11, B12),  h);
    Matrix M7 = strassen(sub(A12, A22), add(B21, B22),  h);

    Matrix C11 = add(sub(add(M1, M4), M5), M7);   // M1 + M4 - M5 + M7
    Matrix C12 = add(M3, M5);
    Matrix C21 = add(M2, M4);
    Matrix C22 = add(add(sub(M1, M2), M3), M6);   // M1 - M2 + M3 + M6

    return combine(C11, C12, C21, C22);           // Θ(n^2)
}
```

---

# 16. Order Statistics — Quick Select, k-th Smallest, k-th Largest

## 16.1 Definitions

> The **i-th order statistic** of a set of n elements is the **i-th smallest** element.

| Name | Order statistic |
|---|---|
| **Minimum** | 1st order statistic |
| **Maximum** | n-th order statistic |
| **Lower median** | ⌊(n+1)/2⌋-th |
| **Upper median** | ⌈(n+1)/2⌉-th |
| **k-th smallest** | k-th |
| **k-th largest** | **(n − k + 1)-th smallest** |

> 🔑 **Memorise the conversion:** `k-th largest = (n − k + 1)-th smallest`.
> With **0-indexed** array positions in the sorted order: k-th smallest sits at index `k − 1`, and k-th largest sits at index `n − k`.

**Example.** `A = [7, 10, 4, 3, 20, 15]`, n = 6. Sorted: `[3, 4, 7, 10, 15, 20]`.

- 3rd smallest = **7** (index 2)
- 3rd largest = (6 − 3 + 1) = 4th smallest = **10** (index 6 − 3 = 3) ✔

**The selection problem:** find the k-th smallest **without fully sorting**.

## 16.2 The approaches, ranked

| Approach | Time | Space | Comment |
|---|---|---|---|
| Sort, then index | O(n log n) | O(1)–O(n) | simple, wasteful |
| Min-heap, extract k times | O(n + k log n) | O(n) | good when k is tiny |
| Max-heap of size k | O(n log k) | O(k) | best for streams / k ≪ n |
| **Quick select** | **O(n) average**, O(n²) worst | O(1) | the standard answer |
| Median of medians (BFPRT) | **O(n) worst case** | O(log n) | theoretical guarantee |
| Counting sort based | O(n + range) | O(range) | only for small integer ranges |

**Key insight:** sorting gives you *all* n order statistics, but we only want *one*. Doing full sorting is over-solving the problem — and quick select proves we can do it in linear time.

## 16.3 The partition subroutine (Lomuto scheme)

Partition is the engine of quick select. It picks a **pivot** and rearranges the array so that

```
[ elements ≤ pivot ]  [ pivot ]  [ elements > pivot ]
                          ↑
                    final position p
```

After partition, **`A[p]` is already in its correct sorted position** — that is the whole trick.

```cpp
int partition(vector<int>& A, int lo, int hi) {
    int pivot = A[hi];              // choose the last element as pivot
    int i = lo - 1;                 // boundary of the "≤ pivot" region
    for (int j = lo; j < hi; ++j) {
        if (A[j] <= pivot) {
            ++i;
            swap(A[i], A[j]);
        }
    }
    swap(A[i + 1], A[hi]);          // put the pivot in its place
    return i + 1;                   // the pivot's final index
}
```

**Cost:** exactly `hi − lo` comparisons -> **Θ(n)** time, **Θ(1)** extra space.

### Partition trace on `A = [7, 10, 4, 3, 20, 15]`, lo=0, hi=5

pivot = A[5] = 15, i = −1.

| j | A[j] | A[j] ≤ 15? | Action | Array |
|---|---|---|---|---|
| 0 | 7 | yes | i=0, swap A[0],A[0] | [7,10,4,3,20,15] |
| 1 | 10 | yes | i=1, swap A[1],A[1] | [7,10,4,3,20,15] |
| 2 | 4 | yes | i=2, swap A[2],A[2] | [7,10,4,3,20,15] |
| 3 | 3 | yes | i=3, swap A[3],A[3] | [7,10,4,3,20,15] |
| 4 | 20 | no | — | [7,10,4,3,20,15] |

Final swap: `A[i+1] = A[4]` with `A[5]` -> `[7, 10, 4, 3, 15, 20]`, **return 4**.

Check: everything left of index 4 is ≤ 15, everything right is > 15. ✔

## 16.4 Quick Select — the algorithm

**Idea.** Partition once. Look at where the pivot landed:

- pivot index **== target** -> we found it, **stop**
- pivot index **> target** -> the answer lies in the **left** part -> recurse left only
- pivot index **< target** -> the answer lies in the **right** part -> recurse right only

Unlike quick sort, we recurse into **only one side**. That is the entire difference, and it is what turns n log n into n.

**Pseudocode:**

```
QUICK-SELECT(A, lo, hi, target)         // target = 0-indexed position wanted
1  if lo == hi
2      return A[lo]
3  p = PARTITION(A, lo, hi)
4  if p == target
5      return A[p]
6  else if target < p
7      return QUICK-SELECT(A, lo, p−1, target)
8  else
9      return QUICK-SELECT(A, p+1, hi, target)
```

**C++ — iterative (preferred: O(1) space, no stack):**

```cpp
// returns the element that would sit at index `target` after sorting
int quickSelect(vector<int>& A, int target) {
    int lo = 0, hi = (int)A.size() - 1;
    while (lo <= hi) {
        int p = partition(A, lo, hi);
        if (p == target)      return A[p];
        else if (p < target)  lo = p + 1;      // answer is to the right
        else                  hi = p - 1;      // answer is to the left
    }
    return -1;                                  // unreachable for valid target
}

int kthSmallest(vector<int> A, int k) {         // k is 1-indexed
    return quickSelect(A, k - 1);
}

int kthLargest(vector<int> A, int k) {          // k is 1-indexed
    return quickSelect(A, (int)A.size() - k);
}
```

## 16.5 Complete trace — 3rd smallest of `[7, 10, 4, 3, 20, 15]`

n = 6, k = 3 -> `target = k − 1 = 2`. (Expected answer from sorted `[3,4,7,10,15,20]` is **7**.)

**Iteration 1** — lo=0, hi=5, pivot = 15.

From the trace above: array becomes `[7, 10, 4, 3, 15, 20]`, p = 4.
`p = 4 > target = 2` -> search left -> **hi = 3**.

**Iteration 2** — lo=0, hi=3, subarray `[7, 10, 4, 3]`, pivot = A[3] = 3.

| j | A[j] | ≤ 3? | Action |
|---|---|---|---|
| 0 | 7 | no | — |
| 1 | 10 | no | — |
| 2 | 4 | no | — |

i stays −1; final swap `A[0] ↔ A[3]` -> `[3, 10, 4, 7, 15, 20]`, **p = 0**.
`p = 0 < target = 2` -> search right -> **lo = 1**.

**Iteration 3** — lo=1, hi=3, subarray `[10, 4, 7]`, pivot = A[3] = 7.

| j | A[j] | ≤ 7? | Action | Array |
|---|---|---|---|---|
| 1 | 10 | no | — | [3,10,4,7,15,20] |
| 2 | 4 | yes | i=1, swap A[1],A[2] | [3,4,10,7,15,20] |

Final swap `A[2] ↔ A[3]` -> `[3, 4, 7, 10, 15, 20]`, **p = 2**.
`p == target` -> **return A[2] = 7** ✔

**Work done:** 5 + 3 + 2 = 10 comparisons, versus the ~15 that a full sort of 6 elements would need — and the gap widens dramatically with n.

## 16.6 Complexity analysis

### Best case — the pivot lands on the target immediately

```
T(n) = Θ(n)      one partition and done
```

### Average case — the pivot splits roughly in half

```
T(n) = T(n/2) + Θ(n)
```

Solve by backward substitution:

```
T(n) = n + n/2 + n/4 + n/8 + … + 1
     = n(1 + 1/2 + 1/4 + …)
     < 2n
     = Θ(n)
```

Master theorem: `a = 1, b = 2, log₂1 = 0`, watershed `n⁰ = 1`. `f(n) = n = Ω(n^(0+1))` -> **Case 3** -> `T(n) = Θ(n)`.

> 🔑 **The comparison that explains everything:**
> merge sort `T(n) = 2T(n/2) + n = Θ(n log n)`
> quick select `T(n) = 1T(n/2) + n = Θ(n)`
> One recursive call instead of two. That single change removes the `log n` factor entirely, because the total work forms a **decreasing geometric series** instead of a flat one.

### A sharper average-case argument (the 3:1 split bound)

A pivot is "good" if it falls in the middle half of the array (between the 25th and 75th percentile). A random pivot is good with probability 1/2, so on average every 2 partitions produce a good split, which shrinks the problem to at most `3n/4`.

```
T(n) ≤ T(3n/4) + Θ(n)
     ≤ n + (3/4)n + (9/16)n + …
     = n · 1/(1 − 3/4)
     = 4n = Θ(n)
```

Expected comparisons are about `4n` — linear with a small constant.

### Worst case — the pivot is always the extreme

Input already sorted (or reverse sorted) with last-element pivot: each partition peels off exactly one element.

```
T(n) = T(n−1) + Θ(n)
     = n + (n−1) + (n−2) + … + 1
     = n(n+1)/2
     = Θ(n²)
```

**Example:** `A = [1, 2, 3, 4, 5]` looking for the minimum. Pivot 5 lands at index 4, recurse on `[1,2,3,4]`, pivot 4 lands at index 3, and so on — a full quadratic disaster.

### Space

- **Iterative version: Θ(1)** — the loop replaces the recursion entirely (tail recursion eliminated by hand).
- Recursive version: Θ(log n) average, Θ(n) worst-case stack depth.

## 16.7 How to avoid the worst case

| Technique | Effect |
|---|---|
| **Randomised pivot** — `swap(A[hi], A[lo + rand()%(hi-lo+1)])` before partitioning | worst case becomes vanishingly unlikely; **expected** O(n) for *every* input |
| **Median-of-three** — pivot = median of A[lo], A[mid], A[hi] | kills the sorted-input worst case cheaply; the standard practical choice |
| **Median of medians (BFPRT)** | guarantees O(n) **worst case**, but with a large constant |

```cpp
int randomizedPartition(vector<int>& A, int lo, int hi) {
    int r = lo + rand() % (hi - lo + 1);
    swap(A[r], A[hi]);            // move the random pivot to the end
    return partition(A, lo, hi);
}
```

With a random pivot, no adversary can choose an input that forces Θ(n²) — the bad case depends on the coin flips, not on the data.

## 16.8 Median of Medians — worst-case linear selection (outline)

**The algorithm:**

1. Divide the n elements into `⌈n/5⌉` groups of 5.
2. Sort each group (constant work each, since 5 is a constant) and take its median.
3. **Recursively** find the median `x` of those `n/5` medians.
4. Partition the array around `x`.
5. Recurse into the appropriate side.

**Why the pivot is good.** At least half of the `n/5` group medians are ≤ x, and each of those groups contributes at least 3 elements ≤ x. So

```
number of elements ≤ x  ≥  3 · (1/2)(n/5)  ≈  3n/10
```

Symmetrically at least 3n/10 are ≥ x. Therefore each side of the partition holds **at most 7n/10** elements — a guaranteed constant-fraction split.

**Recurrence:**

```
T(n) ≤ T(n/5)      ← finding the median of medians
     + T(7n/10)    ← recursing into one side
     + Θ(n)        ← grouping and partitioning
```

Since `1/5 + 7/10 = 9/10 < 1`, the work shrinks geometrically:

```
T(n) ≤ cn(1 + 9/10 + (9/10)² + …) = cn · 10 = Θ(n)
```

**T(n) = Θ(n) in the worst case.** ∎

> **Why groups of 5?** With groups of 3 the recurrence becomes `T(n/3) + T(2n/3) + Θ(n)`, and `1/3 + 2/3 = 1` — the series no longer converges, giving Θ(n log n). Five is the smallest odd group size that makes the fractions sum to less than 1.

In practice the constant is large (roughly 10n comparisons), so randomised quick select is preferred; median-of-medians matters mainly as a proof that worst-case linear selection is *possible*.

## 16.9 k-th largest — the three usual solutions

**Problem:** find the k-th largest element of `A = [3, 2, 1, 5, 6, 4]`, k = 2. Answer: **5**.

### Solution A — quick select (best general answer)

```cpp
int findKthLargest(vector<int>& A, int k) {
    return quickSelect(A, (int)A.size() - k);   // index n-k in sorted order
}
```

n = 6, k = 2 -> target index = 4. Sorted `[1,2,3,4,5,6]`, index 4 is **5** ✔
**O(n) average, O(1) space.**

### Solution B — min-heap of size k (best when k ≪ n, and works on streams)

```cpp
int findKthLargest(vector<int>& A, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;   // min-heap
    for (int x : A) {
        minHeap.push(x);
        if ((int)minHeap.size() > k) minHeap.pop();  // drop the smallest
    }
    return minHeap.top();     // the heap holds the k largest; its root is the k-th
}
```

**O(n log k) time, O(k) space.** The heap always contains the k largest values seen so far, so its minimum is the k-th largest. This is the version to use when the data arrives as a stream and you cannot hold it all.

### Solution C — sort (simplest, use when n is small)

```cpp
int findKthLargest(vector<int> A, int k) {
    sort(A.begin(), A.end(), greater<int>());
    return A[k - 1];
}
```

**O(n log n) time.**

### Choosing between them

| Situation | Use |
|---|---|
| One query, array in memory | **quick select** |
| k very small, or streaming data | **min-heap of size k** |
| Many different k on the same array | **sort once**, then answer each in O(1) |
| Small integer value range | counting sort |
| Hard worst-case guarantee required | median of medians |

## 16.10 Related problems solved with the same machinery

| Problem | Method |
|---|---|
| Median of an array | quick select with target `n/2` |
| Top-k largest elements | quick select to index `n−k`, then take everything to the right (unsorted, O(n)) |
| k closest points to origin | quick select on squared distance |
| k-th smallest in a sorted matrix | binary search on the value, not the index |
| k-th largest in a stream | min-heap of size k |
| Median of two sorted arrays | binary search partitioning, O(log(min(m,n))) |

## 16.11 Quick select vs quick sort — the final comparison

| | Quick sort | Quick select |
|---|---|---|
| Recursive calls after partition | **2** (both sides) | **1** (one side) |
| Recurrence (avg) | `2T(n/2) + n` | `T(n/2) + n` |
| Average time | Θ(n log n) | **Θ(n)** |
| Worst time | Θ(n²) | Θ(n²) |
| Output | fully sorted array | one element |
| Space (iterative) | Θ(log n) | **Θ(1)** |
---

# 17. Practice Questions with Full Solutions

Work each one on paper *before* reading the solution. The solutions show the full method, not just the answer, because that is what earns marks.

## Part A — Asymptotic notation

### Q1. Prove that `7n + 5 = O(n)`, stating c and n₀.

**Solution.** We need `7n + 5 ≤ c·n` for all n ≥ n₀.

```
7n + 5 ≤ 7n + 5n = 12n     whenever 5 ≤ 5n, i.e. n ≥ 1
```

Take **c = 12, n₀ = 1**. (Also valid: c = 8, n₀ = 5, since `7n + 5 ≤ 8n ⟺ 5 ≤ n`.) ∎

---

### Q2. Prove that `n² + 10n = Θ(n²)`.

**Solution.** Two parts.

*Upper bound (O):* for n ≥ 1, `10n ≤ 10n²`, so `n² + 10n ≤ 11n²` -> c₂ = 11.
*Lower bound (Ω):* `n² + 10n ≥ n²` for all n ≥ 1 -> c₁ = 1.

Hence `1·n² ≤ n² + 10n ≤ 11·n²` for all n ≥ 1, so **Θ(n²)** with c₁ = 1, c₂ = 11, n₀ = 1. ∎

---

### Q3. Prove that `2ⁿ⁺¹ = O(2ⁿ)` but `2^(2n) ≠ O(2ⁿ)`.

**Solution.**

*First part:* `2ⁿ⁺¹ = 2·2ⁿ ≤ 2·2ⁿ`, so c = 2, n₀ = 1. ✔

*Second part:* suppose `2^(2n) ≤ c·2ⁿ`. Then `(2ⁿ)²/2ⁿ = 2ⁿ ≤ c` for all n ≥ n₀ — impossible, since 2ⁿ is unbounded. ∎

---

### Q4. Arrange in increasing order of growth: `n², 2ⁿ, n log n, n!, log n, n, √n, n³, 1, log log n`.

**Solution.**

```
1 < log log n < log n < √n < n < n log n < n² < n³ < 2ⁿ < n!
```

---

### Q5. Is `n² = O(n³)`? Is `n³ = O(n²)`? Is `n² = Θ(n³)`?

**Solution.**

- `n² = O(n³)` — **yes**, with c = 1, n₀ = 1. Big-O is an upper bound and may be loose.
- `n³ = O(n²)` — **no**. It would force `n ≤ c`.
- `n² = Θ(n³)` — **no**, because the Ω half fails: `n²/n³ = 1/n -> 0`.

---

### Q6. Compare `f(n) = n^(log₂ 3)` and `g(n) = 3^(log₂ n)`.

**Solution.** Use the identity `a^(log_b c) = c^(log_b a)` with a = 3, b = 2, c = n:

```
3^(log₂ n) = n^(log₂ 3)
```

So `f(n) = g(n)` exactly — **f = Θ(g)**, and both are ≈ n^1.585.

---

### Q7. Simplify to a Θ class: (a) `log(n!)` (b) `n^(1/log n)` (c) `√(n log n)` (d) `2^(log₄ n)`

**Solution.**

(a) Stirling: `log(n!) = Θ(n log n)`.
(b) Let `x = n^(1/log₂ n)`. Then `log₂ x = (log₂ n)/(log₂ n) = 1`, so `x = 2` — **Θ(1)**.
(c) `√(n log n) = n^0.5 (log n)^0.5` — **Θ(√(n log n))**, which lies strictly between √n and n.
(d) `2^(log₄ n) = n^(log₄ 2) = n^(1/2)` — **Θ(√n)**.

---

## Part B — Iterative analysis

### Q8. Give the time complexity.

```cpp
for (int i = 1; i <= n; i++)
    for (int j = 1; j <= n; j += i)
        cout << "*";
```

**Solution.** For a fixed `i`, the inner loop runs about `n/i` times. Total:

```
Σ(i=1..n) n/i = n · Σ(i=1..n) 1/i = n · H(n) = n·Θ(log n) = Θ(n log n)
```

**Answer: Θ(n log n)** — the harmonic series is the key. Many students wrongly say Θ(n²).

---

### Q9. Give the time complexity.

```cpp
int i = n;
while (i > 0) {
    for (int j = 0; j < i; j++)
        doWork();
    i = i / 2;
}
```

**Solution.** The outer loop values of `i` are `n, n/2, n/4, …, 1`. The inner loop runs `i` times, so total work is

```
n + n/2 + n/4 + … + 1 = 2n − 1 = Θ(n)
```

**Answer: Θ(n)** — a decreasing geometric series, *not* Θ(n log n).

---

### Q10. Give the time complexity.

```cpp
for (int i = 0; i < n; i++)
    for (int j = 0; j < i * i; j++)
        for (int k = 0; k < j; k++)
            doWork();
```

**Solution.** Inner two loops for a fixed i: `Σ(j=0..i²−1) j ≈ i⁴/2`. Then

```
Σ(i=0..n−1) i⁴/2 ≈ (1/2)·n⁵/5 = Θ(n⁵)
```

**Answer: Θ(n⁵)** (using `Σ i^k = Θ(n^(k+1))`).

---

### Q11. How many times is the loop body executed?

```cpp
for (int i = 2; i < n; i = i * i)
    doWork();
```

**Solution.** i takes `2, 4, 16, 256, …` i.e. `2^(2^k)`. The loop stops when `2^(2^k) ≥ n`, so `2^k ≥ log₂ n`, so `k ≥ log₂ log₂ n`.

**Answer: Θ(log log n).**

---

## Part C — Insertion sort and merge sort

### Q12. Trace insertion sort on `[31, 41, 59, 26, 41, 58]` and count comparisons.

**Solution.**

| Pass j | key | Array after pass | Comparisons |
|---|---|---|---|
| 1 | 41 | [31, 41, 59, 26, 41, 58] | 1 (41 > 31, stop) |
| 2 | 59 | [31, 41, 59, 26, 41, 58] | 1 (59 > 41, stop) |
| 3 | 26 | [26, 31, 41, 59, 41, 58] | 3 (59, 41, 31 all > 26; loop ends at i<0) |
| 4 | 41 | [26, 31, 41, 41, 59, 58] | 2 (59 > 41; then 41 > 41 false, stop) |
| 5 | 58 | [26, 31, 41, 41, 58, 59] | 2 (59 > 58; then 41 > 58 false) |

**Total = 1+1+3+2+2 = 9 comparisons.** Final: `[26, 31, 41, 41, 58, 59]`.

Note pass 4: the test is `A[i] > key`, and `41 > 41` is false, so the new 41 stops **behind** the old one — **stability** in action.

---

### Q13. What input makes insertion sort run in Θ(n)? In Θ(n²)? Why is the average still Θ(n²)?

**Solution.**

- **Θ(n):** an already-sorted array. Each `while` test fails immediately -> `n − 1` comparisons, 0 shifts.
- **Θ(n²):** a reverse-sorted array. Each key travels to the front -> `n(n−1)/2` comparisons and shifts.
- **Average Θ(n²):** for a random permutation each key is expected to move about half of the sorted prefix, giving `≈ n²/4` operations — only a factor-2 improvement on the worst case, so the order of growth is unchanged.

---

### Q14. Insertion sort makes `n(n−1)/2` comparisons in the worst case. How many for n = 1000? Compare with merge sort.

**Solution.**

```
Insertion sort: 1000 × 999 / 2 = 499 500 comparisons
Merge sort:     ≈ n log₂ n = 1000 × 10 = 10 000 comparisons
```

**Merge sort does ~50× less work**, and the gap grows as n increases.

---

### Q15. Trace merge sort on `[5, 2, 4, 7, 1, 3, 2, 6]`.

**Solution.**

**Divide:**

```
[5,2,4,7,1,3,2,6]
[5,2,4,7]        [1,3,2,6]
[5,2]  [4,7]     [1,3]  [2,6]
[5][2] [4][7]    [1][3] [2][6]
```

**Merge upward:**

```
[5],[2]      -> [2,5]
[4],[7]      -> [4,7]
[2,5],[4,7]  -> [2,4,5,7]

[1],[3]      -> [1,3]
[2],[6]      -> [2,6]
[1,3],[2,6]  -> [1,2,3,6]

[2,4,5,7],[1,2,3,6] -> [1,2,2,3,4,5,6,7]
```

**Final: `[1, 2, 2, 3, 4, 5, 6, 7]`.**

---

### Q16. Why is merge sort Θ(n log n) even in the best case, while insertion sort is Θ(n)?

**Solution.** Merge sort's recursion structure is **independent of the data**: it always divides down to single elements (log n levels) and always merges back, touching all n elements per level. It never inspects whether the input is already ordered. Insertion sort's inner `while` loop is **data-dependent** and exits immediately on sorted input, giving Θ(n). Merge sort trades adaptivity for a guaranteed bound. (Timsort recovers adaptivity by detecting existing "runs".)

---

## Part D — Recurrences

### Q17. Solve `T(n) = T(n−1) + n`, `T(1) = 1`, by backward substitution.

**Solution.**

```
T(n) = T(n−1) + n = T(n−2) + (n−1) + n = … = T(1) + 2 + 3 + … + n
     = 1 + [n(n+1)/2 − 1] = n(n+1)/2 = Θ(n²)
```

---

### Q18. Solve `T(n) = 2T(n/2) + n²` using (a) the master theorem and (b) a recursion tree.

**Solution.**

*(a)* `a = 2, b = 2, log₂2 = 1`, watershed `n`. `f(n) = n² = Ω(n^(1+1))` ✔ with ε = 1.
Regularity: `2·(n/2)² = n²/2 ≤ (1/2)n²` -> c = 1/2 < 1 ✔. **Case 3 -> Θ(n²).**

*(b)* Level i has `2^i` nodes of size `n/2^i`, each costing `(n/2^i)² = n²/4^i`. Level total = `2^i · n²/4^i = n²/2^i`.

```
T(n) = Σ(i=0..log n) n²/2^i < n² · 2 = Θ(n²)
```

Decreasing geometric -> the **root dominates**. ✔ Same answer.

---

### Q19. Solve `T(n) = 3T(n/2) + n²`.

**Solution.** `a = 3, b = 2, log₂3 ≈ 1.585`, watershed `n^1.585`.
`f(n) = n² = Ω(n^(1.585 + 0.4))` ✔.
Regularity: `3(n/2)² = (3/4)n² ≤ c·n²` with c = 3/4 < 1 ✔.

**Case 3 -> `T(n) = Θ(n²)`.**

---

### Q20. Solve `T(n) = 2T(n/4) + √n`.

**Solution.** `a = 2, b = 4`, `log₄2 = 1/2`, watershed `n^0.5 = √n`.
`f(n) = √n = Θ(n^(log_b a))` -> **Case 2**.

**`T(n) = Θ(√n · log n)`.**

---

### Q21. Solve `T(n) = 2T(n/2) + n log n`. Explain why the master theorem fails.

**Solution.** Watershed `n^(log₂2) = n`. The ratio `f(n)/n^(log_b a) = log n` grows, so Case 2 fails; but `log n` is smaller than every `n^ε`, so `f(n) ≠ Ω(n^(1+ε))` for any ε > 0 and Case 3 fails too. **The standard master theorem is silent.**

*Recursion tree:* level i has `2^i` nodes of size `n/2^i`, so the level total is

```
2^i · (n/2^i) log(n/2^i) = n(log n − i)
```

```
T(n) = Σ(i=0..log n) n(log n − i) = n Σ(k=0..log n) k = n·(log n)(log n + 1)/2 = Θ(n log² n)
```

*Extended master theorem:* `k = 1, p = 1`, `log_b a = 1 = k`, `p > −1` -> `Θ(n log² n)` ✔

---

### Q22. Solve `T(n) = T(n/2) + T(n/4) + n`.

**Solution.** Not of master-theorem form (unequal subproblem sizes). Use a tree.

| Level | Level cost |
|---|---|
| 0 | n |
| 1 | n/2 + n/4 = (3/4)n |
| 2 | (3/4)²n |
| i | (3/4)^i n |

```
T(n) ≤ n Σ(i=0..∞) (3/4)^i = n · 1/(1 − 3/4) = 4n = O(n)
```

And `T(n) ≥ n`, so **T(n) = Θ(n)**.

*Verify by substitution:* guess `T(n) ≤ cn`. Then `T(n) ≤ c(n/2) + c(n/4) + n = (3c/4)n + n ≤ cn` provided `c/4 ≥ 1`, i.e. **c ≥ 4** ✔ ∎

---

### Q23. Use substitution to prove `T(n) = 2T(⌊n/2⌋) + n` is `O(n log n)`.

**Solution.** Guess `T(n) ≤ cn log₂ n`.

```
T(n) ≤ 2c⌊n/2⌋log₂⌊n/2⌋ + n
     ≤ 2c(n/2)log₂(n/2) + n
     = cn log₂ n − cn + n
     ≤ cn log₂ n           provided c ≥ 1
```

Base: at n = 2, `T(2) = 2T(1) + 2 = 2c₁ + 2 ≤ 2c` requires `c ≥ c₁ + 1`. Choose `c = max(1, c₁+1)`, `n₀ = 2`. ∎

---

### Q24. Show that guessing `T(n) ≤ cn` fails for `T(n) = 2T(n/2) + 1`, then fix it.

**Solution.**

*Failed attempt:* `T(n) ≤ 2c(n/2) + 1 = cn + 1`, which is **not** `≤ cn`. The constant grew — the induction does not close.

*Fix — subtract a lower-order term.* Guess `T(n) ≤ cn − d`:

```
T(n) ≤ 2[c(n/2) − d] + 1 = cn − 2d + 1 ≤ cn − d   ⟺  d ≥ 1
```

Choose d = 1. **T(n) = O(n).** ∎

---

## Part E — Strassen and order statistics

### Q25. State Strassen's recurrence and solve it. How many multiplications does it save?

**Solution.**

```
T(n) = 7T(n/2) + Θ(n²)
```

`a = 7, b = 2`, `log₂7 ≈ 2.807`, watershed `n^2.807`. `f(n) = n² = O(n^(2.807 − 0.8))` -> **Case 1**:

```
T(n) = Θ(n^log₂7) = Θ(n^2.807)
```

Standard block D&C uses **8** multiplications and gives Θ(n³). Strassen uses **7**, saving one multiplication per level at the cost of extra additions (18 instead of 4). Because the saving compounds across log n levels, the exponent drops from 3 to 2.807.

---

### Q26. Multiply `A = [[2,3],[4,1]]` and `B = [[1,5],[2,6]]` using Strassen.

**Solution.** A11=2, A12=3, A21=4, A22=1; B11=1, B12=5, B21=2, B22=6.

```
M1 = (A11+A22)(B11+B22) = (2+1)(1+6) = 3×7  = 21
M2 = (A21+A22)·B11      = (4+1)×1    = 5×1  =  5
M3 = A11·(B12−B22)      = 2×(5−6)    = 2×(−1) = −2
M4 = A22·(B21−B11)      = 1×(2−1)    = 1×1  =  1
M5 = (A11+A12)·B22      = (2+3)×6    = 5×6  = 30
M6 = (A21−A11)(B11+B12) = (4−2)(1+5) = 2×6  = 12
M7 = (A12−A22)(B21+B22) = (3−1)(2+6) = 2×8  = 16
```

```
C11 = M1 + M4 − M5 + M7 = 21 + 1 − 30 + 16 = 8
C12 = M3 + M5           = −2 + 30          = 28
C21 = M2 + M4           = 5 + 1            = 6
C22 = M1 − M2 + M3 + M6 = 21 − 5 − 2 + 12  = 26
```

```
C = ⎡  8  28 ⎤
    ⎣  6  26 ⎦
```

**Check by the standard method:**
C11 = 2·1 + 3·2 = 8 ✔, C12 = 2·5 + 3·6 = 28 ✔, C21 = 4·1 + 1·2 = 6 ✔, C22 = 4·5 + 1·6 = 26 ✔

---

### Q27. Find the 4th smallest element of `A = [12, 3, 5, 7, 19, 26, 8]` using quick select. Show every partition.

**Solution.** n = 7, k = 4 -> **target index = 3**. (Sorted: `[3,5,7,8,12,19,26]`, so the answer should be **8**.)

**Partition 1:** lo = 0, hi = 6, pivot = `A[6]` = **8**, i starts at −1.

| j | A[j] | ≤ 8? | Action | Array |
|---|---|---|---|---|
| 0 | 12 | no | — | [12,3,5,7,19,26,8] |
| 1 | 3 | yes | i=0, swap A[0],A[1] | [3,12,5,7,19,26,8] |
| 2 | 5 | yes | i=1, swap A[1],A[2] | [3,5,12,7,19,26,8] |
| 3 | 7 | yes | i=2, swap A[2],A[3] | [3,5,7,12,19,26,8] |
| 4 | 19 | no | — | — |
| 5 | 26 | no | — | — |

Final swap `A[i+1]=A[3]` ↔ `A[6]`: `[3,5,7,8,19,26,12]`, **p = 3**.

`p == target = 3` -> **return A[3] = 8** ✔

Found in a **single partition**, 6 comparisons — versus the ~16 a full sort would need.

---

### Q28. Explain why quick select is Θ(n) on average but quick sort is Θ(n log n), even though both use the same partition.

**Solution.** After partitioning, quick **sort** must sort *both* halves:

```
T(n) = 2T(n/2) + Θ(n)     -> every level costs n, and there are log n levels -> Θ(n log n)
```

Quick **select** knows which half contains the answer and discards the other:

```
T(n) = 1T(n/2) + Θ(n)     -> level costs n, n/2, n/4, … -> a decreasing geometric series -> Θ(n)
```

The discarded half is never touched again. The total `n + n/2 + n/4 + … < 2n` is linear, so the `log n` factor disappears.

---

### Q29. Give the worst-case input for quick select with last-element pivot, and the resulting complexity.

**Solution.** A **sorted** (or reverse-sorted) array, searching for an element at the far end — e.g. `A = [1,2,3,4,5]` searching for the minimum. Each partition places the pivot at an end and removes exactly one element:

```
T(n) = T(n−1) + Θ(n) = n + (n−1) + … + 1 = n(n+1)/2 = Θ(n²)
```

**Fix:** randomise the pivot, or use median-of-three, or median-of-medians for a hard O(n) guarantee.

---

### Q30. Find the 2nd largest of `[3, 2, 3, 1, 2, 4, 5, 5, 6]` and state the complexity of each approach.

**Solution.** n = 9, k = 2 -> target index = `n − k = 7`. Sorted: `[1,2,2,3,3,4,5,5,6]`, index 7 = **5**.

| Approach | Complexity | Result |
|---|---|---|
| Sort descending, take A[1] | O(n log n) | `[6,5,5,4,3,3,2,2,1]` -> **5** |
| Min-heap of size 2 | O(n log 2) = O(n) | heap ends as {5, 6}, root = **5** |
| Quick select to index 7 | O(n) average | **5** |

Note that duplicates are handled naturally — the 2nd largest **value in sorted position** is 5, not 6. If the question asked for the 2nd *distinct* largest, the answer would also be 5 here, but in general you must deduplicate first.

---

## Part F — MCQ rapid fire (with answers)

| # | Question | Answer |
|---|---|---|
| 1 | Worst case of insertion sort | **Θ(n²)** |
| 2 | Best case of insertion sort | **Θ(n)** — already sorted |
| 3 | Worst case of merge sort | **Θ(n log n)** |
| 4 | Auxiliary space of merge sort on arrays | **Θ(n)** |
| 5 | Is merge sort stable? | **Yes** (because of `L[i] <= R[j]`) |
| 6 | Is insertion sort stable? | **Yes** (because of the strict `>`) |
| 7 | `T(n)=2T(n/2)+n` solves to | **Θ(n log n)** |
| 8 | `T(n)=T(n/2)+n` solves to | **Θ(n)** |
| 9 | `T(n)=2T(n/2)+1` solves to | **Θ(n)** |
| 10 | `T(n)=T(n−1)+1` solves to | **Θ(n)** |
| 11 | Strassen's complexity | **Θ(n^2.807)** = Θ(n^log₂7) |
| 12 | Multiplications in Strassen per level | **7** |
| 13 | Multiplications in naive block D&C | **8** |
| 14 | Quick select average case | **Θ(n)** |
| 15 | Quick select worst case | **Θ(n²)** |
| 16 | Worst-case-linear selection algorithm | **Median of medians (BFPRT)** |
| 17 | Why groups of 5 in median of medians? | 1/5 + 7/10 < 1, so the series converges |
| 18 | k-th largest equals which order statistic? | **(n − k + 1)-th smallest** |
| 19 | `f = Θ(g)` iff | `f = O(g)` **and** `f = Ω(g)` |
| 20 | `log(n!)` is | **Θ(n log n)** (Stirling) |
| 21 | Master theorem applies to | `T(n) = aT(n/b) + f(n)`, a ≥ 1, b > 1 constants |
| 22 | `T(n)=2T(n/2)+n log n` | **Θ(n log² n)** (master theorem is silent) |
| 23 | Which notation is a strict upper bound? | **little-o** |
| 24 | Sorting lower bound (comparison based) | **Ω(n log n)** |
| 25 | Recursive binary search space | **Θ(log n)** stack |

---

## Part G — Common mistakes to avoid

| Mistake | Correction |
|---|---|
| "Big-O = worst case" | O/Ω/Θ bound *functions*; best/worst/average choose *which* function. Independent axes. |
| Saying `Σ(i=1..n) i = Θ(n)` | It is `n(n+1)/2 = Θ(n²)`. |
| Calling a triangular double loop Θ(n) | Half of n² is still Θ(n²). |
| Applying the master theorem to `2T(n/2) + n log n` | It does not apply — use a tree or the extended form -> Θ(n log² n). |
| Forgetting the regularity condition in Case 3 | Always check `a·f(n/b) ≤ c·f(n)` with c < 1. |
| Letting the constant grow in substitution | The induction must land back on the **same** c, or the proof is void. |
| Ignoring recursion-stack space | Recursive binary search is Θ(log n) space, not Θ(1). |
| Saying `2^(2n) = O(2ⁿ)` | False. `2^(n+1) = O(2ⁿ)` is true; multiplying the exponent is not a constant factor. |
| Writing Strassen's M-terms in the wrong order | Matrix multiplication is not commutative. |
| Believing quick select is O(n log n) | It is **O(n)** average, because only **one** side is recursed into. |
| Believing quick sort's worst case is O(n log n) | It is **O(n²)**; merge sort is the one with the guarantee. |

---

# 18. One-Page Revision Sheet

## Asymptotic definitions

| Notation | Definition | Means |
|---|---|---|
| `f = O(g)` | ∃ c, n₀ > 0 : `f(n) ≤ c·g(n)` ∀ n ≥ n₀ | ≤ |
| `f = Ω(g)` | ∃ c, n₀ > 0 : `f(n) ≥ c·g(n)` ∀ n ≥ n₀ | ≥ |
| `f = Θ(g)` | ∃ c₁,c₂,n₀ > 0 : `c₁g(n) ≤ f(n) ≤ c₂g(n)` ∀ n ≥ n₀ | = |
| `f = o(g)` | ∀ c > 0 ∃ n₀ : `f(n) < c·g(n)` ∀ n ≥ n₀ | < |
| `f = ω(g)` | ∀ c > 0 ∃ n₀ : `f(n) > c·g(n)` ∀ n ≥ n₀ | > |

**Limit test:** `L = lim f/g`. `L = 0` -> o; `0 < L < ∞` -> Θ; `L = ∞` -> ω.

## Growth hierarchy

```
1 < log log n < log n < √n < n < n log n < n² < n³ < 2ⁿ < 3ⁿ < n! < nⁿ
```

## Summations

```
Σ(i=1..n) 1     = n
Σ(i=1..n) i     = n(n+1)/2       = Θ(n²)
Σ(i=1..n) i²    = n(n+1)(2n+1)/6 = Θ(n³)
Σ(i=0..n) r^i   = (r^(n+1)−1)/(r−1);  r<1 -> Θ(1);  r>1 -> Θ(r^n)
Σ(i=1..n) 1/i   = H(n) = Θ(log n)
log(n!)         = Θ(n log n)
```

## Loop-shape cheatsheet

| Loop | Iterations |
|---|---|
| `i++` to n | Θ(n) |
| `i *= 2` to n | Θ(log n) |
| `i = i*i` from 2 to n | Θ(log log n) |
| `i*i < n` | Θ(√n) |
| nested to n, n | Θ(n²) |
| inner bounded by outer | Θ(n²) |
| `j += i` inside `i` loop | Θ(n log n) — harmonic |

## Master theorem — `T(n) = aT(n/b) + f(n)`

Compare `f(n)` with `n^(log_b a)`:

| Case | Condition | Result |
|---|---|---|
| 1 | `f(n) = O(n^(log_b a − ε))` | `Θ(n^(log_b a))` |
| 2 | `f(n) = Θ(n^(log_b a))` | `Θ(n^(log_b a) log n)` |
| 3 | `f(n) = Ω(n^(log_b a + ε))` **and** `a f(n/b) ≤ c f(n)`, c < 1 | `Θ(f(n))` |

**Extended form** for `f(n) = Θ(n^k log^p n)`:
`log_b a > k` -> `Θ(n^(log_b a))` · `log_b a = k, p > −1` -> `Θ(n^k log^(p+1) n)` · `log_b a < k` -> `Θ(n^k log^p n)`

**Subtract-and-conquer** `T(n) = aT(n−b) + O(n^k)`:
`a < 1` -> `O(n^k)` · `a = 1` -> `O(n^(k+1))` · `a > 1` -> `O(n^k a^(n/b))`

## Recurrences to know by heart

| Recurrence | Answer |
|---|---|
| `T(n−1) + 1` | Θ(n) |
| `T(n−1) + n` | Θ(n²) |
| `2T(n−1) + 1` | Θ(2ⁿ) |
| `T(n/2) + 1` | Θ(log n) |
| `T(n/2) + n` | **Θ(n)** — quick select |
| `2T(n/2) + 1` | Θ(n) |
| `2T(n/2) + n` | **Θ(n log n)** — merge sort |
| `2T(n/2) + n log n` | Θ(n log² n) |
| `2T(n/2) + n²` | Θ(n²) |
| `7T(n/2) + n²` | **Θ(n^2.807)** — Strassen |
| `8T(n/2) + n²` | Θ(n³) — naive block D&C |
| `T(n/3) + T(2n/3) + n` | Θ(n log n) |

## The four algorithms of Unit 1

| Algorithm | Recurrence | Best | Avg | Worst | Space | Stable |
|---|---|---|---|---|---|---|
| **Insertion sort** | iterative | Θ(n) | Θ(n²) | Θ(n²) | Θ(1) | ✔ |
| **Merge sort** | `2T(n/2)+n` | Θ(n log n) | Θ(n log n) | Θ(n log n) | Θ(n) | ✔ |
| **Strassen** | `7T(n/2)+n²` | — | — | Θ(n^2.807) | Θ(n²) | — |
| **Quick select** | `T(n/2)+n` | Θ(n) | Θ(n) | Θ(n²) | Θ(1) | — |

## Strassen's seven products (write these from memory)

```
M1 = (A11+A22)(B11+B22)        C11 = M1 + M4 − M5 + M7
M2 = (A21+A22)·B11             C12 = M3 + M5
M3 = A11·(B12−B22)             C21 = M2 + M4
M4 = A22·(B21−B11)             C22 = M1 − M2 + M3 + M6
M5 = (A11+A12)·B22
M6 = (A21−A11)(B11+B12)
M7 = (A12−A22)(B21+B22)
```

## Order statistics

```
k-th largest = (n − k + 1)-th smallest
0-indexed:  k-th smallest -> index k−1      k-th largest -> index n−k
```

Quick select: partition once, recurse into **one** side only.
Average Θ(n) · Worst Θ(n²) · Randomised pivot fixes the worst case in expectation · Median of medians gives worst-case Θ(n).

---

## Further reading

- **Levitin**, *Introduction to the Design and Analysis of Algorithms* — Chapters 2 (Fundamentals of the Analysis of Algorithm Efficiency), 4 and 5 (Divide and Conquer).
- **CLRS**, *Introduction to Algorithms* — Chapters 2 (Getting Started), 3 (Growth of Functions), 4 (Divide-and-Conquer and the Master Theorem), 9 (Medians and Order Statistics), 28.2 (Strassen).
- [VisuAlgo](https://visualgo.net/en) — animated sorting and selection.
- [CP-Algorithms](https://cp-algorithms.com/) — proofs and implementations.
- [GeeksforGeeks DSA](https://www.geeksforgeeks.org/dsa/) — worked examples and practice problems.

---

*Unit 1 reading material — CSE 408 Design and Analysis of Algorithms.*
