+++
title = "Zero-Knowledge: Constructing a SNARK Proof (Part 2/3)"
date = 2026-04-17
description = "A trusted setup, eight curve commitments, and five pairing checks: how to turn a polynomial identity into a working zero-knowledge SNARK."

[taxonomies]
tags = ["crypto", "computer-science"]

[extra]
katex = true
social_media_card = "/img/zk-proofs-part2-banner.webp"
+++

![Twin origami cranes engraved with pairing equations, meeting at a luminous focal beam](/img/zk-proofs-part2-banner.webp)

Equation (3) from [Part 1](@/blog/zk-computation-to-polynomials.md) reduced a program to a single polynomial check: for any random $\tau$, a valid witness satisfies

$$L(\tau) \cdot R(\tau) - O(\tau) = H(\tau) \cdot Z(\tau)$$

That is a clean algebraic statement, but it is not yet a proof. Two gaps remain. The prover must evaluate at a $\tau$ they do not know, and the evaluation must come from the QAP's actual polynomials, not arbitrary ones cooked up to pass the check.

This post closes both gaps, and a few more we'll uncover along the way. By the end, we'll have a full SNARK: eight curve points, five pairing checks, and a proof that reveals nothing about the witness.

## Moving Evaluation into Curve Space

The first gap closes by hiding $\tau$ inside curve points. A **trusted setup ceremony** picks a random scalar $\tau$ and publishes its powers in two prime-order elliptic-curve groups:

$$G_1,\ \tau G_1,\ \tau^2 G_1,\ \ldots,\ \tau^d G_1$$

