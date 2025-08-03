### `main.py`
Lógicamente, el archivo principal es `main.py`. Como bien se aclara en [[README_main]], utilizamos los siguientes comandos para entrenar y probar el modelo, respectivamente:
```shell
python main.py --mode "train"
```
```shell
python main.py --mode "test"
```

En `src/utils/utils.py` se encuentra el archivo con los parámetros del modelo.

Los parámetros por defecto (por lo menos, los presentes en el [repositorio](https://github.com/AndreaBe99/community_membership_hiding/blob/main/main.py) al momento de escribri esto) incluyen:

- Datasets (`FilePaths`):
  - `KAR`
  - `WORDS`
  - `VOTE`
  - `POW`
  - `FB_75`

- Algoritmos de detección de comunidades (`DetectionAlgorithmsNames`):
  - `GRE`
  - `LOUV`
  - `WALK`
  
  - Valores de `tau` ($\tau$): `[0.3, 0.5, 0.8]`

- Parámetros para ocultamiento de nodos:
  - `beta`: `[0.5, 1, 2]`
  - Interpretación: multiplicador del grado medio del grafo

- Parámetros para ocultamiento de comunidades (comentado):
  - `beta`: `[1, 3, 5, 10]`
  - Interpretación: porcentaje de acciones de reestructuración (añadir/eliminar aristas)

- Notas adicionales:
  - Se utiliza `NodeHiding.run_experiment()` para pruebas
  - `CommunityHiding.run_experiment()` está presente pero comentado


### utils.py
Ver [[Variables, parámetros y ecuaciones]] para un enfoque más "teórico"


