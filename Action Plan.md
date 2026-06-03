# Action Plan — QWorld Workshop

> *Documento de planificación activa para la preparación y el éxito en el taller de QWorld*

---

## Objetivos Principales

1. **Alineación con la Evaluación**: Asegurar que cada concepto clave evaluado en los laboratorios (Labs) y cuestionarios (Quizzes) esté documentado y comprendido en las notas personales.
2. **Dominio Teórico-Matemático**: Transicionar de la teoría de complejidad (Día 1) a la manipulación algebraica clásica (Día 2) y al formalismo cuántico (Día 3).
3. **Resolución Práctica**: Resolver el 100% de los cuestionarios (Para Estudiar) incluidos al final de cada lectura técnica.
4. **Organización del Workspace**: Mantener un repositorio limpio con convenciones de nomenclatura estrictas para notas y código.

---

## Hoja de Ruta Específica por Sesión

```mermaid
graph LR
    classDef default fill:#1a1a2e,stroke:#4e54c8,stroke-width:2px,color:#ffffff;
    
    D1[Dia 1: Complejidad y NISQ] --> D2[Dia 2: Algebra Lineal Clasica]
    D2 --> D3[Dia 3: Formalismo Cuantico]
```

### Día 1: Prospects of Quantum Computing (Fundamentos)
*   **Foco Teórico**: Tesis de Church-Turing, clases de complejidad (P, NP, BQP, NP-completo), era NISQ y dequantización.
*   **Acciones Concretas**:
    - [x] Analizar la diferencia operativa entre generar y verificar soluciones.
    - [x] Comprender la posición del algoritmo de Shor respecto a la clase P y BQP.
    - [x] Estudiar las limitaciones de confianza al simular sistemas fuera de NP en hardware NISQ.

### Día 2: Introduction to Classical Systems (Álgebra Lineal Clásica)
*   **Foco Teórico**: Vectores de probabilidad, combinaciones convexas, matrices estocásticas, producto tensorial, CNOT clásica.
*   **Acciones Concretas**:
    - [ ] Realizar demostraciones manuales de indexación y multiplicación de vectores y matrices estocásticas.
    - [ ] Resolver el ejercicio de la moneda sesgada con transición probabilística y determinista.
    - [ ] Demostrar analíticamente por qué un estado conjunto con correlaciones (ej. $v = [1/2, 0, 0, 1/2]^T$) no es separable ($u \otimes w$).
    - [ ] Construir la matriz de CNOT clásica $4 \times 4$ y demostrar su indescomponibilidad.

### Día 3: Introducción a Sistemas Cuánticos (Mecánica Cuántica del Qubit)
*   **Foco Teórico**: Amplitudes complejas, regla de Born, matrices unitarias, notación de Dirac (bra-ket), compuerta de Hadamard, interferencia, separabilidad y teorema de Gottesman-Knill.
*   **Acciones Concretas**:
    - [ ] Practicar la normalización de vectores con amplitudes reales/complejas e identificar probabilidades.
    - [ ] Demostrar analíticamente que $H$ es unitaria ($H^T H = I$) y autoinversa ($H^2 = I$).
    - [ ] Evaluar el comportamiento de interferencia comparando Hadamard doble con/sin medición intermedia.
    - [ ] Calcular productos internos y demostrar ortogonalidad de la base de Hadamard ($\langle + | - \rangle = 0$).

---

## Protocolo de Estudio Activo

Para cada lección del taller, seguir estos 4 pasos sistemáticos:

> **Paso 1: Pre-lectura y Glosario**  
> Leer la Ficha Técnica y la Sinopsis para identificar la terminología crítica. Crear un glosario de términos nuevos.

> **Paso 2: Derivación Matemática**  
> No memorizar fórmulas. Escribir en papel o notebook de notas las derivaciones de los productos tensoriales y las aplicaciones de compuertas paso a paso.

> **Paso 3: Autoevaluación Práctica**  
> Resolver sin ver apuntes las 5 preguntas de la sección Para Estudiar de cada día. Validar respuestas contra el material original.

> **Paso 4: Mitigación de Errores**  
> Repasar exhaustivamente la sección Puntos de Confusión de cada día (ej. evitar confundir la superposición cuántica con probabilidad clásica, o no determinismo con probabilismo).

---

## Estructura del Proyecto y Nomenclatura

Para mantener el orden del workshop, se establece el siguiente estándar de nombres de archivos:

*   **Notas Teóricas**: `Notes_Day[X]_[Short_Topic].md`
    *   *Ejemplo*: [Notes_Day2_Classical_Systems.md](Lectures/Day-1-Qworld.md)
*   **Notebooks de Laboratorio**: `Lab[X]_[Topic_Name].ipynb`
    *   *Ejemplo*: `Lab1_Quantum_States.ipynb`
*   **Cuestionarios / Prácticas**: `Quiz[X]_[Topic_Name].md`
    *   *Ejemplo*: `Quiz2_Stochastic_Matrices.md`