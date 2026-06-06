# QWorld Bootcamp — Technical Session Summary
## From QUBO to the Ising Model: Higher-Order Reductions, Max-3SAT, and Variational Quantum Algorithms (VQA / VQE)

> Structured, cleaned, and expanded technical notes built from the automatic transcript of the recorded session (speaker: *Jibran Rashid*) and the accompanying slide deck. Spoken-language noise, logistics, and digressions have been removed; transcription errors have been corrected against the quantum-computing context; and textual descriptions of states, operators, and matrices have been rewritten in precise Dirac/linear-algebra notation. Where the slides and the audio disagree, the slides take precedence. Sections marked *Extended context* add researched background that goes beyond the raw sources, at the user's explicit request.

---

## 1. Technical Sheet

- **Session topic:** Bridging classical optimization formulations and quantum hardware — reducing higher-order (super-quadratic) terms to QUBO, formulating Max-3SAT as QUBO, introducing Variational Quantum Algorithms (VQA) and the Variational Quantum Eigensolver (VQE), the physics of observables and measurement, the variational principle, and the equivalence between the QUBO model and the Ising model (illustrated through Max-Cut).
- **Key concepts:** QUBO & higher-order term reduction (auxiliary variable + Rosenberg-type penalty); Max-3SAT clause penalties; Hamiltonians, observables, and expectation values; the variational principle and VQE; the Ising model ↔ QUBO equivalence (Max-Cut).
- **Tools / Frameworks:** Conceptual/mathematical (linear algebra, Dirac notation); references to **D-Wave Ocean SDK** and the **Binary Quadratic Model (BQM)** for the hands-on notebooks; Qiskit-style measurement reasoning (basis changes via Hadamard). No live coding was shown in this session — the labs are deferred to the notebooks.
- **Location in the bootcamp:** Optimization block (QUBO → Max-Cut → Ising model) transitioning into the variational-algorithms block (VQA / VQE), and serving as the conceptual bridge to **quantum annealing / the quantum adiabatic algorithm**, which is announced as the next day's topic. This is a follow-up session: it explicitly continues from a prior day in which the Traveling Salesman Problem (TSP) was formulated as a QUBO.

---

## 2. Synopsis

This session completes the optimization arc of the bootcamp and opens the door to quantum algorithms for optimization. It begins by closing a gap left open the previous day: arbitrary optimization problems can contain monomials of degree higher than two, yet the target model — QUBO — admits only linear and quadratic terms because the underlying hardware supports only **2-local** (pairwise) interactions between qubits. A general degree-reduction procedure is presented: select a pairwise product $x_i x_j$ inside the highest-degree term, replace it with a fresh binary variable $y_{ij}$, and append a penalty that forces $y_{ij}=x_i x_j$ at optimality. This machinery is immediately exercised on **Max-3SAT**, where each clause — classified by how many of its literals are negated (zero, one, two, or three) — is translated into a polynomial $g(x)$ that equals $1$ exactly when the clause is satisfied; maximizing $\sum_i g_i(x)$ over all clauses yields the optimization objective, whose cubic terms are then quadratized by the same penalty trick. The narrative then pivots from *formulation* to *solution*: Variational Quantum Algorithms are introduced as the hybrid quantum–classical paradigm of the NISQ era, in which a parameterized circuit prepares a state, measurement yields a cost, and a classical optimizer updates the parameters in a feedback loop. The Variational Quantum Eigensolver specializes this idea to finding ground-state energies, motivated by the chemistry intuition of minimizing molecular energy and grounded in the linear algebra of Hermitian operators, eigenvalues as measurement outcomes, expectation values $\langle\psi|H|\psi\rangle$, and the **variational principle** $E_0 \le \langle\psi|H|\psi\rangle$. Finally, the **Ising model** is shown to have an energy function identical in form to QUBO; via the change of variables $x_i=\tfrac{1-S_i}{2}$, QUBO and Ising are proven equivalent — Max-Cut being the worked example — giving the mathematical objective an honest physical substrate that a quantum device can actually realize.

---

## 3. Subtopic Breakdown

### 3.1 Reducing Higher-Order Terms to QUBO

