# QWorld Bootcamp — Session 7: Solving Max-Cut via Grover's Algorithm

*Instructor: Jibran Rashid · Module: QNickel (the bridge between QBronze and QCobalt)*

> **How to read this document.** Sections 1–5 are the faithful summary of the session itself, built strictly from the lecture transcript and slide deck. Sections 6–10 are an added **research layer** that situates the session in the wider literature — every external claim there is cited. Where the lecture and outside sources are interleaved, external facts are marked *[Research context]* so the session's own content stays cleanly identifiable.

---

## Table of Contents

1. Technical Summary Card
2. Synopsis
3. Topic Breakdown
4. Confusion Points & Edge Cases
5. Flashcard Review
6. Research Context: Grover, Amplitude Amplification & Optimal Iterations
7. Research Context: The Classical Frontier for Max-Cut
8. Research Context: Where This Sits Among Quantum Optimizers (QAOA / VQE)
9. Research Context: "Quantum Advantage Without Structure" — The Debate the Instructor Referenced
10. Method Comparison, Glossary & References

---

## 1. Technical Summary Card

- **Session topic:** Applying Grover's unstructured search as a generic subroutine to the Max-Cut combinatorial optimization problem — including the "inversion around the mean" interpretation of Grover, the explicit circuit for the diffusion operator, and the full oracle construction for Max-Cut.
- **Key concepts:** Grover diffusion operator (inversion around the mean); Max-Cut and its decision version; oracle construction via edge-checking; decision→optimization reduction via binary search; NP-completeness and the quadratic quantum speedup.
- **Tools / Frameworks:** Conceptual circuit construction — Hadamard layers, multi-controlled-$Z$, CNOT-based edge checks, quantum adders, phase kickback, and uncomputation. The associated notebooks/assessments use **Google's Cirq** library rather than Qiskit.
- **Position in bootcamp:** Seventh session, drawn from the optional **QNickel** module. Positioned deliberately as the transition from the introductory **QBronze** content to the applied, hackathon-oriented **QCobalt** material. Max-Cut returns later under variational algorithms (QAOA / VQE). The QNickel assessments are optional and not graded toward the Bronze or Cobalt certificates.

---

## 2. Synopsis

This session completes the treatment of Grover's algorithm and then repurposes it as a generic subroutine for a genuinely hard combinatorial problem. It opens by revisiting the core Grover step through a second, complementary lens: rather than viewing the iteration purely as a rotation between "good" and "bad" subspaces, the Grover operator is decomposed as $G = U Z_f$, where $U = -H^{\otimes n} Z_0 H^{\otimes n} = 2|h\rangle\langle h| - I$ acts as an *inversion around the mean*. Applied to an arbitrary superposition, $U$ sends each amplitude $\alpha_x$ to $2\mu - \alpha_x$, where $\mu$ is the mean amplitude. Starting from a uniform superposition with a single marked item, the oracle's sign flip pushes the marked amplitude from $-1/\sqrt{N}$ up to roughly $3/\sqrt{N}$ after one step, growing additively until the success probability peaks — after which continued iteration *reduces* it, making the stopping point critical. The session then pivots to Max-Cut: given an undirected graph, partition the vertices into two sets so as to maximize the number of crossing edges. Its decision form ("is there a cut of size at least $k$?") is NP-complete, and brute force scans all $2^n$ partitions. Grover yields a quadratic improvement to $\mathcal{O}(\sqrt{2^n})$. The remainder builds the oracle concretely — encoding vertices as qubits, checking edges via parity, summing satisfied edges, comparing the sum to $k$, and applying a phase via kickback — first for the easy bipartite case, then generalized to arbitrary graphs with a binary-search outer loop. A recurring meta-theme frames the hackathon: a generic Grover speedup uses no problem structure, and genuine quantum advantage demands identifying exploitable structure in the underlying problem.

---

## 3. Topic Breakdown

### 3.1 Grover as Inversion Around the Mean

