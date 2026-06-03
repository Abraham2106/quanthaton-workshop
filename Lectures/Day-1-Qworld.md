# Resumen Técnico de Sesión — Prospects of Quantum Computing

---

## 1. Ficha Técnica

| Campo | Detalle |
|---|---|
| **Tema** | Panorámica histórica, computacional y de complejidad algorítmica de la computación cuántica |
| **Conceptos clave** | Tesis de Church-Turing (estándar y extendida), Clases P / NP / NP-completo / BQP, Ventaja cuántica, Algoritmo de Shor, Verificabilidad de soluciones |
| **Herramientas** | Ninguna de uso directo (sesión introductoria). Se mencionan IBM Quantum y Rigetti como ejemplos de acceso por la nube |

---

## 2. Sinopsis

La sesión inaugura el bootcamp con una conferencia panorámica que traza el arco histórico y conceptual de la computación cuántica desde sus raíces en la información cuántica hasta el estado actual del campo. El punto de partida es la distinción entre computación cuántica y el campo más amplio de la información cuántica: la distribución cuántica de clave (QKD), la metrología cuántica y el sensado cuántico son aplicaciones ya comercialmente viables que preexisten o coexisten con la computación cuántica propiamente dicha. La motivación original de Feynman —simular sistemas cuánticos en computadoras, tarea que crece exponencialmente en memoria y tiempo para máquinas clásicas— sitúa el problema fundamental que la computación cuántica pretende atacar.

El núcleo de la sesión es una introducción rigurosa pero accesible a la teoría de la complejidad computacional clásica, tratada como prerequisito conceptual imprescindible. La máquina de Turing, la tesis de Church-Turing y su versión extendida —que añade el requisito de *eficiencia* en la simulación entre modelos computacionales— proporcionan el lenguaje para formular con precisión qué significa que un problema sea "difícil". Las clases P y NP se definen con claridad operativa: P como el conjunto de problemas resolubles en tiempo polinomial determinista, y NP como aquellos para los cuales una solución candidata puede *verificarse* en tiempo polinomial. Esta distinción entre *generar* y *verificar* una solución resulta ser la clave para entender por qué la factorización entera —el problema que el algoritmo de Shor resuelve eficientemente en un ordenador cuántico— pertenece a NP pero no se sabe si pertenece a P.

La sesión culmina posicionando la clase BQP (Bounded-error Quantum Polynomial time) dentro del panorama de complejidad, mostrando que se espera que BQP no esté contenido en NP y viceversa, con consecuencias directas para la búsqueda de ventaja cuántica en la era NISQ: los problemas accesibles hoy en hardware cuántico no tienen, en general, ventaja cuántica demostrada formalmente, y los que sí la tienen (Shor, Grover) requieren hardware con corrección de errores que aún no existe a escala.

---

## 3. Desglose por Subtemas

### 3.1 El campo de la información cuántica y la motivación de Feynman

La computación cuántica se entiende mejor como una subdisciplina dentro de la información cuántica, un campo más amplio que incluye QKD, sensado cuántico y metrología. La motivación de Feynman —formulada alrededor de 1981-82, de forma casi simultánea a propuestas del físico soviético Yuri Manin— parte de una observación fundamental: el espacio de estados de un sistema cuántico de N partículas crece exponencialmente con N. Cualquier simulación de ese sistema en una computadora clásica requiere, por tanto, una memoria y un tiempo que se vuelven inviables con rapidez. La solución propuesta por Feynman es construir el propio computador sobre principios cuánticos: que el hardware comparta la misma naturaleza que el sistema a simular. Esta motivación original —simular física cuántica— sigue siendo una de las aplicaciones más sólidas y mejor fundamentadas del campo, junto a la factorización entera que Shor formalizaría una década después.

### 3.2 Tesis de Church-Turing y su versión extendida

La tesis de Church-Turing establece que todo problema computable puede resolverse en una máquina de Turing, modelo que captura matemáticamente cualquier computadora física convencional. La máquina de Turing opera con una memoria interna finita (la cabeza lectora) y una cinta externa potencialmente ilimitada sobre la que escribe y lee, resolviendo en cada paso qué símbolo escribir y en qué dirección moverse. Esta abstracción, aunque asume memoria ilimitada, es justificable porque el requerimiento de memoria siempre puede expresarse en función del tamaño N de la entrada: no se fija un techo absoluto, sino una cota en términos de N.

