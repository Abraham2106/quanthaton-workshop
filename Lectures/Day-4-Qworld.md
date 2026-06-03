# Resumen Tecnico - Geometria Cuantica, Entrelazamiento y Circuitos

---

## 1. Ficha Tecnica

**Tema de la sesion:** Visualizacion geometrica de compuertas de un qubit en el circulo unitario, preparacion del estado de Bell, distincion entre correlacion clasica y entrelazamiento cuantico, compuerta Toffoli (CCNOT) y evaluacion completa de un circuito de 3 qubits con analisis de cancelacion de amplitudes.

**Conceptos clave:**
- Rotaciones y reflexiones en el circulo unitario
- Estado de Bell y base de Bell
- Entrelazamiento vs. correlacion clasica perfecta
- Compuerta Toffoli (CCNOT)
- Interferencia destructiva de amplitudes como mecanismo computacional cuantico

**Herramientas/Frameworks mencionados:** Qiskit (mencionado para programacion; la sesion de codigo queda como ejercicio entre sesiones). Las diapositivas utilizan matematica pura con notacion de Dirac.

---

## 2. Sinopsis

La sesion parte de la recapitulacion del formalismo QBronze: el estado de $n$ qubits como $|\psi\rangle = \sum_{i=0}^{2^n-1} \alpha_i |i\rangle$ con amplitudes reales normalizadas, la evolucion unitaria $U|\psi\rangle = |\phi\rangle$ donde $U^T U = I$, y la regla de medicion $\Pr[i] = \alpha_i^2$. Sobre esa base, la sesion construye una perspectiva geometrica de las compuertas de un qubit: la compuerta de rotacion $U(\theta)$, la compuerta $X$ como reflexion respecto a la recta $y = x$, la compuerta $Z$ como reflexion respecto al eje horizontal, y la compuerta Hadamard como reflexion respecto al eje que forma un angulo de $\pi/8$ (22.5 grados) con el eje horizontal. Esta interpretacion geometrica no se generaliza de forma igualmente elegante a dos o mas qubits.

La sesion avanza hacia la preparacion del estado de Bell $|\Phi^+\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$ mediante el circuito canonico $H \otimes I$ seguido de CNOT, y luego introduce la base de Bell completa $\{|\Phi^\pm\rangle, |\Psi^\pm\rangle\}$. Un largo pasaje se dedica a desmitificar la descripcion popular del entrelazamiento: se argumenta con rigor que la correlacion perfecta que exhibe el estado de Bell puede reproducirse exactamente mediante un vector de probabilidad clasico ($\hat{V} = \frac{1}{2}(1,0,0,1)^T$), y que la comunicacion instantanea no es un rasgo exclusivo del entrelazamiento cuantico. La distincion genuinamente cuantica reside en las correlaciones no locales (violaciones de desigualdades de Bell), que no se generan en la sesion pero se mencionan como motivacion futura.

La sesion concluye con el bloque mas tecnico: la compuerta Toffoli (CCNOT) sobre tres bits y una evaluacion paso a paso de un circuito de 3 qubits ($H$, $CCNOT$, $H$) cuyos qubits 2 y 3 se inicializan en estados genericos $\alpha|0\rangle + \beta|1\rangle$ y $\gamma|0\rangle + \delta|1\rangle$. El objetivo central es exhibir la cancelacion destructiva de amplitudes como el mecanismo fundamental que distingue la computacion cuantica de la clasica.

---

## 3. Desglose por Subtemas

### 3.1 Geometria de compuertas de un qubit en el circulo unitario

El espacio de estados de un qubit real puede representarse como la mitad superior del circulo unitario, donde cada punto en la circunferencia corresponde a un estado normalizado $\cos\theta|0\rangle + \sin\theta|1\rangle$. Esta visualizacion permite interpretar cada compuerta unitaria de un qubit como una isometria del plano: una rotacion o una reflexion.

La compuerta de rotacion parametrizada por $\theta$:

$$
U(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}
$$

