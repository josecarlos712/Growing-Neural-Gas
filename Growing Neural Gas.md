#                           Growing Neural Gas

El **GNG** ( Growing Neural Gas) es un algoritmo de clustering,

Un algoritmo de clustering puede ser descrito como un proceso de organización de una colección de vectores k-dimensionales en grupos cuyos miembros comparten características.

El principal objetivo del clustering es reducir una gran cantidad de datos en bruto, en categorías con un numero reducido de componentes similares.

Los algoritmos de clustering mas conocidos para tratar este tipo de problemas son K-medias, SOM y GNG

En algunos casos no existe información suficiente para determinar a priori el numero de categorías se necesarias para agrupar la colección de vectores.

<img src=".\images\CC.png" style="zoom: 45%;" />



Esto es un ejemplo de clustering agrupando los vectores (Cuadrados) en clústers, donde cada uno de ellos posee los cuadrados más cercanos (Mismo color).

GNG es un algoritmo que tiene parámetros constantes desde el principio y no necesita decidir un numero de categorías a priori.

El problema de determinar cuando GNG debe parar es ampliamente discutido.

### Introducción al algoritmo

Antes de profundizar en el algoritmo explicaremos 2 conceptos :

* Diagrama de Voronoid: Dado un conjunto de nodos, delimitamos el área en grupos (colores) donde cada color representa los datos con un nodo común (El mas cercano), quedando así las fronteras entre colores (aristas de Voronoid)  los puntos que equidistan de dos o mas nodos.

<img src=".\images\vor.png" style="zoom: 30%;" />



* Triangulación de Voronoid: Conectar mediante aristas los nodos que comparten aristas de Voronoid.

<img src=".\images\vor2.png" style="zoom: 30%;" />

Una vez aclarados estos conceptos, que nos ayudaran a entender el algoritmo GNG, procederemos con la introducción del mismo.

GNG es un algoritmo adaptativo respecto al cambio lento de la distribución a lo largo del tiempo.

Comenzando con 2 nodos adyacentes, el algoritmo construye un grafo, en el cual los nodos considerados vecinos (aquellos que comparten similitudes) van unidos por una arista.

GNG utiliza parámetros constantes en el tiempo, por lo que no es necesario decidir un numero de nodos a priori, pues los nodos son añadidos durante la ejecución.

La creación de nuevos nodos, cesa cuando un criterio definido por el usuario así lo decide o el tamaño de la red ha alcanzado su limite.

Cada uno de estos nodos tiene su propio vector de referencia o posición **W** , además de un valor interno que denominaremos error.

Además estos nodos estarán unidos por aristas ponderadas cuyo peso será definido como la edad de la arista.

### Pseudo Código

**Inicialización del algoritmo:** Se generan 2 nodos adyacentes, con vector de posición aleatorio.

* Generamos un vector de referencia **X** seleccionando arbitrariamente uno de los vectores almacenados en la base de datos, que se mantendrá constante durante una iteración del algoritmo.

* Localizamos los nodos **S** y **T**, siendo estos mismos el primer y segundo nodos mas cercano a **X** respectivamente.
* Actualizamos el error del nodo **S** adicionando la distancia al cuadrado entre **S** y el vector de referencia **X**.

$$
{Error}_s ← {Error}_s + {||{W}_s-x||}^2
$$
* Movemos **S** y sus nodos vecinos mediante la siguiente función, siendo **n** los nodos vecinos de **S** y **ew** y **en** dos parámetros que el usuario define entre 0 y 1.


$$
{W}_s ← {W}_s + {e}_w (x-{W}_s)
$$

$$
{W}_n ← {W}_n + {e}_n (x-{W}_n)
$$

 

* Incrementamos la edad de todas las aristas que unen a **S**  con sus vecinos.
* Si **S** y **T** no son conexos, se crea una arista de edad 0 entre ellos.
* Se comprueba que no existe ninguna arista con una edad superior a la marcada por el parámetro **A_max_limit**, si se diera este caso, se eliminaría dicha arista, comprobando que si al hacerlo algún nodo queda aislado, este será eliminado.
* Si la iteración es múltiplo del parámetro **Lambda**  y además el numero de nodos no supera a **max-node-counts**,  se busca un nodo **U** que será el de mayor error y el nodo **V**  de mayor error, vecino a **U**. Insertamos un nuevo nodo **R** cuyo vector de referencia sea el mínimo equidistante a **U** y **V** , sucesivamente :
  * Eliminamos la arista entre **U** y **V**.
  * Conectamos **U** con **R**  y **R**  con **V**.
