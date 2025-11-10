## Proyecto original
Este proyecto de tesis comenzó en base al paper [[Evading Community Detection via Counterfactual Neighborhood Search.pdf|Evading Community Detection via Counterfactual Neighborhood Search]].

El paper describe un método que dado un grafo, un algoritmo de detección de comunidades y un nodo objetivo, permite "ocultar" el nodo objetivo del algoritmo de detección de comunidades, en el sentido de cambiar la estructura de comunidades para dejar de ser identificado como parte de la comunidad original.

Esto se logra utilizando un agente entrenado con reinforcement learning, utilizando un método [[Algoritmos Actor-Critic|Actor-Critic]] (específicamente, [[Referencias#(26) Advantage Actor-Critic (A2C) |A2C, o Advantage Actor Critic]]).
Los autores llaman a este proceso **Community Memerbship Hiding**.

El problema original tiene las siguientes restricciones:
- Se necesita información completa sobre el grafo.
- Se oculta un único nodo objetivo $u$.
- Las comunidades no se solapan.
- El algoritmo de detección se considera *black-box*.
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
- $sim(\cdot, \cdot)$ es una función de similaridad entre dos comunidades. Se pueden utilizar varias definiciones, pero en este caso los autores utilizaron el [[Glosario#Conceptos introducidos en las referencias|Coeficiente Sørensen-Dice]], que en pocas palabras determina qué tantos nodos comparten ambas comunidades.

Con respecto al agente, utilizan la siguiente arquitectura de red:
![Model Architecture](Tesis/READMES/images/model_architecture_background.png)
[[node2vec]] es un algoritmo que genera embeddings de cada nodo a partir del grafo, con un proceso similar a **word2vec**, donde se genera un vector para cada nodo que resume su "signficado" o propiedades y posición dentro del grafo.

En pocas palabras, node2vec realiza caminatas aleatorias de segundo orden (tiene en cuenta el estado actual y el anterior) para generar oraciones, con los nodos como palabras, que luego son pasadas a un modelo skip-gram con negative sampling que genera los embeddings.

Tiene varios hiperparámetros, pero los más importantes $p$ y $q$ (probabilidades de volver al nodo anterior, y de explorar nodos "lejanos" al anterior, respectivamente) determinan si los embeddings representan **homofilia** (nodos altamente interconectados y que pertenecen a clusters similares deben ser embebidos cerca) o **equivalencia estructural** (nodos con roles estructurales similares en las redes deben ser embebidos cerca, sin hacer énfasis en su conectividad "real").

Sin embargo, tengo la **hipótesis** de que estos embeddings pueden ser contraproducentes, ya que se calculan una única vez en el grafo original y no se vuelven a computar, y a medida que el grafo cambia van dejando de representar la realidad. Además, también se podría argumentar que es un estilo de "trampa" puesto que le dice qué nodos ocultar en base a sus características (*information leakage)*. Quizás es mejor dejar que el agente aprenda por sí solo por qué y cuáles nodos ocultar.

Es importante entender la idea detrás de tener un actor y un crítico, en vez de únicamente un actor. 
El **actor** es quien decide la acción a tomar, aprende una política $\pi(a | s)$, una distribución de probabilidad sobre acciones dado el estado actual. 
El **crítico** "evalúa las acciones", y aprende una función de valor $V(s)$, que nos dice "qué tan bueno" es el estado actual bajo la política actual (devuelve un escalar).

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
*Además, cree una herramienta  para visualizar los grafos antes y después de las modificaciones:*
![[Screenshot_20251110_143436.png]]


Con el código ya funcionando (porque aunque no logré replicar los resultados del paper, sí obtenía lo mismo que figuraba en el repositorio), pusimos una serie de objetivos sencillos para comenzar a variar el problema:
- Ver qué sucede si solo permitimos eliminar ejes.
- Ver qué sucede si solo permitimos agregar ejes.
- Redefinir el problema para unir dos comunidades $\mathcal{C}_1, \mathcal{C}_2$.

En pocas palabras, restringir las acciones posibles del agente aún más reduce el rendimiento en todos los casos, pero sorpresivamente sólo eliminar ejes tiene una efectividad levemente menor con una cantidad de operaciones realizadas mucho menor.

Para redefinir el problema, hay que cambiar las funciones de pérdida, las acciones permitidas, el entorno de simulación, la entrada y salida de la red neuronal, etc...
Por lo tanto, me concentré en modularizar el código para en el futuro tener una [["Fábrica" de experimentos]] que nos permita plantear nuevos problemas de forma "rápida y sencilla" (y evitar el planteamiento *ad hoc* presente en el código original).

Al analizar el código con mayor profunidad, me di cuenta de que la red neuronal no estaba teniendo en cuenta el nodo objetivo, sino que recibía como entrada el grafo completo, y era el entorno de simulación el que se encargaba de "adaptar" el output de la red para actuar sobre el nodo objetivo.

Esto me llevo a pensar que, en vez de estar aprendiendo una política que se adapta en base a cuál es el nodo objetivo, el agente está aprendiendo una política "general" de ocultamiento de comunidades, que dado un grafo logra cambiar su estructura de comunidades, y como consecuencia aumenta la probabilidad de que el nodo objetivo sea uno de los nodos afectados. 
Lo importante es que la red **no sabe** quién es el nodo objetivo, por lo que resulta imposible que sus acciones dependan directamente de esta información (por ejemplo, tener más en cuenta la vecindad del nodo), sino que únicamente tienen raíz en la estructura general del grafo. 

Para verificar esto, volví a bajar el código original y verifiqué los outputs del actor y el crítico para la primer iteración del proceso, con semillas fijas, variando únicamente el nodo objetivo.
Independientemente del nodo seleccionado, tanto las probabildades del actor como el escalar del crítico eran iguales, lo que implica que no tienen en cuenta este nodo al realizar las estimaciones (en las iteraciones siguientes, como los ejes agregados/eliminados dependen del nodo objetivo, obtengo grafos distintos por cada nodo, por lo que lógicamente los valores estimados también difieren. Esto no invalida lo que planteo).

El agente original pierde mucho rendimiento con grafos grandes, seguramente debido a que no actúa en basa al nodo objetivo. Como **hipótesis**, si mi razonamiento es correcto, el agente nuevo debería tener un rendimiento similar sin importar el tamaño del grafo, o como mínimo obtener un rendimiento mayor al original.

*Notas: volviendo a lo que decía antes, la idea es desarrollar un agente robusto y bien pensado, que sirva como base para resolver distintos problemas sobre grafos, y no quedarnos fijados en el problema de community membership hiding.
Seguramente cada problema lleva a utilizar distintas estructuras de red y selección de hiperparámetros, pero la idea sería poder cambiar únicamente las funciones de pérdida y las acciones permitidas y obtener un modelo que resuelva un problema completamente distinto, manteniendo la estructura de la red neuronal lo más similar posible.*
### Problemas con el código/modelo originales
Voy a incluir extractos del código original, puesto que la raíz de estos problemas no viene de la definición conceptual, sino de su implementación.

De igual forma, no voy a incluir un "paso a paso" de por qué el código funciona como funciona, puesto que sería lo mismo que tratar de seguir un *call stack* de las funciones: es tedioso e innecesario, y resultaría más fácil leer el código en sí. Explico por qué el código causa los problemas qué causa.

*Disclaimer: es posible que haya malinterpretado o entendido mal el porqué real de estos problemas, puesto que no hice un testing riguroso de cambiar el código y ver qué pasa. Las conclusiones llegan de razonar el proceso del código, y la idea es una vez que tenga el agente nuevo (que incorpora el nodo objetivo a la red neuronal), hacer una comparación más precisa. 
Bajo ningún punto de vista quiero atacar o criticar a los autores originales. El código es excelente y está muy bien documentado, simplemente no se alinea directamente con nuestro objetivo general de atacar diversos problemas más allá del original.*

#### Problemas de configuración
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

**Solución:** Utilizo el método 1 al entrenar, y al testear puedo utilizar el método 2 para pocas corridas, o un nuevo método "proporcional" (donde la probabilidad de elegir la comunidad es proporcional a su tamaño) para una evaluación/entrenamiento más general.

```python
 """ Graph Encoder Parameters """ ""
    RANDOM_NODE2VEC = True  # If True, use Random features instead of Node2Vec
    EMBEDDING_DIM = 128  # 256
    WALK_NUMBER = 5  # 5, 10
    WALK_LENGTH = 40  # 40, 80```
```

No sólo que se utilizan features aleatorias en vez de utilizar **node2vec**, sino que tampoco se utilizaban ni los pesos de los ejes, ni los embeddings (aleatorios o no) de node2vec.

```python
# a2c.py

# faltarían los parámetros group_node_attrs y group_edge_attrs
state = from_networkx(graph).to(self.device)
```


```python
	# Change target community and target node with a probability of EPSILON
	EPSILON = [0]  # Between 0 and 100
```

Supongo que este parámetro $\epsilon$ lo habrán ido variando a medida que probaban diversos modelos, pero setear un epsilon igual a cero hace que siempre entrenemos al modelo con el mismo nodo, lo que evita aprender patrones más generales, puede llevar a overfittear, y depende mucho de qué tan "fácil de ocultar" es el nodo *(según los autores originales, hay nodos mucho más fáciles y mucho más difíciles de ocultar que otros, según sus características y posición en el grafo original)*.

#### Código general
Como último "error", en la función `training_step()`, encargada de computar las pérdidas de los modelos, no se utilizaba correctamente la GPU:
```python
value_losses.append(F.smooth_l1_loss(value, torch.tensor([R]).to(self.device)))
#...
advantage = R - value.item()
```
En pocas palabras, constantemente se transferían valores entre la CPU y la GPU, y viceversa, de forma innecesaria, desaprovechando el potencial de los tensores de CUDA. Lo arreglé manteniendo todo en la GPU.

A nivel código y documentación, está todo muy limpio, prolijo y bien documentado. El código es fácil de entender y modificar, con comentarios explicando cada cosa.
*(quizás muchos comentarios, creo que el código tiene que "hablar por sí solo". De igual forma, el enfoque de este proyecto va más allá de la ingeniería del software y la calidad del código...)*
## Estado actual
Construí un modelo que sí tiene en cuenta el nodo objetivo, aunque sin entender por completo si su estructura es buena, útil, o si verdaderamente tiene sentido.

Desde la última semana estuve leyendo varios recursos sobre aprendizaje por refuerzo:
- [Reinforcement learning on graphs: A survey](https://arxiv.org/abs/2204.06127) 
- Reinforcement Learning: An Introduction (2nd Edition) (LCCN 2018023826 | ISBN 9780262039246).
- [Inductive Representation Learning on Large Graphs](https://arxiv.org/abs/1706.02216)

Hice algunos tests con el modelo, pero nada concluyente por ahora. Quiero entender bien lo que estoy haciendo y por qué antes de arrancar con la fase de prueba y error.

El próximo objetivo es hacer un modelo que "funcione", en el sentido que tiene en cuenta el nodo objetivo y obtiene un rendimiento como mínimo igual al anterior, entendiendo el flujo de los datos y el porqué del funcionamiento.

Como objetivo alternativo, podría mantener la estructura original, que trabaja sobre grafos, y tratar de plantear los problemas sobre grafos en general. Al final del día, en reinforcement learning el desafío viene no sólo de diseñar una estructura de red efectiva, sino también de diseñar bien las funciones de pérdida y recompensa, el espacio de acciones, y la entrada y procesamiento de datos.
De igual forma, priorizo lograr un modelo que tenga en cuenta el nodo objetivo. Me parece más útil, práctico, y a nivel de aprendizaje personal es un desafío mucho mayor, con la recompensa de aprender reinforcement learning.

*Nota: Envíe un correo a los autores originales preguntando el por qué no tuvieron en cuenta el nodo objetivo. Todavía espero la respuesta, pero es posible que hayan considerado esto y luego de evaluar su plausibilidad, hayan optado por un enfoque que considera todo el grafo.*