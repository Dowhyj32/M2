¿Para que modelar estadisticamente grafos? 

¿Como explico kas propiedades que observamos en grafos reales? -> small-worl o distribuciones de grado con ley de potencia

¿Fenómeno de colas pesadas?

Vamos a ver de modelos mas sencillos a mas complejos

Modelo de Grafos Aleatorios
    -Conj de grafos posibles (G)
    -P_theta distribucion de proba sobre G
    -theta vector de parametros del modelo

    Especificación del modelo:
        1. P(.) uniforme en G
        2. Modelo generativo (small world, preferential attachment, copying models)
        3. Usado en ciencias sociales o economía (no lo vamos a ver mucho)
        4. Modelar la tendencia de los nodos a conectarse mediante Variables Latentes (stochastic block models graphons, random dot product graphs)

    Grafos Aleatorios Clásicos
        Asignamos misma probabilidad a todos los grafos (no-dirigidos)

        Erdös-Renyi(-Gilbert): se notará ER(n,p) ->n:nodos; p:proba
        (Fijo los nodos y sorteo las aristas)
        la arista (u,v) existe con proba p independiente del resto

        #Vecinos = n*p

            Propiedades ER(n,p):
                Distribución: Binomial (pues cada arista es independiente)
                E{D_i} = p*(n-1)

                ¿Hoeffding?

                Transición de fase en la aparición de una componente gigante (a medida que n*p se agranda, quedan menos componentes conexas hasta llegar a un grafo conexo). Se aprecia un cambio de fase (mirar grafico)

                n*p>1 -> tiene componente gigante tamo O(n)
                n*p<1 -> tienen componentes de tamaño O(log(n))


                Algoritmo donde se demuestra esta propiedad
                activo: A_t
                inactivo: I_t
                explorado: E_t

                Normalmente vamos a buscar que el grafo no quede disconexo

                Devuelve una cadena de Markov en tiempo continuo (¿Que es? ¿A que se refiere con tiempo continuo (+inf??)?)

                Se puede ver que la cadena converge a una cierta función determinística mas el ruido (Ver como se van escalando los ejes del gráfico)

                Se llega a una ec. diferencial que tiene sol. exacta y podemos compararla con la simulación. Esto sucede cuando la cantidad de vecinos es mayor a 1

                Diametro del grafo (distancia maxima entre 2 nodos)
                Clustering coefficient?? Estos modelos no cumplen con ciertas propiedades de la practica, por que?

            Configuration models (modelos mas realistas)

                Generalizando el modelo ER
                    Chequear las escalas de los gráficos y ver como decaen (relación con colas pesadas)

                Nos interesa generar grafos con distribución que decae?
                ER no te genera los graficos que decaen. Vamos a necesitar otra cosa -> CONFIGURATION MODEL

                Idea configuration models: 
                    secuencia de grados dada 'd'

                ¿Como se genera el grafo? 
                    -> Matching Algorithm (se usa para análisis): dados nodos con semi-aristas, se sortean al azar y se crea un secuencia de aristas. Y con eso llego al grafo? Como se que es válido ese grafo? EL profe dijo algo sobre la distribucion, a que se refiere?

                    Hay ciertas restricciones para ejecutar este algoritmo

                    ->Switching Algorithm: 

                Algunos resultados:
                    1. Transicion de fase en la aparicion de una componente gigante
                    2. Clustering coefficient se va a 0
                    3. 
                    4. 
   
    Modelo Small World
        'seis grados de separación'

        ¿Cual es la distancia minima entre 2 nodos en la red? Nos vamos a preguntar -> Experimento de Milgram

        Milgram muestra que los caminos ortos existen y en abundancia
        Las redes tipicamente tienen diametro chico y triangulos

        Diapo 28y29 (Queremos combinar ambos grafos) -> Modelo Watts-Strogatz

        Controla la cantidad de aristas y tiene diametro chico (sin embargo no es lo mas eficiente)

    Modelo de Variables Latentes
        Los nodos tienen una clase asignada

        Modelos de clases latentes:
        Modelos de vectores latentes (continuo): 

        Ejmplo: Actores que comparten películas, las distintas comunidades de cine las pienso como variables latentes? 

        Stochastic Block Models (SBMs): (esto es para generar el grafo o para detectar comunidades?)
            Nodos separados en clases, y hay una matriz de probabilidades de conexión. Normalmente es mas probabe que te conectes con alguien de tu comunidad

            Diapo 36: Anotar defs.
            Diapo 37: Matriz Q, que se puede observar según el valor de Q. Heterofilio?

    RESUTALDO SOBRE APROXIMACIONES

    Extensiones de SBMs
        -Degree-corrected SBMs
        -
        -

    Random dot product graphs (RDPGs)
        Conectados: vectores alineados
        No conectados: vectores ortogonales con prod interno cero

        Conexcion a otros modelos

        Estimacion de las posiciones latentes, cuanto da X_LS??
        Calcular autovalores y me quedo con los autovectores asociados a los autovalores mas grandes (d mas significativo) -> se explica en diapo 47 (diagonalizacion de la matriz)

        de donde saco d? si el modelo esta bien se deberia observar un 'codo'

        Adjacency Spectral Embedding (ASE)


        Diapo 50: notar la ortogonalidad de los embeddings
            ->Alineacion del vecotr
            ->Magnitud del vector

        Online change point detection: quiero ver si en algun momento cambió algun puntp



    Geometry Random Graphs
        Ventajas y desventajas

        Para lograr lo que queremos: usar geometrías no euclideas

        Curvatura: recordar definicion de circunferencia para este caso

        