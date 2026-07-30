# Clase 4 – Graph Neural Networks (GNNs)

## Intro

- Algunos ejemplos *(no desarrollado en el apunte)*

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

### De información local a global

- Se puede operar el mismo filtro en distintos grafos.

![alt text](image-2.png)

- En principio, la familia de funciones definida así cumple con lo que se buscaba (independencia del orden, aplicable a grafos de distinto tamaño).

> Hasta acá se vieron filtros. Sigue el aprendizaje.

## GNN: Construcción

- Esquema: Señal → Filtro → Nueva señal.
- Aprendiendo usando un **Graph Perceptron**.
- La salida da un parámetro $H$ (tensor) — se va a querer elegir la cantidad de $h$'s.
- Ventaja de las GNN: *(sin terminar en el apunte — ver puntos a revisar)*

## Múltiples features

*(visto muy rápido en el apunte)*

- En las librerías se habla poco de "convolución"; se habla más de **message passing**.
- Ambos "son lo mismo".

## Entrenando una GNN

- Una vez construida la GNN, se quiere llegar a un vector que represente algo.
- Aprender ⟺ minimizar la pérdida.

### Predicción a nivel de nodo

- Ejemplo: decidir si un usuario de una red social es un bot.
- *(pregunta sin terminar en el apunte: "Training? Ocultar o dejar" — ver puntos a revisar)*

### Predicción de aristas

*(mencionado como título, sin desarrollo en el apunte)*

### Clasificar grafos

- Si la identidad de los nodos **no** importa: *(frase incompleta en el apunte — ver puntos a revisar)*
- Si la identidad de los nodos **sí** importa, una GNN "no es lo mejor".

## Permutation equivariance

*(mencionado como título, sin desarrollo en el apunte)*

## Perturbaciones

- Fourier *(mencionado, sin desarrollo en el apunte)*

---

## ⚠️ Puntos poco claros / a revisar

- "Graph Convolution => GSP => Graph Filtering (??)" — no queda clara la relación entre estos tres conceptos tal como está anotada; el signo de interrogación sugiere que ni en el momento quedó claro.
- La referencia a "(Diapo 11)" no se entiende sin ver la diapositiva correspondiente.
- "Ventaja GNN:" quedó sin completar en el apunte.
- En predicción a nivel de nodo hay ítems vacíos y una frase sin terminar ("Training? Ocultar o dejar") — no se entiende qué se quiso anotar ahí (¿ocultar etiquetas de nodos para entrenar/testear?).
- "Predicción de aristas" quedó solo como título, sin contenido.
- En "Clasificar grafos", la frase "Si la identidad de nodos NO importa," quedó incompleta (no se anotó qué pasa en ese caso).
- "Permutation equivariance" y "Perturbaciones" quedaron como títulos sueltos, prácticamente sin desarrollo (en Perturbaciones solo se anotó la palabra "Fourier").
- "Sección 11, algoritmo" es una referencia aislada al final del apunte, sin contexto de a qué algoritmo se refiere.
