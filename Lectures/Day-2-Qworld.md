# Resumen Técnico de Sesión — Introduction to Classical Systems 
---

## 1. Ficha Técnica

**Tema de la sesión:** Fundamentos matemáticos de sistemas clásicos: de la computación determinista a la probabilística, vectores de probabilidad, matrices estocásticas, producto tensorial y la operación CNOT como transformación controlada.

**Conceptos clave:**
- Sistemas clásicos deterministas y su representación mediante funciones bit a bit
- Vectores de probabilidad y combinaciones convexas como modelo del estado probabilístico
- Matrices estocásticas como transformaciones válidas de sistemas clásicos
- Producto tensorial de vectores y matrices para sistemas compuestos
- Operación CNOT como ejemplo de transformación clásica controlada no separable

**Herramientas/Frameworks mencionados:** Matemática pura (álgebra lineal, probabilidad combinatoria). Jupyter Notebooks del repositorio QWorld (Bronze). No se usa Qiskit ni Cirq en esta sesión.

---

## 2. Sinopsis

La sesión constituye el primer bloque técnico formal del bootcamp y tiene como objetivo construir, desde cero, el formalismo matemático que describe los sistemas clásicos de información. La motivación central es estratégica: antes de introducir la mecánica cuántica, se establece un marco algebraico para la computación clásica de modo que, cuando el formalismo cuántico se presente en la siguiente sesión, las similitudes y diferencias sean inmediatamente comparables y apreciables.

La exposición parte de la noción más elemental de sistema físico: un dispositivo con un conjunto finito de estados etiquetados, cuyo cambio puede ser determinista o probabilístico. Para el caso determinista de un bit clásico, se enumeran las cuatro funciones posibles $f:\{0,1\} \to \{0,1\}$ (constante-0, identidad, negación, constante-1), y se generaliza el razonamiento al caso de $n$ bits, obteniendo $2^{2^n}$ funciones posibles de decisión, una cantidad super-exponencial que ilustra la inmensidad del espacio de búsqueda en computación y aprendizaje automático.

A continuación, la sesión introduce el modelo probabilístico: cuando no se tiene información completa sobre el estado del sistema, este se representa mediante un vector de probabilidad $\hat{v} = (p,\, 1-p)^\top$, con entradas no negativas que suman 1. Todo estado probabilístico se expresa como combinación convexa de los estados deterministas extremos. Las transformaciones válidas sobre estos vectores son precisamente las matrices estocásticas: matrices cuadradas con entradas no negativas y columnas que son vectores de probabilidad. Cada entrada $(i,j)$ de la matriz admite una interpretación directa como probabilidad condicional de transición al estado $i$ dado que el sistema se encontraba en el estado $j$.

Para sistemas compuestos, se introduce el producto tensorial como el mecanismo natural para combinar vectores de probabilidad independientes, obteniendo vectores de dimensión $2^n$ que representan todas las probabilidades conjuntas. Un resultado clave es que no todo vector de probabilidad de dimensión 4 admite descomposición tensorial en vectores de dimensión 2, lo que refleja la existencia de correlaciones clásicas entre sistemas. Lo mismo ocurre con las matrices de transformación: no toda matriz estocástica $4\times 4$ es el producto tensorial de dos matrices $2\times 2$. La sesión cierra con la construcción explícita de la matriz $4\times 4$ de la operación CNOT (Controlled-NOT) como ejemplo paradigmático de transformación clásica controlada que crea correlaciones y, por tanto, no admite descomposición tensorial.

---

## 3. Desglose por Subtemas

### 3.1 Sistemas clásicos deterministas: espacio de estados y funciones de transición

El punto de partida formal es la abstracción de un sistema físico como un dispositivo $X$ con un conjunto finito no vacío de estados posibles, denominado alfabeto $\Sigma$. Para el caso más elemental, un bit clásico, $\Sigma = \{0, 1\}$. Para $n$ bits combinados, el espacio de estados es el conjunto de todas las cadenas binarias de longitud $n$, con cardinalidad $|\Sigma^n| = 2^n$. El énfasis es que estas cadenas deben tratarse como cadenas de símbolos, no como representaciones enteras, porque el estado del sistema compuesto codifica el estado individual de cada uno de sus $n$ componentes.

