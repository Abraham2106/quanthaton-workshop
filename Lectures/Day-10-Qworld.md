# Technical Summary — From Simulated Annealing to Quantum Annealing and QAOA

> QWorld Bootcamp · Session OQI26-1-10 · Instructor: Jibran Rashid
> Sources processed: automatic session transcript (VTT) + slide notes ("Recocido Simulado, AQC, exponenciales de matrices y QAOA"). Where the audio transcription was garbled, the slide notes were treated as authoritative.

---

## 1. Technical Sheet

- **Session topic:** Heuristic optimization bridged into quantum hardware — simulated annealing, adiabatic quantum computing (AQC) and the adiabatic theorem, matrix exponentials of Hermitian/Pauli operators, quantum annealing via tunnelling, and the Quantum Approximate Optimization Algorithm (QAOA) as a discretized, variational realization of adiabatic evolution.
- **Key concepts:** Simulated annealing & the Metropolis acceptance rule; the quantum adiabatic theorem & the spectral gap; matrix/operator exponentiation ($e^{-iPt}=\cos(t)I - i\sin(t)P$); Trotterization; the QAOA cost ($H_C$) and mixer ($H_B$) Hamiltonians with parameters $(\gamma,\beta)$.
- **Tools / Frameworks:** Conceptually framework-agnostic linear algebra and quantum mechanics. Implementation pointers target **D-Wave Ocean** (binary quadratic models, quantum annealing library) rather than gate-based SDKs; the same circuit maps naturally onto **Qiskit/Cirq** ($H$, $R_X$, $R_{ZZ}$ gates). No live code was written on screen — the notebooks were referenced, not executed.
- **Location in the bootcamp:** Final block of the curriculum — **Optimization (QUBO, Max-Cut, Ising) + Variational Algorithms (VQA, QAOA, VQE)**. This session is the capstone that fuses the optimization-encoding block (QUBO/Ising, covered "yesterday") with the variational-principle block, and points students toward the closing D-Wave assessments and hackathon.

---

## 2. Synopsis

This session builds a continuous conceptual bridge from a purely classical optimization heuristic to a hybrid quantum-classical algorithm runnable on near-term (NISQ) hardware. It opens with **simulated annealing**, an optimization method inspired by metallurgical annealing: a system is "heated" and slowly "cooled" so that, by occasionally accepting cost-increasing moves with probability $e^{-\Delta/t}$, it escapes local minima and converges toward a global optimum. The narrative then asks what a *quantum* analogue would look like and arrives at **adiabatic quantum computing**, governed by the **adiabatic theorem**: if a system is initialized in the easily prepared ground state of an initial Hamiltonian $H_i$ and the Hamiltonian is deformed slowly enough into a problem Hamiltonian $H_f$, the system remains in the instantaneous ground state, so a final measurement reveals the encoded solution. "Slowly enough" is dictated by the **spectral gap** between the ground state and the first excited state — the central technical obstacle, since controlling that gap is itself hard. To turn this continuous evolution into executable gates, the session develops the mathematics of **matrix exponentials** for Hermitian and Pauli operators, then uses **Trotterization** to split a non-commuting Hamiltonian sum into a product of implementable rotations, incurring an $O(\delta t^2)$ error. Discretizing the adiabatic schedule in this way yields **QAOA**: $p$ alternating layers of a cost unitary $e^{-i\gamma_k H_C}$ and a mixer unitary $e^{-i\beta_k H_B}$ acting on the uniform superposition $|s\rangle = |+\rangle^{\otimes n}$, with a classical optimizer tuning $(\gamma,\beta)$ to minimize $\langle\psi(\gamma,\beta)|H_C|\psi(\gamma,\beta)\rangle$. The arc closes by reconnecting QAOA to both the variational principle and quantum annealing, and by noting that for Max-Cut the Ising Hamiltonian derived earlier *is* $H_C$, so the algorithm is essentially specified once that encoding is in hand.

---

## 3. Subtopic Breakdown

### 3.1 Simulated Annealing — the classical baseline and its intuition

Simulated annealing is introduced as a **classical, heuristic optimization algorithm** that imitates a physical process from metallurgy. When a metal is heated and then cooled gradually, its atoms settle into a low-energy, highly ordered crystal lattice, and defects or cracks tend to vanish; annealing borrows this picture to search a cost landscape. Crucially, the instructor stresses that the *simulated* version does not need to model any real physical system — it is just an optimization procedure on par with Newton–Raphson or gradient descent, used to find a minimum or maximum over some terrain. Its distinguishing virtue is robustness on "unreasonable", highly fluctuating landscapes, paid for by the difficulty of giving tight general performance bounds.

