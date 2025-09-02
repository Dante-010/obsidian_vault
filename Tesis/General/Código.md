### main.py
Lógicamente, el archivo principal es `main.py`. Como bien se aclara en [[README_main]], utilizamos los siguientes comandos para entrenar y probar el modelo, respectivamente:
```shell
python main.py --mode "train"
```
```shell
python main.py --mode "test"
```

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

### utils.py
En este archivo se encuentran la gran mayoría de parámetros e hiperparámetros a modificar del modelo, junto a sus explicaciones.

Agregué algunos hiperparámetros que estaban hardcodeados, y saqué la necesidad de cambiar manualmente los hiperparámetros a la hora de testear (se guardan 

automáticamente con el método `save_checkpoint()` del agente)

### graph_env.py
Define el entorno donde el agente interactúa y aprende. Se define el grafo, las acciones posibles, el algoritmos de detección utilizado, las funciones de similaridad, etc...

### agent.py
Define el agente. En `a2c.py` está la red neuronal, que devuelve un vector de probabilidades de cuales son los nodos que mejor cumplen la función de ocultar, y un valor estimado de "qué tan bueno" es el grafo actual si seguimos aplicando la política actual.
Hasta donde logré entender, no se marca directamente que un nodo es el nodo objetivo, sino que se deja que la red aprenda patrones por sí sola, y luego se restringe su actuar a un conjunto de nodos restringidos (en base al nodo objetivo).