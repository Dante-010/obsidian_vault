## Definiciones y notación
### $\mathcal{G} = (\mathcal{V}, \mathcal{E})$
Grafo dirigido arbitrario, donde:
	- $\mathcal{V}$ es un conjunto de $n$ nodos.
	- $\mathcal{E} \subseteq \mathcal{V} \times \mathcal{V}$ conjunto de $m$ aristas.
	- **Opcionalmente** un conjunto de $p$ atributos de nodos.
	  Cada nodo $u \in \mathcal{V}$ se asocia con un feature vector correspondiente $x_u \in \mathbb{R}^p$
	- La estructura de conexiones de $\mathcal{G}$ se representa con una matriz binaria de adyacencia $A \in \{0, 1\}^{n \times n}$, donde $A_{u,v} = 1$ si y solo si la arista $(u, v) \in \mathcal{E}$, sino es $0$.

### $f(\cdot)$ *(en el contexto de community detection)*
Función que toma un grafo como entrada y genera una partición de sus nodos $\mathcal{V}$ en un conjunto de comunidades **no vacías y no superpuestas** $\{\mathcal{C}_1,\ldots, \mathcal{C}_k\}$ (es decir, $f(\mathcal{G}) = \{\mathcal{C}_1,\ldots, \mathcal{C}_k\}$), con $k$ generalmente desconocido.
  
Si queremos que las comunidades sean **superpuestas**, consideramos un vector estocástico $k$-dimensional $c_u$, donde $c_{u,i} = P(u \in \mathcal{C_i})$, y $\sum^k_{i=1} c_{u,i} = 1$. En el caso no superpuesto, $c_u$ tiene una sola entrada no nula igual a $1$ (y el resto en $0$).

Se utiliza la notación $i^*_u = \arg \max_i(c_{u,i})$ para definir el índice de la comunidad a la cual un nodo específico $u$ pertenece en base al resultado de $f(\mathcal{G})$.
### $h_{\theta}(\mathcal{G})$ 
Función "creada" por community membership hiding, parametrizada por $\theta$, que toma como entrada el grafo original $\mathcal{G}$ y devuelve $\mathcal{G}' = (\mathcal{V}, \mathcal{E}')$, donde $u$ no pertenece a su comunidad original $\mathcal{C}_i$.

### $\mathcal{C}_i'$
Comunidad asignada a $u$ luego de correr $f(\mathcal{G})$.