The session begins by finishing an idea left open in the previous session: an alternative interpretation of Grover's algorithm. Recall the three steps — prepare $|0^n\rangle$ and apply $H^{\otimes n}$; apply the Grover operator $G = -H^{\otimes n} Z_0 H^{\otimes n} Z_f$ some $k$ times; then measure. The previous session treated the work step as a *rotation* in the two-dimensional subspace spanned by the good and bad strings. Here the same step is recast algebraically by isolating the transformation
$$U = -H^{\otimes n} Z_0 H^{\otimes n} = 2|h\rangle\langle h| - I,$$
so that $G = U Z_f$. The key bookkeeping subtlety is that the overall minus sign — previously absorbed together with $Z_f$ — is now kept inside $U$. In Dirac form $U$ is the reflection $2|h\rangle\langle h| - I$, and in matrix form it is the all-ones matrix scaled by $2/N$ minus the identity:
$$U = \frac{2}{N}\begin{pmatrix}1 & \cdots & 1\\ \vdots & \ddots & \vdots\\ 1 & \cdots & 1\end{pmatrix} - I.$$

Applying $U$ to an arbitrary state $\sum_x \alpha_x |x\rangle$ and pulling $U$ inside the sum, the all-ones matrix maps each $|x\rangle$ to the mean of all amplitudes while the identity returns $\alpha_x$ itself, giving the amplitude update
$$U\left(\sum_x \alpha_x |x\rangle\right) = \sum_x (2\mu - \alpha_x)|x\rangle, \qquad \mu = \frac{1}{N}\sum_x \alpha_x.$$
This is why the operation is called *inversion around the mean*: every amplitude is reflected through $\mu$. The visualization makes the dynamics concrete. After step one all $2^n$ amplitudes sit uniformly at $1/\sqrt{N}$ (one bar, the target $x^*$, highlighted). Applying $Z_f$ flips the marked amplitude's sign:
$$Z_f|x^*\rangle = (-1)^{f(x^*)}|x^*\rangle = -|x^*\rangle.$$
The mean $\bar{\alpha}$ then drops slightly below $1/\sqrt{N}$, but with a single marked item among many it is still well approximated by $1/\sqrt{N}$. The subsequent inversion lifts the marked amplitude:
$$2\mu - \alpha_{x^*} = 2\left(\tfrac{1}{\sqrt{N}}\right) - \left(-\tfrac{1}{\sqrt{N}}\right) \approx \frac{3}{\sqrt{N}},$$
and a further iteration would carry it to $5/\sqrt{N}$, and so on, until the numerator reaches roughly $\sqrt{N}$ and the success probability is near one. Crucially, the approximation $\mu \approx 1/\sqrt{N}$ breaks once most amplitude has concentrated on $x^*$; past the optimum the amplitude begins to *shrink* again — the amplitude-amplification mirror of over-rotating past the target in the rotation picture. Identifying the correct number of iterations is therefore essential. *(See §6 for the closed-form optimal iteration count from the literature.)*

### 3.2 Implementing the Diffusion Operator

Before moving to applications, the session opens the two black boxes that a real run of Grover must physically implement: the oracle and the diffusion operator $-H^{\otimes n} Z_0 H^{\otimes n}$ (equivalently, up to sign conventions, $I - 2|0^n\rangle\langle 0^n|$). The diffusion operator is built by sandwiching the $Z_0$ reflection between Hadamard layers, where
$$Z_0|x\rangle = \begin{cases} -|x\rangle & \text{if } x = 0^n \\ |x\rangle & \text{otherwise.}\end{cases}$$
The circuit realizing $Z_0$ is a three-layer construction: a layer of Pauli-$X$ gates on every qubit, a multi-controlled-$Z$, then another layer of $X$ gates. The argument that this implements $Z_0$ proceeds by cases. When the incoming string is $0^n$, the first $X$ layer flips every qubit to $1$, so all controls of the multi-controlled-$Z$ are satisfied and the $Z$ fires on the (now $|1\rangle$) target; since $Z|1\rangle = -|1\rangle$, exactly the desired global $-1$ phase is acquired, and the final $X$ layer restores the original $0^n$. For any other string, at least one qubit is $0$ before the controlled-$Z$: if it is a control, $Z$ never fires and the $X$ layers cancel (identity); if it happens to be the target, then $Z|0\rangle = |0\rangle$ leaves the state unchanged. Either way non-zero strings pass through untouched, which is precisely $Z_0$. Wrapping this in Hadamards yields the diffusion operator — an instance of a multi-controlled-operation trick that recurs throughout quantum circuit design.

