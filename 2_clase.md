# Clase 2 – Modelos Estadísticos de Grafos

> Apuntes tomados en clase, completados y verificados contra `clase_2.pdf` (69 diapositivas). Formato pensado para repaso rápido de cara al multiple choice: lo importante en negrita, fórmulas clave, ejemplos solo cuando ayudan a fijar el concepto.

## Motivación: ¿para qué modelar estadísticamente grafos?

1. **Explicar mecanismos** detrás de propiedades observadas en grafos reales (ej: efectos *small-world*, distribuciones de grado con ley de potencia)
2. **Modelo nulo** para testear propiedades de un grafo (ej: ¿es raro el clustering coefficient que observo?)
3. **Evaluar factores predictivos** de relaciones entre nodos (ej: ¿hay efectos transitivos o de reciprocidad?)
4. **Generación de grafos** para evaluar algoritmos en condiciones controladas

---

## Modelo de Grafos Aleatorios (marco general)

Colección $\{P(G), G \in \mathcal{G} : \theta \in \Theta\}$:

- **$\mathcal{G}$:** conjunto de grafos posibles bajo el modelo
- **$P(\cdot)$:** distribución de probabilidad sobre $\mathcal{G}$
- **$\theta$:** vector de parámetros, $\theta \in \Theta$

La riqueza y utilidad del modelo depende de cómo se especifique $P(\cdot)$.

### Especificación del modelo (de lo simple a lo complejo)

1. $P(\cdot)$ **uniforme** en $\mathcal{G}$, respetando restricciones estructurales — ej: Erdős-Rényi, Configuration model
2. Inducir $P(\cdot)$ mediante un **modelo generativo** — ej: small world, preferential attachment, copying models, deep generative models
3. Modelar **propiedades estructurales** y su efecto sobre la topología de $\mathcal{G}$ — ej: *exponential random graph models*
4. Modelar la tendencia a conectarse mediante **variables latentes** — ej: SBM, grafones, RDPGs

> El costo computacional de los algoritmos de inferencia y generación es un aspecto importante a la hora de elegir.

---

## Grafos Aleatorios Clásicos: Erdős–Rényi(-Gilbert)

Se asigna la misma probabilidad a todos los grafos no dirigidos de cierto orden y tamaño.

- Colección $\mathcal{G}_{N_v,N_e}$: grafos $G(V,E)$ con $|V|=N_v$, $|E|=N_e$. $P(G) = \binom{N}{N_e}^{-1}$ para cada $G$, con $N = \binom{N_v}{2}$.
- **ER(n, p)** (o $G_{n,p}$): caso particular no dirigido, $n$ nodos, cada arista $(u,v)$ existe con probabilidad $p$ independiente del resto.
- **Simulación eficiente:** sortear $\binom{n}{2}$ Bernoulli(p) i.i.d. es ineficiente si $p \ll n^{-1}$ (grafo esparso, mayoría ceros). Mejor: sortear Geo(p) i.i.d. para decidir qué aristas generar → corre en $O(N_v+N_e)$.

### Propiedades de ER(n, p)

**P1) Distribución de grados:**
$$D_i = \sum_{j \neq i} A_{ij}, \quad (A_{ij})_{j\neq i} \sim \text{Ber}(p) \text{ i.i.d.} \Rightarrow D_i \sim \text{Bin}(n-1, p)$$

$P(d)$ está concentrado alrededor de $p(n-1)$ con colas exponenciales — cota de **Hoeffding**:
$$P(|D_i - p(n-1)| \geq t) \leq e^{-2t^2/(n-1)}$$

> **¿Por qué importa esto?** La Binomial es una distribución de **cola liviana**: la probabilidad de un nodo con grado muy distinto al promedio cae exponencialmente rápido (por Hoeffding). Es decir, en ER **todos los nodos tienen un grado parecido**. Esto contrasta fuerte con las redes reales, que suelen tener **colas pesadas** (ver nota en Configuration Models) — es una de las razones por las que ER no las modela bien.

**P2) Transición de fase en la aparición de una componente gigante:**

- $np > 1$ → componente gigante de tamaño $O(n)$ w.h.p.
- $np < 1$ → componentes de tamaño $O(\log n)$ w.h.p.

