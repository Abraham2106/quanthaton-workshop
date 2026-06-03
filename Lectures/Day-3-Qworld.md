# Resumen Técnico — Introducción a Sistemas Cuánticos

---

## 1. Ficha Técnica

**Tema de la sesión:** Fundamentos del qubit: estado cuántico, operaciones unitarias, notación de Dirac, compuerta de Hadamard y simulabilidad clásica de estados separables.

**Conceptos clave:**
- Qubit y superposición de amplitudes
- Regla de Born y extracción de probabilidades
- Matrices unitarias como transformaciones cuánticas válidas
- Notación bra-ket (Dirac) y base computacional
- Compuerta de Hadamard e interferencia cuántica
- Estados separables versus entrelazados y simulabilidad clásica

**Herramientas/Frameworks mencionados:** Matemática pura (álgebra lineal sobre números complejos/reales). Qiskit se menciona como herramienta para la sesión siguiente. No se utiliza ningún framework de programación en esta sesión.

---

## 2. Sinopsis

La sesión constituye el primer encuentro directo con la formalización matemática de un sistema cuántico, tomando como punto de partida el formalismo vectorial de probabilidad ya introducido en la sesión anterior para sistemas clásicos. El argumento conductor es simple pero poderoso: el único cambio que introduce la mecánica cuántica en la descripción vectorial de un sistema de dos niveles es reemplazar los escalares probabilísticos $p \in [0,1]$ por amplitudes $\alpha, \beta \in \mathbb{C}$. Este único paso abre un espacio de estados radicalmente más rico, aunque impone nuevas restricciones de normalización que se derivan de la Regla de Born.

A partir de esta definición del qubit como vector de amplitudes, la sesión construye progresivamente el resto del formalismo: la condición $\alpha^2 + \beta^2 = 1$ (para amplitudes reales) permite visualizar el estado como un punto en el círculo unitario; las transformaciones válidas sobre estados cuánticos se identifican con matrices unitarias (ortogonales en el caso real), que preservan la norma y son reversibles; la notación bra-ket se introduce como una convención notacional sin nueva matemática, aunque con ventajas computacionales evidentes. La compuerta de Hadamard sirve como primer ejemplo concreto de transformación unitaria con entradas negativas, y su análisis comparativo frente a la moneda clásica justa ilustra de forma precisa el fenómeno de interferencia cuántica: aplicar Hadamard dos veces sin medir en el medio produce una evolución completamente determinista ($|0\rangle \to |0\rangle$, $|1\rangle \to |1\rangle$), mientras que insertar una medición intermedia destruye esa cancelación y restituye el comportamiento estadístico. La sesión cierra con una motivación inicial del entrelazamiento: los estados de $n$ qubits que admiten una descomposición en producto tensorial de qubits individuales (estados separables) pueden simularse eficientemente en un computador clásico con memoria $\mathcal{O}(n)$, lo que implica que cualquier ventaja computacional cuántica genuina debe surgir de estados no separables.

---

## 3. Desglose por Subtemas

### 3.1 Del bit clásico al qubit: amplitudes y Regla de Born

El sistema clásico de dos niveles se describe mediante un vector de probabilidad convexo:

$$\hat{v} = p\begin{pmatrix}1\\0\end{pmatrix} + (1-p)\begin{pmatrix}0\\1\end{pmatrix} = \begin{pmatrix}p\\1-p\end{pmatrix}, \quad p \geq 0$$

La transición al qubit consiste en sustituir los escalares probabilísticos por amplitudes $\alpha, \beta \in \mathbb{C}$:

$$|\psi\rangle = \alpha\begin{pmatrix}1\\0\end{pmatrix} + \beta\begin{pmatrix}0\\1\end{pmatrix} = \begin{pmatrix}\alpha\\\beta\end{pmatrix}$$

Las amplitudes no son probabilidades directamente. La conexión entre ambas la provee la **Regla de Born**: la probabilidad de observar el resultado $i$ es $|\alpha_i|^2$. Para amplitudes reales, esto equivale a $\alpha^2$ y $\beta^2$ respectivamente, con la restricción de normalización $\alpha^2 + \beta^2 = 1$.