### 3.3 Max-Cut, Its Decision Version, and the Decision-to-Optimization Reduction

A detour into graph theory sets up the problem. An undirected graph is a pair $G = (V, E)$ with vertices $V = \{v_1, \dots, v_n\}$ and edges as unordered pairs $E = \{(v_i, v_j) \mid v_i, v_j \in V\}$; this is the CS/data-structure sense of "graph," not a plotted curve. A *cut* is a partition of $V$ into two non-empty sets $S$ and $R = V - S$, and the *size of a cut* is defined over edges, not vertices: it is the number of edges crossing between $S$ and $R$. **Max-Cut** asks for the partition maximizing this crossing count. The worked example fixes a 5-vertex graph where $S = \{1, 2, 5\}$ and $R = \{3, 4\}$ gives $f(1)=f(2)=f(5)=1$, $f(3)=f(4)=0$, and **size of cut = 5** (the maximum, non-uniquely achieved). This introduces the coloring function $f$ mapping each vertex to $0$ or $1$ according to its partition.

Because complexity classes like NP concern decision problems, Max-Cut is reformulated as: *given $G$, does there exist a cut of size at least $k$?* This decision version is **NP-complete**, with $2^n$ possible partitions/colorings, and the related Min-Cut (via max-flow/min-cut) is in P — a distinction worth keeping separate. The optimization answer is recovered from the decision oracle by **binary search**: ask whether a cut of size at least $L/2$ exists; on "no" search downward toward $L/4$, on "yes" search upward toward $3L/4$, recursing to pin down the optimum with only logarithmic overhead. This same logarithmic overhead would appear in the classical setting too. Since the best known classical Max-Cut for arbitrary graphs is essentially brute force at $\mathcal{O}(2^n)$, Grover delivers a quadratic improvement to $\mathcal{O}(\sqrt{2^n})$ — respectable, though not the exponential-to-polynomial leap one might hope for.

### 3.4 Oracle Construction: Edge Checking, the Bipartite Case, and the General Sum-and-Compare

The heart of the practical content is building $Z_f$ for Max-Cut. Information is encoded by assigning $n$ qubits to $n$ vertices: $|0\rangle$ means a vertex is colored red (set $R$), $|1\rangle$ means blue (set $S$); e.g. $|v_1 v_2 v_3 v_4\rangle = |1011\rangle$ encodes a specific coloring. The fundamental primitive is **edge checking**: an edge contributes to the cut only when its two endpoints differ in color. Mapping $00 \to 0$, $11 \to 0$, $01 \to 1$, $10 \to 1$, this is exactly parity (XOR), $e_{12} = v_1 \oplus v_2$, implemented with two CNOTs from the two vertex qubits into an auxiliary edge qubit initialized in $|0\rangle$.

The construction is first presented for **bipartite graphs**, where $V$ splits into two sets with no internal edges, so *every* edge crosses the cut and the max-cut size equals the total number of edges. For a concrete bipartite graph with 4 vertices and 3 edges, the oracle: applies $H^{\otimes n}$ to the vertex qubits to superpose all $2^n$ colorings; runs edge checks $e_{02}, e_{03}, e_{13}$ (three edges, two CNOTs each, six CNOTs total); then — because in the bipartite case a valid coloring must light up *all* edge qubits — applies a multi-controlled-NOT onto an answer qubit followed by a $Z$, using phase kickback to stamp a $-1$ on correctly-colored states; and finally *uncomputes* the edge checks with $U^\dagger$ to free the auxiliary qubits, leaving only the phase. The net effect is $Z_f$ with $Z_f|x\rangle = (-1)^{f(x)}|x\rangle$ over the four vertex qubits. However, since deciding bipartiteness (and hence its max-cut) is classically solvable in $\mathcal{O}(n^2)$ while the quantum routine is $\mathcal{O}(\sqrt{2^n})$, this is only a *proof of concept*, not a genuine advantage.