La versión extendida de la tesis (ECT) añade el requisito de *eficiencia*: no solo todos los modelos computacionales son equivalentes en computabilidad, sino que pueden *simularse eficientemente* entre sí, sin incurrir en costes exponenciales al traducir de una plataforma a otra. Es precisamente este requisito el que el algoritmo de Shor pone en cuestión: si la factorización entera es eficiente en un ordenador cuántico pero no en uno clásico, la ECT quedaría falsificada. Sin embargo, esta falsificación sigue siendo incompleta porque no existe una demostración de que ningún algoritmo clásico pueda factorizar en tiempo polinomial.

### 3.3 Clases de complejidad: P, NP, NP-completo y BQP

La clase **P** (tiempo polinomial determinista) recoge todos los problemas cuya solución puede generarse mediante un algoritmo acotado por un polinomio en el tamaño de la entrada: verificación de primalidad, programación lineal, conectividad en grafos. La clase **NP** (tiempo polinomial no determinista) se entiende de manera operativa como la clase de problemas para los cuales, dado un candidato a solución, su corrección puede *verificarse* en tiempo polinomial determinista. El ejemplo canónico es la factorización entera: dado el número 15 y la lista candidata {5, 3}, basta multiplicar para verificar que 5 × 3 = 15; generarlo requiere otro nivel de esfuerzo.

**NP-completo** designa el subconjunto de los problemas más difíciles dentro de NP (en el sentido de la reducibilidad polinomial): satisfacibilidad booleana (SAT), problema del viajante, empaquetado de contenedores, entre muchos otros. La creencia dominante —aunque no demostrada— es que P ≠ NP, problema abierto por el cual el Clay Mathematics Institute ofrece un millón de dólares desde hace más de 25 años.

**BQP** (Bounded-error Quantum Polynomial time) es la clase de problemas resolubles eficientemente en una computadora cuántica, definida formalmente sobre una *máquina de Turing cuántica*. La factorización entera pertenece a BQP gracias al algoritmo de Shor. Se cree que BQP contiene problemas fuera de NP (como ciertas simulaciones de sistemas cuánticos) y que NP-completo queda fuera de BQP. Esta disposición tiene una consecuencia crítica: los problemas en BQP pero fuera de NP no son verificables eficientemente de forma clásica, generando una crisis de confianza en los resultados de dispositivos cuánticos NISQ que pretendan resolverlos.

```
Relación esperada entre clases (bajo el supuesto P ≠ NP):

        ┌─────────────────────────────────────────┐
        │                 BQP                     │
        │    ┌──────────────────────────┐         │
        │    │          NP              │         │
        │    │   ┌───────────────┐      │         │
        │    │   │  NP-completo  │      │         │
        │    │   └───────────────┘      │         │
        │    │         ┌───┐            │         │
        │    │         │ P │            │         │
        │    │         └───┘            │         │
        │    │  Factorización ∈ NP\NPC  │         │
        │    └──────────────────────────┘         │
        │  (partes de BQP fuera de NP)            │
        └─────────────────────────────────────────┘
```

### 3.4 La era NISQ y la búsqueda de ventaja cuántica

El término NISQ (*Noisy Intermediate-Scale Quantum*), acuñado por John Preskill, describe el estado actual del hardware cuántico: dispositivos de cientos o pocos miles de qubits sin corrección de errores ni tolerancia a fallos completas, con tasas de error que se acumulan rápidamente y limitan la profundidad de los circuitos ejecutables. En este contexto, la pregunta no es "¿qué haré con un ordenador cuántico universal a gran escala?" sino "¿qué problema bien definido puedo resolver mejor que cualquier solución clásica conocida, usando el hardware ruidoso disponible hoy?".

La búsqueda de ventaja cuántica es un proceso dinámico y no un hito fijo: cuando un algoritmo cuántico supera a los clásicos, la comunidad clásica responde con mejoras algorítmicas o de hardware, y el ciclo se repite — de modo análogo a la carrera en criptografía entre longitudes de clave y poder de cómputo. Un ejemplo paradigmático es el fenómeno de *dequantización*: Ewin Tang demostró que un algoritmo cuántico de recomendación que parecía ofrecer ventaja exponencial podía simularse eficientemente de forma clásica, absorbiendo la idea cuántica clave sin necesitar hardware cuántico.

El "sweet spot" que la comunidad persigue es la intersección de tres propiedades: ventaja cuántica demostrada en principio, verificabilidad eficiente de la solución, e implementabilidad en hardware NISQ. Ningún problema conocido satisface hoy las tres simultáneamente:

| Problema / Algoritmo | Ventaja cuántica en principio | Verificable eficientemente | Implementable en NISQ |
|---|:---:|:---:|:---:|
| Algoritmo de Shor (factorización) | Sí | Sí | No |
| Algoritmo de Grover (búsqueda) | Sí (cuadrática) | Sí | Parcial |
| Boson Sampling | Sí | No | Sí |
| Random Circuit Sampling (Google) | Sí | No | Sí |
| Algoritmos variacionales (QAOA, VQE) | No (no demostrada) | Sí | Sí |
| **Sweet spot buscado** | **Sí** | **Sí** | **Sí** |

