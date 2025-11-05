## Proyecto original
Este proyecto de tesis comenzó en base al paper [[Evading Community Detection via Counterfactual Neighborhood Search.pdf|Evading Community Detection via Counterfactual Neighborhood Search]].

El paper describe un método para dados un grafo, un algoritmo de detección de comunidades y un nodo objetivo, "ocultar" el nodo del algoritmo de detección de comunidades, en el sentido de cambiar la estructura de comunidades para dejar de ser identificado como parte de la comunidad original.

Esto se logra utilizando un agente de inteligencia artificial, entrenado con Reinforcement Learning y utilizando un método [[Algoritmos Actor-Critic|Actor-Critic]] (específicamente, [[Referencias#(26) Advantage Actor-Critic (A2C) |A2C, o Advantage Actor Critic]]).
Los autores llaman a este proceso **Community Memerbship Hiding**.

El problema original tiene las siguientes restricciones:
- Se necesita información completa sobre el grafo.
- Se oculta un único nodo objetivo $u$.
- Las comunidades no se solapan.
- El algoritmo de detección se considera "black-box".
- Solo se pueden **eliminar** ejes salientes desde $u$ a nodos $v$ **dentro** de su comunidad, y solo se pueden **agregar** ejes salientes desde $u$ a nodos $v$ **fuera** de su comunidad (se asume que el grafo es dirigido, pero este razonamiento se puede extender a grafos no dirigidos).

*Como nota de esto último, en teoría solo permitir estas acciones se alinea más con el objetivo de ocultar el nodo, pero podría ser que limitar al agente y no permitirle aprender por su cuenta sea contraproductivo*.

La noción de "ocultar" un nodo de una comunidad no es algo que esté formalmente definido de antemano, y en el paper se considera que un nodo está oculto o separado de su comunidad original cuando vale la siguiente ecuación:
$$
sim(\mathcal{C}_i - u, \mathcal{C}_i' - u) \leq \tau
$$
donde:
- $\mathcal{C}_i$ es la comunidad original de $u$ (en el grafo original $\mathcal{G}$).
- $\mathcal{C}_i'$ es la comunidad asignada a $u$ en el grafo modificado por el agente (en el grafo final $\mathcal{G}'$).
- $\tau \in [0, 1]$ es un *threshold* u valor objetivo que determina cuándo se comienza a considerar a $u$ como ocultado. Un valor de $1$ implica que ambas comunidades pueden ser idénticas, y un valor de $0$ requiere que sean completamente distintas.
- $sim(\cdot, \cdot)$ es una función de similaridad entre dos comunidades. Se pueden utilizar varias definiciones, pero en este caso los autores utilizaron el [[Glosario#Conceptos introducidos en las referencias|Coeficiente Sørensen-Dice]], que en pocas palabras determina que tantos nodos comparten ambas comunidades.

Con respecto al agente, utilizan la siguiente arquitectura de red:
![Model Architecture](Tesis/READMES/images/model_architecture_background.png)
[[node2vec]] es un algoritmo que genera embeddings de cada nodo a partir del grafo, con un proceso similar a **word2vec**, donde se genera un vector para cada nodo que resume su "signficado" o propiedades y posición dentro del grafo.

Planteamos una serie de [[Alternativas del problema original |Alternativas]], siendo las principales:
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

Discutiendo con Esteban y Gabriel, concluimos que es más "interesante" o efectivo dar un paso atrás y considerar problemas generales sobre grafos, de ahí el nombre de la tesis.
La idea es tomar como base el código original, específicamente el agente de Reinforcement Learning, y trabajar sobre problemas más generales sobre grafos.

Esteban escribió una taxonomía muy útil que habla sobre todas las diferentes maneras de variar el problema, o de encontrar nuevos problemas/objetivos.
Gabriel mencionó el nuevo paper escrito por los autores originales recientemente, [[The Right to Hide]], que trata el mismo problema de Community Membership Hiding, pero resuelto con un método más tradicional (definen una función de pérdida diferenciable, lo que les permite aplicar un método de descenso por gradiente adaptado al problema). Este paper es sumamente útil, puesto que nos da un nuevo benchmark para comparar nuestro agente, y el hecho de que los autores sigan trabajando con este problema nos da pie a interactuar con ellos en el caso de que surjan dudas o problemas.
### Roadmap
Comencé a trabajar en la tesis propiamente hablando en Septiembre. El primer mes estuve familiarizándome con el problema, los términos utilizados, y el código creado por los autores originales. Hice énfasis especial en leer todo el código y entender bien a "bajo nivel" lo que está pasando cuando entrenamos y testeamos el agente.

Lo primero que hice fue tratar de replicar el paper. Sorpresivamente, ejecutando el código tal cuál se encontraba en el repositorio, no lograba obtener los resultados presentados. 
Analizando el código, el primer "problema" o controversia fue el uso de una estrategia de selección de nodos a testear que elegía comunidades que cumplían ciertas restricciones de tamaño (este problema y otros los vemos en detalle en [[#Problemas con el modelo original]])

Esto llevó a enviarles un correo a los autores, que nos aclararon que para entrenar era razonable seleccionar los nodos de forma aleatoria, y que al testear decidieron elegir un subconjunto de comunidades de distintos tamaños (específicamente, 0.2, 0.5 y 0.8 en relación a la comunidad de mayor tamaño) para tener un sampleo más general y menos sesgado.
De igual manera, al volver a correr el código con estos cambios, tampoco logré obtener los resultados originales (el proceso más detallado se puede ver en [[Experimentos]]).

*Como nota menor, hice algunos cambios menores para mejorar la "Quality of Life": el modelo agrega los hiperparámetros a los checkpoints para no tener que setearlos a mano cuando queremos testear, se loggean los tiempos de testeo y entrenamiento, se setean todas las semillas correspondientes de los generadores aleatorios de números para poder obtener resultados verdaderamente reproducibles, y demás cambios en el código para entenderlo y modificarlo de forma más sencilla*.

Con el código ya funcionando (porque aunque no logré replicar los resultados del paper, sí obtenía lo mismo que figuraba en el repositorio), pusimos una serie de objetivos sencillos para comenzar a variar el problema:
- Ver qué sucede si solo permitimos eliminar ejes.
- Ver qué sucede si solo permitimos agregar ejes.
- Redefinir el problema para unir dos comunidades $\mathcal{C}_1, \mathcal{C}_2$.

En pocas palabras, restringir las acciones posibles del agente aún más reduce el rendimiento en todos los casos, pero sorpresivamente sólo eliminar ejes tiene una efectividad levemente menor con una cantidad de operaciones realizados mucho menor.

Para redefinir el problema, hay que cambiar las funciones de pérdida, las acciones permitidas, el entorno de simulación, la entrada y salida de la red neuronal, etc...
Por lo tanto, me concentré en modularizar el código para en el futuro tener una [["Fábrica" de experimentos]] que nos permita plantear nuevos problemas de forma "rápida y sencilla" (y evitar el planteamiento *ad hoc* presente en el código original).

Al analizar el código con mayor profunidad, me di cuenta de que la red neuronal no estaba teniendo en cuenta el nodo objetivo, sino que recibía como entrada el grafo completo, y era el entorno de simulación el que se encargaba de "adaptar" el output de la red para actuar sobre el nodo objetivo.

Esto me llevo a pensar que, en vez de estar aprendiendo una política que se adapta en base a cuál es el nodo objetivo, el agente está aprendiendo una política "general" de ocultamiento de comunidades, que dado un grafo logra cambiar su estructura de comunidades, y como consecuencia aumenta la probabilidad de que el nodo objetivo sea uno de los nodos afectados. 
Lo importante es que la red **no sabe** quién es el nodo objetivo, por lo que resulta imposible que sus acciones dependan directamente de esta información (por ejemplo, tener más en cuenta la vecindad del nodo), sino que únicamente tienen raíz en la estructura general del grafo. 

La verdad es que no verifiqué esto directamente de forma experimental, sino que llegué a esta conclusión únicamente basándome en el código. Todavía no tengo desarrollado el agente nuevo, que sí tiene en cuenta cuál es el nodo objetivo.
El agente original pierde mucho rendimiento con grafos grandes, seguramente porque no actúa en basa al nodo objetivo. Como **hipótesis**, si mi razonamiento es correcto, el agente nuevo debería tener un rendimiento similar sin importar el tamaño del grafo, o como mínimo obtener un rendimiento mayor al original.

*Notas: volviendo a lo que decía antes, la idea es desarrollar un agente robusto y bien pensado, que sirva como base para resolver distintos problemas sobre grafos, y no quedarnos fijados en el problema de community membership hiding.
Seguramente cada problema lleva a utilizar distintas estructuras de red y selección de hiperparámetros, pero la idea sería poder cambiar únicamente las funciones de pérdida y las acciones permitidas y obtener un modelo que resuelva un problema completamente distinto, manteniendo la estructura de la red neuronal lo más similar posible.*
### Problemas con el código/modelo originales
Voy a incluir extractos del código original, puesto que la raíz de estos problemas no viene de la definición conceptual, sino de la implementación.

De igual forma, no voy a incluir un "paso a paso" de por qué el código funciona como funciona, puesto que sería lo mismo que tratar de seguir un *call stack* de las funciones: es tedioso e innecesario, y resultaría más fácil leer el código en sí. Explico por qué el código causa los problemas qué causa.

*Disclaimer: es posible que haya malinterpretado o entendido mal el porqué real de estos problemas, puesto que no hice un testing riguroso de cambiar el código y ver qué pasa. Las conclusiones llegan de razonar el proceso del código, y la idea es una vez que tenga el agente nuevo (que incorpora el nodo objetivo a la red neuronal), hacer una comparación más precisa. 
Bajo ningún punto de vista quiero atacar o criticar a los autores originales. El código es excelente y está muy bien documentado, simplemente no se alinea directamente con nuestro objetivo general de atacar diversos problemas más allá del original.*

#### Problemas de `utils.py`
En `utils.py` (archivo de parámetros, hiperparámetros y configuración general) aparece, entre otras cosas, lo siguiente:
```python
    # Method to change the target community
    # - 1: choose a random community
    # - 2: choose the community with the length closest to the half of the maximum
    #       length of the communities.
    # - 3: choose a community based on the distribution of the number of
    #       nodes in the communities
    COMMUNITY_CHANGE_METHOD = 2

    PREFERRED_COMMUNITY_SIZE = [0.2, 0.5, 0.8]
```

Volviendo a lo mencionado anteriormente, en ningún momento se aclara ni qué método usar tanto al entrenar como al testear, ni el por qué de estas elecciones.
Utilizar el método 2 al entrenar introduce un sesgo, ya que solo testea con esos tamaños de comunidades (y en particular, por cómo está implementado el método, siempre con las mismas comunidades).

**Solución:** Utilizo el método 1 al entrenar, y al testear puedo utilizar el método 2 para pocas corridas, o un nuevo método "proporcional" (donde la probabilidad de elegir la comunidad es proporcional a su tamaño) para un testeo más general (también lo puedo utilizar para entrenar).

```python
 """ Graph Encoder Parameters """ ""
    RANDOM_NODE2VEC = True  # If True, use Random features instead of Node2Vec
    EMBEDDING_DIM = 128  # 256
    WALK_NUMBER = 5  # 5, 10
    WALK_LENGTH = 40  # 40, 80```
```

No sólo que se utilizan features aleatorias en vez de utilizar **node2vec** (a mi parecer, gravísimo, puesto que node2vec es una parte clave del proceso. Es un gran encargado de codificar la información )

```python
    # Change target community and target node with a probability of EPSILON
    EPSILON = [0]  # Between 0 and 100
```

