# Clase 3 – Inferencia de grafos

## Objetivo

Inferir un grafo a partir de datos (señales observadas en los nodos). Se busca el grafo que **mejor explica la señal** — ¿cómo usar GSP (Graph Signal Processing) para inferir la topología del grafo?

## Los 3 problemas de inferencia de grafos

La pregunta general en todos los casos: ¿hay o no hay una arista entre dos nodos? ¿con qué peso?

1. **Predicción de enlace** (presentado clase anterior): se conoce parte del grafo y se quieren inferir las aristas que faltan.
2. **Association network inference** (tema de hoy): se tienen señales en los nodos pero no se conoce ninguna arista — se quiere inferir el grafo completo.
3. **Tomographic network topology inference**: no se conoce ninguna arista y tampoco se tiene información (de señales).

## Association Networks

**Definición:** los vértices quedan unidos por una arista si hay un nivel suficiente de "asociación" entre los atributos de cada par de vértices.

**Datos:** matriz donde las filas representan los nodos y las columnas la señal $x$ a lo largo del tiempo.

**Preguntas centrales:**

- ¿El grafo inferido es único? → depende de las condiciones.
- ¿Cuántas mediciones hay que hacer del experimento? Se quiere estimar las aristas dado un número de mediciones. Cuanto más larga sea la señal, mejor la estimación.

Dados los nodos y la señal, se define una **función de similitud** para medir cuánto se parecen dos señales. Elegir esta función es una de las decisiones centrales del método.

### Redes de correlación

Idea: si dos nodos están correlacionados, sus señales son similares.

- Se calcula la **correlación** entre cada par de señales temporales.
- **Grafo de correlación:** se pone una arista entre dos nodos si la correlación es distinta de cero.
- Association network inference $\iff$ inferencia de correlaciones no nulas.

**Test de hipótesis:** $H_0$ se define como lo que se quiere que pase. Habría que hacer $N^2$ tests de hipótesis, pero por simetría alcanza con $N(N-1)/2$ — de todas formas resulta costoso computacionalmente.

**Estadísticos usados para el test (cómo se hace la cuenta):**

- Correlaciones empíricas.
- **Transformada de Fisher:** lleva la correlación empírica a una variable con distribución (aproximadamente) normal, lo que simplifica controlar la significancia del test.

**Testeo múltiple:** al hacer muchos tests de hipótesis a la vez (uno por par de nodos) aparece el problema de testeo múltiple en grafos. Se corrige con **False Discovery Rate (FDR)**.

### Correlaciones parciales

**Motivación:** correlación no implica causalidad — un par de nodos puede estar correlacionado porque ambos están influenciados por un tercer nodo, no porque estén conectados directamente entre sí. (Para estos problemas los grafos no tienen self-loops.)

Las **correlaciones parciales** capturan mejor la influencia directa entre nodos, descontando el efecto de terceros nodos.

**Redes de correlaciones parciales:** se construyen igual que las redes de correlación, pero usando correlación parcial en vez de correlación simple.

## Undirected Gaussian Graphical Models

*(mencionado como título en el apunte, sin desarrollo — ver puntos a revisar)*

### Digresión: sparsity y norma $\ell_1$

- La norma $\ell_1$ induce **esparsidad** (estimador **Lasso**): una ventaja de esta norma es que hace que algunas coordenadas se anulen exactamente.
- **Graphical Lasso:** estima toda la matriz (de correlación parcial) en conjunto.

### Covariance selection meets linear regression

Objetivo: mejorar la explicación del método / chequear el algoritmo, estimando la matriz por partes usando combinaciones lineales y viendo qué nodos aportan más.

- Método: se toma la primera fila, se hace una regresión, y se conservan los pocos coeficientes más significativos.
- Preguntas abiertas anotadas en clase: ¿problemas del método? ¿ventajas? *(sin responder en el apunte — ver puntos a revisar)*

**Neighborhood-based sparse regression:** variante en la que las regresiones se hacen de forma independiente (en vez de conjunta).

> Estos modelos (Gaussian graphical models, graphical Lasso, covariance selection) son para **variables aleatorias Gaussianas**.

## Aprendiendo grafos a partir de observaciones de señales suaves

Recordando el ejemplo de la primera clase: a medida que avanza la señal, es menos suave. Frecuencia grande → coeficiente chico.

Ahora se infiere el grafo a partir de la **matriz Laplaciana**: se aprende un grafo a partir de una señal suave.

**Formulación y algoritmo:** hay que definir cuánta importancia se le da a la suavidad y cuánta a la esparsidad.

**Signal smoothness meets edge sparsity:** mencionado como alternativa a usar directamente la Laplaciana *(sin más detalle en el apunte)*.

## ⚠️ Puntos poco claros / a revisar

- "$H_0$ se define como lo que se quiere que pase" — la frase tal cual anotada no queda clara; revisar con las diapositivas cómo se elige exactamente la hipótesis nula en este contexto.
- "Undirected Gaussian Graphical Models" aparece solo como título en el apunte, sin contenido desarrollado.
- Las preguntas planteadas en clase sobre "Covariance selection meets linear regression" (¿problemas del método?, ¿ventajas?) quedaron sin responder en el apunte.
- La referencia a "Diapo 41" (ejemplo de la primera clase, señal cada vez menos suave) no queda del todo clara sin ver la diapositiva.
- "Signal smoothness meets edge sparsity" se menciona como alternativa a usar la Laplaciana, pero sin ningún detalle de en qué consiste.
- No queda claro cómo se relacionan "Covariance selection meets linear regression" / "Neighborhood-based sparse regression" con el Graphical Lasso mencionado antes (¿son métodos alternativos para el mismo problema, o pasos distintos?).
- El tercer problema de inferencia, "Tomographic network topology inference", queda solo enunciado (sin aristas conocidas ni información), sin desarrollo posterior en este apunte.