Esta restricción admite una parametrización trigonométrica natural: $\alpha = \cos\theta$, $\beta = \sin\theta$, con $\theta \in [0, 2\pi)$, lo que representa geométricamente al qubit como un vector unitario en el círculo unitario. La identidad $\cos^2\theta + \sin^2\theta = 1$ garantiza automáticamente la normalización. Se menciona que cuando se levantan las restricciones a números complejos, la visualización completa del qubit corresponde a la esfera de Bloch en tres dimensiones, aunque su uso explícito queda fuera del alcance de la mayoría de las sesiones del bootcamp.

Un punto epistemológico relevante que se desarrolla con cuidado: la superposición no debe interpretarse ingenuamente como "el sistema está en los estados 0 y 1 al mismo tiempo". El ponente argumenta que tampoco interpretamos el vector de probabilidad clásico $\begin{pmatrix}p\\1-p\end{pmatrix}$ como que el sistema ocupa simultáneamente dos estados, sino como una descripción matemática del conocimiento incompleto. La superposición cuántica es simplemente una descripción en un espacio más rico, cuyas consecuencias físicas distintas se revelan a través del comportamiento en experimentos específicos, no a través de interpretaciones ontológicas directas.

### 3.2 Transformaciones cuánticas y matrices unitarias

Una transformación cuántica válida es una matriz $U$ tal que, aplicada a cualquier estado cuántico válido, produce otro estado cuántico válido. Formalmente, si $\sum_i \alpha_i^2 = 1$, entonces $\sum_i \beta_i^2 = 1$ donde $\hat{w} = U\hat{v}$.

Las matrices que satisfacen esta condición son exactamente las **matrices unitarias**. Para entradas reales, equivalen a matrices ortogonales y satisfacen:

$$U^T U = I \quad \Leftrightarrow \quad U^T = U^{-1}$$

En el caso complejo general:

$$\bar{U}^T U = I$$

donde $\bar{U}^T$ denota la transpuesta conjugada (adjunta o hermitiana). Una consecuencia fundamental es que $U^{-1}$ existe siempre, lo que significa que toda evolución cuántica unitaria es **reversible**: la información no se pierde en el proceso. Esto distingue fundamentalmente a las transformaciones cuánticas de las matrices estocásticas clásicas, que en general no son invertibles.

Se introduce también la clase de matrices **hermitianas** ($\bar{H}^T = H$), cuya relevancia quedará manifiesta en sesiones posteriores (en el contexto de observables y mediciones). Para los propósitos de esta sesión, el foco permanece en las unitarias.

### 3.3 Notación bra-ket (Dirac)

La notación de Dirac no introduce nueva matemática, sino una forma más concisa y expresiva de escribir vectores y productos internos. Los vectores columna se denominan **kets**:

$$|0\rangle = \begin{pmatrix}1\\0\end{pmatrix}, \qquad |1\rangle = \begin{pmatrix}0\\1\end{pmatrix}$$

Los vectores fila correspondientes (transpuestos conjugados) se denominan **bras**:

$$\langle 0| = \begin{pmatrix}1 & 0\end{pmatrix}, \qquad \langle 1| = \begin{pmatrix}0 & 1\end{pmatrix}$$

Un estado arbitrario se escribe:

$$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$$

El **producto interno** entre dos estados $|\phi\rangle = \gamma|0\rangle + \delta|1\rangle$ y $|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$ se denota $\langle\phi|\psi\rangle$ y se calcula expandiendo:

$$\langle\phi|\psi\rangle = (\bar{\gamma}\langle 0| + \bar{\delta}\langle 1|)(\alpha|0\rangle + \beta|1\rangle) = \bar{\gamma}\alpha\underbrace{\langle 0|0\rangle}_{1} + \bar{\gamma}\beta\underbrace{\langle 0|1\rangle}_{0} + \bar{\delta}\alpha\underbrace{\langle 1|0\rangle}_{0} + \bar{\delta}\beta\underbrace{\langle 1|1\rangle}_{1} = \bar{\gamma}\alpha + \bar{\delta}\beta$$

La simplificación se debe a la ortogonalidad de la base computacional. El conjunto $\{|0\rangle, |1\rangle\}$ se denomina **base computacional** o **base estándar**.

Para sistemas de $n$ qubits, la notación ket resulta especialmente conveniente porque el label interior codifica directamente el índice del vector de amplitudes. Un estado de $n$ qubits tiene dimensión $2^n$, y escribir en notación Dirac estados dispersos (con pocas entradas no nulas) es mucho más compacto que listar el vector completo.

El producto tensorial de dos estados individuales se escribe naturalmente en ket notation:

$$|\psi\rangle \otimes |\phi\rangle = (\alpha|0\rangle + \beta|1\rangle) \otimes (\gamma|0\rangle + \delta|1\rangle) = \alpha\gamma|00\rangle + \alpha\delta|01\rangle + \beta\gamma|10\rangle + \beta\delta|11\rangle$$

### 3.4 Compuerta de Hadamard e interferencia cuántica

La compuerta de Hadamard es la primera transformación cuántica concreta del bootcamp y sirve como herramienta para ilustrar un fenómeno esencialmente cuántico: la interferencia. Se define como:

$$H = \frac{1}{\sqrt{2}}\begin{pmatrix}1 & 1\\1 & -1\end{pmatrix}$$

Se puede verificar que $H^T H = I$ (es simétrica y unitaria), y que $H^T = H$, por lo que $H^2 = I$: Hadamard es su propia inversa.

Su acción sobre la base computacional produce la **base de Hadamard**:

$$H|0\rangle = \frac{1}{\sqrt{2}}\begin{pmatrix}1\\1\end{pmatrix} = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) \equiv |+\rangle$$

$$H|1\rangle = \frac{1}{\sqrt{2}}\begin{pmatrix}1\\-1\end{pmatrix} = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle) \equiv |-\rangle$$

Ambos estados, $|+\rangle$ y $|-\rangle$, muestran la misma distribución de probabilidad al medirse: $\frac{1}{2}$ para $|0\rangle$ y $\frac{1}{2}$ para $|1\rangle$. El signo negativo es, aparentemente, invisible a la medición directa.

La distinción con respecto al caso clásico se hace explícita mediante la comparación con la moneda justa clásica $F = \frac{1}{2}\begin{pmatrix}1&1\\1&1\end{pmatrix}$:

**Caso clásico:** $F$ aplicado a cualquier vector de probabilidad siempre produce $\frac{1}{2}\begin{pmatrix}1\\1\end{pmatrix}$. Aplicar $F$ dos veces en secuencia (con o sin observación intermedia) siempre devuelve la misma distribución uniforme. La observación intermedia no tiene impacto en el resultado final.

**Caso cuántico sin medición intermedia:** $H \circ H = I$. Aplicar Hadamard dos veces seguidas produce un comportamiento completamente determinista: $|0\rangle \to |+\rangle \to |0\rangle$ y $|1\rangle \to |-\rangle \to |1\rangle$. Esta cancelación ocurre porque la amplitud negativa de $|-\rangle$ permite que los términos se cancelen constructiva y destructivamente (interferencia). Este es el resultado cualitativamente distinto del caso clásico.

**Caso cuántico con medición intermedia:** Tras aplicar $H$ y medir, el estado colapsa a $|0\rangle$ o $|1\rangle$ con probabilidad $\frac{1}{2}$ cada uno. Aplicar $H$ nuevamente devuelve el sistema a $|+\rangle$ o $|-\rangle$ respectivamente, recuperando superposición. El resultado final ya no es determinista. En este caso, el comportamiento vuelve a coincidir cualitativamente con el clásico.

La conclusión es que **el signo negativo importa**, pero solo se manifiesta cuando no hay medición que interrumpa la coherencia cuántica del sistema. Esta diferencia cualitativa es la primera evidencia concreta de por qué la mecánica cuántica no es una simple generalización cosmética del probabilismo clásico.

### 3.5 Simulabilidad clásica y estados separables

Para motivar la relevancia computacional del entrelazamiento (tema de la siguiente sesión), se plantea la pregunta: ¿cuándo puede un sistema de $n$ qubits simularse eficientemente en un computador clásico?

Un estado de $n$ qubits es **separable** si puede escribirse como producto tensorial de estados individuales:

$$|\psi\rangle = (\alpha_1|0\rangle + \beta_1|1\rangle) \otimes \cdots \otimes (\alpha_n|0\rangle + \beta_n|1\rangle)$$

En este caso, almacenar y actualizar el estado completo solo requiere mantener los $n$ valores $\alpha_i$ (los $\beta_i$ se deducen de la normalización). La memoria necesaria es $\mathcal{O}(n)$ en lugar de $\mathcal{O}(2^n)$. Cualquier circuito cuántico que mantenga la separabilidad a lo largo de toda su evolución puede simularse en tiempo polinomial clásicamente, es decir, pertenece a la clase P.