**Theoretical foundation.** A QUBO (Quadratic Unconstrained Binary Optimization) objective is a polynomial in binary variables $x_i\in\{0,1\}$ containing only linear and quadratic monomials. The session stresses *why* the restriction to degree two is not arbitrary: the quantum hardware and the quantum algorithms targeted later are motivated by a **2-local** representation, in which, at any instant, the machine can couple at most two individual particles (qubits) at a time. That pairwise coupling is exactly a quadratic term, so cubic and higher-order monomials have no native hardware counterpart. A general optimization problem, however, may well contain such terms — the slides give the progression from linear ($x_1+x_2-x_3$), to quadratic ($x_1x_2+x_2x_3-x_3x_1-3$), to cubic ($x_1x_2x_3+x_2x_4x_5-x_6x_1x_2$). The speaker is careful to flag that QUBO is *one* possible format among many (one could choose a formulation that is not restricted to quadratic terms, or solve the same formulation classically); QUBO is chosen precisely because it matches the hardware and the quantum algorithm to come.

**How it was carried out.** The degree-reduction recipe (a *quadratization*) is:

1. Identify a pairwise product $x_i x_j$ appearing inside the **highest-degree** term (choosing the highest-degree term guarantees the overall degree drops by one).
2. Introduce a new binary variable and substitute $y_{ij} \leftarrow x_i x_j$ throughout, e.g. defining $y_{45}=x_4 x_5$ so that $x_2 x_4 x_5 \to x_2\,y_{45}$.
3. Because the solver has no built-in knowledge that $y_{ij}$ must equal $x_i x_j$, append a **penalty term** that is zero when the relationship holds and strictly positive when it is violated. The candidate given is

$$
P\left(x_i x_j - 2x_i y_{ij} - 2x_j y_{ij} + 3y_{ij}\right),
$$

where $P>0$ is a sufficiently large penalty weight.

**Why the penalty works (the key check).** When the constraint is satisfied, substitute $y_{ij}=x_i x_j$:

$$
x_i x_j - 2x_i x_j - 2x_j x_i + 3x_i x_j = 0,
$$

so a consistent assignment incurs no penalty. When $y_{ij}\neq x_i x_j$, one builds a small truth table over the three variables $(x_i,x_j,y_{ij})$. It suffices to examine only the rows where the AND of $x_i,x_j$ differs from $y_{ij}$ (there are four such inconsistent rows among the eight total); in every one of them the expression evaluates to a strictly positive value, exactly the desired behavior. The remaining four rows are the consistent ones, already shown to evaluate to $0$ by direct substitution. The speaker notes a side observation surfaced by a participant: this particular penalty also behaves sensibly in the boundary case $y_{ij}=0$, even though that was not the original motivation.

> *Extended context.* This construction is the classical **Rosenberg quadratization** (Rosenberg, 1975), the standard route from Higher-Order Binary Optimization (HOBO/PUBO) to QUBO. The generic Rosenberg penalty is often written $P\,(x_i x_j - 2x_i y - 2x_j y + 3y)$, matching the slide. Each reduction adds one auxiliary variable per replaced product; reducing many high-degree terms therefore carries a real **overhead** in qubit/variable count (the speaker explicitly calls this out). Modern research (e.g. compressed quadratizations and SAT-specific encodings) aims to minimize the number of auxiliary variables and the size of penalty coefficients — relevant because penalty magnitude affects solver/annealer performance.

### 3.2 Graph Colouring as a QUBO

**Theoretical foundation.** Graph colouring is the problem of assigning one of $k$ colours to each vertex of an undirected graph $G=(V,E)$ such that no two adjacent vertices (connected by an edge) share the same colour. It is a canonical **constraint-satisfaction problem** with no objective to minimize — the goal is purely feasibility. In the QUBO/optimization context, feasibility problems are handled by converting every constraint into a **penalty term** and then minimizing the total penalty; a solution with penalty $0$ is a valid colouring.

**Variable encoding.** Introduce $n\times k$ binary variables

$x_{i,c} = \begin{cases} 1 & \text{if vertex } i \text{ is assigned colour } c, \\ 0 & \text{otherwise,} \end{cases} \quad 0\le i<n,\;0\le c<k.$

Each vertex therefore has a block of $k$ binary variables, one per colour, and a valid colouring corresponds to exactly one of them being $1$ in each block.

**Constraint 1 — each vertex receives exactly one colour.**

$\sum_{c=0}^{k-1} x_{i,c} = 1 \quad \forall\, i.$

The equality $\sum_c x_{i,c} = 1$ can be squared and penalized: it is zero iff the constraint is satisfied and strictly positive otherwise.

