                                            Growing Neural Gas

El growing neural gas es un algoritmo de clusterización, lo cual consiste en agrupar una base de  datos en  una serie de nodos que tengan similitudes a la base de datos,  logrando de esta forma poder sacar un patrón en un grupo de datos.

<img src=".\images\martillo.png" style="zoom: 85%;" />



En las imágenes podemos observar la nube de datos en gris,mientras los nodos están en rojo.



<img src=".\images\manzana.png" style="zoom: 85%;" />



## Explicación Algoritmo

Usaremos nodos k los cuales contiene a  un vector de posición  W (en netlogo usaremos las coordenadas de la tortuga),una lista con los vecinos de un nodo (Como netlogo permite trabajar con grafos no necesitaremos esto), una variable de error y para facilitarnos la tareas un id de nodo

Como inicio para el algoritmo generamos 2 nodos iniciales y los unimos entre si con una arista que ponderaremos con una variable llamada edad.

El bucle del algoritmo comienza generando un punto de referencia x que tendrá las coordenadas de algún dato de la nube de datos.

Después localizamos los 2 nodos más cercanos a x (s y t).

Al error de s lo modificamos así

$$
{Error}_s <- {Error}_s + {||{W}_s-x||}^2
$$
Movemos s y sus nodos conexos con la siguiente ecuación:


$$
{W}_s ← {W}_s + {e}_w (x-{W}_s)
$$

$$
{W}_n ← {W}_n + {e}_n (x-{W}_n)
$$



 Siendo n los  nodos vecinos de S  y las e variables definidas con un valor decidido por la persona.

Posteriormente incrementamos la edad de las aristas entre s y sus vecinos.

Si s y t no están conectados se conectan mediante una arista ponderada.

Ahora buscamos si existe una arista con edad mayor a a_limit, si es así se borra por lo que si al hacer eso queda un nodo desconectado él mismo es borrado.

Si la iteración es múltiplo de lambda y además el numero de nodos no es  mayor al establecido por max-node-counts realiza los siguiente:

Se busca el nodo u con el mayor error y el nodo v que es el nodo vecino a v con mayor error, insertamos entre u y v un nodo r cuyo vector de referencia sea la media posición entre u y v, sucesivamente borramos la arista entre u y v y conectamos u con r y r con v mediante aristas.

Sucesivamente reestablecemos los errores de u, v y r siguiendo la siguiente asignación.


$$
{ error}_u <- {\alpha} * { error}_u
$$

$$
{ error}_v <- {\alpha} * { error}_v
$$

$$
{ error}_r <-  { error}_u
$$

Siendo Alpha una variable que nosotros decidimos.

Y así hasta que se cumpla un límite de parada establecido.

Finalmente ajustamos todos los errores aplicando esta otra asignación.

 Con Beta otra variable ajustable.


$$
{ error}_j <- {\beta} * { error}_j
$$


# Código y su explicación

Para este trabajo nosotros nos hemos decantado por crear una librería con el código de GNG,la cual será usada en el código,el cual contiene un par de funciones que nos sirven para cargar unos ficheros de datos que previamente hemos creado con distintas formas además de otro botón que genera una nube de datos aleatoria en función de de unos parámetros establecidos por nosotros como la cantidad de puntosa crear.

El botón de run llama a una función step de la librería principal que ejecuta el algoritmo

Este es el step que equivale al algoritmo usando funciones externas creadas por nosotros.

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

## Conclusiones:

Para concluir este código me gustaría hacer algunas demostraciones sobre lo que modifica los valores de ciertos parámetros.

Lambda muy pequeño.



<img src=".\images\L2.png" style="zoom: 85%;" />



Lambda muy grande 

<img src=".\images\L1.png" style="zoom: 85%;" />

Aquí podemos comprobar que lambda es el parámetro que ajusta la velocidad de creación de nodos.