El razonamiento inverso es igualmente importante: si existe un algoritmo cuántico que no puede simularse eficientemente de forma clásica (como se conjetura para el algoritmo de Shor), entonces su evolución debe pasar por estados que no son separables. Esto implica que **el entrelazamiento es un recurso necesario para la ventaja computacional cuántica genuina**. Se menciona además que el Teorema de Gottesman-Knill (referido como "Nil-Flamm" en la transcripción) extiende esta simulabilidad eficiente a una clase más amplia de estados con correlaciones no triviales, aunque de estructura restringida.

---

## 4. Puntos de Confusión y Casos de Esquina

**La superposición como "estar en dos estados al mismo tiempo."** Esta es la fuente de confusión más recurrente de la sesión. El ponente la aborda con cuidado: el vector de amplitudes $\alpha|0\rangle + \beta|1\rangle$ no implica necesariamente que el sistema esté simultáneamente en $|0\rangle$ y $|1\rangle$, del mismo modo que el vector de probabilidad clásico $\begin{pmatrix}p\\1-p\end{pmatrix}$ no implica que la moneda esté en cara y cruz al mismo tiempo. La superposición es una descripción matemática que vive en un espacio más rico; sus consecuencias físicas distintivas se manifiestan solo en escenarios específicos como la interferencia, no en la interpretación ontológica directa del formalismo.

**El signo negativo en las amplitudes y su visibilidad.** Una fuente de confusión es que $|+\rangle$ y $|-\rangle$ producen la misma distribución de probabilidad al medirse directamente ($\frac{1}{2}$ para 0 y $\frac{1}{2}$ para 1). Esto podría llevar a pensar que el signo negativo es irrelevante. No lo es: el signo negativo es el mecanismo que permite la interferencia destructiva y constructiva, visible cuando los estados evolucionan sin medición intermedia (como en $H \circ H = I$). La medición destruye esta coherencia.

**Amplitudes complejas versus reales.** La sesión trabaja con amplitudes reales para simplificar, pero esto no es la generalidad. Para algoritmos que requieren la Transformada de Fourier Cuántica (como el algoritmo de Shor), las amplitudes complejas son necesarias. La restricción a reales corresponde geométricamente a trabajar en el círculo unitario 2D en lugar de la esfera de Bloch 3D.

**Matrices unitarias versus hermitianas.** Se mencionan ambas clases. Las unitarias son las transformaciones de evolución cuántica reversibles ($U^\dagger U = I$). Las hermitianas ($H^\dagger = H$) corresponden a observables físicos y aparecerán más adelante. No deben confundirse entre sí, ni tampoco con la compuerta específica llamada Hadamard (que es tanto unitaria como hermitiana, pero esto es una coincidencia de esa compuerta, no la norma general).

**Separabilidad y entrelazamiento.** El hecho de que un estado sea "no separable" no basta para garantizar ventaja computacional cuántica. El Teorema de Gottesman-Knill muestra que hay estados entrelazados que aún admiten simulación eficiente clásica. La pregunta de qué tipo de entrelazamiento es necesario y suficiente para la ventaja computacional es un problema abierto.

---

## 5. Para Estudiar

1. Dado el estado $|\psi\rangle = \frac{\sqrt{3}}{2}|0\rangle - \frac{1}{2}|1\rangle$, verifica que está normalizado y calcula la probabilidad de obtener el resultado 1 al medirlo.

2. Demuestra que la compuerta de Hadamard $H = \frac{1}{\sqrt{2}}\begin{pmatrix}1&1\\1&-1\end{pmatrix}$ satisface $H^T H = I$ y concluye que es unitaria. Luego calcula $H^2$ y explica el resultado.

3. Explica la diferencia entre el estado $|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$ y el estado $|-\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle)$: ¿cuándo son indistinguibles experimentalmente y cuándo no?

4. Un circuito de $n = 100$ qubits opera exclusivamente sobre estados separables en todo momento. Argumenta por qué puede simularse con memoria $\mathcal{O}(n)$ en lugar de $\mathcal{O}(2^n)$, y qué implicación tiene esto para la necesidad del entrelazamiento en la computación cuántica.

5. Calcula el producto interno $\langle +|-\rangle$ usando la definición bra-ket y verifica que $|+\rangle$ y $|-\rangle$ son ortogonales. ¿Por qué esto es relevante para que formen una base del espacio de estados de un qubit?