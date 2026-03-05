+++
title = "Zero-Knowledge: Turning Computation into Polynomials (Part 1/3)"
date = 2026-03-04
draft = true
description = "From a program to a few polynomial equations: how SNARKs encode computation through flattening, R1CS, and the QAP transformation"

[taxonomies]
tags = ["crypto", "computer-science"]

[extra]
katex = true
social_media_card = "/img/zk-computation-to-polynomials-banner.webp"
+++

![Origami crane folded from a sheet of mathematical notation](/img/zk-computation-to-polynomials-banner.webp)

Zero knowledge keeps coming up. Balaji [frames it](https://x.com/balajis/status/2022462579713675506) as the counterweight to AI: "Artificial intelligence is the attack. Zero knowledge is the defense." [Podcasts](https://zeroknowledge.fm/), conference talks, [crypto roadmaps](https://strawmap.org/): ZK is everywhere. I wanted to understand what's actually going on under the hood, so [as promised](@/blog/verkle-trees-polynomial-commitments.md#what-s-next), I traced the math from scratch. The part that surprised me most wasn't the cryptography. It was the step before: how do you take a program and turn it into something cryptography can even work with?

That's what this post covers. We'll take a small program, break it into elementary operations, encode those operations as matrices, and collapse everything into a single equation that a SNARK[^snark] can check. This is Part 1 of three (Part 2 covers the proof protocol, Part 3 covers applications). I'll assume familiarity with [finite fields](@/blog/math-behind-private-key.md#fields-numbers-with-arithmetic) and [polynomial commitments](@/blog/verkle-trees-polynomial-commitments.md) from my earlier posts.

## Proving Without Showing

Alice claims she knows some value $x$ such that $f(x) = y$. Bob wants to be convinced, but he doesn't want to run $f$ himself (maybe it's expensive), and Alice doesn't want to reveal $x$ (maybe it's secret). A **zero-knowledge proof** lets Alice convince Bob of both things simultaneously.

A zero-knowledge proof guarantees two things:

- **Privacy**: Alice doesn't reveal $x$.
- **Succinctness**: Bob's verification work is tiny compared to running $f$.

