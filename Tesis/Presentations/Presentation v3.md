
# Graph Community Membership Hiding

### Analysis, Replication, and Dual-Agent Architectures

---

## Original Project Scope

### Goal

“Hide” a target node $u$ from a community detection algorithm treated as a black box.

### Method

A Reinforcement Learning agent (A2C) that modifies the graph structure.

### Constraints

- Full graph information available
    
- Non-overlapping communities
    
- Allowed actions:
    
    - Remove intra-community edges
        
    - Add inter-community edges
        
---
### Formal Definition of Hiding

A node $u$ is considered hidden when:

$$\text{sim}(\mathcal{C}_i - u,\; \mathcal{C}_i' - u) \leq \tau$$

Where:

- $\mathcal{C}_i$: original community
    
- $\mathcal{C}_i'$: community in the modified graph $\mathcal{G}'$
    
- $\tau \in [0,1]$: similarity threshold
    
- $\text{sim}(\cdot,\cdot)$: Sørensen–Dice coefficient
    

**Note:**

- $\tau = 1$ ⇒ identical communities
    
- $\tau = 0$ ⇒ complete separation
    

---

## Original Architecture Analysis

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/model_architecture_background.png)

---
### The Node2Vec Bottleneck

- Node2Vec embeddings are computed once, on the original graph.
    
- **Hypothesis:** Embeddings become obsolete as the agent modifies the graph.
    
- **Risk:** Information leakage — the agent is guided by static structural traits.
    
---
### Why Actor–Critic (A2C)?

**Why not train only an Actor?**

$$\nabla_{\theta} J(\theta)=\mathbb{E}_{s,a}\left[\nabla_{\theta} \log \pi_{\theta}(a \mid s)\,(R_t - b)\right]$$

Without a baseline ($b$), policy gradients suffer from high variance.
---
**The Critic:**

- Learns the value function $V(s)$
    
- Enables computation of the Advantage:
    
    $$A_t = R_t - V(s_t)$$
    
- This stabilizes learning when rewards are noisy or delayed.
    
---
### Critical Finding: Structural Invariance

A deep inspection of the computational graph revealed the neural network _never_ receives the target node $u$ as input.

- **The environment:** Feeds the full graph $G$ and masks outputs so actions apply to $u$.
    
- **Result:** The policy $\pi(a \mid s)$ is identical for any target node, given the same graph. (This was validated through correspondence with the original authors.)
    
---
### The Core Problem

Standard GNNs are permutation invariant. To solve node hiding, we must explicitly break symmetry by injecting the target node identity into every stage of decision-making.

---

## Early Work: Target-Aware GNNs

To address permutation invariance, we redesigned the monolithic architecture to explicitly condition on the node being hidden.

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-ac.png)

---
### Shared GNN Embedding (Backbone)

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-backbone.png)

---
- Stack of **GATv2Conv layers** (multi-head attention with edge attributes).
    
- **Input:** Node features $X \in \mathbb{R}^{N \times d}$ and Graph structure (edge_index, edge_attr).
    
- **Output:** Node embeddings $H \in \mathbb{R}^{N \times h}$.
    
- **Why GATv2?** Native support for edge features and attention conditioned on both source and target nodes.
    
---
### Actor Network (Policy Head)
![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-actor.png)
---

- Produce one logit per node, representing action preference.
    
- Policy is explicitly conditioned on the target node.
    
- **Scoring function:**
    
    $$\ell_i = f(W_1 h_i + W_2 h_t)$$
    
    Where $h_i$ is the candidate node embedding and $h_t$ is the target node embedding.
    
- Final MLP outputs a scalar logit per node (target node logit is masked to $-\infty$).
    
---
### Critic Network (Value Head)
![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-critic.png)
---
- Estimate the state value: $V(G, t)$
    
- **Attention Pooling:** Node importance is calculated as $\alpha_i = \text{softmax}(g(h_i))$, and the graph embedding becomes $h_G = \sum_i \alpha_i h_i$.
    
- Concatenates global graph embedding with target node embedding to output a scalar value.
---
# Graph Feature Engineering for Node Hiding

---

## Feature Overview

Each node is described by **7 features**, divided into:

| Feature Type | Frequency | Purpose |
|-------------|-----------|---------|
| **Static**  | Once (`reset()`) | Persistent context (target identity, original community) |
| **Dynamic** | Every step | Tracks structural change and hiding progress |

---

## Static Features – Structural Identity

These encode the *rules of the game* and the initial graph state.