$\text{Penalty}_1 = P \cdot \sum_{i=0}^{n-1} \left(1 - \sum_{c=0}^{k-1} x_{i,c}\right)^2.$

Expanding the square yields cross-terms, which are all quadratic in the $x_{i,c}$ variables — so this term is already QUBO-compatible.

**Constraint 2 — adjacent vertices must receive different colours.**

For every edge $(i,j)\in E$ and every colour $c$:

$x_{i,c} + x_{j,c} \le 1.$

This says it is forbidden for both vertex $i$ and vertex $j$ to be assigned colour $c$ simultaneously. The inequality $x+y\le1$ for binary variables is equivalent to requiring $xy=0$ (they cannot both be $1$). The standard QUBO penalty for this is $g(x,y)=xy$: it equals $1$ (and thus costs $P$) exactly when both variables are $1$ (the forbidden case), and $0$ otherwise. Summing over all edges and all colours gives

$\text{Penalty}_2 = P \cdot \sum_{(i,j)\in E}\sum_{c=0}^{k-1} x_{i,c}\, x_{j,c}.$

All terms are **quadratic** (products of two binary variables), so this is also directly expressible in QUBO form.

**Full objective.** Because there is no objective function to minimize beyond satisfying the constraints, the complete QUBO objective is the sum of the two penalties alone:

$\min\; \left[P\cdot\sum_{i=0}^{n-1}\left(1-\sum_{c=0}^{k-1}x_{i,c}\right)^2 + P\cdot\sum_{(i,j)\in E}\sum_{c=0}^{k-1}x_{i,c}\,x_{j,c}\right].$

A global minimum of $0$ corresponds to a valid $k$-colouring; if the minimum is strictly positive the graph is not $k$-colourable.

**Key insight on penalty weight $P$.** The penalty weight must be chosen large enough that the optimizer cannot "profit" from violating a constraint. A common rule of thumb is to pick $P$ larger than any possible objective gain; for pure-penalty problems like colouring, any $P>0$ suffices in principle, but in practice noisy solvers and finite precision require $P$ to be set carefully relative to the scale of the other coefficients.

> *Extended context.* Graph $k$-colouring is NP-complete for $k\ge3$. The QUBO formulation above has $nk$ binary variables, which is manageable for small graphs but can be expensive for large or densely coloured instances. The quadratic terms from Penalty$_2$ grow like $|E|\cdot k$ in number; for dense graphs this can be large. Alternative formulations (e.g., using fewer variables or hierarchical decompositions) are an active area. On D-Wave hardware the same BQM/Ising representation handles this directly.

### 3.3 Max-3SAT as a QUBO

**Theoretical foundation.** A SAT instance is a Boolean formula $\phi$ in **Conjunctive Normal Form (CNF)**: an AND of clauses, $\phi = C_1 \wedge C_2 \wedge \cdots \wedge C_n$, where each clause is an OR of literals and a literal $y_k$ is either a variable $x_k$ or its negation $\overline{x}_k$. In **3-SAT**, each clause contains at most three literals. The *decision* problem asks whether there exists an assignment making $\phi$ evaluate to $1$ (satisfiable); the *optimization* version, **Max-3SAT**, drops the all-or-nothing requirement and instead **maximizes the number of satisfied clauses**. Any Boolean formula can be converted to CNF, so this is a fully general target. The speaker explicitly corrects a slide typo on the fly: within a clause the literals are OR-ed together, and the clauses are AND-ed — that is the CNF structure.

**How it was carried out.** Each clause is mapped to a polynomial penalty/score function $g(x)$ that equals $1$ exactly when the clause is satisfied and $0$ otherwise. The four cases are organized by **how many literals are negated**. From the slides:

- **Zero negations**, $(x_i \vee x_j \vee x_k)$:
$$
g(x) = x_i + x_j + x_k - x_i x_j - x_i x_k - x_j x_k + x_i x_j x_k.
$$
- **One negation**, $(x_i \vee x_j \vee \overline{x}_k)$:
$$
g(x) = 1 - x_k + x_i x_k + x_j x_k - x_i x_j x_k.
$$
- **Two negations**, $(x_i \vee \overline{x}_j \vee \overline{x}_k)$:
$$
g(x) = 1 - x_j x_k + x_i x_j x_k.
$$
- **Three negations**, $(\overline{x}_i \vee \overline{x}_j \vee \overline{x}_k)$:
$$
g(x) = 1 - x_i x_j x_k.
$$

