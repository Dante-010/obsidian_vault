>  Incluyo únicamente las referencias que leí y/o consider útiles para algo que vayamos a realizar o para explicar algún concepto.

### Template
### (N) TITULO
#### REFERENCIA https://doi.org/URL
> [!abstract] Abstracto
>  ABSTRACTO

## Originales
### (8) Community Deception
####  Valeria Fionda and Giuseppe Pirrò. 2018. Community Deception or: How to Stop Fearing Community Detection Algorithms. IEEE Transactions on Knowledge and Data Engineering 30, 4 (2018), 660–673. https://ieeexplore.ieee.org/document/8118127. 2776133
> [!abstract] Abstracto
>  In this paper, we research the community deception problem. Tackling this problemconsists in developing techniques to hide a target community (C) fromcommunity detection algorithms. This need emerges whenever a group (e.g., activists, police enforcements, or network participants in general) want to observe and cooperate in a social networkwhile avoiding to be detected. We introduce and formalize the community deception problemand devise an efficient algorithmthat allows to achieve deception by identifying a certain number (beta) of C's members connections to be rewired. Deception can be practically achieved in social networks like Facebook by friending or unfriending networkmembers as indicated by our algorithm. We compare our approachwith another technique based onmodularity. By considering a variety of (large) real networks, we provide a systematic evaluation of the robustness of community detection algorithms to deception techniques. Finally, we open some challenging research questions about the design of detection algorithms robust to deception techniques.

---
### (10) Betweenness Centrality
#### Linton C. Freeman. 1977. A Set of Measures of Centrality Based on Betweenness. Sociometry 40, 1 (1977), 35–41. http://www.jstor.org/stable/3033543
> [!abstract] Abstracto
>  A Family of new measures of point and graph centrality based on early intuitions of Bavelas (1948) is introduced. These measures define centrality in terms of the degree to which a point falls on the shortest path between others and therefore has a potential for control of communication. They may be used to index centrality in any large or small network of symmetrical relations, whether connected or unconnected.

---
### (13) Node2Vec
#### Aditya Grover and Jure Leskovec. 2016. node2vec: Scalable Feature Learning for Networks. https://arxiv.org/abs/1607.00653 \[cs.SI\]
> [!abstract] Abstracto
>  Prediction tasks over nodes and edges in networks require careful
effort in engineering features used by learning algorithms. Recent
research in the broader field of representation learning has led to
significant progress in automating prediction by learning the fea-
tures themselves. However, present feature learning approaches
are not expressive enough to capture the diversity of connectivity
patterns observed in networks.
Here we propose node2vec, an algorithmic framework for learn-
ing continuous feature representations for nodes in networks. In
node2vec, we learn a mapping of nodes to a low-dimensional space
of features that maximizes the likelihood of preserving network
neighborhoods of nodes. We define a flexible notion of a node’s
network neighborhood and design a biased random walk procedure,
which efficiently explores diverse neighborhoods. Our algorithm
generalizes prior work which is based on rigid notions of network
neighborhoods, and we argue that the added flexibility in exploring
neighborhoods is the key to learning richer representations.
We demonstrate the efficacy of node2vec over existing state-of-
the-art techniques on multi-label classification and link prediction
in several real-world networks from diverse domains. Taken to-
gether, our work represents a new way for efficiently learning state-
of-the-art task-independent representations in complex networks.

**node2vec** utiliza el hecho de que las caminatas aleatorias en un grafo pueden ser tratadas como oraciones en un corpus. Cada nodo es tratado como una palabra individual, y una caminata aleatoria es tratada como una oración. Si les damos estas oraciones a un modelo como **word2vec** (que define *embeddings* para palabras/tokens), los caminos encontrados por las caminatas, ahora oraciones, pueden ser analizados con técnicas tradicionales de data-mining para documentos.

En el caso del paper, se puede utilizar node2vec para extraer feature vectors de los nodos en el caso que el grafo no tenga features propios (también existe la opción de simplemente asignar features aleatorias).

Se mencionan dos conceptos interesantes para nodos en grafos (específicamente, la relación entre los nodos originales y su *embedding*): **homofilia** (nodos altamente interconectados y que pertenecen a clusters similares deben ser embedidos cerca), y **equivalencia estructural** (nodos con roles estructurales similares en las redes deben ser embedidos cerca, sin hacer énfasis en su conectividad "real").

