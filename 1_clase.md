# Clase 1 – Introducción y motivación

**Aprendizaje Automático para Datos en Grafos**
Docentes: Marcelo Fiori (IMERL, UDELAR – Universidad de la República) y Federico Larroca
Contenido del curso basado en el material de Gonzalo Mateos (University of Rochester)

> Apuntes tomados en clase, complementados con `clase_introduccion.pdf`. Cubren la clase hasta la hoja 54 inclusive, con la excepción de las diapositivas de **"Covarianzas y componentes principales"**, que se saltearon.

## Organización del curso

- 5 clases + 4 notebooks exploratorios (los notebooks no son obligatorios, son herramientas de apoyo)
- **Aprobación:** cuestionario final de ≈10 preguntas de opción múltiple (mínimo 6 correctas)

---

## ¿Qué es un grafo?

Según la RAE, un grafo es *"una colección de cosas interconectadas"*: hay múltiples cosas (nodos) y estas están conectadas (aristas) — dos extremos.

### Contexto histórico

- **1735** — L. Euler funda la teoría de grafos: los siete puentes de Königsberg
- **1845** — Leyes de los circuitos eléctricos (Kirchhoff)
- **1874** — Estructura molecular en química (Cayley)
- **1930** — Representación en red de interacciones sociales
- Redes eléctricas de potencia; telecomunicaciones e Internet
- Google, Facebook, Twitter

> Entender sistemas complejos ⇔ entender las redes detrás de ellos.

---

## Motivación: impacto de las redes

- **Salud:** predicción de epidemias; el Proyecto Conectoma Humano (*Human Connectome Project*) para mapear la circuitería cerebral.
- **Seguridad nacional:** el análisis de redes sociales fue clave para capturar a Saddam Hussein.
- **Descubrimiento científico:** el plegamiento de proteínas (predecir la estructura 3D de una proteína a partir de su secuencia 1D de aminoácidos).

### Objetivos de la ciencia de datos en redes

- **Revelar** patrones y propiedades estadísticas de los datos de red
- **Entender** los fundamentos del comportamiento y la estructura de las redes
- **Diseñar** redes más eficientes en el uso de recursos, robustas y socialmente inteligentes

> Inferencia estadística ≈ aprendizaje automático: la teoría de grafos se encuentra con la inferencia estadística.

---

## Enfoque del curso

**Procesamiento de señales y aprendizaje a partir de datos de red.**

Un desafío central es que estos datos no tienen fuertes *priors* estructurales ni geométricos (a diferencia de, por ejemplo, imágenes o señales temporales regulares).

### ¿Qué se puede hacer con estos datos?

- Visualización de grafos y descubrimiento de patrones
- Modelado y generación de grafos
- Clustering y detección de comunidades
- Predicción de enlaces (aristas)
- Clasificación de nodos y aprendizaje semisupervisado
- Clasificación de grafos

Todo esto se plantea tanto para grafos dirigidos como no dirigidos (que además pueden ser dinámicos). En ambos casos los grafos se representan mediante matrices — la matriz de adyacencia es simétrica en el caso no dirigido, lo cual resulta muy útil. Los métodos para trabajar con estos datos pueden ser **probabilísticos** o **determinísticos**, y en general se apoyan en la estructura (el grafo) subyacente.

---

## Señales en grafos

Una **señal** es un atributo asociado a cada nodo del grafo (por ejemplo, una etiqueta binaria del vértice). Se representa como un vector sobre la cantidad de nodos (o como una matriz, si se trata de una señal temporal).

> Las aristas también pueden tener atributos, pero eso no es el enfoque de este curso.

> **Pregunta abierta (anotada en clase):** ¿por qué trabajar con señales?

### Procesamiento de señales en grafos (GSP)

- Grafo $G$ con matriz de adyacencia $A \in \mathbb{R}^{N \times N}$, donde $A_{ij}$ representa la proximidad entre los nodos $i$ y $j$.
- Señal $x \in \mathbb{R}^N$ sobre el grafo, donde $x_i$ es el valor de la señal en el nodo $i$.
- Idea central del GSP: **explotar la estructura codificada en $A$** para procesar $x$.

---

## De la estructura de la señal a la Transformada de Fourier en grafos

El procesamiento de señales consiste, en esencia, en explotar la estructura de la señal.

**Ejemplo — el tiempo discreto como grafo cíclico:**

- El grafo es un ciclo, representado por una matriz **circulante** (unos en la diagonal desplazada, $A_{i,i+1}$).
- El tiempo $n$ sigue al tiempo $n-1$ ⇒ el valor $x_n$ es similar a $x_{n-1}$ ⇒ se espera que la señal sea periódica (suave).
- Calculando los autovectores de la matriz circulante se obtienen las **bases de Fourier**.
- Hacer la transformada de Fourier es, en el fondo, hacer un **cambio de base** del vector señal.

**Generalización — Transformada de Fourier en Grafos (GFT):**

- Se define un operador de desplazamiento en el grafo $S$ (puede ser la matriz de adyacencia, el Laplaciano, u otro), con descomposición en autovectores $S = V \Lambda V^{-1}$ (se apoya en el teorema espectral).
- **GFT (Graph Fourier Transform):** proyección de la señal sobre el espacio de autovectores del operador de desplazamiento $S$.

---

## Modos de frecuencia del Laplaciano

La **variación total** de la señal $x$ con respecto al Laplaciano $L$:

$$TV(x) = x^\top L x = \sum_{i,j} A_{ij}(x_i - x_j)^2$$

Es una medida de **suavidad** de la señal sobre el grafo $G$ (energía de Dirichlet): cuanto más parecidos son los valores de $x$ entre nodos conectados, menor es $TV(x)$.
