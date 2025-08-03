### Configuración

Los experimentos se llevaron a cabo en una instancia EC2 de AWS del tipo [`g4dn.xlarge`](https://aws.amazon.com/ec2/instance-types/g4/), con las siguientes especificaciones:

- **vCPUs:** 4 (Intel Cascade Lake a 2.5 GHz, arquitectura de 24 núcleos)
- **Memoria:** 16 GiB de RAM
- **GPU:** NVIDIA T4
- **Sistema operativo:** Ubuntu 24.04

Se utilizó la siguiente imagen de máquina de Amazon (AMI):

> [**Deep Learning Base OSS Nvidia Driver GPU AMI (Ubuntu 24.04) 20250620**](https://docs.aws.amazon.com/dlami/latest/devguide/aws-deep-learning-base-gpu-ami-ubuntu-24-04.html)

Ver [[Código]] para una explicación de cómo ejecutar los experimentos y qué función cumple cada archivo.
## Datasets, parámetros y configuración por defecto
En [[readme_datasets]] tenemos la descripción de cada dataset.

### 1. Replicar el paper
La documentación existente es excelente, basta con correr `main.py` (luego de instalar las dependencias necesarias, corremos primero con `--mode "train"` y luego `--mode "test"`).

### 2. Cambiar la semilla de la generación aleatoria de números
Comenzamos con algo sencillo. Al utilizar aleatoriedad en los métodos de cómputo, lógicamente se espera una leve diferencia, pero no debería notarse si el experimento está bien diseñado.