* Modificamos los errores de **U**,  **V** y **R** siguiendo la siguiente asignación mediante un parámetro Alpha predefinido.


$$
{ error}_u ← {\alpha} * { error}_u
$$

$$
{ error}_v ← {\alpha} * { error}_v
$$

$$
{ error}_r ← { error}_u
$$





* Ajustamos los errores de los nodos inalterados anteriormente mediante el parámetro Beta, como se muestra a continuación.




$$
{ error}_j ←   { error}_j - {\beta} * { error}_j
$$


### Distribución y formato del código en NetLogo

Para este trabajo nos hemos decantado por crear una librería con el código de GNG.

* El siguiente método, realiza la preparación para la ejecución de la primera iteración del algoritmo.
  * Limpia los patches, resetea los ticks.
  * Se crean los dos nodos iniciales y se interconectan.

```
to GNG:setup
  clear-turtles
  reset-ticks
  set node-count 0
  set x-node [0 0]
  repeat 2 [ GNG:crear-nodo (list (random max-pxcor) (random max-pycor)) ]
  create-turtles 1 [
    set xcor 0
    set ycor 0
    set color red
    set shape "circle"
    set size 2
    set label "x"
  ]
  set x-node turtles with [label = "x"]
  GNG:edge-to 0 1
end

 
```

* El siguiente método se iterará asiduamente siguiendo el pseudo-código expuesto anteriormente.

~~~netlogo
to GNG:step
  ; Se establece el vector de entrada
  if (node-count - 1) >= max-node-count [ stop ]
  let nuevo-x one-of data with[color = grey]
  ask x-node [
    set xcor [pxcor] of nuevo-x
    set ycor [pycor] of nuevo-x
  ]
  ; Se obtienen los nodos s y t mas cercanos al nodo 0
  let ws GNG:nearest
  let wt GNG:nearest-except ws
  ; Se actualiza el error del nodo s
  let newerror (GNG:distancia x-node GNG:nodes with [node-id = ws]) ^ 2
  ask GNG:nodes with [node-id = ws] [set node-error node-error + newerror]
  ; Se mueve s y los vecinos dependiendo de la distancia a x-node
  GNG:move-node ws
  ; Se actualiza la edad de los links de s
  ; Los links cuya edad supere alimit se borran
  ask GNG:nodes with [node-id = ws] [
    ; Si existe el link entre s y t, su edad se resetea
    ;print (word "me: " one-of GNG:nodes with [node-id = ws] " elem: " one-of GNG:nodes with [node-id = wt] " lista" [node-id] of GNG:edge-neighbors)
    ifelse member? wt [node-id] of GNG:edge-neighbors [
      ;show (word "arista " (GNG:edge ws wt))
      ask GNG:edge GNG:who ws GNG:who wt [set edge-age 0]
    ]
    ; Si no existe el link entre s y t se crea uno 
    [ GNG:edge-to ws wt ]
    
    ask my-GNG:edges [
      set edge-age edge-age + 1
      ; si la edad del link es mayor a alimit, este muere
      if edge-age >= alimit [
        die 
      ]
    ]
  ]
  ; Si algun nodo se ha quedado sin links, muere
  ask GNG:nodes with [count GNG:edge-neighbors = 0] [ die ]
  ; Si la iteracion actual es multiplo de lambda, se crea un nuevo nodo r
  ; Sea u el nodo con mayor error
  let nlist sort GNG:nodes
  let errors map [n -> [node-error] of n] nlist
  let u one-of GNG:nodes with [node-error = max errors]
  ; Sea v el vecino de u con mayor error
  ask u [ set nlist sort GNG:edge-neighbors ]
  set errors map [n -> [node-error] of n] nlist
  let v 0
  ask u [ set v one-of GNG:edge-neighbors with [node-error = max errors] ]
  ; Se crea r en el punto medio de u y v si no se ha alcanzado el limite de nodos
  if (ticks mod lambda = 0) and node-count < max-node-count [
    let posu GNG:check (list [xcor] of u [ycor] of u)
    let posv GNG:check (list [xcor] of v [ycor] of v)
    let newr (list round ((first posu + first posv) / 2) ;xcor
      round ((last posu + last posv) / 2)) ;ycor ; v + u / 2
    GNG:crear-nodo newr ; se crea el nodo r
    let r one-of GNG:nodes with [xcor = first newr and ycor = last newr]  ; almacenar en r el nodo r
    GNG:edge-to [node-id] of u [node-id] of r ; se crea link u r
    GNG:edge-to [node-id] of v [node-id] of r ; se crea link v r
    ask GNG:edge GNG:who [node-id] of u GNG:who [node-id] of v [ die ] ; se elimina link u v
    ; Se aplican nuevos errores para u, v y r
    ask u [ set node-error alpha * node-error]
    ask v [ set node-error alpha * node-error]
    ask r [ set node-error [node-error] of u]
    ; se decrementa el error de todos los nodos por un factor beta
    ask GNG:nodes [ set node-error node-error - beta * node-error]
  ]
  tick