*Demostración (idea):* proceso de exploración con estados activo ($A_t$), inactivo ($I_t$), explorado ($E_t$). El par $(E_t, A_t)$ es una cadena de Markov a tiempo continuo. Reescalando con $p=c/n$ (vecinos promedio $\to c$) y tomando $n\to\infty$ (límite fluido), se llega a una ecuación diferencial con solución analítica:
$$e_\infty = \mu, \qquad a_\infty = 1-\mu-e^{-c}$$
Hay componente gigante $\iff c>1$, y su tamaño (proporcional a $n$) es la solución positiva de $1-\mu = e^{-c}$.

**P3)** Clustering coefficient pequeño, de orden $O(n^{-1})$, y diámetro corto, de orden $O(\log n)$ w.h.p.

> **¿Qué es el clustering coefficient?** Mide qué tan seguido los vecinos de un nodo también son vecinos entre sí (proporción de "triángulos cerrados" sobre tríos conectados). *Interpretación:* alto → los amigos de tus amigos también son tus amigos (típico en redes sociales reales, por homofilia). En ER, $cl(G)=O(n^{-1})\to 0$: casi no hay triángulos, porque cada arista se sortea de forma puramente independiente y al azar, sin ninguna noción de "cercanía" entre nodos.
>
> **¿Qué es el diámetro de un grafo?** Es la mayor de las distancias más cortas entre cualquier par de nodos — el "peor caso" para llegar de un nodo a otro. *Interpretación:* chico → cualquier par de nodos está a pocos saltos (efecto "mundo pequeño"). En ER, $diam(G)=O(\log n)$: aunque las conexiones son puramente aleatorias, el grafo ya es "pequeño" — el problema de ER no es el diámetro, es que **le falta clustering**.
>
> Esta combinación (diámetro chico pero clustering → 0) es precisamente lo que **no** coincide con redes reales, que tienen ambas cosas a la vez — motiva Configuration models y luego Watts-Strogatz.

---

## Configuration Models

**Receta para generalizar ER:** especificar $\mathcal{G}$ con grafos de orden $N_v$ que además cumplan cierta característica, y asignar probabilidad uniforme dentro de esa colección.

- Si solo se conoce la cantidad media de vecinos $c$ → ER(n, c/n) es razonable.
- Si se conoce la **secuencia o distribución de grados completa** → hace falta el **configuration model**.

> **¿Qué es una distribución de grado de "cola pesada" (heavy-tailed)?** El profesor insiste bastante en esto, así que vale la pena tenerlo claro. Es lo opuesto a la Binomial de ER: la probabilidad de nodos con grado muy alto **no** decae exponencialmente, sino mucho más lento — típicamente como una **ley de potencia (power-law):** $P(d) \propto d^{-\tau}$. *Interpretación práctica:* en vez de que todos los nodos se parezcan (como en ER), aparecen unos pocos nodos "hub" con muchísimas más conexiones que el resto — pensá en aeropuertos muy conectados, sitios web muy enlazados, o actores que trabajaron en cientos de películas. Casi todas las redes reales (Internet, redes sociales, la WWW, colaboración de actores) muestran este fenómeno, y **ER no lo puede generar** (su distribución de grado siempre está concentrada, con colas exponenciales). Esta es la motivación central del configuration model: dado que ER falla en esto, se necesita un modelo que permita *fijar directamente* la distribución de grado deseada (heavy-tailed o la que sea).
>
> Ejemplos empíricos con colas pesadas mencionados en clase: conectividad entre Autonomous Systems, actores compartiendo películas en IMDb, amigos en Facebook, la WWW.

**Definición:** secuencia de grados dada $d=\{d_1,\dots,d_{N_v}\}$. El tamaño queda determinado: $N_e = \sum_i d_i/2$. Equivale a especificar el modelo como distribución condicional sobre $\mathcal{G}_{N_v,N_e}$.

- Útiles como modelos **nulos** (comparar grafo observado vs. uno con $P(d)$ power-law) y para imponer restricciones de conectividad (ej: sin nodos aislados).

### Simulación — CM$_n$(d)

- **Matching algorithm** (usado para el análisis): nodos con semi-aristas ("stubs"), se unen al azar.
  - No toda secuencia de grados es "gráfica"; pueden aparecer self-loops y multi-aristas → se borran.
  - Se puede probar que, bajo condiciones (ej. media finita), la distribución resultante $P^{(er)}(d)$ converge a $P(d)$: $\sum_d |P^{(er)}(d)-P(d)| \to 0$.
  - Además, CM$_n$(d) genera al azar uniforme entre todos los grafos con esa secuencia de grados.
