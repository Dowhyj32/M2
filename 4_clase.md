# Clase 4 – Graph Neural Networks (GNNs)

## Intro

- Algunos ejemplos de motivación 🔹 *(de diapositivas)*:
  - **Clasificación de nodos:** ¿a qué área pertenece un nuevo paper de arXiv? Se usa el grafo de citas (papers ya categorizados citados por el nuevo).
  - **Predicción de aristas / datos faltantes:** estimar el canal (RSSI) entre un dispositivo y varios *access points* a partir de mediciones parciales — equivale a "completar" enlaces faltantes en el grafo.
  - **Clasificación de grafos:** localización indoor de un usuario usando únicamente el grafo de mediciones RSSI a distintos APs.

## Motivación: Empirical Risk Minimization (ERM)

- Tenemos: conjunto de entrenamiento, una clase de funciones (acá es donde entran las GNN), y una función de pérdida (mide la bondad de ajuste).
- Buscamos, dentro de la familia de funciones, la que minimiza la función de pérdida.
- ¿Qué es un dato en este contexto?
  - Un grafo
  - Señal en los nodos
  - Atributos en las aristas

### Representando datos en grafos

- Se toma una decisión: ponerle un orden a los nodos, y por ende también a la señal (es arbitrario).
- Se busca que la función **no dependa del orden** elegido, y que sea aplicable a grafos de distinto tamaño (poder operar sobre grafos distintos con la misma función).

## Convoluciones en grafos

- Referencia: Convolutional Neural Networks (CNNs).
- **CNN:** toma una matriz y genera otra usando la convolución.
- En señales en tiempo discreto, la cercanía (entre instantes) contiene información.

![alt text](image.png)

- Hace falta un buen orden para que la sucesión sea suave. Como es una serie de tiempo, el grafo que la representa es una línea.
- Se fija el orden de los nodos → permutaciones → queda definida la matriz $S$ genérica (**Graph Shift Operator**).
- **Graph convolution:** combinación lineal de versiones desplazadas de la señal (el grafo es el mismo, solo se desplaza la señal hacia la derecha de a un paso; esto se logra multiplicando por un operador).

![alt text](image-1.png)

- $K$ se elige (expande la "máscara" del filtro).
- 🔹 *(de diapositivas)* La cadena **"Graph Convolution ⇒ Graph Signal Processing ⇒ Graph Filtering"** no expresa una relación causal entre tres conceptos distintos: son tres nombres / ángulos de la misma idea (análogo a "Convolution ⇒ Signal Processing ⇒ Filtering" en el procesamiento de señales clásico).

### De información local a global

- Se puede operar el mismo filtro en distintos grafos.

![alt text](image-2.png)

- En principio, la familia de funciones definida así cumple con lo que se buscaba (independencia del orden, aplicable a grafos de distinto tamaño).

> Hasta acá se vieron filtros. Sigue el aprendizaje.

## GNN: Construcción

- Esquema: Señal → Filtro → Nueva señal.
- Aprendiendo usando un **Graph Perceptron**.
- La salida da un parámetro $H$ (tensor) — se va a querer elegir la cantidad de $h$'s.
- Un graph filter puro tiene expresividad limitada: solo puede aprender funciones **lineales**. El **Graph Perceptron** agrega una no linealidad puntual $\sigma$ a la salida del filtro; apilando varios perceptrones (capas) se aumenta aún más la expresividad (eso es la GNN).
- La única diferencia entre una GNN y un graph filter es esa no linealidad punto a punto — pero en la práctica las GNN funcionan mucho mejor (la explicación formal de por qué viene más adelante, con el análisis de estabilidad ante perturbaciones).
- 🔹 *(de diapositivas)* **Ventajas de la GNN** (resuelve el punto pendiente):
  - **Transferencia entre grafos:** el GSO $S$ se puede reinterpretar como una *entrada* (no como un parámetro fijo) → una GNN ya entrenada (el tensor $H^\star$) se puede aplicar a otro grafo sin reentrenar.
  - **Menos parámetros, generaliza mejor:** frente a una red "plana" que viera toda la matriz de adyacencia como una entrada más, la GNN comparte los mismos parámetros $H$ en todos los nodos (no dependen de $N$) y aprovecha la estructura local del grafo.
  - **Aplica a grafos de distinto tamaño**, a diferencia de una red neuronal genérica atada a un $N$ fijo.

## Múltiples features

*(visto muy rápido en el apunte)*

- En las librerías se habla poco de "convolución"; se habla más de **message passing**.
- Ambos "son lo mismo".
- 🔹 *(de diapositivas)* Formalización: una señal con múltiples features es una matriz $X$ de tamaño $N$ (nodos) × $F$ (features) — cada fila son los features de un nodo, cada columna es una graph signal individual. La convolución se generaliza a
$$Y = \sum_{k} S^k X H_k$$
  donde multiplicar por $S$ desplaza cada feature (columna) por separado, y multiplicar por $H$ combina los features **dentro de cada nodo** (no mezcla nodos entre sí).
- 🔹 *(de diapositivas)* Hiperparámetros de una GNN multi-feature (afectan la cantidad de parámetros a aprender):
  - $L$: número de capas.
  - $K_\ell$: orden del filtro en la capa $\ell$.
  - $F_\ell$: cantidad de features en la capa $\ell$.

### Message Passing 🔹 *(de diapositivas)*

