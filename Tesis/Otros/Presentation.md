---
theme: black
transition: slide
highlightTheme: zenburn
---

# Graph Community Membership Hiding
## Analysis, Replication, and Novel Architectures

---

## Original Project Scope

**The Goal:**
"Hide" a target node $u$ from a community detection algorithm (considered a *Black-box*).

**The Method:**
Reinforcement Learning Agent (**A2C**) modifying graph structure.

**Constraints:**
* Full graph information available.
* Non-overlapping communities.
* **Allowed Actions:** Remove intra-community edges / Add inter-community edges.

---

## Formal Definition of "Hiding"

Node $u$ is considered hidden when:

$$
sim(\mathcal{C}_i - u, \mathcal{C}_i' - u) \leq \tau
$$

* $\mathcal{C}_i$: Original community.
* $\mathcal{C}_i'$: New community in $\mathcal{G}'$.
* $\tau \in [0, 1]$: Similarity threshold.
* $sim(\cdot, \cdot)$: Sørensen-Dice coefficient.

Note:
A threshold of 1 implies identical communities. 0 requires complete separation.

---
## The Original Architecture

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/model_architecture_background.png)

---

**The Node2Vec Bottleneck:**
* Embeddings are calculated **once** on the original graph.
* **Hypothesis:** They become obsolete as the agent modifies the graph.
* **Risk:** Information Leakage (tells the agent what to hide based on static traits).

---

## Why Actor-Critic (A2C)?

Why not just train an Actor?

$\nabla_{\theta} J(\theta) = \mathbb{E}_{s,a} \left[ \nabla_{\theta} \log \pi_{\theta}(a \mid s) \, (R_t - b) \right]$

Without a baseline, gradients have high variance.

**The Critic:**
Estimates the value function $V(s)$ to calculate the **Advantage**:

$$
A_t = R_t - V(s_t)
$$

This stabilizes training against noisy total rewards.

---
### Critical Finding: Structural Invariance
Deep analysis of the tensor flow revealed the Neural Network **does not receive the target node $u$ as input**.

1.  Input is the full Graph $G$.
2.  Environment "masks" outputs to act on $u$.
3.  **Result:** Policy $\pi(a|s)$ is identical for *any* target node given the same graph.
> Validated via correspondence with original authors.

---
## The Core Problem

Standard GNNs are **Permutation Invariant**.


We must explicitly **break symmetry** by injecting the target's identity into every layer of the decision-making process.

---
# The New Model
#### Moving from Global to Local-Aware
Architecture designed to hide a specific node from community detection.

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-ac.png)

---

# Shared GNN Embedding
    
---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-backbone.png)

---
- Stack of **GATv2Conv** layers
    
- Multi-head attention + edge attributes
    
- Per-layer:
    
    - Attention-based message passing
        
    - LayerNorm
        
    - ELU + Dropout (except last layer)
        

**Input:**

- Node features $(X \in \mathbb{R}^{N \times d})$
    
- Graph structure $((\text{edge_index}, \text{edge_attr}))$
    

**Output:**

- Node embeddings $(H \in \mathbb{R}^{N \times h})$

---

## Why GATv2?

- Attention coefficients are **conditioned on both source and target nodes**
    
- Naturally incorporates **edge features**
    
- More expressive than GCN / GraphSAGE in heterogeneous graphs
    
- Well-suited for decision-making over relational structure
    

---

## Actor Network (Policy Head)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-actor.png)

---

**Goal:** produce a logit for each node = action preference

Key idea:

- Action space = nodes of the graph
    
- Policy is conditioned on a **target node**
    

Scoring function:
$\ell_i = f(W_1 h_i + W_2 h_t) $

where:

- $(h_i)$: embedding of candidate node
    
- $(h_t)$: embedding of target node
    

---

## Actor – Implementation Details

- Separate linear projections:
    
    - `lin_node(h_i)`
        
    - `lin_target(h_t)`
        
- Broadcasting sum implements a bilinear-style interaction efficiently (instead of concatenating target's embedding to each other node's embedding, which leaves you with a vector of size $[N \times 2H]$, which is very inefficient)
    
- Final MLP produces **one scalar logit per node**
    
- Logit of the target node is masked to $(-\infty)$
    

Nodes are masked externally only to valid actions (connect to "outside" node, remove "inside" nodes)

---
## Critic Network (Value Head)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-critic.png)