in $\mathbb{G}_1$, plus the parallel powers $G_2, \tau G_2, \ldots, \tau^d G_2$ in $\mathbb{G}_2$, then destroys the scalar itself. A group of participants each contributes randomness to $\tau$, and as long as one of them destroys their share honestly, nobody can reconstruct it. The destroyed randomness is called **toxic waste**. What survives is the list of points above: the **Common Reference String (CRS)**, following the same pattern as the [trusted setup](@/blog/verkle-trees-polynomial-commitments.md#trusted-setup) from the Verkle post.

![Trusted setup burns τ into the CRS rails of G₁ and G₂ points; four polynomial coefficients pair with gold points and converge into a wax-sealed commitment P(τ)·G₁](/img/zk-trusted-setup-and-commitment.webp)

Anyone who holds the coefficients of a polynomial $P$ can now compute $P(\tau) \cdot G_1$ without learning $\tau$: multiply each published $\tau^k G_1$ by the coefficient $p_k$ and sum. The result is a **commitment** to the polynomial's value at the hidden point, and it is the core tool that lets the prover hand the verifier witness-dependent values without revealing the witness.

That CRS will keep growing through the rest of this post as new checks call for new fixed points. Every addition follows the same recipe: an expression in the secret scalars, evaluated during the ceremony and published only as a curve point.

## Binding the Commitments to the QAP Polynomials

Let's start with three commitments: one per aggregate polynomial $L$, $R$, and $O$. Take $\pi_A$, the commitment to $L(\tau)$, which per Part 1 should be computed as the witness-weighted sum of per-variable polynomials $L_i$:[^letters]

$$\pi_A = L(\tau) \cdot G_1 = \sum_i s_i \cdot L_i(\tau) \cdot G_1$$

where $s_i$ are the witness values. The **Knowledge of Exponent (KoE)** assumption is what forces the prover to publish a $\pi_A$ computed using the equation above.

Given a pair $(P, \alpha P)$, a curve point and itself shifted by an unknown scalar $\alpha$, KoE says the only way to produce another pair with the same shift is to scale the original pair by a second known scalar $c$, as $(cP, c \cdot \alpha P) = (cP, \alpha \cdot cP)$.[^koe] KoE extends naturally to linear combinations: publish $(P_1, \alpha P_1), \ldots, (P_n, \alpha P_n)$ and the prover can return another $\alpha$-shifted pair $(C, \alpha C)$ if and only if $C = \sum_j c_j P_j$ for coefficients $c_j$ the prover knows.

So the prover **must** build from the published $P_j$ values, which is the property we need to bind $\pi_A$ to the $L_i$. Since both prover and verifier know the $L_i$ polynomials, the setup can hardcode them into the CRS: for each variable $i$, it publishes the pair $\big(L_i(\tau) \cdot G_1,\ \alpha_A \cdot L_i(\tau) \cdot G_1\big)$ with a secret scalar $\alpha_A$. The prover combines these pairs with witness weights $s_i$, using the plain halves to form $\pi_A$ and the $\alpha_A$-shifted halves to form $\pi_A'$.

With these $\alpha$-shifted pairs and $\alpha_A \cdot G_2$ also in the CRS, the verifier can run a pairing check confirming $(\pi_A, \pi_A')$ has the right shift:

$$e(\pi_A',\ G_2) = e(\pi_A,\ \alpha_A \cdot G_2)$$

A **pairing** $e(P, Q)$ takes two curve points and produces an element of a multiplicative target group $\mathbb{G}_T$. Bilinearity gives $e(aP, bQ) = e(P, Q)^{ab}$, so both sides above reduce to $e(\pi_A, G_2)^{\alpha_A}$, and the check passes iff $\pi_A' = \alpha_A \cdot \pi_A$.[^pairings]

The same construction with fresh scalars $\alpha_B$ and $\alpha_C$ gives $(\pi_B, \pi_B')$ and $(\pi_C, \pi_C')$. $\pi_B$ is built in $G_2$ rather than $G_1$ so it can later pair with $\pi_A$, while $\pi_C$ stays in $G_1$.

In summary:

$$\begin{aligned}
\pi_A &= \sum_i s_i \cdot L_i(\tau) \cdot G_1 \\\\
\pi_B &= \sum_i s_i' \cdot R_i(\tau) \cdot G_2 \\\\
\pi_C &= \sum_i s_i'' \cdot O_i(\tau) \cdot G_1
\end{aligned} \tag{1}$$

with α-shifted companions:

$$\begin{aligned}
\pi_A' &= \alpha_A \cdot \pi_A \\\\
\pi_B' &= \alpha_B \cdot \pi_B \\\\
\pi_C' &= \alpha_C \cdot \pi_C
\end{aligned} \tag{2}$$

Nothing yet forces $s_i = s_i' = s_i''$: we could have three candidate witnesses passed in parallel.

## Binding the Commitments to a Single Witness

A prover who passes every check without holding a single coherent witness hasn't proved anything. The fix is a fourth commitment $\pi_K$ the verifier uses to check whether the three witness sets agree.

For each variable $i$, the CRS publishes a single point that bundles all three sides under a secret $\beta$:

$$\beta \big(L_i(\tau) + R_i(\tau) + O_i(\tau)\big) \cdot G_1$$

The prover scales each bundle with a single weight $r_i$ per variable to form:

$$\begin{aligned}
\pi_K = \beta \cdot G_1 \cdot \sum_i r_i \cdot \big({}&L_i(\tau) + R_i(\tau) \\\\
&{}+ O_i(\tau)\big) \tag{3}
\end{aligned}$$

With $\beta \cdot G_1$ and $\beta \cdot G_2$ also added to the CRS, the verifier runs a pairing check:

$$\begin{aligned}
e(\pi_K, G_2) &= e(\pi_A + \pi_C,\ \beta \cdot G_2) \\\\
&\quad \cdot e(\beta \cdot G_1,\ \pi_B)
\end{aligned}$$

$\pi_B$ pairs separately since it lives in $G_2$ while $\pi_A, \pi_C$ stay in $G_1$. By bilinearity, both sides reduce to the same power of $e(G_1, G_2)$. Matching exponents, canceling β, and equating per term in the sum:

$$\begin{aligned}
&r_i \cdot \big(L_i(\tau) + R_i(\tau) + O_i(\tau)\big) \\\\
&\quad = s_i \cdot L_i(\tau) + s_i' \cdot R_i(\tau) \\\\
&\quad\quad {} + s_i'' \cdot O_i(\tau), \quad \forall i
\end{aligned}$$

Assuming the per-variable polynomials $L_i, R_i, O_i$ are linearly independent, the check pushes $r_i$ to match each of $s_i, s_i', s_i''$ for every $i$, forcing the first three commitments to collapse onto a single witness.

$\alpha$ binds each proof element to the QAP polynomials; β binds the three elements to the same witness. We now have a consistent prover. One gap remains: their witness must satisfy the gate constraints.

## Binding the Commitments to a Satisfying Witness

Part 1 showed that the quotient $H(t) = (L(t) \cdot R(t) - O(t)) / Z(t)$ exists as a polynomial iff every gate is satisfied, where $Z$ is the target polynomial vanishing at all gate roots. The prover commits $H(\tau)$ using the CRS powers of $\tau$:

$$\pi_H = H(\tau) \cdot G_1 = \sum_j h_j \cdot (\tau^j \cdot G_1) \tag{4}$$

Once $Z(\tau) \cdot G_2$ joins the CRS, the verifier runs:

$$e(\pi_A,\ \pi_B) = e(\pi_H,\ Z(\tau) \cdot G_2) \cdot e(\pi_C,\ G_2)$$

Both sides reduce to powers of $e(G_1, G_2)$; by bilinearity, matching exponents gives:

$$L(\tau) \cdot R(\tau) = H(\tau) \cdot Z(\tau) + O(\tau)$$

That is the QAP identity $L \cdot R - O = H \cdot Z$ evaluated at $\tau$, now lifted into curve space. By [Schwartz-Zippel](@/blog/zk-computation-to-polynomials.md#what-one-equation-buys-us), if the identity holds at a hidden random $\tau$, it holds as polynomials with overwhelming probability.

## A Working Core, Two Gaps Left

Equations (1) through (4) supply the eight commitments shown on the left, wired to the five pairing checks on the right:

![Eight proof-element wax seals (six gold G₁, two silver G₂) wired to five pairing-check clauses, color-coded by family — KoE in gold, β-consistency in terracotta, divisibility in crimson](/img/zk-snark-architecture.webp)

What we have is a working SNARK, modulo a CRS-level patch that closes a remaining soundness gap (ρ, covered in the appendix). Two gaps remain on the main path:

- **Public inputs.** Nothing yet pins the claimed output to what the verifier expects. The next section adds a public/private split.
- **Zero knowledge.** The commitments could leak information about the witness. The final section adds blinding to close that gap.

## Pinning the Public Variables

A typical SNARK claim reads "this program, run on my private inputs, produces this public output." The verifier has the output; they want a proof the run was honest. The five pairing checks, though, accept *any* consistent witness: the prover can pick whatever output they like and still pass.

Fix it by splitting the witness vector so the verifier owns the public half:

$$\mathbf{s} = [\underbrace{s_0, s_1, \ldots, s_\ell}\_{\text{public}} \mid \underbrace{s_{\ell+1}, \ldots, s_n}\_{\text{private}}]$$

The public half holds the constant 1 and whatever values the verifier needs visible, usually the claimed output. The private half holds everything else. From here on, $\pi_A$ sums only over private indices; $\pi_B$ and $\pi_C$ stay as defined, since one anchor on the public side is enough:[^aside]

$$\pi_A = \sum_{i \text{ private}} s_i \cdot L_i(\tau) \cdot G_1$$

The missing public half falls to the verifier, and the CRS already has the $L_i(\tau) \cdot G_1$ they need (from the KoE setup). The verifier picks the public-index ones and combines them with the claimed values on their side:

$$L_\text{pub}(\tau) \cdot G_1 = \sum_{i \text{ public}} s_i \cdot L_i(\tau) \cdot G_1$$

Wherever the verification equations used the full $L(\tau) \cdot G_1$, they now use $L_\text{pub}(\tau) \cdot G_1 + \pi_A$: the verifier's half plus the prover's, reassembled on the curve.

The divisibility check becomes:

$$\begin{aligned}
e\big(L_\text{pub}(\tau) \cdot G_1 + \pi_A,\ \pi_B\big) ={}& e(\pi_H,\ Z(\tau) \cdot G_2) \\\\
&\cdot e(\pi_C,\ G_2)
\end{aligned}$$

Same argument as before forces the QAP identity, now on $L = L_\text{pub} + L_\text{priv}$: it holds iff $\pi_A, \pi_B, \pi_C, \pi_H$ were computed from a witness matching the verifier's $L_\text{pub}$. The verifier now has visibility into the output the prover claims: a proof passes only when it's computed against that exact claim.

## Blinding the Commitments

Everything above is **sound**: any proof the verifier accepts means a valid witness exists for the claim. But soundness doesn't imply privacy. $\pi_A, \pi_B, \pi_C$ are deterministic functions of the witness, so anyone with a candidate witness in mind can recompute $\pi_A$ from the public CRS and check it against the published proof: match confirms the guess, mismatch rules it out. Zero-knowledge demands the proof reveal *nothing* about the witness.

The fix: randomize each commitment with a shift the verification equations absorb. The prover picks fresh $\delta_L, \delta_R, \delta_O$ per proof:[^threedeltas]

$$\begin{aligned}
\pi_A &= L(\tau) \cdot G_1 + \delta_L \cdot Z(\tau) \cdot G_1 \\\\
\pi_B &= R(\tau) \cdot G_2 + \delta_R \cdot Z(\tau) \cdot G_2 \\\\
\pi_C &= O(\tau) \cdot G_1 + \delta_O \cdot Z(\tau) \cdot G_1
\end{aligned}$$

The divisibility check already uses $Z(\tau) \cdot G_2$. Adding $Z(\tau) \cdot G_1$ to the CRS lets the prover form each shift on the curve.

Divisibility survives. Expand the products and substitute $L \cdot R = H \cdot Z + O$ for the bare $LR$ term. The $O$ cancels with $-O$, leaving everything multiplied by $Z$:

$$\begin{aligned}
&\underbrace{(L + \delta_L Z)}\_{L_{\text{new}}}\underbrace{(R + \delta_R Z)}\_{R_{\text{new}}} - \underbrace{(O + \delta_O Z)}\_{O_{\text{new}}} \\\\
&\quad = Z \cdot \underbrace{(H + \delta_L R + \delta_R L - \delta_O + \delta_L \delta_R Z)}\_{H_{\text{new}}}
\end{aligned}$$

The prover now computes $\pi_H = H_{\text{new}}(\tau) \cdot G_1$. The α- and β-checks balance by the same trick: the CRS adds matching shifts ($\alpha_A Z(\tau) G_1$, $\beta Z(\tau) G_1$, etc.) and the companion elements pick them up.

![Same witness yields three identical gold wax-seal commitments without blinding (attacker's lens matches), versus three distinct seals with independent blinding droplets (attacker's lens sees only a fog of candidates)](/img/zk-blinding.webp)

For any fixed witness, $\pi_A, \pi_B, \pi_C$ are uniformly distributed in their groups (δ is uniform, $Z(\tau) \neq 0$). The attacker who tried recomputing $\pi_A$ from a candidate witness now sees only noise: eight curve points, five pairing checks, and zero bits about the witness.

## What's Next

Together with the ρ patch, the scheme we just derived is **Pinocchio/BCTV14** (2013–2014), the construction Zcash shipped with its original shielded pool in 2016. Modern refinements have pushed the same pattern much further: a private-payment circuit runs about 100,000 constraints and fits in a proof a few hundred bytes long; a zkEVM rollup aggregates tens of millions of constraints and lands around the same size after wrapping.[^wrapping]

Part 3 will pick up two threads. First, how modern protocols (Groth16, PLONK, STARKs) reshape this pattern to shrink proofs, replace per-circuit setup with a universal one, or drop pairings entirely. Second, what production ZK systems prove in practice: Zcash's shielded transfers, zkEVM rollups, and newer applications.

---

## Appendix

Two appendices: ρ (linear dependence among the $L_i, R_i, O_i$) and a reference card in literature notation.

<details>
<summary>Why ρ is needed</summary>

The body's β-check argument relied on linear independence of $\{L_i, R_i, O_i\}$. That assumption doesn't hold in practice: with $N$ variables and $d$ gates, the $3N$ per-variable polynomials live in a $d$-dimensional space, so once $3N > d$ (most circuits), linear relations are the norm.

**What fails when independence fails.** The β-check reduces to:

$$\begin{aligned}
&\sum_i (r_i - s_i) L_i(t) \\\\
&\quad + (r_i - s_i') R_i(t) \\\\
&\quad + (r_i - s_i'') O_i(t) \equiv 0
\end{aligned}$$

Under independence, the only solution is $s_i = s_i' = s_i'' = r_i$: a unique witness. Dependence opens non-trivial solutions; divisibility rejects some but not all, and any that passes both checks is an invalid proof the verifier accepts.

**The ρ fix.** Two secret scalars $\rho_A$ and $\rho_B$ scale the three sides differently: every $L_i$ becomes $\rho_A L_i$, every $R_i$ becomes $\rho_B R_i$, every $O_i$ becomes $\rho_A \rho_B O_i$. The commitments pick up the tags:

$$\begin{aligned}
\pi_A &= \rho_A L(\tau) \cdot G_1 \\\\
\pi_B &= \rho_B R(\tau) \cdot G_2 \\\\
\pi_C &= \rho_A \rho_B O(\tau) \cdot G_1
\end{aligned}$$

The β-identity, rewritten with these tags:

$$\begin{aligned}
&\sum_i (r_i - s_i) \rho_A L_i(t) \\\\
&\quad + (r_i - s_i') \rho_B R_i(t) \\\\
&\quad + (r_i - s_i'') \rho_A \rho_B O_i(t) \equiv 0
\end{aligned}$$

Since $\rho_A, \rho_B$ are secret and algebraically independent, distinct ρ-monomials can't cancel: each coefficient must vanish separately. That forces $r_i = s_i$ (from $\rho_A$), $r_i = s_i'$ (from $\rho_B$), $r_i = s_i''$ (from $\rho_A \rho_B$). Single witness, restored.[^within]

The other checks stay balanced. The α-checks scale uniformly with $\rho_A$ or $\rho_B$ on each side, so the KoE relations still hold. Divisibility picks up a common $\rho_A \rho_B$ factor on both sides: the CRS publishes $\rho_A \rho_B Z(\tau) \cdot G_2$ instead of $Z(\tau) \cdot G_2$, and the factor cancels.

</details>

<details>
<summary>Reference: full BCTV14 specification in literature notation</summary>

Papers use bracket notation $[x]_j = x \cdot G_j$ and call the per-variable polynomials $A_i, B_i, C_i$ rather than $L_i, R_i, O_i$. Greek scalars ($\alpha, \beta, \gamma, \rho, \delta, \tau$) keep their names across the literature. This appendix restates the full protocol in that notation and maps each piece back to the body.

Assumes ρ (from the earlier appendix) and γ are live. Under those, body expressions pick up ρ-factors (the body's $\pi_A$ becomes $\rho_A L_\text{priv}(\tau) \cdot G_1$) and the β-check is γ-gated. The map below translates this post's notation to the literature's.

**Notation map.** Left column is the post's notation; right column is what you'll see in BCTV14, Groth16, and most derivatives.

{% table(wide=true) %}
| This post | Literature | What it is |
|---|---|---|
| $L_i, R_i, O_i$ | $A_i, B_i, C_i$ | per-variable polynomials |
| $L, R, O$ | $A, B, C$ | aggregate (witness-weighted) polynomials |
| $P(\tau) \cdot G_j$ | $[P(\tau)]_j$ | commit to $P$ in group $j$ |
| $\pi_A$ (private-only) | $[A\_{\text{mid}}]_1$ | prover's A-side commitment |
| $\pi_B, \pi_C$ | $[B]_2, [C]_1$ | B-side and C-side commitments |
| $\pi_A', \pi_B', \pi_C'$ | $[A'\_{\text{mid}}]_1, [B']_1, [C']_1$ | α-shifted companions (all in $G_1$) |
| $\pi_K$ | $[K]_1$ | β-bundle combination tying the witness across L, R, O |
| $\pi_H$ | $[H]_1$ | quotient commitment |
| $L_\text{pub}(\tau) \cdot G_1$ | $\mathbf{vk}\_x$ | verifier's public-input reconstruction |
| $\mathbf{s}$ | $\mathbf{a}$, $\mathbf{w}$ (varies) | witness / assignment vector |
| $Z(\tau)$ | $Z(\tau)$, $t(\tau)$ (varies) | target polynomial at τ |
{% end %}

One subtle point on $\pi_B'$: the body writes $\pi_B' = \alpha_B \pi_B \in G_2$ for brevity, while BCTV14 puts $[B']_1$ in $G_1$ via a separate KoE pair. Both check the same α-shift relation; the PK below carries the $G_1$ elements BCTV14's construction needs.

**Setup secrets** (destroyed after the ceremony):

- $\tau$: hidden evaluation point
- $\alpha_A, \alpha_B, \alpha_C$: KoE shifts
- $\beta$: witness-consistency scalar
- $\gamma$: β-check gate[^gamma]
- $\rho_A, \rho_B$: tag L, R, O distinctly so cross-family terms in the β-identity can't cancel

The prover picks $\delta_L, \delta_R, \delta_O$ fresh per proof.

**Proving key** (used by the prover):

- Powers of $\tau$: $[\tau^k]_1, [\tau^k]_2$ for $k = 0, \ldots, d$
- Per private variable $i$:
  - A-side: $[\rho_A A_i(\tau)]_1, [\alpha_A \rho_A A_i(\tau)]_1$
  - B-side:
    - $[\rho_B B_i(\tau)]_1$
    - $[\rho_B B_i(\tau)]_2$
    - $[\alpha_B \rho_B B_i(\tau)]_1$
  - C-side: $[\rho_A \rho_B C_i(\tau)]_1, [\alpha_C \rho_A \rho_B C_i(\tau)]_1$
  - β-bundle: $[\beta(\rho_A A_i(\tau) + \rho_B B_i(\tau) + \rho_A \rho_B C_i(\tau))]_1$

**Verification key** (used by the verifier):

- $[1]_2$
- $[\alpha_A]_2, [\alpha_B]_1, [\alpha_C]_2$
- $[\gamma]_2, [\beta\gamma]_1, [\beta\gamma]_2$
- $[\rho_A \rho_B Z(\tau)]_2$
- Per public variable $i$ (including $i = 0$ for the constant 1): $\mathbf{vk}\_{\text{IC},i} = [\rho\_A A\_i(\tau)]\_1$. The verifier forms $\mathbf{vk}\_x = \sum\_{i \in \text{pub}} s\_i \cdot \mathbf{vk}\_{\text{IC},i}$.

**Proof** (8 group elements):

$$\begin{aligned}
\pi = \big( &[A\_{\text{mid}}]_1,\ [A'\_{\text{mid}}]_1,\ [B]_2,\ [B']_1, \\\\
&[C]_1,\ [C']_1,\ [K]_1,\ [H]_1 \big)
\end{aligned}$$

**Verification equations** (5 pairing checks):

1. **α for A** (KoE): $e([A'\_{\text{mid}}]_1, [1]_2) = e([A\_{\text{mid}}]_1, [\alpha_A]_2)$
2. **α for B** (KoE): $e([B']_1, [1]_2) = e([\alpha_B]_1, [B]_2)$
3. **α for C** (KoE): $e([C']_1, [1]_2) = e([C]_1, [\alpha_C]_2)$
4. **β-consistency (γ-gated):** $e([K]_1, [\gamma]_2) = e([A\_{\text{mid}}]_1 + [C]_1, [\beta\gamma]_2) \cdot e([\beta\gamma]_1, [B]_2)$
5. **Divisibility:** $e(\mathbf{vk}\_x + [A\_{\text{mid}}]_1, [B]_2) = e([H]_1, [\rho_A \rho_B Z(\tau)]_2) \cdot e([C]_1, [1]_2)$

For the exact ZK-blinding CRS elements and public-input encoding, see libsnark's `r1cs_ppzksnark` reference implementation.

</details>

---

[^letters]: Pinocchio and BCTV14 write the per-variable polynomials as $A_i, B_i, C_i$. This post uses $L_i, R_i, O_i$ to stay consistent with Part 1 and with the aggregate $L, R, O$.
[^aside]: Splitting all three sides would triple the per-public-input CRS points without adding leverage.
[^pairings]: For a worked example showing the derivation from polynomial identity to bilinear check, see the Verkle post's [verifying the proof with pairings](@/blog/verkle-trees-polynomial-commitments.md#verifying-the-proof-with-pairings) section.
[^koe]: KoE goes beyond standard discrete-log hardness. DL hardness says "you can't find $\alpha$ given $\alpha G$", which is falsifiable: anyone can try. KoE says something stronger: *whenever* a prover returns a correctly α-shifted pair $(C, \alpha C)$, there exists an algorithm (an **extractor**) that recovers the coefficients the prover used to build $C$. That's a claim about the existence of an algorithm, not a computational bound, so no experiment can disprove it. If KoE breaks, SNARKs break.
[^threedeltas]: Three independent scalars, not one shared. A shared δ would correlate the commitments: solve for δ from one, predict the others.
[^wrapping]: zkEVMs typically prove the bulk of their work with a STARK (fast, no trusted setup, large proof), then re-prove that STARK verification inside a Groth16 circuit. The "wrap" collapses the large STARK proof down to a small on-chain one.
[^within]: The per-monomial identities require each of the three families ($L_i$, $R_i$, $O_i$, each taken over all $i$) to be linearly independent on its own. Internal dependence within a family (e.g., among the $L_i$) can still exist, but it's only a concern when the dependence involves public-input indices or crosses the public/private boundary, which QAP construction rules out via mechanisms outside this post's scope.
[^gamma]: γ is a CRS-generation artifact of the soundness reduction inherited from GGPR/Pinocchio, not a protocol-level defense. The full story is in [GGPR §5.3](https://eprint.iacr.org/2012/215.pdf) (EUROCRYPT 2013; ePrint 2012/215); in the generic group model γ isn't strictly needed ([Gabizon 2019](https://eprint.iacr.org/2019/119.pdf)).
