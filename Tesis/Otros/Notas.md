- Grafos "especiales", cadenas, árboles, k-regulares  
- Random graphs "Erdos"  
- Barabasi-Albert model  
- Mayor NMI => Mayor similitud entre grafo original y nuevo.

$\text{(similaridad)} \, sim(\mathcal{G}, \mathcal{G}) \leq \tau \implies \checkmark \text{(objetivo cumplido)}$

El método 2, para training (como tiene epsilon 0), el agente "elige" una comunidad con una cantidad de nodos proporcional al 0.2 del valor máximo de las comunidades, y siempre entrena con esta comunidad (ver Utils.PREFERRED_COMMUNITY_SIZE).

**ver epsilon = probabilidad de cambiar de comunidad en el training**
En training, NUNCA se cambia el nodo, solo se da la opcion de cambiar la comunidad (que con epsilon=0 no se hace) AL FINAL SI, LOS UTILS ESTABAN PARA TESTING

La solución es utilizar un epsilon mayor (METODO 1), para que el agente vaya cambiando de comunidad y aprende a ocultar nodos en todas las situaciones (**ver epsilon-decay**).

En testing, en base a Utils.PREFERRED_COMMUNITY_SIZE, se prueban con comunidades de distintos tamaños. Se cambia de comunidad UNA vez por cada tamaño (ejemplo, Utils.PREFERRED_COMMUNITY_SIZE = $[0.2, 0.5, 0.8]$, cambio 3 veces), y una vez cambiada la comunidad, se cambia de nodo Utils.STEPS_EVAL veces (100 por defecto). **ver si hace falta cambiar esto**.

Entonces, la hipótesis original no está del todo mal, pero no está bien tampoco.
##### Mini TODO
- GUARDAR TODA LA INFORMACION DEL AGENTE Y ENTORNO AL ENTRENAR
- Entrenar un agente permitiendo TODAS las operaciones en ejes (es decir, agregar/sacar con cualquiera).
- Ver cómo calculo la varianza en steps_needed.

VARIANTES (HACER FABRICA DE VARIANTES?)
	- Ver la variante geometrica (definir un "backend"? Adaptar a Reinforcement Learning?)
	- SOLO SACAR/SOLO PONER (problema original)
	- CAMBIO EL PROBLEMA (Join C1 and C2)


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
- [ ] Solo poner y/o sacar (pensar antes de hacerlo, y comparar con resultados).
- [ ] Asegurarse de que en cada instancia se mantenga (o no) la transferabilidad.
- [ ] Hacer algoritmos *naive* o *especiales* y ver si sigue la transferabilidad (white-box y black-box)
- [ ] Ver convergencia en base a modelo (solo agregar, solo borrar).
- [ ] Probar con datasets "triviales"  
- [ ] Agregar una feature de "target_node"  