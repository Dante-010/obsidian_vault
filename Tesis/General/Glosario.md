## Conceptos generales/conocidos
- **[Markov Decision Process](https://en.wikipedia.org/wiki/Markov_decision_process):** Modelo para toma de decisiones secuenciales, utilizado en *reinforcement learning* para modelar la interacción entre un agente y su entorno.
- **[F score](https://en.wikipedia.org/wiki/F-score):** $\frac{2TP}{2TP + FP + FN}$
- **[Deep reinforcement learning](https://en.wikipedia.org/wiki/Deep_reinforcement_learning):** Reinforcement learning implementado con deep learning. Los agentes aprenden a realizar decisiones interactuando con su entorno con el objetivo de maximizar recompensas acumulativas, utilizando redes neuronales profundas para representar reglas, funciones de valor, o modelos del entorno.
  Utilizar redes neuronales profundas permitió que métodos de reinforcement learning puedan ser utilizados con espacios de entrada continuos o de alta dimensionalidad.
- **Community detection (ver [[Algoritmos de detección de comunidades]]):** Los algoritmos de detección de comunidades se utilizan para particionar un grafo en clusters. Se pueden categorizar en dos grandes tipos: **overlapping** y **non-overlapping** (superpuestos y no supuerpuestos), donde la principal diferencia es si un nodo puede pertenecer a varios clusters al mismo tiempo, o no, respectivamente.
- **Community** *(dentro de community detection)*: No hay una definición universalmente  aceptada, pero en general se busca que una comunidad exhiba fuertes conexiones intra-cluster, y relativamente débiles conexiones inter-cluster.
- **(Condicional) Contrafactual:** Oraciones condicionales que discuten lo que podría haber sido verdad bajo circunstancias diferentes. Un **grafo contrafactual** es una versión hipotética de un grafo utilizada para responder preguntas de la forma: ¿qué pasaría con (una predicción, una comunidad, una clasificación, etc.) si cambiásemos parte del grafo?
- [[Algoritmos Actor-Critic]].
- **Episodios (en RL):** Una corrida completa del agente. Comienza con el agente en un estado inicial, y termina cuando el agente llega a un estado "final" (cumplió el objetivo, falló, o se alcanzó un límite de tiempo/recursos). Aparte, una tarea puede ser *episódica*, donde se divide la interacción en episodios, y el entorno se reinicia cada vez, o *continua*, donde no hay estado terminal y el entorno evoluciona constantemente.
- **Ablation study:** Aplicado a la Inteligencia Artificial, consta de remover partes de un  sistema de inteligencia artificial, reentrenar y evaluar su rendimiento. Permite ver qué partes son importantes, y en un sistema bien diseñado el rendimiento no debería verse altamente afectado.
- **Funciones de similaridad/distancia (entre grafos y comunidades):** ver [[Funciones de similaridad (grafos y comunidades)]]
- **Curriculum learning:** Técnica en Machine Learning basada en entrenar un modelo con ejemplos de dificultad incremental, donde la definición de "dificultad" puede ser provista externamente o descubierta como parte del proceso de entrenamiento.
  La idea es obtener un buen rendimiento de forma "rápida", o converger a un mejor óptimo local si el óptimo global no es encontrado.
  


---
## Conceptos introducidos en las referencias
- **Deception score** ([[Referencias#(8) Community Deception|8]]): Evalúa el nivel de ocultamiento de una comunidad $\mathcal{C}$ dentro de una estructura de comunidad $\overline{C}$ encontrada por algún algoritmo (ver referencia para definición formal, tiene sentido casi que únicamente dentro del contexto del paper).
- **Community Deception** ([[Referencias#(8) Community Deception|8]]): Especialización de community membership hiding, donde el objetivo es esconder una comunidad entera de los algoritmos de detección de comunidades.
- **Coeficiente Dice-Sørensen (DSC)**:  $$\operatorname{DSC}(C_i, C_i^{t}) = \frac{2 \left| C_i \cap C_i^{t} \right|}{\left| C_i \right| + \left| C_i^{t} \right|}$$
  Devuelve un valor entre $0$ (no hay similaridad) y $1$ (fuerte similaridad).
- **Normalized Mutual Information (NMI)**
  El **score NMI** utilizado para medir la similitud entre estructuras de comunidad, toma valores entre 0 (sin información mutua) y 1 (correlación perfecta).  
  Se expresa como:

  $$
   \operatorname{NMI}(K, K^{t}) = I_{\text{norm}}(X : Y) = \frac{H(X) + H(Y) - H(X, Y)}{\frac{1}{2} \bigl( H(X) + H(Y) \bigr)},
   $$

  donde:  
	- $H(X)$ y $H(Y)$ son las entropías de las variables aleatorias $X$ y $Y$ asociadas a las particiones $(K = f(\mathcal{G}))$ y $(K^{t} = f(\mathcal{G}^{t}))$, respectivamente.  
	- $H(X, Y)$ es la entropía conjunta.  

  Para convertir esta métrica en una distancia, se utiliza:  $1 - \operatorname{NMI}(K, K^{t})$.
- **Distancia de Jaccard**:
  La distancia de Jaccard, adaptada al caso de dos grafos $\mathcal{G}$ y $\mathcal{G}^{t}$ se define como:  

  $$\operatorname{Jaccard}(\mathcal{G}, \mathcal{G}^{t}) = \frac{|\mathcal{G} \cup \mathcal{G}^{t}| - |\mathcal{G} \cap \mathcal{G}^{t}|}{|\mathcal{G} \cup \mathcal{G}^{t}|} = \frac{\sum_{i,j} |A_{i,j} - A^{t}_{i,j}|}{\sum_{i,j} \max(A_{i,j}, A^{t}_{i,j})},$$

  donde:  
	- $A_{i,j}$ denota la entrada $(i, j)$-ésima de la matriz de adyacencia del grafo original $\mathcal{G}$.  
	- $A^{t}_{i,j}$ es la entrada correspondiente para el grafo $\mathcal{G}^{t}$.  

  La distancia de Jaccard vale 0 cuando los dos grafos son idénticos y 1 cuando son completamente diferentes.
  
- **Homofilia y Equivalencia estructural**: ver [[Referencias#(13) Node2Vec]].

---
## Conceptos propios al paper
- [[Community Membership Hiding]].
- **Transferability:** La propiedad de la solución original de mantener su efectividad sin importar el algortimo utilizado (en especial, si el algoritmo no apareció en la fase de entrenamiento).