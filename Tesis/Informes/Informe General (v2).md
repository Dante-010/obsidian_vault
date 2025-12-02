## Proyecto original
Este proyecto de tesis comenzó en base al paper [[Evading Community Detection via Counterfactual Neighborhood Search.pdf|Evading Community Detection via Counterfactual Neighborhood Search]].

El paper describe un método que dado un grafo, un algoritmo de detección de comunidades y un nodo objetivo, permite "ocultar" el nodo objetivo del algoritmo de detección de comunidades, en el sentido de cambiar la estructura de comunidades para dejar de ser identificado como parte de la comunidad original.

Esto se logra utilizando un agente entrenado con reinforcement learning, utilizando un método [[Algoritmos Actor-Critic|Actor-Critic]] (específicamente, [[Referencias#(26) Advantage Actor-Critic (A2C) |A2C, o Advantage Actor Critic]]).
Se utiliza este método puesto que por cómo definen la función de pérdida, obtienen una fórmula no diferenciable, lo que implica que no pueden aplicar métodos más tradicionales como gradient descent.

Los autores llaman a este proceso **Community Memerbship Hiding**.

El problema original tiene las siguientes restricciones:
- Se necesita información completa sobre el grafo.
- Se oculta (es decir, se cambia de comunidad) un único nodo objetivo $u$.
- Las comunidades no se solapan.
- El algoritmo de detección se considera *black-box*.
- Solo se pueden **eliminar** ejes salientes desde $u$ a nodos $v$ **dentro** de su comunidad, y solo se pueden **agregar** ejes salientes desde $u$ a nodos $v$ **fuera** de su comunidad (se asume que el grafo es dirigido, pero este razonamiento se puede extender a grafos no dirigidos).

*Como nota de esto último, en teoría solo permitir estas acciones se alinea más con el objetivo de ocultar el nodo, pero podría ser que limitar al agente y no permitirle aprender por su cuenta sea contraproductivo
Además, en un caso real es muy raro que $u$ tenga conocimiento total de su comunidad y/o del grafo..*

La noción de "ocultar" un nodo de una comunidad no es algo que esté formalmente definido de antemano, y en el paper se considera que un nodo está oculto o separado de su comunidad original cuando vale la siguiente ecuación:
$$
sim(\mathcal{C}_i - u, \mathcal{C}_i' - u) \leq \tau
$$
donde:
- $\mathcal{C}_i$ es la comunidad original de $u$ (en el grafo original $\mathcal{G}$).
- $\mathcal{C}_i'$ es la comunidad asignada a $u$ en el grafo modificado por el agente (en el grafo final $\mathcal{G}'$).
- $\tau \in [0, 1]$ es un *threshold* u valor objetivo que determina cuándo se comienza a considerar a $u$ como ocultado. Un valor de $1$ implica que ambas comunidades pueden ser idénticas, y un valor de $0$ requiere que sean completamente distintas.
- $sim(\cdot, \cdot)$ es una función de similaridad entre dos comunidades. Se pueden utilizar varias definiciones, pero en este caso los autores utilizaron el [[Glosario#Conceptos introducidos en las referencias|Coeficiente Sørensen-Dice]], que en pocas palabras determina qué tantos nodos comparten ambas comunidades.

Con respecto al agente, utilizan la siguiente arquitectura de red:
![Model Architecture](Tesis/READMES/images/model_architecture_background.png)
[[node2vec]] es un algoritmo que genera embeddings de cada nodo a partir del grafo, con un proceso similar a **word2vec**, donde se genera un vector para cada nodo que resume su "signficado" o propiedades y posición dentro del grafo.

En pocas palabras, node2vec realiza caminatas aleatorias de segundo orden (tiene en cuenta el estado actual y el anterior) para generar oraciones, con los nodos como palabras, que luego son pasadas a un modelo skip-gram con negative sampling que genera los embeddings. Según la elección de hiperparámetros, los embeddings representan  **homofilia** (nodos altamente interconectados y que pertenecen a clusters similares deben ser embebidos cerca) o **equivalencia estructural** (nodos con roles estructurales similares en las redes deben ser embebidos cerca, sin hacer énfasis en su conectividad "real").

Sin embargo, tengo la **hipótesis** de que estos embeddings pueden ser contraproducentes, ya que se calculan una única vez en el grafo original y no se vuelven a computar, y a medida que el grafo cambia van dejando de representar la realidad. 
Además, también se podría argumentar que es un estilo de "trampa" puesto que le dice qué nodos ocultar en base a sus características (*information leakage)*,  en vez de dejar que el agente aprenda por sí solo por qué y cuáles nodos ocultar.

Es importante entender la idea detrás de tener un actor y un crítico, en vez de únicamente un actor. 
- El **actor** es quien decide la acción a tomar, aprende una política $\pi(a | s)$, una distribución de probabilidad sobre acciones dado el estado actual. Notar que esta política no es "fija", en el sentido que es el resultado de una red neuronal, y no una serie de reglas determinísticas.
- El **crítico** "evalúa las acciones", y aprende una función de valor $V(s)$, que nos dice "qué tan bueno" es el estado actual bajo la política actual (devuelve un escalar).

Si se entrena únicamente el actor, su gradiente presenta mucho ruido:
$$
\nabla_{\theta} J(\theta) = \mathbb{E}_{s,a} \left[ \nabla_{\theta} \log \pi_{\theta}(a \mid s) \, (R_t - b) \right]
$$
Donde:

- $( R_t )$ = Recompensa total (suma de recompensas futuras)  
- $( b )$ = baseline para reducir la varianza.  

Sin una baseline "buena", la gradiente puede tener mucha varianza, lo que desestabiliza y ralentiza el entrenamiento.

El trabajo del crítico es estimar $b \approx V(s)$, la **recompensa esperada** desde el estado actual.
En vez de utilizar las recompensas totales (que varían considerablemente), utilizamos la **advantage**:
$$
A_t = R_t - V(s_t)
$$

Planteamos una serie de [[Alternativas del problema original |Alternativas]] al problema original, siendo las principales:
- Trabajar con comunidades solapadas.
- Trabajar con información parcial/incompleta.
- Trabajar con algoritmos "white-box" (es decir, ver si podemos aprovechar ciertos comportamientos del algoritmo de detección si sabemos cómo funciona).
- Darles features a los nodos, y/o pesos a los ejes.
- Considerar como objetivo que $u$ deje de compartir comunidad con un conjunto selecto de nodos.
- Implementar Multi Agent Reinforcement Learning (MARL), donde en vez de tener un único agente tenemos varios agentes (cada uno "representando" un nodo) trabajando en conjunto.

Esta es la base del proyecto original.

-----
# Taxonomía de Problemas de Evasión en Grafos
##### Expandiendo el Espacio de Problemas y Analizando Escenarios Alternativos

Discutiendo con Esteban y Gabriel, concluimos que es más interesante o efectivo (en el sentido de obtener resultados) dar un paso atrás y considerar diversos problemas sobre grafos, de ahí el título de la tesis.
La idea es tomar como base el código original, específicamente el agente de Reinforcement Learning, y trabajar sobre problemas más generales sobre grafos.

Esteban escribió una taxonomía muy útil que habla sobre todas las diferentes maneras de variar el problema, o de encontrar nuevos problemas/objetivos.
Gabriel mencionó el nuevo paper escrito por los autores originales recientemente, [[The Right to Hide]], que trata el mismo problema de Community Membership Hiding, pero resuelto con un método más tradicional (definen una función de pérdida diferenciable, lo que les permite aplicar un método de descenso por gradiente adaptado al problema). 
Este paper es sumamente útil, puesto que nos da un nuevo benchmark para comparar nuestro agente, y el hecho de que los autores sigan trabajando con este problema nos da pie a interactuar con ellos en el caso de que surjan dudas o problemas.
### Roadmap
Comencé a trabajar en la tesis propiamente hablando en Septiembre de 2025. El primer mes estuve familiarizándome con el problema, los términos utilizados, y el código creado por los autores originales. Hice énfasis especial en leer todo el código y entender bien a "bajo nivel" lo que está pasando cuando entrenamos y testeamos el agente.

Lo primero que hice fue tratar de replicar el paper. Sorpresivamente, ejecutando el código tal cuál se encontraba en el repositorio, no lograba obtener los resultados presentados. 
Analizando el código, el primer "problema" o controversia fue el uso de una estrategia de selección de nodos a testear que elegía comunidades que cumplían ciertas restricciones de tamaño (este problema y otros los vemos en detalle en [[#Problemas con el modelo original]])

Esto llevó a enviarles un correo a los autores, que nos aclararon que para entrenar era razonable seleccionar los nodos de forma aleatoria, y que al testear decidieron elegir un subconjunto de comunidades de distintos tamaños para tener un sampleo más general y menos sesgado.
De igual manera, al volver a correr el código con estos cambios, tampoco logré obtener los resultados originales (el proceso más detallado se puede ver en [[Experimentos]]).

*Como nota menor, hice algunos cambios menores para mejorar la "Quality of Life": el modelo agrega los hiperparámetros a los checkpoints para no tener que setearlos a mano cuando queremos testear, se loggean los tiempos de testeo y entrenamiento, se setean todas las semillas correspondientes de los generadores aleatorios de números para poder obtener resultados verdaderamente reproducibles, y demás cambios en el código para entenderlo y modificarlo de forma más sencilla*.
*Además, cree una herramienta  para visualizar los grafos antes y después de las modificaciones:*
![[Screenshot_20251110_143436.png]]

Con el código ya funcionando (porque aunque no logré replicar los resultados del paper, sí obtenía lo mismo que figuraba en el repositorio), pusimos una serie de objetivos sencillos para comenzar a variar el problema:
- Ver qué sucede si solo permitimos eliminar ejes.
- Ver qué sucede si solo permitimos agregar ejes.
- Redefinir el problema para unir dos comunidades $\mathcal{C}_1, \mathcal{C}_2$.

Plantear los primeros dos problemas es bastante sencillo, puesto que ya existe la funcionalidad para restringir las acciones del agente.

Los experimentos muestran que el modelo original siempre obtiene un resultado mayor en comparación a sus versiones restringidas. 
El modelo de solo agregar ejes funciona un poco mejor que el de solo eliminar (esto puede explicarse en base a que el modelo original, en promedio, agrega un eje en el $80\%$ de los casos, frente a un $20\%$ de eliminación).
Sin embargo, el modelo que únicamente elimina ejes obtiene un rendimiento $\approx6\%$ menor realizando prácticamente la mitad de las operaciones.

De igual manera, los experimentos realizados sobre estas variantes fueron pocos, y sobre grafos "pequeños". Lo que sucedió es que al querer plantear la tercer variante, me percaté de una incongruencia en el código, y dejé de lado estos experimentos por el momento (ver [[#Cambio estructural de la red neuronal]]).
##### Método de entrenamiento/testing original
- Al **entrenar** el agente, se selecciona un grafo y un algoritmo de detección, y en cada episodio hay una probabilidad $\epsilon$ de cambiar el nodo objetivo, según una estrategia arbitraria (en general, se elige un nodo de forma aleatoria). 
  En cada episodio, el agente modifica el grafo hasta ocultar el nodo (según $sim(\cdot, \cdot)$), o quedarse sin presupuesto.
  Finalmente calcula el *loss*, se aplica backpropagation, y se repite el proceso.
- Al **testear** se eligen varios grafos y algoritmos de detección, y sobre cada uno de ellos se corren un cierto número de tareas de ocultamiento, variando nodo y/o comunidades. Los resultados se agrupan por grafo y algoritmo.
- No existe la noción de *train, test, validation* sets, sino que se trabaja con grafos completos.

*Nota: Cuando hablo del "entorno de simulación" me refiero a la parte del código (específicamente el archivo `graph_env.py` que se encarga de cargar el grafo, guardar su estado, realizar las modificaciones, calcular las funciones de pérdida y recompensa, seleccionar el nodo objetivo, determinar las acciones posibles, y todo lo demás relacionado al grafo.
El código del agente que se encarga de recibir el grafo, pasárselo a la red neuronal, obtener los resultados, realizar backpropagation, etc., está en el archivo `agent.py` (la red neuronal está definida en `a2c.py`, `actor.py`, `critic.py`).*

Para redefinir el problema, hay que cambiar las funciones de pérdida, las acciones permitidas, el entorno de simulación, la entrada y salida de la red neuronal, etc...
Por lo tanto, modularizé el código para en el futuro tener una [["Fábrica" de experimentos]] que nos permita plantear nuevos problemas de forma "rápida y sencilla" (y evitar el planteamiento *ad hoc* presente en el código original).
#### Cambio estructural de la red neuronal
Al analizar el código con mayor profunidad, me di cuenta de que la red neuronal no estaba teniendo en cuenta el nodo objetivo, sino que recibía como entrada el grafo completo, y era el entorno de simulación el que se encargaba de "adaptar" el output de la red para actuar sobre el nodo objetivo.
 
Lo importante de esto es que la red **no sabe** quién es el nodo objetivo, por lo que resulta imposible que sus acciones dependan directamente de esta información (por ejemplo, tener más en cuenta la vecindad del nodo), sino que únicamente tienen raíz en la estructura general del grafo. 

Para verificar esto, volví a bajar el código original y verifiqué los outputs del actor y el crítico para el primer paso del *rewiring*, con semillas fijas, cambiando el nodo objetivo en cada iteración.

*Notas: la red neuronal original recibe como entrada un grafo y devuelve un conjunto de probabilidades sobre los nodos del grafo y un escalar, que indican respectivamente la probabilidad de que agregar o sacar un eje entre el nodo $v_i$ y el nodo objetivo $u$ aumente la recompensa esperada (es decir, sobre qué nodo es mejor actuar) y la recompensa esperada dados el grafo y la política actual.*
*El proceso de rewiring es simplemente tomar el output del actor y agregar un eje entre el mejor nodo $v$ y el nodo objetivo $u$ si no existe, o eliminarlo si ya existe. 
Al testear, no es necesario utilizar la red neuronal del crítico, únicamente se utiliza la red del actor.*

Independientemente del nodo seleccionado, las probabildades del actor y el escalar del crítico eran iguales, lo que implica que no tienen en cuenta al nodo objetivo al realizar las estimaciones.
Además, al enviar un correo a los autores preguntando sobre esto, recibimos una confirmación de que efectivamente la red neuronal se comporta de este manera.

El agente original pierde mucho rendimiento con grafos grandes, probablemente debido a que no actúa en basa al nodo objetivo. Como **hipótesis**, el agente nuevo debería tener un rendimiento similar sin importar el tamaño del grafo, o como mínimo obtener un rendimiento mayor al original.

*Notas: volviendo a lo que decía antes, la idea es desarrollar un agente robusto y bien pensado, que sirva como base para resolver distintos problemas sobre grafos, y no quedarnos fijados en el problema de community membership hiding.
Seguramente cada problema lleva a utilizar distintas estructuras de red y selección de hiperparámetros, pero la idea sería poder cambiar únicamente las funciones de pérdida y las acciones permitidas y obtener un modelo que resuelva un problema completamente distinto, manteniendo la estructura de la red neuronal lo más similar posible.*

-----
## Estado actual
Dada la respuesta positiva de Matteo, decidimos continuar por el camino de agregar el nodo objetivo como entrada adicional a la red neuronal.

El próximo objetivo es hacer un modelo que "funcione", en el sentido que tiene en cuenta el nodo objetivo y obtiene un rendimiento como mínimo igual al anterior, entendiendo el flujo de los datos y el porqué del funcionamiento.

Logré hacer un modelo "trivial" que aprendió a elliminar todos los ejes para aislar el nodo, obteniendo un *success rate* excelente, pero a costa de la NMI (*normalized mutual information*). Como la métrica objetivo del paper es el F1 score entre estos dos valores, no lo puedo considerar un modelo exitoso, pero sirve como *proof of concept*.

Como consejo de Esteban y Gabriel, en vez de utilizar node2vec para las features, utilizamos features más comunes al trabajar con grafos:
- **Degree.**
- **Betweenness centrality.**
- **PageRank**.
- ...

Luego podemos aplicar un análisis estilo **PCA** (*Principal Component Analysis)* para determinar cuáles son las más importantes, y obtener una especie de explicación sobre la solución del problema.

Además, leyendo bibliografía y papers sobre el tema descubrí varias técnicas y optimizaciones para trabajar sobre grafos que vale la pena aplicar. Recordemos que el concepto de las **Graph Neural Networks (GNN)** es algo relativamente nuevo, apareciendo las primeras **Graph Convolutional Netwokrs (GCN)** recién en 2017, por lo que todavía se siguen descubriendo nuevas estrategias para mejorar los modelos, o se inventan nuevas arquitecturas de red.