The speaker walks the **zero-negation** case through its truth table to build intuition. With all three variables $0$, no term contributes and $g=0$ — the single unsatisfying assignment of a pure OR. As soon as any single variable is $1$, that linear term contributes $1$ and the clause evaluates to $1$. When two variables are $1$ (say $x_j=x_k=1$), the two positive linear terms give $+1+1$ but the quadratic $-x_jx_k$ subtracts $1$, netting $1$. When all three are $1$, the three linear terms give $+3$, the three pairwise products subtract $-1-1-1$, and the triple product adds $+1$, again netting $1$. So $g=1$ on all seven satisfying rows and $g=0$ only on the all-zeros row — precisely the OR's truth table. The other three cases follow the same pattern (the one-negation case flips the role of $x_k$; the all-negated case evaluates to $0$ only when all three variables are $1$). Permutations handle which specific literals are negated.

**Assembling the full objective.** For clause $C_i$ with its function $g_i(x)$, the Max-3SAT objective is

$$
f(x) = \max \sum_{i=1}^{n} g_i(x),
$$

i.e. count how many clauses evaluate to $1$ and maximize over all variable assignments. Because each clause's polynomial contains a **cubic** term $x_i x_j x_k$, the final step is to apply the Section 3.1 degree-reduction (introduce auxiliaries $y_{ij}$ and penalties) to obtain a genuine QUBO. The speaker emphasizes that this whole pipeline is **mechanical and automatable**, but worth seeing once end-to-end.

> *Extended context.* This clause-by-clause polynomial scoring is the textbook pseudo-Boolean encoding of OR/satisfaction; alternative, more variable-efficient 3SAT-to-QUBO transformations exist (Chancellor's, Nüßlein's, and "pattern QUBO" constructions), which trade off auxiliary-variable count, penalty gaps, and robustness on noisy annealers.

### 3.3 Variational Quantum Algorithms (VQA) and the Variational Quantum Eigensolver (VQE)

**Theoretical foundation.** Having a QUBO is only half the story; a *quantum* algorithm must now solve an optimization problem. **Variational Quantum Algorithms** are the dominant NISQ-era answer. The core object is a **parameterized (parametrized) quantum circuit** — sometimes called a PQC or *ansatz* — whose gates depend on tunable parameters $\theta$. The prepared state therefore depends on $\theta$, and one searches for the $\theta$ that makes the output "better and better" for the problem at hand. The **hybrid loop** is:

1. The **quantum computer** runs the parameterized circuit and **measures**, producing data.
2. Measurement results are post-processed into a **cost** (defined by how the problem was reduced to the circuit).
3. A **classical optimizer** evaluates the cost and computes updated parameters.
4. The new $\theta_{k+1}$ are fed back into the circuit, and the loop repeats.

This generic template applies to **quantum chemistry, combinatorial optimization, and quantum machine learning / quantum neural networks**. An honest caveat is raised: unlike Grover's algorithm — whose complexity can be rigorously characterized — it is generally **hard to prove complexity-theoretic guarantees** about variational algorithms; only special-case results tend to be available.

**VQE and the physics of energy.** The **Variational Quantum Eigensolver** specializes the VQA loop to finding the **ground-state energy** of a quantum system — the lowest eigenvalue of its **Hamiltonian**. The motivating intuition is high-school chemistry: a molecule's electrons occupy orbitals, and one seeks the configuration of **lowest energy**. The optimization payoff: if a quantum system can be made to evolve to (or have its energy estimated near) its ground state, and if an optimization problem is *encoded* into that system's Hamiltonian, then finding the ground state yields the optimal value of the original problem.

The necessary linear algebra is built up carefully:

- **Eigenvalues/eigenvectors:** for a matrix $A$ and vector $v$, generically $Av=w$; for special vectors $A\,v_\lambda = \lambda\, v_\lambda$, where $v_\lambda$ is the eigenvector and $\lambda$ the eigenvalue.
- **Hamiltonian = Hermitian operator:** $H = H^\dagger$, where $\dagger$ denotes the conjugate transpose (entrywise complex conjugate followed by transpose). The Hamiltonian represents the system's total energy.
- **Unitary evolution ↔ Hamiltonian:** a state evolves as $|\psi(t)\rangle = U\,|\psi(0)\rangle$ with $U = e^{-iHt}$ (a matrix exponential). This is the integrated form of the Schrödinger equation $i\,\tfrac{d}{dt}|\psi\rangle = H|\psi\rangle$. Unitary $U$ captures discrete-step evolution; $H$ captures continuous-time evolution. (A participant asks whether this resembles Euler's formula — yes, Euler's identity is essentially a scalar special case of this matrix exponential.)

