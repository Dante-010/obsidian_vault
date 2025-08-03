## Conceptos generales/conocidos
- **[Markov Decision Process](https://en.wikipedia.org/wiki/Markov_decision_process):** Modelo para toma de decisiones secuenciales, utilizado en *reinforcement learning* para modelar la interacción entre un agente y su entorno.
- **[F score](https://en.wikipedia.org/wiki/F-score):** $\frac{2TP}{2TP + FP + FN}$
- **[Deep reinforcement learning](https://en.wikipedia.org/wiki/Deep_reinforcement_learning)**: Reinforcement learning implementado con deep learning. Los agentes aprenden a realizar decisiones interactuando con su entorno con el objetivo de maximizar recompensas acumulativas, utilizando redes neuronales profundas para representar reglas, funciones de valor, o modelos del entorno.
  Utilizar redes neuronales profundas permitió que métodos de reinforcement learning puedan ser utilizados con espacios de entrada continuos o de alta dimensionalidad.

- **Community detection (ver [[Algoritmos de detección de comunidades]])**: Los algoritmos de detección de comunidades se utilizan para particionar un grafo en clusters. Se pueden categorizar en dos grandes tipos: **overlapping** y **non-overlapping** (superpuestos y no supuerpuestos), donde la principal diferencia es si un nodo puede pertenecer a varios clusters al mismo tiempo, o no, respectivamente.
- **Community** *(dentro de community detection)*: No hay una definición universalmente aceptada, pero en general se busca que una comunidad exhiba fuertes conexiones intra-cluster, y relativamente débiles conexiones inter-cluster.
---
## Conceptos introducidos en las referencias
- **Deception score** ([[Referencias#(8) Community Deception|8]]):
- **Community Deception** ([[Referencias#(8) Community Deception|8]]): Especialización de community membership hiding, donde el objetivo es esconder una comunidad entera de los algoritmos de detección de comunidades.
---
## Conceptos propios al paper
- [[Community membership hiding]].
- **Transferability:** La propiedad de la solución original de mantener su efectividad sin importar el algortimo utilizado (en especial, si el algoritmo no apareció en la fase de entrenamiento).