# Unit 2 — String Matching Algorithms and Computational Geometry

**Course:** CSE 408 — Design and Analysis of Algorithms
**Textbook:** *Introduction to the Design and Analysis of Algorithms* — Anany Levitin
**Reference:** *Introduction to Algorithms* — Cormen, Leiserson, Rivest, Stein (CLRS)

---

## Syllabus covered in this file

> **String Matching Algorithms and Computational Geometry:** Sequential Search and Brute-Force String Matching, Naive Pattern Matching, Rabin-Karp Algorithm, Knuth-Morris-Pratt Algorithm, Data Structures for String Processing: Trie (Prefix Tree), Computational Geometry: Closest-Pair Problem, Convex Hull.

---

## Table of Contents

1. [What is String Matching? — Definitions and Notation](#1-what-is-string-matching--definitions-and-notation)
2. [The Brute-Force Paradigm and Sequential Search](#2-the-brute-force-paradigm-and-sequential-search)
3. [Brute-Force String Matching / Naive Pattern Matching — Complete Analysis](#3-brute-force-string-matching--naive-pattern-matching--complete-analysis)
4. [Rabin-Karp Algorithm — String Matching with Hashing](#4-rabin-karp-algorithm--string-matching-with-hashing)
5. [Knuth-Morris-Pratt (KMP) Algorithm](#5-knuth-morris-pratt-kmp-algorithm)
6. [Comparing the Three String-Matching Algorithms](#6-comparing-the-three-string-matching-algorithms)
7. [Trie (Prefix Tree) — Data Structure for String Processing](#7-trie-prefix-tree--data-structure-for-string-processing)
8. [Computational Geometry — Basic Tools](#8-computational-geometry--basic-tools)
9. [Closest-Pair Problem](#9-closest-pair-problem)
10. [Convex Hull — Graham Scan](#10-convex-hull--graham-scan)
11. [Practice Questions — Unit 2 (Full Solutions)](#11-practice-questions--unit-2-full-solutions)
12. [Combined Practice Questions — Units 1 & 2](#12-combined-practice-questions--units-1--2)
13. [One-Page Revision Sheet — Unit 2](#13-one-page-revision-sheet--unit-2)

---

# 1. What is String Matching? — Definitions and Notation

> **String matching (pattern matching)** is the problem of finding all occurrences of a short string, the **pattern** `P`, inside a longer string, the **text** `T`.

### Notation used throughout this unit

| Symbol | Meaning |
|---|---|
| `T` | the text, length `n = |T|` |
| `P` | the pattern, length `m = |P|`, and always `m ≤ n` |
| `Σ` | the alphabet the characters are drawn from (e.g. `{A,C,G,T}` for DNA, ASCII, or just `{0,1}`) |
| **shift `s`** | an alignment of `P` against `T` starting at text index `s` (0-indexed: `T[s..s+m-1]`) |
| **valid shift** | a shift `s` for which `T[s..s+m-1] = P` character-for-character — i.e. an actual occurrence |

**Goal:** find every valid shift `s` (0 ≤ s ≤ n − m), or just the first one, depending on the problem.

### Why this matters (applications)

| Application | What is text, what is pattern |
|---|---|
| Text editor "Find" (Ctrl+F) | document = text, search box = pattern |
| Plagiarism / similarity detection | document corpus = text, suspect passage = pattern |
| DNA / genome analysis | genome sequence = text, gene motif = pattern |
| Antivirus scanners | file being scanned = text, virus signature = pattern |
| Search engines, `grep` | indexed documents = text, query = pattern |
| Compilers (lexical analysis) | source code = text, keywords/tokens = pattern |
| Network intrusion detection | packet stream = text, attack signature = pattern |

### The four algorithms of this unit, at a glance

| Algorithm | Idea | Time |
|---|---|---|
| **Brute-force / Naive** | Try every alignment, compare character by character | O(nm) worst case |
| **Rabin-Karp** | Hash each window; compare hashes first, characters only on a hash match | O(n+m) average, O(nm) worst |
| **Knuth-Morris-Pratt (KMP)** | Precompute pattern self-overlap so the text pointer never backtracks | O(n+m) **always** |
| **Trie** | Not a matching algorithm by itself — a tree that stores many patterns for fast prefix/multi-pattern lookup | O(L) per word, L = word length |

---

# 2. The Brute-Force Paradigm and Sequential Search

## 2.1 What "brute force" means as a design technique

> **Brute force** is a straightforward approach to solving a problem, usually directly based on the problem's *statement* and the *definitions* of the concepts involved, rather than on any clever insight into the problem's structure.

It is the first strategy you should always be able to write down — even if you then replace it with something faster. Several Unit 2 algorithms and *all* of Unit 1's simplest algorithms are brute-force:

| Algorithm | Brute-force because… |
|---|---|
| Sequential (linear) search | checks every element, exactly as "search" is defined |
| Selection sort / bubble sort | directly implements "repeatedly find/swap the extreme element" |
| Brute-force string matching | tries every possible alignment of P against T |
| Closest-pair by definition | checks every pair of points and keeps the minimum distance |
| Convex hull by definition | tests every pair of points as a candidate hull edge |

## 2.2 Sequential search — the baseline

```cpp
int sequentialSearch(const vector<int>& A, int key) {
    for (int i = 0; i < (int)A.size(); ++i)
        if (A[i] == key) return i;   // BASIC OPERATION
    return -1;
}
```

This is the Unit 1 "linear search" revisited. Its relevance here is **structural**: brute-force string matching is nothing but sequential search generalised from "does one element equal the key?" to "does a whole substring equal the pattern?". Every alignment `s` is like one "slot" being tested; the test itself now costs up to `m` comparisons instead of 1.

| | Sequential search | Brute-force string matching |
|---|---|---|
| What is tested at each position | one element `== key`? | `m` characters `== P`? |
| Number of positions tested | n | n − m + 1 |
| Cost per test | O(1) | O(m) |
| Total worst case | **O(n)** | **O(nm)** |

---

# 3. Brute-Force String Matching / Naive Pattern Matching — Complete Analysis

> ⚠️ **Same algorithm, two names.** Levitin's textbook calls this **brute-force string matching**; CLRS calls the identical algorithm the **naive string matcher**. Your syllabus lists both names because different sources use different terms — do not treat them as two separate algorithms in an exam.

## 3.1 The idea

Slide the pattern over the text one position at a time. At each position (shift), compare characters left to right until either a mismatch occurs or the whole pattern matches.

## 3.2 Pseudocode (CLRS style, 1-indexed)

```
NAIVE-STRING-MATCHER(T, P)
1  n = T.length
2  m = P.length
3  for s = 0 to n - m
4      if P[1..m] == T[s+1 .. s+m]
5          print "Pattern occurs with shift" s
```

## 3.3 C++ implementation (Practical 5)

```cpp
#include <vector>
#include <string>
using namespace std;

vector<int> naiveStringMatch(const string& text, const string& pattern) {
    vector<int> occurrences;
    int n = text.size(), m = pattern.size();
    for (int s = 0; s <= n - m; ++s) {       // try every shift
        int j = 0;
        while (j < m && text[s + j] == pattern[j])  // BASIC OPERATION
            ++j;
        if (j == m)                          // fell through — full match
            occurrences.push_back(s);
    }
    return occurrences;
}
```

## 3.4 Worked trace — best-ish case with a real match

`T = "ABABDABACDABABCABAB"` (n = 20), `P = "ABABCABAB"` (m = 9).

Most shifts fail within the first 1–2 comparisons because the second character of `T` at that offset rarely matches `P[1]`. The genuine match occurs at shift **10**:

```
T = ABABDABACD ABABCABAB
                ^ shift 10, T[10..18] = "ABABCABAB" = P   ✔
```

## 3.5 Worst-case analysis — the example that matters for exams

**Worst case: `T` = all the same character, `P` = that character repeated with one different character at the very end.**

`T = "AAAAAAAAAA"` (n = 10, ten A's), `P = "AAAAB"` (m = 5).

Pattern never occurs, and every single shift is forced to compare **all 5 characters** before the mismatch is discovered on the last one:

| shift s | comparisons before mismatch | where it fails |
|---|---|---|
| 0 | 5 | `P[4]='B'` vs `T[4]='A'` |
| 1 | 5 | `P[4]='B'` vs `T[5]='A'` |
| 2 | 5 | … |
| 3 | 5 | … |
| 4 | 5 | … |
| 5 | 5 | `P[4]='B'` vs `T[9]='A'` |

n − m + 1 = 6 shifts, 5 comparisons each -> **30 = m(n − m + 1) comparisons**.

```
C_worst(n, m) = m(n − m + 1) = Θ(nm)
```

When `m` is a fixed fraction of `n` (e.g. m ≈ n/2), this becomes **Θ(n²)** — quadratic, same order as insertion sort's worst case.

## 3.6 Best case

The very first character mismatches at almost every shift (typical of "random" text over a large alphabet):

```
C_best(n, m) = n − m + 1 = Θ(n)
```

## 3.7 Complexity summary

| Measure | Best case | Worst case |
|---|---|---|
| Time | **Θ(n)** | **Θ(nm)** |
| Space (auxiliary) | Θ(1) | Θ(1) |

**Why it's still taught:** it needs **zero preprocessing** and **zero extra memory**, and for typical English-text searches with a large alphabet the worst case almost never triggers — average behaviour is close to Θ(n). It only becomes genuinely bad on **highly repetitive** alphabets (DNA's 4 letters, bitstrings) — exactly where Rabin-Karp and KMP earn their keep.

---

# 4. Rabin-Karp Algorithm — String Matching with Hashing

## 4.1 The idea

Comparing `m` characters at every one of the `n − m + 1` shifts is wasteful when most shifts are hopeless. **Rabin-Karp** first computes a single number — a **hash** — for the pattern and for every length-`m` window of the text, and compares those numbers instead of the full strings. Two equal-length strings that are actually equal *must* hash to the same value, so:

- **hash(window) ≠ hash(P)** -> definitely not a match, skip immediately (no character comparison needed).
- **hash(window) = hash(P)** -> *possibly* a match — a **candidate**. Must verify character-by-character, because different strings can collide to the same hash (a **spurious hit**).

The trick that makes this fast is computing every window's hash **incrementally** in O(1) from the previous window's hash — a **rolling hash** — instead of recomputing it from scratch each time.

## 4.2 The polynomial (Horner's rule) hash

Treat each `m`-character window as an `m`-digit number in base `d = |Σ|` (e.g. d = 256 for bytes, or d = 10 for the toy decimal examples below), then reduce it modulo a prime `q` so the numbers stay small:

```
t_s = ( T[s]·d^(m-1) + T[s+1]·d^(m-2) + … + T[s+m-1]·d^0 )  mod q
p   = ( P[0]·d^(m-1) + P[1]·d^(m-2) + … + P[m-1]·d^0 )      mod q
```

## 4.3 The rolling update — O(1) per shift

To go from window `s` to window `s+1` we drop the leading digit `T[s]` and append the new trailing digit `T[s+m]`:

```
h = d^(m-1) mod q                                    (precompute once)

t_{s+1} = ( d · ( t_s − T[s]·h ) + T[s+m] )  mod q
```

This is a **constant number of arithmetic operations**, regardless of `m` — the same idea as a sliding window.

## 4.4 Pseudocode (CLRS)

```
RABIN-KARP-MATCHER(T, P, d, q)
1  n = T.length ; m = P.length
2  h = d^(m-1) mod q
3  p = 0 ; t0 = 0
4  for i = 1 to m                       // preprocessing: hash P and first window
5      p  = (d·p  + P[i])  mod q
6      t0 = (d·t0 + T[i])  mod q
7  for s = 0 to n - m
8      if p == t_s
9          if P[1..m] == T[s+1..s+m]    // verify — guard against spurious hits
10             print "valid shift" s
11     if s < n - m
12         t_{s+1} = (d·(t_s − T[s+1]·h) + T[s+m+1]) mod q
```

## 4.5 C++ implementation (Practical 6)

```cpp
#include <vector>
#include <string>
using namespace std;

vector<int> rabinKarp(const string& text, const string& pattern,
                       int d = 256, int q = 101) {
    vector<int> occurrences;
    int n = text.size(), m = pattern.size();
    if (m > n) return occurrences;

    long long h = 1;                       // h = d^(m-1) mod q
    for (int i = 0; i < m - 1; ++i) h = (h * d) % q;

    long long p = 0, t = 0;                // hash of pattern, hash of current window
    for (int i = 0; i < m; ++i) {
        p = (d * p + pattern[i]) % q;
        t = (d * t + text[i]) % q;
    }

    for (int s = 0; s <= n - m; ++s) {
        if (p == t) {                       // hash matches — verify to rule out spurious hit
            if (text.compare(s, m, pattern) == 0)
                occurrences.push_back(s);
        }
        if (s < n - m) {                    // roll the hash forward
            t = (d * (t - text[s] * h) + text[s + m]) % q;
            if (t < 0) t += q;               // C++ % can return negative
        }
    }
    return occurrences;
}
```

> ⚠️ **The `if (t < 0) t += q;` line is not optional.** C++'s `%` can return a negative remainder when the left operand is negative (unlike Python's `%`). Forgetting this is the single most common Rabin-Karp bug.

## 4.6 Fully worked, hand-verified trace

`d = 10`, `q = 11`, `P = "42"` (m = 2), `T = "204220"` (n = 6).

**Pattern hash:** `p = 42 mod 11 = 9`.

**Rolling multiplier:** `h = d^(m-1) mod q = 10^1 mod 11 = 10`.

| s | window | direct value mod 11 | rolling formula check | hash match (=9)? | verify chars | result |
|---|---|---:|---|:--:|---|---|
| 0 | "20" | 9 | `t0 = 20 mod 11 = 9` | ✔ | "20" ≠ "42" | **spurious hit** |
| 1 | "04" | 4 | `t1=(10·(9−2·10)+4) mod11 = 4` | ✘ | — | skip, no comparison |
| 2 | "42" | 9 | `t2=(10·(4−0·10)+2) mod11 = 9` | ✔ | "42" = "42" | **real match at shift 2** |
| 3 | "22" | 0 | `t3=(10·(9−4·10)+2) mod11 = 0` | ✘ | — | skip |
| 4 | "20" | 9 | `t4=(10·(0−2·10)+0) mod11 = 9` | ✔ | "20" ≠ "42" | **spurious hit** |

Every rolling-formula value matches the value obtained by hashing the window directly — confirming the O(1) update is correct. The trace shows **both** essential behaviours in one small example: a genuine match (shift 2) and two spurious hits (shifts 0 and 4) that are correctly rejected only *after* the O(m) character check — this is precisely why line 9 of the pseudocode can never be skipped.

## 4.7 Choosing `q`

- `q` should be a **prime** roughly the same size as the machine word, so that `d·q` still fits without overflow (CLRS uses primes like 101, or picks `q` so `d·q < 2^31`).
- A **larger prime** -> fewer collisions -> fewer spurious hits -> closer to the average-case running time.
- A **poorly chosen `q`** (e.g. one that divides `d^k − d^j` for many pairs, or simply a small `q`) can make *every* shift a spurious hit, degrading Rabin-Karp to the naive algorithm's worst case.

## 4.8 Complexity

| Case | Time | Why |
|---|---|---|
| Preprocessing | Θ(m) | one pass to hash `P` and the first window |
| Average / practical | **Θ(n + m)** | few or no spurious hits; each shift does O(1) hash work, verification is rare |
| Worst case | **Θ((n−m+1)·m) = Θ(nm)** | if (by bad luck or an adversarial input) every shift is a spurious hit, we still pay the full O(m) verification every time |

**Rabin-Karp's real strength** is not a single pattern search (KMP already guarantees O(n+m) worst case for that) — it is **multi-pattern search**: hashing lets you check a text window against *thousands* of pattern hashes in a hash set in O(1) expected time per window, which is exactly how plagiarism detectors and many antivirus scanners work.

---

# 5. Knuth-Morris-Pratt (KMP) Algorithm

## 5.1 The key insight naive matching wastes

When a naive matcher fails partway through a comparison, it discards **all** the information gained from the characters that *did* match and restarts the text pointer one position to the right. KMP's insight: **the pattern already tells you, in advance, how far you can safely shift** without re-examining text characters you have already looked at — because you know exactly which prefix of `P` just matched.

> **Guarantee:** the text pointer `i` **never moves backward**. Every character of `T` is examined a bounded number of times, giving a hard **O(n + m)** bound with *no* worst case blow-up, unlike naive matching or Rabin-Karp.

## 5.2 The failure function / LPS array

For a pattern `P` of length `m`, define:

> `lps[i]` = the length of the **longest proper prefix** of `P[0..i]` that is **also a suffix** of `P[0..i]`.

("proper" = not the whole substring itself.) This single array is precomputed once from the pattern, in O(m) time, and drives all of the matching phase's shifting decisions.

### Worked example — LPS for `P = "AABAAC"` (m = 6)

| i | P[i] | Longest proper prefix that's also a suffix of P[0..i] | lps[i] |
|---|---|---|---|
| 0 | A | — (single char has no proper prefix) | 0 |
| 1 | A | "A" | 1 |
| 2 | B | none ("AAB" — no proper prefix matches a suffix) | 0 |
| 3 | A | "A" | 1 |
| 4 | A | "AA" | 2 |
| 5 | C | none | 0 |

**`lps = [0, 1, 0, 1, 2, 0]`**

### Construction algorithm

```
COMPUTE-LPS(P, m)
1  lps[0] = 0
2  len = 0                 // length of the previous longest prefix-suffix
3  i = 1
4  while i < m
5      if P[i] == P[len]
6          len = len + 1
7          lps[i] = len
8          i = i + 1
9      else if len != 0
10         len = lps[len - 1]      // fall back — do NOT advance i
11     else
12         lps[i] = 0
13         i = i + 1
```

**C++:**

```cpp
vector<int> computeLPS(const string& pattern) {
    int m = pattern.size();
    vector<int> lps(m, 0);
    int len = 0, i = 1;
    while (i < m) {
        if (pattern[i] == pattern[len]) {
            lps[i++] = ++len;
        } else if (len != 0) {
            len = lps[len - 1];      // fall back using lps itself — i does not move
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

**Trace of `lps` construction for `"AABAAC"` step by step:**

```
lps[0] = 0 ,  len = 0,  i = 1
i=1: P[1]='A', P[len]=P[0]='A' -> match, len=1, lps[1]=1, i=2
i=2: P[2]='B', P[len]=P[1]='A' -> mismatch, len!=0 -> len = lps[0] = 0
     P[2]='B', P[len]=P[0]='A' -> mismatch, len==0 -> lps[2]=0, i=3
i=3: P[3]='A', P[len]=P[0]='A' -> match, len=1, lps[3]=1, i=4
i=4: P[4]='A', P[len]=P[1]='A' -> match, len=2, lps[4]=2, i=5
i=5: P[5]='C', P[len]=P[2]='B' -> mismatch, len!=0 -> len = lps[1] = 1
     P[5]='C', P[len]=P[1]='A' -> mismatch, len!=0 -> len = lps[0] = 0
     P[5]='C', P[len]=P[0]='A' -> mismatch, len==0 -> lps[5]=0, i=6, loop ends
```

Result: `[0,1,0,1,2,0]` — matches the table above. ✔

## 5.3 The matching phase

```
KMP-MATCHER(T, P)
1  n = T.length ; m = P.length
2  lps = COMPUTE-LPS(P, m)
3  i = 0 ; j = 0                     // i scans T, j scans P
4  while i < n
5      if T[i] == P[j]
6          i++ ; j++
7          if j == m
8              print "match at shift" (i - j)
9              j = lps[j - 1]        // look for the next overlapping match
10     else if j != 0
11         j = lps[j - 1]            // fall back in P — i does NOT move
12     else
13         i++                       // no partial match to salvage — advance i
```

**C++ implementation (Practical 7):**

```cpp
#include <vector>
#include <string>
using namespace std;

vector<int> kmpSearch(const string& text, const string& pattern) {
    int n = text.size(), m = pattern.size();
    vector<int> occurrences;
    if (m == 0) return occurrences;
    vector<int> lps = computeLPS(pattern);

    int i = 0, j = 0;
    while (i < n) {
        if (text[i] == pattern[j]) {
            ++i; ++j;
            if (j == m) {
                occurrences.push_back(i - j);
                j = lps[j - 1];
            }
        } else if (j != 0) {
            j = lps[j - 1];          // KEY STEP: i never moves here
        } else {
            ++i;
        }
    }
    return occurrences;
}
```

## 5.4 Worked trace 1 — clean run, no fallback needed

`T = "XXAABAACYY"` (n = 10), `P = "AABAAC"` (m = 6), `lps = [0,1,0,1,2,0]`.

| i | j | T[i] | P[j] | Action |
|---|---|---|---|---|
| 0 | 0 | X | A | mismatch, j=0 -> i++ |
| 1 | 0 | X | A | mismatch, j=0 -> i++ |
| 2 | 0 | A | A | match -> i=3,j=1 |
| 3 | 1 | A | A | match -> i=4,j=2 |
| 4 | 2 | B | B | match -> i=5,j=3 |
| 5 | 3 | A | A | match -> i=6,j=4 |
| 6 | 4 | A | A | match -> i=7,j=5 |
| 7 | 5 | C | C | match -> i=8,j=6=m -> **match at shift 8−6 = 2** |

`T[2..7] = "AABAAC"` ✔.

## 5.5 Worked trace 2 — the fallback step in action (the important one)

`T2 = "AABAAABAAC"` (n = 10), same `P = "AABAAC"`, `lps = [0,1,0,1,2,0]`.

| i | j | T2[i] | P[j] | Action |
|---|---|---|---|---|
| 0 | 0 | A | A | match -> i=1,j=1 |
| 1 | 1 | A | A | match -> i=2,j=2 |
| 2 | 2 | B | B | match -> i=3,j=3 |
| 3 | 3 | A | A | match -> i=4,j=4 |
| 4 | 4 | A | A | match -> i=5,j=5 |
| **5** | **5** | **A** | **C** | **mismatch!** j≠0 -> `j = lps[4] = 2` (**i stays at 5**) |
| 5 | 2 | A | B | mismatch, j≠0 -> `j = lps[1] = 1` (**i still 5**) |
| 5 | 1 | A | A | match -> i=6,j=2 |
| 6 | 2 | B | B | match -> i=7,j=3 |
| 7 | 3 | A | A | match -> i=8,j=4 |
| 8 | 4 | A | A | match -> i=9,j=5 |
| 9 | 5 | C | C | match -> i=10,j=6=m -> **match at shift 10−6 = 4** |

`T2[4..9] = "AABAAC"` ✔. Notice at text index `i = 5` the algorithm tried **three different values of `j`** (5, then 2, then 1) **without ever moving `i`** — this is the entire saving over the naive algorithm, which would have restarted comparison from `T2[1]`, `T2[2]`, … one shift at a time, re-reading characters it had already seen.

## 5.6 Why the complexity is O(n + m) — the amortised argument

- **Building `lps`:** the pointer `len` in `COMPUTE-LPS` increases by at most 1 per iteration of the outer `while`, and every fallback (`len = lps[len-1]`) strictly decreases it. Since `len` can rise at most `m` times total, it can also fall at most `m` times total -> **O(m)**.
- **Matching phase:** the same argument applies to `i` and `j` — `j` can only be *incremented* when `i` is also incremented (at most `n` times total across the whole run), and every fallback decreases `j` without touching `i`. So the total number of increments of `j` across the whole algorithm is bounded by the total number of increments of `i`, which is at most `n`. Hence matching is **O(n)**.

```
Total: O(m) preprocessing + O(n) matching = Θ(n + m)   — always, no worst case exception.
```

## 5.7 Space complexity

`Θ(m)` for the `lps` array. `Θ(1)` beyond that.

---

# 6. Comparing the Three String-Matching Algorithms

| | Naive / Brute-force | Rabin-Karp | KMP |
|---|---|---|---|
| Preprocessing | none | Θ(m) | Θ(m) |
| Best case | Θ(n) | Θ(n+m) | Θ(n+m) |
| Average case | ~Θ(n) (large alphabet) | **Θ(n+m)** | Θ(n+m) |
| **Worst case** | **Θ(nm)** | **Θ(nm)** (many spurious hits) | **Θ(n+m)** — guaranteed |
| Extra space | Θ(1) | Θ(1) | Θ(m) |
| Text pointer `i` | can effectively re-scan | can effectively re-scan (on collision) | **never moves backward** |
| Best suited for | short/one-off searches, simplicity | **multiple-pattern** search (hash set of pattern hashes) | single-pattern search with a hard time guarantee |

> ⚠️ **Common exam trap.** "Rabin-Karp is always faster than naive matching" is **false** — with an unlucky modulus `q` its worst case is exactly as bad as the naive algorithm's, Θ(nm). Only **KMP** removes the quadratic worst case entirely.

---

# 7. Trie (Prefix Tree) — Data Structure for String Processing

## 7.1 What problem it solves

Storing `n` strings in a plain array or BST means a search still costs you string **comparisons**, each up to `O(L)` — so a lookup is `O(L log n)` at best. A **Trie** ("re**trie**val" tree) restructures storage around shared **prefixes**, so lookup, insert, and prefix search all cost **O(L)**, completely independent of how many other strings are stored.

> **Definition.** A Trie is a rooted tree in which each edge is labelled with a single character. The path from the root to a node spells out a prefix; a node is marked `isEndOfWord` if that prefix is itself a complete stored word.

## 7.2 Structure for words `{"cat", "car", "card", "dog"}`

```
                (root)
               /      \
              c        d
              |        |
              a        o
             / \        \
            t   r        g*
            *   * \
                    d
                    *
```
(`*` marks `isEndOfWord = true`.)

- "cat", "car", "card" all **share the prefix "ca"** — stored only once.
- "car" is a proper prefix of "card": the node for "car" is marked as end-of-word **and** still has a child `d`, so both "car" and "card" are recognised.

## 7.3 Node definition and core operations

```cpp
#include <string>
using namespace std;

struct TrieNode {
    TrieNode* children[26] = {nullptr};   // for lowercase a–z; use a map for a larger alphabet
    bool isEndOfWord = false;
};

class Trie {
    TrieNode* root;
public:
    Trie() { root = new TrieNode(); }

    void insert(const string& word) {                 // O(L)
        TrieNode* node = root;
        for (char ch : word) {
            int idx = ch - 'a';
            if (!node->children[idx])
                node->children[idx] = new TrieNode();
            node = node->children[idx];
        }
        node->isEndOfWord = true;
    }

    bool search(const string& word) {                 // O(L) — exact word
        TrieNode* node = findNode(word);
        return node != nullptr && node->isEndOfWord;
    }

    bool startsWith(const string& prefix) {            // O(L) — any word with this prefix?
        return findNode(prefix) != nullptr;
    }

private:
    TrieNode* findNode(const string& s) {
        TrieNode* node = root;
        for (char ch : s) {
            int idx = ch - 'a';
            if (!node->children[idx]) return nullptr;
            node = node->children[idx];
        }
        return node;
    }
};
```

**Trace — inserting `{"cat", "car", "card", "dog"}` then querying:**

```
insert("cat")  -> creates path root-c-a-t, marks t as end
insert("car")  -> reuses root-c-a, creates r, marks r as end
insert("card") -> reuses root-c-a-r (r stays marked end!), creates d, marks d as end
insert("dog")  -> creates path root-d-o-g, marks g as end

search("car")     -> walk c-a-r, node exists AND isEndOfWord=true  -> true
search("ca")      -> walk c-a, node exists but isEndOfWord=false   -> false
startsWith("ca")  -> walk c-a, node exists                        -> true
search("cars")    -> walk c-a-r, then children['s'-'a'] is null   -> false
```

## 7.4 Complexity

| Operation | Time | Why |
|---|---|---|
| Insert a word of length L | **Θ(L)** | one pointer step per character, independent of how many words already stored |
| Search a word of length L | **Θ(L)** | same reasoning |
| Prefix search (`startsWith`) | **Θ(L)** where L = prefix length | same walk, stop early |
| Space (worst case) | O(ALPHABET_SIZE × N × L) | N words of length L, no shared prefixes at all |
| Space (typical, shared prefixes) | much less | shared prefixes are stored once |

> 🔑 **The headline fact:** Trie lookup time depends only on the **length of the query string**, never on **how many** strings are stored — unlike a hash table (O(L) average but O(L) *worst* with collisions and no ordering) or a BST of strings (O(L log n)).

## 7.5 Applications

- **Autocomplete / predictive text** (search bars, IDE code completion) — walk to the prefix node, then DFS its subtree to list all completions.
- **Spell checkers** — O(L) membership test; near-miss suggestions via bounded edit-distance walks over the trie.
- **IP routing tables** — a binary trie over address bits supports **longest-prefix matching**.
- **T9 / phone keypad text prediction.**
- **Word games** (Boggle, Scrabble solvers) — prune a DFS the moment the current path is not a trie prefix.

---

# 8. Computational Geometry — Basic Tools

Computational geometry studies algorithms for problems phrased in terms of geometric objects — points, lines, polygons. Both algorithms in this unit (Closest-Pair, Convex Hull) are built from the same two primitives.

## 8.1 Euclidean distance

```
dist(P1, P2) = √[ (x1 − x2)² + (y1 − y2)² ]
```

For **comparing** distances (not reporting the actual value) you can skip the square root and compare **squared distances** — this avoids floating point and is faster.

## 8.2 The cross product / orientation test

For three points `O`, `A`, `B`, define:

```
cross(O, A, B) = (A.x − O.x)(B.y − O.y) − (A.y − O.y)(B.x − O.x)
```

This is the z-component of the cross product of vectors `OA` and `OB`, and it tells you the **turn direction** at `A` when walking `O -> A -> B`:

| Sign of cross(O,A,B) | Meaning |
|---|---|
| `> 0` | **counter-clockwise turn** (left turn) |
| `< 0` | **clockwise turn** (right turn) |
| `= 0` | `O`, `A`, `B` are **collinear** |

```cpp
long long cross(const Point& O, const Point& A, const Point& B) {
    return (long long)(A.x - O.x) * (B.y - O.y)
         - (long long)(A.y - O.y) * (B.x - O.x);
}
```

This one function is the workhorse of **Graham Scan** (Section 10) and of countless other geometric algorithms (line intersection, polygon area, point-in-polygon).

---

# 9. Closest-Pair Problem

## 9.1 Problem statement

> Given `n` points in the plane, find the pair with the smallest Euclidean distance between them.

## 9.2 Brute force — O(n²)

Check every pair, exactly as the problem is defined.

```cpp
pair<int,int> closestPairBrute(const vector<Point>& pts) {
    int n = pts.size();
    double best = 1e18; pair<int,int> ans;
    for (int i = 0; i < n; ++i)
        for (int j = i + 1; j < n; ++j) {           // BASIC OPERATION
            double d = dist(pts[i], pts[j]);
            if (d < best) { best = d; ans = {i, j}; }
        }
    return ans;
}
```

`C(n) = n(n−1)/2 = Θ(n²)` distance computations — identical shape to Unit 1's element-uniqueness example.

## 9.3 Divide-and-conquer — O(n log n)

The classic Levitin/CLRS algorithm brings this down to `Θ(n log n)`:

1. **Sort** all points by x-coordinate (once, up front, O(n log n)).
2. **Divide:** split the point set into a left half `L` and right half `R` by a vertical line at the median x.
3. **Conquer:** recursively find the closest pair in `L` and in `R`. Let `δ = min(δ_L, δ_R)`.
4. **Combine (the clever step):** the true closest pair might straddle the dividing line. It can only do so if both points lie within a **strip of width `2δ`** centred on the dividing line. Collect the strip's points, sort them by **y-coordinate**, and for each point check only the next few points in that sorted order (a geometric packing argument proves **at most 7** neighbours ever need checking) — this step is `O(n)` per level if the y-sorted order is maintained across recursion (or `O(n log n)` per level with a plain re-sort).

```
CLOSEST-PAIR(P sorted by x)
1  if |P| <= 3: return brute force on P
2  split P into L (left half), R (right half) by x-median
3  δL = CLOSEST-PAIR(L)
4  δR = CLOSEST-PAIR(R)
5  δ  = min(δL, δR)
6  build the strip: all points within δ of the dividing line
7  sort strip by y; check each point against the next ≤ 7 points in that order
8  return the smaller of δ and any strip distance found
```

### Why "at most 7 neighbours" — the geometric packing argument

Any two points inside the strip that are **both within `δ`** of each other must lie in a `δ × 2δ` rectangle straddling the line. If you try to pack points into that rectangle so that **every pair is at least `δ` apart** (otherwise they'd already have beaten `δ`, a contradiction), geometry limits you to **at most 8 points**, so any one point needs comparison with **at most 7** others in y-order — a constant, not `n`. This is what turns the "combine" step linear.

### The recurrence and its master-theorem solution

```
T(n) = 2T(n/2) + O(n)          (combine step is O(n) with maintained y-order)
```

By the Master Theorem (Unit 1, Case 2: `a=2, b=2, f(n)=Θ(n^{log_b a})=Θ(n)`):

```
T(n) = Θ(n log n)
```

(If you re-sort the strip by y at every level instead of maintaining it, combine costs `O(n log n)` per level, giving `T(n) = 2T(n/2) + O(n log n) = Θ(n log² n)` — still a correct, commonly taught version, just not optimal. Know both; be explicit about which combine-step cost you're assuming.)

## 9.4 Worked example

Points: `A(2,3), B(12,30), C(40,50), D(5,1), E(12,10), F(3,4)` (the classic textbook set).

**Brute-force check of the closest candidates:**

```
dist(A,F) = √[(2−3)² + (3−4)²] = √2 ≈ 1.414   ← smallest
dist(A,D) = √[(2−5)² + (3−1)²] = √13 ≈ 3.606
dist(D,E) = √[(5−12)²+(1−10)²] = √130 ≈ 11.40
dist(E,B) = √[(12−12)²+(10−30)²] = 20
```

**Closest pair: `A(2,3)` and `F(3,4)`, distance `√2 ≈ 1.414`.**

The divide-and-conquer version reaches the same answer by: sorting by x -> `{A,F,D,E,B,C}` -> splitting -> recursing on each half (finding `A,F` as the best pair inside the left half) -> checking the narrow strip around the median and confirming nothing crosses it more closely. Same result, far fewer comparisons once `n` is large.

## 9.5 Complexity summary

| Approach | Time | Space |
|---|---|---|
| Brute force | Θ(n²) | Θ(1) |
| Divide-and-conquer (maintained y-order) | **Θ(n log n)** | Θ(n) |
| Divide-and-conquer (re-sort strip each level) | Θ(n log² n) | Θ(n) |

---

# 10. Convex Hull — Graham Scan

## 10.1 Problem statement

> Given `n` points in the plane, find the smallest convex polygon that contains all of them. The polygon's vertices are a subset of the input points.

**Applications:** collision detection (approximate an object's outline), GIS / map boundary simplification, image processing (shape outlines), pattern recognition, robot motion planning.

## 10.2 Graham Scan algorithm

1. **Find the anchor point `P0`:** the point with the lowest y-coordinate (break ties by lowest x). It is guaranteed to be on the hull.
2. **Sort** the remaining `n − 1` points by **polar angle** around `P0`, ascending. Break ties (points collinear with `P0`) by **distance from `P0`, ascending** — this makes closer collinear points appear first, so the scan below naturally discards them.
3. **Scan:** push `P0` and the first two sorted points onto a stack. For every remaining point `p` in sorted order:
   - while the stack has ≥ 2 points and the last two points on the stack **do not make a strict left turn** with `p` (i.e. `cross(second-to-top, top, p) ≤ 0`), **pop** the stack;
   - then push `p`.
4. When done, the stack holds the convex hull vertices in **counter-clockwise** order.

```
GRAHAM-SCAN(points[0..n-1])
1  P0 = point with lowest y (tie: lowest x)
2  sort other points by polar angle around P0; ties by distance ascending
3  stack.push(P0); stack.push(points[0]); stack.push(points[1])
4  for i = 2 to n-2
5      while size(stack) >= 2 and cross(second-top, top, points[i]) <= 0
6          stack.pop()
7      stack.push(points[i])
8  return stack          // the hull, counter-clockwise
```

## 10.3 C++ implementation (Practical 9)

```cpp
#include <vector>
#include <algorithm>
#include <cmath>
using namespace std;

struct Point { long long x, y; };

Point pivot;

long long cross(const Point& O, const Point& A, const Point& B) {
    return (A.x - O.x) * (B.y - O.y) - (A.y - O.y) * (B.x - O.x);
}

long long dist2(const Point& a, const Point& b) {
    long long dx = a.x - b.x, dy = a.y - b.y;
    return dx * dx + dy * dy;
}

vector<Point> grahamScan(vector<Point> pts) {
    int n = pts.size();
    if (n < 3) return pts;

    // Step 1: anchor = lowest y (then lowest x)
    int lowest = 0;
    for (int i = 1; i < n; ++i)
        if (pts[i].y < pts[lowest].y ||
           (pts[i].y == pts[lowest].y && pts[i].x < pts[lowest].x))
            lowest = i;
    swap(pts[0], pts[lowest]);
    pivot = pts[0];

    // Step 2: sort by polar angle around pivot; ties by distance ascending
    sort(pts.begin() + 1, pts.end(), [](const Point& a, const Point& b) {
        long long c = cross(pivot, a, b);
        if (c == 0) return dist2(pivot, a) < dist2(pivot, b);
        return c > 0;                     // smaller polar angle first
    });

    // Step 3: scan
    vector<Point> hull;
    hull.push_back(pts[0]);
    hull.push_back(pts[1]);
    for (int i = 2; i < n; ++i) {
        while (hull.size() >= 2 &&
               cross(hull[hull.size() - 2], hull.back(), pts[i]) <= 0)
            hull.pop_back();
        hull.push_back(pts[i]);
    }
    return hull;
}
```

## 10.4 Fully worked, hand-verified trace

Points: `A(0,3), B(2,2), C(1,1), D(2,1), E(3,0), F(0,0), G(3,3)`.

**Step 1 — anchor:** lowest y is 0, shared by `E(3,0)` and `F(0,0)`; tie-break by lowest x -> **`P0 = F(0,0)`**.

**Step 2 — polar angles from F:**

| Point | angle from F | distance from F |
|---|---|---|
| E(3,0) | 0° | 3.00 |
| D(2,1) | 26.57° | 2.24 |
| C(1,1) | 45° | 1.41 |
| B(2,2) | 45° | 2.83 |
| G(3,3) | 45° | 4.24 |
| A(0,3) | 90° | 3.00 |

C, B, G are collinear with F (all on the line y = x); sorted by distance ascending they come out **C, B, G**.

**Sorted order: `E, D, C, B, G, A`.**

**Step 3 — scan** (stack shown after each step; `cross(2nd-top, top, candidate)`):

| Action | cross test | Result | Stack after |
|---|---|---|---|
| init | — | — | [F, E] |
| push D | cross(F,E,D) = 3 > 0 | keep | [F, E, D] |
| push C | cross(E,D,C) = 1 > 0 | keep | [F, E, D, C] |
| try B | cross(D,C,B) = −1 ≤ 0 | **pop C** | [F, E, D] |
| try B | cross(E,D,B) = −1 ≤ 0 | **pop D** | [F, E] |
| push B | cross(F,E,B) = 6 > 0 | keep | [F, E, B] |
| try G | cross(E,B,G) = −3 ≤ 0 | **pop B** | [F, E] |
| push G | cross(F,E,G) = 9 > 0 | keep | [F, E, G] |
| push A | cross(E,G,A) = 9 > 0 | keep | [F, E, G, A] |

**Final hull: `F(0,0) -> E(3,0) -> G(3,3) -> A(0,3)`** — exactly the square boundary, with the three interior points `B, C, D` correctly excluded.

```
        A(0,3) ────────────── G(3,3)
          │        B(2,2)        │
          │     C(1,1) D(2,1)    │
          │                      │
        F(0,0) ────────────── E(3,0)
```

## 10.5 Complexity

| Step | Time |
|---|---|
| Find anchor | O(n) |
| Sort by polar angle | **O(n log n)** — dominates |
| Scan (push/pop) | O(n) amortised — each point is pushed once and popped at most once |
| **Total** | **Θ(n log n)** |

**Why the scan is O(n), not O(n²), despite the nested-looking `while`:** exactly the amortised argument from Unit 1 (Section 7.9) — every point is pushed onto the stack **exactly once** across the whole algorithm, so it can be popped **at most once**. Total pushes + pops ≤ 2n = O(n).

## 10.6 Edge cases

- **All points collinear:** no 2-D hull exists; typically return the two extreme points as a degenerate "hull".
- **Duplicate points:** remove duplicates before sorting, or they will create zero-length vectors and undefined angle comparisons.
- **Ties in polar angle:** must break by distance (nearer first) as shown above — get this backwards and the scan will incorrectly keep an interior point.

---

# 11. Practice Questions — Unit 2 (Full Solutions)

## Part A — Brute-force string matching

### Q1. Trace brute-force string matching of `P = "AAB"` against `T = "AAAAAB"` and count total character comparisons.

**Solution.** n = 6, m = 3, shifts s = 0..3.

| s | window | comparisons | outcome |
|---|---|---|---|
| 0 | AAA | A=A, A=A, B≠A -> 3 | fail |
| 1 | AAA | 3 | fail |
| 2 | AAA | 3 | fail |
| 3 | AAB | A=A, A=A, B=B -> 3 | **match** |

**Total = 12 comparisons.** Match found at shift 3.

---

### Q2. Construct a worst-case input for the naive algorithm with n = 12, m = 4 and state the exact comparison count.

**Solution.** `T = "AAAAAAAAAAAA"` (12 A's), `P = "AAAB"` (never occurs). Every one of the `n − m + 1 = 9` shifts compares all 4 characters before failing on the last.

```
C = m(n − m + 1) = 4 × 9 = 36 comparisons
```

---

## Part B — Rabin-Karp

### Q3. With `d = 10, q = 13`, compute the hash of `P = "26"` and of the text window `"39"`. Are they a spurious hit, a real match, or no hash collision?

**Solution.**

```
p       = 26 mod 13 = 0
"39"    = 39 mod 13 = 0
```

Hashes collide (both 0) but `"26" ≠ "39"` -> **spurious hit** — the algorithm would compare characters, find them unequal, and correctly move on.

---

### Q4. Why does Rabin-Karp verify with a full character comparison even after a hash match, while KMP never needs to "re-verify" after `j` reaches `m`?

**Solution.** Rabin-Karp's hash is a many-to-one function — different substrings can collide to the same value (a spurious hit), so a hash match is only *evidence*, not proof. KMP never compresses information lossily: `j == m` is reached only by `m` consecutive **real** character matches (line 5's `text[i] == pattern[j]` test), so it is already a proven match, not a probabilistic one.

---

## Part C — KMP

### Q5. Compute the LPS array for `P = "AAAA"`.

**Solution.**

```
i=0: lps[0]=0
i=1: P[1]=A=P[0]=A -> len=1, lps[1]=1
i=2: P[2]=A=P[1]=A -> len=2, lps[2]=2
i=3: P[3]=A=P[2]=A -> len=3, lps[3]=3
```

**`lps = [0,1,2,3]`** — for an all-repeated-character pattern, `lps[i] = i`.

---

### Q6. Compute the LPS array for `P = "ABCDE"` (all distinct characters). What does the result tell you about KMP's behaviour on patterns with no self-overlap?

**Solution.** No character ever repeats, so no prefix can equal a later suffix.

**`lps = [0,0,0,0,0]`**

Every mismatch falls back to `j = 0`, so `i` must simply advance — KMP degenerates to exactly the naive algorithm's shift-by-one behaviour on such a pattern (but is still never worse than O(n+m), since a fresh comparison starts only where the last one left off).

---

### Q7. Using `lps = [0,1,0,1,2,0]` for `P = "AABAAC"`, trace KMP on `T = "AABAABAABAAC"` (n = 12) and report every shift examined and the final match position.

**Solution.** T indices: 0A 1A 2B 3A 4A 5B 6A 7A 8B 9A 10A 11C.

```
i=0,j=0: A=A -> i1,j1
i=1,j=1: A=A -> i2,j2
i=2,j=2: B=B -> i3,j3
i=3,j=3: A=A -> i4,j4
i=4,j=4: A=A -> i5,j5
i=5,j=5: T[5]=B vs P[5]=C -> mismatch, j=lps[4]=2
i=5,j=2: T[5]=B vs P[2]=B -> match -> i6,j3
i=6,j=3: T[6]=A vs P[3]=A -> match -> i7,j4
i=7,j=4: T[7]=A vs P[4]=A -> match -> i8,j5
i=8,j=5: T[8]=B vs P[5]=C -> mismatch, j=lps[4]=2
i=8,j=2: T[8]=B vs P[2]=B -> match -> i9,j3
i=9,j=3: T[9]=A vs P[3]=A -> match -> i10,j4
i=10,j=4: T[10]=A vs P[4]=A -> match -> i11,j5
i=11,j=5: T[11]=C vs P[5]=C -> match -> i12,j6=m -> MATCH at shift 12-6=6
```

**Match at shift 6** (`T[6..11] = "AABAAC"` ✔). Note `i` climbed from 0 to 12 **monotonically** — it never once decreased, even though two mismatches occurred.

---

## Part D — Trie

### Q8. Insert `{"go", "gone", "good", "gate"}` into an empty trie. Draw the resulting structure and state which nodes are `isEndOfWord`.

**Solution.**

```
root
 └─ g
    ├─ o
    │  ├─* (end of "go")
    │  ├─ n
    │  │  └─ e *  (end of "gone")
    │  └─ o
    │     └─ d *  (end of "good")
    └─ a
       └─ t
          └─ e *  (end of "gate")
```

`isEndOfWord = true` at: the `o` right after `g-o` (word "go"), the `e` after `g-o-n-e` ("gone"), the `d` after `g-o-o-d` ("good"), the `e` after `g-a-t-e` ("gate"). All other nodes are structural only.

---

### Q9. What does `startsWith("go")` return after the trie in Q8 is built, and why is it different from `search("go")`?

**Solution.** Both return **true**, but for different reasons: `search("go")` is true because the node reached by walking "g"->"o" has `isEndOfWord = true` (an *exact* stored word). `startsWith("go")` is true simply because that node **exists** — it doesn't check `isEndOfWord` at all, so it would also return true for a prefix like "gon" which is not itself a complete word but leads to "gone".

---

### Q10. Why is trie search O(L) rather than O(L log n) like a balanced BST of strings, or O(1) amortised like a hash table? Give the key structural reason.

**Solution.** A BST of strings must still perform up to `O(log n)` string **comparisons** during the search, each costing up to `O(L)`, giving `O(L log n)`. A trie instead spends exactly one `O(1)` pointer step **per character of the query**, and the number of stored words `n` never enters the walk at all — so it is `Θ(L)` regardless of whether 10 or 10 million words are stored. A hash table matches trie speed for exact search, but cannot answer `startsWith` efficiently and offers no ordering; the trie supports **prefix** queries natively because a prefix is literally a path from the root.

---

## Part E — Closest pair and convex hull

### Q11. Given points `(1,1), (2,2), (3,3), (10,10), (10.5,10.5)`, find the closest pair by brute force and state the number of distance computations.

**Solution.** n = 5, so `C(5,2) = 10` pairs checked.

```
dist((10,10),(10.5,10.5)) = √(0.25+0.25) = √0.5 ≈ 0.707   ← smallest
dist((1,1),(2,2)) = √2 ≈ 1.414
dist((2,2),(3,3)) = √2 ≈ 1.414
```

**Closest pair: `(10,10)` and `(10.5,10.5)`**, distance ≈ 0.707.

---

### Q12. State the recurrence for divide-and-conquer closest-pair (with maintained y-order) and solve it. Which Master Theorem case applies?

**Solution.**

```
T(n) = 2T(n/2) + Θ(n)
```

`a = 2, b = 2`, watershed `n^{log_2 2} = n^1 = n`. Since `f(n) = Θ(n) = Θ(n^{log_b a})`, this is **Case 2** -> `T(n) = Θ(n log n)`.

(This is the identical recurrence shape as merge sort, Unit 1 Section 9 — same proof, same answer.)

---

### Q13. In the closest-pair combine step, why is it provably enough to check each strip point against only the next 7 points in y-sorted order, instead of all other strip points?

**Solution.** Any pair of strip points that could possibly beat the current best distance `δ` must both lie within `δ` of the dividing line **and** within `δ` of each other in y-coordinate — i.e. inside a `2δ × δ` rectangle. If every pair inside that rectangle were required to be **at least `δ`** apart (otherwise one of them would already have set a smaller `δ`), simple packing geometry shows the rectangle can hold **at most 8** such points. So for any one point, **at most 7** others in the rectangle are close enough in y-order to be worth checking — a constant, independent of `n`.

---

### Q14. Points `(0,0), (4,0), (4,4), (0,4), (2,2)`. Find the anchor for Graham Scan and sort the rest by polar angle.

**Solution.** Lowest y = 0, shared by `(0,0)` and `(4,0)`; tie-break lowest x -> **anchor = (0,0)**.

```
angle((4,0))  = 0°
angle((2,2))  = 45°
angle((4,4))  = 45°   (collinear with (2,2) on line y=x; farther, so sorts after (2,2))
angle((0,4))  = 90°
```

**Sorted order: `(4,0), (2,2), (4,4), (0,4)`.**

---

### Q15. Continue Q14: run the Graham Scan and report the final hull. Explain why `(2,2)` is excluded.

**Solution.** Stack starts `[(0,0), (4,0)]`; push `(2,2)` -> `[(0,0),(4,0),(2,2)]` (only 2 elements before push, no test yet — need ≥2 in stack *before* the candidate to test, so this push happens directly per the initialisation rule.)

- Try push `(4,4)`: test `cross((4,0),(2,2),(4,4))`
  `= (2-4)(4-0) - (2-0)(4-4) = (-2)(4) - (2)(0) = -8` ≤ 0 -> **pop `(2,2)`**. Stack: `[(0,0),(4,0)]`.
  Push `(4,4)`: stack `[(0,0),(4,0),(4,4)]`.
- Try push `(0,4)`: test `cross((4,0),(4,4),(0,4))`
  `= (4-4)(4-0) - (4-0)(0-4) = 0 - (4)(-4) = 16 > 0` -> keep. Push `(0,4)`.

**Final hull: `(0,0) -> (4,0) -> (4,4) -> (0,4)`** — the outer square.

`(2,2)` is excluded because it is the **exact centre** of the square, strictly inside every edge — the cross-product test at `(4,0),(2,2),(4,4)` detects a **clockwise (right) turn**, proving `(2,2)` lies "inside" relative to the sweep and must be popped.

---

# 12. Combined Practice Questions — Units 1 & 2

A mixed set spanning both units — the kind of range a mid-term or end-term paper actually uses.

### CQ1. (Unit 1) Prove `4n² + 3n = Θ(n²)`.

**Solution.** Upper: `4n²+3n ≤ 4n²+3n² = 7n²` for n ≥ 1 -> c₂=7. Lower: `4n²+3n ≥ 4n²` for n ≥ 0 -> c₁=4. So `4n² ≤ 4n²+3n ≤ 7n²` for n ≥ 1 -> **Θ(n²)**, c₁=4, c₂=7, n₀=1. ∎

### CQ2. (Unit 2) Give the worst-case time of brute-force string matching, and name **one** input pair `(T, P)` of length 8 and 3 that achieves it.

**Solution.** Worst case `Θ(nm)`. Example: `T = "AAAAAAAA"` (8 A's), `P = "AAB"` (never found) — every shift compares all 3 characters before failing, giving `3 × 6 = 18 = m(n−m+1)` comparisons.

### CQ3. (Unit 1) Solve `T(n) = 2T(n/2) + n` and name one Unit-2 algorithm with exactly this recurrence.

**Solution.** Master theorem Case 2 -> `Θ(n log n)`. **Closest-pair divide-and-conquer** (Section 9.3) has this identical recurrence — and so does merge sort.

### CQ4. (Unit 2) Explain in one sentence each why KMP is `Θ(n+m)` worst case but Rabin-Karp is only `Θ(n+m)` on *average*.

**Solution.** KMP's text pointer `i` provably never decreases, so total work is structurally bounded by `O(n+m)` regardless of the input. Rabin-Karp's bound depends on the **number of spurious hash collisions**, which for an adversarial input/modulus combination can be as high as every single shift, degrading to `Θ(nm)`.

### CQ5. (Unit 1) What is the auxiliary space of recursive merge sort, and what is the auxiliary space of the divide-and-conquer closest-pair algorithm? Why are they the same order?

**Solution.** Both are **Θ(n)**: merge sort needs an O(n) temporary buffer for merging, and closest-pair needs O(n) for the sorted-by-x and sorted-by-y auxiliary arrays maintained across recursive calls. Both also carry an O(log n) recursion stack, which is dominated by the O(n) term.

### CQ6. (Unit 2) A trie stores 500,000 English words, average length 7. Give the time to check whether a 7-letter string is a stored word, and contrast it with a sorted-array binary search over the same 500,000 words.

**Solution.** Trie: **Θ(7) = Θ(L)**, independent of the 500,000. Binary search over a sorted array: `Θ(log₂ 500000) ≈ 19` **string comparisons**, each itself up to `O(L)` in the worst case (when strings share a long common prefix) -> up to `Θ(L log n) ≈ 133` character operations. The trie is asymptotically faster because `n` (the collection size) does not appear in its bound at all.

### CQ7. (Unit 1) State the Master Theorem case for `T(n) = 7T(n/2) + n²` (Strassen) and for `T(n) = 2T(n/2) + n` (merge sort / closest pair). Why do they land in different cases?

**Solution.** Strassen: `log₂7 ≈ 2.807 > 2`, so `f(n)=n²` grows **slower** than `n^{log_b a}` -> **Case 1** -> `Θ(n^2.807)`. Merge sort / closest pair: `log₂2 = 1`, and `f(n)=n = Θ(n^1)` matches exactly -> **Case 2** -> `Θ(n log n)`. The difference is entirely in how `f(n)` (the "combine" cost) compares to `n^{log_b a}` (the cost implied purely by the branching factor and subproblem size).

### CQ8. (Unit 2) In Graham Scan, what would go wrong if ties in polar angle were broken by **farthest point first** instead of nearest first?

**Solution.** The scan would push the farther collinear point first, then when the nearer one is processed, the cross-product test against it and its predecessor would find it **inside** the already-pushed farther point's line and incorrectly pop the *outer* (correct) hull point instead of discarding the inner one — corrupting the hull. Nearest-first ensures interior collinear points get popped, not hull vertices.

### CQ9. (Unit 1) Sort the growth rates: `n log n`, `2ⁿ`, `n^2.807`, `log n`, `n`, `n²`.

**Solution.** `log n < n < n log n < n² < n^2.807 < 2ⁿ`.

### CQ10. (Unit 2) A closest-pair combine step re-sorts the strip by y at every recursion level instead of maintaining sorted order across calls. Give the resulting recurrence and its solution.

**Solution.** `T(n) = 2T(n/2) + O(n log n)` — Case 2 of the **extended** master theorem (`f(n) = Θ(n^1 log^1 n)`, `log_b a = 1 = k`, `p=1 > −1`) -> `T(n) = Θ(n log² n)`.

---

## Rapid-fire MCQ — both units

| # | Question | Answer |
|---|---|---|
| 1 | Worst-case time of naive string matching | **Θ(nm)** |
| 2 | Worst-case time of Rabin-Karp | **Θ(nm)** (bad modulus) |
| 3 | Worst-case time of KMP | **Θ(n+m)** — always |
| 4 | KMP preprocessing time | Θ(m) — building the LPS array |
| 5 | What does `lps[i]` mean? | length of the longest proper prefix of `P[0..i]` that is also a suffix |
| 6 | Does KMP's text pointer `i` ever decrease? | **No, never** |
| 7 | Rabin-Karp's main real-world advantage over KMP | efficient **multi-pattern** search via a hash set |
| 8 | Trie search time for a word of length L | **Θ(L)**, independent of n |
| 9 | Trie space, worst case, N words length L | O(ALPHABET × N × L) |
| 10 | Closest-pair brute force | Θ(n²) |
| 11 | Closest-pair divide-and-conquer | Θ(n log n) |
| 12 | Master theorem case for `T(n)=2T(n/2)+n` | Case 2 -> Θ(n log n) |
| 13 | Why at most 7 neighbours checked in the closest-pair strip | packing argument — max 8 points fit in the δ×2δ rectangle |
| 14 | Convex hull, Graham Scan time | **Θ(n log n)** (sorting dominates) |
| 15 | Graham Scan anchor point selection rule | lowest y, tie-break lowest x |
| 16 | Sign of `cross(O,A,B) > 0` means | counter-clockwise / left turn |
| 17 | Sign of `cross(O,A,B) = 0` means | collinear |
| 18 | Graham Scan tie-break for equal polar angle | sort by **distance ascending** |
| 19 | Amortised cost argument shared by Graham Scan and Unit 1's two-pointer example | each element pushed/advanced **at most once** total |
| 20 | Strassen's multiplication count per level | 7 (vs. 8 for naive block D&C) |

---

# 13. One-Page Revision Sheet — Unit 2

## String matching complexity, side by side

| Algorithm | Preprocessing | Best | Average | **Worst** | Extra space |
|---|---|---|---|---|---|
| Naive / brute-force | — | Θ(n) | ~Θ(n) | **Θ(nm)** | Θ(1) |
| Rabin-Karp | Θ(m) | Θ(n+m) | **Θ(n+m)** | Θ(nm) | Θ(1) |
| KMP | Θ(m) | Θ(n+m) | Θ(n+m) | **Θ(n+m)** | Θ(m) |

## Rabin-Karp rolling hash

```
h = d^(m-1) mod q                                  (once)
t_{s+1} = ( d·(t_s − T[s]·h) + T[s+m] ) mod q       (per shift, O(1))
```
Always verify a hash match character-by-character — hashes can collide (spurious hits).

## KMP LPS array

```
lps[i] = length of longest proper prefix of P[0..i] that is also a suffix of P[0..i]
```
Build: O(m). Match: O(n). On mismatch with `j != 0`: `j = lps[j-1]`, **i never moves**.

## Trie

```
Insert / Search / Prefix-search  ->  all Θ(L),  L = length of the query string
```
`isEndOfWord` marks complete words; existence of a node alone only proves a **prefix** is stored.

## Computational geometry primitives

```
cross(O,A,B) = (A.x−O.x)(B.y−O.y) − (A.y−O.y)(B.x−O.x)
   > 0  counter-clockwise (left turn)      < 0  clockwise (right turn)      = 0  collinear
```

## Closest pair

```
Brute force:            Θ(n²)
Divide-and-conquer:      T(n) = 2T(n/2) + Θ(n)   ->  Θ(n log n)   [strip check: ≤ 7 neighbours in y-order]
```

## Convex hull — Graham Scan

```
1. Anchor  = lowest y, tie-break lowest x
2. Sort remaining points by polar angle around anchor; ties by distance ascending
3. Scan with a stack: pop while cross(2nd-top, top, next) <= 0; then push
   Total time: Θ(n log n)  (sort dominates; scan itself is Θ(n) amortised)
```

## The unifying thread back to Unit 1

| Unit 2 fact | Unit 1 concept it reuses |
|---|---|
| Closest-pair recurrence `2T(n/2)+n` | identical to merge sort -> Master Theorem Case 2 |
| KMP's "`i` never decreases" proof | identical amortised argument to Unit 1 §7.9's two-pointer example |
| Graham Scan push/pop bound | same "each element handled O(1) times total" amortised reasoning |
| Naive string matching worst case `Θ(nm)` | same shape as Unit 1's nested-loop `Θ(n²)` triangular sum, generalised by `m` |

---

## Further reading

- **Levitin**, *Introduction to the Design and Analysis of Algorithms* — Chapter 3 (Brute Force: string matching, closest pair, convex hull), Chapter 7 (Space-and-Time Tradeoffs: hashing, tries).
- **CLRS**, *Introduction to Algorithms* — Chapter 32 (String Matching: naive, Rabin-Karp, KMP), Chapter 33 (Computational Geometry: closest pair, convex hull).
- [VisuAlgo](https://visualgo.net/en) — animated string matching and convex hull.
- [CP-Algorithms](https://cp-algorithms.com/) — proofs and implementations of KMP, Rabin-Karp, Convex Hull (Andrew's monotone chain, a close cousin of Graham Scan).
- [GeeksforGeeks DSA](https://www.geeksforgeeks.org/dsa/) — worked examples and additional practice problems.

---

*Unit 2 reading material — CSE 408 Design and Analysis of Algorithms.*