Our running example for the entire post (borrowed from [Vitalik's QAP walkthrough](https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649)):

$$f(x) = x^3 + x + 5$$

Alice claims $f(3) = 35$. By the end of the post, we'll have transformed this claim into a single polynomial divisibility check.

Step one is to break the computation into pieces small enough to encode as constraints.

## Flattening: Breaking Computation into Gates

The idea is to rewrite $f(3) = 35$ as a sequence of elementary operations, each of the form `result = left (op) right`. Think of it as disassembling a formula into its simplest possible operations:

```
sym1 = x * x          // gate 1: squaring
sym2 = sym1 * x        // gate 2: cubing
sym3 = sym2 + x        // gate 3: addition
out  = sym3 + 5        // gate 4: add constant
```

Each line is a **gate**. The full set is an **arithmetic circuit**. The process of encoding computation as gates is called **arithmetization**, and it's the first stage of the SNARK pipeline. The flattened form encodes exactly the same logic as $f(x)$. Just a different representation.

![Expression tree for x³ + x + 5 flattened into four sequential gates](/img/zk-flattening.webp)

We now have four gates. The next step: express each one as a constraint that a verifier can check.

## R1CS: Constraints as Dot Products

The next step encodes each gate as a constraint on a single shared vector. The format is called a **Rank-1 Constraint System (R1CS)**. All the arithmetic from here on happens over a [finite field](@/blog/math-behind-private-key.md#fields-numbers-with-arithmetic) $\mathbb{F}_p$. We use small integers to keep the example readable, but in practice $p \sim 2^{255}$.

The vector $\mathbf{s}$ is a flat list of every variable in the circuit:

$$\mathbf{s} = [1,\ x,\ \text{out},\ \text{sym1},\ \text{sym2},\ \text{sym3}] \tag{1}$$

With $x = 3$:

$$\mathbf{s} = [1,\ 3,\ 35,\ 9,\ 27,\ 30]$$

That leading 1 isn't a variable: it's a constant term that we'll see in action when gate 4 encodes the "+ 5". This vector, with all its intermediate values filled in, is the **witness**: it's everything the prover computed.

Each gate becomes a constraint of the form:

$$(\mathbf{L}_i \cdot \mathbf{s}) \times (\mathbf{R}_i \cdot \mathbf{s}) = (\mathbf{O}_i \cdot \mathbf{s}) \tag{2}$$

where $\mathbf{L}_i$ (**l**eft input), $\mathbf{R}_i$ (**r**ight input), and $\mathbf{O}_i$ (**o**utput) are the $i$-th rows of matrices $L$, $R$, and $O$. Each row vector "selects" the right variables from $\mathbf{s}$.

### Gates 1 and 2: encoding multiplication

Gate 1 is $\text{sym1} = x \times x$. Looking back at $\mathbf{s}$ from equation $(1)$, we need vectors that select $x$ on the left, $x$ on the right, and $\text{sym1}$ as the result:

$$\mathbf{L}_1 = [0, 1, 0, 0, 0, 0] \quad \text{(selects } x \text{)}$$

$$\mathbf{R}_1 = [0, 1, 0, 0, 0, 0] \quad \text{(selects } x \text{)}$$

$$\mathbf{O}_1 = [0, 0, 0, 1, 0, 0] \quad \text{(selects sym1)}$$

Check: $(\mathbf{L}_1 \cdot \mathbf{s}) \times (\mathbf{R}_1 \cdot \mathbf{s}) = 3 \times 3 = 9 = (\mathbf{O}_1 \cdot \mathbf{s})$. It works.

Gate 2 ($\text{sym2} = \text{sym1} \times x$) follows the same multiplication pattern: $\mathbf{L}_2$ selects $\text{sym1}$, $\mathbf{R}_2$ selects $x$, and $\mathbf{O}_2$ selects $\text{sym2}$.

### Gates 3 and 4: encoding addition

Gate 3 has no multiplication: $\text{sym3} = \text{sym2} + x$. We encode it as $(\text{sym2} + x) \times 1 = \text{sym3}$, folding the addition into the $\mathbf{L}$ vector. The right side is just the constant 1:

$$\mathbf{L}_3 = [0, 1, 0, 0, 1, 0]$$

$$\mathbf{R}_3 = [1, 0, 0, 0, 0, 0]$$

$$\mathbf{O}_3 = [0, 0, 0, 0, 0, 1]$$

Check: $(27 + 3) \times 1 = 30$. Gate 4 ($\text{out} = \text{sym3} + 5$) works the same way, with the constant 5 in the first column of $\mathbf{L}_4$: $\mathbf{L}_4 = [5, 0, 0, 0, 0, 1]$. Check: $(5 + 30) \times 1 = 35$.

The key insight: additions don't add constraints. They ride along in the $\mathbf{L}_i$ or $\mathbf{O}_i$ vectors of existing gates. If the original computation had been $(\text{sym2} + x) \times 2$ instead of a pure addition, that would still be one gate: $\mathbf{L}_i$ selects $\text{sym2} + x$, $\mathbf{R}_i$ selects the constant 2, and the addition costs nothing extra. The constraint count is driven by multiplications; additions just come along for free.

### The Full Matrices

<details>
<summary>Each row is a gate, each column header an entry in $\mathbf{s}$. Click to expand.</summary>

{% table() %}

|       | 1 | $x$ | out | sym1 | sym2 | sym3 |
|-------|---|-----|-----|------|------|------|
| $\mathbf{L}_1$ | 0 | 1   | 0   | 0    | 0    | 0    |
| $\mathbf{L}_2$ | 0 | 0   | 0   | 1    | 0    | 0    |
| $\mathbf{L}_3$ | 0 | 1   | 0   | 0    | 1    | 0    |
| $\mathbf{L}_4$ | 5 | 0   | 0   | 0    | 0    | 1    |
{% end %}

{% table() %}

|       | 1 | $x$ | out | sym1 | sym2 | sym3 |
|-------|---|-----|-----|------|------|------|
| $\mathbf{R}_1$ | 0 | 1   | 0   | 0    | 0    | 0    |
| $\mathbf{R}_2$ | 0 | 1   | 0   | 0    | 0    | 0    |
| $\mathbf{R}_3$ | 1 | 0   | 0   | 0    | 0    | 0    |
| $\mathbf{R}_4$ | 1 | 0   | 0   | 0    | 0    | 0    |
{% end %}

{% table() %}

|       | 1 | $x$ | out | sym1 | sym2 | sym3 |
|-------|---|-----|-----|------|------|------|
| $\mathbf{O}_1$ | 0 | 0   | 0   | 1    | 0    | 0    |
| $\mathbf{O}_2$ | 0 | 0   | 0   | 0    | 1    | 0    |
| $\mathbf{O}_3$ | 0 | 0   | 0   | 0    | 0    | 1    |
| $\mathbf{O}_4$ | 0 | 0   | 1   | 0    | 0    | 0    |
{% end %}

</details>

A valid witness satisfies all four instances of equation $(2)$ simultaneously. An invalid witness (e.g., wrong intermediate values, wrong output) fails at least one. We've now established that R1CS works, but also that it requires checking each gate separately: four dot products for four gates.

## QAP: From Dot Products to Polynomials

The **Quadratic Arithmetic Program (QAP)** transformation converts those four separate R1CS checks into a single polynomial divisibility check. This is the step that makes SNARKs succinct.

The technique: take each *column* of the L, R, O matrices and turn it into a polynomial via [Lagrange interpolation](@/blog/verkle-trees-polynomial-commitments.md#from-values-to-a-polynomial). The Verkle trees post already covered this: encoding discrete values as evaluations of a polynomial. Same trick, different data.

Concretely, column $j$ of matrix $L$ has 4 values (one per gate). Treat these as evaluations at points $t = 1, 2, 3, 4$ and interpolate a degree-3 polynomial $L_j(t)$.[^tvar] For example, the $x$-column of $L$ has values $[1, 0, 1, 0]$, so $L_1(t)$ is the unique degree-3 polynomial passing through the points $(1, 1),\ (2, 0),\ (3, 1),\ (4, 0)$.

Repeat for every column of L, R, and O. Result: 6 polynomials each for L, R, and O since $\mathbf{s}$ has 6 entries (18 polynomials total).

We want a single polynomial that, at each gate point $t = i$, evaluates to the dot product $\mathbf{L}_i \cdot \mathbf{s}$. A useful property makes this possible: a weighted sum of polynomials is itself a polynomial. The weights just scale and combine the coefficients. So we can use the witness entries $s_j$ as weights on the column polynomials:

$$L(t) = \sum_{j=0}^{5} s_j \cdot L_j(t)$$

$$R(t) = \sum_{j=0}^{5} s_j \cdot R_j(t)$$

$$O(t) = \sum_{j=0}^{5} s_j \cdot O_j(t)$$

At any gate point $t = i$, $L(i)$ evaluates to $\mathbf{L}_i \cdot \mathbf{s}$: the left-hand dot product for gate $i$. The same holds for $R(i)$ and $O(i)$. **One polynomial per side (3 total), encoding all four gates at once.**

For a valid witness, every gate constraint holds, so $L(t) \cdot R(t) - O(t) = 0$ at $t = 1, 2, 3, 4$. Call this expression $T(t)$. It's a polynomial with roots at $1, 2, 3, 4$.

Define the **target polynomial** $Z(t) = (t-1)(t-2)(t-3)(t-4)$, which has roots at those same four points.

Since $T(t)$ and $Z(t)$ share the same roots, $Z(t)$ divides $T(t)$ with no remainder. The prover can perform polynomial long division to obtain a **quotient polynomial** $H(t)$ such that:

$$T(t) = H(t) \cdot Z(t) \tag{3}$$

If even one gate constraint were violated, $T(t)$ would lose a root, the division would leave a remainder, and no polynomial $H(t)$ would exist.

![Pipeline from program to polynomial divisibility check: Program, Flatten, R1CS, QAP, Check](/img/zk-pipeline.webp)

## What One Equation Buys Us

The **Schwartz-Zippel lemma** makes equation $(3)$ powerful: evaluate both sides at a random $\tau$ to verify a computation; if the polynomials differ, the chance of accidentally passing is negligible.[^sz] One random evaluation replaces all $n$ gate checks. Production circuits have millions of gates (Zcash Sapling uses ~1.5 million, zkEVM rollups tens of millions). That's where the "succinct" in SNARK comes from.

## What's Missing

Two problems stand between the QAP and an actual proof.

**The random point must be secret.** If the prover knew $\tau$ beforehand, they could pick values for $T(\tau)$ and $H(\tau)$ that satisfy equation $(3)$ without a valid witness.

**The prover must be bound to the circuit's polynomials.** Even with a secret $\tau$, nothing forces the prover to use the QAP's column polynomials. They could fabricate unrelated polynomials that pass the divisibility check.

Part 2 will add the cryptography: a trusted setup that hides $\tau$, pairing-based checks that bind the prover to the circuit's polynomials, and a blinding step that makes the proof reveal nothing about the witness.

<!-- TODO: Add link to Part 2 when published -->

---

[^snark]: **S**uccinct **N**on-interactive **AR**gument of **K**nowledge.
[^tvar]: We use $t$ for the polynomial variable to avoid collision with $x$, which is already the circuit input.
[^sz]: If $P$ and $Q$ are distinct polynomials of degree at most $d$ over $\mathbb{F}_p$, their difference $D = P - Q$ is non-zero with degree at most $d$, so it has at most $d$ roots. A random evaluation point lands on one of those roots with probability at most $d/p$. With $p \sim 2^{255}$ and $d$ in the thousands, this is negligible.
