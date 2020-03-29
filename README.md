# SEIRS+ Model

Clonado del repositorio original de [SEIRS Model](https://github.com/ryansmcgee/seirsplus).
<br>

Este paquete implementa el modelo dinámico de enfermedades infecciosas generalizado SEIRS con extensiones que modelan el efecto de factores incluyendo la estructura poblacional, distancia social, pruebas de la enfermedad, rastreo de contactos y cuarentena de contactos detectados.

Notablemente, este paquete incluye una implementación estocástica de los modelos en redes dinámicas.

**README Contenidos:**
* [ Descripción del Modelo ](#model)
   * [ Dinámica de SEIRS ](#model-seirs)
   * [ Dinámica de SEIRS con pruebas ](#model-seirstesting)
   * [ Modelo Determinístico ](#model-determ)
   * [ Modelo de Red ](#model-network)
      * [ Modelo de Red con Pruebas, Rastreo de Contactos y Cuarentena ](#model-network-ttq)
* [ Utilización del Código ](#usage)
   * [ Guía Rápida ](#usage-start)
   * [ Instalación e Importación del paquete ](#usage-install)
   * [ Inicialización del Modelo ](#usage-init)
      * [ Modelo Determinístico ](#usage-init-determ)
      * [ Modelo de Red ](#usage-init-network)
   * [ Ejecutando el Modelo ](#usage-run)
   * [ Accediendo a los datos de Simulación ](#usage-data)
   * [ Cambiando parámetros en una simulación ](#usage-checkpoints)
   * [ Specifying Interaction Networks ](#usage-networks)
   * [ Visualización ](#usage-viz)
  
<a name="model"></a>
## Descripción del Modelo

<a name="model-seirs"></a>
### Dinámica de SEIRS

El fundamento de los modelos en este paquete es el clásico modelo SEIR de enfermedades infecciosas. El modelo SEIR es un modelo compartimental estándar en el cual la población es dividida en **susceptible (S)**, **expuesta (E)**, **infectada (I)**, y **recuperada (R)**. Un miembro susceptible de la población se convierte en expuesto (infección latente) cuando ha tenido contacto con un individuo infectado, y progresa hacia los estados de infección y recuperado. En el modelo SEIRS, individuos recuperados podrían ser suceptiles a recontagiarse después de cierto tiempo de recuperación (aunque la re-susceptibilidad podría ser excluida si se desea o no aplica). 
<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRS_diagram.png" width="400"></div>
</p>

Las tasas de transmisión entre estados están dadas por los parámetros:
* β: tasa de transmisión (transmisiones por S-I contacto por tiempo)
* σ: tasa de progresión (inverso del período de incubación)
* γ: tasa de recuperación (inverso del período de infección)
* ξ: tasa de re-susceptibilidad (inverso del período de inmunidad temporal; 0 si la inmunidad es permanente)
* μ<sub>I</sub>: tasa de mortalidad de la enfermedad (muertes por individuos infectados por tiempo)

<a name="model-seirstesting"></a>
### Dinámica de SEIRS con Pruebas de la enfermedad

El efecto de examinar la infección en la dinámica puede ser modelado introduciendo estados correspondientes a **exposición detectada (*D<sub>E</sub>*)** e **infección detectada (*D<sub>I</sub>*)**. Los individuos expuestos e infectados son examinados a tasas *θ<sub>E</sub>* y *θ<sub>I</sub>*, respectivamente, y las tasas de pruebas positivas son *ψ<sub>E</sub>* y *ψ<sub>I</sub>*, respectivamente  (la tasa de falsos positivos se asume ser cero, de modo que individuos susceptibles nunca dan positivo). Evaluaciones positivas mueven al individuo en estados de caso detectado, donde las tasas de transmisión, progresión, recuperación, y/o mortalidad (así como conectividad de la red en el modelo de red) podrían ser diferentes de los casos no detectados.

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRStesting_diagram.png" width="400"></div>
</p>

Las tasas de transición entre estados están dadas por los parámetros:
* β: tasa de transmisión (transmisiones por contacto S-I por tiempo)
* σ: tasa de progresión (inverso del período de incubación)
* γ: tasa de recuperación (inverso del período de infección)
* μ<sub>I</sub>: tasa de mortalidad de la enfermedad (muertes por infección individual por tiempo)
* ξ: tasa de re-susceptibilidad (inverso del período de inmudinad temporal; 0 si hay inmunidad permanente)
* θ<sub>E</sub>: tasa de evaluación de individuos expuestos 
* θ<sub>I</sub>: tasa de evaluación de infecciones individuales 
* ψ<sub>E</sub>: tasa de pruebas positivas de individuos expuestos
* ψ<sub>I</sub>: tasa de pruebas positivas de individuos infectados 
* β<sub>D</sub>: tasa de transmisión de casos detectados (transmisiones por S-D<sub>I</sub> contacto por tiempo)
* σ<sub>D</sub>: tasa de progresión de casos detectados (inverso del período de incubación)
* γ<sub>D</sub>: tasa de recuperación de casos detectados (inverso del período de infección)
* μ<sub>D</sub>: tasa de mortalidad dela enfermedad para los casos detectados (muertes por infecciones por tiempo)

*Dinámicas vitales también son consideradas en los modelos(opcionales, pero inactivas por default), pero no son discutidas en el README.* 

*Vea las [documentación de las ecuaciones del modelo](https://github.com/ryansmcgee/seirsplus/blob/master/docs/SEIRSplus_Model.pdf) para más información de las ecuaciones del modelo.*


<a name="model-determ"></a>
### Modelo Determinístico

La evolución de la dinámica de SEIRS descrita arriba puede ser explicada por el siguiente sistema de ecuaciones diferenciales. Es importante resaltar que esta versión del modelo es determinística y asume una población uniformemente distribuida. 

#### Dinámica de SEIRS

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRS_deterministic_equations.png" width="210"></div>
</p>

donde *S*, *E*, *I*, *R*, y *F* son los números de individuos susceptibles, expuestos, infectados, recuperados, y fallecidos, respectivamente, y *N* es el número total de individuos en la población (parámetros que se describen arriba).

#### Dinámica de SEIRS aplicando Pruebas de Infección

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRStesting_deterministic_equations.png" width="400"></div>
</p>

donde *S*, *E*, *I*, *D<sub>E</sub>*, *D<sub>I</sub>*, *R*, y *F* son los números de individuos susceptibles, expuestos, infectados, expuestos detectados, infectados detectados, recuperados, y fallecidos, respectivamente, y *N* es el número total de individuos en la población  (parámetros que se describen arriba).

<a name="model-network"></a>
### Modelo de Red

El modelo SEIRS estándar captura características importantes de la dinámica de las infecciones, pero es determinístico y asume una distribución poblacional uniforme (donde cada individuo es igualmente propenso a interactuar con otro individuo). Sin embargo, es importante considerar los efectos estocásticos y la estructura de la red de contacto cuando se estudia transmisión de enfermedades y el efecto de intervenciones como distanciamiento social y rastreo de contactos.

Este paquete incluye la implementación de la dinámica de SEIRS en redes estocásticas con comportamiento dinámico. 
Esto sirve para analizar la relación entre la estructura de la red y las tasas de transmisión efectivas, incluido el efecto de las intervenciones basadas en la restricción de la red de contagios, como el distanciamiento social, la cuarentena y el rastreo de contactos.

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/network_contacts.png" height="250">

Considere un grafo **_G_** representando individuos (nodos) y sus interacciones (aristas/lados). Cada individuo (nodo) tiene un estado (*S, E, I, D<sub>E</sub>, D<sub>I</sub>, R, or F*). El conjunto de nodos adyacentes (conectado por una arista) a un individuo define el conjunto de "contactos cercanos" (resaltados en negro). En determinado instante, cada individuo hace contacto con un indivduo aleatorio de su red de contactos cercanos con probabilidad *(1-p)β* o con un individuo aleatorio de toda cualquier punto de la red (resaltado en azul) con probabilidad *pβ*. Los últimos contactos globales representan individuos interactuando con la población en general (es decir, individuos fuera del círculo social, como en el transporte público, en un evento, etc.) con cierta probabilidad. Cuando un individuo susceptible interactúa con un individuo infeccioso, queda expuesto. El parámetro *p* define la localización de la red: para *p=0* un individuo solo interactua con sus contactos cercanos, mientras que *p=1* representa una población mezclada uniformemente. Las intervenciones de distanciamiento social podrían incrementar la focalización de la red (reducir *p*) y/o reducir la conectividad local de la red (reducir el grado de los individuos).

Cada nodo *i* representa un individuo que se encuentra en estado *X<sub>i</sub>*, el cual puede transicionar hacia otro estado de acuerdo a las siguientes probabilidades: 

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRSnetwork_transitions.png" width="500"></div>
</p>

donde *δ<sub>Xi=A</sub> = 1* representa que el estado de *X_i* es *A*, o *0* si no lo es, y donde *C<sub>G</sub>(i)* denota el conjunto de contactos cercanos al nodo *i*. Para poblaciones grandes y *p=1*, este modelo estocástico aborda la misma dinámica como el modelo determinístico SEIRS.

Esta implementación se basa en el trabajo de Dottori et al. (2015).
* Dottori, M. and Fabricius, G., 2015. SIR model on a dynamical network and the endemic state of an infectious disease. Physica A: Statistical Mechanics and its Applications, 434, pp.25-35.

<a name="model-network-ttq"></a>
#### Modelo de Red con Pruebas de la Enfermedad, Rastreo de Contactos y Cuarentena

##### Pruebas y Rastreo de Contactos

Como en el modelo determinístico, individuos expuestos e infectados son evaluados a tasas *θ<sub>E</sub>* y *θ<sub>I</sub>*, respectivamente, y estos arrojan resultan infectados a tasas *ψ<sub>E</sub>* y *ψ<sub>I</sub>*, respectivamente (la tasa de falsos positivos es asumida como cero, así que personas susceptibles nunca dan positivo). Al evaluar positivamente un individuo lo mueve al estado de caso detectado (*D<sub>E</sub>* o *D<sub>I</sub>*), donde las tasas de transmisión, progresión, recuperación, y/o mortalidad (así como la conectividad de la red en el modelo) puede ser diferentes que en los casos no detectados.

Considerar la interacción de la red nos permite modelar el rastreo de contactos, donde los contactos más cercanos de un individuo detectado como positivo son más propensos a ser evaluados a la enfermedad. En este modleo, un individuo se examina debido al rastreo de contactos a una tasa de *φ* veces el número de contactos que ha dado positivo.

##### Cuarentena

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/network_contacts_quarantine.png" height="250">

Consideremos también otro grafo **_Q_** el cual representa las interacciones que cada individuo tiene si ellos son evaluados como contagiados de la enfermedad (por ejemplo individuos en estados *D<sub>E</sub>* o *D<sub>I</sub>*) y entran en cuarentena.
La cuarentena tiene el efecto de dejar caer una fracción de las aristas que conectan al individuo en cuarentena con otros (de acuerdo con una regla de elección del usuario al generar el grafo *Q*). Las aristas de *Q* (resaltados en púrpura) para cada individuo son entonces un subconjunto de las aristas normales de * G * para ese individuo. El conjunto de nodos adyacentes a un individuo en cuarentena define su conjunto de "contactos en cuarentena" (resaltados en púrpura). En un momento dado, un individuo en cuarentena puede entrar en contacto con otro individuo en esa cuarentena con probabilidad *(1-p) β <sub> D </sub>*. Una persona en cuarentena también puede ponerse en contacto con una persona aleatoria desde cualquier lugar de la red con una tasa *qp β<sub>D</sub>*.

Cada nodo *i* tiene un estado *X<sub>i</sub>* que se actualiza de acuerdo a las siguientes probabilidades de trasición:

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRSnetworktesting_transitions.png" width="800"></div>
</p>

donde *δ<sub>Xi=A</sub>=1* si el estado de  *X<sub>i</sub>* es *A*, o *0* si no lo es, y donde *C<sub>G</sub>(i)* y *C<sub>Q</sub>(i)* denota el conjunto de contactos cercanos y contactos de cuarentena del nodo *i*, respectivamente. Para poblaciones grandes y *p=1*, este modelo estocástico aborda la misma dinámica que el modeo determinístico SEIRS (sin rastreo de contactos, que no estpa incluido en el modelo uniforme-mixto).

<a name="usage"></a>
## Utilización de Código

Este paquete está diseñado para ser utilizado ampliamente. Escenarios complejos pueden ser simulados con muy pocas líneas de código o, en muchos casos, sin ninguna codificación o conocimientos de python simplemente modificando los parámetros en las notebooks de ejemplo. Vea la sección Quick Start y el resto de la documentación para más detalles.

No se deje engañar por la longitud del archivo README, ejecutar estos modelos es rápido y fácil. El paquete hace todo el trabajo duro por usted. Por ejemplo, aquí hay un script completo que simula las dinámicas de SEIRS en una red con distanciamiento social, pruebas, rastreo de contactos y cuarentena en solo 10 líneas de código (vea las [notebooks de ejmplo](https://github.com/ryansmcgee/seirsplus/tree/master/examples) para más detalles de este ejemplo):
```python
# Importando librerías
from seirsplus.models import *
import networkx

numNodes = 10000 # Número de nodos del modelo
baseGraph    = networkx.barabasi_albert_graph(n=numNodes, m=9) # Grafo inicial

# Grafo Exponencial con escala de 100
G_normal     = custom_exponential_graph(baseGraph, scale=100) 

# Interacciones con distanciamiento social:
G_distancing = custom_exponential_graph(baseGraph, scale=10)

# Interacciones con Cuarentena:
G_quarantine = custom_exponential_graph(baseGraph, scale=5)

#Modelo SERIRS de Red
model = SEIRSNetworkModel(G=G_normal, beta=0.155, sigma=1/5.2, gamma=1/12.39, mu_I=0.0004, p=0.5,
                          Q=G_quarantine, beta_D=0.155, sigma_D=1/5.2, gamma_D=1/12.39, mu_D=0.0004,
                          theta_E=0.02, theta_I=0.02, phi_E=0.2, phi_I=0.2, psi_E=1.0, psi_I=1.0, q=0.5,
                          initI=10)

checkpoints = {'t': [20, 100], 'G': [G_distancing, G_normal], 'p': [0.1, 0.5], 'theta_E': [0.02, 0.02], 'theta_I': [0.02, 0.02], 'phi_E':   [0.2, 0.2], 'phi_I':   [0.2, 0.2]}

#Corriendo la simulación para T=300
model.run(T=300, checkpoints=checkpoints)

# Mostrando las infecciones
model.figure_infections()
```

<a name="usage-start"></a>
### Quick Start

El directorio de [```ejemplos```](https://github.com/ryansmcgee/seirsplus/tree/master/examples) contiene dos Jupyter notebooks: una para el modelo determinístico y otra para el [modelo de red](https://github.com/ryansmcgee/seirsplus/blob/master/examples/network_model_demo.ipynb). Estas notebooks recorren simulaciones completas utilizando cada uno de estos modelos con una descripción de los pasos involucrados.

**Estas notebooks también pueden servir como entornos limitados listos para ejecutar para probar sus propios escenarios de simulación simplemente cambiando los valores de los parámetros en la notebook.**

<a name="usage-install"></a>
### Instalando e Importando los paquetes

Todo el código necesario para ejecutar el modelo se importa desde el módulo ```models``` de este paquete.

#### Instalando el paquete usando ```pip```
El paquete puede ser instalado en su máquina escribiendo en la command line:

```> pip install seirsplus```

o bien:

```> sudo pip install seirsplus```

Entonces, el módulo ```models``` puede ser importado en los scripts como se muestra a continuación:

```python
from seirsplus.models import *
import networkx
```

#### *Alternativamente, también puede copiar y pegar el código en su máquina*

*Puede usar el código del modelo sin instalar un paquete copiando el archivo del módulo ```models.py``` en un directorio de su máquina. En este caso, la forma más fácil de usar el módulo es colocar sus scripts en el mismo directorio que el módulo e importar el módulo como se muestra aquí: *

```python
from models import *
```
<a name="usage-init"></a>
### Inicializando el Modelo

<a name="usage-init-determ"></a>
#### Modelo Determinístico

Todos los parámetros del modelo, incluidas las redes de interacción de cuarentena normales y opcionales, se establecen en la llamada al constructor ```SEIRSModel``` constructor. Los parámetros básicos de SEIR ```beta```, ```sigma```, ```gamma```, y ```initN``` son los únicos argumentos requeridos. Todos los demás argumentos representan parámetros para la dinámica del modelo extendido opcional; estos parámetros opcionales toman valores predeterminados que desactivan su dinámica correspondiente cuando no se proporcionan en el constructor.

Constructor Argumento | Descripción del Parametero | Tipo de Dato | Valor Default
-----|-----|-----|-----
```beta   ``` | tasa de transmisión | float | REQUIRED
```sigma  ``` | tasa de progresión | float | REQUIRED
```gamma  ``` | tasa de recuperación | float | REQUIRED
```xi     ``` | tasa de re-susceptibilidad | float | 0
```mu_I   ``` | tasa de mortalidad relacionada a la infección | float | 0
```mu_0   ``` | tasa de mortalidad base | float | 0 
```nu     ``` | tasa de nacimiento base | float | 0 
```beta_D ``` | tasa de transmisión para casos detectados | float | None (set equal to ```beta```) 
```sigma_D``` | tasa de progresión para detected cases | float | None (set equal to ```sigma```)  
```gamma_D``` | tasa de recuperación para casos detectados | float | None (set equal to ```gamma```)  
```mu_D   ``` | tasa de mortalidad de casos detectados | float | None (set equal to ```mu_I```) 
```theta_E``` | tasa de evaluación de individuos expuestos | float | 0 
```theta_I``` | tasa de evaluación de individuos infectados | float | 0 
```psi_E  ``` | probabilidad de tests positivos para individuos expuestos | float | 0 
```psi_I  ``` | probabilidad de test positivos para individuos infectados | float | 0
```initN  ``` | Número total de individuos inicial | int | 10
```initI  ``` | Número total de individuos infectados | int | 10
```initE  ``` | Número total de individuos expuestos | int | 0 
```initD_E``` | Número total de individuos infectados detectados | int | 0 
```initD_I``` | Número total de individuos detectados expuestos | int | 0 
```initR  ``` | Número total de individuos recuperados | int | 0
```initF  ``` | Número total de individuos fallecidos | int | 0

##### SEIR Básico

```python
model = SEIRSModel(beta=0.155, sigma=1/5.2, gamma=1/12.39, initN=100000, initI=100)
```


##### SEIRS Básico

```python
model = SEIRSModel(beta=0.155, sigma=1/5.2, gamma=1/12.39, xi=0.001, initN=100000, initI=100)
```

##### SEIR con pruebas de enfermedades y diferentes tasas de progresión para casos detectados (donde los parámetros de evaluación ```theta``` y ```psi``` > 0, y se proporcionan parámetros para casos detectados)

```python
model = SEIRSModel(beta=0.155, sigma=1/5.2, gamma=1/12.39, initN=100000, initI=100,
                   beta_D=0.100, sigma_D=1/4.0, gamma_D=1/9.0, theta_E=0.02, theta_I=0.02, psi_E=1.0, psi_I=1.0)
```


<a name="usage-init-network"></a>
#### Modelo de Red

Todos los valores de los parámetros del modelo, incluida la red de interacción y la red de cuarentena (opcional), se configuran en la llamada al constructor ```SEIRSNetworkModel```. La red de interacción ```G``` y los parámetros básicos de SEIR ```beta```, ```sigma```, y ```gamma``` son los únicos argumentos requeridos. Todos los demás argumentos representan parámetros opcionales para la dinámica del modelo extendido; estos parámetros opcionales toman valores predeterminados que desactivan su dinámica correspondiente cuando no se proporcionan en el constructor.

**_Poblaciones Heterogeneas:_** A los nodos se les pueden asignar diferentes valores para un parámetro dado pasando una lista de valores (con longitud = número de nodos) para ese parámetro en el constructor. Todos los parámetros del constructor se enumeran y describen a continuación, seguidos de ejemplos de casos de uso para diversas elaboraciones del modelo que se muestran a continuación.

Argumento del Constructor | Descripción del Parámetro | Tipo de Dato | Valor Default
-----|-----|-----|-----
```G      ``` | grafo expecificando la red de interacciones | ```networkx Graph``` or ```numpy 2d array```  | REQUIRED 
```beta   ``` | tasa de transmisión | float | REQUIRED
```sigma  ``` | tasa de progresión | float | REQUIRED
```gamma  ``` | tasa de recuperación | float | REQUIRED
```xi     ``` | tasa de re-susceptibilidad | float | 0
```mu_I   ``` | tasa de mortalidad de infectados | float | 0
```mu_0   ``` | tasa de mortalidad base | float | 0 
```nu     ``` | tasa de nacimiento base | float | 0 
```p      ``` | probabilidad de interacciones globales (red focalizada) | float | 0
```Q      ``` | grafo especificando el efecto de la cuarentena en la red de interacciones | ```networkx Graph``` or ```numpy 2d array``` | None 
```beta_D ``` | tasa de transmisión de casos detectados | float | None (set equal to ```beta```) 
```sigma_D``` | tasa de progresión de casos detectados | float | None (set equal to ```sigma```)  
```gamma_D``` | tasa de recuperación de casos detectados | float | None (set equal to ```gamma```)  
```mu_D   ``` | tasa de mortalidad de infectados de casos detectados | float | None (set equal to ```mu_I```) 
```theta_E``` | tasa de examinación para individuos expuestos | float | 0 
```theta_I``` | tasa de examinación para individuos infectados | float | 0 
```phi_E  ``` | tasa de rastreo de contactos para individuos expuestos | float | 0 
```phi_I  ``` | tasa de rastreo de contactos para individuos infectados | float | 0 
```psi_E  ``` | probabilidad de examen positivo para individuos expuestos| float | 0 
```psi_I  ``` | probabilidad de examen positivo para individuos infectados| float | 0
```q      ``` | probabilidad de interacciones globales para individuos en cuarentena | float | 0
```initI  ``` | número inicial de individuos infectados | int | 10
```initE  ``` | número inicial de individuos expuestos | int | 0 
```initD_E``` | número inicial de individuos infectados detectados | int | 0 
```initD_I``` | número inicial de individuos expuestos detectados | int | 0 
```initR  ``` | número inicial de individuos recuperados | int | 0
```initF  ``` | número inicial de individuos fallecidos | int | 0

##### Modelo básico SEIR en una red

```python
model = SEIRSNetworkModel(G=myGraph, beta=0.155, sigma=1/5.2, gamma=1/12.39, initI=100)
```


##### Modelo básico SEIRS en una red

```python
model = SEIRSNetworkModel(G=myGraph, beta=0.155, sigma=1/5.2, gamma=1/12.39, xi=0.001, initI=100)
```


##### Modelo SEIR en una red con interacciones globales (p>0)

```python
model = SEIRSNetworkModel(G=myGraph, beta=0.155, sigma=1/5.2, gamma=1/12.39, p=0.5, initI=100)
```

##### Modelo SEIR en una red con pruebas de enfermedad y cuarentena (```theta``` y ```psi``` parámetros de pruebas de enfermedad > 0, red de cuarentena ```Q``` es proveída)

```python
model = SEIRSNetworkModel(G=myNetwork, beta=0.155, sigma=1/5.2, gamma=1/12.39, p=0.5,
                          Q=quarantineNetwork, q=0.5,
                          theta_E=0.02, theta_I=0.02, psi_E=1.0, psi_I=1.0, 
                          initI=100)
```

##### Modelo SEIR en una red con pruebas de enfermedad, cuarentena, y rastreo de contactos (```theta``` y ```psi``` parámetros de pruebas de enfermedad > 0, red de cuarentena ```Q``` es proveída , el rastreo de contacto ```phi``` > 0)

```python
model = SEIRSNetworkModel(G=myNetwork, beta=0.155, sigma=1/5.2, gamma=1/12.39, p=0.5,
                          Q=quarantineNetwork, q=0.5,
                          theta_E=0.02, theta_I=0.02, phi_E=0.2, phi_I=0.2, psi_E=1.0, psi_I=1.0,  
                          initI=100)
```

<a name="usage-run"></a>
### Corriendo el Modelo

Las dinámicas de una red estocástica del modelo SEIS son simuladas usando el algoritmo de Gillepsie.

Una vez el modelo esté inicializado, la simulación se puede correr llamando la siguiente función:

```python
model.run(T=300)
```

La función ```run()``` tiene los siguientes argumentos

Argumento | Descripción | Tipo de Dato | Valor Default
-----|-----|-----|-----
```T``` | tiempo de simulación | numeric | REQUIRED
```checkpoints``` | dicionario de checkpoints (ver la siguiente sección) | dictionary | ```None```
```print_interval``` | (solo para modelo de red) intervalo de tiempo a imprimir el estatus de la simulación en la consola | numeric | 10
```verbose``` | Si es ```True```, imprime el conteo en cada estado en el intervalo de impresión, si es falso solo imprime el tiempo | bool | ```False```

<a name="usage-run"></a>
### Accediendo a los datos de  Simulación

Los valores de los parámetros del modelo y las series temporales variables generadas por la simulación se almacenan en los atributos de ```SEIRSModel``` o ```SEIRSNetworkModel``` que esté siendo utilizado, y pueden ser accedidos como se describe a continuación:

```python
S = model.numS      # serie de tiempo de S
E = model.numE      # serie de tiempo de E
I = model.numI      # serie de tiempo de I
D_E = model.numD_E    # serie de tiempo de D_E
D_I = model.numD_I    # serie de tiempo de D_I
R = model.numR      # serie de tiempo de R
F = model.numF      # serie de tiempo de F

t = model.tseries   # valores de tiempo correspondientes a las series de arriba

G_normal     = model.G    # grafo de red de interacción
G_quarantine = model.Q    # grafo de red de cuarentena

beta = model.beta   # valor del parámetro beta (o lista de valores de beta para cada nodo si se usa un modelo de red)
# similar a los otros parámetros
```
*Nota: el paquete incluye métodos para graficar esos valores. Ver más adelante*

<a name="usage-networks"></a>
### Especificando Redes de Interacción

Este modelo incluye un modelo de la dinámica poblacional de SERIS con una red de interacción estructurada (diferente a los modelos determinísticos estándar SIR/SEIR/SEIRS, los cuales asumen una población distribuida uniformemente). Cuando se usa el modelo de red, se debe especificar un grafo que represente la red de interacción de la población, donde cada nodo representa a un individuo en la población y las aristas conectan a individuos que tienen interacciones regulares.

La red de interacción puede ser especificada por un objeto **```Graph```** de la librería **```networkx```** o un **arreglo 2D de ```numpy```** reprsentando la matriz de adyacencia, cualquiera de los cuales puede ser definido y generado por cualquier método.

Este modelo SEIRS + también implementa dinámicas correspondientes a las pruebas de individuos para detectar la enfermedad y el traslado de individuos con infecciones detectadas a un estado en el que su tasa de recuperación, mortalidad, etc. puede ser diferente. Además, dado que este modelo considera a los individuos en una red de interacción, se puede especificar un gráfico separado que define las interacciones para los individuos con casos detectados (es decir, la red de "interacción de cuarentena").

Los escenarios de epidemia de interés a menudo implican redes de interacción que cambian con el tiempo. Se pueden definir y utilizar múltiples redes de interacción en diferentes momentos en la simulación del modelo utilizando la función de puntos de control (descrita en la sección a continuación).

**_Nota:_** *El tiempo de simulación aumenta con el tamaño de la red. Las redes pequeñas simulan rápidamente, pero tienen más volatilidad estocástica. Las redes con ~ 10,000 son lo suficientemente grandes como para producir dinámicas de población per cápita que generalmente son consistentes con las de redes más grandes, pero lo suficientemente pequeñas como para simular rápidamente. Recomendamos el uso de redes con ~ 10,000 nodos para la creación de prototipos de parámetros y escenarios, que luego se pueden ejecutar en redes más grandes si se requiere más precisión*

#### Grafo Exponencial Personalizado

Las redes de interacción humana a menudo se asemejan a redes sin escala con distribuciones de grados exponenciales.
Este paquete incluye una función que genera grafos de distribuciones de grados exponenciales y dos colas exponenciales llamada ```custom_exponential_graph()```. El método de generar estos gráficos también facilita la eliminación de aristas de un grafo de referencia y disminuye el grado de la red, lo cual es útil para generar redes que representan el distanciamiento social y las condiciones de cuarentena.

Algoritmos comunes para generar grafos de ley de potencia, como el algoritmo de apego preferencial Barabasi-Albert, producen grafos que tienen un grado mínimo; es decir, ningún nodo tiene menos de *m* aristas para algún valor de *m*, lo que no es realista para las redes de interacción. La función ```custom_exponential_graph()``` simplemente produce grafos con distribuciones de grados que tienen un pico cerca de sus medias y colas exponencial en la dirección de los grados alto y bajo. (No se hacen afirmaciones sobre el realismo o el rigor de estos gráficos).

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/degreeDistn_compareToBAGraph1.png" height="250">

La función genera grafos usando el siguiente algoritmo:
* Inicia con un grafo del tipo Barabasi-Albert (o cualquier gráfico que sea opcionalmente provisto por el usuario).
* Para cada nodo:
    * Contar el número de vecinos *n*  del nodo 
    * Genera un número aleatorio *r* de una distribución exponencial con media=```scale```. Si *r>n*, actualizar a *r=n*. 
    * Seleccionar aleatoriamente *r* vecinos del nodo que se mantendrán, borrar las aristas de todos los otros vecinos. 
Cuando se inicialice el grafo Barabasi-Albert (BA), esto genera una nueva gráfica que tiene un pico en su media y colas aproximadamente exponenciales en ambas direcciones, como se muestra a la derecha.


<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/degreeDistn_compareToBAGraph4.png" height="250">

Como este algoritmo comienza con un grafo con conexiones definidas y crea un nuevo grafo al romper un cierto número de conexiones, también facilita tomar un gráfico existente y hacer un subgrafo que también tiene colas exponenciales y media movida a la izquierda. Esto se puede utilizar para generar subgrafos de distanciamiento social y cuarentena. La magnitud de la ruptura de reducción de aristas se modula mediante el parámetro ```scale```. A la derecha hay algunos ejemplos de gráficos con un grado medias progresivamente más pequeñas generadas utilizando el mismo grafo de referencia (del tipo Barabasi-Albert), que son resultados particulares del conjunto de aristas iniciales.

La función ```custom_exponential_graph()``` tiene los siguientes argumentos

base_graph=None, scale=100, min_num_edges=0, m=9, n=None
Argumento | Descripción | Tipo de Dato | Valor por Default
-----|-----|-----|-----
```base_graph``` | Grafo para inicializar el algoritmo. Si ```None```, generará un grafo del tipo Barabasi-Albert para usar como el punto de partida con parámetros ```n``` y ```m``` | objeto ```networkx``` ```Graph``` | ```None```
```scale``` | Media de la distribución exponencial utilizada para generar el ```base_graph``` a manterner. Valores grandes darán como resultado grafos que se aproximan al ```base_graph``` original, y valores muy pequeños darán como resultado en grafos más dispersos que  el original ```base_graph```  | numeric | 100
```min_num_edges``` | Número mínimo de aristas que todos los nodos deben tener en el grafo generado | int | 0
```n``` | Parámetro *n* del algoritmo Barabasi-Albert (número de nodos a añadir) | int | ```None``` (valor requerido cuando no se da un grafo inicial ```base_graph```)
```m``` | Parámetro *m* del algoritmo Barabasi-Albert (número de aristas a añadir a cada nodo) | int | 9

<a name="usage-checkpoints"></a>
### Cambiando parámetros durante una simulación

Los parámetros del modelo pueden ser cambiados durante una simulación usando checkpoints. Un diccionario contiene una lista de tiempos de checkpoint (```checkpoints['t']```) en los que los parámetros van a cambiar, así como las listas de nuevos valores a asignar a los parámetros del modelo. 

Por ejemplo, se puede correr una simulación con estos nuevos valores usando ```checkpoints```:
```python
checkpoints = {'t':       [20, 100], 
               'G':       [G_distancing, G_normal], 
               'p':       [0.1, 0.5], 
               'theta_E': [0.02, 0.02], 
               'theta_I': [0.02, 0.02], 
               'phi_E':   [0.2, 0.2], 
               'phi_I':   [0.2, 0.2]}
```

*Los checkpoints mostrados aqupi corresponden al inicio del distanciamiento social y tiempo de evaluación en ```t=20``` (el grafo ```G``` se actualiza al nuevo grafo con distanciamiento ```G_distancing``` y el parámetro de localización ```p``` disminuye a ```0.1```; los parámetros de evaluación ```theta_E```, ```theta_I```, ```phi```, y ```phi_I``` son establecidos a valores no nulos) y por último deteniendo el distanciamiento social en tiempo ```t=100``` (```G``` y ```p``` regresando a sus valores normales; los parámetros de evaluación permanencen diferentes a cero).*

Cualquier parámetro del modelo definido en el constructor puede ser actualizado de esta forma. Solo parámetros que son incluidos en el diccionario de checkpoint actualizan sus valores, los demás permanecen estáticos.

Los casos de uso de esta función incluyen: 

* Parámetros cambiantes durante una simulación, como tasas de transición o tasas de evaluación cada día, semana, o en una secuencia específica de días, etc.
* Inicial y finalización de las intervencionesm tales como el distanciamiento social (cambiando las redes de interacción), rastreo de contactos y evaluaciones (estableciendo parámetros relevantes a valores diferentes), etc.

**_Simulaciones consecutivas_**: *También se puede correr el mismo modelo múltiples veces. Cada vez que se ejecuta la función ```run()``` se reinicia una nueva simulación desde el estado en que finalizó la simulación previa. 
Por ejemplo:*
```python
model.run(T=100)    # simula el modelo 100 días
# ... 
# hace otras cosas, como simular datos o cambiar parámetros  
# ...
model.run(T=200)    # simula el modelo 200 veces días más, iniciando donde la primera simulación concluyó
```

<a name="usage-viz"></a>
### Visualización

### Visualizando resultados
Las clases ```SEIRSModel``` y ```SEIRSNetworkModel``` tienen una función ```plot()``` para graficar los resultados de las simulaciones en matplotlib. Esta función genera una linea de frecuencia de cada estado del modelo en la población, pero hay muchos argumentos que pueden ser utilizados para personalizar la gráfica.

Esas clases también tienen funciones para generar una figura completa a partir de los resultados de la simulación del modelo (opcionalmente, se pueden proporcionar argumentos para personalizar los gráficos generados por estas funciones, ver más abajo).

- ```figure_basic()``` llama la función ```plot()``` con parámetros con parámetros predeterminados para generar una gráfica de líneas de la frecuencia de cada estado en la población.
- ```figure_infections()``` llama la función ```plot()``` con parámetros con parámetros predeterminados para generar una gráfica de áreas de la frecuencia de los estados de infección en la población (*E*, *I*, *D<sub>E</sub>*, *D<sub>I</sub>*).

Los parámetros que puede ser suministrado a cualquiera de las funciones incluyen:
Argument | Description 
-----|-----
```plot_S``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```plot_E``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```plot_I``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```plot_R``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```plot_F``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```plot_D_E``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```plot_D_I``` | ```'line'```, ```shaded```, ```'stacked'```, or ```False``` 
```combine_D``` | ```True``` or ```False```
```color_S``` | matplotlib color of line or stacked area
```color_E``` | matplotlib color of line or stacked area
```color_I``` | matplotlib color of line or stacked area
```color_R``` | matplotlib color of line or stacked area
```color_F``` | matplotlib color of line or stacked area
```color_D_E``` | matplotlib color of line or stacked area
```color_D_I``` | matplotlib color of line or stacked area
```color_reference``` | matplotlib color of line or stacked area
```dashed_reference_results``` | ```seirsplus``` model object containing results to be plotted as a dashed-line reference curve
```dashed_reference_label``` | ```string``` for labeling the reference curve in the legend
```shaded_reference_results``` | ```seirsplus``` model object containing results to be plotted as a dashed-line reference curve
```shaded_reference_label``` | ```string``` for labeling the reference curve in the legend
```vlines``` | ```list``` of x positions at which to plot vertical lines
```vline_colors``` | ```list``` of ```matplotlib``` colors corresponding to the vertical lines
```vline_styles``` | ```list``` of ```matplotlib``` ```linestyle``` ```string```s corresponding to the vertical lines
```vline_labels``` | ```list``` of ```string``` labels corresponding to the vertical lines
```ylim``` | max y-axis value 
```xlim``` | max x-axis value
```legend``` | display legend, ```True``` or ```False```
```title``` | ```string``` plot title
```side_title``` | ```string``` plot title along y-axis
```plot_percentages``` | if ```True``` plot percentage of population in each state, else plot absolute counts
```figsize``` | ```tuple``` specifying figure x and y dimensions
```use_seaborn``` | if ```True``` import ```seaborn``` and use ```seaborn``` styles