toma $|0\rangle$ al estado $\cos\theta|0\rangle + \sin\theta|1\rangle$, girando el vector de estado un angulo $\theta$ en sentido antihorario. Su reversibilidad es inmediata: rotar por $-\theta$ invierte la operacion, confirmando que $U^T U = I$.

La compuerta $X = \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix}$ es geometricamente una reflexion respecto a la recta $y = x$, lo que intercambia $|0\rangle \leftrightarrow |1\rangle$. Por linealidad, $X(\cos\theta|0\rangle + \sin\theta|1\rangle) = \sin\theta|0\rangle + \cos\theta|1\rangle$. Esta compuerta es clasica y cuantica simultaneamente: actua identicamente sobre vectores de probabilidad clasicos y sobre estados cuanticos, siendo el NOT cuantico.

La compuerta $Z = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix}$ es una reflexion respecto al eje horizontal ($x$). Deja $|0\rangle$ invariante y envia $|1\rangle \to -|1\rangle$. Este signo negativo no tiene analogo clasico (las probabilidades no admiten valores negativos), por lo que $Z$ es una compuerta exclusivamente cuantica.

La compuerta Hadamard $H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}$ es una reflexion respecto al eje que forma $\pi/8$ (22.5 grados) con el eje $x$. Sus efectos sobre la base estandar son $H|0\rangle = |+\rangle$ y $H|1\rangle = |-\rangle$, donde $|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$ y $|-\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle)$ forman la base de Hadamard (o base de signos). Los estados $|+\rangle$ y $|-\rangle$ son ortogonales y residen en cuadrantes opuestos del circulo, separados por 90 grados. Una propiedad crucial es que $H$ es autoinversa: $H^2 = I$, lo que significa que aplicar $H$ dos veces consecutivas sin ninguna operacion intermedia cancela el efecto neto.

Esta representacion geometrica solo es completa para un qubit. Para dos o mas qubits no existe una visualizacion igualmente elegante y exhaustiva; la esfera de Bloch proporciona la generalizacion correcta para un qubit en el caso complejo, pero no tiene un analogo directo para sistemas multipartitos.

---

### 3.2 Preparacion del estado de Bell y la base de Bell

La sesion introduce el circuito canonico de entrelazamiento sobre dos qubits inicializados en $|00\rangle$. El circuito consta de dos pasos temporales.

En $t_1$ se aplica $H \otimes I$, obteniendo:

$$
(H|0\rangle) \otimes |0\rangle = |+\rangle \otimes |0\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |10\rangle)
$$

Este estado es separable: se puede escribir explicitamente como $|+\rangle \otimes |0\rangle$, lo que demuestra que la superposicion por si sola no implica entrelazamiento. En este punto el primer qubit esta en superposicion y el segundo permanece en $|0\rangle$, sin correlacion cuantica entre ellos.

En $t_2$ se aplica CNOT con el primer qubit como control y el segundo como objetivo. La CNOT actua componente a componente: cuando el control es $|0\rangle$, el objetivo no cambia; cuando el control es $|1\rangle$, el objetivo se invierte. Por lo tanto:

$$
CNOT \cdot \frac{1}{\sqrt{2}}(|00\rangle + |10\rangle) = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) = |\Phi^+\rangle
$$

El resultado es el estado de Bell $|\Phi^+\rangle$, representado como el vector $\frac{1}{\sqrt{2}}(1,0,0,1)^T$ en la base computacional $\{|00\rangle, |01\rangle, |10\rangle, |11\rangle\}$. Este vector no puede escribirse como producto tensorial de dos estados individuales, lo que define formalmente el entrelazamiento: el estado es no separable.

La sesion define la base de Bell completa, formada por los cuatro estados maximalmente entrelazados de dos qubits:

$$
|\Phi^\pm\rangle = \frac{1}{\sqrt{2}}(|00\rangle \pm |11\rangle), \qquad |\Psi^\pm\rangle = \frac{1}{\sqrt{2}}(|01\rangle \pm |10\rangle)
$$

