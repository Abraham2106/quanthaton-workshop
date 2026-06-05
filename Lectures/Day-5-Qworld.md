# Technical Summary — Session 5: Qiskit Programming, Superdense Coding, and Quantum Teleportation

## 1. Technical Sheet

- **Session topic:** First hands-on quantum circuit programming in Qiskit, followed by a detailed, calculation-by-calculation treatment of two foundational quantum communication protocols — *superdense coding* and *quantum teleportation*.
- **Key concepts:** Pre-shared entanglement as a resource, the four Bell states, the Bell basis vs. the computational basis, the duality between transmitting a qubit and transmitting classical bits, global phase, and the structural link to quantum error correction.
- **Tools / Frameworks:** Qiskit (`QuantumRegister`, `ClassicalRegister`, `QuantumCircuit`, `AerSimulator`); classical simulation of circuits; linear algebra and Dirac notation throughout.
- **Position in the bootcamp:** Bridges the block on *states, qubits, circuits, and gates* into *protocols and programming in Qiskit*. As the fifth session, it consolidates the 2–3 qubit toy models and explicitly sets up the next session's generalization to $n$ qubits and the first non-trivial quantum algorithm.

---

## 2. Synopsis

This session is the course's first real fusion of formalism and executable code, paired with a careful physical reading of two protocols that historically demonstrated that quantum information can do something classical information cannot. It opens with Qiskit syntax: how quantum and classical registers are declared, why a classical register is mandatory (every measurement collapses a qubit into a classical bit that must be stored somewhere), how gates are applied to zero-indexed qubits, and how the `AerSimulator` runs a circuit many times — $1024$ shots in the example — to reconstruct the output probability distribution. This machinery is used to numerically confirm the previous day's three-qubit circuit, in which auxiliary states are prepared with $X$ and $H$, a Toffoli gate $CCX$ is applied, and a final $H$ yields the measurement outcome $|1\rangle$ with certainty. From this foundation the two central protocols are derived by explicitly tracking the state vector through each gate. Superdense coding shows how, starting from a shared Bell pair $|\phi^{+}\rangle$, Alice encodes two classical bits by conditionally applying $Z$ and $X$ to her half, transmits a single qubit, and lets Bob decode both bits with a $CNOT$ followed by $H$. Quantum teleportation inverts this logic: Alice transfers an unknown state $|\psi\rangle$ to Bob using pre-shared entanglement, a Bell-type measurement, and two classical bits of communication — without cloning the state and without violating relativistic causality. The session closes by framing the two protocols as mutual inverses and by reinterpreting Bob's conditional corrections as a special case of quantum error correction, where the Pauli operators $X$ and $Z$ generate every possible single-qubit error.

---

## 3. Subtopic Breakdown

### Programming and simulating circuits in Qiskit

The conceptual core of this block is that a quantum circuit is assembled from two distinct kinds of memory, and that this duality is not bookkeeping but a reflection of physics. A `QuantumRegister` holds the qubits that evolve unitarily; a `ClassicalRegister` is required because measurement is irreversible and projective — the moment a qubit is read, its amplitude collapses to a single bit, and that bit needs a classical home. The two registers are bound together into a `QuantumCircuit`. Gates are then applied with a deliberately transparent syntax: `qc.x(q[0])` for the Pauli $X$ (the NOT gate, which sends $|0\rangle \leftrightarrow |1\rangle$), `qc.h(q[0])` for Hadamard, and so on, with qubits indexed from zero. A measurement instruction specifies both which qubit is measured and which classical bit receives the result; this is exactly why the drawn circuit shows a *double* parallel line for the flow of classical information, in contrast to the *single* line that carries quantum information.

Execution is where the probabilistic nature of quantum mechanics becomes operationally visible. A job is submitted to the `AerSimulator` and run a chosen number of times — $1024$ shots in the demonstration — after which `get_counts` returns how often each outcome appeared. The instructor's analogy is precise: running a circuit once is like flipping a coin once — you learn a single outcome, not the distribution. To estimate the underlying probabilities you must sample repeatedly. For a deterministic circuit (one that outputs a basis state with certainty) a single shot would suffice, but in general the output is a superposition and sampling is essential. Several practical details round out the picture: the `barrier` is purely a visual partition with no computational effect; the default qubit ordering places $Q_0$ at the top (discussed as a *little-endian vs. big-endian* convention — pick one and stay consistent); and the circuit can be rendered as ASCII text or, more cleanly, with the Matplotlib (`MPL`) drawer. A point worth internalizing early: this is a **classical simulation** of a quantum circuit, not execution on real quantum hardware — nothing here is sampling a physical quantum device.