- **Switching algorithm:** parte de un grafo inicial y aleatoriamente intercambia pares de aristas, repitiendo ~$100 N_e$ veces.

### Resultados de CM$_n$(d)

1. **Transición de fase** en la aparición de componente gigante, tamaño $O(n)$. Condición: $\kappa = E\{D(D-1)\}/E\{D\} > 1$ (en ER equivale a $c$, igual que antes). Intuición: el grado (menos uno) de un vecino típico debe superar 1 en promedio — se usa la distribución *size-biased* del grado del vecino.
2. **Clustering coefficient → 0**, igual que en ER.
3. **Distancias típicas** dependen de $P(d)$:
   - Si $\text{Var}(D)<\infty$ → distancias típicas $O(\log n)$
   - Power-law $P(d)\propto d^{-\tau}$ con $\tau\in(2,3)$ (media finita, varianza infinita, es decir, **cola pesada de verdad**) → distancias típicas $O(\log\log n)$, aún más chico que en el caso ER-like
4. Se pueden acotar el *independence number* de estos grafos analizando procesos de exploración mediante límites fluidos (Brightwell-Janson-Luczak'17; Bermolen-Jonckheere-Moyal'17).

---

## Modelo Small World (Watts-Strogatz)

### Contexto histórico

- "Seis grados de separación" se popularizó con la obra de teatro de [Guare'90], pero el concepto es más viejo.
- **Karinthy (1929):** cuento húngaro — el mundo se "achica" por el aumento de conectividad humana; apuesta a poder contactar a cualquiera con ≤5 intermediarios.
- **Kochen-Pool (1950s):** primer tratamiento matemático formal, pero deja sin responder la cuestión de los "grados de separación".
- **Milgram (1967):** experimento que sí lo mide.

### Experimento de Milgram

- Pregunta: ¿cuál es la distancia típica entre dos personas en la red social global?
- Método: 296 cartas enviadas desde Wichita (Kansas) y Omaha (Nebraska) a un único contacto en Boston. Regla 1: si el destinatario actual conoce al contacto por su nombre de pila, se la manda directo. Regla 2: si no, la reenvía a quien crea más cercano al contacto.
- **Resultado:** 64 de 296 cartas llegaron, con largo promedio $\mu = 6.2$ → inspiró la obra de Guare.
- **Conclusión:** caminos cortos conectan a dos personas cualesquiera, y en abundancia.

### ¿Es razonable la teoría del mundo pequeño?

Si cada persona tiene 100 amigos, cada uno con otros 100... tras 5 grados hay $10^{10}$ personas (más del doble de la población mundial) — **no es un modelo razonable** para una red social real, que típicamente tiene:

- **Homofilia** [Lazarsfeld'54]
- **Triángulos cerrados abundantes** [Rapoport'53]

Pregunta clave: ¿cómo tener una red muy estructurada localmente pero pequeña globalmente?

### Estructura vs. aleatoriedad como extremos

| Modelo | Clustering | Diámetro |
|---|---|---|
| Lattice regular $G_r$ (cada nodo conectado a sus $2r$ vecinos más cercanos) | alto: $cl(G_r)=\frac{3r-3}{4r-2}$ | alto: $diam(G_r)=\frac{N_v}{2r}$ |
| ER$(N_v,p)$ con $p=O(N_v^{-1})$ | bajo: $O(N_v^{-1})$ | bajo: $O(\log N_v)$ |

### El modelo Watts-Strogatz

Mezcla estructura con una pizca de aleatoriedad:

1. Inicializar con un lattice regular con el clustering buscado
2. Introducir atajos: cada arista se reconecta aleatoriamente con probabilidad (pequeña) $p$

La reconexión interpola entre ambos extremos. En un rango intermedio de $p$ se logra **clustering alto y diámetro bajo simultáneamente** (la "zona small-world").

**Propiedades:**

- P1) Para $N_v$ grande: $cl(G) \approx cl(G_r)(1-p)^3$
- P2) La distribución de grados se concentra alrededor de su media $2r$ (sigue sin ser de cola pesada — ese problema lo resuelve el configuration model, no Watts-Strogatz)

**Aplicaciones:** dispersión de rumores/noticias (falsas), dispersión de epidemias, búsqueda de información en redes.

---

## Modelos de Variables Latentes

Ampliamente usadas para modelar datos con observaciones parciales (ej: Hidden Markov Models, análisis factorial). Dos variantes en redes:

- **Modelos de clases latentes:** la pertenencia a una clase no observada marca la tendencia a conectarse (→ SBM)
- **Modelos de vectores latentes:** la conexión depende de qué tan "cerca" están los nodos en un espacio latente (→ RDPG)

> Ejemplo: red de colaboración de actores en IMDb (2017-2021), $N_v=21617$, $N_e=73702$. Distintas tasas de conexión según dónde trabajan (Hollywood, Bollywood, Nollywood, independientes...). Ni ER ni configuration model capturan esta estructura.

### Stochastic Block Models (SBM)

Grupos/comunidades $C_1,\dots,C_Q$, con tasas de conexión $\theta_{qr}$ entre/dentro de grupos.

**Modelo generativo** para grafo no dirigido $G(V,E)$:

- Cada vértice $i$ pertenece independientemente a $C_q$ con probabilidad $\pi_q$ ($\pi=[\pi_1,\dots,\pi_Q]$, $\sum \pi_q=1$)
- Para cada par $i \in C_q, j \in C_r$: $(i,j)\in E$ con probabilidad $\theta_{qr}$

Formalmente, con $Z_{iq}=\mathbb{1}\{i\in C_q\}$: $Z_i \overset{iid}{\sim} \text{Multinomial}(1,\pi)$, y $A_{ij}\mid Z_i,Z_j \sim \text{Bernoulli}(\theta_{z_i,z_j})$.

- **Parámetros:** $Q$ proporciones $\pi_q$ + $Q(Q+1)/2$ probabilidades $\theta_{qr}$.
- Es una mezcla de grafos ER: $P(A_{ij}=1) = \sum_{q,r} \pi_q \pi_r \theta_{qr}$.
- Más flexible que ER/configuration para capturar estructura real, pero con temas de identificabilidad [Allman'11] y transición de fase propia [Söderberg'02-03].
- Ejemplo de matriz $\theta$ (3 comunidades, caso **asortativo**: valores en la diagonal ≫ fuera de diagonal → más probable conectarse dentro de la propia comunidad):
$$\pi = \begin{bmatrix}1/3\\1/3\\1/3\end{bmatrix}, \quad \theta = \begin{bmatrix}0.5 & 0.1 & 0.05\\ 0.1 & 0.3 & 0.05\\ 0.05 & 0.05 & 0.9\end{bmatrix}$$

> Nota: el SBM tampoco resuelve por sí solo el problema de la cola pesada del grado — dentro de cada bloque, el grado sigue siendo aproximadamente Binomial/concentrado (es una mezcla de ER). Para eso está el **degree-corrected SBM** (ver Extensiones, más abajo).

### Grafones (graphons) y f-random graphs

Variante **no paramétrica** del SBM:

$$U_1,\dots,U_{N_v} \overset{iid}{\sim} \text{Unif}[0,1], \qquad A_{ij}\mid U_i=u_i,U_j=u_j \sim \text{Bernoulli}(f(u_i,u_j))$$

- **Grafón:** función simétrica y medible $f:[0,1]^2\to[0,1]$. El grafo resultante es un *f-random graph*.
- $U_i$ da la posición latente de cada nodo; $f(u_i,u_j)$ la probabilidad de conexión.
- **El SBM es un caso particular:** particionar $[0,1]$ en $Q$ subintervalos de largo $\pi_1,\dots,\pi_Q$, tomar el producto cartesiano ($Q^2$ bloques), y definir $f$ constante a trozos con "altura" $\theta_{qr}$ en cada bloque.
- Cualquier función medible se aproxima por una constante a trozos → cualquier f-random graph se aproxima (en distribución) por un SBM, pero el número de bloques $Q$ necesario puede ser enorme.

### Plausibilidad: ¿puede un SBM aproximar cualquier grafo?

- **Cut distance** entre grafos $G, G'$ (mismo $|V|=N_v$): $d(G,G') = \frac{1}{N_v^2}\max_{S,T} \left|\sum_{i\in S,j\in T}(A_{ij}-A'_{ij})\right|$ — es una métrica.
- **Teorema:** para todo $\varepsilon>0$ y todo grafo $G$, existe una partición en $Q \leq 2^{2/\varepsilon^2}$ clases tal que $d(G,G_P)\leq \varepsilon$ (donde $G_P$ es la aproximación por bloques, con peso = densidad de aristas entre bloques).
- Justifica que un SBM puede aproximar cualquier grafo — pero la cota en $Q$ puede ser gigantesca.

### Extensiones de SBM

- **Degree-corrected SBM:** permite comunidades con distribución de grados amplia (cola pesada dentro de cada bloque) [Karrer-Newman'11]
- **Mixed-membership SBM:** los nodos pueden pertenecer a más de una clase [Airoldi'08]
- **SBM jerárquicos:** clustering jerárquico combinado con SBM [Clauset et al'08]

---

## Random Dot Product Graphs (RDPG)

Espacio latente $\mathcal{X}_d \subseteq \mathbb{R}^d$ tal que $x^\top y \in [0,1]$ para todo $x,y$; distribución de producto interno $F$.

$$x_1,\dots,x_{N_v} \overset{iid}{\sim} F, \qquad A_{ij}\mid x_i,x_j \sim \text{Bernoulli}(x_i^\top x_j)$$

Posiciones $X=[x_1,\dots,x_{N_v}]^\top \in \mathbb{R}^{N_v\times d}$.

### Conexión con otros modelos

- **ER(n,p)** = RDPG con $d=1$, $\mathcal{X}_d=\{\sqrt{p}\}$
- **SBM** = RDPG con $F$ tal que $P(X=x_q)=\pi_q$ y $x_q^\top x_r = \theta_{qr}$ → **los RDPG son al menos tan expresivos como los SBM**
- Caso especial de modelos de posición latente [Hoff'02]: $A_{ij}\sim\text{Bernoulli}(\kappa(x_i,x_j))$; el RDPG aproxima cualquier $\kappa$ con $d$ suficientemente grande [Tang'13]

### Estimación: Adjacency Spectral Embedding (ASE)

- MLE es la idea natural pero inviable para $N_v$ grande.
- $P_{ij}=P((i,j)\in E)$; el modelo dice $P=XX^\top$, y $A$ es una realización ruidosa de $P$ ($E[A]=P$) → mínimos cuadrados: $\hat{X}_{LS} = \arg\min_X \|XX^\top - A\|_F^2$
- $A$ real y simétrica: $A=U\Lambda U^\top$ (autovalores $\lambda_1\geq\dots\geq\lambda_{N_v}$). Tomando los $d$ autovectores de mayores autovalores positivos: $\hat{\Lambda}=\text{diag}(\lambda_1^+,\dots,\lambda_d^+)$, $\hat{U}=[u_1,\dots,u_d]$.
- **ASE:** $\hat{X}_{LS} = \hat{U}\hat{\Lambda}^{1/2}$ (mejor aproximación de rango $d$ semidefinida positiva de $A$)
- **¿Es única la solución?** No — el producto interno es invariante a rotaciones ($XW(XW)^\top = XX^\top$ si $WW^\top=I$). El embedding es identificable **módulo rotaciones**.
- Elegir $d$: buscar un "codo" en el gráfico de autovalores (scree plot).

### Interpretabilidad

- Ejemplo Zachary's karate club ($N_v=34$): con $d=2$, el administrador y el instructor del club quedan con embeddings **ortogonales**.
- **Alineación del vector** → afinidad/comunidad entre nodos
- **Magnitud del vector** → conectividad (grado) del nodo
- Ejemplo grafo bipartito de votación del Senado de EEUU: polarización clara en 2019, no tanto en 1977.

### Extensiones y aplicación: online change point detection

- **Generalized RDPG** [Rubin-Delanchy'17]: cuando $A$ no es semidefinida positiva
- Extensiones a grafos pesados y/o dirigidos [Marenco et al]
- **Detección de cambios online:** entrenar con $m$ grafos "limpios" (sin cambios); monitorear la suma acumulada $S[m,k]=\sum_{t=m+1}^{m+k}(\hat{X}\hat{X}^\top - A[t])$; bajo la hipótesis nula (sin cambio), $\psi[m,k]=\|S[m,k]\|^2$ tiene una distribución $\chi^2$ generalizada. Liviano: costo $O(N^2)$.
  - Ejemplo real: red inalámbrica del Plan Ceibal (6 APs), detecta que un AP fue movido físicamente — se explica el cambio vía la interpretabilidad del ASE.

---

## Grafos Geométricos (RGG)

Espacio vectorial con distancia $d(u,v)$ y subconjunto $B$:
$$x_1,\dots,x_{N_v} \overset{iid}{\sim} \text{Unif}(B), \qquad A_{ij} = \mathbb{1}(d(x_i,x_j)<r)$$

Ejemplo clásico: $\mathbb{R}^2$ con norma Euclídea, $B$ = disco de radio $R$.

**Propiedades (suponiendo $r \ll R$):**

- P1) Grado promedio: $\bar{D} = \rho \pi r^2$, con densidad $\rho=N_v/(\pi R^2)$
- P2) Distribución de grados **Poisson**: $P(D_i=d) = \bar{D}^d e^{-\bar{D}}/d!$ → buen clustering, pero **cola cae muy rápido (no es cola pesada)** — mismo problema estructural que ER, no captura hubs.

**Solución:** no usar geometría Euclídea → geometría hiperbólica (que, como se ve abajo, sí produce cola pesada / power-law).

---

## Geometría Hiperbólica y Curvatura

- Círculo de radio $r$ en esfera de radio $R$: $C(r) = 2\pi R\sin(r/R) < 2\pi r$ — **curvatura positiva** → perímetros/áreas *menores* que en el plano.
- **Curvatura negativa:** perímetros/áreas *mayores* que en el plano (requiere puntos silla).
- **Pseudoesfera** (curvatura negativa constante): mapa conforme $\to$ métrica $ds' = R\frac{\sqrt{dx^2+dy^2}}{y}$ — el **modelo del semiplano de Poincaré**.
- Otras representaciones equivalentes: **disco de Poincaré-Beltrami** (el círculo unitario es el horizonte) y el **hiperboloide** con producto de Minkowski.

### Hyperbolic Random Graphs

Puntos uniformes en un círculo de radio $R$, pero en espacio **hiperbólico**; cada nodo se conecta con los que están a distancia $R$ o menos. El volumen crece **exponencialmente** con la distancia: $C(r)\approx e^r$, $A(r)\approx e^r$.

**Propiedades:**

- P1) Grado medio $\bar{D} \propto N_v e^{-R/2}$ → escalando $R$ logarítmicamente ($R=2\log(N_v/\nu)$) se obtiene un grafo esparso con grado medio finito, **independiente de $N_v$**
- P2) Distribución de grados **power-law con exponente 3** (cola pesada — a diferencia del RGG euclídeo de arriba) — otros exponentes posibles con distribución no uniforme del radio
- P3) Clustering controlable con un parámetro de "temperatura" $T$ en la versión soft: $p_{ij} = \dfrac{1}{1+e^{\frac{1}{2T}(d(x_i,x_j)-R)}}$
- P4) También produce el efecto small-world