Estos cuatro estados forman una base ortonormal para el espacio de Hilbert de dos qubits y representan la maxima cantidad de entrelazamiento posible. El estado de Bell es la unidad de recurso de entrelazamiento en protocolos de informacion cuantica, a veces denominado ebit. Se senala que los estados de Bell son mas relevantes en informacion cuantica (teleportacion, distribucion de clave cuantica) que en computacion cuantica pura, donde el entrelazamiento se genera y explota de maneras mas generales.

---

### 3.3 Entrelazamiento cuantico vs. correlacion clasica perfecta: una distincion critica

Este es uno de los bloques conceptuales mas elaborados de la sesion y responde a una confusion muy extendida en la divulgacion cientifica. La pregunta central es: ¿que tiene de unico el entrelazamiento cuantico respecto a la correlacion clasica?

El argumento tipico en divulgacion dice que si Alice y Bob comparten el estado $|\Phi^+\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$, y Alice mide su qubit obteniendo $|0\rangle$, ella "instantaneamente" sabe que Bob tiene $|0\rangle$, sin importar la distancia. Esto se presenta como evidencia de comunicacion instantanea o de algo mistico en el entrelazamiento.

La sesion refuta este argumento mediante una analogia clasica exactamente equivalente. Supongase que un preparador lanza una moneda equilibrada: si sale cara, coloca un papel rojo en dos sobres; si sale cruz, coloca un papel verde en ambos. Alice y Bob toman un sobre cada uno y van a planetas distintos sin abrir el sobre. Cuando Alice abre el suyo y ve el papel rojo, "instantaneamente" sabe que Bob tiene papel rojo. Este experimento se describe matematicamente con el vector de probabilidad clasico:

$$
\hat{V} = \frac{1}{2}\begin{pmatrix}1\\0\\0\\1\end{pmatrix}
$$

No ocurre ninguna comunicacion. La "certeza instantanea" es simplemente la realizacion de una correlacion perfecta generada en el momento de la preparacion. Este mismo argumento, aplicado al estado de Bell, tampoco implica comunicacion cuantica: la correlacion es real, pero no es exclusivamente cuantica.

La distincion genuinamente cuantica se enuncia con precision: el entrelazamiento cuantico puede generar correlaciones no locales, es decir, distribuciones de probabilidad conjuntas sobre los resultados de Alice y Bob que no pueden reproducirse mediante ningun protocolo clasico que no incluya comunicacion entre las partes. Estas correlaciones no locales violan las desigualdades de Bell y son la firma matematicamente rigurosa de la unicidad cuantica. La sesion menciona explicitamente este punto pero aclara que el desarrollo matematico de las desigualdades de Bell queda fuera del alcance de la sesion presente.

Se senala tambien una distincion mas sutil en el nivel ontologico: en el caso clasico, el color del papel esta determinado desde el momento en que se sella el sobre; la incertidumbre de Alice es epistemica (ausencia de informacion). En el caso cuantico, segun las interpretaciones estandar de la mecanica cuantica, el resultado no esta determinado hasta el acto de la medicion; la superposicion es genuina, no meramente epistemica. Esta distincion, aunque no afecta directamente a los calculos de la sesion, es relevante para entender por que el entrelazamiento cuantico puede usarse en criptografia para garantizar seguridad incondicional.

---

### 3.4 Compuerta Toffoli y evaluacion del circuito de 3 qubits: interferencia de amplitudes

La compuerta Toffoli (CCNOT) generaliza la CNOT a tres qubits: dos qubits de control ($x$, $y$) y un qubit objetivo ($z$). El objetivo se invierte si y solo si ambos controles estan en $|1\rangle$; en todos los demas casos los tres qubits permanecen inalterados. Su tabla de verdad es:

| Entrada $ \|xyz\rangle $ | Salida |
|---|---|
| $\|000\rangle$, $\|001\rangle$, $\|010\rangle$, $\|011\rangle$, $\|100\rangle$, $\|101\rangle$ | Sin cambio |
| $\|110\rangle$ | $\|111\rangle$ |
| $\|111\rangle$ | $\|110\rangle$ |