As a concrete verification exercise, the three-qubit circuit from the prior session is rebuilt. The second qubit is prepared in $|1\rangle$ via $X$; the third qubit is prepared in $|-\rangle$ via $X$ followed by $H$ (since $H|1\rangle = |-\rangle$); $H$ is applied to the first qubit; then the Toffoli gate $CCX$ acts with the first and second qubits as controls and the third as target; finally $H$ is applied again to the first qubit, which is then measured. Across all shots the outcome is $1$, confirming the earlier algebraic claim that there exists a choice of parameters $\alpha,\beta,\gamma,\delta$ for which the first qubit is measured as $|1\rangle$ with certainty. The exercise is pedagogically important because it closes the loop between hand calculation and simulator output: the math predicted determinism, and the histogram delivers it.

### Superdense coding

The governing principle is that a single qubit, when backed by entanglement established in advance, can carry more classical information than a single classical bit. The setup: Alice holds two classical bits $a$ and $b$, and she shares with Bob a maximally entangled Bell state

$$|\phi^{+}\rangle = \frac{1}{\sqrt{2}}\left(|0\rangle_{A}|0\rangle_{B} + |1\rangle_{A}|1\rangle_{B}\right),$$

where within each entangled pair the first qubit belongs to Alice and the second to Bob; the dotted line in the circuit denotes their physical separation. The instructor stresses a subtlety that is the single most common source of error: this state is **not** separable into "Alice's part" $\otimes$ "Bob's part." Because it is entangled, no description of the form $|\cdot\rangle_A \otimes |\cdot\rangle_B$ exists; one must reason about the joint state.

Alice encodes her two bits onto her half of the pair using *classically controlled* gates — note that the control bit being classical is perfectly fine and in fact simpler than a quantum control, since Alice can just inspect the value. If $a=1$ she applies $Z$, recalling that

$$Z|0\rangle = |0\rangle, \qquad Z|1\rangle = -|1\rangle,$$

which is why $Z$ is sometimes drawn as the $-1$ phase. If $b=1$ she applies $X$ (a $CNOT$ with her bit as control). The net effect is elegant: depending on $ab$, Alice steers the shared pair into one of the four mutually orthogonal Bell states. She then sends her qubit to Bob over a quantum channel. Now holding both qubits, Bob applies $CNOT$ followed by $H$ on the first qubit — precisely the inverse of the Bell-state preparation procedure — which rotates the Bell basis back into the computational basis, so a direct measurement reads off $ab$.

The protocol is verified exhaustively over the four inputs, tracking the joint state after Alice's encoding ($t_2$) and after Bob's decoding:

$$
\begin{array}{c|c|c|c}
ab & t_1\ (\text{Ctrl-}Z) & t_2\ (\text{Ctrl-}X) & \text{Output after } CNOT,\,H \\\\ \hline
00 & |\phi^{+}\rangle & |\phi^{+}\rangle & |00\rangle \\\\
01 & |\phi^{+}\rangle & |\psi^{+}\rangle & |01\rangle \\\\
10 & |\phi^{-}\rangle & |\phi^{-}\rangle & |10\rangle \\\\
11 & |\phi^{-}\rangle & |\psi^{-}\rangle & -|11\rangle
\end{array}
$$

where the four Bell states are

$$|\phi^{\pm}\rangle = \tfrac{1}{\sqrt{2}}(|00\rangle \pm |11\rangle), \qquad |\psi^{\pm}\rangle = \tfrac{1}{\sqrt{2}}(|10\rangle \pm |01\rangle).$$