| Feature                  | Significance                       |
| ------------------------ | ---------------------------------- |
| **Is Target**            | Identifies the protected node      |
| **In Target Community**  | Defines the deception boundary     |
| **Is Original Neighbor** | Remembers the initial neighborhood |

---

## Dynamic Features

Capture the evolving relationship between nodes and the target.

| Feature                 | Significance                                                                                                                                         |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Normalized Degree**   | Tracks local density $\text{degree}(n)/\text{max_degree}(G)$<br>(for each node)                                                                      |
| **Is Current Neighbor** | Binary edge indicator                                                                                                                                |
| **Common Neighbors**    | Structural overlap with target<br>$∣N(\text{target}) \cap N(n)∣$                                                                                     |
| **Jaccard Similarity**  | Measures the probability that a neighbor of either node is a neighbor of both. $\frac{∣N(\text{target}) \cap N(n)∣}{{∣N(\text{target}) \cup N(n)∣}}$ |


---
### Proof of Concept: Initial (“Trivial”) Success

- Target-aware prototype implemented.
    
- Learned strategy: simply isolate the target node.
    
- **Results:** Success Rate: **100%**, NMI: **Low** (structure destroyed).
    
- **Conclusion:** Data flow was correct; the reward function required refinement.
    

---

## The Current Model
## Dual-Agent Actor-Critic GATv2

To further optimize performance, improve NMI, and handle larger graphs, we moved from a monolithic agent to a **Dual-Agent Architecture**.

---
### Scaling and Specialization

We split the responsibilities of graph rewiring into two specialized agents:

1. **The Adder Agent:** Specifically trained to identify and create optimal inter-community edges.
    
2. **The Deleter Agent:** Specifically trained to identify and remove critical intra-community edges.
    
---
### Action Selection Logic (The Critic's Role)

Because we have two separate agents, we must decide at each step whether to add or delete an edge.

- **Mechanism:** We look at the expected reward (state value) output by both the Adder's Critic and the Deleter's Critic.
    
- **Decision:** We choose the action from the agent whose Critic predicts the **highest value**.
    
---
### New Final Architecture

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-dual-agent.png)
_Specialization: Each agent has its own Actor and Critic heads but shares the structural understanding from the GNN backbone._

---

## Overcoming The Scaling Bottleneck

### The Bottleneck: Dynamic Features

As we scaled to larger datasets, feature computation became a critical bottleneck:

- Dynamic features (Common Neighbors, Jaccard Similarity) must be updated for every node after every action.
    
- **Complexity:** As the number of nodes and edges grows, this computation takes up more and more time relatively to the neural network pass.
    
---
### Scaling via Subgraph Optimization

To handle large graphs, we implemented two heuristic strategies to reduce the working space:
---
- **Heuristic 1: k-Neighborhood**
    
    - Extract the k-step neighborhood of the target node.
        
    - _Limitation:_ May miss critical "bridge" nodes in distant communities optimal for new connections.
---
- **Heuristic 2: Policy-Guided Subgraphing (Best Approach)**
    
    - **Initial Pass:** Perform one forward pass of both Actor networks on the full graph.
        
    - **Identification:** Identify the top nodes the agents "want" to interact with (by looking at the logits given by each agent).
        
    - **Expansion:** Take these top nodes and their entire respective communities (e.g., top 5).
        
    - **Reduction:** Construct a subgraph containing only these communities and the target node.
        
    - **Operation:** The agents now only compute dynamic features and take actions within this optimized subgraph.
        
---
## Results
---
### Single-Agent results
*(first approaches)*
---

![[F1_allDatasets_evaluation_node_hiding_greedy_multiAgents.png]]

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/figures/F1_allDatasets_evaluation_node_hiding_louvain_multiAgents.png)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/figures/F1_allDatasets_evaluation_node_hiding_walktrap_multiAgents.png)

---
### Dual-Agent results
*(current state)*
---
![[tradeoff_plot.svg]]
---
*(Some additional graphs were added in order to test the new agent's scaling capacities. Hyperparameters were tuned for each dataset, specially when considering the 2nd heuristic for large graphs, since this allows to trade off some F1 score for a much lower execution time. For static parameters, global performance remains to be tested thoroughly)*

The combination of Dual-Agents and Heuristic 2 allows the model to process graphs previously considered intractable.

---

## Future Roadmap

**Explore New Problem Variants**
- Handle overlapping communities.
    
- Merge communities instead of just hiding nodes.
    
- Introduce adding/deleting nodes as new possible actions
    
- Multi-Community Hiding (expanding dual-agent logic to protect multiple targets via MARL).
    
- Hide from a subset of nodes.
    

---

**END**