---

**Goal:** estimate $(V(G, t))$

Structure:

1. Attention pooling over node embeddings
    
2. Concatenate:
    
    - Global graph embedding
        
    - Target node embedding
        
3. MLP → scalar value
    

---

## Critic – Attention Pooling

Node-level importance: $ \alpha_i = \text{softmax}(g(h_i)) $

Graph embedding: $h_G = \sum_i \alpha_i h_i $

Properties:

- Differentiable, permutation-invariant
    
- Allows the critic to focus on value-relevant substructures
    

---

## Full Forward Pass

1. Encode graph with shared GNN
    
2. Actor head → action logits
    
3. Critic head → state value
    

```text
(x, edge_index, edge_attr, target_node)
        ↓
Shared GNN Encoder
        ↓
   node embeddings
     ↙            ↘
  Actor         Critic
(logits)       (value)
```

---

## Design Advantages

- Shared encoder reduces variance and improves generalization
    
- Explicit conditioning on target node
    
- Actor: local (per-node) decisions
    
- Critic: global reasoning with learned pooling
    
- Fully end-to-end and differentiable
    

---

# Graph Feature Engineering for Node Hiding

---

## Architecture Overview

The agent perceives the graph through **7 distinct features** for every node. These are categorized into **Static** (fixed at the start of the episode) and **Dynamic** (re-calculated after every edge modification).

|Feature Type|Calculation Frequency|Purpose|
|---|---|---|
|**Static**|Once (in `reset()`)|Provides persistent context (Target identity, Community).|
|**Dynamic**|Every Step|Tracks the "hiding" progress and structural changes.|

---

## 1. Static Features (Structural Identity)

These features help the GNN understand the "Rules of the Game"—who the target is and what the original structure looked like.

---

| Feature                  | Significance                                          |
| ------------------------ | ----------------------------------------------------- |
| **Is Target**            | Identifies the node the agent is protecting.          |
| **In Target Community**  | Defines the "Deception" boundary.                     |
| **Is Original Neighbor** | Remembers the initial state before any modifications. |

---
## 2. Dynamic Features

These track the immediate neighborhood of the target and the overall network density as edges are added or removed.

---

| **Feature**             | **Significance**                                                     |
| ----------------------- | -------------------------------------------------------------------- |
| **Normalized Degree**   | Tracks local density; identifies potential hubs or central nodes.    |
| **Is Current Neighbor** | Direct binary indicator of edges connected to the target node.       |
| **Common Neighbors**    | Measures structural overlap between a node and the target.           |
| **Jaccard Similarity**  | Normalized overlap; key metric for tracking "Deception" or blending. |

---
## Results

Comparing the new agent with previous models

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/F1_allDatasets_evaluation_node_hiding_greedy_multiAgents.png)

---
![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/F1_allDatasets_evaluation_node_hiding_louvain_multiAgents.png)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/F1_allDatasets_evaluation_node_hiding_walktrap_multiAgents.png)

---
The new agent outperforms the other ones in almost all cases, most times by a considerable margin. 

(very large datasets remain to be tested)

---
## Current Status & Proof of Concept

**The "Trivial" Success:**
* Developed a prototype with target-awareness.
* **Result:** It learned to delete *all* edges connected to $u$ (Isolation).
* **Metrics:**  Success Rate: 100% (Technically hidden).
    * NMI (Normalized Mutual Information): Low (Destroys structure).
* **Takeaway:** The data flow works; the objective function needs tuning.

---

**Current state**:
- The agent know how to balance **Success Rate** and **Normalized Mutual Information**.
- Outperforms other models and performance does not decay heavily with graph size.
---

## Future Roadmap

1.  **Test larger datasets:**
    * Test datasets with a large number of nodes ($>5000$ at least) and edges ($> 10.000$ at least).
2.  **Try new variants:**
    * Try **joining** two communities, **hiding multiple nodes** (*MARL*) at the same time, hide the node from a **set of other nodes**, **overlapping communities**, etc...
    * Try different detection algorithms in order to find results about *transferability*.

---

3.  **Try new graphs:**
    * Experiment with more artificial and "special" graphs, like chains, trees, k-regular graphs, random graphs (**Erdős–Rényi**).
4.  **Optimize the agent:**
    * Perform a thorough hyperparameter search, and find models for different cases instead of a *one-for-all* model.

---

# END

---