**Observables, measurement, and expectation values.** Hermitian operators serve as **observables**. The crucial property: **Hermitian operators have real eigenvalues** (unitaries do not, in general), so their eigenvalues can be interpreted as the **real-valued outcomes** of measurements. Any observable admits a **spectral decomposition**

$$
H = \sum_{\lambda} \lambda\, P_{\lambda}, \qquad P_{\lambda} = |\psi_\lambda\rangle\langle\psi_\lambda|,
$$

where $P_\lambda$ is the **projector** onto the eigenspace with eigenvalue $\lambda$. The probability of outcome $\lambda$ for state $|\psi\rangle$ is the (generalized) Born rule

$$
p(\lambda) = \big|\langle\psi_\lambda|\psi\rangle\big|^2 = \langle\psi|P_\lambda|\psi\rangle,
$$

where magnitude-squared appears because amplitudes are now allowed to be **complex** (in the introductory QBronze material they had been real). The **expectation value** of the observable is then

$$
\langle H \rangle = \sum_\lambda \lambda\, p(\lambda) = \langle\psi|H|\psi\rangle,
$$

derived by plugging $p(\lambda)=\langle\psi|P_\lambda|\psi\rangle$ into $\sum_\lambda \lambda\, p(\lambda)$ and recognizing $\sum_\lambda \lambda P_\lambda = H$.

**Measuring in other bases (Qiskit-style reasoning).** Hardware measures in the computational ($Z$) basis by default, but one can measure in any basis. To measure in the $X$ basis, use the identity $X = H Z H$ (equivalently $Z = HXH$, with $H$ the Hadamard gate). Then

$$
\langle\psi|X|\psi\rangle = \langle\psi|H Z H|\psi\rangle = \langle\psi'|Z|\psi'\rangle, \qquad |\psi'\rangle = H|\psi\rangle.
$$

So an $X$-basis measurement is implemented by **applying a Hadamard to the state and then measuring in the standard $Z$ basis**. A similar identity (with a bit more algebra) handles the Pauli-$Y$ basis, and arbitrary bases follow from combinations of such rotations.

**The variational principle (the engine of VQE).** For a Hamiltonian $H$ with eigenstates $|\lambda\rangle$ and eigenvalues $E_\lambda$, and ground-state energy $E_0$,

$$
E_0 \;\le\; \langle\psi|H|\psi\rangle \quad \text{for any state } |\psi\rangle.
$$

Intuitively obvious (no state can have energy below the minimum), but operationally powerful: **any** random/parameterized trial state $|\psi\rangle$ furnishes a valid **upper bound** on $E_0$. VQE therefore starts from an arbitrary prepared state, computes $\langle\psi|H|\psi\rangle$ as a starting bound, and then enters the variational loop — estimate the energy from a parameterized circuit, use a classical optimizer to update $\theta_k \to \theta_{k+1}$, re-run, and drive the bound down toward $E_0$. The full proof of the variational inequality is deferred to the posted slides.

> *Extended context.* VQE produces an **upper bound** on the ground-state energy and is **heuristic**: its success depends on the expressiveness of the ansatz and on the classical optimizer; a poor ansatz lacking the needed entanglement cannot reach the true ground state, and rich ansätze can have many parameters and high measurement overhead. The closely related **QAOA** (Quantum Approximate Optimization Algorithm) is the VQA most directly aimed at QUBO/Ising combinatorial objectives — a natural sequel to this session.

### 3.4 The Ising Model and its Equivalence to QUBO (via Max-Cut)

**Theoretical foundation.** The **Ising model** describes the energy of a system of interacting magnetic spins (think electrons with spin up / spin down, $m_s=+\tfrac12$ or $-\tfrac12$). Each spin $S_i$ interacts with an external magnetic field via a local bias $h_i$, and neighboring spins interact through a coupling strength $J_{ij}$. The session frames qubits as laid out on a **lattice** — a 1D line or, more usefully here, a **2D grid** — exactly the language hardware vendors use to describe the geometry of physical qubit interactions. Restricting to **two-neighbor (pairwise) interactions** in the lattice is precisely the Ising-model counterpart of QUBO's quadratic-only restriction. The classical **Ising energy** is