> **La curvatura negativa es la única que permite un modelo con todas las propiedades "realistas" a la vez** (small-world + clustering alto + cola pesada / power-law en el grado). Es el punto de llegada de toda la clase: cada modelo anterior lograba una o dos de estas propiedades, pero ninguno las tres juntas.

**Métodos de embedding:** basados en probabilidad (ajustan $R,T$ maximizando verosimilitud, ej. *Mercator*) o basados en distancia (ubican los embeddings para reflejar distancias conocidas, ej. *Hydra*). Nunca hay forma cerrada → gradientes/heurísticas/aproximaciones. Librería: `github.com/CicadaUY/hypeGRL`.

---

## Resumen y moralejas (tabla del profesor — repaso rápido)

| Modelo | Idea nueva | Qué agrega |
|---|---|---|
| Erdős–Rényi | aristas i.i.d. (modelo nulo) | línea base: diámetro chico, pero clustering → 0 |
| Configuration model | fijar la distribución de grados | cualquier distribución de grados (ej. cola pesada) |
| Watts–Strogatz | lattice + una pizca de azar | clustering alto y small-world a la vez |
| SBM / grafones | clases latentes (y su límite) | comunidades / heterogeneidad |
| RDPG | vectores latentes ($x_i^\top x_j$) | embeddings interpretables (ASE) |
| Hiperbólicos | geometría del espacio latente | todas las propiedades a la vez (incluida la cola pesada) |

**Moralejas:**

- No hay un "mejor" modelo: se elige según el uso (explicar / testear / predecir / generar)
- Más realismo → más parámetros y mayor costo de inferencia/generación; rara vez hay forma cerrada (de ahí gradientes, heurísticas, aproximaciones)
- SBM, grafones, RDPG e hiperbólicos son todos de **variables latentes**: lo que cambia es la geometría del espacio latente
- Esa geometría decide qué propiedades emergen: **la curvatura negativa es la que las produce todas juntas**
