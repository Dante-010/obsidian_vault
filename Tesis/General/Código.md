### main.py
Lógicamente, el archivo principal es `main.py`. Como bien se aclara en [[README_main]], utilizamos los siguientes comandos para entrenar y probar el modelo, respectivamente:
```shell
python main.py --mode "train"
```
```shell
python main.py --mode "test"
```

Lo utilizamos para elegir sobre qué datasets e hiperparámetros entrenar/testear.
En `src/utils/utils.py` se encuentra el archivo con los parámetros del modelo.
Los parámetros por defecto (por lo menos, los presentes en el [repositorio](https://github.com/AndreaBe99/community_membership_hiding/blob/main/main.py) al momento de escribrir esto) son:

```python
class HyperParams(Enum):
    """Hyperparameters for the Environment"""

    # ! REAL GRAPH Graph path (change the following line to change the graph)
    GRAPH_NAME = FilePaths.WORDS.value
    # ! Define the detection algorithm to use (change the following line to change the algorithm)
    DETECTION_ALG_NAME = DetectionAlgorithmsNames.GRE.value
    # Multiplier for the rewiring action number, i.e. (mean_degree * BETA)
    BETA = 1
    # ! Strength of the deception constraint, value between 0 (hard) and 1 (soft)
    TAU = 0.5
    COMMUNITY_SIMILARITY_FUNCTION = SimilarityFunctionsNames.SOR.value
    GRAPH_SIMILARITY_FUNCTION = SimilarityFunctionsNames.JAC_1.value


    """ Hyperparameters  Testing """
    # ! Weight to balance the penalty in the reward
    # The higher its value the more importance the penalty will have
    LAMBDA = [0.1]  # [0.01, 0.1, 1]
    # ! Weight to balance the two metrics in the definition of the penalty
    # The higher its value the more importance the distance between communities
    # will have, compared with the distance between graphs
    ALPHA = [0.7]  # [0.3, 0.5, 0.7]
    # Multiplier for the number of maximum steps allowed
    MAX_STEPS_MUL = 1

    # Method to change the target community
    # - 1: choose a random community
    # - 2: choose the community with the length closest to the half of the maximum
    #       length of the communities.
    # - 3: choose a community based on the distribution of the number of
    #       nodes in the communities
    COMMUNITY_CHANGE_METHOD = 2

    PREFERRED_COMMUNITY_SIZE = [0.2, 0.5, 0.8]

    """ Graph Encoder Parameters """
    RANDOM_NODE2VEC = True  # If True, use Random features instead of Node2Vec
    EMBEDDING_DIM = 128  # 256
    WALK_NUMBER = 5  # 5, 10
    WALK_LENGTH = 40  # 40, 80

    """ Agent Parameters """
    # Change target community and target node with a probability of EPSILON
    EPSILON = [0]  # Between 0 and 100
    # Networl Architecture
    HIDDEN_SIZE_1 = 64
    HIDDEN_SIZE_2 = 64
    # Regularization parameters
    DROPOUT = 0.2
    WEIGHT_DECAY = 1e-3
    # Hyperparameters for the ActorCritic
    EPS_CLIP = np.finfo(np.float32).eps.item()  # 0.2
    BEST_REWARD = -np.inf
    # ° Hyperparameters  Testing ° #
    # ! Learning rate, it controls how fast the network learns
    LR = [7e-4]  # [1e-7, 1e-4, 1e-1]
    # ! Discount factor:
    # - 0: only the reward on the next step is important
    # - 1: a reward in the future is as important as a reward on the next step
    GAMMA = [0.95]  # [0.9, 0.95]

    """ Training Parameters """
    # Number of episodes to collect experience
    MAX_EPISODES = 10000
    # Dictonary for logging
    LOG_DICT = {
        # List of rewards per episode
        "train_reward_list": [],
        # Avg reward per episode, with the last value multiplied per 10 if the
        # goal is reached
        "train_reward_mul": [],
        # Total reward per episode
        "train_reward": [],
        # Number of steps per episode
        "train_steps": [],
        # Average reward per episode
        "train_avg_reward": [],
        # Average Actor loss per episode
        "a_loss": [],
        # Average Critic loss per episode
        "v_loss": [],
        # set max number of training episodes
        "train_episodes": MAX_EPISODES,
    }

    """Evaluation Parameters"""
    # ! Change the following parameters according to the hyperparameters to test
    STEPS_EVAL = 100
```

*(Se agregaron algunos más para tener mayor precisión en la configuración, y buen estándar de código)*
### utils.py
En este archivo se encuentran la gran mayoría de parámetros e hiperparámetros a modificar del modelo, junto a sus explicaciones.

Agregué algunos hiperparámetros que estaban hardcodeados, y saqué la necesidad de cambiar manualmente los hiperparámetros a la hora de testear (se guardan automáticamente con el método `save_checkpoint()` del agente)

### graph_env.py
Define el entorno donde el agente interactúa y aprende. Se define el grafo, las acciones posibles, el algoritmos de detección utilizado, las funciones de similaridad, etc...

Funciona como el "backend" del código, puesto que se encarga de la "simulación" del grafo, lo que implica llevar a cabo las acciones sobre el grafo, calcular las recompensas, definir las acciones posibles, y demás.
### agent.py
Define el agente. En `a2c.py` está la red neuronal, que devuelve un vector de probabilidades de cuales son los nodos que mejor cumplen la función de ocultar, y un valor estimado de "qué tan bueno" es el grafo actual si seguimos aplicando la política actual.

Hasta donde logré entender, no se marca directamente que un nodo es el nodo objetivo, sino que se deja que la red aprenda patrones por sí sola, y luego se restringe su actuar a un conjunto de nodos restringidos. Mi idea es cambiar esto para ver si obtengo alguna mejora.

#### plots de entrenamiento
Al finalizar el entrenamiento del agente, se generan los siguientes plots:
- `training_a_loss`
- `training_goal_reached`
- `training_goal_reward`
- `training_reward`
- `training_steps`
- `training_v_loss`

----
## Proceso de entrenamiento y testeo
### Entrenamiento
Primero, se establecen los hiperparámetros, datasets, algoritmos, etc...
Luego se hace grid search sobre los hiperparámetros (ver [[Variables, Parámetros y Ecuaciones]])

El entrenamiento es como cualquier otro de RL, donde se asignan los parámetros de las redes neuronales, se computan las pérdidas y se aplica backpropagation.

La cantidad de pasos de entrenamiento está determinada por `MAX_EPISODES` en `utils.py`, y utilizo el método 1 ([[Métodos de selección de comunidad#1. `RANDOM_COMMUNITY`|RANDOM_COMMUNITY]]) para garantizar que el agente se entrene sobre varios tipos de nodos/comunidades (notar que si $\epsilon = 0$ no va a haber cambio de nodo/comunidad).

### Testeo
El agente se pone en modo de evaluación, los hiperparámetros son cargados directamente del archivo `.pth` (para mayor comodidad y reproducibilidad), y se testea sobre uno o varios betas y taus.

Para pruebas "chicas" se puede utilizar el método 2 ([[Métodos de selección de comunidad#2. `CLOSEST_SIZE_PERCENTAGE`|CLOSEST_SIZE_PERCENTAGE]]), ya que actúa sobre distintos tamaños de comunidades. Si se quiere probar el rendimiento "real" del agente, es más recomendable el método 1.

Cabe aclarar que para cada test se define un "entorno" de testing, siendo el principal para el paper original `hiding_node.py` que define baselines (medidas para comparar) y demás funciones útiles

--- 
#### Cambios "importantes"
Al correr los tests del agente, se estaba cargando el checkpoint (parámetros, pesos, hiperparámetros...) de forma repetida. Ahora se carga una única vez antes de testear.

Además, en vez de computar los baselines cada vez que testeamos, los computo una única vez en todos los datasets para varios $\tau$ y $\beta$ (ahorra **mucho** tiempo de testing).

Como la idea es ir más allá del problema original, modularizamos el código para poder reutilizar el mismo agente y entorno para modelar distintos problemas.

Ver [[Cambios Importantes sobre la Red Neuronal]].