$$
E_{\text{Ising}}(S) = \sum_i h_i\, S_i + \sum_{i,j} J_{ij}\, S_i S_j, \qquad S_i \in \{+1,-1\}.
$$

The configuration of lowest energy is the **ground state**; finding it for a 2D Ising model is **NP-hard** classically — which is exactly why mapping NP-hard/NP-complete problems onto such physical systems is appealing for quantum approaches.

**From energy to a cost Hamiltonian.** Promoting spins to the Pauli-$Z$ operator gives the **cost Hamiltonian**

$$
H_C = \sum_i h_i\, Z_i + \sum_{i,j} J_{ij}\, Z_i Z_j.
$$

The session shows *why* this reproduces the energy expression: apply $H_C$ to a computational-basis lattice state $|\psi\rangle$. Because such states are **eigenstates of each $Z_i$** with eigenvalue $S_i=\pm1$, each $Z_i|\psi_i\rangle = S_i|\psi_i\rangle$, so

$$
\langle\psi|H_C|\psi\rangle = \sum_i h_i S_i + \sum_{i,j} J_{ij}\, S_i S_j = E_{\text{Ising}}(S).
$$

The expectation value of the cost Hamiltonian *is* the Ising energy of the configuration — this is the link between "cost" in optimization and "energy" in physics.

**Worked example — Max-Cut in spin variables.** For a graph, assign each vertex a spin: $S_i=+1$ if vertex $i$ is in set $R$, $S_i=-1$ if in set $S$. The product

$$
S_i S_j = \begin{cases} +1 & \text{same group}, \\ -1 & \text{different groups.}\end{cases}
$$

Working in $\pm1$ replaces the binary-variable **parity** check with a **product**. The cut size (number of edges crossing the partition) is

$$
\text{cut} = \frac{1}{2} \sum_{(i,j)\in E} \big(1 - S_i S_j\big).
$$

When the endpoints are in the same group, $1-S_iS_j = 1-1 = 0$ (no contribution); when they differ, $1-S_iS_j = 1-(-1) = 2$, hence the factor $\tfrac12$ normalizes each cut edge to count once. To match QUBO/Ising's **minimization** convention, maximizing the cut is rewritten as

$$
\min\; \frac{1}{2}\sum_{(i,j)\in E}\big(S_i S_j - 1\big),
$$

and since the $-1$ is a constant offset (irrelevant to the location of the optimum) it can be dropped, leaving the clean cost Hamiltonian

$$
H_C = \sum_{(i,j)\in E} Z_i Z_j,
$$

exactly the pairwise $Z_iZ_j$ form of the Ising lattice.

**The QUBO ↔ Ising equivalence (the punchline).** The only real difference between the two models is the variable domain: QUBO uses $x_i\in\{0,1\}$, Ising uses $S_i\in\{-1,+1\}$. The change of variables is

$$
x_i = \frac{1 - S_i}{2}.
$$

The Max-Cut QUBO objective is (verifiable directly)

$$
\min \sum_{(i,j)\in E} \big(-x_i - x_j + 2x_i x_j\big).
$$

Substituting $x_i = \tfrac{1-S_i}{2}$ and $x_j=\tfrac{1-S_j}{2}$:

$$
\sum_{(i,j)\in E} -\!\left(\frac{1-S_i}{2}\right) - \left(\frac{1-S_j}{2}\right) + 2\left(\frac{1-S_i}{2}\right)\!\left(\frac{1-S_j}{2}\right).
$$

Expanding the product term gives $\tfrac{1 + S_iS_j - S_i - S_j}{2}$; the lone $-\tfrac{1-S_i}{2}$ and $-\tfrac{1-S_j}{2}$ contribute $\tfrac{-1+S_i}{2}+\tfrac{-1+S_j}{2}$. The $S_i$ and $S_j$ linear pieces cancel, and the constants combine ($-1-1+1 = -1$ inside the half), leaving

$$
= \frac{1}{2}\sum_{(i,j)\in E}\big(S_i S_j - 1\big),
$$

which is exactly the Ising/Max-Cut minimization objective. **Minimizing the QUBO is identical to minimizing the Ising configuration.**

