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
[[README_main]] explica de manera excelente el concepto de [[Community membership hiding]], [[Advantage Actor-Critic (A2C)]] y el código necesario para replicar los resultados que obtuvieron.

Ver [[Código]] para entender qué hace cada módulo y saber cómo determinar qué queremos correr y [[Experimentos]] para ver los resultados. 

En [[Variables, parámetros y ecuaciones]], se encuentran explicaciones y posibles valores a explorar/ya explorados.

En [[Glosario]], anoté brevemente términos importantes para entender el paper, ya sean conceptos "comunes" o conocidos, como conceptos obtenidos de las [[Referencias]].

-------

### Alternativas posibles
> However, the community deception task, as defined by Fionda and Pirrò [8] , does not have a binary outcome, unlike our community membership hiding goal. Instead, the authors intro- duce a smooth measure, the Deception Score, that combines three criteria for effective community masking: reachability, spreadness, and hiding. While it might seem plausible to extend our method for community deception by running multiple membership hiding tasks for each node in the target community, this straightforward strategy might be too aggressive due to our more stringent (i.e., binary) definition of deception goal. A more nuanced approach could involve leveraging the structural properties of each node in the community to mask (e.g., their degree) to cleverly select target nodes for membership hiding. Further exploration of this strategy is left for future research.

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=2&selection=22,11,46,28|Evading Community Detection via Counterfactual Neighborhood Search, page 2]]

