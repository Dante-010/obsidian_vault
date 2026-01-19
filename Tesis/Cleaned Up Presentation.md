# Graph Community Membership Hiding  
## Analysis, Replication, and Novel Architectures

---

## Original Project Scope

- **Goal**

	“Hide” a target node $u$ from a community detection algorithm treated as a **black box**.
	&nbsp;

- **Method**

	A **Reinforcement Learning agent (A2C)** that modifies the graph structure.
	&nbsp;

- **Constraints**

	- Full graph information available  
	- Non-overlapping communities  
	- **Allowed actions:**  
	  - Remove intra-community edges  
	  - Add inter-community edges  

---

## Formal Definition of *Hiding*

A node $u$ is considered *hidden* when:

$\text{sim}(\mathcal{C}_i - u,\; \mathcal{C}_i' - u) \leq \tau$

&nbsp;
&nbsp;

Where:
- $\mathcal{C}_i$: original community  
- $\mathcal{C}_i'$: community in the modified graph $\mathcal{G}'$  
- $\tau \in [0,1]$: similarity threshold  
- $\text{sim}(\cdot,\cdot)$: Sørensen–Dice coefficient  

**Note:**  
- $\tau = 1$ ⇒ identical communities  
- $\tau = 0$ ⇒ complete separation  

---

## Original Architecture

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/model_architecture_background.png)

---

### The Node2Vec Bottleneck

- Node2Vec embeddings are computed **once**, on the original graph  
- **Hypothesis:** embeddings become obsolete as the agent modifies the graph  
- **Risk:** information leakage — the agent is guided by static structural traits  

---

## Why Actor–Critic (A2C)?

Why not train only an Actor?

$\nabla_{\theta} J(\theta)=\mathbb{E}_{s,a}\left[\nabla_{\theta} \log \pi_{\theta}(a \mid s)\,(R_t - b)\right]$

Without a baseline ($b$), policy gradients suffer from **high variance**. 

&nbsp;
&nbsp;

**The Critic**:

- Learns the value function $V(s)$  
- Enables computation of the **Advantage**:

$$
A_t = R_t - V(s_t)
$$
&nbsp;
&nbsp;

This stabilizes learning when rewards are noisy or delayed.

---

## Critical Finding: Structural Invariance

- A deep inspection of the computational graph revealed the neural network **never receives the target node $u$ as input**
- The environment:
  1. Feeds the full graph $G$
  2. Masks outputs so actions apply to $u$
- **Result:** the policy $\pi(a \mid s)$ is *identical* for any target node, given the same graph

> This was validated through correspondence with the original authors.

---

## The Core Problem
- Standard GNNs are **permutation invariant**.

- To solve node hiding, we must explicitly **break symmetry** by injecting the *target node identity* into every stage of decision making.

---

# The New Model  
### From Global to Target-Aware Reasoning

A redesigned architecture that explicitly conditions on the node being hidden.

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-ac.png)

---
##### Shared GNN Embedding
---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-backbone.png)

---
### Backbone Architecture

Stack of **GATv2Conv** layers  
Multi-head attention with edge attributes  

Each layer includes:
- Attention-based message passing  
- LayerNorm  
- ELU + Dropout (except final layer)  

---
 **Input**
- Node features: $X \in \mathbb{R}^{N \times d}$  
- Graph structure: $(\text{edge_index}, \text{edge_attr})$  
&nbsp;&nbsp;

**Output**
- Node embeddings: $H \in \mathbb{R}^{N \times h}$  

---

## Why GATv2?

- Attention conditioned on **both source and target nodes**  
- Native support for **edge features**  
- More expressive than GCN / GraphSAGE  
- Well-suited for relational decision making  

---

#### Actor Network (Policy Head)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-actor.png)

---

### Actor Objective

Produce **one logit per node**, representing action preference.

- Action space = graph nodes  
- Policy is explicitly conditioned on the **target node**

Scoring function:

$$
\ell_i = f(W_1 h_i + W_2 h_t)
$$

Where:
- $h_i$: candidate node embedding  
- $h_t$: target node embedding  

---

- Separate linear projections:
  - `lin_node(h_i)`
  - `lin_target(h_t)`
- Broadcasting sum enables efficient interaction  
  (avoids concatenating to $[N \times 2H]$ tensors)
- Final MLP outputs a **scalar logit per node**
- Logit of the target node is masked to $-\infty$

External masking ensures only **valid actions**:
- Add edges to “outside” nodes  
- Remove edges from “inside” nodes  

---

#### Critic Network (Value Head)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-critic.png)

---

### Critic Objective

Estimate the state value:

$$
V(G, t)
$$
&nbsp;
- **Architecture**
	1. Attention pooling over node embeddings  
	2. Concatenate:
	   - Global graph embedding  
	   - Target node embedding  
	1. MLP → scalar value  

---

### Critic – Attention Pooling

Node importance:
$$
\alpha_i = \text{softmax}(g(h_i))
$$

Graph embedding:
$$
h_G = \sum_i \alpha_i h_i
$$
&nbsp;

- **Properties**
	- Differentiable  
	- Permutation invariant  
	- Focuses on value-relevant substructures  

---

## Full Forward Pass

1. Encode graph using shared GNN  
2. Actor head → action logits  
3. Critic head → state value  

```text
		(x, edge_index, edge_attr, target_node)
					        ↓
					Shared GNN Encoder
					        ↓
			   Node Embeddings + target node
					 ↙            ↘
				  Actor          Critic
				 (logits)       (value)
```

---

## Design Advantages

- Shared encoder improves generalization  
- Explicit target conditioning  
- Actor: local, node-level decisions  
- Critic: global reasoning via learned pooling  
- Fully end-to-end and differentiable  

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

## Results

Performance comparison against previous approaches.

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/F1_allDatasets_evaluation_node_hiding_greedy_multiAgents.png)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/F1_allDatasets_evaluation_node_hiding_louvain_multiAgents.png)

---

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/F1_allDatasets_evaluation_node_hiding_walktrap_multiAgents.png)

---

The proposed agent outperforms prior methods in nearly all settings, often by a **large margin**.

*(Very large datasets are still under evaluation.)*

---

## Proof of Concept

**Initial (“Trivial”) Success**

- Target-aware prototype implemented  
- Learned strategy: **isolate the target node**
- Results:
  - Success Rate: **100%**
  - NMI: **Low** (structure destroyed)

**Conclusion:**  
Data flow is correct; the reward function required refinement.

---

### Current State

- Agent balances **Success Rate** and **NMI**
- Consistently outperforms baselines  
- Performance does not appear to degrade strongly with graph size  (*thorough testing remains*)

---

## Future Roadmap

1. **Scale up evaluation**
   - Graphs with $>10{,}000$ nodes  
   - $>20{,}000$ edges  

2. **Explore new variants**
   - Merge communities  
   - Hide multiple nodes (MARL)  
   - Hide from a *subset* of nodes  
   - Overlapping communities  
   - Transferability across detection algorithms
   - ...

---

3. **Test new graph families**
   - Chains, trees  
   - $k$-regular graphs  
   - Erdős–Rényi random graphs 
   - ...

4. **Optimize the agent**
   - Extensive hyperparameter search  
   - Specialized models for different regimes  

---

# END
