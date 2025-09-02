El **Aprendizaje por Refuerzo** es un paradigma del aprendizaje automático en el cual un *agente* interactúa con un *entorno* $\mathcal{E}$ a lo largo de una secuencia discreta de pasos temporales $t = 0, 1, \dots, T$.  

En cada paso:
1. El agente observa un **estado** $s_t \in \mathcal{S}$.
2. Selecciona una **acción** $a_t \in \mathcal{A}(s_t)$ según una **política** $\pi$.
3. El entorno devuelve una **recompensa** $r_t \in \mathbb{R}$ y un nuevo estado $s_{t+1}$.

El objetivo del agente es maximizar la **suma acumulada de recompensas** (*retorno*).

### Función de política
La **política** $\pi$ describe la estrategia de decisión del agente:

- **Determinista**:  
$$
\pi: \mathcal{S} \to \mathcal{A}, \quad a_t = \pi(s_t)
$$

- **Estocástica**:  
$$
\pi(a \mid s) = \mathbb{P}(a_t = a \mid s_t = s)
$$
### Retorno esperado
El **retorno** desde un estado $s_t$ se define como:  
$$
G_t = \sum_{k=0}^{\infty} \gamma^{k} r_{t+k+1}
$$
donde $\gamma \in [0, 1]$ es el **factor de descuento**.

### Función de valor de estado
La **función de valor** bajo una política $\pi$ es:  
$$
V^{\pi}(s) = \mathbb{E}_{\pi} \left[ G_t \mid s_t = s \right]
$$
mide el retorno esperado al iniciar en el estado $s$ y seguir la política $\pi$.

### Función de valor de acción (Q-función)
$$
Q^{\pi}(s, a) = \mathbb{E}_{\pi} \left[ G_t \mid s_t = s, a_t = a \right]
$$
representa el retorno esperado al tomar la acción $a$ en el estado $s$ y luego seguir $\pi$.

### Objetivo de optimización
El problema de control óptimo en RL puede formularse como:  
$$
\pi^{*} = \arg\max_{\pi} \; \mathbb{E} \left[ \sum_{t=0}^{T} \gamma^t r(s_t, a_t) \right]
$$
donde $\pi^{*}$ es la política que maximiza el retorno esperado.
