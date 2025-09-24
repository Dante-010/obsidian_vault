### Configuración

Los experimentos se llevaron a cabo en una instancia EC2 de AWS del tipo [`g4dn.xlarge`](https://aws.amazon.com/ec2/instance-types/g4/), con las siguientes especificaciones:

- **vCPUs:** 4 (Intel Cascade Lake a 2.5 GHz, arquitectura de 24 núcleos)
- **Memoria:** 16 GiB de RAM
- **GPU:** NVIDIA T4
- **Sistema operativo:** Ubuntu 24.04

Se utilizó la siguiente imagen de máquina de Amazon (AMI):

> [**Deep Learning Base OSS Nvidia Driver GPU AMI (Ubuntu 24.04) 20250620**](https://docs.aws.amazon.com/dlami/latest/devguide/aws-deep-learning-base-gpu-ami-ubuntu-24-04.html)

Ver [[Código]] para una explicación de cómo ejecutar los experimentos y qué función cumple cada archivo.
## Datasets, parámetros y configuración por defecto
En [[readme_datasets]] tenemos la descripción de cada dataset.

Según el paper, entrenaron el agente con el dataset **words**, y lo evaluaron con cuatro datasets adicionales: **kar**, **vote**, **pow**, y **fb-75**.
Se entrena el agente utilizando únicamente el algoritmo **greedy**, y para testear además usan **louvain** y **walktrap**.
![[Pasted image 20250812063631.png]]
Como métrica de similaridad ($\text{sim}(\cdot)$, dentro de $\ell_\text{decept}$), utilizan el **DSC**, y para las distancias entre estructuras de comunidades y grafos antes y después de tomar una acción utilizan **NMI** score para la comparación de comunidades, y la distancia **Jaccard** para comparación de grafos (ver [[Glosario#Conceptos introducidos en las referencias|glosario]] para las definiciones).

Una vez entrenado el agente, se lo compara con cinco baselines:
1. **Random-based:**  
   Selecciona aleatoriamente un nodo del grafo. Si el nodo elegido es vecino del nodo objetivo, se elimina la arista entre ellos; si no, se añade una nueva arista. Esta aleatoriedad busca enmascarar la pertenencia comunitaria real del nodo.
2. **Degree-based:**  
   Se enfoca en nodos con los grados más altos y los reconecta. Al priorizar nodos con alta conectividad, este método pretende interrumpir las conexiones centrales dentro de la comunidad inicial para favorecer el ocultamiento.
3. **Betweenness-based:**  
   Prioriza nodos con la mayor centralidad de intermediación ([[Referencias#(10) Betweenness Centrality]]) $b(u)$, calculada como:  
  $$
  b(u) = \sum_{s \neq u \neq t} \frac{\sigma_{s,t}(u)}{\sigma_{s,t}},
  $$  
   donde $\sigma_{s,t}$ es el número total de caminos más cortos entre $s$ y $t$, y $\sigma_{s,t}(u)$ los que pasan por $u$. Busca desconectar nodos estratégicos para afectar la estructura comunitaria.

4. **Roam-based:**  
   Basado en la heurística Roam [[Referencias#(41) Roam Based Heuristic]], diseñada para reducir la centralidad y la influencia del nodo objetivo dentro de su comunidad, ayudando a su camuflaje.

5. **Greedy-based:**  
   Una heurística “relajado” que prioriza acciones de reconexión prometedoras enfocándose en nodos de mayor grado, sin explorar exhaustivamente todas las opciones posibles (ver [[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=6&selection=595,0,777,13|Evading Community Detection via Counterfactual Neighborhood Search, page 6]]).

Para evaluar el desempeño del algoritmo, se utilizan principalmente tres métricas. La **Tasa de Éxito (SR)** mide el porcentaje de veces que el nodo objetivo es ocultado exitosamente de su comunidad original, considerando las restricciones definidas. Por otro lado, la **Información Mutua Normalizada (NMI)** cuantifica la similitud entre la estructura comunitaria original y la resultante tras modificar el grafo, donde un valor alto indica que la estructura se preserva con menor costo. 
Como SR y NMI son métricas contrastantes, donde un aumento en una suele implicar una disminución en la otra, se emplea la media armónica (similar al **score F1**) entre ambas para evaluar el equilibrio óptimo entre ocultamiento efectivo y conservación estructural, calculada como $\frac{2 \times SR \times NMI}{SR + NMI}$.

A la hora de testear, se prueban todas las combinaciones de los siguientes valores para los siguientes parámetros parámetros:
- $\tau = (0.3, 0.5, 0.8)$
- $\beta = (\frac{1}{2}\mu, 1\mu, 2\mu, \text{ donde } \mu = \frac{|\mathcal{E}|}{|\mathcal{V}|})$ 

> For each parameter combination, dataset, and community detection algorithm, we conducted a total of 100 experiments. In each iteration, we randomly select a node from a different community than the previous one for concealment. The reported results are based on the average outcomes across all runs. Furthermore, we explore two distinct setups: symmetric and asymmetric. In the symmetric case, our DRL-Agent is trained and tested using the same community detection algorithm used for the membership hiding task. Instead, the asymmetric setup evaluates the performance of our method when tested on a community detection algorithm different from the one used for training. This second setting allows us to assess the transferability of our method to community detection algorithms unseen at training time. 

[[Evading Community Detection via Counterfactual Neighborhood Search.pdf#page=7&selection=154,2,185,36|Evading Community Detection via Counterfactual Neighborhood Search, page 7]]

--- 
### Replicar el paper
La documentación existente es excelente, basta con correr `main.py` (luego de instalar las dependencias necesarias, corremos primero con `--mode "train"` y luego `--mode "test"`).

Dado que en `graph_env.py` se utiliza:
```python
random.seed(time.time())
```
*Probablemente* no vamos a obtener los mismos resultados, pero deberíamos observar algo considerablemente similar. Adaptando algunos parámetros para corresponderse con lo dicho en el paper, y dejando todo lo demás por defecto, obtenemos los siguientes resultados:

![[allDatasets_tau_0.5_beta_1_sr_nmi_lineplot.png]]
$(\tau = 0.5, \beta = 1)$

Como se esperaba, obtenemos resultados prácticamente idénticos, salvo cierto grado de aleatoriedad. Utilizamos esto como base para los próximos experimentos.

### "Testeo" la reproducibilidad


### Agrego entropía
En `agent.py` había un parámetro `self.entropy_coeff` sin utilizar. Lo agregué al paso de entrenamiento (función de loss) y reentrené los agentes sin cambiar nada.

### Grid search "general"
Quiero saber cómo y qué parámetros influyen en el aprendizaje del agente.

### Modifico estructura de la red
Ver [[Cambios en el agente]] para el motivo y explicación de los cambios