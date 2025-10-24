## 1. `RANDOM_COMMUNITY`

Selecciona una comunidad al azar entre todas las comunidades disponibles, asegurando que tenga más de un nodo y, en caso de que ya exista una comunidad previa, que no sea la misma que la anterior.  
Es el método más adecuado para el entrenamiento, ya que introduce variabilidad y permite que el agente aprenda a adaptarse a distintas configuraciones.  
Si se ejecuta muchas veces, este método es el que mejor refleja el rendimiento real del agente.
## 2. `CLOSEST_SIZE_PERCENTAGE`

Selecciona la comunidad cuyo tamaño está más cerca de un porcentaje determinado (por ejemplo, `preferred_community_size`) del tamaño máximo entre todas las comunidades.  
Este método suele utilizarse en las etapas de testing, para evaluar el comportamiento del agente en comunidades de distintos tamaños de manera más controlada.
## 3. `PROPORTIONAL_SIZE`

Selecciona una comunidad de forma aleatoria, pero con una probabilidad proporcional a su tamaño.  
De este modo, las comunidades más grandes tienen una mayor probabilidad de ser elegidas.
## 4. `RANDOM_NODE`

Primero elige un nodo al azar del grafo, y luego selecciona la comunidad a la que pertenece ese nodo.  
Como la probabilidad de elegir un nodo de una comunidad grande es mayor, este método también tiende a favorecer comunidades grandes, de manera similar al método proporcional.

## 5. `CURRICULUM_LEARNING`
Ver [[Glosario]].

---

`RANDOM_COMMUNITY` es el más apropiado para el entrenamiento, mientras que `CLOSEST_SIZE_PERCENTAGE` se utiliza normalmente en la evaluación para asegurar la cobertura de distintos tamaños de comunidad. Sin embargo, repetir múltiples ejecuciones con `RANDOM_COMMUNITY` es lo que realmente permite medir el desempeño general del agente.
