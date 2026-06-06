# QCobalt Session Technical Summary — Combinatorial Optimization, QUBO, and the Traveling Salesman Problem

> Structured technical summary of a recorded QWorld / OQI bootcamp session (instructor: Jibran Rashid), reconstructed from the automatic transcript (`OQI26-1-8.vtt`) and the accompanying slide notes (`DIAPOSITIVAS5.md`). Spoken-language noise, logistics, and digressions have been removed; automatic-transcription errors have been corrected against the quantum-optimization context (e.g. "Kubo / Hugh Cobalt / Google" → **QUBO**, "QCobalt" → the course module name, "STP" → **SDP**, "NISC" → **NISQ**, "D-Waves" → **D-Wave**, "eta / beta" → slack variable $\eta$). Where the slides and the audio disagree, the slides take priority. Sections marked **“Enrichment”** add researched background that was implied but not fully developed live; everything else is faithful to the two source files.

---

## 1. Technical Sheet

- **Session topic:** Mathematical formulation of combinatorial optimization problems as **QUBO** (Quadratic Unconstrained Binary Optimization) — building the modeling layer needed before any quantum solver is introduced. Worked applications: **Max-Cut** and the **Traveling Salesman Problem (TSP)**.
- **Key concepts:** Combinatorial optimization & NP-completeness; QUBO objective $f(x)=x^\top Q x$; penalty method for constraints; integer→binary encoding with minimal qubit count; slack variables for inequalities.
- **Tools / Frameworks:** Pure mathematics for the formulation; **D-Wave Ocean SDK** (the `dimod` Binary Quadratic Model) is the target implementation platform for the Jupyter notebooks. This is the *third* quantum library introduced in the workshop (after the gate-model libraries used earlier). No Qiskit or Cirq in this particular session, and crucially **no quantum hardware is required** — everything today is classical modeling.
- **Location in the bootcamp:** Opening lecture of the **QCobalt** module — the “last leg” of the workshop, devoted to **optimization algorithms for the NISQ era**. It sits in the *Optimization* block of the QWorld arc (QUBO, Max-Cut, Ising model) and directly precedes the variational/annealing algorithms (the actual quantum solvers) that the course covers on the following days. It builds on the previous day’s session, where **Grover’s algorithm** was applied as a subroutine to Max-Cut.

---

## 2. Synopsis

This session opens the QCobalt module by establishing the mathematical scaffolding required to express hard combinatorial problems in a single canonical form that a quantum optimizer can ingest. It begins by situating combinatorial optimization at the intersection of combinatorics (exponentially large discrete solution spaces) and mathematical programming, distinguishing the tractable regimes — linear programming, semidefinite programming over positive-semidefinite matrices, and convex optimization — from the NP-complete problems (Max-Cut, TSP, SAT) that motivate quantum approaches. A deliberate caveat frames the whole module: quantum computers are not expected to solve NP-complete problems in polynomial time, and any claimed NISQ “advantage” must be benchmarked against a state-of-the-art classical algorithm fine-tuned for the same sub-class, not against brute force. The core construction is the QUBO model: minimize $f(x)=x^\top Q x$ over a binary vector $x\in\{0,1\}^n$, with $Q$ upper-triangular, exploiting $x_i^2=x_i$ to split the expression into linear (diagonal) and quadratic (off-diagonal) coefficients, and converting any maximization into minimization via $\max f \to \min(-f)$. From this foundation the lecture builds outward in three moves: (i) it re-expresses Max-Cut by encoding each edge with the XOR-like term $x_i+x_j-2x_ix_j$; (ii) it introduces constraints through additive penalty terms $P_i\,g_i(x)$, including a catalogue of canonical penalties and the squared-residual penalty for linear equalities, with slack variables $\eta$ absorbing inequalities; and (iii) it encodes bounded integers into a minimal number of binary variables using a special most-significant-bit construction that prevents overshooting the upper bound. It culminates in a full $n^2$-variable QUBO formulation of TSP, with row/column “visit-once” penalties and a wrap-around cost term enforcing a Hamiltonian cycle.

---

## 3. Subtopic Breakdown

### 3.1 Combinatorial Optimization, NP-Completeness, and the Honest NISQ Framing

