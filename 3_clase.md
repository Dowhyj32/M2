# Clase 3 – Inferencia de grafos

> Apunte de clase, completado y verificado contra `inferencia.pdf` (56 diapositivas, Marcelo Fiori). Lo agregado a partir de las diapositivas está marcado con 🔹.

## Objetivo

Inferir un grafo a partir de datos (señales observadas en los nodos). Se busca el grafo que **mejor explica la señal** — ¿cómo usar GSP (Graph Signal Processing) para inferir la topología del grafo?

🔹 **Ejemplo motivador:** en *network neuroscience*, inferir la red funcional del cerebro a partir de señales de fMRI (qué regiones se comunican entre sí).

🔹 La mayoría de los trabajos de GSP asumen un grafo **conocido** y estudian cómo afecta a las señales (tiene sentido en redes físicas, donde los enlaces son observables). Acá se hace el **camino inverso**: usar GSP para inferir la topología. Motivo práctico: mantener actualizada la topología real de una red suele ser difícil (tamaño, reconfiguración, privacidad, seguridad).

🔹 **Objetivo formal:** recuperar una *red latente*, o una representación en grafo de los datos.

## Los 3 problemas de inferencia de grafos

La pregunta general en todos los casos: ¿hay o no hay una arista entre dos nodos? ¿con qué peso?

