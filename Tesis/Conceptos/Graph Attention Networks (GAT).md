**Graph Attention Networks** (GATs) son una arquitectura de redes neuronales que opera en datos estructurados en grafos, utilizando un mecanismo de **self-attention** para computar representaciones de nodos.

Dado un grafo $G = (V, E)$ con características de nodos $\mathbf{h} = \{ \vec{h}_1, \vec{h}_2, ..., \vec{h}_N \}, \vec{h}_i \in \mathbb{R}^F$, el objetivo es calcular características actualizadas $\vec{h}_i'$.

## Capa GAT (Original)

Para un nodo $i$, la capa GAT computa:

$$
\vec{h}_i' = \sigma \left( \sum_{j \in \mathcal{N}_i} \alpha_{ij} \mathbf{W} \vec{h}_j \right)
$$

donde $\mathcal{N}_i$ es la vecindad del nodo $i$ (incluyéndose a sí mismo), $\mathbf{W}$ es una matriz de pesos entrenable y $\alpha_{ij}$ son los **coeficientes de atención**:

$$
\alpha_{ij} = \frac{\exp\left(\text{LeakyReLU}\left(\vec{a}^T [\mathbf{W}\vec{h}_i \mathbin\| \mathbf{W}\vec{h}_j]\right)\right)}{\sum_{k \in \mathcal{N}_i} \exp\left(\text{LeakyReLU}\left(\vec{a}^T [\mathbf{W}\vec{h}_i \mathbin\| \mathbf{W}\vec{h}_k]\right)\right)}
$$

Acá, $\vec{a}$ es un vector de atención entrenable, y $\mathbin\|$ denota concatenación.

### Limitación
La función de scoring de atención en GAT tiene la forma:
$$
e(\vec{h}_i, \vec{h}_j) = \vec{a}^T \text{LeakyReLU}(\mathbf{W}[\vec{h}_i \mathbin\| \vec{h}_j])
$$
Como $\text{LeakyReLU}$ y la transformación lineal $\vec{a}^T$ se aplican **después** de la concatenación, la función de scoring **no es suficientemente expresiva**. El orden de las operaciones puede hacer que el mecanismo de atención sea *estático* respecto al nodo query (consulta).

---

## GATv2Conv: Atención Dinámica en Grafos

**GATv2Conv** (Brody et al., 2021) modifica el orden de las operaciones para crear un mecanismo de atención **dinámico** y **estrictamente más expresivo**.

### El Cambio
GATv2 computa los coeficientes de atención como:

$$
e(\vec{h}_i, \vec{h}_j) = \vec{a}^T \text{LeakyReLU}\left( \mathbf{W}[\vec{h}_i \mathbin\| \vec{h}_j] \right)
$$

¿Parece igual? La clave está en el *orden de las transformaciones lineales*. La implementación real se puede ver como:

$$
e(\vec{h}_i, \vec{h}_j) = \vec{a}^T \text{LeakyReLU}\left( \mathbf{W}_1 \vec{h}_i + \mathbf{W}_2 \vec{h}_j \right)
$$

Más precisamente, cambiando el orden de las operaciones de GAT:
`Lineal(concat) → LeakyReLU → Lineal`  
a:  
`Lineal(query) + Lineal(key) → LeakyReLU → Lineal`

### ¿Por qué GATv2 rinde mejor?

1.  **Atención Dinámica**: En GAT, para un conjunto fijo de keys $\{\vec{h}_j\}$, el ranking de los scores de atención $\alpha_{ij}$ es **monótono** y puede ser el mismo para todas las queries $\vec{h}_i$. GATv2 permite que el ranking cambie por cada query → atención verdaderamente *dinámica*.
2.  **Aproximador Universal**: La función de scoring de GATv2 es una función continua del query y el key, permitiéndole aproximar cualquier patrón de atención deseado.
3.  **Arregla la Atención Estática**: GAT puede fallar en problemas simples donde la atención óptima depende del nodo query (ej: el benchmark sintético "Key-Query"). GATv2 resuelve estos casos.
4.  **Mejora Empírica**: Suele mostrar mejor rendimiento en benchmarks de clasificación de nodos y grafos, especialmente cuando importan relaciones complejas que dependen del query.

### Resumen de Fórmulas

**Coeficiente de atención GATv2**:
$$
\alpha_{ij} = \frac{\exp\left(\vec{a}^T \text{LeakyReLU}\left(\mathbf{W}[\vec{h}_i \mathbin\| \vec{h}_j]\right)\right)}{\sum_{k \in \mathcal{N}_i} \exp\left(\vec{a}^T \text{LeakyReLU}\left(\mathbf{W}[\vec{h}_i \mathbin\| \vec{h}_k]\right)\right)}
$$
*Ojo*: La diferencia es que $\mathbf{W}$ se aplica *antes* de la no linealidad, y $\vec{a}$ después, haciendo que toda la función sea un MLP de 2 capas sobre las características concatenadas.