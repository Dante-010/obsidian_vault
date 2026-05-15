
# Graph Community Membership Hiding

### Analysis, Replication, and Dual-Agent Architectures

---

## What Are We Talking About?

**The big picture:** Social networks, citation graphs, protein interactions — all of these can be modeled as graphs. Within those graphs, algorithms can automatically detect *communities*: dense clusters of nodes that tend to be more connected to each other than to the rest of the network.

**The problem:** Sometimes you *don't* want to be detected as part of a community. Think political affiliation, religious groups, medical conditions inferred from who you interact with — all of this can be inferred just from graph structure, without reading a single message.

**Our goal:** Make a target node "invisible" to community detection algorithms, by making minimal, strategic changes to the graph.

---

## Original Project Scope

### Goal

"Hide" a target node $u$ from a community detection algorithm treated as a **black box** (we don't get to look inside the algorithm — we can only query it).

### Method

A Reinforcement Learning agent (A2C) that learns to modify the graph structure.

### Constraints

- Full graph information available
    
- Non-overlapping communities (each node belongs to exactly one community)
    
- Allowed actions:
    
    - Remove edges between $u$ and nodes in its own community
        
    - Add edges between $u$ and nodes in *other* communities
        
---

### Formal Definition of Hiding

A node $u$ is considered hidden when:

$$\text{sim}(\mathcal{C}_i - u,\; \mathcal{C}_i' - u) \leq \tau$$

Where:

- $\mathcal{C}_i$: original community of $u$
    
- $\mathcal{C}_i'$: community assigned to $u$ in the modified graph $\mathcal{G}'$
    
- $\tau \in [0,1]$: similarity threshold ("how different is different enough?")
    
- $\text{sim}(\cdot,\cdot)$: Sørensen–Dice coefficient (measures set overlap)
    

**Intuition for $\tau$:**

- $\tau = 0$ ⇒ the new community must share *no* members with the original — extremely hard
    
- $\tau = 1$ ⇒ any change in community membership counts — trivially easy
    
- We use $\tau \in \{0.3, 0.5, 0.8\}$ for evaluation
    

---

## A Crash Course on Reinforcement Learning

*(Skip this if you already know RL — but it's worth a quick read even if you do.)*

---

### The Core Idea

Reinforcement Learning is a framework for training an **agent** to make sequential decisions in an **environment** in order to maximize some notion of **cumulative reward**.

The agent is not told what to do. It is not given labeled examples. It simply tries things, observes what happens, and gradually figures out what works.

> Think of training a dog: you don't hand it a manual. You reward good behavior and correct bad behavior, and eventually the dog learns.

---

### The Key Components

| Term | What It Means | In Our Problem |
|------|--------------|----------------|
| **Agent** | The decision-maker | Our neural network |
| **Environment** | The world the agent interacts with | The graph + community detection algorithm |
| **State** $s$ | A snapshot of the current situation | The current graph structure and node features |
| **Action** $a$ | A choice the agent can make | Add or remove a specific edge |
| **Reward** $r$ | Feedback signal (scalar) | +1 for hiding success, −1 for budget exhaustion, small step costs |
| **Episode** | One full run from start to end | Starting from the original graph, applying actions until $u$ is hidden or budget runs out |

---

### The RL Loop

At every step $t$:

1. Agent observes the current state $s_t$
2. Agent chooses an action $a_t$ according to its **policy** $\pi(a \mid s)$
3. Environment transitions to a new state $s_{t+1}$
4. Agent receives a scalar reward $r_t$
5. Repeat until the episode ends

The agent's goal is to find a policy $\pi^*$ that maximizes the **expected return**:

$$G_t = \sum_{k=0}^{\infty} \gamma^k r_{t+k}$$

where $\gamma \in [0,1]$ is a **discount factor** that makes future rewards worth slightly less than immediate ones (a reward now is better than the same reward later).

---

### What Is a Policy?

A **policy** $\pi(a \mid s)$ is a probability distribution over actions given a state. It tells the agent: "when you see *this* situation, here is how likely you are to take each possible action."

- A **deterministic** policy always picks the same action for a given state.
- A **stochastic** policy samples from a distribution — useful during training because it encourages **exploration**.

Our agent's policy is parametrized by a neural network with weights $\theta$. We write $\pi_\theta(a \mid s)$.

---

### The Exploration vs. Exploitation Tradeoff

This is one of the most fundamental tensions in RL:

- **Exploitation:** Do what you already know works well.
- **Exploration:** Try new things — maybe something better exists.

If you exploit too much, you get stuck in a local optimum. If you explore too much, you waste time on random actions.

> Analogy: you found a restaurant you like. Do you keep going there (exploitation), or try new places hoping to find something even better (exploration)?

In practice, stochastic policies naturally balance this: the agent tends toward good actions but still occasionally tries others.

---

### Policy Gradient Methods

How do we actually train a policy network?

We want to adjust $\theta$ to increase the probability of actions that led to high rewards. The **policy gradient theorem** gives us the gradient to do exactly that:

$$\nabla_{\theta} J(\theta) = \mathbb{E}_{s,a}\left[\nabla_{\theta} \log \pi_{\theta}(a \mid s) \cdot G_t\right]$$

Intuitively: if an action led to a high return $G_t$, increase its probability. If it led to a low return, decrease it.

The problem? $G_t$ can have **very high variance** — the same action in similar situations can lead to very different outcomes depending on what happens later. This makes learning slow and unstable.

---

### Why Actor–Critic (A2C)?

**Why not train only an Actor (policy)?**

High variance in the gradient estimate makes training unstable. The fix is to subtract a **baseline** $b$ from the return:

$$\nabla_{\theta} J(\theta)=\mathbb{E}_{s,a}\left[\nabla_{\theta} \log \pi_{\theta}(a \mid s)\,(G_t - b)\right]$$

Subtracting a baseline doesn't bias the gradient (you can verify this mathematically), but it *dramatically* reduces variance.

**What is the best baseline?** The **value function** $V(s)$ — the expected return from state $s$ under the current policy.

**The Critic:**

- A second neural network that learns to estimate $V(s)$
    
- Enables computation of the **Advantage**:
    
    $$A_t = G_t - V(s_t)$$
    
    *"This action was better (or worse) than what I expected from this state."*
    
- This is far more informative than the raw return: instead of asking "was this good?", we ask "was this better than average?"
    
- This stabilizes learning when rewards are noisy or delayed.

**Summary:** The **Actor** decides what to do; the **Critic** evaluates how good the current situation is. They train together — the Actor improves its policy, the Critic improves its value estimates.

---

### Why RL for This Problem?

You might wonder: why not just formulate this as a supervised learning or optimization problem?

- **No labeled data:** We don't know in advance which edges to add/remove for each graph. There is no "correct answer" to learn from directly.
    
- **Sequential decisions:** Each action changes the graph, which changes what the best next action is. This is inherently a multi-step problem.
    
- **Black-box environment:** We treat the community detection algorithm as opaque — we can only observe its output. Gradient-based approaches require access to the algorithm's internals.
    
RL is a natural fit: it handles sequential decisions, requires no labeled data, and only needs to *query* the environment rather than differentiate through it.

---

## Original Architecture Analysis

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/model_architecture_background.png)

The original agent of Bernini et al. (2024) uses a GCN (Graph Convolutional Network) to encode the graph state, with node2vec embeddings as initial features.

---

### The Node2Vec Bottleneck

- **Node2Vec** learns node embeddings by simulating random walks on the graph, then training a skip-gram model — similar to word2vec but for graphs.
    
- These embeddings are computed **once**, on the original graph, before training begins.
    
- **Hypothesis:** As the agent rewires the graph, the embeddings become stale. The model is making decisions based on a description of a graph that no longer exists.
    
- **Risk:** The agent is guided by structural information that no longer reflects the current state.
    
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

A deep inspection of the computational graph revealed the neural network **never** receives the target node $u$ as input.

- **The environment:** Feeds the full graph $\mathcal{G}$ and masks outputs so that actions apply to $u$.
    
- **The consequence:** The policy $\pi(a \mid s)$ is **identical for any target node**, given the same graph.

Why? Standard GNNs are **permutation-invariant**: if you relabel the nodes, the output doesn't change. So if the network doesn't know *which* node is the target, it can't distinguish between different targets.

*(This was validated through correspondence with the original authors.)*

---

### The Core Problem

Standard GNNs are permutation invariant. To solve node hiding, we must explicitly break symmetry by injecting the target node identity into every stage of decision-making.

> Analogy: imagine asking someone to protect a specific person in a crowd, but not telling them who that person is. They might protect everyone a little bit, but won't specialize where it matters.

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

    **Why GATv2 instead of a plain GCN?** Standard GCNs aggregate neighbor information with fixed, pre-computed weights. GATv2 learns *dynamic* attention weights: it computes a score for every (source, target) node pair at each layer, so the model learns which neighbors to pay attention to. This is especially useful here because the importance of a neighbor can depend on whether it is inside or outside the target's community.
    
- **Input:** Node features $X \in \mathbb{R}^{N \times d}$ and Graph structure (edge\_index, edge\_attr).
    
- **Output:** Node embeddings $H \in \mathbb{R}^{N \times h}$, one $h$-dimensional vector per node that encodes its structural role in the graph.
    
- **Why GATv2?** Native support for edge features and attention conditioned on both source and target nodes. Unlike GATv1, the attention is truly dynamic — it depends on both endpoints of an edge rather than just one.
    
---

### Actor Network (Policy Head)

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-actor.png)

---

- Produce one logit per node, representing action preference.
    
- Policy is **explicitly conditioned on the target node**.
    
- **Scoring function:**
    
    $$\ell_i = f(W_1 h_i + W_2 h_t)$$
    
    Where $h_i$ is the candidate node's embedding and $h_t$ is the target node's embedding. By mixing the two, the scoring function asks: "how useful is node $i$ for *this particular target*?"
    
- Final MLP outputs a scalar logit per node. The target node's own logit is masked to $-\infty$ (we can't add or remove an edge from a node to itself).