**Why this matters.** A participant's question is answered head-on: QUBO and Ising are *equivalent*, but QUBO is "just a mathematical formulation" with no physical substrate — you could simply solve it on a classical computer. The mapping to Ising endows the same objective with an **actual physical model** that quantum physics can describe and that real quantum hardware (e.g. annealers) can realize, so the QUBO problem becomes solvable via the quantum algorithm. A second subtlety is clarified: the sign in the mapping ($1+S_i$ vs $1-S_i$ over $2$) corresponds to whether one is maximizing or minimizing — the notebooks introduce $\tfrac{1+S_i}{2}$ as general motivation but use $\tfrac{1-S_i}{2}$ for the Max-Cut PUBO↔Ising construction because Max-Cut starts as a maximization that must be turned into a minimization (the minus sign).

> *Extended context.* This is the standard, well-known QUBO–Ising isomorphism ($x = \tfrac{1-s}{2}$, equivalently $s = 1-2x$). D-Wave's tooling exposes it directly through the **Binary Quadratic Model (BQM)** in the **Ocean SDK**, which is just an implementation of QUBO plus the interconversions, applied to problems like TSP and Max-Cut. One practical caveat from the literature: arbitrary QUBO coefficients can map to Ising couplings of mixed sign, which matters for hardware whose couplers have constrained ranges.

---

## 4. Points of Confusion and Edge Cases

- **"Why only quadratic? Why not pick an unrestricted formulation?"** The restriction to degree two is a **hardware constraint** (2-local couplings), not a mathematical necessity. Any higher-order problem *can* be reduced, but the reduction is not free.
- **The penalty term is non-obvious.** It is genuinely *not* clear by inspection that $P(x_ix_j - 2x_iy_{ij} - 2x_jy_{ij} + 3y_{ij})$ enforces $y_{ij}=x_ix_j$. The reliable way to convince yourself is the **truth table**: verify the expression is $0$ on the four consistent rows (direct substitution) and strictly positive on the four inconsistent rows. Choosing the penalty weight $P$ too small can let a violating assignment win — the penalty must dominate any objective gain from cheating.
- **Auxiliary-variable overhead.** Every degree reduction introduces a new variable $y_{ij}$. For problems riddled with high-order terms (like Max-3SAT, whose every clause is cubic), this can substantially inflate the variable/qubit count — a real scaling concern, not just bookkeeping.
- **CNF structure / a live slide typo.** Inside a clause the literals are **OR**-ed; the clauses are **AND**-ed. The speaker mis-wrote AND vs OR on the slide and corrected it mid-session — a classic source of confusion when reading hastily.
- **Projector vs Hermitian "sandwich."** The expressions $\langle\psi|P_\lambda|\psi\rangle$ and $\langle\psi|H|\psi\rangle$ look identical in form (something sandwiched between a bra and a ket), but the *type* of the middle operator changes the meaning entirely: a **projector** $P_\lambda$ yields a **probability** (Born rule), while a **Hermitian observable** $H$ yields an **expectation value**. The speaker flags this as a personal early stumbling block.
- **Hermitian vs unitary eigenvalues.** Only **Hermitian** operators are guaranteed **real** eigenvalues (hence usable as measurement outcomes); unitary operators have eigenvalues on the unit circle (complex phases). Conflating the two breaks the "eigenvalue = measurable outcome" interpretation.
- **Measuring in a non-$Z$ basis.** You do not need exotic hardware: rotate the **state** into the computational basis first (apply $H$ for the $X$ basis, an appropriate rotation for $Y$), then do the ordinary $Z$-basis measurement. Forgetting the basis-change rotation silently measures the wrong observable.
- **VQE returns an upper bound, not the exact answer.** The variational principle guarantees $\langle\psi|H|\psi\rangle \ge E_0$; a trial state only ever *upper-bounds* the ground-state energy. Convergence quality hinges on ansatz and optimizer, and strong complexity guarantees are generally unavailable for variational methods (unlike Grover).
- **QUBO and Ising are the *same* problem in different clothes.** Don't treat them as two methods. The sign convention in $x_i = \tfrac{1\pm S_i}{2}$ encodes whether you are maximizing or minimizing; mixing up the sign flips the optimization direction. Constant offsets (like the $-1$ in the Max-Cut objective) can be dropped without changing the optimizer.
- **"Same group" arithmetic in Max-Cut.** With $\pm1$ spins, same-group edges give $1-S_iS_j=0$ and cross edges give $1-S_iS_j=2$ — hence the $\tfrac12$ normalization. Working in $\{0,1\}$ you'd use parity (XOR); in $\{-1,+1\}$ you use the product. Swapping these intuitions is a common slip.

