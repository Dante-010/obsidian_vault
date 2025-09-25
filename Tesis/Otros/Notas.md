- Una branch por agente (? Difícil para luego unir los datos de test: crear repo aparte, hacer el código más modular...)  
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
## Prioridades
- Anotar el proceso del codigo.
- GUARDAR TODA LA INFORMACION DEL AGENTE Y ENTORNO AL ENTRENAR
- Loggear en el método random los tamaños (relativos) de las comunidades y hacer un post-análisis
- Ablation study (Machine Learning)
- Aplicar las mismas "variantes" al paper Right to Hide.

##### Mini TODO
- Preguntarle a Esteban o Gabriel el tema del testing (estadísticamente hablando, cuántas corridas necesito?).
- Terminar lo pendiente en variables, parametros y ecuaciones.
- Escribir el "paso a paso" del testing y training.
- Probar el agente original reentrenado con el metodo 1
- Entrenar un agente permitiendo TODAS las operaciones en ejes (es decir, agregar/sacar con cualquiera).
- Ver cómo calculo la varianza en steps_needed