The **general construction** drops the assumption that all edges must cross. After the Hadamard layer and edge checks, the circuit must *count* how many edge qubits are set: a quantum adder sums the edge-check outputs into sum qubits (for 3 edge qubits the maximum sum is 3, needing 2 sum qubits), built from the same carry/half-adder logic as classical digital design — "nothing quantum" about the arithmetic itself. A **number-checking** comparator then tests whether the sum is $\ge k$; for a 3-bit sum, checking $k \ge 100_2$ (i.e. $\ge 4$) reduces to inspecting the most significant bit, while checks like $k = 010_2$ or $k \ge 110_2$ are realized with configured Toffoli and $X$ gate patterns. If the sum meets the threshold, a $Z$ completes $Z_f$. This whole circuit becomes the oracle inside Grover, run $\sqrt{N}$ times; and because the optimum is found by binary search over thresholds, each node of the search tree is a *separate* Grover run, giving the quadratic speedup with an additional logarithmic factor. With multiple optimal cuts, Grover returns *one* satisfying solution (uniformly among them), not all of them — the standard multi-marked-item behavior.

---

## 4. Confusion Points & Edge Cases

The most important friction point the instructor returns to repeatedly is the timing of Grover iterations. Amplitude amplification is not monotone: the marked amplitude grows additively (from $-1/\sqrt{N}$ to about $3/\sqrt{N}$, then $5/\sqrt{N}$, and so on) only while the mean is still well approximated by $1/\sqrt{N}$. Once most of the weight has concentrated on the target, that approximation fails and further iterations *decrease* the success probability — exactly analogous to over-rotating past the target state in the rotation picture. Learners who assume the amplitude simply saturates at one and stays there will badly misjudge the optimal iteration count.

A second cluster of confusion concerns the definition of a cut and its size. The size of a cut is counted over *edges that cross between $S$ and $R$*, not over vertices in either set — a distinction the session emphasizes explicitly because it is easy to start counting vertices by reflex. Closely related is the decision-versus-optimization distinction: NP-completeness is a statement about the *decision* version ("is there a cut of size at least $k$?"), since the class NP is defined over decision problems, and the optimization version is reached only through the binary-search reduction. The session also flags the literature's NP-hard-versus-NP-complete phrasing as a point that trips people up, and warns against conflating Max-Cut (NP-complete) with the superficially similar Min-Cut / max-flow problem, which is solvable in polynomial time.

The bipartite special case is itself a pedagogical trap. It is tempting to declare quantum advantage after building a working bipartite oracle, but the instructor is emphatic that this is only a proof of concept: bipartiteness and its max-cut are classically decidable in $\mathcal{O}(n^2)$, so the $\mathcal{O}(\sqrt{2^n})$ quantum routine is strictly worse there. Genuine advantage requires the "Goldilocks" regime — a restricted problem subclass that is still classically hard yet exposes structure a quantum algorithm can exploit. This ties to the broadest cautionary theme: a generic Grover substitution uses no problem structure, and a well-tuned classical heuristic will often match or beat it. The Mark Zhandry example — a structureless problem that appeared to give exponential quantum advantage — is offered as a reminder to scrutinize any advantage claim, especially for hackathon proposals. *(§9 unpacks what this example actually is in the literature.)*

Finally, two implementation subtleties deserve care. Phase kickback requires the answer qubit to be properly prepared (in $|-\rangle$) so that the $Z$ imprints the intended $-1$ on satisfying colorings, and the auxiliary edge qubits must be *uncomputed* (via $U^\dagger$) so they are disentangled and reusable — skipping the cleanup leaves garbage entangled with the register. And in the general oracle, the symbol $k$ shifts meaning between the input threshold of the decision problem and the running comparison value at each node of the binary-search tree; the comparison is always against the *left-hand side* sum (the size of the cut found), with the fixed threshold on the right.

---

## 5. Flashcard Review

**Q:** In the "inversion around the mean" view, what is the amplitude update rule applied by $U = 2|h\rangle\langle h| - I$?
**A:** Each amplitude $\alpha_x$ becomes $2\mu - \alpha_x$, where $\mu = \frac{1}{N}\sum_x \alpha_x$ is the mean amplitude — i.e., every amplitude is reflected through the mean.

**Q:** Starting from a uniform superposition with one marked item, what is the marked amplitude after one Grover iteration, and why does iterating too long hurt?
**A:** It rises from $-1/\sqrt{N}$ to about $2(1/\sqrt{N}) - (-1/\sqrt{N}) \approx 3/\sqrt{N}$. Iterating past the optimum decreases it again, because the approximation $\mu \approx 1/\sqrt{N}$ fails once amplitude concentrates on the target (over-rotation).