---

## 5. To Study (Flashcards)

1. **Q:** Why is QUBO restricted to linear and quadratic terms, and how do you handle a cubic term such as $x_2 x_4 x_5$?  
   **A:** Because the hardware/algorithm is **2-local** (only pairwise couplings). Reduce the degree by substituting a pairwise product with a new variable, $y_{45}=x_4x_5$, and add the Rosenberg penalty $P(x_4x_5 - 2x_4y_{45} - 2x_5y_{45} + 3y_{45})$, which is $0$ iff $y_{45}=x_4x_5$.

2. **Q:** In Max-3SAT, what is the clause function for a clause with **zero** negations, $(x_i\vee x_j\vee x_k)$, and what does the global objective look like?  
   **A:** $g(x)=x_i+x_j+x_k-x_ix_j-x_ix_k-x_jx_k+x_ix_jx_k$ (equals $1$ for every satisfying assignment, $0$ only when all three are $0$). Global objective: $f(x)=\max\sum_{i=1}^{n} g_i(x)$, then quadratize the cubic terms.

3. **Q:** State the variational principle and explain why it makes VQE work.  
   **A:** For any state $|\psi\rangle$, $\langle\psi|H|\psi\rangle \ge E_0$ (the ground-state energy). Thus any trial/parameterized state gives a valid **upper bound** on $E_0$; a classical optimizer tunes the circuit parameters $\theta$ to minimize $\langle\psi(\theta)|H|\psi(\theta)\rangle$ toward $E_0$.

4. **Q:** How do you measure in the $X$ basis on hardware that only measures in $Z$, and why does it work?  
   **A:** Apply a Hadamard to the state, then measure in $Z$: since $X=HZH$, $\langle\psi|X|\psi\rangle=\langle\psi'|Z|\psi'\rangle$ with $|\psi'\rangle=H|\psi\rangle$.

5. **Q:** Give the change of variables linking QUBO and Ising, and the Max-Cut cost Hamiltonian.  
   **A:** $x_i=\dfrac{1-S_i}{2}$ (equivalently $S_i=1-2x_i$), with $x_i\in\{0,1\}$, $S_i\in\{-1,+1\}$. Substituting into the Max-Cut QUBO yields $\tfrac12\sum_{(i,j)\in E}(S_iS_j-1)$; dropping the constant gives $H_C=\sum_{(i,j)\in E} Z_iZ_j$.

6. **Q:** What distinguishes $\langle\psi|P_\lambda|\psi\rangle$ from $\langle\psi|H|\psi\rangle$?  
   **A:** With a **projector** $P_\lambda=|\psi_\lambda\rangle\langle\psi_\lambda|$ it is the **probability** $p(\lambda)$ (Born rule); with a **Hermitian observable** $H$ it is the **expectation value** $\langle H\rangle=\sum_\lambda \lambda\,p(\lambda)$.

---

### Appendix — Source Fidelity Notes

- The session is a **continuation**: it picks up from a prior day's TSP-as-QUBO example and ends by pointing forward to **quantum annealing / the adiabatic algorithm** (next day) and to D-Wave **Ocean SDK** notebooks (Ising model, Ising-conversion, and Binary Quadratic Model, applied to TSP and Max-Cut).
- Where the audio garbled notation ("Kubo/Kuber/Cobalt/cubo" → **QUBO**; "icing/ISIN" → **Ising**; "poly Z / poly Y" → **Pauli $Z$ / Pauli $Y$**; "hadamar" → **Hadamard**; "kiskit" → **Qiskit**; "Bond's rule / Bonduru" → **Born rule**; "Schroding" → **Schrödinger**; "various shipment principle" → **variational principle**; "eigensolver" terms standardized), the slide content was used as the authority and the math was rewritten in clean Dirac/linear-algebra notation.
- The speaker repeatedly noted that the deep quantum-physics background (observables, spectral decomposition, the matrix-exponential/Schrödinger link) is **enriching but not strictly required** for the assessments, which center on implementing the optimization pipeline in the D-Wave notebooks.