Al igual que CNOT, la compuerta Toffoli es valida tanto para informacion clasica como cuantica, ya que es reversible. Es la base de la computacion reversible universal y aparece como primitiva en numerosos algoritmos cuanticos.

El ejercicio central de la sesion evalua el siguiente circuito de 3 qubits:

- Estado inicial: $|0\rangle \otimes (\alpha|0\rangle + \beta|1\rangle) \otimes (\gamma|0\rangle + \delta|1\rangle)$
- $t_1$: se aplica $H$ al primer qubit $\Rightarrow$ $|+\rangle(\alpha|0\rangle+\beta|1\rangle)(\gamma|0\rangle+\delta|1\rangle)$
- $t_2$: se aplica $CCNOT$ con los qubits 1 y 2 como controles y el qubit 3 como objetivo
- $t_3$: se aplica nuevamente $H$ al primer qubit
- Medicion: se mide el primer qubit

La motivacion del instructor es pedagogicamente astuta: su intuicion inicial de que los dos operadores Hadamard se cancelarian (dado que $H^2 = I$) resulta incorrecta, precisamente porque la CCNOT no actua de forma transparente sobre el primer qubit cuando los qubits 2 y 3 estan en superposicion. La CCNOT entrelaza los tres sistemas, y ese entrelazamiento impide que los Hadamard se cancelen.

La evaluacion en $t_2$ produce la siguiente transformacion: todas las componentes donde los qubits 1 y 2 no estan simultaneamente en $|11\rangle$ permanecen intactas; el unico termino que cambia es $\beta|11\rangle(\gamma|0\rangle+\delta|1\rangle) \to \beta|11\rangle(\gamma|1\rangle+\delta|0\rangle)$, invirtiendo el tercer qubit.

Tras aplicar $H$ al primer qubit en $t_3$ y expandir todos los terminos, recogiendo por estado base, el estado final toma la forma (con un factor global de $\frac{1}{2}$):

$$
\frac{1}{2}\Big[2\alpha\gamma|000\rangle + 2\alpha\delta|001\rangle + \beta(\gamma+\delta)|010\rangle + \beta(\gamma+\delta)|011\rangle
$$
$$
+ \beta(\gamma-\delta)|110\rangle + \beta(\delta-\gamma)|111\rangle\Big]
$$

Los estados $|100\rangle$ y $|101\rangle$ tienen amplitud cero. La razon es la cancelacion destructiva: la expansion del termino $\alpha|10\rangle$ procedente de uno de los Hadamard produce un $+\alpha|100\rangle$ y un $-\alpha|100\rangle$ que se anulan mutuamente. Esta cancelacion es imposible en la aritmetica clasica de probabilidades, donde todas las entradas son no negativas. Es el mecanismo de interferencia destructiva, analogo a la interferencia de ondas en fisica clasica, operando sobre amplitudes de probabilidad.

La probabilidad de medir el primer qubit en $|1\rangle$ resulta ser:

$$
\eta = \frac{1}{2}\beta^2(\delta-\gamma)^2
$$

Condicionado a que el primer qubit mida $|1\rangle$, los unicos estados compatibles son $|110\rangle$ y $|111\rangle$, ambos con el segundo qubit en $|1\rangle$. Por tanto, el segundo qubit se encuentra en $|1\rangle$ con certeza absoluta en ese caso, demostrando el efecto del entrelazamiento sobre las probabilidades condicionales.

Para maximizar $\eta = \frac{1}{2}$ se puede elegir $\alpha=0$, $\beta=1$, $\delta=\frac{1}{\sqrt{2}}$, $\gamma=-\frac{1}{\sqrt{2}}$ (el tercer qubit inicializado en el estado $|-\rangle$), lo que hace que $(\delta-\gamma)^2 = (\sqrt{2})^2 = 2$ y por tanto $\eta = \frac{1}{2} \cdot 1 \cdot 2 = 1$.

---

## 4. Puntos de Confusion y Casos de Esquina