- Las librerías (ej. PyTorch Geometric) no hablan de "convolución" sino de **message passing** — es el mismo concepto, con $K_\ell = 1$ por capa: cada capa agrega información solo de los vecinos inmediatos, y el alcance mayor se logra apilando varias capas.
- Varias arquitecturas conocidas son casos particulares de este mismo framework, con distintas elecciones de $S$ y $H$: Spectral GCNNs, ChebNets, Diffusion CNNs, entre otras.

## Entrenando una GNN

- Una vez construida la GNN, se quiere llegar a un vector que represente algo.
- Aprender ⟺ minimizar la pérdida.
- 🔹 *(de diapositivas)* Ejemplos de función de pérdida: **L2** para regresión, **cross-entropy** para clasificación.

### Predicción a nivel de nodo

- Ejemplo: decidir si un usuario de una red social es un bot.
- 🔹 *(de diapositivas, resuelve punto pendiente)* Durante el **entrenamiento** solo participan los nodos del training set (se usa una versión recortada del $S$); al **predecir**, se usa el $S$ completo (todos los nodos).

### Predicción de aristas 🔹 *(de diapositivas)*

- Se usa la salida de la GNN en los **dos nodos extremos** de una arista para estimar si esa arista existe (o cuánto vale su peso).
- Ejemplos: recomendación de contenido a usuarios, completar datos faltantes en una base de datos relacional, estimar la atenuación en una red inalámbrica.

### Clasificar grafos

- Si la identidad de los nodos **no** importa: 🔹 *(de diapositivas)* se puede resumir/agregar el grafo de forma simétrica (agregación permutation-invariant) — es el caso donde una GNN estándar rinde bien.
- Si la identidad de los nodos **sí** importa, una GNN "no es lo mejor": 🔹 *(de diapositivas)* alternativas son **Edge-Variant GNNs**, **Feature Augmentation** y **Attention**.

## Permutation equivariance

- 🔹 *(de diapositivas)* Una GNN es equivariante a permutaciones si, al reordenar los nodos del grafo (y su señal), la salida se reordena exactamente igual, sin cambiar de valores — es el análogo, en grafos, de la invarianza a traslaciones de las CNN.
- 🔹 Por qué es valiosa: permite aprovechar simetrías internas del grafo y que la GNN generalice entre distintos órdenes/etiquetados de nodos sin reentrenar.
- 🔹 No siempre es deseable: si la identidad/posición de un nodo debería importar para la tarea, forzar la equivariance es una limitación (mismo caso que en "Clasificar grafos" arriba — alternativas: Edge-Variant GNNs, Feature Augmentation, Attention).

## Perturbaciones

- Fourier *(mencionado, sin desarrollo completo en el apunte)*
- 🔹 *(de diapositivas)* Motivación: a diferencia de las convoluciones temporales clásicas, no hay una intuición directa sobre la estabilidad de las graph convolutions frente a cambios en el grafo. Para estudiar eso se recurre a la **Graph Fourier Transform**: permite ver qué le pasa a un filtro cuando el grafo se perturba levemente.
- *(El desarrollo completo — transformada, respuesta en frecuencia, por qué las no linealidades ayudan a la estabilidad — sigue más adelante en las diapositivas; probablemente continúe en la próxima clase.)*

---

## 🎯 Posibles puntos de examen

- Diferencia entre un **graph filter** y un **Graph Perceptron** (la no linealidad punto a punto) y por qué la segunda es más expresiva.
- Qué es el **Graph Shift Operator** $S$ y qué rol cumple (matriz que codifica al grafo, ej. adyacencia).
- **Message passing ≡ graph convolution con $K=1$** por capa; el alcance mayor se logra apilando capas.
- Por qué durante el **entrenamiento** de una GNN para predicción a nivel de nodo solo participan los nodos del training set (S recortada), mientras que en **inferencia** se usa el grafo completo.
- Los **tres problemas canónicos** en GNN (nodo / arista / grafo) y qué salida de la red se usa en cada caso.
- **Ventajas de una GNN** frente a una red neuronal genérica: comparte parámetros (no dependen de $N$), es transferible entre grafos, aplica a grafos de tamaño variable.
- Definición de **permutation equivariance** y su analogía con la invarianza a traslaciones de las CNN.
- Cuándo la permutation equivariance **no** es una buena idea, y qué alternativas existen (Edge-Variant GNNs, Feature Augmentation, Attention).
- Motivo para introducir la **Graph Fourier Transform**: estudiar la estabilidad de un filtro ante perturbaciones del grafo.

## ⚠️ Puntos poco claros / a revisar

- **Resueltos con las diapositivas:** la relación "Graph Convolution ⇒ GSP ⇒ Graph Filtering" (son sinónimos, no una cadena causal); la "Ventaja GNN" que había quedado sin completar; la duda de si se ocultan o no las etiquetas al entrenar (se recorta el grafo a los nodos de training, no se ocultan etiquetas dentro del mismo grafo); el contenido de "Predicción de aristas"; la frase incompleta de "Clasificar grafos".
- **"Sección 11, algoritmo":** corresponde a la sección "Expresividad" del deck (test de Weisfeiler-Lehman para la capacidad de una GNN de distinguir nodos/grafos). Queda bastante más adelante en las diapositivas (después de Perturbaciones y Wi-Fi Indoor Positioning) — no parece ser parte de esta clase, sino de una siguiente.
- **Permutation equivariance y Perturbaciones:** se agregó la idea conceptual base, pero el desarrollo matemático completo (demostración de equivariancia, transformada de Fourier en grafos, análisis de estabilidad) continúa en las diapositivas más allá de donde llegó esta clase — probablemente se retome en la clase siguiente.
