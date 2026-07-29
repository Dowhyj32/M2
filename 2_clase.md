# Clase 2 – Modelos Estadísticos de Grafos

> Apuntes tomados en clase, complementados con `clase_2.pdf`. Quedan varias preguntas abiertas (anotadas en clase) y un par de secciones incompletas, marcadas explícitamente más abajo para retomar con el material o en la próxima clase.

## Motivación

- ¿Para qué modelar estadísticamente los grafos?
- ¿Cómo explicamos las propiedades que observamos en grafos reales? → *small-world*, distribuciones de grado con ley de potencia
- ¿Fenómeno de colas pesadas?

Vamos a ver modelos, de los más sencillos a los más complejos.

---

## Modelo de Grafos Aleatorios (marco general)

- **G:** conjunto de grafos posibles
- **$P_\theta$:** distribución de probabilidad sobre $G$
- **$\theta$:** vector de parámetros del modelo

### Especificación del modelo

1. $P(\cdot)$ uniforme sobre $G$
2. Modelo generativo (*small world*, *preferential attachment*, *copying models*)
3. Usado en ciencias sociales o economía (no se ve mucho en el curso)
4. Modelar la tendencia de los nodos a conectarse mediante **variables latentes** (*stochastic block models*, *graphons*, *random dot product graphs*)

---

## Grafos Aleatorios Clásicos: Erdős–Rényi(-Gilbert)

Se asigna la misma probabilidad a todos los grafos no dirigidos.

- **ER(n, p)** — n: nodos; p: probabilidad
- Se fijan los nodos y se sortean las aristas: la arista $(u,v)$ existe con probabilidad $p$, independiente del resto.
- Número esperado de vecinos: $n \cdot p$

### Propiedades de ER(n, p)

- **Distribución de grado:** Binomial (cada arista es independiente)
- $E[D_i] = p(n-1)$
- > Pregunta abierta (anotada en clase): ¿Hoeffding? (probablemente relacionado con acotar la concentración del grado alrededor de su media)

**Transición de fase en la aparición de una componente gigante:** a medida que $n \cdot p$ crece, quedan menos componentes conexas, hasta llegar a un grafo conexo. Se aprecia un cambio de fase (ver gráfico):

- $n \cdot p > 1$ → componente gigante de tamaño $O(n)$
- $n \cdot p < 1$ → componentes de tamaño $O(\log n)$

**Algoritmo que demuestra esta propiedad:**

- Estados: activo ($A_t$), inactivo ($I_t$), explorado ($E_t$)
- Normalmente se busca que el grafo no quede disconexo
- Da lugar a una cadena de Markov en tiempo continuo
  - > Pregunta abierta (anotada en clase): ¿qué es exactamente / a qué se refiere con "tiempo continuo"?
- Se observa que la cadena converge a una función determinística más ruido (revisar cómo escalan los ejes del gráfico)
- Se llega a una ecuación diferencial con solución exacta, comparable con la simulación — esto sucede cuando la cantidad de vecinos promedio es mayor a 1

**Diámetro del grafo** (distancia máxima entre 2 nodos) y **clustering coefficient**: estos modelos clásicos no cumplen con ciertas propiedades observadas en la práctica.

> Pregunta abierta (anotada en clase): ¿por qué ER no reproduce el diámetro chico y el clustering alto que se observan en redes reales? (ver más abajo, modelo Small World, que parece apuntar a esta misma limitación)

---

## Configuration Models (modelos más realistas)

Generalizan el modelo ER.

> Nota: revisar las escalas de los gráficos y cómo decaen (relación con colas pesadas).

Interesa generar grafos con una distribución de grado que decaiga como ley de potencia — ER no genera ese tipo de gráficos, por eso se necesita el **configuration model**.

- **Idea:** se fija de antemano una secuencia de grados $d$.
- **Generación del grafo:**
  - **Matching Algorithm** (usado para el análisis teórico): dados los nodos con sus semi-aristas ("stubs"), se sortean al azar y se arma la secuencia de aristas.
    - > Pregunta abierta (anotada en clase): ¿cómo se garantiza que el grafo resultante sea válido (sin auto-loops ni aristas repetidas)? El profesor mencionó algo sobre la distribución resultante — repasar.
    - Hay ciertas restricciones para poder ejecutar este algoritmo.
  - **Switching Algorithm** *(pendiente — no quedó registrado el detalle, repasar en el PDF)*

**Algunos resultados:**

1. Transición de fase en la aparición de una componente gigante (igual que en ER)
2. El clustering coefficient tiende a 0
3. *(pendiente)*
4. *(pendiente)*

---

## Modelo Small World (Watts-Strogatz)

"Seis grados de separación."

¿Cuál es la distancia mínima entre 2 nodos en la red? → **Experimento de Milgram**: muestra que los caminos cortos existen, y en abundancia.

Las redes reales típicamente tienen diámetro chico **y** muchos triángulos (alto clustering) — algo que el modelo ER no logra combinar (diapo 28 y 29).

- **Modelo Watts-Strogatz:** combina una estructura regular (con triángulos) con reconexión aleatoria, para lograr diámetro chico. Controla la cantidad de aristas, aunque no es el método más eficiente.

---

## Modelo de Variables Latentes

Los nodos tienen una clase asignada.

- **Modelos de clases latentes** (discretas)
- **Modelos de vectores latentes** (continuos)

> Ejemplo: actores que comparten películas — las distintas comunidades de cine pensadas como variables latentes.

### Stochastic Block Models (SBMs)

Los nodos se separan en clases, y hay una matriz de probabilidades de conexión entre clases. Normalmente es más probable conectarse con alguien de la propia comunidad.

> Pregunta abierta (anotada en clase): ¿el SBM se usa para generar el grafo, para detectar comunidades, o para ambas cosas?

- *(pendiente: anotar las definiciones formales de diapo 36)*
- **Matriz Q** (diapo 37): según sus valores se puede observar el tipo de estructura — ¿heterofilia?

### Extensiones de SBMs

- **Degree-corrected SBMs**
- *(pendiente — quedaron sin anotar más extensiones, repasar diapositivas)*

---

## Random Dot Product Graphs (RDPGs)

- **Conectados:** vectores latentes alineados
- **No conectados:** vectores ortogonales, producto interno cero
- *(pendiente: conexión con otros modelos — no quedó registrado)*

**Estimación de las posiciones latentes:**

- Se calculan los autovalores y se toman los autovectores asociados a los autovalores más grandes (los $d$ más significativos) — diapo 47 (diagonalización de la matriz).
- ¿De dónde sale $d$? Si el modelo está bien especificado, debería observarse un "codo" en el gráfico de autovalores.
- Esto es el **Adjacency Spectral Embedding (ASE)**.

Diapo 50: notar la ortogonalidad de los embeddings — dos aspectos a considerar:

- Alineación del vector
- Magnitud del vector

**Aplicación:** *online change point detection* — se quiere detectar si, y en qué momento, cambia el modelo subyacente a lo largo del tiempo.

---

## Geometric Random Graphs

- Ventajas y desventajas *(pendiente: no quedó el detalle)*
- Para capturar mejor las propiedades deseadas: usar geometrías no euclídeas
- **Curvatura:** recordar la definición de circunferencia en este contexto (geometría no euclídea)