Una transformación determinista de un bit a un bit es una función $f: \{0,1\} \to \{0,1\}$. Existen exactamente cuatro tales funciones: la constante-0, la identidad, la negación y la constante-1. La enumeración de estas cuatro funciones sirve como ejercicio para introducir el concepto de tabla de verdad y para preparar la transición hacia la representación matricial.

La generalización al caso de $n$ bits como entrada y 1 bit como salida —es decir, el caso de un problema de decisión— revela que la cantidad de funciones posibles es $2^{2^n}$. Esto se obtiene notando que una función queda determinada por la asignación de un valor binario a cada una de las $2^n$ entradas posibles; el número total de tales asignaciones es exactamente $2^{2^n}$. Esta cantidad crece super-exponencialmente en $n$, lo que motiva la reflexión sobre la dificultad inherente de aprender una función específica en un espacio tan vasto, analogía directamente aplicable al aprendizaje automático.

### 3.2 Sistemas clásicos probabilísticos: vectores de probabilidad y combinaciones convexas

Cuando el estado del sistema no se conoce con certeza, se representa mediante un vector de probabilidad. Para un bit clásico en el estado $x$ con probabilidades $p$ y $1-p$:

$$
\hat{v} = \begin{pmatrix} p \\ 1-p \end{pmatrix}, \quad p \in [0, 1]
$$

donde la componente $v_\sigma$ indica la probabilidad de que el sistema se encuentre en el estado etiquetado con $\sigma$. Esta representación es abstracta: cuando se observa el sistema, no se "ve" el vector $\hat{v}$, sino que se obtiene un resultado concreto $\sigma \in \Sigma$ con probabilidad $v_\sigma$. Este punto se subraya especialmente porque tendrá un análogo profundo en el formalismo cuántico, donde el estado tampoco es directamente observable.

Todo vector de probabilidad de dimensión 2 puede escribirse como combinación convexa de los dos vectores deterministas $\mathbf{e}_0 = (1,0)^\top$ y $\mathbf{e}_1 = (0,1)^\top$:

$$
\hat{v} = p \begin{pmatrix} 1 \\ 0 \end{pmatrix} + (1-p) \begin{pmatrix} 0 \\ 1 \end{pmatrix}
$$

Una combinación convexa es una combinación lineal en la que los coeficientes son no negativos y suman 1, es decir, son ellos mismos probabilidades. Geométricamente, el espacio de todos los estados probabilísticos posibles de un bit clásico es el segmento de recta que une los dos puntos deterministas $\mathbf{e}_0$ y $\mathbf{e}_1$ en $\mathbb{R}^2$. El parámetro $p$ recorre este segmento, y la geometría del espacio de estados clásico es, por tanto, estrictamente unidimensional. Esta imagen geométrica se anticipa como punto de contraste con el estado cuántico, cuyo espacio de estados es mucho más rico.

### 3.3 Matrices estocásticas como transformaciones probabilísticas

Las transformaciones válidas sobre vectores de probabilidad son las matrices estocásticas. Una matriz $M$ es estocástica si y solo si:

1. Todas sus entradas son no negativas: $M_{ij} \geq 0$.
2. Cada columna es un vector de probabilidad: $\sum_i M_{ij} = 1$ para todo $j$.

La interpretación de cada entrada $M_{ij}$ es la de una probabilidad condicional: es la probabilidad de que el sistema transite al estado $i$ dado que se encontraba en el estado $j$. Esta interpretación surge de forma natural cuando se exige que el resultado de aplicar la matriz a un vector de probabilidad válido sea también un vector de probabilidad válido.

Para el caso de un bit clásico, las cuatro funciones deterministas se corresponden con cuatro matrices estocásticas $2\times 2$:

$$
A_1 = \begin{pmatrix} 1 & 1 \\ 0 & 0 \end{pmatrix}, \quad A_2 = \begin{pmatrix} 1 & 0 \\ 0 & 1 \end{pmatrix}, \quad A_3 = \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix}, \quad A_4 = \begin{pmatrix} 0 & 0 \\ 1 & 1 \end{pmatrix}
$$