To make the mechanism concrete, consider $ab=00$: both controls are off, the state stays $|\phi^{+}\rangle$ through $t_2$; Bob's $CNOT$ maps $\tfrac{1}{\sqrt 2}(|00\rangle+|11\rangle) \to \tfrac{1}{\sqrt 2}(|00\rangle+|10\rangle) = |+\rangle|0\rangle$, and since $H|+\rangle = |0\rangle$ the output is $|00\rangle$. The key takeaway: by transmitting **one** qubit plus consuming the pre-shared entanglement, Alice conveys **two** classical bits. The entanglement is genuinely "used up" — the joint state migrates from the Bell basis into a separable computational-basis state. The $-1$ in the $ab=11$ case is a **global phase** and is physically unobservable: measurement probabilities depend on the squared modulus, so $-|11\rangle$ and $|11\rangle$ are indistinguishable. As an advanced aside, the instructor notes that Bob could instead perform a *Bell-basis measurement* directly — because the four Bell states are orthogonal, he can perfectly discriminate them once both qubits are in his lab, which he could not do while the pair was split between the two parties. The whole analysis assumes an idealized, noise-free regime, and the gate order matters (swapping $Z$ and $X$, or Bob's two operations, changes the result).

### Quantum teleportation

This protocol is the conceptual inverse of superdense coding and is subtler. Alice transfers an **unknown** state $|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$ to Bob without ever learning $\alpha$ or $\beta$ — the state could be handed to her in a black box — using pre-shared entanglement plus classical communication. The initial three-qubit state is

$$|\psi\rangle \otimes |\phi^{+}\rangle = (\alpha|0\rangle+\beta|1\rangle)\otimes\frac{1}{\sqrt{2}}(|00\rangle+|11\rangle) = \frac{1}{\sqrt{2}}\big(\alpha|000\rangle+\alpha|011\rangle+\beta|100\rangle+\beta|111\rangle\big).$$

Alice applies $CNOT$ with $|\psi\rangle$ as control and her half of the Bell pair as target, then $H$ on $|\psi\rangle$. After expanding and regrouping so that her two qubits are factored to the front, the state becomes

$$\frac{1}{2}\Big[\,|00\rangle(\alpha|0\rangle+\beta|1\rangle) + |01\rangle(\alpha|1\rangle+\beta|0\rangle) + |10\rangle(\alpha|0\rangle-\beta|1\rangle) + |11\rangle(\alpha|1\rangle-\beta|0\rangle)\,\Big].$$

This single line is the heart of the protocol: it shows that whatever pair of bits Alice measures, Bob's remaining qubit is $|\psi\rangle$ up to a known Pauli correction. Alice measures her two qubits — which destroys the entanglement — and sends the two classical outcomes to Bob over a classical channel. Bob then applies the matching correction:

$$
\begin{array}{c|c|c}
\text{Alice's outcome} & \text{Bob's pre-correction state} & \text{Correction} \\\\ \hline
00 & \alpha|0\rangle+\beta|1\rangle & \text{none} \\\\
01 & \alpha|1\rangle+\beta|0\rangle & X \\\\
10 & \alpha|0\rangle-\beta|1\rangle & Z \\\\
11 & \alpha|1\rangle-\beta|0\rangle & ZX
\end{array}
$$

For example, the $01$ branch is $|\psi\rangle$ "off by an $X$," so applying $X(\alpha|1\rangle+\beta|0\rangle) = \alpha|0\rangle+\beta|1\rangle = |\psi\rangle$; the $11$ branch carries both a bit-flip and a sign-flip, so Bob applies $X$ first and then $Z$. In all four cases Bob recovers exactly $|\psi\rangle$.

The interpretive discussion is where the depth lies. First, **no-cloning is respected**: the original copy of $|\psi\rangle$ at Alice is destroyed by her measurement, so no second copy is ever created — consistent with the impossibility of copying an arbitrary unknown state. Second, **there is no instantaneous transfer and no violation of special relativity**: Bob cannot apply his corrections until the two classical bits arrive, and those bits travel no faster than light, so the protocol's speed is bounded by the classical channel. Third, the transfer is exact to arbitrary precision in $\alpha,\beta$ even though the state is unknown — imagine $\alpha$ encoding the complete works of Shakespeare in binary; Bob's reconstructed state carries that same amplitude. Fourth, if $|\psi\rangle$ were itself entangled with a third party (Charlie), one can verify that Bob's final qubit remains entangled with Charlie in exactly the same way, even though Bob never interacted with Charlie. The unifying punchline: the bit pattern $ab$ behaves like an **error syndrome**, flagging whether an $X$ error (bit-flip), a $Z$ error (sign-flip), or both occurred. Because $X$ and $Z$ together generate every single-qubit error, teleportation is structurally a special instance of quantum error correction — "correcting" the discrepancy between what Alice intended to send and what Bob holds before applying his gates.