The theoretical anchor of the session is the realization that the problems of interest live in **discrete solution spaces of exponential size**, where the task is to identify, among combinatorially many candidate structures, the one optimizing an objective. An optimization problem is stated generically as $\max/\min\ f(x)$ subject to constraints such as $Ax \le b$, and the *shape* of $f$ and its constraints determines the algorithmic regime. If $f$ is linear, the problem is a **linear program**, which is solvable optimally in polynomial time classically — so nothing quantum is needed there. If a matrix $A$ is required to be **positive semidefinite** (equivalently, all eigenvalues $\ge 0$), the problem becomes a **semidefinite program (SDP)**. More broadly, **convex optimization** admits good algorithms, but the combinatorial problems arising here are generally *non-convex*, which is precisely why they are hard. The canonical hard instances named are **Max-Cut**, **TSP**, and **SAT**; the instructor highlights that **SAT is the “poster child” NP-complete problem** and the universal target of reductions — to prove a new problem is hard, you reduce a known NP-complete problem (most reliably SAT) to it.

The pedagogical framing is unusually candid and worth preserving: the broad consensus is that **quantum computers will not solve NP-complete problems in polynomial time**. The realistic goal of the hackathon-oriented NISQ work is to find a *sub-class* of instances on which current hardware *might* show advantage. The instructor stresses a methodological trap: if you fine-tune your quantum hardware/parameters for a sub-class, you must compare against a classical algorithm *also* tuned for that sub-class — "apples to apples" — never against generic brute force. This session itself contains **nothing quantum**: it is entirely classical modeling. The connection to quantum solvers (why the QUBO form matters) is deferred to subsequent days.

*Enrichment:* QUBO is itself NP-hard, and many classical problems (maximum cut, graph coloring, number partitioning) have known QUBO embeddings; its importance for quantum computing comes from its close relationship to the **Ising model**, which is the native form solved by **quantum annealing** / adiabatic quantum computation.

### 3.2 The QUBO Model: Objective Function and the $Q$ Matrix

The central object is the QUBO objective
$$
f(x) = x^\top Q x = \sum_{i=1}^{n}\sum_{j=1}^{n} Q_{ij}\, x_i x_j,
$$
where $Q$ is taken (for convenience) as an **upper-triangular $n\times n$ matrix** and $x$ is a **binary vector**, $x_i\in\{0,1\}$. Because each variable is binary, the key algebraic identity $x_i^2 = x_i$ holds, which lets the quadratic form decompose cleanly into a **linear part** (the diagonal) and a **quadratic part** (the strictly upper-triangular entries):
$$
f(x) = \sum_{i} Q_{ii}\, x_i \;+\; \sum_{i<j} Q_{ij}\, x_i x_j.
$$
The "quadratic" in QUBO refers exactly to this second sum; the "unconstrained" reflects that, by definition, the bare model carries no side constraints (these are added later via penalties); and "binary" is the $\{0,1\}$ restriction on $x$. Two foundational moves are emphasized. First, **a QUBO is by default a minimization**, and any maximization is trivially recast as $\max f(x) \equiv \min\,(-f(x))$ — a generic sign trick that recurs throughout optimization. Second, the model **forbids terms of degree higher than two** (no cubics like $x_ix_jx_k$); handling such higher-order terms is postponed.

**Worked example (from the notebooks).** Given
$$
f(x_1,x_2,x_3,x_4) = -5x_1 - 3x_2 - 8x_3 - 6x_4 + 4x_1x_2 + 8x_1x_3 + 2x_2x_3 + 10x_3x_4,
$$
the matrix is $4\times 4$ (four variables — a brief live confusion arose because the slide momentarily wrote five variables and because indices started at $0$, but it resolves to four). Reading **linear coefficients onto the diagonal** and **quadratic coefficients onto the $(i,j)$ upper-triangular entries** yields
$$
Q = \begin{pmatrix} -5 & 4 & 8 & 0 \\ 0 & -3 & 2 & 0 \\ 0 & 0 & -8 & 10 \\ 0 & 0 & 0 & -6 \end{pmatrix}.
$$
There is no $x_1x_4$ term, so that entry is $0$; the entire lower triangle is $0$ by the upper-triangular convention. The takeaway: *modeling is just bookkeeping* — read the objective’s coefficients and populate $Q$.

