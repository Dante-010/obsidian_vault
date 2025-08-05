- $\mathcal{G} = (\mathcal{V}, \mathcal{E})$
  Grafo dirigido arbitrario, donde:
	- $\mathcal{V}$ es un conjunto de $n$ nodos.
	- $\mathcal{E} \subseteq \mathcal{V} \times \mathcal{V}$ conjunto de $m$ aristas.
	- **Opcionalmente** un conjunto de $p$ atributos de nodos.
	  Cada nodo $u \in \mathcal{V}$ se asocia con un feature vector correspondiente $x_u \in \mathbb{R}^p$
	- La estructura de conexiones de $\mathcal{G}$ se representa con una matriz binaria de adyacencia $A \in \{0, 1\}^{n \times n}$, donde $A_{u,v} = 1$ si y solo si la arista $(u, v) \in \mathcal{E}$, sino es $0$.

- $f(\cdot)$ *(en el contexto de community detection)*
  Función que toma un grafo como entrada y genera una partición de sus nodos $\mathcal{V}$ en un conjunto de comunidades **no vacías y no superpuestas** $\{\mathcal{C}_1,\ldots, \mathcal{C}_k\}$ (es decir, $f(\mathcal{G}) = \{\mathcal{C}_1,\ldots, \mathcal{C}_k\}$), con $k$ generalmente desconocido.
  
  Si queremos que las comunidades sean **superpuestas**, consideramos un vector estocástico $k$-dimensional $c_u$, donde $c_{u,i} = P(u \in \mathcal{C_i})$, y $\sum^k_{i=1} c_{u,i} = 1$. En el caso no superpuesto, $c_u$ tiene una sola entrada no nula igual a $1$ (y el resto en $0$).

  Se utiliza la notación $i^*_u = \arg \max_i(c_{u,i})$ para definir el índice de la comunidad a la cual un nodo específico $u$ pertenece en base al resultado de $f(\mathcal{G})$.
  
  En el contexto del paper, $f(\cdot)$ se trata como black-box (es decir, no nos importa el algoritmo).
  Además, se asume que quien sea que ejecute [[Community Membership Hiding]] puede ejecutar el algoritmo de detección de comunidades $f(\cdot)$, y posee información completa sobre el grafo.
- $h_{\theta}(\mathcal{G})$ 
  Función "creada" por community membership hiding, parametrizada por $\theta$, que toma como entrada el grafo original $\mathcal{G}$ y devuelve $\mathcal{G}' = (\mathcal{V}, \mathcal{E}')$, donde $u$ no pertenece a su comunidad original $\mathcal{C}_i$.
  
- 
