# QWorld Bootcamp — Session 6: Introduction to Quantum Algorithms & Grover's Algorithm

## 1. Technical Sheet

- **Session topic:** Mathematical machinery for quantum algorithms (reversible computation, Boolean oracles, phase kickback, the $n$-qubit Hadamard identity) culminating in a complete derivation of **Grover's algorithm** for unstructured search.
- **Key concepts:** Reversible transformations / Boolean oracle $B_f$; phase kickback; $H^{\otimes n}$ identity and the mod-2 inner product; Grover iterate $G$ as a rotation; quadratic speedup $\mathcal{O}(\sqrt{N})$.
- **Tools / Frameworks:** Primarily pure mathematics (linear algebra + Dirac notation). Qiskit is referenced only in passing (how one would implement $H$ on each qubit and build oracle/XOR circuits from CNOTs); no live coding.
- **Location in the bootcamp:** Transition block from *circuits and gates* into *advanced algorithms*. This is the on-ramp to **Grover's search and amplitude amplification**, and it is explicitly positioned as the bridge toward the next modules on combinatorial optimization (QUBO, Max-Cut, Ising) and variational algorithms.

## 2. Synopsis

This session assembles the minimal mathematical toolkit required to reason about oracle-based quantum algorithms and then deploys it to derive Grover's search from first principles. It opens with reversibility: every quantum transformation is unitary and therefore invertible, yet canonical classical gates such as AND are not, since the output bit does not determine the inputs. The resolution is to embed any Boolean function $f$ in a reversible unitary oracle $B_f:|a\rangle|b\rangle \mapsto |a\rangle|b \oplus f(a)\rangle$, which is its own inverse and stores the result via mod-2 addition into an ancilla. A pivotal refinement is phase kickback: initializing the ancilla in $|-\rangle$ leaves it unchanged while transferring the function value into a global sign, $B_f|a\rangle|-\rangle = (-1)^{f(a)}|a\rangle|-\rangle$, which both prevents unwanted entanglement and yields the phase oracle used throughout. The treatment then generalizes the Hadamard transform to $n$ qubits, establishing $H^{\otimes n}|a\rangle = \tfrac{1}{\sqrt{2^n}}\sum_b (-1)^{a\cdot b}|b\rangle$, where the mod-2 inner product governs every interference sign and, on $|0^n\rangle$, produces the uniform superposition. With these primitives, Grover's algorithm is framed as unstructured search over $N = 2^n$ candidates: prepare uniform superposition, iterate $G = -H^{\otimes n} Z_0 H^{\otimes n} Z_f$, and measure. By collapsing the exponential state space into the two-dimensional span of good and bad subspaces $\{|A\rangle, |B\rangle\}$, $G$ is revealed to act as a rotation by $2\theta$ with $\sin\theta = \sqrt{a/N}$. Choosing $k \approx \tfrac{\pi}{4}\sqrt{N}$ rotations aligns the state with the target, delivering the provably optimal quadratic speedup over the classical $\Omega(N)$ bound.

## 3. Subtopic Breakdown

### Reversible Transformations and Classical Gates as Unitaries

The conceptual starting point is that a quantum transformation is, by definition, represented by a unitary matrix, and unitaries are always invertible. This immediately clashes with classical digital logic, where standard gates discard information. The AND gate is the canonical example: it maps two input bits $x, y$ to the single output bit $AND(x,y)$, and given only that output one generally cannot recover the inputs — if the output is $1$ then both inputs were $1$, but if it is $0$ then three distinct input pairs are consistent with it. Because the input information is not present in the output, the map is irreversible. The presenter ties this directly to physics through Landauer's principle: the irreversibility of the millions of AND gates in a laptop is exactly why erasing information dissipates heat, which is part of why the device warms up.

The classical fix, developed historically in the theory of reversible computing, is to enlarge the system. Two inputs should map to (at least) two outputs; the trick is to use three bits, where a third bit is initialized to $0$ and ends up holding the result while the two original inputs pass through unchanged: $\tilde{AND}:(x,y,0)\mapsto(x,y,AND(x,y))$. At the cost of a little extra memory (an ancilla), any Boolean function becomes reversible. Lifted into the quantum setting, this becomes the Boolean oracle

$$
B_f:\;|a\rangle|b\rangle \longrightarrow |a\rangle|b \oplus f(a)\rangle, \qquad f:\{0,1\}^n \to \{0,1\}.
$$