*Enrichment (Ising connection, foreshadowing later days):* The reason the binary/quadratic restriction matters is that QUBO maps directly onto the Ising energy $E(s)=\sum_i h_i s_i + \sum_{i<j} J_{ij} s_i s_j$ with spins $s_i\in\{-1,+1\}$, via the linear change of variables $x_i = (1+s_i)/2$ (equivalently $s_i = 2x_i-1$). Linear QUBO terms become single-qubit $Z_i$ biases $h_i$ and quadratic terms become two-body $Z_iZ_j$ couplings $J_{ij}$. This is exactly the Hamiltonian a D-Wave annealer minimizes, which is *why* the course insists on the binary form.

### 3.3 Max-Cut as a QUBO, and Adding Constraints via Penalties

**Max-Cut formulation.** Building directly on the previous day’s Grover-based treatment, each vertex $i$ of the graph gets a binary variable $x_i$: $x_i=0$ places it in set $R$, $x_i=1$ places it in set $S$. An edge is “cut” when its endpoints lie in different sets — exactly the **parity / XOR** condition that the earlier quantum circuit computed with two CNOTs into an answer qubit. The arithmetized edge-check function reproducing the XOR truth table is
$$
x_i \oplus x_j \;\longmapsto\; x_i + x_j - 2x_ix_j,
$$
which returns $1$ exactly when $x_i\neq x_j$ and $0$ otherwise (verified on all four input pairs). This expression uses only linear and quadratic terms, so it is QUBO-legal. The instructor stresses that **this choice is not unique** — other functions satisfy the same truth table, and the “right” choice can depend on the problem; the absolute-value description is avoided precisely because it does not fit the QUBO shape, whereas this polynomial does. The full objective sums the edge term over all edges, so the sum **counts the cut size**, which Max-Cut maximizes; in minimization form:
$$
\min \sum_{(i,j)\in E} \big(-x_i - x_j + 2x_ix_j\big).
$$
For the slides’ specific 5-vertex graph (edges $(1,2),(1,3),(2,4),(3,4),(3,5),(4,5)$), expanding the sum and combining coefficients gives diagonal entries $(-2,-2,-3,-3,-2)$ — each $-1$ contribution counted once per incident edge — and six off-diagonal $+2$ entries, one per edge, with all remaining entries zero. That is the $Q$ matrix handed to the solver.

**Penalties for constraints.** Real problems carry constraints, but QUBO is unconstrained, so each constraint is folded into the objective as an additive **penalty**:
$$
f(x) + \sum_i P_i\, g_i(x),
$$
where $g_i(x)=0$ when constraint $i$ is satisfied and $g_i(x)>0$ when violated, and $P_i>0$ is the penalty coefficient (the actual amount added). The optimizer is thereby *incentivized* to satisfy the constraint to avoid inflating the minimized objective. A catalogue of canonical penalty functions for two binary variables was given (each easy to verify on $\{0,1\}^2$):

| Constraint | Penalty $g(x,y)$ |
| :--- | :--- |
| $x + y \le 1$ | $xy$ |
| $x + y \ge 1$ | $1 - x - y + xy$ |
| $x + y = 1$ | $1 - x - y + 2xy$ |
| $x \le y$ | $x - xy$ |
| $x = y$ | $x + y - 2xy$ |

Note the careful distinction between the “$\ge$” and “$=$” rows (the latter carries the $2xy$ term). For a **general linear equality** over integers, $\sum_{i=1}^k a_i y_i = b$, the penalty is the **squared residual**
$$
f(y_1,\dots,y_k) + P\left(\sum_{i=1}^k a_i y_i - b\right)^2,
$$
where the square is essential: with integers the residual can be positive or negative, but squaring makes the penalty non-negative and exactly zero at equality. For a **linear inequality** $\sum_i a_i y_i \le b$, introduce a **slack variable** $\eta \ge 0$ that absorbs the gap, turning the inequality into an equality $\sum_i a_i y_i + \eta = b$, after which the same squared-residual penalty applies. A “$\ge$” inequality is first multiplied by $-1$ to reach the standard “$\le$” form before adding slack.

*Choosing $P$ (the “Goldilocks” zone):* the penalty must be **large enough** that violating the constraint never improves the objective, but **not so large** that it numerically swamps the true objective and distorts the optimum. The right magnitude is dictated by the problem’s coefficient scales; intuition is built in the notebooks. *(Enrichment: a standard sufficient rule is to set $P$ larger than the largest achievable swing of the objective — in TSP below, $P > \max_{i,j} w(i,j)$ suffices.)*