- A **softmax** over valid logits gives the action probability distribution $\pi(a \mid s)$.
    
---

### Critic Network (Value Head)

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-diagram-critic.png)

---

- Estimates the state value: $V(\mathcal{G}, u)$ — "how good is the current graph state for hiding $u$?"
    
- **Attention Pooling:** We can't just average all node embeddings (that would ignore the target). Instead, we compute a weighted average — a node's importance is $\alpha_i = \text{softmax}(g(h_i))$ — giving us a compact graph-level representation $h_G = \sum_i \alpha_i h_i$.
    
- Concatenate $h_G$ (global graph state) with $h_t$ (target-specific context) and pass through an MLP to output a scalar value estimate.

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

These encode the *rules of the game* and the initial graph state. They are computed once at the start of each episode and don't change.

| Feature                  | Significance                       |
| ------------------------ | ---------------------------------- |
| **Is Target**            | A binary flag: "this is the node we're hiding" — this is the key information that breaks permutation invariance |
| **In Target Community**  | Defines the deception boundary — nodes with this flag set are the ones we want to disconnect $u$ from |
| **Is Original Neighbor** | Remembers the initial neighborhood — helps the agent distinguish between edges it removed and edges that were never there |

---

## Dynamic Features

Capture the *evolving* relationship between each node and the target, updated after every action.