**Q:** How is the size of a cut defined, and what does Max-Cut optimize?
**A:** The size of a cut is the number of edges crossing between the two partition sets $S$ and $R$ (counted over edges, not vertices). Max-Cut seeks the partition that maximizes this crossing count.

**Q:** How does edge checking work, and what circuit implements it?
**A:** An edge contributes only when its endpoints have different colors, i.e. $e_{ij} = v_i \oplus v_j$ (parity). It is implemented with two CNOTs from the two vertex qubits into an auxiliary edge qubit, so the auxiliary becomes $1$ iff the endpoints differ.

**Q:** Why does the general (non-bipartite) oracle need a summation step that the bipartite oracle does not, and what is the resulting complexity?
**A:** In a bipartite graph every edge crosses, so a valid coloring lights up *all* edge qubits and a multi-controlled-$Z$ suffices. For general graphs only some edges cross, so the circuit must sum the satisfied edge qubits and compare the total to $k$. Grover then gives $\mathcal{O}(\sqrt{2^n})$ versus the classical $\mathcal{O}(2^n)$, with a logarithmic binary-search overhead since each threshold is a separate Grover run.

**Q:** What three steps turn a predicate into a Grover oracle, and why is the last one needed?
**A:** Compute the predicate, apply a $Z$ (phase kickback) to the answer qubit, then *uncompute* the predicate. Uncomputation disentangles and resets the auxiliary qubits so they can be reused and so the interference in the diffusion step works correctly.

**Q:** Why is solving Max-Cut on bipartite graphs *not* a real quantum advantage?
**A:** Bipartiteness (and thus its max-cut, which equals the total edge count) can be decided classically in $\mathcal{O}(n^2)$, far faster than the $\mathcal{O}(\sqrt{2^n})$ Grover routine. It is only a proof of concept for the oracle structure.

---

## 6. Research Context: Grover, Amplitude Amplification & Optimal Iterations

Grover's algorithm, introduced by Lov Grover in 1996, is the canonical quantum routine for unstructured search: it finds a marked item among $N$ possibilities in $\mathcal{O}(\sqrt{N})$ oracle queries versus $\mathcal{O}(N)$ classically.[^grover-wiki][^ibm-grover] Unlike Shor's factoring algorithm, the speedup is only **quadratic, not exponential**, and this quadratic factor is provably optimal for a true black-box search.[^grover-wiki] The session's "inversion around the mean" picture is the standard textbook geometric description: the Grover iterate is a product of two reflections (oracle reflection $Z_f$, then the diffusion reflection about the uniform state), which composes to a rotation in a two-dimensional plane.[^azure-grover]

The session stressed that stopping at the right moment is critical but did not give the closed form. The literature does: for $M$ marked items among $N$, the optimal number of iterations is
$$N_{\text{optimal}} = \left\lfloor \frac{\pi}{4}\sqrt{\frac{N}{M}} - \frac{1}{2} \right\rfloor,$$
and for a single marked item this is $\lfloor \frac{\pi}{4}\sqrt{N} \rfloor$.[^azure-grover][^iters-se] The full operator applied to the register is $\left(-H^{\otimes n} O_0 H^{\otimes n} O_f\right)^{N_{\text{optimal}}} H^{\otimes n}$, matching the session's $G = -H^{\otimes n} Z_0 H^{\otimes n} Z_f$ exactly.[^azure-grover] Two practical caveats from the wider literature reinforce the session's cautions: the success probability is *periodic* in the iteration count (over-rotation genuinely lowers it), and for the special case $N = 4$ a single iteration finds the answer with certainty.[^ibm-iters] One further engineering note worth flagging for the hackathon: the headline $\sqrt{N}$ advantage does **not** account for quantum error-correction overhead, which in fault-tolerant estimates can erode the effective speedup (sometimes described as closer to quartic in realistic regimes).[^quera-grover] Oracles also rely on **uncomputation** — the exact $U^\dagger$ cleanup the session showed — which roughly doubles the gate count but is required to avoid leftover entanglement with ancilla qubits.[^quera-grover]