### 3.4 Encoding Bounded Integers into a Minimal Number of Binary Variables

Because QUBO variables must be binary, integer variables must be **bit-encoded**, and — critically for quantum hardware, where each binary variable costs a **qubit** — with as *few* bits as possible. The relevant quantity is not the magnitude of the integer but the **number of distinct values** it can take, i.e. the gap between its lower and upper bounds $\underline{y}_i \le y_i \le \overline{y}_i$. The number of bits is
$$
N = \big\lceil \log_2(\overline{y}_i - \underline{y}_i + 1)\big\rceil,
$$
and the encoding is the special construction
$$
y_i = \underline{y}_i + \sum_{j=0}^{N-2} 2^j x_j^i + \left(\overline{y}_i - \underline{y}_i - \sum_{j=0}^{N-2} 2^j\right) x_{N-1}^i.
$$
The first term restores the constant offset (the lower bound). The middle sum is an ordinary binary expansion but **stops at the second-most-significant bit** ($j$ up to $N-2$, not $N-1$). The final term gives the **most significant bit** a *reduced* weight, chosen so that the maximum representable value never exceeds $\overline{y}_i$. Intuition: if the MSB is $0$, the value is fully determined by the standard lower bits; if the MSB is $1$, its special coefficient caps the total at the true upper bound rather than allowing a standard power-of-two overshoot.

**Worked example ($\underline{y}=50,\ \overline{y}=100,\ y=82$).** Here $N=\lceil\log_2 51\rceil = 6$ (50 possibilities → 6 bits). The five lower bits $j=0..4$ represent at most $31$; the MSB then carries weight $\overline{y}-\underline{y}-31 = 50-31 = 19$ (not $32$). So $82$ is reconstructed as $50$ (offset) $+ 19$ (MSB on) $+ 13$ (from the lower bits, $1101$), giving the **non-standard** bit string $y = 101101$. The point: this encoding deliberately differs from the ordinary binary representation precisely to **enforce the upper-bound constraint** ($\le 100$) inside the binary model.

*Slack-variable bounds.* The slack $\eta$ is likewise integer and must be bit-encoded; its range is
$$
0 \le \eta \le \left(b - \sum_{i=1}^{k} \min\{a_i \underline{y}_i,\, a_i \overline{y}_i\}\right).
$$
The lower bound is $0$ (no slack needed when the inequality is tight); the upper bound is read off by isolating $\eta$ in the equality and taking the most favorable values of each $y_i$.

**Full worked example (slides §6).** Minimize $f(y_1,y_2)$ subject to $0\le y_1\le 8$, $0\le y_2\le 5$, and $y_1 + y_2 \ge 10$.
1. **To equality:** multiply by $-1$ → $-y_1 - y_2 \le -10$; add slack → $-y_1 - y_2 + \eta = -10$.
2. **Penalized objective:** $f(y_1,y_2) + P(-y_1 - y_2 + \eta + 10)^2$.
3. **Slack bounds:** with $a_1=a_2=-1$, $b=-10$: $0\le\eta\le -10 - (\min\{0,-8\}+\min\{0,-5\}) = -10-(-13)=3$, so $0\le\eta\le 3$.
4. **Binary encodings:**
   - $y_1$: $N=\lceil\log_2 9\rceil = 4$, so $y_1 = \sum_{j=0}^{2} 2^j x_j^1 + (8-0-7)x_3^1 = \sum_{j=0}^{2} 2^j x_j^1 + x_3^1$.
   - $y_2$: $N=\lceil\log_2 6\rceil = 3$, so $y_2 = \sum_{j=0}^{1} 2^j x_j^2 + (5-0-3)x_2^2 = \sum_{j=0}^{1} 2^j x_j^2 + 2x_2^2$.
   - $\eta$: bounds $0..3$ → $N=2$, so $\eta = x_0^\eta + 2x_1^\eta$.

Each occurrence of $y_1, y_2, \eta$ in the penalized objective is then **substituted** by these expressions, eliminating all integer variables and leaving a pure QUBO. In practice (D-Wave Ocean / library solvers) these conversions are automated, but doing one by hand cements the mechanics.

### 3.5 The Traveling Salesman Problem as a QUBO