The first register $|a\rangle$ (which may be $n$ qubits) passes through unchanged; the second register accumulates $f(a)$ via XOR (mod-2 addition). Setting $b = 0$ leaves exactly $f(a)$ in the second register. Crucially, $B_f$ is its own inverse: applying it twice gives $|b \oplus f(a) \oplus f(a)\rangle = |b\rangle$, since the two parities cancel. For the rest of the session $B_f$ is treated as an oracle / black box — its internal circuit depends on the particular $f$ (simple cases can be built from CNOTs), and the concrete construction is deferred.

### Phase Kickback

Phase kickback is the trick that turns the oracle from something that writes a bit into something that writes a sign, and it is used in many (not all) quantum algorithms. Instead of initializing the ancilla to a computational-basis state, initialize it to $|-\rangle = \tfrac{1}{\sqrt2}(|0\rangle - |1\rangle)$ and apply $B_f$. Pushing $B_f$ through the superposition and using the definition gives

$$
B_f|a\rangle|-\rangle = |a\rangle\,\tfrac{1}{\sqrt2}\big(|0 \oplus f(a)\rangle - |1 \oplus f(a)\rangle\big).
$$

Resolving by cases makes the effect explicit. If $f(a)=0$, the ancilla is $\tfrac{1}{\sqrt2}(|0\rangle - |1\rangle) = |-\rangle$. If $f(a)=1$, it is $\tfrac{1}{\sqrt2}(|1\rangle - |0\rangle) = -|-\rangle$. Both cases combine into the single statement

$$
B_f:\;|a\rangle|-\rangle \longrightarrow (-1)^{f(a)}|a\rangle|-\rangle.
$$

The striking observation is that the ancilla is returned **unchanged**: the value of $f(a)$ has been "kicked" out of the ancilla's state and into the phase in front of the whole state. This is more than cosmetic. Because the ancilla is left exactly as it was, it cannot become entangled with the input register — a real risk if the function value were left stored in it. This effect is special to $|-\rangle$; the analogous computation with $|+\rangle$ does not produce the sign. Phase kickback becomes powerful precisely when the input register $|a\rangle$ is itself in superposition over many values of $a$, which is exactly the regime quantum algorithms exploit; the oracle then stamps a function-dependent sign onto every branch simultaneously.

### The Hadamard Transform on $n$ Qubits

The third primitive is a closed form for Hadamard applied across many qubits, so the long term-by-term expansions used in earlier sessions are no longer necessary. For a single qubit, combining the images of $|0\rangle$ and $|1\rangle$ into one formula gives

$$
H|a\rangle = \frac{1}{\sqrt2}\big(|0\rangle + (-1)^a|1\rangle\big) = \frac{1}{\sqrt2}\sum_{b\in\{0,1\}}(-1)^{a\cdot b}|b\rangle.
$$

Applying $H \otimes H$ to a two-qubit basis state and multiplying the two single-qubit sums yields a double sum whose sign is controlled by $a\cdot b = a_1 b_1 \oplus a_2 b_2$. The pattern generalizes immediately to $n$ qubits:

$$
H^{\otimes n}|a\rangle = \frac{1}{\sqrt{2^n}}\sum_{b\in\{0,1\}^n}(-1)^{a\cdot b}|b\rangle, \qquad a\cdot b = a_1 b_1 \oplus a_2 b_2 \oplus \dots \oplus a_n b_n.
$$

The single governing rule is that a minus sign appears in front of a basis term exactly when the mod-2 inner product between the input string and the output string has odd parity. Applied to the all-zero input $|0^n\rangle$, every inner product vanishes and all signs are positive, so

$$
H^{\otimes n}|0^n\rangle = \frac{1}{\sqrt{2^n}}\sum_{x}|x\rangle \equiv |h\rangle, \qquad N = 2^n,
$$

the uniform superposition over all $N$ strings. This is the formal content behind the popular slogan that a quantum computer "considers all possibilities at once." A student question prompts an important clarification: $H$ is best understood not as a single rotation but as a **reflection** about the axis at $\pi/8$ on the Bloch circle, because rotating $|0\rangle \to |+\rangle$ needs $45^\circ$ while $|1\rangle \to |-\rangle$ needs a different amount, so no single rotation angle captures both. It is also stressed that $H^{\otimes n}$ is genuinely a tensor product of independent single-qubit Hadamards (in Qiskit, just apply $H$ to each wire), not an entangling joint gate.

### Grover's Algorithm: Statement, Geometry, and Iteration Count