~~~

Los siguientes métodos son los utilizados para la importación y representación de los datos en el espacio vectorial. Han sido creados para realizar las pruebas de nuestra librería. 

Todo usuario que desee dar uso a la librería del algoritmo GNG deberá reimplementar cada método de importación y representación del espacio vectorial según el fin de su propósito.

* Generación aleatoria de n datos distribuidos en bolsas por el espacio.

  ```
  to setup-data2 [n]
    ca
    GNG:setup
    ask patches [set pcolor white]
    let bags 1 + random dispersion
    create-data bags [
      setxy random-xcor random-ycor
      set size 1
      set shape "dot"
      set color grey ]
    repeat n - bags
    [
      ask one-of data
      [hatch-data 1 [set heading random 360 fd random-float 5]]]
  end
  ```

* Mediante la carga de un fichero externo con las coordenadas de los datos en formato **CSV** aprovechando la extensión homónima:

  ```
  to-report load [f]
    ; Read dataset
    let files csv:from-file f
    set files remove-duplicates files
    ;Normalize and transform output
    set files shuffle bf files
    ;set files remove-item 0 files
    report files
  end
  
  to setup-file
    ca
    GNG:setup
    ask patches [set pcolor white]
    foreach load word "Ejemplos\\" Ejemplo  [x ->
      create-data 1 [
      setxy (item 0 x) (item 1 x)
      set size 1
      set shape "dot"
      set node-error 0.0
      set color grey ]
    ]
  
  ```

  

Finalmente colocamos un botón reiterante que llame a la función step de la librería para ejecutar el algoritmo.



### Juego de Parámetros

Nos gustaría finalizar este trabajo hablando sobre los efectos que tienen los parámetros sobre el algoritmo

* **Lambda**

  Como podemos observar en las graficas, lambda es un regulador de velocidad en la creación de nodos. 

  El algoritmo tiene un contador interno de ticks (pulsos), se crea un nodo nuevo cada vez que dicho contador sea múltiplo de lambda, por lo que si se pone un numero muy grande el tiempo entre la creación de nuevos nodos aumenta.

  Lambda pequeño.



<img src=".\images\L2.png" style="zoom: 85%;" />



Lambda grande

<img src=".\images\L1.png" style="zoom: 85%;" />

* **Alpha** y **Beta**

  Ambos son parámetros usados para modificar la velocidad de decremento de los errores de los nodos.

  En nuestros ejemplos no se aprecia diferencia alguna en la comparativa de valores que hemos evaluado.

* **A_limit**

  Este parámetro limita la edad que pueden alcanzar las aristas, puede que en otros modelos mas complejos se aprecie, sin embargo nuestros ejemplos de prueba no consiguen magnificar cambios en el clustering apreciables.

* **E w**

  Es un factor multiplicativo que va ha determinar los saltos de los nodos.

  Al incrementar este parámetro podemos observar que el movimiento de los nodos es tan errante que no consiguen converger en una solución optima.

  ### Pruebas realizadas

  Ejemplo aleatorio:

  <img src=".\images\C3.png" style="zoom: 85%;" />

  Manzana:

  

  <img src=".\images\manzana.png" style="zoom: 85%;" />

  Martillo:

  

  <img src=".\images\martillo.png" style="zoom: 85%;" />

  Cometa:

  <img src=".\images\C1.png" style="zoom: 85%;" />

  Escudo:

  <img src=".\images\C2.png" style="zoom: 85%;" />