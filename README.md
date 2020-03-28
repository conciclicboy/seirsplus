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

El fundamento de los modelos en este paquete es el clásico modelo SEIR de enfermedades infecciosas. El modelo SEIR es un modelo compartimental estándar en el cual la población es dividida en **susceptible (S)**, **expuesta (E)**, **infectada (I)**, and **recuperada (R)**. Un miembro susceptible de la población se convierte en expuesto (infección latente) cuando ha tenido contacto con un individuo infectado, y progresa hacia los estados de infección y recuperado. En el modelo SEIRS, individuos recuperados podrían ser suceptiles a recontagiarse después de cierto tiempo de recuperación (aunque la re-susceptibilidad podría ser excluida si se desea o no aplica). 
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

El efecto de examinar la infección en la dinámica puede ser modelado introduciendo estados correspondientes a **exposición detectada (*D<sub>E</sub>*)** e **infección detectada (*D<sub>I</sub>*)**. Individuos expuestos e infectados son examinados a tasas *θ<sub>E</sub>* y *θ<sub>I</sub>*, respectivamente, y examinados test positivamente por la infección con tasas *ψ<sub>E</sub>* y *ψ<sub>I</sub>*, respectivamente  (la tasa de falsos positivos se asume ser cero, de modo que individuos susceptibles nunca dan positivo). Evaluaciones positivas mueven al individuo en estados de caso detectado, donde las tasas de transmisión, progresión, recuperación, y/o mortalidad (así como conectividad de la red en el modelo de red) podrían ser diferentes de los casos no detectados.

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRStesting_diagram.png" width="400"></div>
</p>

Las tasad de transición entre estados están dadas por los parámetros:
* β: tasa de transmisión (transmisiones por contacto S-I contact por tiempo)
* σ: tasa de progresión (inverso del período de incubación)
* γ: tasa de recuperación (inverso del período de infección)
* μ<sub>I</sub>: tasa de mortalidad de la enfermedad (muertes por infección individual por tiempo)
* ξ: tasa de re-susceptibilidad (inverso del período de inmudinad temporal; 0 si hay inmunidad permanente)
* θ<sub>E</sub>: tasa de evaluación de individuos expuestos 
* θ<sub>I</sub>: tasa de evaluación de infeccionesn individuales 
* ψ<sub>E</sub>: tasa de pruebas positivas de individuos expuestos
* ψ<sub>I</sub>: tasa de pruebas positivas para individuos infectados 
* β<sub>D</sub>: tasa de transmisión de casos detectados (transmisiones por S-D<sub>I</sub> contacto por tiempo)
* σ<sub>D</sub>: tasa de progresión de casos detectados (inverso del período de incubación)
* γ<sub>D</sub>: tasa de recuperación de casos detectados (inverso del período de infección)
* μ<sub>D</sub>: tasa de mortalidad dela enfermedad para los casos detectedos (muertes por infecciones por tiempo)

*Dinámicas vitales también son consideradas en los modelos(opcionales, pero inactivas por default), pero no son discutidas en el README.* 