### $sim(\cdot, \cdot)$
Función de similaridad entre dos comunidades $\mathcal{C}$ y  $\mathcal{C}'$. 
La idea es considerar $sim(\mathcal{C}_i - u, \mathcal{C}_i' - u) \leq \tau$, donde $\tau \in [0, 1]$. Se excluye $u$ puesto que por definición, pertenece a ambas comunidades. $\tau = 0$ representa el escenario más estricto, donde ambas comunidades deben ser **completamente** distintas. $\tau = 1$ es más tolerante, permitiendo un **solapamiento máximo** entre $\mathcal{C}$ y $\mathcal{C}'$. Sin embargo, tomar $\tau = 1$ puede ocasionar que ambas comunidades sean totalmente idénticas, contradiciendo el objetivo principal del problema, por lo que se suele considerar $\tau \in [0,1)$.

Se utiliza el DSC en el paper (ver [[Glosario]]), pero en el código hay varias métricas para elegir.
### $$ \begin{equation} \tag{1}
  \mathcal{L}(h_\theta; \mathcal{G}, f, u) = \ell_{\text{decept}}(\mathcal{G}, h_\theta(\mathcal{G});f,u) + \lambda \ell_{\text{dist}}(\mathcal{G},h_\theta(\mathcal{G});f)
  \end{equation} $$
Función de pérdida de la tarea de *community membership hiding*. La idea es lograr que $u$ deje de pertenecer a su comunidad original, minimizando la "distancia" entre $f(\mathcal{G})$ y $f(\mathcal{G}')$.

$\ell_{\text{decept}}$ penaliza cuando el objetivo no es satisfecho. Consideramos $\Gamma$, el conjunto de grafos de entrada que **no** cumplen el objetivo (es decir, mantienen a $u$ como parte de $\mathcal{C}_i$). Formalmente, consideramos $\tilde{\mathcal{C}}_i$, la comunidad a la que $u$ es asignado al aplicar $f$ al grafo de entrada $\tilde{\mathcal{G}}$. 
Definimos $\Gamma = \left\{ \tilde{\mathcal{G}} \;\middle|\; \operatorname{sim}(\mathcal{C}_i \setminus \{u\}, \tilde{\mathcal{C}}_i \setminus \{u\}) > \tau \right\}.$
Ahora podemos calcular $\ell_{\text{decept}}$ :
$$
\ell_{\text{decept}}(\mathcal{G}, h_\boldsymbol{\theta}(\mathcal{G}); f) = \mathbb{1}_{\Gamma}(h_\boldsymbol{\theta}(\mathcal{G})). \tag{2}
$$

Donde, $\mathbb{1}_{\Gamma}(h_\boldsymbol{\theta}(\mathcal{G}))$ es la función indicadora 0-1, que evalúa a 1 si $h_\boldsymbol{\theta}(\mathcal{G}) \in \Gamma$, y 0 sino.

 $\ell_{\text{dist}}$ es una función compuesta diseñada para evaluar la disimilaridad entre dos grafos y sus respectivas comunidades encontradas por $f$. Cumple dos funciones: evitar que el nuevo grafo $h_\theta(\mathcal{G})$ diverga significativamente del grafo original $\mathcal{G}$, y previene que la nueva estructura de comunidades $f(h_\theta(\mathcal{G}))$ difiera sustancialmente de la estructura anterior $f(\mathcal{G})$.

### $\mathcal{E}^-_{u,i}$, $\mathcal{E}^+_{u,i}$
Aristas candidatas a ser removidas y agregadas, respectivamente. Definidas formalmente como:
$$
\mathcal{E}^{-}_{u,i} = \left\{ (u, v) \;\middle|\; u, v \in \mathcal{C}_i \wedge (u, v) \in \mathcal{E} \right\}
$$
(Aristas salientes de $u$, con $u$ y $v$ pertenecientes a la misma comunidad).
$$
\mathcal{E}^{+}_{u,i} = \left\{ (u, v) \;\middle|\; u \in \mathcal{C}_i,\, v \notin \mathcal{C}_i \wedge (u, v) \notin \mathcal{E} \right\}.
$$
(Aristas salientes de $u$, con $v$ fuera de la comunidad de $u$)

### $\beta$  (presupuesto)
Máxima cantidad de modificaciones de grafo permitidas.
### $$
\begin{aligned}
\boldsymbol{\theta}^* &= \arg\min_{\boldsymbol{\theta}} \left\{ \mathcal{L}(h_{\boldsymbol{\theta}}; \mathcal{G}, f, u)\right\} \\
&\text{sujeto a:} \quad |\mathcal{B}_{u,i}| \leq \beta
\end{aligned} \tag{3}
$$
La solución de community membership hiding se basa en encontrar el modelo óptimo $h^* = h_\theta$ cuyos parámetros $\theta$ minimizan $(1)$, resolviendo el objetivo restringido de arriba.
$\mathcal{B}_{u,i} \subseteq  \mathcal{E}^-_{u,i}, \mathcal{E}^+_{u,i}$ es el conjunto de modificaciones de aristas seleccionadas.
Esta ecuación es similar a la tarea de optimización de encontrar el mejor *grafo contrafactual* $\mathcal{G}* = h^*(G)$ que al ser pasado a $f$ cambia su salida para ocultar el nodo objetivo $u$ de su comunidad.

### $\mathcal{M} = \{ \mathcal{S}, \mathcal{A}, \mathcal{P}, \mathcal{p}_0, r, \gamma \}$ 
Proceso de decisión de Markov
#### $\mathcal{S}$ (estados)
En cada paso de tiempo $t$, $\mathcal{S}_t = s_t = \mathcal{G}^t \in \mathcal{S}$ es el grafo de entrada modificado actual.
En la práctica, reemplazamos $\mathcal{G}^t$ con su matriz de adyacencia $A_t \in \{0,1\}^{n \times n}$. $t = 0 \implies \mathcal{G}^0 = \mathcal{G} \; (A^0 = A)$ 

#### $\mathcal{A}$ (acciones)
$\mathcal{A} = \{a_t\}$, consistiendo de todas las operaciones válidas de modificación del grafo, asumiendo que el nodo $u$ pertenece a la comunidad $\mathcal{C}_i$.
$$
\mathcal{A} = \{ \operatorname{del}(u, v) \mid (u, v) \in \mathcal{E}^-_{u,i} \} \cup \{ \operatorname{add}(u, v) \mid (u, v) \in \mathcal{E}^+_{u,i} \} \tag{4}
$$
#### $\mathcal{P}$ (probabilidad de transición)
Dada la acción $a_t \in \mathcal{A}$ tomada por el agente en la iteración $t$, el agente transiciona determinísticamente del estado $s_t$ al estado $s_{t+1}$.
$\mathcal{P} : \mathcal{S} \times \mathcal{A} \times \mathcal{S} \rightarrow [0,1]$ asocia una probabilidad de transición a cada par estado-acción.
- $\mathcal{P}(s_{t+1} \mid s_t, a_t) = 1$ si $s_{t+1}$ es el siguiente estado que resulta de la aplicación de la acción $a_t$ en el estado $s_t$.
- $\mathcal{P}(s_{t+1} \mid s_t, a_t) = 0$ en caso contrario.

#### $r$ (recompensa)
Dada la acción $a_t$ que lleva al agente del estado $s_t$ a $s_{t+1}$, se define
$$
r(s_t, a_t) =
\begin{cases}
1 - \lambda \left( \ell_{\text{dist}}^{t} - \ell_{\text{dist}}^{t-1} \right), & \text{si se alcanza el objetivo}, \\[4pt]
-\lambda \left( \ell_{\text{dist}}^{t} - \ell_{\text{dist}}^{t-1} \right), & \text{en caso contrario}.
\end{cases} \tag{5}
$$
El objetivo se considera alcanzado cuando $f(\mathcal{G}^t)$ lleva a $u \in \mathcal{C}^t_i \neq \mathcal{C}_i$ tal que $sim(\mathcal{C}_i - \{u\}, \mathcal{C}^t_i-\{u\}) \leq \tau$. Además, $\ell_{\text{dist}}^{t} = \ell_{\text{dist}}\left( \mathcal{G}, \mathcal{G}_{t} ; f \right)$ mide la penalización en base al grafo antes y después de la acción $a_t$, y $\lambda \in \mathbb{R}_{>0}$ es un parámetro que controla su peso.
Más precisamente,
$$
\ell_{\text{dist}}(\mathcal{G}, \mathcal{G}^{t} ; f)
= \alpha \times d_{\text{community}} + (1 - \alpha) \times d_{\text{graph}} \tag{6}
$$
donde $d_{\text{community}}$ es la distancia entre las estructuras de las comunidades $f(\mathcal{G})$ y $f(\mathcal{G}^t)$, $d_\text{graph}$ mide la distancia entre los grafos $\mathcal{G}$ y $\mathcal{G}^t$, y $\alpha \in [0,1]$ balancea la importancia de cada distancia.

#### $\pi_\theta$ (política)
Minimizando $(3)$ podemos obtener la política óptima $\pi^*$ (aquella que nos da la mayor recompensa esperada para cualquier estado), lo que nos lleva al modelo óptimo $h^*$ (el que mejor predice las recompensas en el MDP).
$$
\theta^* = \arg\max_{\theta} \sum_{t=1}^{T} r\left( s_t, \pi_{\theta}(s_t) \right)
$$
donde $T$ es el máximo número de pasos por episodio tomados por el agente, y por lo tanto es siempre menor o igual que el número permitido de modificaciones de grafo $\beta$.

**(ver [[README_main#Advantage Actor-Critic (A2C)]] para las fórmulas y explicación de la resolución)**

---

Tenemos muchos hiperparámetros para cambiar y elegir, pero hacemos grid search sobre los siguientes:
- $\eta$ **learning rate**: qué tan rápido aprende el algoritmo, valores razonables entre $1 \times 10^{-4}$ y $1 \times 10^{-3}$.
- $\gamma$ **discount factor**: qué tan importante son las recompensas futuras, en comparación a las más "próximas".
- $\lambda$ **peso en la función de pérdida**: determina el balance entre la recompensa y la penalización por la diferencia entre un grafo $t$ y $t-1$ en base a la diferencia entre $\ell_\text{dist}$. En resumen, fija la importancia de que los grafos sucesivos se "parezcan".
- $\alpha$ **peso en $\ell_{dist}$**: tradeoff entre considerar las distancias entre grafos y estructuras de comunidades al calcular $\ell_\text{dist}$.
- $\epsilon$ **probabilidad de cambio de comunidad/nodo objetivo al entrenar**: al entrenar, con probabilidad $\epsilon$, se cambia de comunidad (y por lo tanto, de nodo objetivo).
- $\mathrm{c}_{entropy}$ **coeficiente de entropía**: determina si el agente prioriza la exploración (encontrar nuevas políticas, posiblemente mejores) o la explotación (maximizar recompensa con la política actual). 

Además, al testear iteramos sobre estos dos parámetros:
- $\beta$ (ver [[#$ beta$ (presupuesto) |presupuesto]])
- $\tau$ (ver [[#$sim( cdot, cdot)$|sim(.,.)]])