| Feature                 | Significance |
| ----------------------- | ------------ |
| **Normalized Degree**   | $\text{degree}(n)/\text{max\_degree}(\mathcal{G})$ — tracks local density changes as edges are added/removed |
| **Is Current Neighbor** | Binary: is there currently an edge between this node and the target? Tracks the agent's own actions |
| **Common Neighbors**    | $|N(\text{target}) \cap N(n)|$ — nodes with many common neighbors with $u$ are structurally close to it |
| **Jaccard Similarity**  | $\frac{|N(\text{target}) \cap N(n)|}{|N(\text{target}) \cup N(n)|}$ — normalized version of common neighbors; between 0 (no overlap) and 1 (identical neighborhoods) |

---

### Proof of Concept: Initial ("Trivial") Success

- Target-aware prototype implemented and trained.
    
- Learned strategy: simply **isolate the target node** — remove all its intra-community edges and add inter-community ones indiscriminately.
    
- **Results:** Success Rate: **100%**, NMI: **Low** (structure is destroyed).
    
- **Conclusion:** The information flow was correct (the agent did learn to target $u$), but the reward function needed refinement to penalize structural damage.
    

---

## The Current Model: Dual-Agent Actor-Critic GATv2

To further optimize performance, improve NMI, and handle larger graphs, we moved from a monolithic agent to a **Dual-Agent Architecture**.

---

### Why Dual Agents?

The rewiring task has two fundamentally different sub-problems:

- **Deciding which edges to add** (connecting $u$ to a new community) requires thinking about which external community would make the best "new home" for $u$.
    
