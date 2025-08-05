##### Expanding the Problem Space and Analyzing Alternative Scenarios
---
Nuestro proyecto de tesis toma de base el paper **Evading Community Detection via**
**Counterfactual Neighborhood Search** *(Andrea Bernini, Fabrizio Silvestri y Gabriele Tolomei)* y expande sobre el problema propuesto. 

> [!abstract] Abstracto original
>  Community detection techniques are useful for social media plat- forms to discover tightly connected groups of users who share common interests. However, this functionality often comes at the expense of potentially exposing individuals to privacy breaches by inadvertently revealing their tastes or preferences. Therefore, some users may wish to preserve their anonymity and opt out of community detection for various reasons, such as affiliation with political or religious organizations, without leaving the platform. In this study, we address the challenge of community membership hiding, which involves strategically altering the structural proper- ties of a network graph to prevent one or more nodes from being identified by a given community detection algorithm. We tackle this problem by formulating it as a constrained counterfactual graph ob- jective, and we solve it via deep reinforcement learning. Extensive experiments demonstrate that our method outperforms existing baselines, striking the best balance between accuracy and cost.

> The core challenge of this problem lies in determining how to strategically modify the structural properties of a network graph, effectively excluding one or more nodes from being identified by a given community detection algorithm. 

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=1&selection=188,0,191,37|Evading Community Detection via Counterfactual Neighborhood Search, page 1]]

> Our main contributions are summarized below. 
> • We formulate the community membership hiding problem as a constrained counterfactual graph objective. 
> • We cast this problem within an MDP framework and solved it via DRL. 
> • We utilize a graph neural network (GNN) representation to capture the structural complexity of the input graph, which in turn is used by the DRL agent to make its decisions. 
> • We validate the performance of our method in comparison with existing baselines using standard quality metrics. 
> • We publicly release both the source code and the data utilized in this study to encourage reproducibility.

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=2&selection=56,0,82,43|Evading Community Detection via Counterfactual Neighborhood Search, page 2]]

---
[[README_main]] explica de manera excelente el concepto de [[Community Membership Hiding]], [[Advantage Actor-Critic (A2C)]] y el código necesario para replicar los resultados que obtuvieron.

Ver [[Código]] para entender qué hace cada módulo y saber cómo determinar qué queremos correr y [[Experimentos]] para ver los resultados. 

En [[Variables, parámetros y ecuaciones]], se encuentran explicaciones y posibles valores a explorar/ya explorados.

En [[Glosario]], anoté brevemente términos importantes para entender el paper, ya sean conceptos "comunes" o conocidos, como conceptos obtenidos de las [[Referencias]].

-------

### Alternativas posibles

#### Introducidas en el paper

> However, the community deception task, as defined by Fionda and Pirrò [8] , does not have a binary outcome, unlike our community membership hiding goal. Instead, the authors intro- duce a smooth measure, the Deception Score, that combines three criteria for effective community masking: reachability, spreadness, and hiding. While it might seem plausible to extend our method for community deception by running multiple membership hiding tasks for each node in the target community, this straightforward strategy might be too aggressive due to our more stringent (i.e., binary) definition of deception goal. A more nuanced approach could involve leveraging the structural properties of each node in the community to mask (e.g., their degree) to cleverly select target nodes for membership hiding. Further exploration of this strategy is left for future research.

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=2&selection=22,11,46,28|Evading Community Detection via Counterfactual Neighborhood Search, page 2]]

Hacemos el approach "naive" que realiza varias membership hiding tasks para cada nodo, y el approach más optimizado de considerar las propiedades estructurales de los nodos para elegir los más indicados.

----

> We leave the exploration of overlapping communities for future work.

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=3&selection=170,2,171,59|Evading Community Detection via Counterfactual Neighborhood Search, page 3]]

Tenemos que adaptar el algoritmo para poder trabajar con overlapping communities (ver [[Variables, parámetros y ecuaciones]]).

----
> Anyway, the rationale behind how communities are found is irrelevant to our task, and, hereinafter, we will treat the community detection technique 𝑓 (·) as a "black box."

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=3&selection=218,3,226,17|Evading Community Detection via Counterfactual Neighborhood Search, page 3]]

Observamos qué sucede si tenemos información del algoritmo (en teoría, esto nos permite aprovechar el funcionamiento del algoritmo, tomando ventaja de ciertas heurísticas o estructuras para lograr un resultado más óptimo).

----

> While altering node features holds potential interest, that aspect is reserved for future work.

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=3&selection=238,22,239,50|Evading Community Detection via Counterfactual Neighborhood Search, page 3]]

Consideramos las variantes del algoritmo donde los nodos tienen features, y vemos qué sucede si modificamos las features.

----

>  Nevertheless, our method might work under more relaxed conditions (e.g., partial graph knowledge), as discussed in Section 7.1

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=3&selection=259,13,261,24|Evading Community Detection via Counterfactual Neighborhood Search, page 3]]

Vemos qué sucede si no tenemos información completa sobre el grafo (por ejemplo, sólo los vecinos, sólo mi cluster, sólo clusters vecinos, etc...).