correspondientes a constante-0, identidad, negación y constante-1, respectivamente. Cualquier matriz estocástica se puede obtener como combinación convexa de estas cuatro matrices deterministas, lo que proporciona una descripción alternativa del conjunto de transformaciones probabilísticas válidas.

Como ejercicio integrador, se analiza el siguiente protocolo sobre una moneda con probabilidades de cara $q$ y cruz $p = 1-q$:

1. Lanzar la moneda.
2. Si el resultado es cara, volver a lanzarla (flip probabilístico, matriz de probabilidades $\begin{pmatrix} p & p \\ q & q \end{pmatrix}$ ... no, ver abajo).
3. Si el resultado es cruz, voltear la moneda determinísticamente para obtener cara.

La matriz de transformación del segundo paso es:

$$
M = \begin{pmatrix} 0 & p \\ 1 & q \end{pmatrix}
$$

donde la primera columna corresponde al caso "estado=cruz" (transición determinista a cara: probabilidad 1 de pasar a cara, 0 de permanecer en cruz) y la segunda al caso "estado=cara" (relanzamiento con probabilidades $p$ de cruz y $q$ de cara). Aplicando $M$ al vector de estado inicial $\hat{v} = (p, q)^\top$, el estado final resulta ser $(pq,\; p + q^2)^\top$, obtenido por multiplicación matricial estándar.

### 3.4 Producto tensorial: sistemas compuestos, correlaciones y la operación CNOT

Para dos sistemas independientes con vectores de probabilidad $\hat{u} = (p, q)^\top$ y $\hat{w} = (r, s)^\top$, el estado del sistema compuesto se obtiene mediante el producto tensorial:

$$
\hat{u} \otimes \hat{w} = \begin{pmatrix} pr \\ ps \\ qr \\ qs \end{pmatrix}
$$

donde las entradas representan las probabilidades de los cuatro eventos conjuntos posibles: $\{00, 01, 10, 11\}$. El vector resultante tiene dimensión $2^2 = 4$ para dos bits, y $2^n$ en general para $n$ bits, lo que refleja el crecimiento exponencial del espacio de estados —un fenómeno que ya ocurre en el caso clásico y no es exclusivo de la mecánica cuántica.

Un resultado fundamental es que no todo vector de probabilidad de dimensión 4 admite descomposición tensorial. Por ejemplo, el vector $\hat{v} = (\frac{1}{2}, 0, 0, \frac{1}{2})^\top$ —que representa una distribución uniforme sobre los eventos $\{00, 11\}$, es decir, dos sistemas perfectamente correlacionados— no puede escribirse como $\hat{u} \otimes \hat{w}$ para ninguna elección de $\hat{u}$ y $\hat{w}$. La demostración es directa: si $ps = 0$, entonces $p = 0$ (lo que implica $pr = 0 \neq \frac{1}{2}$) o $s = 0$ (lo que implica $qs = 0 \neq \frac{1}{2}$). Esta indescomponibilidad refleja la existencia de correlaciones clásicas entre los dos bits y no es una característica exclusiva del entrelazamiento cuántico; es simplemente la descripción algebraica de variables aleatorias dependientes.

Lo mismo se aplica a las transformaciones: no toda matriz estocástica $4 \times 4$ es el producto tensorial de dos matrices $2 \times 2$. El ejemplo paradigmático es la operación CNOT (Controlled-NOT), cuya matriz $4 \times 4$ se construye explícitamente durante la sesión. Etiquetando los estados del sistema de dos bits como $\{00, 01, 10, 11\}$, donde el primer índice es el bit de control $x$ y el segundo es el bit objetivo $y$, la operación aplica la regla $(x, y) \mapsto (x,\, x \oplus y)$:

- Si $x = 0$: $y$ no cambia (bloque de identidad).
- Si $x = 1$: $y$ se niega (bloque NOT).

La matriz resultante es:

$$
\text{CNOT} = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 \\ 0 & 0 & 1 & 0 \end{pmatrix}
$$

Esta matriz no admite descomposición $A \otimes B$ para matrices $2\times 2$ porque la transformación genera correlaciones entre $x$ e $y$: el resultado en el bit objetivo depende del estado del bit de control, de modo que no puede describirse como dos operaciones independientes sobre cada subsistema.

