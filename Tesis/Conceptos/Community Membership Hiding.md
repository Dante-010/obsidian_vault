En pocas palabras, se busca permitirle a un nodo objetivo en un grafo evitar ser detectado como miembro de un cluster específico del grafo, determinado por un algoritmo de *community detection*.
Este objetivo se logra sugiriéndole al nodo en cuestión cómo estratégicamente modificar sus conexiones con otros nodos.

Se asume (en el paper original) que quien ejecute este algoritmo **puede ejecutar el algoritmo de community detection**, y **posee información completa del grafo**.