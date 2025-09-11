- Una branch por agente (? Difícil para luego unir los datos de test: crear repo aparte, hacer el código más modular...)  
- Grafos "especiales", cadenas, árboles, k-regulares  
- Random graphs "Erdos"  
- Barabasi-Albert model  
- Mayor NMI => Mayor similitud entre grafo original y nuevo.

El método 2, para training (como tiene epsilon 0), el agente "elige" una comunidad con una cantidad de nodos proporcional al 0.2 del valor máximo de las comunidades, y siempre entrena con esta comunidad (ver Utils.PREFERRED_COMMUNITY_SIZE).

**ver epsilon = probabilidad de cambiar de comunidad en el training**
En training, NUNCA se cambia el nodo, solo se da la opcion de cambiar la comunidad (que con epsilon=0 no se hace)

Esto causa un overfitteo a esta comunidad en particular, y más en general a las comunidades con un 0.2 de nodos en relación a la máxima comunidad (en cuanto a nodos).

La solución es utilizar un epsilon mayor, para que el agente vaya cambiando de comunidad y aprende a ocultar nodos en todas las situaciones (**ver epsilon-decay**).

En testing, en base a Utils.PREFERRED_COMMUNITY_SIZE, se prueban con comunidades de distintos tamaños. Se cambia de comunidad UNA vez por cada tamaño (ejemplo, Utils.PREFERRED_COMMUNITY_SIZE = $[0.2, 0.5, 0.8]$, cambio 3 veces), y una vez cambiada la comunidad, se cambia de nodo Utils.STEPS_EVAL veces (100 por defecto). **ver si hace falta cambiar esto**.

Entonces, la hipótesis original no está del todo mal, pero no está bien tampoco.
## Prioridades
- Anotar el proceso del codigo.
- Verficar verdaderamente que el caso de la comunidad mediana sea asi, y mostrarlo experimentalmente (con todos los datasets). Una vez hecho esto, escribir el correo electronico.
- GUARDAR TODA LA INFORMACION DEL AGENTE Y ENTORNO AL ENTRENAR
- El agente original hace "trampa", pues les da preferencia a las comunidades con valor 