*[Research context]* The general Grover-for-Max-Cut approach the session sketched has been formalized: a 2023 arXiv result gives a quantum Max-Cut algorithm for arbitrary graphs with temporal complexity $\mathcal{O}(\sqrt{2^n})$ (and spatial complexity $\mathcal{O}(m^2)$ in edges), and argues this is optimal among oracle-based quantum algorithms for NP-complete problems — precisely the quadratic-speedup regime described in lecture.[^qmaxcut-arxiv]

---

## 7. Research Context: The Classical Frontier for Max-Cut

The session framed the classical baseline as essentially brute force ($\mathcal{O}(2^n)$ exact). That is true for *exact* solution, but the practical classical story — highly relevant to the instructor's "a tuned classical heuristic often wins" warning — is richer.

- **Equivalence to maximum bipartite subgraph.** A maximum cut corresponds exactly to a maximum-size bipartite subgraph of $G$; finding it is NP-hard in general.[^maxcut-wiki] This is the formal backbone of the session's observation that bipartite graphs are the "easy" case.
- **Polynomially solvable special classes.** Beyond bipartite graphs (trivial: the answer is all edges), Max-Cut is polynomial-time solvable for **planar graphs** and, more generally, **weakly bipartite graphs** (a class that contains both bipartite and planar graphs), solvable via the ellipsoid method and odd-cycle constraints.[^maxcut-wiki][^weakly-bipartite] This is exactly the "find exploitable structure" lesson, made concrete: the right structural promise can move Max-Cut from intractable to tractable *classically*.
- **The Goemans–Williamson (GW) approximation.** The most celebrated classical result is the 1995 Goemans–Williamson algorithm, which uses **semidefinite programming (SDP) plus randomized rounding** to produce, in polynomial time, a cut whose expected size is at least $\alpha_{GW} \approx 0.87856$ times the optimum.[^gw-paper][^maxcut-wiki] Under the Unique Games Conjecture this ratio is the best any polynomial-time algorithm can achieve; unconditionally, it is NP-hard to approximate Max-Cut better than $16/17 \approx 0.941$.[^maxcut-wiki] Recent work (2025) shows the standard SDP can be pushed slightly beyond $\alpha_{GW}$ on certain graph families.[^triangles]

The takeaway for a hackathon team: any quantum Max-Cut proposal is competing not against naive brute force but against a 0.878-approximation that runs in polynomial time and against polynomial *exact* algorithms on structured instances. This is precisely why the instructor insisted that a generic Grover speedup, which ignores all problem structure, rarely translates into real-world advantage.

---

## 8. Research Context: Where This Sits Among Quantum Optimizers (QAOA / VQE)

The instructor previewed that "tomorrow" (QCobalt) shifts to optimization algorithms of the NISQ era — QAOA and variational quantum eigensolvers — whose advantage, unlike Grover's, is *not* provable. Some context for that transition:

- **QAOA.** The Quantum Approximate Optimization Algorithm (Farhi, Goldstone, Gutmann, 2014) is the leading variational approach to Max-Cut. It is a **hybrid classical-quantum** method: a shallow parameterized circuit (alternating cost and mixer layers) prepares a trial state, a classical optimizer tunes the parameters to maximize the expected cut value, and the best-sampled bitstring is returned.[^cirq-qaoa][^pennylane-qaoa] Its shallow circuits make it attractive for NISQ hardware, but — as the instructor flagged — performance guarantees are largely heuristic.[^qaoa2-arxiv]
- **Cirq connection.** Notably, Google's **Cirq** (the library used in this session's QNickel notebooks) ships an official QAOA-for-Max-Cut tutorial that runs on the Sycamore hardware graph, so the same problem the session solves with Grover is the standard pedagogical target for QAOA too.[^cirq-qaoa] This makes Max-Cut a natural through-line from QNickel into QCobalt.
- **Scaling tricks.** Because NISQ devices have few qubits, divide-and-conquer variants such as **QAOA-in-QAOA (QAOA²)** decompose large Max-Cut instances into subgraphs solved in parallel, and have been reported as competitive with classical methods around the ~2000-node scale in simulation.[^qaoa2-applied][^qaoa2-arxiv] There is also a NISQ scheme solving an $n$-vertex Max-Cut with only $\log(n)$ qubits.[^log-qubits]