TSP is defined on a **directed, complete, weighted graph** $G=(V,E,w)$ with $V=\{0,\dots,n-1\}$ and weights $w(i,j)\in\mathbb{R}$ (with $w(i,j)=w(j,i)$ if undirected). The goal is the **shortest cycle visiting every vertex exactly once and returning to the start**; the start vertex is *not* given as input — once a cycle is found, any vertex can serve as the start. Missing edges are made present by assigning them a **prohibitively large weight** so they never appear in an optimal tour (a safe choice is a weight exceeding the sum of all real weights).

The formulation needs $\mathbf{n^2}$ **binary variables** $x_{i,t}$, where $x_{i,t}=1$ iff vertex $i$ is visited at time-step $t$ (with $0\le i,t < n$). The “time” index encodes the *order* of the visit — it is not literally time but the position in the tour — which is essential because the cost depends on consecutive transitions.

- **Total cost (objective to minimize):**
$$
\sum_{i,j=0}^{n-1} w(i,j) \sum_{t=0}^{n-1} x_{i,t}\, x_{j,t+1},
$$
with the **wrap-around convention** $x_{i,n} \equiv x_{i,0}$ so the cost of returning to the start node is counted (start vertex $=$ end vertex, enforcing a Hamiltonian cycle). The term $w(i,j)x_{i,t}x_{j,t+1}$ contributes exactly when the tour is at $i$ at step $t$ and at $j$ at step $t+1$; the inner sum runs over all positions, $i\neq j$.

- **Constraint 1 — each vertex visited exactly once:** $\sum_{t=0}^{n-1} x_{i,t} = 1$ for all $i$ ($n$ constraints), with penalty
$$
P\sum_{i=0}^{n-1}\left(1 - \sum_{t=0}^{n-1} x_{i,t}\right)^2.
$$

- **Constraint 2 — exactly one vertex visited at each time:** $\sum_{i=0}^{n-1} x_{i,t} = 1$ for all $t$, with the analogous penalty
$$
P\sum_{t=0}^{n-1}\left(1 - \sum_{i=0}^{n-1} x_{i,t}\right)^2.
$$
This second constraint prevents “cheating” by visiting two vertices in parallel at the same step.

- **Penalty magnitude:** choose $P > \max_{i,j} w(i,j)$ (strictly larger than any edge weight), so that violating a constraint can never be cheaper than taking a real edge — otherwise a “race condition” would let the optimizer break feasibility to save cost.

The slides’ small worked instance is a directed 3-vertex graph with $w(0{\to}1)=15,\ w(1{\to}0)=7,\ w(0{\to}2)=10,\ w(2{\to}0)=9,\ w(1{\to}2)=8,\ w(2{\to}1)=14$.

*Enrichment (the classical/quantum complexity remark, corrected and sourced):* the instructor noted that the best **classical** exact method for TSP is essentially brute force (Bellman–Held–Karp dynamic programming runs in $O(2^n)$ time), and that the **best known quantum algorithm runs in roughly $1.78^n$** rather than $2^n$. That speedup comes **not** from a NISQ/QUBO approach but from combining classical **dynamic programming** with **Grover-style amplitude amplification** for a quadratic improvement — the Ambainis et al. quantum dynamic-programming result for vertex ordering / Hamiltonian-path problems. The honest moral: a NISQ QUBO encoding is a convenient *modeling* device, but does not by itself confer the proven speedup; expect quantum advantage only when genuine structural insight is added.

---

## 4. Points of Confusion and Corner Cases

