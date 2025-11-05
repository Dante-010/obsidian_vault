**node2vec** utiliza el hecho de que las caminatas aleatorias en un grafo pueden ser tratadas como oraciones en un corpus. Cada nodo es tratado como una palabra individual, y una caminata aleatoria es tratada como una oración. Si les damos estas oraciones a un modelo como **word2vec** (que define *embeddings* para palabras/tokens), los caminos encontrados por las caminatas, ahora oraciones, pueden ser analizados con técnicas tradicionales de data-mining para documentos.

En el caso del paper, se puede utilizar node2vec para extraer feature vectors de los nodos en el caso que el grafo no tenga features propios (también existe la opción de simplemente asignar features aleatorias).

Se mencionan dos conceptos interesantes para nodos en grafos (específicamente, la relación entre los nodos originales y su *embedding*): **homofilia** (nodos altamente interconectados y que pertenecen a clusters similares deben ser embedidos cerca), y **equivalencia estructural** (nodos con roles estructurales similares en las redes deben ser embedidos cerca, sin hacer énfasis en su conectividad "real").

Como resumen general, el algoritmos node2vec es un proceso de dos partes: 
- Primero, utiliza caminatas aleatorias sesgadas de segundo orden para generar secuencias de nodos u "oraciones".
- Una vez generadas las secuencias, se utilizan como entrada para un modelo skip-gram con negative sampling

*Notas: segundo orden implica que la caminata depende tanto del estado actual como del anterior, y los sesgos se establecen con parámetros $p$ y $q$ que determinan la probabilidad de volver al nodo anterior, y la probabilidad de explorar nodos "lejanos" al nodo anterior, respectivamente.
Según la elección de estos parámetros, se puede priorizar el embedding de homofilia o de equivalencia estructural de cada nodo.*

*Otros parámetros existentes son la dimensión del embedding, el número de caminatas por nodo, y la longitud de cada caminata.*

*Un modelo skip-gram recibe un nodo objetivo y un contexto. Su objetivo es dada una palabra, predecir su contexto. Trabaja con una "context window", que determina qué tan grande es el contexto. Negative sampling es una estrategia que se basa en lo siguiente: en vez de tratar de actualizar  todos los pesos para cada para input-context (lo cual escala pésimo), tomo un conjunto de palabras para las cuales seteo los pesos de forma tal que la red neuronal da un output de 0 (es decir, esas palabras no están en el contexto de la palabra objetivo). Por lo tanto, solo tengo que actualizar los pesos relacionados a estas palabras "negativas", que son elegidas en base a su frecuencia y característica (se suele utilizar una distribución unigram elevada a 3/4, donde las palabras más frecuentes son más probables de ser seleccionadas como muestras negativas).*