El primero y mas importante es la confusion entre superposicion y entrelazamiento. Tras aplicar $H \otimes I$ al estado $|00\rangle$ se obtiene un estado en superposicion, pero dicho estado es separable: $|+\rangle \otimes |0\rangle$. La superposicion no implica entrelazamiento. El entrelazamiento aparece cuando el estado conjunto no puede escribirse como producto tensorial de estados individuales, lo que ocurre solo tras la CNOT.

El segundo es la cancelacion de amplitudes que el instructor mismo confundio en tiempo real. Cuando la CCNOT entrelaza los tres qubits y luego se aplica un segundo Hadamard, la intuicion de que "$H^2 = I$ cancela los dos Hadamard" es erronea porque la CCNOT modifica la dependencia del primer qubit con los otros dos. El signo negativo introducido por $H|1\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle)$ genera amplitudes negativas que se cancelan con amplitudes positivas para los estados $|100\rangle$ y $|101\rangle$. En un circuito clasico equivalente no existe este mecanismo porque las probabilidades son siempre no negativas.

El tercero es la interpretacion de la correlacion del estado de Bell como comunicacion instantanea. Como se desarrolla extensamente en la sesion, esta correlacion es exactamente reproducible por un vector de probabilidad clasico. La unicidad del entrelazamiento no esta en la correlacion de resultados sino en la posibilidad de generar correlaciones no locales (violaciones de Bell) que requieren entrelazamiento genuino.

Un cuarto punto de atencion es la normalizacion. La distincion entre $\hat{V} = \frac{1}{2}(1,0,0,1)^T$ (vector de probabilidad clasico valido) y $|\Phi^+\rangle = \frac{1}{\sqrt{2}}(1,0,0,1)^T$ (estado cuantico valido) no es trivial: la condicion de normalizacion para estados cuanticos es $\sum_i \alpha_i^2 = 1$, por lo que el factor $\frac{1}{2}$ del vector clasico no satisface la condicion cuantica. Proporcionar un vector de probabilidad clasico como entrada a un computador cuantico no tiene sentido operacional.

Finalmente, la convension de inicializacion: salvo indicacion contraria, todos los qubits de un circuito cuantico se inicializan a $|0\rangle$. Esta es una convencion estandar de la comunidad, no un requisito fisico absoluto, y traza su origen a las condiciones de DiVincenzo para la computacion cuantica escalable.

---

## 5. Para Estudiar

1. La compuerta $Z$ actua sobre la base computacional como $Z|0\rangle = |0\rangle$ y $Z|1\rangle = -|1\rangle$. Describe su efecto geometrico en el circulo unitario e indica por que no tiene analogo como compuerta clasica.

2. Dado el circuito $H$-CNOT aplicado a $|00\rangle$, ¿en que momento exacto se genera entrelazamiento? Justifica tu respuesta demostrando que el estado tras $H \otimes I$ es separable y el estado tras CNOT no lo es.

3. Enuncia la diferencia entre la correlacion perfecta clasica representada por $\hat{V} = \frac{1}{2}(1,0,0,1)^T$ y el estado de Bell $|\Phi^+\rangle = \frac{1}{\sqrt{2}}(|00\rangle+|11\rangle)$. ¿En que experimento fisico se manifestaria una diferencia genuina?

4. En el circuito de 3 qubits ($H$, $CCNOT$, $H$) con el primer qubit inicializado en $|0\rangle$ y los otros dos en estados genericos, la probabilidad de medir el primer qubit en $|1\rangle$ es $\eta = \frac{1}{2}\beta^2(\delta-\gamma)^2$. ¿Que eleccion de $\alpha, \beta, \gamma, \delta$ (respetando la normalizacion de cada qubit) maximiza $\eta$ y cual es su valor maximo?

5. Explica por que la interferencia destructiva de amplitudes en un circuito cuantico no tiene analogo en un circuito clasico probabilistico, y describe en terminos intuitivos como este fenomeno puede ser explotado para suprimir resultados indeseados en un algoritmo cuantico.