---

## 4. Puntos de Confusión y Casos de Esquina

**El cero como entero versus el cero como cadena binaria.** Al escalar de 1 a $n$ bits, existe la tentación de simplificar la cadena $\underbrace{00\ldots0}_{n}$ como el entero 0. Sin embargo, el formalismo exige tratarla como una cadena de $n$ símbolos, cada uno indicando el estado de un subsistema individual. La simplificación aritmética destruye la información sobre qué sistema está en qué estado.

**Distinción entre $2^n$ (estados) y $2^{2^n}$ (funciones de decisión).** Un error frecuente es confundir el número de estados del sistema ($2^n$) con el número de funciones de decisión posibles sobre esos estados ($2^{2^n}$). El razonamiento correcto: una función de decisión asigna un bit a cada una de las $2^n$ entradas; hay $2^{2^n}$ maneras de hacer esa asignación. Esta distinción tiene implicaciones directas en teoría de la complejidad y en aprendizaje automático.

**Convexidad versus linealidad.** La combinación convexa no es simplemente una combinación lineal. Requiere adicionalmente que los coeficientes sean no negativos y sumen 1. Los estados probabilísticos son combinaciones convexas de estados deterministas, no combinaciones lineales arbitrarias. Confundir ambas nociones lleva a representar "estados" con entradas negativas, lo que carece de interpretación probabilística.

**La probabilidad no tiene realidad física directa.** Un punto enfatizado explícitamente durante la sesión: cuando se observa un sistema físico, se obtiene un resultado concreto, no el vector de probabilidad. La probabilidad cuantifica la información (o la ignorancia) del observador sobre el sistema, pero no es directamente observable. Este punto se anticipa como crucial para entender la diferencia entre estado cuántico y resultado de medición en la sesión siguiente.

**Indescomponibilidad tensorial como correlación, no como "magia cuántica".** Es habitual presentar la indescomponibilidad de un estado conjunto como una característica exclusivamente cuántica. Sin embargo, esta propiedad ya aparece en sistemas clásicos correlacionados. Lo que es genuinamente cuántico no es la indescomponibilidad en sí, sino el tipo específico de correlaciones que permite la mecánica cuántica (entrelazamiento), que excede lo que puede lograrse con sistemas clásicos.

**El CNOT no admite descomposición tensorial como transformación, aunque sí es una transformación determinista.** El CNOT es una permutación determinista de los estados del sistema de dos bits, por lo que es en particular una matriz estocástica (con entradas en $\{0,1\}$ y columnas que suman 1). Pero no puede escribirse como $A \otimes B$. La razón es que introduce correlaciones entre los dos subsistemas: la acción sobre el bit objetivo depende del estado del bit de control.

---

## 5. Para Estudiar

**Pregunta 1.** ¿Cuántas funciones $f: \{0,1\}^n \to \{0,1\}$ existen? ¿Por qué el resultado es $2^{2^n}$ y no $2^n$ ni $4^n$?

**Pregunta 2.** ¿Qué dos propiedades debe satisfacer una matriz para ser una matriz estocástica? ¿Cómo se interpreta cada entrada $M_{ij}$ en términos de probabilidades condicionales?

**Pregunta 3.** Dado el vector de probabilidad $\hat{v} = \begin{pmatrix} 1/2 \\ 0 \\ 0 \\ 1/2 \end{pmatrix}$, ¿puede escribirse como $\hat{u} \otimes \hat{w}$ para vectores de probabilidad de dimensión 2? Demuestre por qué sí o por qué no.

**Pregunta 4.** Describa la regla de transformación de la operación CNOT sobre los estados de dos bits $\{00, 01, 10, 11\}$ y construya su matriz $4\times 4$. ¿Por qué esta matriz no admite descomposición tensorial $A \otimes B$?

**Pregunta 5.** ¿Cuál es la diferencia entre una combinación convexa y una combinación lineal general? ¿Por qué el estado probabilístico de un bit clásico se representa como combinación convexa de los estados deterministas y no como una combinación lineal arbitraria?