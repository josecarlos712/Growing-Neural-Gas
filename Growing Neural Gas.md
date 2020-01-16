#                           Growing Neural Gas

El **GNG** ( Growing Neural Gas) es un algoritmo de clustering (Agrupación en español),

los algoritmos de clustering son algoritmos que agrupan vectores de datos en clústeres donde los vectores del mismo clúster son similares entre si.

<img src=".\images\CC.png" style="zoom: 45%;" />



Este es un ejemplo de clustering agrupando los vectores (Cuadrados) en clústeres, donde cada clúster posee los cuadrados mas cercanos (Mismo color).

### Explicación Algoritmo

Antes de explicar en que consiste el algoritmo explicare 2 conceptos:

* Diagrama de Voronoid: Dado un conjunto de nodos ,delimitamos el área en grupos (colores) donde cada color representa los datos con un nodo común (El mas cercano), quedando así las fronteras entre colores (Aristas de Voronoid)  los puntos que equidistan de dos o mas nodos.

<img src=".\images\vor.png" style="zoom: 30%;" />



* Triangulación de Voronoid: Conectar mediante aristas los nodos que comparten Aristas de Voronoid.

<img src=".\images\vor2.png" style="zoom: 30%;" />





Este algoritmo creara nodos que representan los clústeres de datos y aristas que indicaran la vecindad de  dichos nodos.

Cada uno de estos nodos su propio vector de referencia o posición **W** , además de un valor interno que denominaremos error.

Además estos nodos estarán unidos por aristas ponderadas cuyo peso será llamado edad de la arista.



### Pseudo Código

**Inicialización del algoritmo:** Se genera 2 nodos 

* Generamos un vector de referencia **X** sacando su posición de una Base de datos.

* Localizamos los nodos **S** y **T**, siendo estos mismos los nodos mas cercanos a **X**
* Modificamos el error del  nodo ganador  **S** mediante esta función.



$$
{Error}_s <- {Error}_s + {||{W}_s-x||}^2
$$
* Movemos **S** y sus nodos conexos con la siguiente función,siendo **N** los nodos vecinos de **S** y **e** dos variables que el usuario parametriza entre 0 y 1.


$$
{W}_s ← {W}_s + {e}_w (x-{W}_s)
$$

$$
{W}_n ← {W}_n + {e}_n (x-{W}_n)
$$

 

* Incrementamos la edad de las aristas entre **S** y sus vecinos.

* Si **S** y **T** no son conexos,se crea una arista de edad 0 entre ellos.

* Se comprueba que no existe ninguna arista con una edad superior a la marcada por el parámetro **A_max_limit**, si se diera ese caso se eliminaría esa arista, comprobando que no si al hacerlo se queda un nodo sin conectar,ese nodo será borrado.

* Si la iteración es múltiplo del parámetro **Lambda**  y además el numero de nodos no supera a **max-node-counts**  se busca el nodo **U** con el mayor error y el nodo **V** que es el nodo vecino a **U** con mayor error, insertamos un nuevo nodo **R** cuyo vector de referencia sea la media posición entre **U** y **V** , sucesivamente borramos la arista entre **U** y **V**  y conectamos **U** con **R**  y **R**  con **V**  mediante aristas.

* Modificamos los errores de **U**,  **V** y **R** siguiendo la siguiente asignación con un parámetro Alpha que nosotros escogemos.


$$
{ error}_u <- {\alpha} * { error}_u
$$

$$
{ error}_v <- {\alpha} * { error}_v
$$

$$
{ error}_r <-  { error}_u
$$





* Ajustamos los errores de los nodos j con el parámetro Beta




$$
{ error}_j <--    { error}_j - {\beta} * { error}_j
$$


### Distribución y formato del código en netlogo

Para este trabajo nosotros nos hemos decantado por crear una librería con el código de GNG,con  2 to do principales (setup y step).

* Este es el  setup que inicializa el paso 0 del algoritmo.