---

## 4. Points of Confusion and Corner Cases

The most heavily emphasized source of friction is the **correct reading of the entangled state** $|\phi^{+}\rangle$. It must not be read as "Alice has $|+\rangle$ and Bob has $|0\rangle/|1\rangle$." The superscript $+$ is merely a label on the Bell state, not a tensor factor; and the right assignment is that, within each pair, the *first* qubit is Alice's and the *second* is Bob's — not that one entire side belongs to one party. Most calculation mistakes trace back to mislabeling which qubit sits with whom.

A second delicate point is **global vs. relative phase**. The $-1$ appearing in the $ab=11$ branch of superdense coding looks like it breaks the result, but it is experimentally undetectable because it vanishes when probabilities are formed: $-|11\rangle$ and $|11\rangle$ yield identical statistics. The crucial distinction is that a *global* phase is irrelevant, whereas a *relative* phase (e.g., the sign between $|0\rangle$ and $|1\rangle$ inside a superposition) is physically meaningful and observable.

**Gate order matters.** Swapping the controlled-$Z$ and controlled-$X$, or reversing Bob's two operations, changes the output — the same way that preparing a Bell state with $H$ then $CNOT$ differs from $CNOT$ then $H$. In teleportation's $11$ correction, $ZX$ specifically means apply $X$ first, then $Z$.

It is also clarified that the **classical bits $a$ and $b$ never change** during Alice's encoding; only the quantum state evolves. The qubit that ends up holding $b$ on Bob's side is already in its final value after the transmission step, and only the other qubit still needs the final $H$.

The entire treatment assumes an **idealized, noise-free regime** — perfect gates and perfect transmission. This is a strong pedagogical simplification; in reality, transmitting quantum information reliably over fiber or satellite is the hard, unsolved-at-scale engineering problem, and the factor-of-two advantage of superdense coding may not justify that infrastructure on its own. The point is a *proof of concept* that qubits can encode more than classical bits, not a deployment recommendation.

Finally, the word **"teleportation" is deliberately demystified**. It is not the instantaneous matter transfer of *Star Trek*; it is the transfer of a quantum *state* mediated by classical communication, with the original necessarily destroyed. A genuinely open, non-spiritual question is posed: *where* does the state reside in the interval after Alice has measured but before Bob has received the bits and corrected? It is neither fully with Alice (she measured it away) nor yet with Bob (he lacks the information to reconstruct it) — it is, in some sense, smeared across the two parties.

---

## 5. Study Questions

1. **In superdense coding, what does Alice transmit and what does Bob obtain?**
   → Alice transmits **one qubit** (plus the consumed pre-shared entanglement) and Bob recovers **two classical bits** $ab$.

2. **In what sense is teleportation the "inverse" of superdense coding?**
   → Teleportation sends **two classical bits** (plus shared entanglement) to transfer **one unknown quantum state**; superdense coding sends one qubit to transfer two classical bits. The classical and quantum resources swap roles.

3. **Why does teleportation violate neither the no-cloning theorem nor relativity?**
   → No-cloning: Alice's original state is destroyed by measurement, so no extra copy exists. Relativity: Bob needs the two classical bits (traveling at $\le c$) before correcting, so the transfer is not instantaneous.

4. **What correction does Bob apply for each received pair $00,01,10,11$ in teleportation?**
   → $00 \to$ nothing; $01 \to X$; $10 \to Z$; $11 \to ZX$ (apply $X$ first, then $Z$).

5. **Why does the $-1$ in the $ab=11$ branch of superdense coding not spoil the protocol?**
   → It is a **global phase**, unobservable under measurement because only $|{-1}|^{2}=1$ enters the probabilities; $-|11\rangle$ is physically indistinguishable from $|11\rangle$.

6. **In Qiskit, why run a circuit many times (e.g., 1024 shots), and what performs the run?**
   → Because quantum output is probabilistic and must be sampled to estimate its distribution; the run is performed by the `AerSimulator` (a classical simulation of the circuit), from which `get_counts` extracts the outcome tallies.

7. **Why is a classical register mandatory in a measured circuit, and what do single vs. double circuit lines represent?**
   → Measurement collapses a qubit into a classical bit that must be stored, so a `ClassicalRegister` is required; a single line carries quantum information and a double parallel line carries classical information.