Grover's algorithm solves **unstructured search** in the black-box model: given $f:\{0,1\}^n\to\{0,1\}$ accessible only as an oracle, find a string $x$ with $f(x)=1$. "Unstructured" is contrasted with searching a sorted dictionary, where binary search exploits ordering to discard half the space per query in $\mathcal{O}(\log N)$ time. Here, learning that $f$ is $0$ on one string tells you nothing about any other string, so classically you must, in the worst case, probe essentially all candidates: $\Omega(N)$ queries with $N = 2^n$. Grover achieves $\mathcal{O}(\sqrt{N})$.

The algorithm has three steps: (1) initialize an $n$-qubit register in $|0^n\rangle$ and apply $H^{\otimes n}$ to get $|h\rangle$; (2) apply the Grover iterate

$$
G = -H^{\otimes n} Z_0 H^{\otimes n} Z_f
$$

a total of $k$ times; (3) measure. The two sign operators are the phase oracle $Z_f|x\rangle = (-1)^{f(x)}|x\rangle$ (this is $B_f$ used via phase kickback, written in condensed form that suppresses the ancilla) and the zero-reflection $Z_0|x\rangle = -|x\rangle$ if $x = 0^n$ and $|x\rangle$ otherwise. The minus sign in $G$ can be moved around freely.

The key analytical move is to avoid tracking all $N$ amplitudes. Since only the binary label good/bad matters, define the good set $A = \{x : f(x)=1\}$ with $a = |A|$ and the bad set $B = \{x : f(x)=0\}$ with $b = |B|$, and the uniform superpositions

$$
|A\rangle = \frac{1}{\sqrt{a}}\sum_{x\in A}|x\rangle, \qquad |B\rangle = \frac{1}{\sqrt{b}}\sum_{x\in B}|x\rangle.
$$

These are orthogonal and span a two-dimensional plane; the state stays in it at all times as $\alpha|A\rangle + \beta|B\rangle$. The initial state decomposes as $|h\rangle = \sqrt{a/N}\,|A\rangle + \sqrt{b/N}\,|B\rangle$. Writing $Z_0 = I - 2|0^n\rangle\langle 0^n|$ in Dirac notation and conjugating by Hadamards collapses the middle of $G$ neatly, because the outer Hadamards cancel against the identity and act on $|0^n\rangle$ to produce $|h\rangle$:

$$
H^{\otimes n} Z_0 H^{\otimes n} = H^{\otimes n}(I - 2|0^n\rangle\langle 0^n|)H^{\otimes n} = I - 2|h\rangle\langle h|.
$$

For the good subspace, $Z_f|A\rangle = -|A\rangle$ (every $x\in A$ has $f(x)=1$), which cancels the leading minus sign, so $G|A\rangle = (I - 2|h\rangle\langle h|)|A\rangle$. Using $\langle h|A\rangle = \sqrt{a/N}$ and re-expanding $|h\rangle$ gives

$$
G|A\rangle = \Big(1 - \frac{2a}{N}\Big)|A\rangle - \frac{2\sqrt{ab}}{N}|B\rangle,
$$

and the analogous (left-as-exercise) computation gives

$$
G|B\rangle = \frac{2\sqrt{ab}}{N}|A\rangle - \Big(1 - \frac{2b}{N}\Big)|B\rangle.
$$

Collecting these into a $2\times2$ matrix $M$ in the $\{|B\rangle, |A\rangle\}$ basis and substituting $\sin\theta = \sqrt{a/N},\ \cos\theta = \sqrt{b/N}$, one recognizes $M$ as a rotation matrix squared — that is, a single application of $G$ rotates the state by angle $2\theta$ in the plane. Since $|h\rangle = \cos\theta\,|B\rangle + \sin\theta\,|A\rangle$ already sits at angle $\theta$ from $|B\rangle$, applying $G$ $k$ times gives

$$
G^k|h\rangle = \cos\big((2k+1)\theta\big)|B\rangle + \sin\big((2k+1)\theta\big)|A\rangle.
$$

We want the amplitude on $|A\rangle$ near $1$, i.e. $(2k+1)\theta \approx \tfrac{\pi}{2}$, giving $k \approx \tfrac{\pi}{4\theta} - \tfrac{1}{2}$. With a single marked item ($a = 1$) and large $N$, the small-angle approximation $\theta = \sin^{-1}\sqrt{1/N} \approx \sqrt{1/N}$ yields

$$
k \approx \frac{\pi}{4}\sqrt{N}.
$$