---

## 4. Puntos de Confusión y Casos de Esquina

**No determinismo ≠ probabilismo.** En teoría de la complejidad, "no determinismo" no significa comportamiento aleatorio. En el lenguaje de las ciencias naturales, no determinismo suele asociarse a aleatoriedad, pero en computación el no determinismo es un mecanismo formal —exploración simultánea de todas las ramas de cómputo— que se captura operativamente mediante la noción de verificación: un problema es NP si un verificador determinista puede comprobar cualquier solución candidata en tiempo polinomial. Esta redefinición operativa elimina la necesidad de invocar el concepto técnico de no determinismo y resulta más intuitiva.

**Factorización entera ∈ NP, pero ∉ NP-completo.** Un error frecuente es confundir la pertenencia de la factorización a NP con que sea un problema NP-completo. En realidad, la factorización se cree que pertenece a NP ∩ co-NP y no a NP-completo, lo que es, paradójicamente, uno de los indicios de que podría ser resoluble por un ordenador cuántico sin que eso implique que BQP ⊇ NP-completo.

**La ventaja cuántica NISQ no está formalizada.** Muchas afirmaciones de ventaja cuántica en el contexto de algoritmos variacionales (QAOA, VQE) son empíricas —"numéricamente parece mejor"— y no son demostraciones de separación exponencial. Esto es cualitativamente diferente de la ventaja cuadrática demostrada de Grover o la ventaja exponencial (condicionada) de Shor.

**La falsificación de la ECT es aún incompleta.** Aunque el algoritmo de Shor ofrece un candidato, la ausencia de una cota inferior que demuestre que la factorización *requiere* tiempo superpolinomial en computadoras clásicas deja la puerta abierta a que alguien encuentre un algoritmo clásico eficiente, eliminando la aplicación "killer" de la computación cuántica.

**Verificación como criterio de confianza en hardware NISQ.** Para problemas en BQP pero fuera de NP —como la simulación del estado base de moléculas grandes— el resultado de un computador cuántico no puede verificarse eficientemente de forma clásica ni simularse clásicamente para comprobarlo. No hay manera eficiente de saber si la salida del dispositivo cuántico es correcta. Esto constituye un problema de confianza abierto y activo en la investigación actual.

---

## 5. Para Estudiar

**Pregunta 1.** ¿Cuál es la diferencia operativa entre P y NP?

> P contiene problemas cuya solución puede *generarse* en tiempo polinomial determinista; NP contiene aquellos cuya solución puede *verificarse* en tiempo polinomial determinista, aunque generarla podría ser exponencialmente más costoso.

---

**Pregunta 2.** ¿Por qué el algoritmo de Shor no demuestra por sí solo que la factorización entera está en P?

> Porque Shor corre en un *computador cuántico*, no en uno clásico; la clase P se define sobre máquinas de Turing clásicas. La factorización pertenece a BQP, y demostrar que no está en P clásico requeriría una cota inferior que hoy no existe.

---

**Pregunta 3.** ¿Qué caracteriza a un problema NP-completo y por qué se cree que BQP no los resuelve eficientemente?

> Un problema NP-completo es aquel al que cualquier problema de NP se reduce en tiempo polinomial. Se cree que BQP ⊄ NP-completo porque ningún algoritmo cuántico conocido proporciona un mecanismo para resolver NP-completo en tiempo polinomial, y la estructura de la interferencia cuántica no parece ofrecer esa capacidad.

---

**Pregunta 4.** ¿Qué es la era NISQ y cuál es el "sweet spot" que la comunidad busca en ella?

> NISQ describe el hardware cuántico actual: ruidoso, sin corrección de errores completa, con cientos a pocos miles de qubits. El sweet spot es encontrar un problema que cumpla simultáneamente: ventaja cuántica demostrada en principio, solución verificable eficientemente de forma clásica, e implementable en hardware NISQ real. Actualmente ningún problema conocido cumple las tres condiciones a la vez.

---

**Pregunta 5.** ¿Por qué la dequantización de un algoritmo cuántico es relevante para evaluar la ventaja cuántica?

> Porque demuestra que la mejora observada podía obtenerse sin hardware cuántico, absorbiendo la idea clave del algoritmo cuántico en un algoritmo clásico. Esto obliga a probar explícitamente que el problema *no puede* simularse eficientemente de forma clásica para afirmar que existe ventaja genuina.