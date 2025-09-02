En pocas palabras, se busca permitirle a un nodo objetivo $u$ en un grafo evitar ser detectado como miembro de un cluster específico del grafo, determinado por un algoritmo de *community detection*.
Este objetivo se logra sugiriéndole al nodo en cuestión cómo estratégicamente modificar sus conexiones con otros nodos.

Se asume (en el paper original) que quien ejecute este algoritmo **puede ejecutar el algoritmo de community detection**, y **posee información completa del grafo**.
## Restricciones originales
- Se elige un **único** nodo objetivo $u$.
- Las comunidades son ***non-overlapping***.
- $f(\cdot)$ es **black-box**.
- Quien quiera aplicar *community membership hiding* puede ejecutar el algoritmo de community detection, y posee **información completa del grafo**
- Los nodos no tienen **features**.
- $u$ solo puede remover aristas salientes a nodos **dentro** de su comunidad,
  y solo puede agregar aristas salientes a nodos **fuera** de su comunidad.
  
   (En el caso que $\mathcal{G}$ sea no direccionado, se puede extender esta lógica y considerar todas las aristas como "bidireccionales", y siguen valiendo las mismas reglas).

  No se permite remover aristas salientes a nodos **externos** puesto que podría aislar a $u$ y su comunidad original aún más, y no se permite agregar aristas salientes a nodos **internos** puesto que mejoraría la conectividad entre $u$ y los demás nodos de su comunidad, ambas acciones apuntando en contra del objetivo de community membership hiding.



