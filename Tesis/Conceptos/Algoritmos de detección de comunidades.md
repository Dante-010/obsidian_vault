La **detección de comunidades** es una de las tareas fundamentales en el análisis de redes complejas. Su objetivo es identificar subconjuntos de nodos (llamados *comunidades* o *clusters*) que están más densamente conectados entre sí que con el resto del grafo.  
Estos métodos permiten descubrir estructuras ocultas en redes sociales, biológicas, de información o de transporte, entre muchas otras. Existen distintos enfoques basados en optimización, dinámica de procesos, propiedades espectrales o medidas de centralidad. 

---
## Algoritmos presentes en el código original

### 1. Louvain
El **algoritmo de Louvain** es un método jerárquico basado en la **maximización de la modularidad**.  
Funciona agrupando nodos de manera iterativa para mejorar la modularidad, una medida que evalúa cuán buena es una partición de la red.  
Es rápido y escalable, por lo que se usa comúnmente en redes grandes.

- **Ventajas:** Alta eficiencia, buenos resultados prácticos.  
- **Desventajas:** Resultados no deterministas; puede quedarse en óptimos locales.

**Referencia:** Blondel et al. (2008), *Fast unfolding of communities in large networks*.

### 2. Walktrap 
El **algoritmo Walktrap** se basa en **paseos aleatorios**.  
La idea es que los paseos aleatorios tienden a permanecer dentro de la misma comunidad por varios pasos.  
El método mide la similitud entre nodos mediante distancias basadas en la probabilidad de transición, y luego aplica una agrupación jerárquica.

- **Ventajas:** Tiende a producir comunidades equilibradas.  
- **Desventajas:** Costoso en redes muy grandes.

**Referencia:** Pons & Latapy (2006), *Computing communities in large networks using random walks*.

### 3. Greedy 
También conocido como **fast greedy**, este algoritmo construye comunidades de forma ascendente mediante una estrategia **voraz de optimización de la modularidad**.  
Empieza con cada nodo como comunidad individual y combina aquellas fusiones que más aumentan la modularidad.

- **Ventajas:** Simplicidad y buena eficiencia.  
- **Desventajas:** Puede quedar atrapado en óptimos locales y no detecta bien estructuras pequeñas.

**Referencia:** Clauset, Newman & Moore (2004), *Finding community structure in very large networks*.

### 4. Infomap 
**Infomap** utiliza principios de **teoría de la información**.  
Modela el flujo de información sobre el grafo mediante paseos aleatorios y busca una codificación que minimice la descripción del recorrido del flujo, de manera que las comunidades correspondan a regiones de bajo “costo de descripción”.

- **Ventajas:** Detecta estructuras naturales de flujo; buen desempeño empírico.  
- **Desventajas:** Puede fragmentar excesivamente si hay ruido.

**Referencia:** Rosvall & Bergstrom (2008), *Maps of random walks on complex networks reveal community structure*.

### 5. Label Propagation
El **algoritmo de propagación de etiquetas** asigna inicialmente una etiqueta única a cada nodo.  
Luego, cada nodo adopta la etiqueta más frecuente entre sus vecinos, repitiendo el proceso hasta alcanzar estabilidad.  
El resultado son comunidades definidas de manera no supervisada y muy rápida.

- **Ventajas:** Extremadamente rápido, no requiere parámetros.  
- **Desventajas:** Resultados no deterministas; puede producir comunidades muy desiguales.

**Referencia:** Raghavan, Albert & Kumara (2007), *Near linear time algorithm to detect community structures in large-scale networks*.

### 6. Eigenvector
Este algoritmo está basado en **métodos espectrales** del grafo.  
Utiliza el **vector propio principal de la matriz modularidad** para dividir el grafo en comunidades.  
La división puede repetirse recursivamente hasta optimizar la modularidad.

- **Ventajas:** Basado en fundamentos matemáticos sólidos.  
- **Desventajas:** Escalabilidad limitada; sensible a ruido.

**Referencia:** Newman (2006), *Finding community structure using the eigenvectors of matrices*.

### 7. Edge Betweenness
Basado en la **intermediación de aristas**, este método elimina iterativamente los enlaces con mayor *betweenness* (número de caminos más cortos que pasan por la arista).  
A medida que se eliminan enlaces, surgen comunidades desconectadas.

- **Ventajas:** Interpretable y conceptualmente simple.  
- **Desventajas:** Computacionalmente costoso; poco escalable.

**Referencia:** Girvan & Newman (2002), *Community structure in social and biological networks*.

### 8. Spinglass
Inspirado en modelos físicos de **vidrios de espín**, este algoritmo interpreta la detección de comunidades como un problema de minimización de energía en un sistema de espines interactuantes.  
Busca la configuración de “espines” (comunidades) que minimiza la energía total del sistema.

- **Ventajas:** Permite incorporar pesos y direccionalidad.  
- **Desventajas:** Computacionalmente intensivo.

**Referencia:** Reichardt & Bornholdt (2006), *Statistical mechanics of community detection*.

### 9. Optimal Modularity
Este método busca la **partición exacta que maximiza la modularidad**, utilizando algoritmos de optimización global.  
Debido a su complejidad exponencial, solo es viable para redes pequeñas.

- **Ventajas:** Encuentra la partición óptima.  
- **Desventajas:** Escalabilidad muy limitada.

**Referencia:** Brandes et al. (2008), *On finding graph clusterings with maximum modularity*.

### 10. Scalable Community Detection
El algoritmo **SCD (Scalable Community Detection)** se basa en la **maximización de la sorpresa**, una medida alternativa a la modularidad.  
La sorpresa evalúa cuán improbable es observar una concentración tan alta de enlaces internos bajo un modelo aleatorio.  
Este método fue diseñado para **grandes redes**, priorizando la precisión y la escalabilidad.

- **Ventajas:** Alta precisión, buen rendimiento en grafos grandes.  
- **Desventajas:** Implementación más compleja; depende de binarios externos.

**Referencia:** Aldecoa & Marín (2013), *Surprise maximization reveals the community structure of complex networks*.

---
## Consideraciones Generales

- La **modularidad** es la métrica más usada, pero tiene limitaciones (resolución y sensibilidad a tamaño).  
- Métodos como **Infomap** o **SCD** ofrecen alternativas basadas en flujo o sorpresa.  
- No existe un algoritmo universal: la elección depende del tamaño, tipo y densidad de la red.