- **Deciding which edges to remove** (cutting ties to the current community) requires thinking about which internal connections are most responsible for $u$'s current community assignment.

These are genuinely different reasoning tasks. A single agent has to learn both simultaneously, which may limit its performance on each. Separating them gives each agent a narrower, more focused optimization target.

---

### Scaling and Specialization

We split the responsibilities of graph rewiring into two specialized agents:

1. **The Adder Agent:** Specifically trained to identify and create optimal inter-community edges.
    
2. **The Deleter Agent:** Specifically trained to identify and remove critical intra-community edges.
    
Each agent has its own Actor and Critic heads, trained independently on their respective action subspace.

---

### Action Selection Logic (The Critic's Role)

Because we have two separate agents, we must decide at each step whether to add or delete an edge.

- **Mechanism:** We query *both* agents' Critics, getting two value estimates: $V_{\text{add}}(\mathcal{G}, u)$ and $V_{\text{del}}(\mathcal{G}, u)$.
    
- **Decision:** We choose the action from the agent whose Critic predicts the **highest value** — i.e., whichever agent thinks the current situation favors its type of action more.

$$\text{act} = \arg\max_{a \in \{\text{add}, \text{del}\}} V_a(\mathcal{G}, u)$$

This is elegant: the Critics, which were originally designed to stabilize training (the A2C trick), are now also used to make architectural-level decisions.

---

### New Final Architecture

![Image](file:///home/dante/Documents/obsidian_vault/Tesis/RECURSOS/images/mermaid-dual-agent.png)

*Specialization: Each agent has its own Actor and Critic heads but shares the structural understanding from the GNN backbone.*

---

## Overcoming The Scaling Bottleneck

### The Bottleneck: Dynamic Features

As we scaled to larger datasets, feature computation became a critical bottleneck:

- Dynamic features (Common Neighbors, Jaccard Similarity) must be recomputed for **every node** after every action.
    
- **Complexity:** $O(|V| \cdot |E|)$ per step in the worst case. On graphs with tens of thousands of nodes and edges, this dominates the runtime — the neural network pass becomes the cheap part.
    
---

### Scaling via Subgraph Optimization

To handle large graphs, we implemented two heuristic strategies to reduce the working space:

---

- **Heuristic 1: $k$-Neighborhood**
    
    - Extract the $k$-step neighborhood of the target node — i.e., all nodes reachable in at most $k$ hops from $u$.
        
    - Intuitive and cheap to compute.
        
    - *Limitation:* May miss critical "bridge" nodes in distant communities that would be ideal for new connections. If the best external community is far away in the graph, this heuristic will never find it.

---

- **Heuristic 2: Policy-Guided Subgraphing (Best Approach)**
    
    - **Initial Pass:** Perform *one* forward pass of both Actor networks on the full graph — cheap, no feature updates needed.
        
    - **Identification:** Look at the raw logits: which nodes does each agent most want to interact with?
        
    - **Expansion:** Take the top-$k$ nodes by logit, then expand to their *entire communities* (not just the nodes themselves).
        
    - **Reduction:** Build a subgraph containing only those communities and the target node.
        
    - **Operation:** All subsequent dynamic feature updates and rewiring actions happen inside this compact subgraph.
        
    - **Key insight:** We use the agents' own preferences to guide subgraph selection — so we're not throwing away relevant information at random; we're keeping exactly what the agents care about.
        
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

*(Some additional graphs were added in order to test the new agent's scaling capacities. Hyperparameters were tuned for each dataset, especially when considering Heuristic 2 for large graphs, since this allows us to trade off some F1 score for a much lower execution time. For static parameters, global performance remains to be tested thoroughly.)*

The combination of Dual-Agents and Heuristic 2 allows the model to process graphs previously considered intractable.

---

## Future Roadmap

**Explore New Problem Variants**

- Handle **overlapping communities** — nodes can belong to multiple communities, so hiding is harder (you must weaken multiple memberships simultaneously).
    
- **Merge communities** instead of just hiding nodes.
    
- Introduce **adding/deleting nodes** as new possible actions.
    
- **Multi-Community Hiding:** protect multiple targets simultaneously via Multi-Agent RL (MARL) — each target gets its own agent, and they must cooperate without destroying each other's progress.
    
- **Hide from a subset of nodes** — partial observability variant.
    

---

**END**
