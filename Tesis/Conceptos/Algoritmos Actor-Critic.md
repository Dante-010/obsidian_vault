Algoritmos de [[Aprendizaje por Refuerzo]], donde un **actor** determina que acciones tomar en base a una función de política, y un **crítico** evalúa esas acciones de acuerdo a una función de valor.
### Componentes principales
- **Política (actor):**  
Una política estocástica parametrizada por $\theta$, denotada como  
$$
\pi_{\theta}(a \mid s)
$$
que define la probabilidad de tomar la acción $a$ dado el estado $s$.  

- **Función de valor (crítico):**  
Una función parametrizada por $\phi$, que puede ser la función de valor del estado  
$$
V_{\phi}(s) \approx V^{\pi}(s)
$$
o la función de valor acción $Q_{\phi}(s,a)$.

### Función de ventaja
La función de ventaja mide qué tan buena es una acción comparada con el valor promedio del estado:  
$$
A^{\pi}(s, a) = Q^{\pi}(s, a) - V^{\pi}(s)
$$

En la práctica, se estima con:  
$$
\hat{A}_t = r_t + \gamma V_{\phi}(s_{t+1}) - V_{\phi}(s_t)
$$
(conocido como **TD error** o error temporal-diferencia).

### Actualización del actor
El objetivo del actor es maximizar la función objetivo:  
$$
J(\theta) = \mathbb{E}_{\pi_{\theta}} \left[ \log \pi_{\theta}(a_t \mid s_t) \hat{A}_t \right]
$$
donde el gradiente de política se calcula como:  
$$
\nabla_{\theta} J(\theta) = \mathbb{E}_{\pi_{\theta}} \left[ \nabla_{\theta} \log \pi_{\theta}(a_t \mid s_t) \hat{A}_t \right]
$$

### Actualización del crítico

El crítico minimiza la función de pérdida del valor temporal-diferencia:  
$$
L(\phi) = \mathbb{E} \left[ \left( r_t + \gamma V_{\phi}(s_{t+1}) - V_{\phi}(s_t) \right)^2 \right]
$$