*Vea las [documentación de las ecuaciones del modelo](https://github.com/ryansmcgee/seirsplus/blob/master/docs/SEIRSplus_Model.pdf) para más información de las ecuaciones del modelo.*


<a name="model-determ"></a>
### Modelo Determinístico

La evolución de la dinpamica de SEIRS descrita arriba puede ser explicada por el siguiente sistema de ecuaciones diferenciales. Es importante resaltar que esta versión del modelo es determinística y asume una población uniformemente distribuida. 

#### Dinámica de SEIRS

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRS_deterministic_equations.png" width="210"></div>
</p>

donde *S*, *E*, *I*, *R*, y *F* son los números de individuos susceptibles, expuestos, infectados, recuperados, y fallecidos, respectivamente, y *N* es el número total de individuos en la población (parámetros que se describen arriba).

#### Dinámica de SEIRS con Pruebas de Enfermedad

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRStesting_deterministic_equations.png" width="400"></div>
</p>

donde *S*, *E*, *I*, *D<sub>E</sub>*, *D<sub>I</sub>*, *R*, y *F* son los números de individuos susceptibles, expuestos, infectados, expuestos detectados, infectados detectados, recuperados, y fallecidos, respectivamente, y *N* es el número total de individuos en la población  (parámetros que se describen arriba).

<a name="model-network"></a>
### Modelo de Red

El modelo SEIRS estándr captura caracterpisticas importantes de la dinámica de las infecciones, pero es determinístico y asume una distribución poblacional uniforme (donde cada individuo es igualmente propenso a interactuar con otro individuo). Sin embargo, es importante considerar los efectos estocásticos y la estructura de la red de contacto cuando se estudia transmisión de enfermedades y el efecto de intervenciones como distanciamiento social y rastreo de contactos.

Este paquete incluye la implementación de la dinámica de SEIRS en redes estocásticas con comportamiento dinámico. 
Esto sirve para analizar la relación entre la estructura de la red y las tasas de transmisión efectivas, incluido el efecto de las intervenciones basadas en la restricción de la red de contagios, como el distanciamiento social, la cuarentena y el rastreo de contactos.

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/network_contacts.png" height="250">

Considere un grafo **_G_** representando individuos (nodos) y sus interacciones (aristas/lados). Cada individuo (nodo) tiene un estado (*S, E, I, D<sub>E</sub>, D<sub>I</sub>, R, or F*). El conjunto de nodos adyacentes (conectado por una arista) a un individuo define el conjunto de "contactos cercanos" (resaltados en negro). En determinado instante, cada individuo hace contacto con un indivduo aleatorio de su red de contactos cercanos con probabilidad *(1-p)β* o con un individuo aleatorio de toda cualquier punto de la red (resaltado en azul) con probabilidad *pβ*. Los últimos contactos globales representan individuos interactuando con la población en general (es decir, individuos fuera del círculo social, como en el transporte público, en un evento, etc.) con cierta probabilidad. Cuando un individuo susceptible interactúa con un individuo infeccioso, queda expuesto. El parámetro *p* define la localización de la red: para *p=0* un individuo solo interactua con sus contactos cercanos, mientras que *p=1* representa una población mezclada uniformemente. Las intervenciones de distanciamiento social podrían incrementar la focalización de la red (reducir *p*) y/o reducir la conectividad local de la red (reducir el grado de los individuos).

Cada nodo *i* tiene un estado *X<sub>i</sub>* que se actualiza de acuerdo a las siguientes probabilidades de transición: 

<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRSnetwork_transitions.png" width="500"></div>
</p>

donde *δ<sub>Xi=A</sub> = 1* si el estado de *X_i* es *A*, o *0* si no lo es, y donde *C<sub>G</sub>(i)* denota el conjunto de contactos cercanos al nodo *i*. Para poblaciones grandes y *p=1*, este modelo estocástico aborda la misma dinámica como el modelo determinístico SEIRS.

Esta implementación se basa en el trabajo de Dottori et al. (2015).
* Dottori, M. and Fabricius, G., 2015. SIR model on a dynamical network and the endemic state of an infectious disease. Physica A: Statistical Mechanics and its Applications, 434, pp.25-35.

<a name="model-network-ttq"></a>
#### Network Model with Testing, Contact Tracing, and Quarantining

##### Testing & Contact Tracing

As with the deterministic model, exposed and infectious individuals are tested at rates *θ<sub>E</sub>* and *θ<sub>I</sub>*, respectively, and test positively for infection with rates *ψ<sub>E</sub>* and *ψ<sub>I</sub>*, respectively (the false positive rate is assumed to be zero, so susceptible individuals never test positive). Testing positive moves an individual into the appropriate detected case state (*D<sub>E</sub>* or *D<sub>I</sub>*), where rates of transmission, progression, recovery, and/or mortality (as well as network connectivity in the network model) may be different than those of undetected cases.

Consideration of interaction networks allows us to model contact tracing, where the close contacts of an positively-tested individual are more likely to be tested in response. In this model, an individual is tested due to contact tracing at a rate equal to *φ* times the number of its close contacts who have tested positively.

##### Quarantining

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/network_contacts_quarantine.png" height="250">

Now we also consider another graph **_Q_** which represents the interactions that each individual has if they test positively for the disease (i.e., individuals in the *D<sub>E</sub>* or *D<sub>I</sub>* states) and enter into a form of quarantine. The quarantine has the effect of dropping some fraction of the edges connecting the quarantined individual to others (according to a rule of the user's choice when generating the graph *Q*). The edges of *Q* (highlighted in purple) for each individual are then a subset of the normal edges of *G* for that individual. The set of nodes that are adjacent to a quarantined individual define their set of "quarantine contacts" (highlighted in purple). At a given time, a quarantined individual may come into contact with another individual in this quarantine contact set with probability *(1-p)β<sub>D</sub>*. A quarantined individual may also be come in contact with a random individual from anywhere in the network with rate *qpβ<sub>D</sub>*.

Each node *i* has a state *X<sub>i</sub>* that updates according to the following probability transition rates: 
<p align="center">
  <img src="https://github.com/ryansmcgee/seirsplus/blob/master/images/SEIRSnetworktesting_transitions.png" width="800"></div>
</p>

where *δ<sub>Xi=A</sub>=1* if the state of *X<sub>i</sub>* is *A*, or *0* if not, and where *C<sub>G</sub>(i)* and *C<sub>Q</sub>(i)* denotes the set of close contacts and quarantine contacts of node *i*, respectively. For large populations and *p=1*, this stochastic model approaches the same dynamics as the deterministic SEIRS model (sans contact tracing, which is not included in the uniformly-mixed model).

<a name="usage"></a>
## Code Usage

This package was designed with broad usability in mind. Complex scenarios can be simulated with very few lines of code or, in many cases, no new coding or knowledge of python by simply modifying the parameter values in the example notebooks provided. See the Quick Start section and the rest of this documentation for more details.

Don't be fooled by the length of the README, running these models is quick and easy. The package does all the hard work for you. As an example, here's a complete script that simulates the SEIRS dyanmics on a network with social distancing, testing, contact tracing, and quarantining in only 10 lines of code (see the [example notebooks](https://github.com/ryansmcgee/seirsplus/tree/master/examples) for more explanation of this example):
```python
from seirsplus.models import *
import networkx

numNodes = 10000
baseGraph    = networkx.barabasi_albert_graph(n=numNodes, m=9)
G_normal     = custom_exponential_graph(baseGraph, scale=100)
# Social distancing interactions:
G_distancing = custom_exponential_graph(baseGraph, scale=10)
# Quarantine interactions:
G_quarantine = custom_exponential_graph(baseGraph, scale=5)

model = SEIRSNetworkModel(G=G_normal, beta=0.155, sigma=1/5.2, gamma=1/12.39, mu_I=0.0004, p=0.5,
                          Q=G_quarantine, beta_D=0.155, sigma_D=1/5.2, gamma_D=1/12.39, mu_D=0.0004,
                          theta_E=0.02, theta_I=0.02, phi_E=0.2, phi_I=0.2, psi_E=1.0, psi_I=1.0, q=0.5,
                          initI=10)

checkpoints = {'t': [20, 100], 'G': [G_distancing, G_normal], 'p': [0.1, 0.5], 'theta_E': [0.02, 0.02], 'theta_I': [0.02, 0.02], 'phi_E':   [0.2, 0.2], 'phi_I':   [0.2, 0.2]}

model.run(T=300, checkpoints=checkpoints)

model.figure_infections()
```

<a name="usage-start"></a>
### Quick Start

The [```examples```](https://github.com/ryansmcgee/seirsplus/tree/master/examples) directory contains two Jupyter notebooks: one for the deterministic model and one for the [network model](https://github.com/ryansmcgee/seirsplus/blob/master/examples/network_model_demo.ipynb). These notebooks walk through full simulations using each of these models with description of the steps involved.

**These notebooks can also serve as ready-to-run sandboxes for trying your own simulation scenarios by simply changing the parameter values in the notebook.**

<a name="usage-install"></a>
### Installing and Importing the Package

All of the code needed to run the model is imported from the ```models``` module of this package.

#### Install the package using ```pip```
The package can be installed on your machine by entering this in the command line:

```> sudo pip install seirsplus```

Then, the ```models``` module can be imported into your scripts as shown here:

```python
from seirsplus.models import *
import networkx
```

#### *Alternatively, manually copy the code to your machine*

*You can use the model code without installing a package by copying the ```models.py``` module file to a directory on your machine. In this case, the easiest way to use the module is to place your scripts in the same directory as the module, and import the module as shown here:*

```python
from models import *
```
<a name="usage-init"></a>
### Initializing the Model

<a name="usage-init-determ"></a>
#### Deterministic Model

All model parameter values, including the normal and (optional) quarantine interaction networks, are set in the call to the ```SEIRSModel``` constructor. The basic SEIR parameters ```beta```, ```sigma```, ```gamma```, and ```initN``` are the only required arguments. All other arguments represent parameters for optional extended model dynamics; these optional parameters take default values that turn off their corresponding dynamics when not provided in the constructor. 

Constructor Argument | Parameter Description | Data Type | Default Value
-----|-----|-----|-----
```beta   ``` | rate of transmission | float | REQUIRED
```sigma  ``` | rate of progression | float | REQUIRED
```gamma  ``` | rate of recovery | float | REQUIRED
```xi     ``` | rate of re-susceptibility | float | 0
```mu_I   ``` | rate of infection-related mortality | float | 0
```mu_0   ``` | rate of baseline mortality | float | 0 
```nu     ``` | rate of baseline birth | float | 0 
```beta_D ``` | rate of transmission for detected cases | float | None (set equal to ```beta```) 
```sigma_D``` | rate of progression for detected cases | float | None (set equal to ```sigma```)  
```gamma_D``` | rate of recovery for detected cases | float | None (set equal to ```gamma```)  
```mu_D   ``` | rate of infection-related mortality for detected cases | float | None (set equal to ```mu_I```) 
```theta_E``` | rate of testing for exposed individuals | float | 0 
```theta_I``` | rate of testing for infectious individuals | float | 0 
```psi_E  ``` | probability of positive tests for exposed individuals | float | 0 
```psi_I  ``` | probability of positive tests for infectious individuals | float | 0
```initN  ``` | initial total number of individuals | int | 10
```initI  ``` | initial number of infectious individuals | int | 10
```initE  ``` | initial number of exposed individuals | int | 0 
```initD_E``` | initial number of detected infectious individuals | int | 0 
```initD_I``` | initial number of detected exposed individuals | int | 0 
```initR  ``` | initial number of recovered individuals | int | 0
```initF  ``` | initial number of deceased individuals | int | 0

##### Basic SEIR

```python
model = SEIRSModel(beta=0.155, sigma=1/5.2, gamma=1/12.39, initN=100000, initI=100)
```


##### Basic SEIRS

```python
model = SEIRSModel(beta=0.155, sigma=1/5.2, gamma=1/12.39, xi=0.001, initN=100000, initI=100)
```

##### SEIR with testing and different progression rates for detected cases (```theta``` and ```psi``` testing params > 0, rate parameters provided for detected states)

```python
model = SEIRSModel(beta=0.155, sigma=1/5.2, gamma=1/12.39, initN=100000, initI=100,
                   beta_D=0.100, sigma_D=1/4.0, gamma_D=1/9.0, theta_E=0.02, theta_I=0.02, psi_E=1.0, psi_I=1.0)
```


<a name="usage-init-network"></a>
#### Network Model

All model parameter values, including the interaction network and (optional) quarantine network, are set in the call to the ```SEIRSNetworkModel``` constructor. The interaction network ```G``` and the basic SEIR parameters ```beta```, ```sigma```, and ```gamma``` are the only required arguments. All other arguments represent parameters for optional extended model dynamics; these optional parameters take default values that turn off their corresponding dynamics when not provided in the constructor. 

**_Heterogeneous populations:_** Nodes can be assigned different values for a given parameter by passing a list of values (with length = number of nodes) for that parameter in the constructor.

All constructor parameters are listed and described below, followed by examples of use cases for various elaborations of the model are shown below (non-exhaustive list of use cases).

Constructor Argument | Parameter Description | Data Type | Default Value
-----|-----|-----|-----
```G      ``` | graph specifying the interaction network | ```networkx Graph``` or ```numpy 2d array```  | REQUIRED 
```beta   ``` | rate of transmission | float | REQUIRED
```sigma  ``` | rate of progression | float | REQUIRED
```gamma  ``` | rate of recovery | float | REQUIRED
```xi     ``` | rate of re-susceptibility | float | 0
```mu_I   ``` | rate of infection-related mortality | float | 0
```mu_0   ``` | rate of baseline mortality | float | 0 
```nu     ``` | rate of baseline birth | float | 0 
```p      ``` | probability of global interactions (network locality) | float | 0
```Q      ``` | graph specifying the quarantine interaction network | ```networkx Graph``` or ```numpy 2d array``` | None 
```beta_D ``` | rate of transmission for detected cases | float | None (set equal to ```beta```) 
```sigma_D``` | rate of progression for detected cases | float | None (set equal to ```sigma```)  
```gamma_D``` | rate of recovery for detected cases | float | None (set equal to ```gamma```)  
```mu_D   ``` | rate of infection-related mortality for detected cases | float | None (set equal to ```mu_I```) 
```theta_E``` | rate of testing for exposed individuals | float | 0 
```theta_I``` | rate of testing for infectious individuals | float | 0 
```phi_E  ``` | rate of contact tracing testing for exposed individuals | float | 0 
```phi_I  ``` | rate of contact tracing testing for infectious individuals | float | 0 
```psi_E  ``` | probability of positive tests for exposed individuals | float | 0 
```psi_I  ``` | probability of positive tests for infectious individuals | float | 0
```q      ``` | probability of global interactions for quarantined individuals | float | 0
```initI  ``` | initial number of infectious individuals | int | 10
```initE  ``` | initial number of exposed individuals | int | 0 
```initD_E``` | initial number of detected infectious individuals | int | 0 
```initD_I``` | initial number of detected exposed individuals | int | 0 
```initR  ``` | initial number of recovered individuals | int | 0
```initF  ``` | initial number of deceased individuals | int | 0

##### Basic SEIR on a network

```python
model = SEIRSNetworkModel(G=myGraph, beta=0.155, sigma=1/5.2, gamma=1/12.39, initI=100)
```


##### Basic SEIRS on a network

```python
model = SEIRSNetworkModel(G=myGraph, beta=0.155, sigma=1/5.2, gamma=1/12.39, xi=0.001, initI=100)
```


##### SEIR on a network with global interactions (p>0)

```python
model = SEIRSNetworkModel(G=myGraph, beta=0.155, sigma=1/5.2, gamma=1/12.39, p=0.5, initI=100)
```

##### SEIR on a network with testing and quarantining (```theta``` and ```psi``` testing params > 0, quarantine network ```Q``` provided)

```python
model = SEIRSNetworkModel(G=myNetwork, beta=0.155, sigma=1/5.2, gamma=1/12.39, p=0.5,
                          Q=quarantineNetwork, q=0.5,
                          theta_E=0.02, theta_I=0.02, psi_E=1.0, psi_I=1.0, 
                          initI=100)
```

##### SEIR on a network with testing, quarantining, and contact tracing (```theta``` and ```psi``` testing params > 0, quarantine network ```Q``` provided, ```phi``` contact tracing params > 0)

```python
model = SEIRSNetworkModel(G=myNetwork, beta=0.155, sigma=1/5.2, gamma=1/12.39, p=0.5,
                          Q=quarantineNetwork, q=0.5,
                          theta_E=0.02, theta_I=0.02, phi_E=0.2, phi_I=0.2, psi_E=1.0, psi_I=1.0,  
                          initI=100)
```

<a name="usage-run"></a>
### Running the Model

Stochastic network SEIRS dynamics are simulated using the Gillepsie algorithm.

Once a model is initialized, a simulation can be run with a call to the following function:

```python
model.run(T=300)
```

The ```run()``` function has the following arguments

Argument | Description | Data Type | Default Value
-----|-----|-----|-----
```T``` | runtime of simulation | numeric | REQUIRED
```checkpoints``` | dictionary of checkpoint lists (see section below) | dictionary | ```None```
```print_interval``` | (network model only) time interval to print sim status to console | numeric | 10
```verbose``` | if ```True```, print count in each state at print intervals, else just the time | bool | ```False```

<a name="usage-run"></a>
### Accessing Simulation Data

Model parameter values and the variable time series generated by the simulation are stored in the attributes of the ```SEIRSModel``` or ```SEIRSNetworkModel``` being used and can be accessed directly as follows:

```python
S = model.numS      # time series of S counts
E = model.numE      # time series of E counts
I = model.numI      # time series of I counts
D_E = model.numD_E    # time series of D_E counts
D_I = model.numD_I    # time series of D_I counts
R = model.numR      # time series of R counts
F = model.numF      # time series of F counts

t = model.tseries   # time values corresponding to the above time series

G_normal     = model.G    # interaction network graph
G_quarantine = model.Q    # quarantine interaction network graph

beta = model.beta   # value of beta parameter (or list of beta values for each node if using network model)
# similar for other parameters
```
*Note: convenience methods for plotting these time series are included in the package. See below.*

<a name="usage-networks"></a>
### Specifying Interaction Networks

This model includes a model of SEIRS dynamics for populations with a structured interaction network (as opposed to standard deterministic SIR/SEIR/SEIRS models, which assume uniform mixing of the population). When using the network model, a graph specifying the interaction network for the population must be specified, where each node represents an individual in the population and edges connect individuals who have regular interactions.

The interaction network can be specified by a **```networkx``` ```Graph```** object or a **```numpy``` 2d array** representing the adjacency matrix, either of which can be defined and generated by any method.

This SEIRS+ model also implements dynamics corresponding to testing individuals for the disease and moving individuals with detected infections into a state where their rate of recovery, mortality, etc may be different. In addition, given that this model considers individuals in an interaction network, a separate graph defining the interactions for individuals with detected cases can be specified (i.e., the "quarantine interaction" network).

Epidemic scenarios of interest often involve interaction networks that change in time. Multiple interaction networks can be defined and used at different times in the model simulation using the checkpoints feature (described in the section below).

**_Note:_** *Simulation time increases with network size. Small networks simulate quickly, but have more stochastic volatility. Networks with ~10,000 are large enough to produce per-capita population dynamics that are generally consistent with those of larger networks, but small enough to simulate quickly. We recommend using networks with ~10,000 nodes for prototyping parameters and scenarios, which can then be run on larger networks if more precision is required.*

#### Custom Exponential Graph

Human interaction networks often resemble scale-free power law networks with exponential degree distributions.
This package includes a ```custom_exponential_graph()``` convenience funciton that generates power-law-like graphs that have degree distributions with two exponential tails. The method of generating these graphs also makes it easy to remove edges from a reference graph and decrease the degree of the network, which is useful for generating networks representing social distancing and quarantine conditions.

Common algorithms for generating power-law graphs, such as the Barabasi-Albert preferential attachment algorithm, produce graphs that have a minimum degree; that is, no node has fewer than *m* edges for some value of *m*, which is unrealistic for interaction networks. This ```custom_exponential_graph()``` function simply produces graphs with degree distributions that have a peak near their mean and exponential tails in the direction of both high and low degrees. (No claims about the realism or rigor of these graphs are made.)

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/degreeDistn_compareToBAGraph1.png" height="250">

This function generates graphs using the following algorithm:
* Start with a Barabasi-Albert preferential attachment power-law graph (or any graph that is optionally provided by the user).
* For each node:
    * Count the number of neighbors *n* of the node 
    * Draw a random number *r* from an exponential distribution with some mean=```scale```. If *r>n*, set *r=n*. 
    * Randomly select *r* of this node’s neighbors to keep, delete the edges to all other neighbors. 
When starting from a Barabasi-Albert (BA) graph, this generates a new graphs that have a peak at their mean and approximately exponential tails in both directions, as shown to the right.

<img align="right" src="https://github.com/ryansmcgee/seirsplus/blob/master/images/degreeDistn_compareToBAGraph4.png" height="250">

Since this algorithm starts with a graph with defined connections and makes a new graph by breaking some number of connections, it also makes it easy to take an existing graph and make a subgraph of it that also has exponential-ish tails and a left-shifted mean. This can be used for generating social distancing and quarantine subgraphs. The amount of edge breaking/degree reduction is modulated by the ```scale``` parameter. To the right are some examples of graphs with progressively lower mean degree generated using the same reference Barabasi-Albert graph, which therefore are all subsets of a common reference set of edges.

The ```custom_exponential_graph()``` function has the following arguments

base_graph=None, scale=100, min_num_edges=0, m=9, n=None
Argument | Description | Data Type | Default Value
-----|-----|-----|-----
```base_graph``` | Graph to use as the starting point for the algorithm. If ```None```, generate a Barabasi-Albert graph to use as the starting point using arguments ```n``` and ```m``` as parameters | ```networkx``` ```Graph``` object | ```None```
```scale``` | Mean of the exponential distribution used to draw ```base_graph``` to keep. Large values result in graphs that approximate the original ```base_graph```, small values result in sparser subgraphs of ```base_graph```  | numeric | 100
```min_num_edges``` | Minimum number of edges that all nodes must have in the generated graph | int | 0
```n``` | *n* parameter for teh Barabasi-Albert algorithm (number of nodes to add) | int | ```None``` (value required when no ```base_graph``` is given)
```m``` | *m* parameter for the Barabasi-Albert algorithm (number of edges added with each added node) | int | 9

<a name="usage-checkpoints"></a>
### Changing parameters during a simulation

Model parameters can be easily changed during a simulation run using checkpoints. A dictionary holds a list of checkpoint times (```checkpoints['t']```) and lists of new values to assign to various model parameters at each checkpoint time. 

Example of running a simulation with ```checkpoints```:
```python
checkpoints = {'t':       [20, 100], 
               'G':       [G_distancing, G_normal], 
               'p':       [0.1, 0.5], 
               'theta_E': [0.02, 0.02], 
               'theta_I': [0.02, 0.02], 
               'phi_E':   [0.2, 0.2], 
               'phi_I':   [0.2, 0.2]}
```

*The checkpoints shown here correspond to starting social distancing and testing at time ```t=20``` (the graph ```G``` is updated to ```G_distancing``` and locality parameter ```p``` is decreased to ```0.1```; testing params ```theta_E```, ```theta_I```, ```phi```, and ```phi_I``` are set to non-zero values) and then stopping social distancing at time ```t=100``` (```G``` and ```p``` changed back to their "normal" values; testing params remain non-zero).*

Any model parameter listed in the model constrcutor can be updated in this way. Only model parameters that are included in the checkpoints dictionary have their values updated at the checkpoint times, all other parameters keep their pre-existing values.

Use cases of this feature include: 

* Changing parameters during a simulation, such as changing transition rates or testing parameters every day, week, on a specific sequence of dates, etc.
* Starting and stopping interventions, such as social distancing (changing interaction network), testing and contact tracing (setting relevant parameters to non-zero or zero values), etc.

**_Consecutive runs_**: *You can also run the same model object multiple times. Each time the ```run()``` function of a given model object is called, it starts a simulation from the state it left off in any previous simulations. 
For example:*
```python
model.run(T=100)    # simulate the model for 100 time units
# ... 
# do other things, such as processing simulation data or changing parameters 
# ...
model.run(T=200)    # simulate the model for an additional 200 time units, picking up where the first sim left off
```

<a name="usage-viz"></a>
### Visualization

### Visualizing the results
The ```SEIRSModel``` and ```SEIRSNetworkModel``` classes have a ```plot()``` convenience function for plotting simulation results on a matplotlib axis. This function generates a line plot of the frequency of each model state in the population by default, but there are many optional arguments that can be used to customize the plot.

These classes also have convenience functions for generating a full figure out of model simulation results (optionally, arguments can be provided to customize the plots generated by these functions, see below). 

- ```figure_basic()``` calls the ```plot()``` function with default parameters to generate a line plot of the frequency of each state in the population.
- ```figure_infections()``` calls the ```plot()``` function with default parameters to generate a stacked area plot of the frequency of only the infection states (*E*, *I*, *D<sub>E</sub>*, *D<sub>I</sub>*) in the population.

Parameters that can be passed to any of the above functions include:
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
