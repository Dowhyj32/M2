Inferencia

Objetivo: Inferir un grafo a partir de datos

¿Como usar GSP para inferir la topología del grafo?

Inferir el grafo a partir de una señal. Vamos a querer buscar el grafo que mejor explica la señal.

HAy 3 problemas: 
    1. prediccion de enlace (presentado el lunes)
    2. Association network inference (HOY)
    3. Tomographic network topology inference
   
PREDICCION DE ENLACES
    Queremos inferir las que no sabemos

Association network inference
    Tenemos señales en los nodos y no conocemos ninguna arista: queremos inferir el grafo completo

Tomographic network topology inference
    No conozco ninguna arista y tampoco tengo informacion

¿Hay o no hay una arista? ¿Con que peso?

ASSOCIATION NETWORKS
    Los vértices están unidos por aristas si hay un nivel suficiente de 'asociación' entre atrinbuots de los pares de vertices

    Matriz: Filas -> representan los nodos
            Columnas -> señal (x) a lo largo del tiempo

    ¿El grafo inferido es único? -> Depende de las condiciones 

    ¿Cuantas mediciones hay que hacer del experimento? 
    Quiero estimar aristas dado un numero de mediciones

    Cuanto mas larga sea la señal, mejor será la estimación

    Dados los nodos y la señal, se puede definir una funcion de similitud con la cual podemos ver cuanto se parecen 2 señales



    Hay que tomar ciertas decisiones:
        
        -Elección de sim:
            (Si dos nodos estan correlacionados la señales son similares)
            
            Redes de correlacion:
                Hacemos correlacion entre 2 señales tmeporales

                Se define un grafo de correlacion -> Si la correlacion es distinta de cero => ponemos una arista entre esos 2 nodos

                Association network inference <=> Inferencia de correlaciones no nulas

                ¿TEST DE HIPOTESIS? H_0 lo definis como el que querés que pase
                N**2 test de hipótesis (malo computacional) (N*(N-1)/2)

                Estadísticos para el test: (COMO HACEMOS LA CUENTA)
                    -Correlaciones empíricas
                    -Transformada de fisher => llegamos a algo que tiene una distribución normal (Simple de controlar significancia)

                Grafos y testeo múltiple (problema de testeo múltiple)

                Corrección: False Discovery Rate (FDR)
            
            Correlaciones parciales
                CORRELACIÓN NO IMPLICA CAUSALIDAD -> un par de nodos puede estar influenciado por un tercer nodo

                (Para estos problemas los grafos no tiene self-loops)

                Para ello se definen las correlaciones parciales que capturan mejor la influencia directa entre nodos

                REDES DE CORRELACIONES PARCIALES
                    Lo mismo que hicimos antes pero con CORRELACION PARCIAL

    Undirected Gausian graphical models


    Digresión: Sparsity y norma l1
        la norma 1 induce esparsidad (estimador Lasso), la ventaja de la norma 1 es que una de las coordenadas se anula

        Graphical Lasso (estimo toda la matriz junta)


    Covariance selection meets linear regression (mejorar explicación / chequear algoritmo)
        (estimo la matriz por partes usando combinaciones lineales y me fijo cuales son los nodos que mas aportan)

        (tomo primer fila, hago regresión y me quedo con los pocos más significativos)

        (¿Problemas del método?)
        (¿VENTAJAS?)
        Neighborhood-based sparse regression
            Hago las regresiones de forma independiente


        ESTOS MODELOS SON PARA VARIABLES ALEATORIAS GAUSSIANAS



APRENDIENDO GRAFOS A PARTIR DE OBSERVACIONES DE SEÑALES SUAVES


DIAPO 41: Recuerdo el ejemplo de la primer clase, a medida que avanzo la señal es menos suave
Frecuencia grande -> coeficiente chico

Ahora estamos infiriendo el grafo a partir de la matriz Laplaciana

Aprender un grafo a partir de una señal suave

Formulacion y algoritmo: cuanto importancia le doy a la suavidad y a la esparsidad

Signal smoothness meets edge sparsity (alternativa a usar la Laplaciana)