The theoretical mechanism is the way it overcomes the **local-vs-global optimum problem** that traps naive descent. Pure gradient descent always moves downhill and therefore gets stuck in the nearest valley, even when a much deeper valley exists elsewhere. The slide notes give the precise procedure: start from a feasible solution $s = s_0$ at temperature $t = t_0$; repeatedly draw a candidate $s'$ from the neighborhood $N(s)$ and compute $\Delta = \text{cost}(s') - \text{cost}(s)$. If $\Delta < 0$ the move is strictly improving and is accepted ($s \leftarrow s'$). Otherwise — and this is the key step — the worse move is still accepted with probability

$$
P(\text{accept}) = e^{-\Delta/t} > p, \qquad p \sim \text{Uniform}(0,1).
$$

This is the **Metropolis acceptance rule**. The temperature $t$ is reduced according to an **annealing (cooling) schedule**; at high $t$ the system explores freely (large jumps), and as $t \to 0$ it focuses and "zooms in" on the global optimum. The instructor illustrated this with the classic Wikipedia GIF in which guess solutions bounce around the landscape early on and then localize as the temperature drops. He also clarified a conceptual subtlety in Q&A: annealing is a heuristic, but a **domain-agnostic** one — the rule "perturb and sometimes accept a worse move" is not tailored to the structure of the specific problem, unlike problem-specific heuristics. This subtopic matters here because it supplies the physical intuition (escape barriers via thermal fluctuation) that the quantum version will replace with a genuinely quantum mechanism.

### 3.2 Adiabatic Quantum Computing and the Adiabatic Theorem

The session next motivates the *quantum* counterpart. Historically, adiabatic quantum computing was proposed by **Farhi et al.** (the foundational "Quantum Computation by Adiabatic Evolution" is Farhi, Goldstone, Gutmann & Sipser, 2000; the NP-complete-instance application appeared in *Science* in 2001), framed around the dates ~2003–2004 in the talk as an alternative way of thinking about quantum algorithms. It was later shown — by **Aharonov, van Dam, Kempe, Landau, Lloyd & Regev (2004)** — that AQC is **polynomially equivalent** to the standard circuit model, i.e., it is universal. The session deliberately does not prove this; it instead *extracts an algorithm* from the adiabatic idea.

The underlying physics is the difference between **discrete and continuous state evolution**. In the circuit picture a unitary jumps the state forward, $|\psi(t)\rangle = U(t,t_s)\,|\psi(t_s)\rangle$. Continuously, evolution is governed by the time-dependent **Schrödinger equation**,

$$
i\hbar\,\frac{d}{dt}|\psi(t)\rangle = H(t)\,|\psi(t)\rangle,
$$

and it is this continuous, Hamiltonian-driven evolution that produces the unitary. The adiabatic strategy is: encode the answer in the **ground state** of a problem Hamiltonian $H_f$ (exactly the Ising/QUBO construction from the previous session, where the ground state encodes the optimal solution), but instead of preparing that hard state directly, start in the easily prepared ground state of a simple $H_i$ and interpolate. The linear schedule given in the slides is

$$
\tilde H(s) = (1-s)\,H_i + s\,H_f, \quad s = t/T \in [0,1], \qquad H(t) = \Big(1 - \tfrac{t}{T}\Big)H_i + \tfrac{t}{T}H_f, \quad t\in[0,T].
$$

The **adiabatic theorem** guarantees that if the system has a unique ground state for all $s$ and the evolution is slow enough (formally, in the limit $T \to \infty$ the Hamiltonian changes arbitrarily slowly), then a system started in the ground state of $H_i$ ends in the ground state of $H_f$. Measuring the final state then yields the ground-state energy — i.e., the solution. The decisive caveat developed at length is the **spectral gap**: "slow enough" is set by the gap between the lowest eigenvalue (ground state) and the next eigenvalue (first excited state) *throughout* the evolution. If that gap becomes very small, there is a non-negligible probability that the system jumps to a higher-energy level, so at $t=T$ it would *not* be in the ground state of $H_f$ — the wrong answer. Controlling the gap is therefore itself a hard problem, and although a **linear** interpolation path is the default, other (non-linear) paths are possible — but each alternative path raises the same hard question of guaranteeing an adequate gap along the way. This tension (universality and elegance vs. the intractable gap-control requirement) is precisely why the session pivots from *exact* adiabatic computing to *approximate* **quantum annealing**, which keeps the spirit of the adiabatic deformation while using it as an approximation tool inside an algorithm.

### 3.3 Matrix Exponentials and Trotterization — turning Hamiltonians into gates

Before the continuous evolution can be discretized into runnable gates, the session installs the necessary linear-algebra machinery: how to **exponentiate a matrix**, specifically a Hermitian one. The general definition is the Taylor series

$$
e^{A} = \sum_{k=0}^{\infty} \frac{A^k}{k!}.
$$

For a **Hermitian** matrix this becomes tractable because Hermitian matrices are diagonalizable as $H = U D U^{\dagger}$, with $D$ diagonal (the eigenvalues) and $U$ unitary. Since $H^n = U D^n U^{\dagger}$ (proved inductively), the series collapses to

$$
e^{H} = U\, e^{D}\, U^{\dagger},
$$

and $e^{D}$ is just the entrywise exponential of the diagonal — easy to compute. The session then specializes to **Pauli operators**, which satisfy $P^2 = I$. This involution property folds the Taylor series into a generalized **Euler identity**:

$$
e^{-iPt} = \cos(t)\,I - i\sin(t)\,P.
$$

Worked for $P = X$, this produces (up to the conventional factor of two in the angle) the rotation gate $R_x(2t)$ — explicitly showing the bridge between **continuous** evolution (a one-parameter family parameterized by $t$) and the **discrete** application of a fixed unitary (choose the specific $t$ that realizes the gate you want). This is the conceptual heart of "how a Hamiltonian becomes a circuit."

The remaining obstacle is that the discretized adiabatic Hamiltonian is a **sum** $H = \alpha H_i + \beta H_f$ sitting inside a single exponential, and in general $H_i$ and $H_f$ **do not commute** ($H_iH_f \neq H_fH_i$), so one *cannot* naively split $e^{-i(H_i+H_f)t}$ into $e^{-iH_i t}e^{-iH_f t}$. The fix is **Trotterization** (the Lie–Trotter product formula):

$$
e^{-i(H_1 + H_2)t} = e^{-iH_1 t}\,e^{-iH_2 t} + O(t^2).
$$

Applied to the chunked evolution $T_k = k\,\delta t$ with $\delta t = T/n$, the full propagator factorizes into a product of small, individually implementable cost and mixer rotations:

$$
U(T,0) \approx \prod_{k=0}^{n} e^{-iH(k\,\delta t)\,\delta t}, \qquad \delta t = \frac{T}{n}.
$$

The instructor was explicit that the dropped $O(\delta t^2)$ term is a *genuine approximation error*, acknowledged in a student exchange: the formulation is a big-picture approximation, and the error shrinks as $n$ grows (more, finer layers → closer to true adiabatic evolution). This subtopic is the technical linchpin: it converts the abstract adiabatic schedule into the concrete layered circuit of the next subtopic.

### 3.4 QAOA — the discretized, variational realization

QAOA (Quantum Approximate Optimization Algorithm; **Farhi, Goldstone & Gutmann, 2014**, arXiv:1411.4028) is presented as the algorithm that drops out of Trotterizing the adiabatic evolution. The takeaway stated outright is: **by iteratively applying $H_i$ (mixer) and $H_f$ (cost), one approximates adiabatic evolution, and the approximation improves as $n$ (the number of layers $p$) grows.**

The pieces, assembled in order:

- **Initial state.** The standard choice is the easily prepared uniform superposition $|s\rangle = |+\rangle^{\otimes n}$, obtained by applying a Hadamard to every qubit of $|0\rangle^{\otimes n}$. This is deliberately chosen to be the **ground state of the mixer Hamiltonian**.
- **Mixer Hamiltonian.** One needs a Hermitian operator whose ground state (eigenvalue $-1$) is $|+\rangle^{\otimes n}$. Recalling that $|+\rangle$ is the $+1$ eigenstate of $X$ and that Hadamard maps the computational basis to the $X$ basis, the answer is

$$
H_B = -\sum_{i=1}^{n} X_i.
$$

The minus sign makes $|+\rangle^{\otimes n}$ the ground state. $H_B$ is easy to prepare and is interpreted as the **source of quantum fluctuations** (each $X$ flips a qubit), which is exactly why it is called the *mixer*. Its layer unitary is a product of single-qubit $X$-rotations, $U(H_B,\beta) = \prod_{i=1}^{n} R_x(2\beta)$ on qubit $i$.
- **Cost Hamiltonian.** $H_C$ encodes the problem so that $H_C|x\rangle = f(x)|x\rangle$. For QUBO/Ising problems (e.g., Max-Cut) it has the familiar Ising form

$$
H_C = \sum_i h_i Z_i + \sum_{i<j} J_{ij}\, Z_i Z_j,
$$

linking directly to the Ising model covered the previous day. Its layer unitary contains single-qubit $Z$-rotations $R_z(2 h_i \gamma)$ for the linear terms and two-qubit $R_{ZZ}$ rotations for the quadratic $Z_iZ_j$ terms, where

$$
R_{ZZ}(\theta) = e^{-i\frac{\theta}{2} Z\otimes Z}
$$

is diagonal with entries $e^{\pm i\theta/2}$ along the diagonal (two of them carrying the opposite sign). In gate-based SDKs this $R_{ZZ}$ is realized as CNOT–$R_z$–CNOT or a native $ZZ$ rotation; the whole QAOA circuit needs only $H$, $R_X$, and $R_{ZZ}$ gates.
- **The parametrized state (ansatz).** Stacking $p$ alternating layers gives

$$
|\psi(\boldsymbol\gamma,\boldsymbol\beta)\rangle = \prod_{k=1}^{p} e^{-i\beta_k H_B}\, e^{-i\gamma_k H_C}\,|s\rangle,
$$

which is exactly the Trotterized product from §3.3. Here $\boldsymbol\gamma = (\gamma_1,\dots,\gamma_p)$ and $\boldsymbol\beta = (\beta_1,\dots,\beta_p)$ are two **lists** of variational parameters, one entry per layer.
- **Hybrid optimization loop.** With an initial choice $(\boldsymbol\gamma^{(0)}, \boldsymbol\beta^{(0)})$, prepare the state and **measure the expectation value** $\langle\psi(\boldsymbol\gamma,\boldsymbol\beta)|H_C|\psi(\boldsymbol\gamma,\boldsymbol\beta)\rangle$. A **classical optimizer** then proposes new parameters $(\boldsymbol\gamma^{(1)}, \boldsymbol\beta^{(1)})$, the state is re-prepared and re-measured, and the loop iterates. The goal is to drive the ansatz toward the ground state of $H_C$, so that the measured energy is the ground-state energy — which decodes to the optimal solution.

The instructor closed the conceptual arc by emphasizing that QAOA **simultaneously instantiates the variational principle** (minimizing an energy expectation over a parametrized family, as in the VQA discussion) **and quantum annealing** (the layered structure approximates adiabatic deformation). It is therefore one concrete recipe for running QUBO-formulated optimization on NISQ devices. For practice, he pointed to the D-Wave notebooks (simulated annealing → binary quadratic models → quantum annealing on D-Wave); running on actual D-Wave hardware is *not* required for the assessments — using the library suffices. A practically important note for the assessments: for **Max-Cut**, the Ising Hamiltonian already derived for the problem *is* $H_C$, so once you plug it in, "the entire QAOA algorithm is essentially specified — you just need to run it."

---

## 4. Points of Confusion and Edge Cases

- **Why accept a worse move?** The non-intuitive step in simulated annealing is *deliberately* accepting cost-increasing candidates with probability $e^{-\Delta/t}$. The point is not randomness for its own sake; it is the only mechanism that lets the search climb out of a local minimum to reach a deeper global one. Students often misread this as a flaw rather than the core feature.
- **"Heuristic" is domain-agnostic here.** Annealing is a heuristic, but unlike problem-tailored heuristics it does not exploit the structure of the specific instance. The acceptance rule is generic; only the cost function and neighborhood are problem-specific.
- **The spectral gap is the whole game in AQC.** The adiabatic theorem's "slowly enough" is not a vague suggestion — it is quantitatively tied to the **minimum gap** between the ground state and the first excited state along the path. A vanishing gap forces an impractically long $T$ or causes a leak to an excited state and a wrong answer. Worse, *estimating/controlling* the gap for an arbitrary $H_f$ is itself hard, which is exactly why exact AQC is replaced by approximate quantum annealing. Non-linear interpolation paths can help but reintroduce the same gap-guarantee problem.
- **Adiabatic ≠ measured at the end like a circuit.** The answer lives in the *ground state* you remain in throughout, not in a mid-evolution snapshot; you read it off only at $t=T$, and only if adiabaticity held.
- **Non-commuting Hamiltonians break the naive split.** Because $[H_i, H_f] \neq 0$ in general, $e^{-i(H_i+H_f)t} \neq e^{-iH_i t}e^{-iH_f t}$. Trotterization is what *makes* the factorization legitimate, at the cost of an explicit $O(\delta t^2)$ error term. The instructor flagged this error openly — it is real, and it shrinks only as the number of layers/time-steps grows.
- **Factor-of-two angle conventions.** $e^{-iXt}$ yields $R_x(2t)$, not $R_x(t)$; likewise the cost rotations carry $2h_i\gamma$ and the $R_{ZZ}$ exponent is $\theta/2$. Sign and factor-of-two conventions for $R_z$/$R_{ZZ}$ differ between textbooks and SDKs (some define $R_z(\theta)=e^{+iZ\theta/2}$, others $e^{-iZ\theta/2}$), and can introduce a global phase or sign flip — a frequent source of "my circuit doesn't match the formula" confusion.
- **$\gamma$ and $\beta$ are per-layer vectors, not scalars.** A genuine point of confusion in the live Q&A: the superscript/subscript notation denotes lists $(\gamma_1,\dots,\gamma_p)$ and $(\beta_1,\dots,\beta_p)$ across the $p$ layers, while the *outer* iteration index (e.g., $\boldsymbol\gamma^{(0)}\!\to\!\boldsymbol\gamma^{(1)}$) tracks the classical optimizer's updates. Conflating "layer index" with "optimization step" is a common mistake.
- **The mixer must match the initial state.** $H_B = -\sum_i X_i$ is chosen precisely so that $|+\rangle^{\otimes n}$ is its ground state. If you change the initial state, you must change the mixer accordingly; the standard pairing is a convention/good-default, not a universal requirement.
- **D-Wave hardware vs. library.** For the assessments you do **not** need physical D-Wave access (availability is region-dependent); working through the Ocean/library implementation is sufficient.

---

## 5. For Study (flashcards)

1. **Q:** In simulated annealing, when $\Delta = \text{cost}(s') - \text{cost}(s) \geq 0$, when is the worse candidate $s'$ accepted?
   **A:** With probability $e^{-\Delta/t}$: draw $p\sim\text{Uniform}(0,1)$ and accept $s'$ if $e^{-\Delta/t} > p$ (the Metropolis rule). Improving moves ($\Delta<0$) are always accepted.

2. **Q:** What does the adiabatic theorem guarantee, and what physical quantity sets how slow the evolution must be?
   **A:** Starting in the ground state of $H_i$ and deforming slowly enough to $H_f$ keeps the system in the instantaneous ground state, ending in the ground state of $H_f$. The required slowness is governed by the **minimum spectral gap** between the ground state and the first excited state along the path.

3. **Q:** Give the Euler-type identity for the exponential of a Pauli operator $P$ (with $P^2=I$), and the gate it produces for $P=X$.
   **A:** $e^{-iPt} = \cos(t)\,I - i\sin(t)\,P$. For $P=X$ this is the rotation $R_x(2t)$.

4. **Q:** Why is Trotterization needed in QAOA, and what error does it introduce?
   **A:** Because $H_B$ and $H_C$ generally do not commute, so $e^{-i(H_B+H_C)t}$ can't be split directly. Trotterization gives $e^{-i(H_1+H_2)t} = e^{-iH_1t}e^{-iH_2t} + O(t^2)$, introducing an $O(\delta t^2)$ error per step that decreases as the number of layers grows.

5. **Q:** Write the QAOA ansatz and name the initial state, mixer, and cost Hamiltonians.
   **A:** $|\psi(\boldsymbol\gamma,\boldsymbol\beta)\rangle = \prod_{k=1}^{p} e^{-i\beta_k H_B} e^{-i\gamma_k H_C}\,|s\rangle$, with initial state $|s\rangle=|+\rangle^{\otimes n}$, mixer $H_B = -\sum_i X_i$, and cost $H_C = \sum_i h_i Z_i + \sum_{i<j} J_{ij} Z_i Z_j$. A classical optimizer tunes $(\boldsymbol\gamma,\boldsymbol\beta)$ to minimize $\langle\psi|H_C|\psi\rangle$.

6. **Q:** For Max-Cut, how do you obtain the QAOA cost Hamiltonian $H_C$?
   **A:** Take the Ising Hamiltonian already derived for Max-Cut and use it directly as $H_C$; the QAOA algorithm is then fully specified up to running the optimization loop.

---

### Sources & fidelity note

The conceptual spine, formulas, and pedagogical emphasis come directly from the session transcript and the accompanying slide notes; the slide notes were used to repair garbled audio (e.g., "poly X" → Pauli $X$, "trautarization" → Trotterization, "ISIN" → Ising, "Kubo" → QUBO, "Furhi" → Farhi, "D-Wave" hardware references). Historical attributions (Farhi–Goldstone–Gutmann–Sipser 2000 for adiabatic evolution; Farhi–Goldstone–Gutmann 2014 for QAOA; Aharonov et al. 2004 for AQC ≡ circuit-model equivalence) and the standard $R_{ZZ}$ gate decomposition were added as verified background context to enrich the explanations; they are consistent with, and extend, what was stated in the session.