- **“Unconstrained” does not mean “no constraints allowed.”** A bare QUBO has no side constraints, but real constraints are reintroduced as **penalties** inside the objective. The subtlety students miss is that $g_i(x)$ is the *penalty term*, not the constraint itself: it must equal $0$ when satisfied and be $>0$ when violated. The instructor corrected himself live — the violated value need not be exactly $1$; it is simply $>0$, and the exact value depends on the chosen penalty function.
- **Choosing the penalty coefficient $P$ is genuinely hard.** Too small and the optimizer happily violates the constraint; too large and it overpowers the real objective, returning a feasible-but-wrong answer. There is a problem-dependent “Goldilocks” window. For TSP a clean rule exists ($P>\max w(i,j)$), but in general it must be tuned.
- **The integer→binary encoding is deliberately *not* the standard binary representation.** The reduced-weight most-significant-bit term exists solely to stop the encoding from representing values above the integer’s upper bound. In the $50\!-\!100$ example, $82$ encodes as $101101$, not as the ordinary binary of $32$ plus offset. Students frequently expect a textbook binary expansion and are surprised.
- **Read $x_j^i$ as indices, not exponents.** Superscripts label which variable ($i$) and subscripts which bit ($j$); $x_3^1$ is “bit 3 of variable 1,” never “$x$ cubed.” Likewise $N$ is computed *per integer variable*, not once for the whole problem.
- **Index base and variable count.** A momentary slide slip wrote “5 variables” for a 4-variable objective; combined with $0$-based indexing this caused confusion about the size of $Q$. Always count distinct variables, not the largest index.
- **The “time” index in TSP is order, not clock time**, and **two constraints are needed**, not one: “each vertex once” (sum over $t$) *and* “one vertex per step” (sum over $i$). Omitting the second permits illegal parallel visits. The **wrap-around** $x_{i,n}=x_{i,0}$ is what turns a path into a cycle and accounts for the return-to-start cost.
- **Missing edges in TSP** must be added with very large weights to satisfy the solver’s completeness assumption; “large” is *relative* to the existing weights (an order of magnitude higher, or larger than the sum of all weights, guarantees exclusion).
- **No higher-than-quadratic terms.** The model rejects cubic (and higher) terms; the fact that a particular penalty (e.g. $(x_1-x_2)^2$) happens to expand into an XOR-like quadratic is circumstantial, not a requirement. Handling true higher-order terms is deferred.
- **Nothing in this session is quantum.** A recurring clarification: the graphs, data structures, and QUBO formulation are pure classical computer science. The link to quantum solvers (why QUBO is the chosen form) arrives only on later days.

---

## 5. For Study (Flashcards)

1. **Q:** What three properties define a QUBO problem? — **A:** A **quadratic** objective $f(x)=x^\top Qx$ (only linear + pairwise terms, no cubics), **unconstrained** in its bare form (constraints added via penalties), over **binary** variables $x_i\in\{0,1\}$; by default a minimization.
2. **Q:** Why can $f(x)=x^\top Qx$ be split into diagonal (linear) and off-diagonal (quadratic) parts? — **A:** Because $x_i\in\{0,1\}\Rightarrow x_i^2=x_i$, so each diagonal term $Q_{ii}x_i^2$ collapses to $Q_{ii}x_i$, a linear coefficient.
3. **Q:** Which polynomial encodes the Max-Cut “edge-cut” (XOR) check, and what does the full objective compute? — **A:** $x_i+x_j-2x_ix_j$ (=1 iff endpoints differ); summed over all edges it equals the **cut size**, maximized (or $\min\sum(-x_i-x_j+2x_ix_j)$).
4. **Q:** How is a linear inequality $\sum_i a_i y_i \le b$ turned into a QUBO penalty? — **A:** Add a non-negative **slack** $\eta$ to get the equality $\sum_i a_i y_i + \eta = b$, then apply the squared-residual penalty $P(\sum_i a_i y_i + \eta - b)^2$.
5. **Q:** How many binary variables does the QUBO TSP need, what does $x_{i,t}$ mean, and what are the two constraints? — **A:** $n^2$ variables; $x_{i,t}=1$ iff vertex $i$ is visited at step $t$; constraints: each vertex exactly once ($\sum_t x_{i,t}=1\ \forall i$) and exactly one vertex per step ($\sum_i x_{i,t}=1\ \forall t$), with cost using $x_{i,n}\equiv x_{i,0}$ for the return-to-start cycle.
6. **Q:** Why use the special integer→binary encoding with a reduced most-significant-bit weight instead of plain binary? — **A:** To use the **minimum number of bits/qubits** ($N=\lceil\log_2(\overline{y}-\underline{y}+1)\rceil$) *and* to **cap** the representable value at the integer’s upper bound, preventing overshoot.
7. **Q:** What is the best known quantum time for exact TSP, and where does the speedup come from? — **A:** About $1.78^n$ (vs. classical $\sim 2^n$), achieved by classical **dynamic programming** plus **Grover/amplitude amplification** — not from a NISQ/QUBO method.

---

*Sources: session transcript `OQI26-1-8.vtt` and slide notes `DIAPOSITIVAS5.md`. Enrichment items (Ising mapping, penalty bounds, $1.78^n$ TSP result, D-Wave Ocean/BQM context) are standard, well-established results added per the request to expand and research the material; they are consistent with, and extend, the content actually delivered in the session.*