```
to GNG:setup
;  ask patches [set pcolor white]
  clear-turtles
  reset-ticks
;  resize-world 0 100 0 100
;  set-patch-size 5
  set node-count 0
;  
;  set ew 0.05 
;  set en 0.0006
;  ;------ Creacion de la nube de puntos -------
;  ; (Crear un metodo que haga esto para una lista de puntos en un csv
;  repeat 50[ ask one-of patches with [pcolor = white and pxcor > 40 and pxcor < 60 and pycor > 40 and pycor < 60] [set pcolor black] ]
;  ;-------- Creacion de la grid y los links introducidos mediante el csv -------------------
  set x-node [0 0]
  repeat 2 [ GNG:crear-nodo (list (random max-pxcor) (random max-pycor)) ]
  create-turtles 1 [
    set xcor 0
    ;print xcor
    set ycor 0
    set color red
    set shape "circle"
    set size 2
    set label "x"
  ]
  set x-node turtles with [label = "x"]
  GNG:edge-to 0 1
  ; Utilizar el metodo /GNG:edge-to n1 n2/
end

 
```

```

to GNG:setup
;  ask patches [set pcolor white]
  clear-turtles
  reset-ticks
;  resize-world 0 100 0 100
;  set-patch-size 5
  set node-count 0
;  
;  set ew 0.05 
;  set en 0.0006
;  ;------ Creacion de la nube de puntos -------
;  ; (Crear un metodo que haga esto para una lista de puntos en un csv
;  repeat 50[ ask one-of patches with [pcolor = white and pxcor > 40 and pxcor < 60 and pycor > 40 and pycor < 60] [set pcolor black] ]
;  ;-------- Creacion de la grid y los links introducidos mediante el csv -------------------
  set x-node [0 0]
  repeat 2 [ GNG:crear-nodo (list (random max-pxcor) (random max-pycor)) ]
  create-turtles 1 [
    set xcor 0
    ;print xcor
    set ycor 0
    set color red
    set shape "circle"
    set size 2
    set label "x"
  ]
  set x-node turtles with [label = "x"]
  GNG:edge-to 0 1
  ; Utilizar el metodo /GNG:edge-to n1 n2/
end
```



* Este es el step que equivale al algoritmo previamente explicado usando funciones externas creadas por nosotros.

~~~netlogo
``` [netlogo
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
end
```
~~~

Todo este código es llamado en nuestro método principal, la cual tiene 2 métodos de creación de bases de datos.

* Generación aleatoria

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

* Mediante carga de un fichero externo con las coordenadas de los datos en formato **CSV** aprovechando la extensión homónima:

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

  

Finalmente colocamos un botón continuo que llame al step de la librería para correr el algoritmo.





### Juego de Parámetros

Nos gustaría finalizar este trabajo hablando sobre los efectos que tienen los parámetros sobre el algoritmo

* **Lambda**

  Como podemos observar en las graficas, lambda es un regulador de velocidad de creación de nodos. 

  Ya que solo se puede crear nodos cuando el contador del algoritmo sea múltiplo, si se ponen un numero muy grande tiene que pasar mucho rato entre la creación de nuevos nodos.

  Lambda muy pequeño.



<img src=".\images\L2.png" style="zoom: 85%;" />



Lambda muy grande 

<img src=".\images\L1.png" style="zoom: 85%;" />

* **Alpha** y **Beta**

  Ambos son parámetros que deciden la velocidad de decremento de los errores,y al ejecutarlo modificado sobre nuestros ejemplo no podemos apreciar ningún cambio significativo.

* **A_limit**

Esto limita  la edad que pueden alcanzar las aristas,puede que otros modelos se aprecie,sin embargo nosotros con nuestros ejemplos de prueba no notamos diferencia alguna.

* **E w**

  Al incrementar este parámetro podemos observar como los nodos se mueven mucho mas ya que este parámetro es el que indica cuanto han de moverse los nodos seleccionados.

  ### Ejemplo finales

  Ejemplo aleatorio

  <img src=".\images\C3.png" style="zoom: 85%;" />

  Manzana:

  

  <img src=".\images\manzana.png" style="zoom: 85%;" />

  Martillo:

  

  <img src=".\images\martillo.png" style="zoom: 85%;" />

  Cometa:

  <img src=".\images\C1.png" style="zoom: 85%;" />

  Escudo:

  <img src=".\images\C2.png" style="zoom: 85%;" />