---
### (18) GCNConv
#### Thomas N. Kipf and Max Welling. 2017. Semi-Supervised Classification with Graph Convolutional Networks. https://arxiv.org/abs/1609.02907 \[cs.LG\]
> [!abstract] Abstracto
>  We present a scalable approach for semi-supervised learning on graph-structured
data that is based on an efficient variant of convolutional neural networks which
operate directly on graphs. We motivate the choice of our convolutional archi-
tecture via a localized first-order approximation of spectral graph convolutions.
Our model scales linearly in the number of graph edges and learns hidden layer
representations that encode both local graph structure and features of nodes. In
a number of experiments on citation networks and on a knowledge graph dataset
we demonstrate that our approach outperforms related methods by a significant
margin.


---
### (26) Advantage Actor-Critic (A2C)
#### Volodymyr Mnih, Adrià Puigdomènech Badia, Mehdi Mirza, Alex Graves, Timothy P. Lillicrap, Tim Harley, David Silver, and Koray Kavukcuoglu. 2016. Asynchronous Methods for Deep Reinforcement Learning. https://arxiv.org/abs/1602.01783 \[cs.LG\]
> [!abstract] Abstracto
>  We propose a conceptually simple and lightweight framework for deep reinforcement learning that uses asynchronous gradient descent for optimization of deep neural network controllers. We present asynchronous variants of four standard reinforcement learning algorithms and show that parallel actor-learners have a stabilizing effect on training allowing all four methods to successfully train neural network controllers. The best performing method, an asynchronous variant of actor-critic, surpasses the current state-of-the-art on the Atari domain while training for half the time on a single multi-core CPU instead of a GPU. Furthermore, we show that asynchronous actor-critic succeeds on a wide variety of continuous motor control problems as well as on a new task of navigating random 3D mazes using a visual input.

---
### (41) Roam Based Heuristic
#### Marcin Waniek, Tomasz P. Michalak, Michael J. Wooldridge, and Talal Rahwan. 2018. Hiding Individuals and Communities in a Social Network. Nature Human Behaviour 2, 2 (Jan 2018), 139–147. https://doi.org/10.1038/s41562-017-0290-3
> [!abstract] Abstracto
>  The Internet and social media have fuelled enormous interest in social network analysis. New tools continue to be developed and used to analyse our personal connections, with particular emphasis on detecting communities or identifying key individuals in a social network. This raises privacy concerns that are likely to exacerbate in the future. With this in mind, we ask the question ‘Can individuals or groups actively manage their connections to evade social network analysis tools?’ By addressing this question, the general public may better protect their privacy, oppressed activist groups may better conceal their existence and security agencies may better understand how terrorists escape detection. We first study how an individual can evade ‘node centrality’ analysis while minimizing the negative impact that this may have on his or her influence. We prove that an optimal solution to this problem is difficult to compute. Despite this hardness, we demonstrate how even a simple heuristic, whereby attention is restricted to the individual’s immediate neighbourhood, can be surprisingly effective in practice; for example, it could easily disguise Mohamed Atta’s leading position within the World Trade Center terrorist network. We also study how a community can increase the likelihood of being overlooked by community-detection algorithms. We propose a measure of concealment—expressing how well a community is hidden—and use it to demonstrate the effectiveness of a simple heuristic, whereby members of the community either ‘unfriend’ certain other members or ‘befriend’ some non-members in a coordinated effort to camouflage their community.

---

## Nuevas
### (1) The Right to Hide: Masking Community Affiliation via Minimal Graph Rewiring

####  Matteo Silvestri et al. The Right to Hide: Masking Community Affilia-
tion via Minimal Graph Rewiring. 2025. arXiv: 2502.00432 [cs.SI]. url:
https://arxiv.org/abs/2502.00432.
> [!abstract] Abstracto
>  Protecting privacy in social graphs may require obscuring nodes' membership in sensitive communities. However, doing so without significantly disrupting the underlying graph topology remains a key challenge. In this work, we address the community membership hiding problem, which involves strategically modifying the graph structure to conceal a target node's affiliation with a community, regardless of the detection algorithm used. We reformulate the original discrete, counterfactual graph search objective as a differentiable constrained optimisation task. To this end, we introduce \nabla-CMH, a new gradient-based method that operates within a feasible modification budget to minimise structural changes while effectively hiding a node's community membership. Extensive experiments on multiple datasets and community detection methods demonstrate that our technique outperforms existing baselines, achieving the best balance between node hiding effectiveness and graph rewiring cost, while preserving computational efficiency.

Ver [[The Right to Hide]].