In short: Grover gives a *provable* quadratic speedup but no structural leverage; QAOA/VQE chase *heuristic* advantage on near-term hardware. The bootcamp's arc deliberately moves from the former to the latter.

---

## 9. Research Context: "Quantum Advantage Without Structure" — The Debate the Instructor Referenced

The instructor mentioned that Mark Zhandry produced a *structureless* problem that seemed to give an exponential quantum advantage, and recalled hearing that the advantage was later overturned by a classical algorithm. Here is what the literature actually contains, so you can judge the claim:

- The relevant result is **Yamakawa & Zhandry, "Verifiable Quantum Advantage without Structure"** (FOCS 2022; *Journal of the ACM*, 2024). It exhibits a search problem that is easy for quantum computers but hard for classical ones, instantiable from any sufficiently "random-looking" cryptographic hash function — i.e., **without** the number-theoretic structure (period finding) underlying Shor-type speedups.[^yz-jacm][^zhandry-pubs] As Zhandry put it, essentially all prior NP-style quantum advantages reduced to period finding, and this gave "at least a second case," suggesting quantum advantage may be more widespread than thought.[^ntt]
- This sits against a backdrop of important *negative* results — "**dequantization**" — where claimed quantum speedups (notably in quantum machine learning, following Ewin Tang's work) were matched by new classical algorithms.[^dequant] That broad pattern is exactly the instructor's cautionary point, even if the specific attribution is worth double-checking.

*[Verify before citing in a talk]* I did **not** find a clear published result showing the *Yamakawa–Zhandry* problem itself was dequantized "last year." The instructor's recollection may be conflating the Yamakawa–Zhandry advantage (which, as of these sources, stands) with the general dequantization trend. The safe, well-supported statement is: *structureless verifiable quantum advantage has been demonstrated in an oracle/cryptographic-hash setting, but the field has repeatedly seen apparent speedups erased by better classical algorithms, so advantage claims warrant scrutiny.* Treat the "it vanished" detail as unverified.

---

## 10. Method Comparison, Glossary & References

### 10.1 Approaches to Max-Cut at a glance

| Approach | Type | Complexity / guarantee | Notes |
| :--- | :--- | :--- | :--- |
| Brute force | Classical exact | $\mathcal{O}(2^n)$ | Scan all partitions; the session's baseline. |
| Grover-based (this session) | Quantum exact (w/ binary search) | $\mathcal{O}(\sqrt{2^n})$, $\times \log$ overhead | Generic quadratic speedup; uses no problem structure. |
| Goemans–Williamson | Classical approximation | Poly-time, $\ge 0.878 \times$ optimum | Optimal ratio under UGC; SDP + randomized rounding.[^gw-paper] |
| Exact on planar / weakly bipartite | Classical exact | Polynomial time | Structure makes it tractable.[^weakly-bipartite] |
| Bipartite Max-Cut | Classical exact | $\mathcal{O}(n^2)$ | Answer = total edge count; the session's "easy" case. |
| QAOA / QAOA² | Hybrid variational (NISQ) | Heuristic, no proof | Shallow circuits; Cirq tutorial targets exactly this.[^cirq-qaoa] |

### 10.2 Glossary

- **Amplitude amplification** — the general technique (Grover is the canonical case) of boosting the amplitude of marked states via repeated reflections; quadratic speedup.[^ibm-grover]
- **Diffusion operator** — $-H^{\otimes n} Z_0 H^{\otimes n}$ (a.k.a. $2|h\rangle\langle h| - I$); the "inversion about the mean" reflection.
- **Phase kickback** — using an ancilla in $|-\rangle$ so that a controlled operation deposits a $(-1)$ phase on the control register rather than flipping the ancilla.
- **Uncomputation** — applying $U^\dagger$ to reset/disentangle ancilla qubits after the phase has been imprinted; roughly doubles gate count.[^quera-grover]
- **Decision vs. optimization** — NP-completeness is defined for decision problems ("cut $\ge k$?"); optimization is recovered by binary search over $k$.
- **Bipartite graph** — vertices 2-colorable with no monochromatic edge; Max-Cut equals the total edge count and is poly-time solvable.[^maxcut-wiki]
- **NISQ** — Noisy Intermediate-Scale Quantum; the current hardware regime that motivates shallow variational algorithms like QAOA.
- **Unique Games Conjecture (UGC)** — a complexity conjecture under which $0.878$ is the best achievable Max-Cut approximation ratio.[^maxcut-wiki]

### 10.3 References

[^grover-wiki]: Grover's algorithm — Wikipedia. https://en.wikipedia.org/wiki/Grover%27s_algorithm
[^ibm-grover]: Grover's algorithm — IBM Quantum tutorials. https://quantum.cloud.ibm.com/docs/tutorials/grovers-algorithm
[^azure-grover]: Theory of Grover's search algorithm — Microsoft Azure Quantum. https://learn.microsoft.com/en-us/azure/quantum/concepts-grovers
[^iters-se]: "Why does the optimal number of iterations involve a floor?" — Quantum Computing Stack Exchange. https://quantumcomputing.stackexchange.com/questions/1939/
[^ibm-iters]: Choosing the number of iterations — IBM Quantum Learning. https://quantum.cloud.ibm.com/learning/courses/fundamentals-of-quantum-algorithms/grover-algorithm/number-of-iterations
[^quera-grover]: Grover's Algorithm — QuEra glossary (error-correction overhead, uncomputation). https://www.quera.com/glossary/grovers-algorithm
[^qmaxcut-arxiv]: "Quantum Speedup for the Maximum Cut Problem" — arXiv:2305.16644. https://arxiv.org/pdf/2305.16644
[^maxcut-wiki]: Maximum cut — Wikipedia (GW ratio, hardness of approximation, planar/poly cases). https://en.wikipedia.org/wiki/Maximum_cut
[^gw-paper]: Goemans & Williamson, "Improved Approximation Algorithms for Maximum Cut and Satisfiability Problems Using Semidefinite Programming," JACM 1995. http://www-math.mit.edu/~goemans/PAPERS/maxcut-jacm.pdf
[^weakly-bipartite]: Grötschel & Pulleyblank, "Weakly bipartite graphs and the Max-cut problem," Oper. Res. Lett. 1981. https://www.sciencedirect.com/science/article/pii/0167637781900201
[^triangles]: "Triangles Improve the 0.878 Approximation for Maxcut" — APPROX/RANDOM 2025. https://drops.dagstuhl.de/entities/document/10.4230/LIPIcs.APPROX/RANDOM.2025.27
[^cirq-qaoa]: QAOA: Max-Cut — Google Quantum AI / Cirq. https://quantumai.google/cirq/experiments/qaoa/qaoa_maxcut
[^pennylane-qaoa]: QAOA for MaxCut — PennyLane Demos. https://pennylane.ai/demos/tutorial_qaoa_maxcut
[^qaoa2-arxiv]: "Hybrid Classical-Quantum Simulation of MaxCut using QAOA-in-QAOA" — arXiv:2406.17383. https://arxiv.org/html/2406.17383v1
[^qaoa2-applied]: "QAOA-in-QAOA: Solving Large-Scale MaxCut Problems on Small Quantum Machines" — Phys. Rev. Applied 19, 024027. https://link.aps.org/doi/10.1103/PhysRevApplied.19.024027
[^log-qubits]: Rančić, "NISQ algorithm for an n-vertex MaxCut with log(n) qubits" — Phys. Rev. Research 5, L012021. https://link.aps.org/doi/10.1103/PhysRevResearch.5.L012021
[^yz-jacm]: Yamakawa & Zhandry, "Verifiable Quantum Advantage without Structure," JACM 2024. https://dl.acm.org/doi/10.1145/3658665
[^zhandry-pubs]: Mark Zhandry — Quantum publications (structureless advantage from hash functions). https://mzhandry.github.io/pubs.quantum.html
[^ntt]: "Yamakawa and Zhandry Advance Verifiable Quantum Advantage" — NTT Research. https://ntt-research.com/yamakawa-and-zhandry-advance-verifiable-quantum-advantage/
[^dequant]: Ewin Tang et al., "A Classical Algorithm Framework for Dequantizing Quantum Machine Learning" (talk). https://www.youtube.com/watch?v=j4k6aVihQXE