🔹 Formalización común (marco de [Kolaczyk'09]): dado un grafo que no se observa (o se observa solo parcialmente), se busca inferirlo a partir de mediciones de señal en los nodos y, eventualmente, el estado de algunas aristas ya conocidas. Formularlo así permite aprovechar herramientas estadísticas ya conocidas: identificabilidad, consistencia, robustez, complejidad computacional.

1. **Predicción de enlaces** (presentado clase anterior): se conoce la señal completa y el estado de las aristas solo para un subconjunto de pares de vértices; se quieren inferir las aristas que faltan (el resto de los pares).
2. **Association network inference** (tema de hoy): se tienen señales en los nodos pero no se conoce ninguna arista — se asume que la conexión entre dos nodos está definida por un nivel de asociación no trivial entre sus señales, y se quiere inferir el estado de arista para **todos** los pares.
3. **Tomographic network topology inference**: se observa la señal solo en algunos vértices ubicados en el "perímetro" del grafo (los accesibles); el objetivo es inferir vértices y aristas del "interior", la parte no observada. Es el caso más difícil de los tres.

## Association Networks

**Definición:** los vértices quedan unidos por una arista si hay un nivel suficiente de "asociación" entre los atributos de cada par de vértices.

🔹 Ejemplos de dominio: *gene-regulatory networks*, *neuro-functional connectivity networks*.

**Datos:** matriz donde las filas representan los nodos y las columnas la señal $x$ a lo largo del tiempo.

**Preguntas centrales:**

- ¿El grafo inferido es único? → depende de las condiciones.
- ¿Cuántas mediciones hay que hacer del experimento? Se quiere estimar las aristas dado un número de mediciones. Cuanto más larga sea la señal, mejor la estimación.

🔹 **Association network inference (definición formal del problema):** dada una señal de atributos observados en los nodos, y una similaridad definida por el usuario $\text{sim}(i,j)=f(x_i,x_j)$ que especifica las aristas, el problema es **inferir los valores no triviales de esa similaridad** a partir de observaciones i.i.d. — ya que el estado real de las aristas (los valores de sim) no es observable directamente.

🔹 Hay **tres categorías de decisiones** a tomar para resolver este problema:

| Decisión | Opciones típicas |
|---|---|
| Elección de sim | correlación, correlación parcial, información mutua |
| Elección de técnica de inferencia | test de hipótesis, regresión, ad hoc |
| Elección de parámetros | umbrales de test, nivel de significancia, regularización |

### Redes de correlación

Idea: si dos nodos están correlacionados, sus señales son similares.

- Se calcula la **correlación** (de Pearson) entre cada par de señales temporales.
- **Grafo de correlación:** se pone una arista entre dos nodos si la correlación es distinta de cero.
- Association network inference $\iff$ inferencia de correlaciones no nulas.

**Test de hipótesis:** $H_0: \rho_{ij}=0$ (no hay correlación) versus $H_1: \rho_{ij}\neq 0$ (hay correlación). Habría que hacer $N^2$ tests de hipótesis, pero por simetría alcanza con $N(N-1)/2$ — de todas formas resulta costoso computacionalmente.

> 🔹 **¿Qué es un test de hipótesis?** Es un procedimiento para decidir, a partir de los datos, entre dos hipótesis: la **hipótesis nula** $H_0$ (por convención, la situación "por defecto" o de ausencia de efecto — acá, "no hay correlación") y la **hipótesis alternativa** $H_1$ (hay efecto). Se calcula un estadístico a partir de los datos y se **rechaza** $H_0$ si ese estadístico resulta demasiado extremo como para ser plausible bajo $H_0$. El **nivel de significancia** $\alpha$ es la probabilidad que se tolera de rechazar $H_0$ por error (falso positivo) cuando en realidad era cierta.

**Estadísticos usados para el test (cómo se hace la cuenta):**

- Correlaciones empíricas.
- **Transformada de Fisher:** lleva la correlación empírica a una variable con distribución (aproximadamente) normal bajo $H_0$, lo que simplifica controlar la significancia del test. Se rechaza $H_0$ si el estadístico supera un umbral asociado a $\alpha$.

**Testeo múltiple:** al hacer muchos tests de hipótesis a la vez (uno por par de nodos) aparece el problema de testeo múltiple en grafos. 🔹 Ejemplo: con $N=100$ nodos y $\alpha=0{,}05$, aunque el grafo real fuera vacío se esperarían **~250 aristas espúreas** solo por azar (de los 4950 pares posibles) — para un grafo grande, el número de falsos positivos puede ser considerable.

Se corrige con **False Discovery Rate (FDR)**: en vez de controlar el error por test individual, se controla la fracción esperada de aristas falsas sobre el total de aristas declaradas. 🔹 Método práctico (Benjamini-Hochberg): se ordenan los $p$-valores de todos los tests, y se declaran arista todos los pares cuyo $p$-valor esté por debajo de un umbral que crece según su posición en el ranking (no un umbral fijo como en el testeo individual).

🔹 **Ejemplo aplicado (E. coli, genes *aroG*/*tyrR*/*lrp*):** *aroG* está regulado por *tyrR* pero no por *lrp* (ground truth conocido). Sin embargo, ambos muestran correlación alta y $p$-valor chico con *aroG* — un caso concreto de cómo la correlación simple puede dar **falsos positivos por factor de confusión**, motivando el uso de correlación parcial.

### Correlaciones parciales

**Motivación:** > 🔹 **"correlación no implica causalidad"** — un par de nodos puede estar correlacionado porque ambos están influenciados por un tercer nodo, no porque estén conectados directamente entre sí. (Para estos problemas los grafos no tienen self-loops.)

Las **correlaciones parciales** capturan mejor la influencia directa entre nodos, descontando el efecto de terceros nodos. 🔹 Se calculan a partir de los bloques de la matriz de covarianza (covarianza y varianzas condicionales, eliminando el efecto lineal de los nodos sobre los que se condiciona).

**Redes de correlaciones parciales:** se construyen igual que las redes de correlación, pero usando correlación parcial en vez de correlación simple — con su propio test de hipótesis análogo.

🔹 **Case study: regulación génica en E. coli.**
- Contexto: los genes que controlan la expresión de otros son *transcription factors* (TFs); los controlados son *targets*; la regulación puede ser activación o represión.
- Datos: 4.345 genes bajo 445 condiciones experimentales, 153 TFs conocidos, pares TF/target de referencia (base RegulonDB).
- Se comparan 3 métodos: (1) correlación de Pearson simple, (2) correlación parcial condicionando a un solo TF por vez, (3) correlación parcial completa condicionando simultáneamente a los 152 TFs restantes.
- Resultado: el método 1 es el peor, pero **ninguno funciona muy bien** — la correlación no es un indicador fuerte de regulación en estos datos (alta precisión pero muy bajo recall).
- Aun así, sirve para *generar hipótesis*: para el TF *lrp* se encontraron 11 interacciones candidatas, 10 confirmadas experimentalmente y 5 nuevas no reportadas antes.

## Undirected Gaussian Graphical Models

🔹 Si se asume que las variables $\{x_i\}$ tienen distribución gaussiana multivariada, vale un **teorema clave**: la correlación parcial entre $x_i$ y $x_j$, condicionando a *todos* los demás nodos, es cero **si y solo si** $x_i$ y $x_j$ son condicionalmente independientes dado el resto. Es decir, bajo el supuesto gaussiano, "correlación parcial (completa) nula" y "sin relación directa entre esos dos nodos" son exactamente lo mismo.

🔹 Esto define el **grafo de independencia condicional**: hay arista entre $i$ y $j$ si y solo si esa correlación parcial completa es distinta de cero — un caso particular y muy usado de red de correlación parcial. También se lo conoce como **Gaussian Markov Random Field (GMRF)**.

🔹 **Covariance selection:** se define la matriz de precisión $\Theta=\Sigma^{-1}$. Resultado clave: bajo el modelo GMRF, una entrada no nula en $\Theta$ equivale exactamente a una arista en el grafo. El problema de inferir $G$ usando esta equivalencia se llama *covariance selection* [Dempster'74]. Los métodos clásicos testean cada par por separado y en general no escalan bien: cuando el número de observaciones $P$ es mucho menor que el número de nodos $N$, estimar $\hat\Sigma$ de forma confiable es difícil.

🔹 **GMRFs con restricción de Laplaciano:** una variante impone que $\Theta$ sea directamente la Laplaciana del grafo. Esta formulación **favorece grafos sobre los que las señales son suaves** — es el puente conceptual hacia la sección siguiente.

### Digresión: sparsity y norma $\ell_1$

- La norma $\ell_1$ induce **esparsidad** (estimador **Lasso** [Tibshirani'94]): una ventaja de esta norma es que hace que algunas coordenadas se anulen exactamente.
- **Graphical Lasso** [Yuan-Lin'07]: estima toda la matriz de precisión en conjunto, con una penalización $\ell_1$ que promueve esparsidad. Efectivo cuando $P \ll N$; da modelos interpretables. 🔹 Existen métodos escalables basados en *coordinate-descent*.

### Covariance selection meets linear regression

Objetivo: mejorar la explicación del método / chequear el algoritmo, estimando la matriz por partes usando combinaciones lineales y viendo qué nodos aportan más.

🔹 Formalización: el valor esperado de $x_i$ condicionado al resto de las variables es una función **lineal** de esas variables, y los coeficientes de esa regresión están directamente relacionados con las entradas de $\Theta$ — el soporte (coeficientes no nulos) de esa regresión coincide exactamente con la vecindad del nodo en el grafo. Esto justifica estimar el grafo vía regresión en vez de estimar toda la matriz de precisión.

- Método práctico: se toma la primera fila, se hace una regresión, y se conservan los pocos coeficientes más significativos.

**Neighborhood-based sparse regression (NBR):** se resuelve un problema Lasso independiente por cada nodo (regresión de ese nodo contra todos los demás). 🔹 Como no hay garantía de que la relación salga simétrica, se usan reglas **OR** o **AND** para decidir si declarar una arista cuando solo uno de los dos nodos "ve" al otro como vecino.

🔹 Los tres enfoques vistos — testear correlaciones parciales, covariance selection (vía $\Theta$), y neighborhood-based regression (vía $\beta$) — son **matemáticamente equivalentes**, solo cambia el camino de cálculo.

🔹 **Comparación NBR vs. Graphical Lasso:**

| | NBR | Graphical Lasso |
|---|---|---|
| Optimiza | verosimilitud condicional por nodo (independientes) | verosimilitud global (regularizada) |
| Fuerza $\Theta\succeq 0$ | No | Sí |
| Velocidad | más rápido, paralelizable | más lento |
| Eficiencia estadística | menor | mayor (usa mejor la info conjunta) |

🔹 NBR se puede extender a variables discretas o mixtas (*Ising-model selection*).

> Estos modelos (Gaussian graphical models, graphical Lasso, covariance selection) son para **variables aleatorias Gaussianas**.

## Aprendiendo grafos a partir de observaciones de señales suaves

🔹 **Motivación:** se busca un grafo sobre el cual las señales admitan regularidad, por ejemplo para predicción por vecino más cercano (*graph smoothing*) o aprendizaje semi-supervisado. Muchos datos de redes reales son suaves porque la formación de la red suele basarse en homofilia o proximidad en un espacio latente (nodos parecidos tienden a conectarse, y por ende a tener valores de señal parecidos).

🔹 **Planteo formal del problema:** dadas observaciones $X$, identificar un grafo $G$ tal que esas señales sean *suaves* en $G$. El criterio de suavidad es la **energía de Dirichlet**, $TV(x)=x^\top Lx$: cuanto más chica, más suave la señal respecto al grafo.

🔹 Ejemplos de aplicación: predicción de función de proteínas en levadura (proteínas conectadas tienden a compartir función); red de colaboración entre abogados (colaboran más entre pares del mismo tipo de práctica legal).

Recordando el ejemplo de la primera clase: a medida que avanza la señal, es menos suave. Frecuencia grande → coeficiente chico.

🔹 **Graph Signal Processing (GSP) y Graph Fourier Transform (GFT):** se usan los vectores propios de la matriz del grafo (adyacencia o Laplaciana) como "base frecuencial", igual que en procesamiento de señales clásico se explota la estructura temporal (un ciclo temporal da lugar a la Transformada de Fourier clásica). La GFT de una señal es su proyección en esa base; la GFT inversa reconstruye la señal original.

🔹 **Modos frecuenciales de la Laplaciana** (esto es a lo que se refería el ejemplo de "Diapo 41"): la Total Variation $TV(x)=x^\top Lx$ mide la suavidad de una señal respecto al grafo. Para los vectores propios de la Laplaciana, ese valor coincide exactamente con el autovalor asociado — por eso los autovalores de $L$ se interpretan como "frecuencias": autovalor chico = modo suave (frecuencia baja), autovalor grande = modo poco suave (frecuencia alta).

Ahora se infiere el grafo a partir de la **matriz Laplaciana**: se aprende un grafo a partir de una señal suave.

🔹 **Modelo de análisis factorial basado en la Laplaciana** [Dong et al'16]: se modela la señal observada como combinación de los modos de Fourier del grafo más ruido, dando más peso a los coeficientes asociados a autovalores chicos (baja frecuencia) — el modelo favorece explícitamente señales "pasa-bajo" (suaves).

🔹 **Inferencia como denoising:** buscar el grafo se puede plantear como buscar simultáneamente la Laplaciana $L$ y una versión "limpia" (sin ruido) de la señal, penalizando la falta de suavidad de esa versión limpia respecto a $L$.

**Formulación y algoritmo:** hay que definir cuánta importancia se le da a la suavidad y cuánta a la esparsidad. 🔹 Concretamente, se combina un término de ajuste a los datos, un término de suavidad y un término de esparsidad de aristas, con restricciones que evitan la solución trivial de grafo vacío. El problema no es conjuntamente convexo en $L$ y en la señal limpia, pero sí es *bi-convexo*, lo que permite resolverlo por **minimización alternada**: fijando la señal limpia se optimiza $L$; fijando $L$, la señal limpia óptima tiene solución cerrada y equivale a aplicar a los datos un filtro pasa-bajos.

🔹 Impacto de los parámetros: más peso a la esparsidad da menos aristas; cuando hay poco ruido, el cociente entre el parámetro de suavidad y el de esparsidad determina la calidad del grafo recuperado.

🔹 **Ejemplo aplicado (temperaturas en Suiza):** con 89 estaciones meteorológicas, usar directamente distancia geográfica no captura bien la similitud de temperatura (por diferencias de altura). Aprendiendo el grafo desde las temperaturas mismas y aplicando *spectral clustering* sobre él aparecen dos clusters sensatos (estaciones altas vs. bajas), cosa que *k-means* directo sobre las temperaturas no logra.

### Signal smoothness meets edge sparsity

🔹 Alternativa a usar la Laplaciana directamente [Kalofolias'16]: existe una relación directa entre la suavidad total de las señales y la norma $\ell_1$ de la matriz de adyacencia ponderada por las distancias entre nodos. En otras palabras, minimizar la falta de suavidad equivale a favorecer aristas entre nodos con señales parecidas (distancia chica) y penalizar las demás. Esto permite reformular el problema directamente en términos de la matriz de adyacencia $A$ en vez de $L$, lo cual simplifica las restricciones (quedan desacopladas).

🔹 El modelo agrega una barrera logarítmica que evita que algún nodo quede con grado cero, y un término que penaliza pesos grandes para controlar la densidad del grafo. Se resuelve con un método primal-dual paralelizable.

🔹 **Ejemplo aplicado (dígitos USPS):** aprendizaje del grafo para clustering espectral en 10 clases de dígitos manuscritos — este enfoque resulta más robusto a la densidad elegida para el grafo (no deja nodos aislados) que un grafo de vecinos más cercanos (k-NN) clásico.

### Aprendizaje de grafos seleccionando aristas

🔹 Alternativa [Chepuri et al'17]: en vez de optimizar sobre pesos continuos, se representa la topología con un vector binario que indica qué aristas están presentes, permitiendo fijar directamente el número exacto de aristas deseado ($K$). Sin ruido, la solución es simple: calcular un "score" de suavidad para cada arista candidata y quedarse con las $K$ de menor score. Existe una versión más realista con ruido en las mediciones.

🔹 **Comparación de este enfoque:**

| A favor | En contra |
|---|---|
| Controla directamente la cantidad de aristas | No garantiza que el grafo quede conectado |
| Algoritmo simple sin ruido | No permite pesos en las aristas (todas iguales) |
| No hace falta imponer restricciones extra para tener una Laplaciana válida | |

### Case study: clasificación con grafos discriminativos

🔹 Si se tienen señales etiquetadas de distintas clases, se puede aprender **un grafo distinto por clase** (asumiendo que las señales de cada clase son suaves respecto a un grafo propio), agregando un término que penaliza que ese grafo también favorezca la suavidad de señales de *otras* clases (discriminabilidad) [Saboksayr et al'21]. Para clasificar una señal nueva, se la proyecta usando la base de Fourier de cada grafo aprendido y se elige la clase cuya proyección concentra más energía en las frecuencias bajas.

🔹 Aplicación real: reconocimiento de emociones a partir de señales EEG (dataset DEAP, 32 personas, 32 canales), clasificando valencia emocional alta vs. baja — accuracy promedio de 92,7%. Los grafos aprendidos por clase muestran diferencias interpretables en la conectividad del lóbulo frontal según la intensidad emocional.

## 🎯 Posibles puntos de examen

- Diferencia entre los 3 problemas canónicos de inferencia de topología: qué se observa y qué se busca en cada uno (predicción de enlaces / association network inference / tomographic).
- Qué es un test de hipótesis, y cómo se define $H_0$ en el contexto de redes de correlación.
- Qué es el problema de testeo múltiple en grafos y para qué sirve el FDR.
- "Correlación no implica causalidad": por qué se necesita correlación parcial y no alcanza con correlación simple.
- Bajo qué supuesto (gaussiano) la correlación parcial completa nula equivale a independencia condicional (GMRF).
- Relación entre la matriz de precisión $\Theta$ y las aristas del grafo (covariance selection).
- Diferencias entre Graphical Lasso y neighborhood-based regression (velocidad vs. eficiencia estadística).
- Qué mide la Total Variation / energía de Dirichlet, y qué representan los autovalores de la Laplaciana (frecuencias del grafo).
- Qué términos combina la función objetivo para aprender un grafo desde señales suaves (ajuste + suavidad + esparsidad).
- Diferencia entre parametrizar el aprendizaje del grafo en $L$ vs. en $A$ (enfoque de Kalofolias).
- Ventajas y desventajas del enfoque de selección de aristas por cardinalidad fija.
