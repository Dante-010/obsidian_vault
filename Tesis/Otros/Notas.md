- Grafos "especiales", cadenas, árboles, k-regulares  
- Random graphs "Erdos"  
- Barabasi-Albert model  
- Mayor NMI => Mayor similitud entre grafo original y nuevo.
- Batch normalization es útil e importante, no se sabe bien por qué.

$\text{(similaridad)} \, sim(\mathcal{G}, \mathcal{G}) \leq \tau \implies \checkmark \text{(objetivo cumplido)}$


VARIANTES (HACER FABRICA DE VARIANTES?)
	- Ver la variante geometrica (definir un "backend"? Adaptar a Reinforcement Learning?)
	- SOLO SACAR/SOLO PONER (problema original) (semi resuelto, ver con nuevo modelo)
	- CAMBIO EL PROBLEMA (Join C1 and C2)

Ver notebook 4. Scaling GNNs

Este "TODO" no es tan importante, son ideas.
- [x] Replicar paper 
- [x] Cambiar seed (random).
- [x] Crear datasets sintéticos (o no, lo importante es que tengan ciertas estructuras/patrones), y tratar de predecir el resultado (hay herramientas en el código para hacer esto, investigar).
- [x] Elegir comunidades con probabilidad relativa a su tamaño
- [x] Pesos en los ejes  
- [x] Computar baselines una sola vez
- [x] Evaluar/graficar varios agentes a la vez  
- [ ] Overlapping communities.
- [ ] Multiple membership hiding.
- [ ] Trabajar con $f(\cdot)$ white-box (conociendo el algoritmo)
- [ ] Alterar node features
- [ ] Conocimiento parcial del grafo
- [x] Solo poner y/o sacar (pensar antes de hacerlo, y comparar con resultados).
- [ ] Asegurarse de que en cada instancia se mantenga (o no) la transferabilidad.
- [ ] Hacer algoritmos *naive* o *especiales* y ver si sigue la transferabilidad (white-box y black-box)
- [x] Ver convergencia en base a modelo (solo agregar, solo borrar).
- [ ] Probar con datasets "triviales"  
- [x] Agregar una feature de "target_node"  

---
1a, 1b, y 2 principales.

1. Original
2. Original con GATV2
3. Original con GatV2 con target node
4. Original node2vec con target node
(en resumen, ver lo que aporta agregar target node SOLO y agregar GATv2 solo)

- Probar sacando TODAS las features (o agregar de a una, probar de poco)
- Agregar feature de la comunidad a la que pertenece cada nodo.

Ejemplos para el 4: Olvidarse de comunidades y meter caminos mas cortos? (o cosas asi, pero olvidarse de comunidades).