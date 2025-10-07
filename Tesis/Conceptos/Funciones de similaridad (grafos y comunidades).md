En el análisis de redes y detección de comunidades, es fundamental poder **comparar** particiones, conjuntos de nodos o grafos completos.  
Las **funciones de similitud** permiten cuantificar cuán parecidos son dos conjuntos de nodos o dos estructuras de red.
A continuación, vemos las funciones presentes en el código original.

---
## Similitud entre Comunidades
Estas funciones se aplican sobre dos listas de nodos `A` y `B`, representando comunidades o subconjuntos dentro de un grafo.  
Todas devuelven un valor entre **0 y 1**, donde 1 indica máxima similitud y 0 indica que no comparten elementos.

### 1. Jaccard
La **similitud de Jaccard** mide la proporción de elementos comunes respecto a todos los elementos distintos en ambos conjuntos.

$$
J(A,B) = \frac{|A \cap B|}{|A \cup B|}
$$

- **Ventajas:** Intuitiva y ampliamente utilizada.  
- **Desventajas:** Tiende a penalizar conjuntos pequeños.

### 2. Overlap
La **similitud de solapamiento** mide la fracción de elementos comunes con respecto al conjunto más pequeño.  

$$
O(A,B) = \frac{|A \cap B|}{\min(|A|, |B|)}
$$

- **Ventajas:** Ideal cuando los conjuntos son de distinto tamaño.  
- **Desventajas:** Puede sobrevalorar la similitud si uno de los conjuntos es muy pequeño.

### 3. Sørensen-Dice
La **similitud de Sørensen-Dice** pondera la intersección de manera doble, comparándola con la suma de los tamaños de ambos conjuntos.  

$$
S(A,B) = \frac{2|A \cap B|}{|A| + |B|}
$$

- **Ventajas:** Más sensible a coincidencias parciales que Jaccard.  
- **Desventajas:** Menos interpretativa si los conjuntos tienen tamaños muy distintos.

---
##  Similitud entre Grafos

Estas funciones comparan **estructuras completas de red**, es decir, los nodos y enlaces de dos grafos `G₁` y `G₂`.  
Se utilizan para medir cuánto cambió una red entre dos versiones o qué tan parecidas son dos redes diferentes.

### 1. Graph Edit Distance
La **distancia de edición de grafos** mide cuántas operaciones (agregar, eliminar o modificar nodos o aristas) son necesarias para transformar un grafo en otro.

$$
GED(G_1, G_2) = \text{mínimo número de ediciones necesarias}
$$

En este caso, se **normaliza** con respecto a un grafo nulo:

$$

GED_{norm}(G_1,G_2) = \frac{GED(G_1,G_2)}{GED(G_1,G_0) + GED(G_2,G_0)}

$$

donde \( G_0 \) es un grafo vacío.

- **Ventajas:** Medida estructural y precisa.  
- **Desventajas:** Costosa computacionalmente; difícil de escalar.

**Referencia:** Gao et al. (2010), *A survey of graph edit distance computation*.

### 2. Jaccard 1
Versión basada en **matrices de adyacencia**.  
Compara las conexiones de dos grafos calculando la diferencia absoluta entre sus matrices.
$$

J_1(G,H) = \frac{\sum_{i,j} |A_{ij}^G - A_{ij}^H|}{\sum_{i,j} \max(A_{ij}^G, A_{ij}^H)}

$$
Aquí, 0 significa que los grafos son idénticos y 1 que son completamente diferentes.

- **Ventajas:** Basada en información matricial; precisa.  
- **Desventajas:** Requiere que ambos grafos tengan el mismo número de nodos.

### 3. Jaccard 2
Versión alternativa basada directamente en los **conjuntos de aristas**.  
Se calcula como la similitud de Jaccard entre los conjuntos de aristas de ambos grafos.

$$
J_2(G,H) = 1 - \frac{|E_G \cap E_H|}{|E_G| + |E_H| - |E_G \cap E_H|}
$$

La medida se invierte para que:
- **0** → grafos idénticos  
- **1** → grafos completamente distintos

- **Ventajas:** Simple y eficiente; se basa solo en los enlaces.  
- **Desventajas:** No considera pesos ni direccionalidad.