Each $G$ contains exactly one oracle call, so Grover finds the marked item with high probability using $\mathcal{O}(\sqrt{N}) = \mathcal{O}(\sqrt{2^n})$ queries — the promised quadratic speedup, which (unlike Shor's exponential advantage) is *provably optimal* for this problem on a quantum computer.

## 4. Points of Confusion and Corner Cases

Several recurring friction points were flagged explicitly. First, the symbol overloading: $\oplus$ here denotes XOR / mod-2 parity on bits, which is **not** the tensor product $\otimes$ (acting on vectors/matrices) and not the direct sum, even though similar circle-symbols are used; context disambiguates, but conflating them is a classic error. Relatedly, XOR is a classical operation, yet it is reversible and therefore implementable as a quantum gate (small cases via CNOTs), so "how can XOR act on a qubit?" is resolved by noting reversibility.

Second, the oracle $B_f$ is a **joint** operation on both registers — the presenter corrected a slide typo that made it look as if $B_f$ acted on the ancilla alone; it must be applied to the input and ancilla together, e.g. to $|a\rangle|0\rangle$ and $|a\rangle|1\rangle$ branch-by-branch when the ancilla is in superposition.

Third, phase kickback requires the $|-\rangle$ ancilla; repeating the algebra with $|+\rangle$ does not yield the phase, and the whole point is that the ancilla returns unchanged so it cannot entangle with the input. Also note the condensed $Z_f$ notation hides the auxiliary qubit(s) actually needed to compute $f$ and kick it into the phase.

Fourth, Hadamard is a reflection (about the $\pi/8$ axis), not a single-angle rotation — a frequent misconception when students try to picture $|0\rangle \to |+\rangle$ and $|1\rangle \to |-\rangle$ as one rotation. And $H^{\otimes n}$ is a tensor product of independent gates, not an entangling joint gate.

Fifth, in the complexity statement, $\Omega(2^n)$ is in terms of the number of *rows* of the truth table $N = 2^n$, not the bit-length $n$ — a common index confusion. Finally, the most important Grover-specific subtlety: **more iterations is not better**. The state rotates monotonically toward $|A\rangle$ and then *past* it; undershooting leaves you short of the target, overshooting reduces success probability. The iteration count $k$ must be chosen (and ideally rounded with a ceiling after dropping the $-\tfrac12$, which is negligible for large $N$). The analysis assumed a known number of marked items (here $a=1$, the hardest case); when $a$ is unknown, one guesses $a$, runs Grover, checks the output classically, and updates the guess.

## 5. Study Questions

1. **Q:** Why can't the classical AND gate be used directly as a quantum gate, and how is it fixed?
   **A:** AND is irreversible (the output doesn't determine the inputs), but quantum gates must be unitary/reversible. Fix: add an ancilla, $(x,y,0)\mapsto(x,y,AND(x,y))$, keeping the inputs.

2. **Q:** State the action of the Boolean oracle $B_f$ and explain why it is its own inverse.
   **A:** $B_f|a\rangle|b\rangle = |a\rangle|b\oplus f(a)\rangle$. Applying it twice gives $b\oplus f(a)\oplus f(a) = b$, so $B_f^2 = I$.

3. **Q:** What is phase kickback, and which ancilla state produces it?
   **A:** With the ancilla in $|-\rangle$, $B_f|a\rangle|-\rangle = (-1)^{f(a)}|a\rangle|-\rangle$: the function value moves into a phase and the ancilla is unchanged. It does not occur with $|+\rangle$.

4. **Q:** Write the $n$-qubit Hadamard identity and the rule for when a minus sign appears.
   **A:** $H^{\otimes n}|a\rangle = \tfrac{1}{\sqrt{2^n}}\sum_b (-1)^{a\cdot b}|b\rangle$; a minus sign appears when the mod-2 inner product $a\cdot b$ is odd.

5. **Q:** In Grover's algorithm, how does $G$ act geometrically and how many iterations are optimal for one marked item?
   **A:** $G$ rotates the state by $2\theta$ (with $\sin\theta=\sqrt{a/N}$) toward $|A\rangle$ in the $\{|A\rangle,|B\rangle\}$ plane; for $a=1$, $k \approx \tfrac{\pi}{4}\sqrt{N}$, giving $\mathcal{O}(\sqrt{N})$ queries.

6. **Q:** Why is running Grover "longer" potentially harmful?
   **A:** The state rotates past $|A\rangle$ if you over-iterate (overshoot), lowering the success probability; the correct $k$ must be used, not the largest possible.
