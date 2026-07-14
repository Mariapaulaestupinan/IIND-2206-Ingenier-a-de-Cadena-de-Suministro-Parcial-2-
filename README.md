<div align="center">

<h1>IIND 2206 · Ingeniería de Cadena de Suministro</h1>

</div>

## Descripción

Este repositorio tiene como propósito aterrizar la teoría vista en clase mediante el desarrollo de ejemplos y casos de estudio basados en problemas reales de logística y optimización.

Se abordarán temas relacionados con **ruta más corta** y **ruteo de vehículos**, utilizando librerías especializadas de Python para construir, solucionar y analizar distintos escenarios desde un enfoque aplicado.

---

## Contenido del repositorio
- [Ruta más corta](#ruta-más-corta) 
- [Ruteo de vehículos](#ruteo-de-vehículos) 

---

## Ruta más corta

En esta sección aprenderemos a modelar y resolver problemas de ruta más corta utilizando la librería **NetworkX**. Adicionalmente, se presentará el pseudocódigo del algoritmo de Dijkstra con el fin de comprender la lógica y estructura subyacente de este tipo de algoritmos aplicados sobre redes.

A lo largo de los ejercicios, nos enfrentaremos con problemas en los que no solo se busca encontrar la ruta de menor distancia, sino también **minimizar tiempos de recorrido**, **costos de transporte** o cualquier métrica de eficiencia definida sobre la red. Se introducirán restricciones operativas que condicionan la disponibilidad de ciertas vías y generan sobrecostos al momento de transitar por determinados arcos. 

### Contenido

- [NetworkX](#networkx)
  - [Funciones principales NetworkX](#funciones-principales-networkx)
- [Dijkstra](#dijkstra)
  - [Pseudocodigo del algoritmo de Dijkstra](#pseudocodigo-del-algoritmo-de-dijkstra)
---

### NetworkX

La librería NetworkX resulta especialmente útil en problemas donde la estructura de la red se encuentra completamente definida desde el inicio y no cambia durante la ejecución del algoritmo. En este tipo de ejercicios, los nodos, arcos y pesos asociados permanecen estáticos, por lo que el proceso consiste principalmente en construir el grafo a partir de la información del problema y posteriormente calcular la ruta más corta.

Bajo este enfoque, las restricciones del problema se incorporan previamente mediante la construcción de la red. Es decir, antes de resolver el modelo se eliminan los arcos que no cumplen las condiciones operativas y se ajustan los costos, tiempos o distancias de acuerdo a las condiciones del problema. Una vez construida la red, el algoritmo opera sobre un grafo fijo que no se modifica dinámicamente mientras se recorre la solución.

A continuación, se estudiarán las principales funciones de la librería NetworkX. Además, se presentarán algunos ejercicios orientados a comprender su implementación en problemas de ruta más corta.

---

### Funciones principales NetworkX

#### Importación de la librería

```python
import networkx as nx
```

#### Inicialización del grafo 
A lo largo del curso se trabajará exclusivamente con **grafos dirigidos**.

```python
G = nx.DiGraph()
```
#### Construcción del grafo

<details>
<summary> Agregar nodos al grafo</summary>

---

**- Agregar un solo nodo**

```python
G = nx.DiGraph()
G.add_node("A")
```

**- Agregar una lista de nodos**

```python
G = nx.DiGraph()
nodos = ["A", "B", "C", "D"]
G.add_nodes_from(nodos)
```
**- Agregar nodos desde un archivo Excel**

Suponga que tiene un archivo Excel con una columna llamada `"Nodos"`, donde cada fila representa un nodo distinto.

```python
import pandas as pd

G = nx.DiGraph()
df = pd.read_excel("Nodos.xlsx")
G.add_nodes_from(df["Nodos"])
```
</details>
<details>
<summary> Agregar arcos al grafo</summary>

---

El parámetro `weight` representa el peso del arco, el cual puede corresponder a una distancia, un tiempo de recorrido o un costo de transporte, según el criterio de optimización definido en el problema.

**- Agregar un solo arco**

```python
G = nx.DiGraph()
G.add_edge("A", "B", weight=5)
```

**- Agregar una lista de arcos**

Cuando cada arco tiene un único atributo de peso:

```python
G = nx.DiGraph()
arcos = [("A", "B", 5), ("B", "C", 3), ("C", "D", 7)]
G.add_weighted_edges_from(arcos)
```

Cuando cada arco tiene múltiples atributos, los atributos de cada arco deben estar organizados en un diccionario, de lo contrario `add_edges_from` no los reconocerá correctamente:

```python
G = nx.DiGraph()
arcos = [
    ("A", "B", {"weight": 5,  "tipo": "restringido", "disponible": True}),
    ("B", "C", {"weight": 3,  "tipo": "normal",      "disponible": True}),
    ("C", "D", {"weight": 7,  "tipo": "restringido", "disponible": False}),
]
G.add_edges_from(arcos)
```

**▸ Agregar arcos desde un archivo Excel**

Suponga que tiene un archivo Excel con columnas "Origen", "Destino" y "Peso", donde cada fila representa un arco de la red y hay un unico atributo:

```python
import pandas as pd

G = nx.DiGraph()
df = pd.read_excel("Arcos.xlsx")

for _, row in df.iterrows():
    G.add_edge(row["Origen"], row["Destino"], weight=row["Peso"])
```

Suponga que tiene un archivo Excel con columnas `"Origen"`, `"Destino"`, `"Peso"`, `"Tipo"` y `"Disponible"`, donde cada fila representa un arco de la red y hay varios atributos:

```python
import pandas as pd

G = nx.DiGraph()
df = pd.read_excel("Arcos.xlsx")

for _, row in df.iterrows():
    G.add_edge(row["Origen"], row["Destino"],
               weight     = row["Peso"],
               tipo       = row["Tipo"],
               disponible = row["Disponible"])
```

</details>
<details>
<summary> Eliminar arcos al grafo</summary>

---

A diferencia de la construcción del grafo, la eliminación de arcos permite depurar la red en función de restricciones operativas y condiciones del problema.

**- Eliminar los arcos que salen de un nodo**

```python
nodo = "A"
arcos_salida = list(G.out_edges(nodo))
G.remove_edges_from(arcos_salida)
```

**- Eliminar los arcos que llegan a un nodo**

```python
nodo = "A"
arcos_entrada = list(G.in_edges(nodo))
G.remove_edges_from(arcos_entrada)
```

**- Eliminar los arcos con un atributo específico**

Elimina aquellos arcos cuyos atributos no cumplen con las condiciones operativas del problema. Si el grafo fue construido con un único atributo `weight`, se puede filtrar directamente sobre él.

```python
# Arcos con un solo atributo
arcos_a_eliminar = []
for u, v, d in G.edges(data=True): # data=True retorna los atributos del arco como diccionario
    if d["weight"] > 10:
        arcos_a_eliminar.append((u, v))

G.remove_edges_from(arcos_a_eliminar)
```

Cuando los arcos tienen múltiples atributos, es posible combinar condiciones sobre cualquiera de ellos para definir un criterio de eliminación.

```python
# Construcción del grafo con múltiples atributos
G = nx.DiGraph()
arcos = [
    ("A", "B", {"weight": 5,  "tipo": "restringido", "disponible": True}),
    ("B", "C", {"weight": 12, "tipo": "normal",      "disponible": False}),
    ("C", "D", {"weight": 8,  "tipo": "restringido", "disponible": False}),
]
G.add_edges_from(arcos)

# Eliminar arcos que sean de tipo restringido y no estén disponibles
arcos_a_eliminar = []
for u, v, d in G.edges(data=True):
    if d["tipo"] == "restringido":
        if d["disponible"] == False:
            arcos_a_eliminar.append((u, v))

G.remove_edges_from(arcos_a_eliminar)
```

**- Eliminar los arcos desde un archivo Excel**

Suponga que tiene un archivo Excel con las columnas `"Origen"` y `"Destino"`, donde cada fila representa un arco que debe ser eliminado de la red.

```python
import pandas as pd

df = pd.read_excel("Arcos_eliminar.xlsx")

arcos_a_eliminar = []
for _, row in df.iterrows():
    arcos_a_eliminar.append((row["Origen"], row["Destino"]))

G.remove_edges_from(arcos_a_eliminar)
```

</details>
<details>
<summary> Modificar atributos de los arcos</summary>

---

En ciertos problemas es necesario ajustar los atributos de los arcos sin eliminarlos de la red. Esto resulta útil cuando se desea incorporar penalizaciones o sobrecostos asociados a condiciones operativas específicas, como el tipo de vía, el estado de la carretera o restricciones horarias.

**- Modificar el atributo de un arco según un criterio**

Suponga que los arcos tienen los atributos `"Costo"` y `"Tipo_Via"`. Si la vía es de tipo terrestre, se desea aumentar el costo en un 20%:

```python
for u, v, d in G.edges(data=True):  # data=True retorna los atributos del arco como diccionario
    if d["Tipo_Via"] == "terrestre":
        d["Costo"] = d["Costo"] * 1.20
```
</details>

#### Solución y resultados

---

Una vez construida la red, calculamos la ruta más corta entre dos nodos. A partir de la solución obtenida es posible consultar sobre dicha ruta cualquier atributo de interés, como el costo acumulado, la distancia total, el tiempo de recorrido o el número de arcos que cumplen con alguna condición particular.

<details>
<summary>Calcular la ruta más corta </summary>
  
Retorna la secuencia de nodos que conforman el camino óptimo entre el nodo de origen y el nodo de destino, minimizando el atributo especificado en `weight`.

```python
ruta = nx.dijkstra_path(G, source="A", target="D", weight="weight")
print(ruta)  # ["A", "C", "B", "D"]
```
</details>
<details>
<summary> Obtener el valor óptimo de la función objetivo </summary>

Retorna el valor de la función objetivo de la ruta óptima segun el atributo especificado en `weight`. 

```python
valor_funcion_objetivo = nx.dijkstra_path_length(G, source="A", target="D", weight="weight")
print(f"${valor_funcion_objetivo}") # $15
```
</details>
<details>
<summary> Obtener la ruta y el valor de la función objetivo en una sola llamada </summary>

```python
valor_funcion_objetivo, ruta = nx.single_source_dijkstra(G, source="A", target="D", weight="weight")
print(ruta)        # ["A", "B", "C", "D"]
print(f"${valor_funcion_objetivo}") # $15
```
</details>
<details>
<summary> Extraer información adicional de la ruta óptima </summary>

En ocasiones el criterio de optimización no es el único atributo de interés. Una vez obtenida la ruta óptima, es posible calcular el valor acumulado de cualquier otro atributo a lo largo de ella.

Suponga que el problema minimiza el costo pero adicionalmente se desea conocer la distancia total recorrida a lo largo de esa misma ruta. En ese caso se puede calcular de la siguiente manera:

```python
# Calcula simultáneamente el costo mínimo y la ruta óptima
costo_total, ruta = nx.single_source_dijkstra(G, source="A", target="D", weight="costo")

# Acumular la distancia total a lo largo de los arcos de la ruta óptima
distancia_total = 0
for i in range(len(ruta) - 1):
    u = ruta[i]       # Nodo de origen del arco
    v = ruta[i + 1]   # Nodo de destino del arco
    distancia_total += G[u][v]["distancia"]  # Suma la distancia del arco

print(f"Ruta:            {ruta}")
print(f"Costo total:     ${costo_total:,.2f}")
print(f"Distancia total: {distancia_total:,.2f} km")
```
</details>
<details>
<summary> Identificar los arcos que cumplen una condición especifica sobre la ruta óptima </summary>

Una vez obtenida la ruta óptima, es posible recorrer los arcos que la conforman e identificar cuáles de ellos satisfacen alguna condición de interés. Por ejemplo, identificar qué tramos de la ruta corresponden a vías terrestres, a qué zona pertenece cada arco o cuáles superan un número determinado de peajes.

```python
# Obtener la ruta óptima
ruta = nx.dijkstra_path(G, source="A", target="D", weight="costo")

# Identificar los arcos de la ruta óptima que son de tipo terrestre 
arcos_terrestres = []
for i in range(len(ruta) - 1):
    u = ruta[i]      # Nodo de origen del arco
    v = ruta[i + 1]  # Nodo de destino del arco
    if G[u][v]["tipo"] == "terrestre":  # Condición sobre el atributo del arco
        arcos_terrestres.append((u, v))

print(f"Ruta:             {ruta}")           # Ruta:             ["A", "B", "C", "D"]
print(f"Arcos terrestres: {arcos_terrestres}") # Arcos terrestres: [("A", "B"), ("C", "D")]
```
</details>
<details>
<summary> Listar todos los caminos posibles entre un origen y un destino </summary>

Cuando el análisis requiere explorar la totalidad de caminos existentes entre dos nodos y no únicamente el óptimo, es posible obtener una lista exhaustiva de todas las rutas disponibles en la red.

```python
todos_los_caminos = list(nx.all_simple_paths(G, source="A", target="D"))

for camino in todos_los_caminos:
    print(camino)
```

Para cada camino encontrado es posible calcular el valor acumulado de cualquier atributo de interés, como el costo total:

```python
todos_los_caminos = list(nx.all_simple_paths(G, source="A", target="D"))

for camino in todos_los_caminos:
    costo = 0
    for i in range(len(camino) - 1):
        u = camino[i]
        v = camino[i + 1]
        costo += G[u][v]["weight"]
    print(f"Camino: {camino} → Costo: {costo}")
```
</details>


#### Visualización del grafo

---

Una vez construida la red y obtenida la solución, es posible visualizar el grafo junto con la ruta óptima para facilitar la interpretación de los resultados.
<details>
<summary> Visualización a partir de datos ingresados manualmente </summary>

```python
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np

# Construcción del grafo
G = nx.DiGraph()
arcos = [
    ("A", "B", {"weight": 4,  "tipo": "terrestre"}),
    ("A", "C", {"weight": 2,  "tipo": "aéreo"}),
    ("B", "C", {"weight": 1,  "tipo": "terrestre"}),
    ("B", "D", {"weight": 5,  "tipo": "aéreo"}),
    ("C", "D", {"weight": 8,  "tipo": "terrestre"}),
    ("C", "E", {"weight": 3,  "tipo": "aéreo"}),
    ("D", "E", {"weight": 2,  "tipo": "terrestre"}),
]
G.add_edges_from(arcos)

# Obtener la ruta óptima
ruta = nx.dijkstra_path(G, source="A", target="E", weight="weight")

# Identificar los arcos de la ruta óptima
arcos_ruta = [(ruta[i], ruta[i + 1]) for i in range(len(ruta) - 1)]

# Definir posición de los nodos
pos = nx.spring_layout(G)

# Dibujar el grafo completo
nx.draw_networkx_nodes(G, pos, node_color="lightblue", node_size=800)
nx.draw_networkx_labels(G, pos, font_size=12, font_weight="bold")
nx.draw_networkx_edges(G, pos, edge_color="gray", arrows=True, arrowsize=20)

# Resaltar la ruta óptima
nx.draw_networkx_edges(G, pos, edgelist=arcos_ruta, edge_color="red", width=2.5, arrows=True, arrowsize=20)

# Etiqueta superior: distancia en km (desplazada hacia arriba)
etiquetas_arriba = {(u, v): f"{d['weight']} km" for u, v, d in G.edges(data=True)}
nx.draw_networkx_edge_labels(G, pos, edge_labels=etiquetas_arriba, font_size=9,
                              label_pos=0.5, bbox=dict(alpha=0), verticalalignment="bottom")

# Etiqueta inferior: tipo de vía (desplazada hacia abajo)
etiquetas_abajo = {(u, v): d["tipo"] for u, v, d in G.edges(data=True)}
nx.draw_networkx_edge_labels(G, pos, edge_labels=etiquetas_abajo, font_size=9,
                              label_pos=0.5, bbox=dict(alpha=0), verticalalignment="top")

plt.title(f"Ruta óptima: {' → '.join(ruta)}")
plt.axis("off")
plt.tight_layout()
plt.show()
```
<div align="center">
  <img src="grafo 1.png" width="600"/>
</div>
</details>
<details>
<summary> Visualizar el grafo y la ruta óptima con datos desde un archivo Excel </summary>
El siguiente archivo contiene la estructura de red utilizada en este ejemplo. Cada fila representa un arco de la red, definido por su nodo de origen, su nodo de destino y el costo asociado al trayecto. <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/Arcos.xlsx" download>Descargar Arcos.xlsx</a>

```python
import networkx as nx
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd

# Construcción del grafo desde Excel
G = nx.DiGraph()
df = pd.read_excel("Arcos.xlsx")

for _, row in df.iterrows():
    G.add_edge(row["Origen"], row["Destino"], weight=row["Costo"])

# Obtener la ruta óptima
ruta = nx.dijkstra_path(G, source="A", target="K", weight="weight")

# Identificar los arcos de la ruta óptima
arcos_ruta = [(ruta[i], ruta[i + 1]) for i in range(len(ruta) - 1)]

# Definir posición de los nodos
pos = nx.spring_layout(G, seed=42)

# Dibujar el grafo completo
nx.draw_networkx_nodes(G, pos, node_color="lightblue", node_size=800)
nx.draw_networkx_labels(G, pos, font_size=12, font_weight="bold")
nx.draw_networkx_edges(G, pos, edge_color="gray", arrows=True, arrowsize=20)

# Resaltar la ruta óptima
nx.draw_networkx_edges(G, pos, edgelist=arcos_ruta, edge_color="red", width=2.5, arrows=True, arrowsize=20)

# Etiqueta: costo en km (desplazada hacia arriba)
etiqueta = {(u, v): f"{d['weight']} km" for u, v, d in G.edges(data=True)}
nx.draw_networkx_edge_labels(G, pos, edge_labels=etiqueta, font_size=9,
                              label_pos=0.5, bbox=dict(alpha=0), verticalalignment="bottom")

plt.title(f"Ruta óptima: {' → '.join(ruta)}")
plt.axis("off")
plt.tight_layout()
plt.show()
```
<div align="center">
  <img src="grafo 2.png" width="600"/>
</div>
</details>

  
### Dijkstra

---

Las librerías como NetworkX proporcionan implementaciones eficientes del algoritmo de Dijkstra, las cuales son especialmente útiles cuando se trabaja con grafos estáticos y completamente definidos. No obstante, existen situaciones en las que estas herramientas resultan insuficientes, particularmente cuando el grafo varía dinámicamente durante la ejecución del algoritmo.

Este tipo de escenarios ocurre, por ejemplo, cuando la disponibilidad de una vía depende de horarios específicos, lo que implica que la estructura del grafo cambia según el momento en que el nodo es alcanzado. De igual forma, puede suceder que el costo de un arco no sea fijo sino que varíe según el modo de transporte utilizado en el tramo anterior, como ocurre cuando se aplica un recargo adicional al realizar un transbordo entre modo ferroviario y terrestre, haciendo que el peso del arco dependa de cómo se llegó al nodo y no únicamente de sus atributos propios. Una situación similar se presenta cuando cada nodo intermedio de la red tiene un límite máximo de riesgo permitido, de modo que la viabilidad de continuar por un arco solo puede verificarse conociendo el camino exacto tomado hasta ese momento. En todos estos casos la lógica de exploración debe adaptarse en cada paso del algoritmo, lo que hace necesario contar con una implementación propia.

En todos estos contextos, la exploración del grafo debe adaptarse dinámicamente en cada iteración del algoritmo, lo que requiere una implementación personalizada en lugar de depender de soluciones predefinidas.

A continuación, se presenta el pseudocodigo paso a paso del algoritmo de Dijkstra para ejercicios con grafos dinamicos.

#### Pseudocodigo del algoritmo de Dijkstra

---

El algoritmo inicia con la configuración de las estructuras necesarias para realizar la búsqueda de la ruta más corta. A cada nodo del grafo se le asigna una distancia inicial infinita, representando que aún no existe un camino conocido hacia ellos. La única excepción es el nodo de origen, cuya distancia se establece en cero debido a que es el punto de partida.

Adicionalmente, se crea un registro de predecesores para cada nodo, el cual permitirá reconstruir la ruta óptima una vez finalice el algoritmo. También se define un conjunto de nodos visitados, inicialmente vacío, encargado de almacenar aquellos nodos cuyo costo mínimo ya fue determinado.

Finalmente, se inicializa una cola de prioridad, que constituye la estructura principal del algoritmo, ya que permite seleccionar en cada iteración el nodo con menor distancia acumulada y explorar sus vecinos de manera eficiente. Esta cola comienza únicamente con el nodo origen y una distancia asociada de cero.

```text
1: for each node n ∈ V do
2:     distance[n] ← ∞
3:     predecessor[n] ← None
4: end for

5: distance[origin] ← 0
6: visited ← ∅
7: priority_queue ← {(0, origin)}
```
En cada iteración, se extrae de la cola de prioridad el nodo con menor distancia acumulada. Si dicho nodo ya fue procesado anteriormente, se descarta y se continúa con la siguiente iteración. En caso contrario, se marca como visitado y se procede a explorar sus vecinos. Este procedimiento garantiza que cada nodo sea evaluado una única vez y siguiendo el orden de menor costo acumulado.

```text
8: while priority_queue is not empty do
9:     (current_dist, u) ← extract-min(priority_queue)

10:    if u ∈ visited then
11:        continue
12:    end if

13:    visited ← visited ∪ {u}
```

Para cada vecino del nodo actual se calcula la distancia tentativa como la suma de la distancia acumulada hasta el nodo actual y el peso del arco que conecta ambos nodos. Si esta nueva distancia es menor que la distancia registrada para ese vecino, se actualiza el valor mínimo encontrado, se registra el nodo actual como su predecesor y se incorpora nuevamente a la cola de prioridad con la nueva distancia. De esta forma el algoritmo siempre explora primero los caminos más prometedores.

```text
14:    for each neighbor v of u do
15:        tentative_dist ← distance[u] + weight(u, v)

16:        if tentative_dist < distance[v] then
17:            distance[v] ← tentative_dist
18:            predecessor[v] ← u
19:            insert (tentative_dist, v) into priority_queue
20:        end if
21:    end for
22: end while
```

Una vez finaliza la ejecución del algoritmo, la ruta óptima se reconstruye utilizando el registro de predecesores almacenado durante el proceso de búsqueda. Para ello, se inicia desde el nodo destino y se retrocede siguiendo la cadena de predecesores hasta llegar al nodo origen. Debido a que este recorrido se realiza en sentido inverso, la secuencia obtenida se organiza nuevamente para mostrar la ruta en el orden correcto desde el origen hasta el destino.

```text
23: path ← empty list
24: current_node ← destination

25: while current_node is not None do
26:     insert current_node at beginning of path
27:     current_node ← predecessor[current_node]
28: end while

29: return path, distance[destination]
```
## Ruteo de vehículos

En esta sección se abordarán problemas de ruteo de vehículos utilizando la librería **PyVRP**.

A diferencia de los problemas de ruta más corta, donde el objetivo es encontrar el camino óptimo entre dos nodos de una red, los problemas de ruteo de vehículos buscan determinar el conjunto de rutas que debe seguir una flota de vehículos para atender a un grupo de clientes.

El objetivo principal es minimizar una función de costo, que puede estar asociada a la distancia recorrida, el tiempo de operación, el costo total de transporte o una combinación de estos criterios. Para ello, se consideran restricciones operativas, como la capacidad máxima de los vehículos, las ventanas de tiempo de los clientes, la cantidad de vehículos disponibles y la compatibilidad de los vehículos con los clientes.

### Contenido

- [PyVRP](#pyvrp)
  - [Funciones principales PyVRP](#funciones-principales-pyvrp)
  - [Ejercicios PyVRP](#ejercicios-pyvrp)

### PyVRP

---

**PyVRP** es una librería de optimización de código abierto especializada en la resolución de problemas de ruteo de vehículos (*Vehicle Routing Problems, VRP*).

Internamente implementa el algoritmo de **búsqueda local iterada (ILS, por sus siglas en inglés)**, el cual parte de una solución inicial y la mejora de manera progresiva mediante un proceso iterativo. En cada iteración combina dos mecanismos: las perturbaciones, que modifican la solución actual para explorar nuevas regiones del espacio de búsqueda y evitar quedar atrapado en óptimos locales; y la busqueda local, que refinan las soluciones generadas para lograr mejoras adicionales. 

Entre las principales funcionalidades de `PyVRP` se encuentran:

- Capacidad de los vehículos.
- Ventanas de tiempo para atención de clientes.
- Flotas homogéneas y heterogéneas.
- Múltiples depósitos.
- Visitas opcionales a clientes.
- Compatibilidad de vehículos
- Distancia máxima de recorrido

Estas características permiten modelar problemas reales de distribución y logística, donde las decisiones de ruteo deben considerar múltiples restricciones operativas y criterios de eficiencia.

#### Funciones principales PyVRP
 
En esta sección se presentan las funciones y clases principales de PyVRP necesarias para modelar y resolver problemas de ruteo de vehículos. Se cubrirá la instalación de la librería, la importación de los módulos necesarios, la creación del modelo y la definición de todos los atributos.
 
---

#### Instalación 
 
PyVRP puede instalarse directamente desde el administrador de paquetes de Python. Basta con ejecutar el siguiente comando en la terminal:
 
```bash
pip install pyvrp
```

#### Importación de librerías
 
Una vez instalada la librería, se importan los módulos necesarios. `Model` es la clase principal con la que se construye el problema.
 
```python
from pyvrp import Model
```
#### Creación del modelo 
 
`Model` es la clase central de PyVRP. Es el objeto sobre el que se construye el problema completo: se le agregan clientes, depósitos, vehículos y arcos. Todo el problema queda contenido en una única instancia de esta clase antes de ser enviado al solver.
 
```python
m = Model()
```
</details> 

### Clientes 
 
Un cliente representa cada punto de la red que debe ser visitado por un vehículo. Se agrega al modelo usando el método `add_client()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `x` | float | Si | Coordenada x de la ubicación del cliente |
| `y` | float | Si | Coordenada y de la ubicación del cliente |
| `delivery` | int | No | Unidades que el cliente solicita del depósito. Por defecto 0 |
| `pickup` | int | No | Unidades que el cliente devuelve al depósito. Por defecto 0 |
| `service_duration` | int | No | Tiempo que tarda el vehículo en atender al cliente antes de continuar con el recorrido. Por defecto 0 |
| `tw_early` | int | No | Inicio de la ventana de atención del cliente. Por defecto 0 |
| `tw_late` | int | No |Tiempo máximo permitido para iniciar el servicio al cliente. Sin restricción si no se especifica |
| `release_time` | int | No | Momento más temprano en el que el vehículo puede partir del depósito hacia este cliente. Por defecto 0 |
| `prize` | int | No | Ingreso que se obtiene al visitar este cliente. Relevante cuando la visita es opcional. Por defecto 0 |
| `required` | bool | No | Indica si el cliente debe ser visitado obligatoriamente. Por defecto `True` |
| `name` | str | No | Nombre descriptivo del cliente. Por defecto cadena vacía |
 
> **Nota:** Los parámetros de tiempo (`service_duration`, `tw_early`, `tw_late`, `release_time`) deben expresarse en **minutos** o **segundos**.

<details>
<summary> Ejemplo 1 — Crear un cliente con coordenadas x, y </summary>
   
La forma más directa de agregar un cliente es proporcionando sus coordenadas geográficas. A continuación se crea un cliente ubicado en las coordenadas (-74.0721, 4.7110), con una demanda de 15 unidades y una ventana de tiempo entre las 8:00 y las 12:00. Dado que PyVRP requiere que los tiempos sean enteros, la ventana de tiempo se convierte a minutos: 8:00 corresponde al minuto 480 y las 12:00 al minuto 720 contados desde la medianoche. El tiempo de servicio es de 10 minutos.
 
```python
m = Model()
 
# Ventana de tiempo en horas
tw_early_h = 8    # 8:00
tw_late_h  = 12   # 12:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 720
 
m.add_client(
    x                = -74.0721,
    y                = 4.7110,
    delivery         = 15,
    service_duration = 10,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    name             = "Cliente_1"
)
```
</details>

<details>
<summary> Ejemplo 2 — Crear un cliente sin coordenadas usando x=0, y=0 </summary>

Una alternativa más sencilla cuando se trabaja con matriz de distancias es asignar `x=0` e `y=0` a todos los clientes como valores artificiales. De esta forma se cumple con la firma del método sin necesidad de coordenadas reales, y las distancias entre nodos quedan definidas exclusivamente por la matriz que se provee al modelo. Esta opción es útil cuando las distancias reales ya están precalculadas y no se quiere depender de coordenadas geográficas.
 
```python
m = Model()
 
# Ventana de tiempo en horas
tw_early_h = 9    # 9:00
tw_late_h  = 14   # 14:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 540
tw_late_min  = tw_late_h  * 60   # 840
 
m.add_client(
    x                = 0,
    y                = 0,
    delivery         = 20,
    service_duration = 15,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    name             = "Cliente_1"
)
```
 
> **Nota:** Al usar coordenadas artificiales `x=0, y=0`, es obligatorio proveer la matriz de distancias al modelo. Si no se hace, se calcularán las distancias desde el origen, lo que producirá resultados incorrectos.

</details>

<details>
<summary> Ejemplo 3 — Cargar clientes desde un archivo Excel </summary>
  
Cuando el número de clientes es grande, la forma más práctica es cargarlos desde un archivo Excel. Imaginemos que tenemos un archivo llamado `clientes_pyvrp.xlsx` con la siguiente estructura:

Donde cada fila representa un cliente con sus coordenadas, demanda en unidades, ventana de tiempo de atención en minutos y tiempo de servicio en minutos. A continuación se muestra cómo leer este archivo y agregar todos los clientes al modelo de forma automática.
 
```python
m = Model()
 
df = pd.read_excel("clientes_pyvrp.xlsx", sheet_name="Clientes")
 
for _, row in df.iterrows():
    m.add_client(
        x                = int(row["x"]),
        y                = int(row["y"]),
        delivery         = int(row["demanda"]),
        service_duration = int(row["service_duration"]),
        tw_early         = int(row["tw_early"]),
        tw_late          = int(row["tw_late"]),
        name             = str(row["nombre"])
    )

print(f"Clientes cargados: {len(m.locations)}")
print("\nPrimeros 5 clientes:")
for client in list(m.locations)[:5]:
    print(f"  {client.name} — demanda: {client.delivery} | tw: [{client.tw_early}, {client.tw_late}] | servicio: {client.service_duration} min")
```
**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/clientes_pyvrp.xlsx" download> Clientes Pyvrp</a>

**Solución:**

Se han cargado un total de **10 clientes** en la instancia del problema.

Primeros 5 clientes: 

| Cliente   | Demanda | Ventana de tiempo (min) | Tiempo de servicio |
|-----------|---------|--------------------------|---------------------|
| Cliente_1 | 15      | [0, 240]                 | 10 min              |
| Cliente_2 | 20      | [60, 300]                | 15 min              |
| Cliente_3 | 10      | [30, 180]                | 8 min               |
| Cliente_4 | 25      | [90, 360]                | 12 min              |
| Cliente_5 | 18      | [120, 420]               | 10 min              |
</details>

#### Deposito 

El depósito es el punto desde el cual parten y al cual regresan los vehículos. Se agrega al modelo usando el método `add_depot()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `x` | float | Si | Coordenada x de la ubicación del depósito |
| `y` | float | Si | Coordenada y de la ubicación del depósito |
| `tw_early` | int | No | Inicio de operación del depósito. Por defecto 0 |
| `tw_late` | int | No | Fin de operación del depósito. Sin restricción si no se especifica |
| `service_duration` | int | No | Tiempo que toma cargar un vehículo al inicio de cada ruta. Por defecto 0 |
| `name` | str | No | Nombre descriptivo del depósito. Por defecto cadena vacía |
 
> **Nota:** Los parámetros de tiempo (`tw_early`, `tw_late`, `service_duration`) deben expresarse en **minutos** o **segundos**
> **Nota:** Los ejercicios que se realizan dentro del curso no contempla escenarios multi depositos

<details>
<summary> Ejemplo 1 — Crear un depósito con coordenadas x, y </summary>

La forma más directa de agregar un depósito es proporcionando sus coordenadas geográficas. A continuación se crea un depósito ubicado en las coordenadas (4.7110, -74.0721), con horario de apertura a las 8:00 y cierre a las 18:00, y un tiempo de carga de 15 minutos.
 
```python
m = Model()
 
# Horario del depósito en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
m.add_depot(
    x                = 4.7110,
    y                = -74.0721,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    service_duration = 15,
    name             = "Deposito_Central"
)
```
</details>

<details>
<summary> Ejemplo 2 — Crear un depósito usando x=0, y=0 </summary>
  
Al igual que con los clientes, cuando se dispone de una matriz de distancias precalculada y no se quiere depender de coordenadas geográficas, se pueden usar `x=0` e `y=0` como valores artificiales. En ese caso es obligatorio proveer la matriz de distancias al modelo.
 
```python
m = Model()
 
# Horario del depósito en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
 
# Conversión a minutos (PyVRP requiere enteros)
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
m.add_depot(
    x                = 0,
    y                = 0,
    tw_early         = tw_early_min,
    tw_late          = tw_late_min,
    service_duration = 15,
    name             = "Deposito_Central"
)
```
 
> **Nota:** Al usar coordenadas artificiales `x=0, y=0`, es obligatorio proveer la matriz de distancias al modelo. Si no se hace, PyVRP calculará distancias euclidianas desde el origen, lo que producirá resultados incorrectos.

</details>

#### Tipos de vehículos 

Un tipo de vehículo agrupa las características operativas que comparten los vehículos de una misma categoría dentro de la flota, como su capacidad, horario de operación, costos asociados y perfil de enrutamiento. Se agrega al modelo usando el método `add_vehicle_type()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `num_available` | int | No | Número de vehículos disponibles de este tipo. Por defecto 1 |
| `capacity` | int | No | Capacidad máxima en unidades que puede transportar el vehículo. Por defecto sin restricción |
| `start_depot` | int | No | Índice del depósito desde el que parten los vehículos de este tipo. Por defecto 0 |
| `end_depot` | int | No | Índice del depósito al que regresan los vehículos de este tipo. Por defecto 0 |
| `fixed_cost` | int | No | Costo fijo por utilizar un vehículo de este tipo, independiente de la ruta. Por defecto 0 |
| `tw_early` | int | No | Inicio de operación del vehículo. Por defecto 0 |
| `tw_late` | int | No | Tiempo de finalización del turno del vehículo. Sin restricción si no se especifica |
| `shift_duration` | int | No | Duración máxima de la ruta. Puede extenderse con horas extra. Sin restricción si no se especifica |
| `max_distance` | int | No | Distancia máxima que puede recorrer el vehículo en una ruta. Sin restricción si no se especifica |
| `unit_distance_cost` | int | No | Costo por unidad de distancia recorrida. Por defecto 1 |
| `unit_duration_cost` | int | No | Costo por unidad de tiempo de duración de la ruta. Por defecto 0 |
| `max_overtime` | int | No | Tiempo adicional permitido después de superar la duración máxima establecida. Por defecto 0 |
| `unit_overtime_cost` | int | No | Costo por unidad de tiempo extra. Por defecto 0 |
| `profile` | Profile | No | Perfil de enrutamiento del vehículo, que define los nodos a atender y la matriz de distancias. Por defecto 0 |
| `name` | str | No | Nombre descriptivo del tipo de vehículo. Por defecto cadena vacía |
 
> **Nota:** Los parámetros de tiempo (`tw_early`, `tw_late`, `shift_duration`, `max_overtime`) deben expresarse en **minutos** o **segundos**. El depósito debe haberse agregado al modelo antes de definir los tipos de vehículos.

<details>
<summary> Ejemplo 1 — Flota homogénea </summary>
 
Una flota homogénea está compuesta por vehículos del mismo tipo con las mismas características operativas. Basta con definir un único tipo de vehículo e indicar cuántas unidades están disponibles. A continuación se define una flota de 3 camiones estándar con una capacidad de 50 unidades, un costo fijo de 500 USD por vehículo, turno entre las 8:00 y las 18:00 con una duración de 8 horas, distancia máxima de 200 km, costo de 2 USD/Km y 1 USD/min. Adicionalmente, los vehículos disponen de hasta 30 minutos extra para completar la ruta y regresar al depósito, con un costo adicional de 3 USD por minuto.
 
```python
m = Model()
 
m.add_depot(
    x    = 4.7110,
    y    = -74.0721,
    name = "Deposito_Central"
)
 
# Turno en horas
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00
shift_h    = 8    # 8 horas de turno

# Crear perfil de enrutamiento
perfil = m.add_profile()

# Conversión a minutos
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
shift_min    = shift_h    * 60   # 480
 
m.add_vehicle_type(
    num_available      = 3,
    capacity           = 50,
    fixed_cost         = 500,
    tw_early           = tw_early_min,
    tw_late            = tw_late_min,
    shift_duration     = shift_min,
    max_distance       = 200,
    unit_distance_cost = 2,
    unit_duration_cost = 1,
    max_overtime       = 30,
    unit_overtime_cost = 3,
    profile            = perfil,
    name               = "Camion_Estandar"
)
```
</details>
<details>
<summary> Ejemplo 2 — Flota heterogénea </summary>
 
Una flota heterogénea combina vehículos de distintos tipos con diferentes capacidades, costos y restricciones operativas. Cada tipo se agrega por separado al modelo. A continuación se definen dos tipos de vehículos: un furgón y un camión grande.
 
```python
m = Model()
 
m.add_depot(
    x    = 4.7110,
    y    = -74.0721,
    name = "Deposito_Central"
)
 
# Turno en horas — compartido por ambos tipos
tw_early_h = 8    # 8:00
tw_late_h  = 18   # 18:00

# Crear perfiles de enrutamiento por tipo de vehículo
perfil_furgon = m.add_profile(name="Mediano")
perfil_camion = m.add_profile(name="Pesado")
 
# Conversión a minutos
tw_early_min = tw_early_h * 60   # 480
tw_late_min  = tw_late_h  * 60   # 1080
 
# Tipo 1 — Furgón
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 30,
    fixed_cost         = 300,
    tw_early           = tw_early_min,
    tw_late            = tw_late_min,
    shift_duration     = 8 * 60,    # 480 min
    max_distance       = 150,
    unit_distance_cost = 3,
    unit_duration_cost = 1,
    max_overtime       = 20,
    unit_overtime_cost = 4,
    profile            = perfil_furgon,
    name               = "Furgon"
)
 
# Tipo 2 — Camión grande
m.add_vehicle_type(
    num_available      = 1,
    capacity           = 80,
    fixed_cost         = 800,
    tw_early           = tw_early_min,
    tw_late            = tw_late_min,
    shift_duration     = 10 * 60,   # 600 min
    max_distance       = 300,
    unit_distance_cost = 2,
    unit_duration_cost = 1,
    max_overtime       = 60,
    unit_overtime_cost = 3,
    profile            = perfil_camion,
    name               = "Camion_Grande"
)
```
</details>

<details>
<summary> Ejemplo 3 — Cargar tipos de vehículos desde un archivo Excel </summary>
 
Cuando la flota tiene varios tipos de vehículos, la forma más práctica es cargarlos desde un archivo Excel. Imaginemos que tenemos un archivo llamado `vehiculos_pyvrp.xlsx` donde los vehículos están distribuidos en tres perfiles: **0** para vehículos livianos, **1** para furgones de tamaño mediano y **2** para camiones pesados.
 
> **Nota:** Los perfiles deben crearse antes de cargar los vehículos. El índice de la columna `profile` en el Excel se usa para seleccionar el objeto perfil correspondiente desde un diccionario.
 
```python
m = Model()
 
m.add_depot(
    x    = 4.7110,
    y    = -74.0721,
    name = "Deposito_Central"
)
 
# Crear los tres perfiles de enrutamiento
perfiles = {
    0: m.add_profile(name="Liviano"),
    1: m.add_profile(name="Mediano"),
    2: m.add_profile(name="Pesado")
}
 
df = pd.read_excel("vehiculos_pyvrp.xlsx", sheet_name="Vehiculos")
 
for _, row in df.iterrows():
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        fixed_cost         = int(row["fixed_cost"]),
        tw_early           = int(row["tw_early"]),
        tw_late            = int(row["tw_late"]),
        shift_duration     = int(row["shift_duration"]),
        max_distance       = int(row["max_distance"]),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        max_overtime       = int(row["max_overtime"]),
        unit_overtime_cost = int(row["unit_overtime_cost"]),
        profile            = perfiles[int(row["profile"])],
        name               = str(row["nombre"])
    )
 
print(f"Tipos de vehículos cargados: {len(m.vehicle_types)}")
for vt in m.vehicle_types:
    print(f"  {vt.name} — capacidad: {vt.capacity} | disponibles: {vt.num_available} | perfil: {vt.profile}")
```

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/vehiculos_pyvrp.xlsx" download> Vehiculos Pyvrp</a>

**Solución:**

Tipos de vehículos cargados: 5

| Tipo de vehículo | Capacidad | Vehículos disponibles | Perfil |
|------------------|-----------|------------------------|--------|
| Moto | 15 | 4 | 0 |
| Furgón pequeño | 30 | 3 | 1 |
| Furgón grande | 50 | 2 | 1 |
| Camión | 80 | 2 | 2 |
| Camión pesado | 120 | 1 | 2 |

</details>

#### Arcos 

Un arco conecta dos ubicaciones de la red e indica la distancia y duración del recorrido entre ellas. Se agrega al modelo usando el método `add_edge()`, que acepta los siguientes parámetros:
 
| Parámetro | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `frm` | Client \| Depot | Si | Ubicación de origen del arco |
| `to` | Client \| Depot | Si | Ubicación de destino del arco |
| `distance` | int | Si | Distancia del recorrido |
| `duration` | int | No | Duración del recorrido. Por defecto 0 |
| `profile` | Profile | No | Perfil de enrutamiento al que pertenece el arco |
 
> **Importante:** Los arcos deben agregarse para **todos los pares ordenados** de ubicaciones del modelo, incluyendo los arcos de una ubicación a sí misma con `distance=0` y `duration=0`. Si se omite algún par, el modelo puede quedar infactible.
 
Aproximación de enteros
 
PyVRP requiere que `distance` y `duration` sean enteros. Dependiendo del origen de los datos, esto puede requerir distintos tratamientos:
 
- **Coordenadas geográficas (latitud y longitud):** la distancia euclidiana no es válida con coordenadas geográficas porque un grado de latitud y un grado de longitud no miden lo mismo en kilómetros, y esa diferencia varía según la ubicación en el planeta. La forma correcta es usar la fórmula de **Haversine**, que calcula la distancia real en kilómetros sobre la superficie de la Tierra. Como el resultado es decimal, es obligatorio escalar las distancias y duraciones multiplicándolas por un factor suficientemente grande antes de convertirlas a entero.

- **Matriz de distancias o duraciones con decimales:** si los valores de la matriz son decimales, aplicar `int()` directamente introduce un error de truncamiento. En ese caso es necesario escalar. Si las distancias son decimales, se escala la distancia y también el `max_distance` para que PyVRP pueda compararlos en las mismas unidades. Si la duración es decimal, ya sea porque proviene de una matriz decimal o porque se calcula como `distancia / velocidad (Km/h) × 60`, se debe escalar junto con todos los tiempos del modelo `(tw_early, tw_late, service_duration, shift_duration)`.
  
En todos los casos donde se escale, es necesario escalar de forma consistente todos los tiempos y costos del modelo (`tw_early`, `tw_late`, `service_duration`, `shift_duration`, `unit_distance_cost`, `unit_duration_cost`, `fixed_cost`) por el mismo factor, y desescalar los resultados al reportar.

> **Nota:** si se escala la distancia, no se debe escalar `unit_distance_cost`aunque este sea decimal, y si se escala la duración, no se debe escalar `unit_duration_cost` aunque este sea decimal. Escalar ambos factores del mismo producto haría que ese componente del costo quede multiplicado por `ESCALA²` mientras que el `fixed_cost` y el `prize` solo por `ESCALA`, rompiendo el balance del objetivo. Adicionalmente, si el modelo tiene restricción de autonomía `(max_distance)`, esta también debe escalarse por el mismo factor que las distancias para que PyVRP pueda compararlas correctamente. Normalmente, se utiliza un valor de 100 para el parámetro `ESCALA`.

<details>
<summary> Ejemplo 1 — Distancia euclidiana calculada desde coordenadas </summary>
 
Los clientes tienen coordenadas geográficas `x` e `y` (latitud y longitud) almacenadas en un archivo Excel. La distancia entre cada par de ubicaciones se calcula con la fórmula de **Haversine**, que mide la distancia real en kilómetros sobre la superficie de la Tierra teniendo en cuenta su curvatura. La duración se obtiene dividiendo esa distancia entre la velocidad promedio, expresada en minutos.
 
```python
import math
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime

ESCALA    = 100     # factor de escala 
VELOCIDAD = 40      # km/h

def haversine(lat1, lon1, lat2, lon2):
    """
    Calcula la distancia en kilómetros entre dos puntos
    dados en grados de latitud y longitud.
    """
    R    = 6371  # radio de la Tierra en km
    phi1 = math.radians(lat1)
    phi2 = math.radians(lat2)
    dphi = math.radians(lat2 - lat1)
    dlam = math.radians(lon2 - lon1)
    a    = math.sin(dphi/2)**2 + math.cos(phi1) * math.cos(phi2) * math.sin(dlam/2)**2
    return 2 * R * math.asin(math.sqrt(a))

m = Model()
perfil = m.add_profile()

df = pd.read_excel("arcos_clientes_coordenadas.xlsx", sheet_name="Clientes")

# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df[df["nombre"] == "Deposito"].iloc[0]
depot = m.add_depot(
    x                = fila_dep["x"],
    y                = fila_dep["y"],
    tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
    tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
    service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
    name             = fila_dep["nombre"]
)

# ── Clientes ──────────────────────────────────────────────────────
for _, row in df[df["nombre"] != "Deposito"].iterrows():
    m.add_client(
        x                = row["x"],
        y                = row["y"],
        delivery         = int(row["delivery"]),
        tw_early         = int(row["tw_early (min)"]         * ESCALA),
        tw_late          = int(row["tw_late (min)"]           * ESCALA),
        service_duration = int(row["service_duration (min)"] * ESCALA),
        name             = str(row["nombre"])
    )

# ── Tipo de vehículo ──────────────────────────────────────────────
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 50,
    tw_early           = int(480 * ESCALA),
    tw_late            = int(1080 * ESCALA),
    shift_duration     = int(50  * ESCALA),
    unit_distance_cost = 3,
    unit_duration_cost = 2,
    profile            = perfil,
    name               = "Vehiculo"
)

# ── Arcos: distancia escalada y duración ──
for frm in m.locations:
    for to in m.locations:
        dist_real = haversine(frm.x, frm.y, to.x, to.y)   # km reales
        dur_min   = (dist_real / VELOCIDAD) * 60
        m.add_edge(frm, to,
                   distance = int(dist_real * ESCALA),
                   duration = int(dur_min   * ESCALA),
                   profile  = perfil)

# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best

# ── Resultados desescalados ───────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.4f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")

for route in sol.routes():
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\nRuta        : {' → '.join(nombres)}")
    print(f"  Distancia : {route.distance() / ESCALA:.4f} km")
    print(f"  Duración  : {route.duration() / ESCALA:.2f} min")
```

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/arcos_clientes_coordenadas.xlsx" download> Clientes Coordenadas</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 9.81 km |
| Duración total | 91.71 min |
| Costo por distancia | 29.43 |
| Costo por duración | 183.42 |
| Costo total | 212.85 |

Rutas asignadas

| Ruta | Distancia | Duración |
|------|-----------|----------|
| Cliente_2 → Cliente_4 | 6.46 km | 46.69 min |
| Cliente_3 → Cliente_1 | 3.35 km | 45.02 min |

</details>

<details>
<summary> Ejemplo 2 — Matriz de distancias </summary>
 
Las distancias entre ubicaciones se obtienen a partir de una matriz almacenada en el archivo Excel, mientras que la duración de cada arco se calcula dividiendo la distancia recorrida por la velocidad promedio definida para el vehículo.

Los clientes se incorporan al modelo respetando el mismo orden en el que aparecen registrados en la hoja de clientes. Posteriormente, durante la construcción de los arcos, se recorren las ubicaciones generadas dentro del modelo y se utiliza el nombre asociado a cada una para consultar directamente la distancia correspondiente en la matriz, utilizando la ubicación de origen como fila y la de destino como columna.

Este procedimiento garantiza que cada arco utilice la distancia correcta entre las ubicaciones correspondientes, evitando depender del orden interno asignado por `PyVRP` y asegurando la consistencia entre los datos de entrada y la representación del problema dentro del modelo.

El archivo `arcos_distancias_matriz.xlsx` contiene dos hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo y tiempo de servicio.
- **Distancias:** matriz cuadrada con nombres en filas y columnas, en el mismo orden en que los clientes fueron agregados al modelo.
 
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime

ESCALA    = 100
VELOCIDAD = 40  # km/h

m = Model()
perfil = m.add_profile()

df = pd.read_excel("arcos_distancia_matriz.xlsx", sheet_name="Clientes")

# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df[df["nombre"] == "Deposito"].iloc[0]
depot = m.add_depot(
    x                = 0,
    y                = 0,
    tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
    tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
    service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
    name             = fila_dep["nombre"]
)

# ── Clientes ──────────────────────────────────────────────────────
for _, row in df[df["nombre"] != "Deposito"].iterrows():
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["delivery"]),
        tw_early         = int(row["tw_early (min)"]         * ESCALA),
        tw_late          = int(row["tw_late (min)"]           * ESCALA),
        service_duration = int(row["service_duration (min)"] * ESCALA),
        name             = str(row["nombre"])
    )

# ── Tipo de vehículo ──────────────────────────────────────────────
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 50,
    tw_early           = int(480  * ESCALA),
    tw_late            = int(1080 * ESCALA),
    shift_duration     = int(100  * ESCALA),
    unit_distance_cost = 3,      # no se escala: distancias sin escalar
    unit_duration_cost = 2,      # no se escala: duración ya escalada y número entero
    profile            = perfil,
    name               = "Vehiculo"
)

# ── Arcos desde matriz de distancias, duración desde velocidad ────
df_dist = pd.read_excel("arcos_distancia_matriz.xlsx", sheet_name="Distancias Km", index_col=0)

locs = list(m.locations)

# Self loops
for loc in locs:
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil)

# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
        dist = df_dist.loc[frm.name, to.name]
        dur  = (dist / VELOCIDAD) * 60
        m.add_edge(frm, to,
                   distance = int(dist),
                   duration = int(dur * ESCALA),
                   profile  = perfil)

# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best

# ── Resultados desescalados ───────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()} km")
print(f"Duración total  : {sol.duration() / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost():.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {sol.distance_cost() + sol.duration_cost() / ESCALA:.2f}")

for route in sol.routes():
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\nRuta        : {' → '.join(nombres)}")
    print(f"  Distancia : {route.distance()} km")
    print(f"  Duración  : {route.duration() / ESCALA:.2f} min")
```
 
> **Nota:** Los nombres de las ubicaciones en el modelo deben coincidir exactamente con los nombres de las filas y columnas de la matriz, ya que los arcos se buscan por nombre con `df_dist.loc[frm.name, to.name]`. Cualquier diferencia de mayúsculas, espacios o caracteres hará que la búsqueda falle.

> **Nota:** Este código funciona tanto para matrices simétricas como asimétricas, ya que recorre todos los pares ordenados `(frm, to)` y agrega un arco independiente para cada dirección.

> **Nota:** En este ejemplo las distancias ya son enteros en kilómetros, por lo que `int(dist)` no produce pérdida de información. Sin embargo, la duración calculada como `(dist / VELOCIDAD) * 60` puede resultar decimal, por lo que se escala por `ESCALA` junto con todos los tiempos del modelo para preservar precisión. `unit_duration_cost` no requiere escalamiento, ya que corresponde a un valor entero. En caso de que este valor fuera decimal, tampoco debería aplicarse ningún factor de escala, debido a que la variable de duración asociada ya fue escalada previamente.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/arcos_distancia_matriz.xlsx" download> Clientes Matriz Distancia</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 69 km |
| Duración total | 180.50 min |
| Costo por distancia | 207 |
| Costo por duración | 361 |
| Costo total | 568 |

Rutas asignadas

| Ruta | Distancia | Duración |
|------|-----------|----------|
| Cliente_4 → Cliente_3 | 30 km | 85 min |
| Cliente_1 → Cliente_2 | 39 km | 95.5 min |
</details>

<details>
<summary> Ejemplo 3 — Matriz de duraciones </summary>
 
Cuando no se dispone de distancias sino únicamente de tiempos de recorrido entre ubicaciones, se puede usar solo la matriz de duraciones y definir `distance=0` en todos los arcos. En este caso el costo del vehículo se define únicamente por unidad de duración. El archivo `arcos_matriz_duracion.xlsx` contiene dos hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo y tiempo de servicio.
- **Duraciones:** matriz cuadrada de tiempos de recorrido en minutos con nombres en filas y columnas.
 
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
m = Model()
perfil = m.add_profile()
 
df = pd.read_excel("arcos_matriz_duracion.xlsx", sheet_name="Clientes")
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df[df["nombre"] == "Deposito"].iloc[0]
depot = m.add_depot(x=0, 
                    y=0, 
                    tw_early = int(fila_dep["tw_early (min)"]),
                    tw_late  = int(fila_dep["tw_late (min)"]),
                    service_duration = int(fila_dep["service_duration (min)"]),
                    name=fila_dep["nombre"])
 
# ── Clientes con location asignado por orden de aparición ─────────
for _, row in df[df["nombre"] != "Deposito"].iterrows():
    m.add_client(
        x                = 0,
        y                = 0,
        delivery         = int(row["delivery"]),
        tw_early         = int(row["tw_early (min)"]),
        tw_late          = int(row["tw_late (min)"]),
        service_duration = int(row["service_duration (min)"]),
        name             = str(row["nombre"])
    )
 
# ── Tipo de vehículo ──────────────────────────────────────────────
# Sin unit_distance_cost porque no hay distancias
m.add_vehicle_type(
    num_available      = 2,
    capacity           = 50,
    tw_early           = 480,
    tw_late            = 1080,
    shift_duration     = 120,
    unit_distance_cost = 0,
    unit_duration_cost = 2,
    profile            = perfil,
    name               = "Vehiculo"
)
 
# ── Arcos desde matriz de duraciones, distance=0 ──────────────────
df_dur = pd.read_excel("arcos_matriz_duracion.xlsx", sheet_name="Duraciones (min)", index_col=0)
locs   = list(m.locations)
 
for frm in locs:
    for to in locs:
        dur = df_dur.loc[frm.name, to.name]
        m.add_edge(frm, to,
                   distance = 0,
                   duration = int(dur),
                   profile  = perfil)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible       : {sol.is_feasible()}")
print(f"Duración total : {sol.duration()} min")
print(f"Costo duración : {sol.duration_cost()}")
 
for route in sol.routes():
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\nRuta       : {' → '.join(nombres)}")
    print(f"  Duración : {route.duration()} min")
    print(f"  Costo    : {route.duration_cost()}")
```
> **Nota:** Este código funciona tanto para matrices simétricas como asimétricas, ya que recorre todos los pares ordenados `(frm, to)` y agrega un arco independiente para cada dirección.

> **Nota:** En este ejemplo no es necesario escalar ya que las duraciones de la matriz son enteras en minutos, por lo que `int(dur)` no produce pérdida de información.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/arcos_matriz_duracion.xlsx" download> Clientes Matriz Duración</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Duración total | 200 min |
| Costo por duración | 400 |

Rutas asignadas

| Ruta | Duración | Costo |
|------|----------|-------|
| Cliente_3 → Cliente_4 | 93 min | 186 |
| Cliente_2 → Cliente_1 | 107 min | 214 |

</details>


#### Perfiles de enrutamiento 
 
Un perfil de enrutamiento define un conjunto de arcos con sus distancias y duraciones, formando una matriz completa de conectividad entre ubicaciones. Cada tipo de vehículo tiene un perfil asignado, y ese perfil determina exactamente qué arcos puede recorrer durante su ruta.
 
Esto permite modelar dos tipos de restricciones operativas que no pueden expresarse directamente con los atributos estándar de PyVRP:
 
- **Restricciones sobre clientes:** ciertos vehículos solo pueden atender a un subconjunto de clientes. Por ejemplo, furgones refrigerados para clientes que requieren cadena de frío, y motos para clientes de paquetería ligera. Esto se modela asignando una distancia muy grande (`INF`) a los arcos que conectan con los clientes que no se pueden atender, haciendo que nunca se incluyan en ninguna ruta.
- **Restricciones sobre caminos:** ciertos vehículos solo pueden circular por ciertas vías. Por ejemplo, los camiones de carga pesada solo pueden circular por vías principales debido a restricciones de peso y dimensiones, mientras que los furgones, al ser vehículos más compactos, tienen acceso tanto a vías principales como a vías secundarias, lo que les permite alcanzar zonas donde los camiones no pueden ingresar. Esto se modela con matrices de distancias distintas para cada perfil, donde los arcos no permitidos tienen distancia `INF`.
En ambos casos, `INF` debe ser un número entero suficientemente grande para que los arcos nunca se elijan, pero que no cause desbordamiento numérico. Un valor como `100000` funciona bien en la mayoría de los casos.
- **Velocidades heterogéneas:** cuando los vehículos de la flota tienen velocidades distintas, la duración de cada arco varía según el tipo de vehículo. Como PyVRP asigna una duración fija por arco, es necesario crear un perfil por tipo de vehículo donde cada perfil tenga matriz de duración con la velocidad correspondiente.

<details>
<summary> Ejemplo 1 — Vehículos con clientes restringidos </summary>
 
FreshRoute opera una flota mixta de **furgones** y **motos** para distribución urbana. Los furgones tienen mayor capacidad y atienden clientes con pedidos grandes, mientras que las motos son más ágiles y atienden clientes con pedidos pequeños. Por política operativa, cada tipo de vehículo solo puede atender a su grupo de clientes asignado.
 
El archivo `perfiles_clientes_restringidos.xlsx` contiene tres hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo (min), tiempo de servicio (min) y `tipo_atencion` que indica si el cliente es atendido por `furgon` o `moto`.
- **Vehiculos:** nombre, cantidad disponible, capacidad, velocidad (Km/h), ventanas de tiempo (min), duración del turno (min) y costos unitarios.
- **Distancias:** matriz de distancias en km entre todas las ubicaciones.

```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
ESCALA = 100      # factor de escala
INF    = 10**4   # distancia prohibida
 
m = Model()
perfil_furgon = m.add_profile(name="Furgon")
perfil_moto   = m.add_profile(name="Moto")
 
df_cli  = pd.read_excel("perfiles_clientes_restringidos.xlsx", sheet_name="Clientes")
df_veh  = pd.read_excel("perfiles_clientes_restringidos.xlsx", sheet_name="Vehiculos")
df_dist = pd.read_excel("perfiles_clientes_restringidos.xlsx", sheet_name="Distancias", index_col=0)
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df_cli[df_cli["nombre"] == "Deposito"].iloc[0]
m.add_depot(x=0, y=0,
            tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
            tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
            service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
            name             = fila_dep["nombre"])
 
# ── Clientes ──────────────────────────────────────────────────────
# Se guarda el tipo de atención de cada cliente para usarlo al construir los arcos
tipo_atencion = {}
for _, row in df_cli[df_cli["nombre"] != "Deposito"].iterrows():
    m.add_client(x=0, y=0,
                 delivery         = int(row["delivery"]),
                 tw_early         = int(row["tw_early (min)"]         * ESCALA),
                 tw_late          = int(row["tw_late (min)"]           * ESCALA),
                 service_duration = int(row["service_duration (min)"] * ESCALA),
                 name             = str(row["nombre"]))
    tipo_atencion[row["nombre"]] = row["tipo_atencion"]
 
# ── Tipos de vehículo ─────────────────────────────────────────────
perfiles_veh = {"Furgon": perfil_furgon, "Moto": perfil_moto}
vel_por_tipo  = {}
 
for _, row in df_veh.iterrows():
    nombre = row["nombre"]
    vel_por_tipo[nombre] = float(row["velocidad (Km/h)"])
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        tw_early           = int(row["tw_early (min)"]       * ESCALA),
        tw_late            = int(row["tw_late (min)"]         * ESCALA),
        shift_duration     = int(row["shift_duration (min)"] * ESCALA),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        profile            = perfiles_veh[nombre],
        name               = nombre
    )
 
# ── Arcos ─────────────────────────────────────────────────────────
# Para cada perfil, los arcos cuyo nodo de salida o llegada no sea atendido por ese vehículo reciben distancia INF
locs = list(m.locations)
 
# Self loops: distancia y duración 0 para ambos perfiles
for loc in locs:
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_furgon)
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_moto)
 
# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
 
        dist_km = df_dist.loc[frm.name, to.name]
 
        # Perfil furgon: INF si el arco involucra un cliente de moto
        if tipo_atencion.get(to.name) == "moto" or tipo_atencion.get(frm.name) == "moto":
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_furgon)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Furgon"]) * 60 * ESCALA),
                       profile  = perfil_furgon)
 
        # Perfil moto: INF si el arco involucra un cliente de furgon
        if tipo_atencion.get(to.name) == "furgon" or tipo_atencion.get(frm.name) == "furgon":
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_moto)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Moto"]) * 60 * ESCALA),
                       profile  = perfil_moto)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.2f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\n  {vt.name}: {' → '.join(nombres)}")
    print(f"    Distancia : {route.distance() / ESCALA:.2f} km")
    print(f"    Duración  : {route.duration() / ESCALA:.2f} min")
```
 
> **Escalado:** las distancias de la matriz son decimales, por lo que se escala por `ESCALA = 100` para preservar la exactitud tanto en la distancia como en la duración calculada.

> **INF:** los arcos prohibidos reciben `distance = INF = 10**4`. Los arcos cuyo nodo de salida y nodo de llegada es igual siempre tienen `distance = 0` y `duration = 0` independientemente del perfil, ya que PyVRP lo exige explícitamente.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/perfiles_clientes_restringidos.xlsx" download> Perfiles Clientes Restringidos</a>

**Solución:**

Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 102.79 km |
| Duración total | 257.09 min |
| Costo por distancia | 247.87 |
| Costo por duración | 381.58 |
| Costo total 629.45 |

Rutas asignadas

| Vehículo | Ruta | Distancia | Duración |
|----------|------|-----------|----------|
| Furgón | Cliente_1 → Cliente_2 → Cliente_3 | 42.29 km | 124.49 min |
| Moto | Cliente_4 → Cliente_6 | 38.90 km | 79.68 min |
| Moto | Cliente_5 | 21.60 km | 52.92 min |
 
</details>
<details>
<summary> Ejemplo 2 — Vehículos con caminos restringidos </summary>
 
LogiRed opera una red de distribución que utiliza **camiones** y **furgones**. Los camiones, por restricciones de peso, solo pueden circular por vías principales, mientras que los furgones, al ser vehículos más compactos, tienen acceso únicamente a vías secundarias, lo que les permite alcanzar zonas donde los camiones no pueden ingresar. La red tiene una única matriz de distancias, pero cada arco tiene asociado un tipo de vía que determina qué vehículos pueden usarlo.
 
El archivo `perfiles_caminos_restringidos.xlsx` contiene cuatro hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo (min) y tiempo de servicio (min).
- **Vehiculos:** nombre, cantidad, capacidad, velocidad (Km/h), ventanas de tiempo (min), duración del turno (min), costos unitarios y `vias_permitidas` que indica el tipo de vía que puede usar ese vehículo (`primaria` o `secundaria`).
- **Distancias:** matriz de distancias en km entre todas las ubicaciones.
- **Vias:** matriz con el tipo de vía de cada arco (`primaria` o `secundaria`).
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
ESCALA = 100       # factor de escala
INF    = 10**4    # distancia prohibida
 
m = Model()
perfil_camion = m.add_profile(name="Camion")
perfil_furgon = m.add_profile(name="Furgon")
 
df_cli  = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Clientes")
df_veh  = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Vehiculos")
df_dist = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Distancias", index_col=0)
df_via  = pd.read_excel("perfiles_caminos_restringidos.xlsx", sheet_name="Vias",       index_col=0)
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df_cli[df_cli["nombre"] == "Deposito"].iloc[0]
m.add_depot(x=0, y=0,
            tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
            tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
            service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
            name             = fila_dep["nombre"])
 
# ── Clientes ──────────────────────────────────────────────────────
for _, row in df_cli[df_cli["nombre"] != "Deposito"].iterrows():
    m.add_client(x=0, y=0,
                 delivery         = int(row["delivery"]),
                 tw_early         = int(row["tw_early (min)"]         * ESCALA),
                 tw_late          = int(row["tw_late (min)"]           * ESCALA),
                 service_duration = int(row["service_duration (min)"] * ESCALA),
                 name             = str(row["nombre"]))
 
# ── Tipos de vehículo ─────────────────────────────────────────────
perfiles_map  = {"Camion": perfil_camion, "Furgon": perfil_furgon}
vel_por_tipo  = {}
vias_por_tipo = {}
 
for _, row in df_veh.iterrows():
    nombre = row["nombre"]
    vel_por_tipo[nombre]  = float(row["velocidad (Km/h)"])
    vias_por_tipo[nombre] = row["vias_permitidas"]
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        tw_early           = int(row["tw_early (min)"]       * ESCALA),
        tw_late            = int(row["tw_late (min)"]         * ESCALA),
        shift_duration     = int(row["shift_duration (min)"] * ESCALA),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        profile            = perfiles_map[nombre],
        name               = nombre
    )
 
# ── Arcos ─────────────────────────────────────────────────────────
# Se consulta la hoja Vias para saber si cada arco está permitido
# para cada tipo de vehículo según su vias_permitidas
locs = list(m.locations)
 
# Self loops: distancia y duración 0 para ambos perfiles
for loc in locs:
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_camion)
    m.add_edge(loc, loc, distance=0, duration=0, profile=perfil_furgon)
 
# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
 
        dist_km  = df_dist.loc[frm.name, to.name]
        tipo_via = df_via.loc[frm.name, to.name]
 
        # Camion: INF si el tipo de vía no coincide con su vía permitida
        if tipo_via != vias_por_tipo["Camion"]:
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_camion)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Camion"]) * 60 * ESCALA),
                       profile  = perfil_camion)
 
        # Furgon: INF si el tipo de vía no coincide con su vía permitida
        if tipo_via != vias_por_tipo["Furgon"]:
            m.add_edge(frm, to, distance=INF, duration=0, profile=perfil_furgon)
        else:
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int((dist_km / vel_por_tipo["Furgon"]) * 60 * ESCALA),
                       profile  = perfil_furgon)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.2f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\n  {vt.name}: {' → '.join(nombres)}")
    print(f"    Distancia : {route.distance() / ESCALA:.2f} km")
    print(f"    Duración  : {route.duration() / ESCALA:.2f} min")
```
 
> **Escalado:** las distancias de la matriz son decimales, por lo que se escala por `ESCALA = 100` para preservar la exactitud tanto en la distancia como en la duración calculada. Todos los tiempos del modelo se escalan por el mismo factor para mantener consistencia interna.

> **Vías restringidas:** en lugar de tener matrices separadas por tipo de vehículo, se usa una única matriz de distancias y una hoja adicional que indica el tipo de vía de cada arco. Al construir los arcos, se consulta esa hoja y se asigna `INF` cuando el tipo de vía no coincide con el permitido por el vehículo, impidiendo que se utilicen esos arcos en la solución.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND-2206-Ingenieria-de-Cadena-de-Suministro/main/perfiles_caminos_restringidos.xlsx" download> Perfiles Caminos Restringidos</a>

**Solución:**
Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 302.79 km |
| Duración total | 234.05 min |
| Costo por distancia | 908.37 |
| Costo por duración | 234.05 |
| Costo total | 1142.42 |

Rutas asignadas

| Vehículo | Ruta | Distancia | Duración |
|----------|------|-----------|----------|
| Furgón | Cliente_6 → Cliente_2 → Cliente_5 | 164.90 km | 133.52 min |
| Furgón | Cliente_3 → Cliente_1 → Cliente_4 | 137.89 km | 100.53 min |

</details>

<details>
<summary> Ejemplo 3 — Flota con velocidades heterogéneas </summary>
 
TransRed opera con una flota mixta compuesta por **camiones**, **furgones** y **motos**, cada uno con una velocidad distinta. Como PyVRP asigna una duración fija por arco, es necesario crear un perfil por tipo de vehículo donde cada perfil tenga sus propias duraciones calculadas con la velocidad correspondiente. La matriz de distancias es la misma para todos, pero la duración de cada arco varía según el vehículo que lo recorre.
 
El archivo `perfiles_velocidades_heterogeneas.xlsx` contiene tres hojas:
 
- **Clientes:** nombre, demanda, ventanas de tiempo (min) y tiempo de servicio (min).
- **Vehiculos:** nombre, cantidad, capacidad, velocidad (Km/h), ventanas de tiempo (min), duración del turno (min) y costos unitarios.
- **Distancias:** matriz de distancias en km entre todas las ubicaciones.
```python
import pandas as pd
from pyvrp import Model
from pyvrp.stop import MaxRuntime
 
ESCALA = 100    # factor de escala para distancias y duraciones decimales
 
m = Model()
perfiles = {}
 
df_cli  = pd.read_excel("perfiles_velocidades_heterogeneas.xlsx", sheet_name="Clientes")
df_veh  = pd.read_excel("perfiles_velocidades_heterogeneas.xlsx", sheet_name="Vehiculos")
df_dist = pd.read_excel("perfiles_velocidades_heterogeneas.xlsx", sheet_name="Distancias", index_col=0)
 
# ── Perfiles: uno por tipo de vehículo ────────────────────────────
for _, row in df_veh.iterrows():
    tipo = row["nombre"]
    vel  = float(row["velocidad (Km/h)"])
    p    = m.add_profile(name=tipo)
    perfiles[tipo] = (p, vel)
 
# ── Depósito ──────────────────────────────────────────────────────
fila_dep = df_cli[df_cli["nombre"] == "Deposito"].iloc[0]
m.add_depot(x=0, y=0,
            tw_early         = int(fila_dep["tw_early (min)"]         * ESCALA),
            tw_late          = int(fila_dep["tw_late (min)"]           * ESCALA),
            service_duration = int(fila_dep["service_duration (min)"] * ESCALA),
            name             = fila_dep["nombre"])
 
# ── Clientes ──────────────────────────────────────────────────────
for _, row in df_cli[df_cli["nombre"] != "Deposito"].iterrows():
    m.add_client(x=0, y=0,
                 delivery         = int(row["delivery"]),
                 tw_early         = int(row["tw_early (min)"]         * ESCALA),
                 tw_late          = int(row["tw_late (min)"]           * ESCALA),
                 service_duration = int(row["service_duration (min)"] * ESCALA),
                 name             = str(row["nombre"]))
 
# ── Tipos de vehículo ─────────────────────────────────────────────
for _, row in df_veh.iterrows():
    nombre = row["nombre"]
    p, vel = perfiles[nombre]
    m.add_vehicle_type(
        num_available      = int(row["num_available"]),
        capacity           = int(row["capacity"]),
        tw_early           = int(row["tw_early (min)"]       * ESCALA),
        tw_late            = int(row["tw_late (min)"]         * ESCALA),
        shift_duration     = int(row["shift_duration (min)"] * ESCALA),
        unit_distance_cost = int(row["unit_distance_cost"]),
        unit_duration_cost = int(row["unit_duration_cost"]),
        profile            = p,
        name               = nombre
    )
 
# ── Arcos ─────────────────────────────────────────────────────────
# Cada perfil recibe sus propios arcos con duraciones calculadas
# según la velocidad del vehículo correspondiente
locs = list(m.locations)
 
# Self loops: distancia y duración 0 para todos los perfiles
for loc in locs:
    for p, vel in perfiles.values():
        m.add_edge(loc, loc, distance=0, duration=0, profile=p)
 
# Arcos entre ubicaciones distintas
for frm in locs:
    for to in locs:
        if frm.name == to.name:
            continue
        dist_km = df_dist.loc[frm.name, to.name]
        for tipo, (p, vel) in perfiles.items():
            dur_min = (dist_km / vel) * 60
            m.add_edge(frm, to,
                       distance = int(dist_km * ESCALA),
                       duration = int(dur_min  * ESCALA),
                       profile  = p)
 
# ── Resolver ──────────────────────────────────────────────────────
result = m.solve(stop=MaxRuntime(3), seed=42)
sol    = result.best
 
# ── Resultados ────────────────────────────────────────────────────
print(f"Factible        : {sol.is_feasible()}")
print(f"Distancia total : {sol.distance()      / ESCALA:.2f} km")
print(f"Duración total  : {sol.duration()      / ESCALA:.2f} min")
print(f"Costo distancia : {sol.distance_cost() / ESCALA:.2f}")
print(f"Costo duración  : {sol.duration_cost() / ESCALA:.2f}")
print(f"Costo total     : {(sol.distance_cost() + sol.duration_cost()) / ESCALA:.2f}")
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
    print(f"\n  {vt.name}: {' → '.join(nombres)}")
    print(f"    Distancia : {route.distance() / ESCALA:.2f} km")
    print(f"    Duración  : {route.duration() / ESCALA:.2f} min")
```
 
> **Escalado:** las distancias de la matriz son decimales, por lo que se escala por `ESCALA = 100` tanto la distancia como la duración de cada arco. Todos los tiempos del modelo se escalan por el mismo factor. `unit_distance_cost` y `unit_duration_cost` no se escalan aunque sean decimales, ya que la distancia y la duración sí fueron escaladas.

**Base de datos:** <a href="https://raw.githubusercontent.com/Mariapaulaestupinan/IIND---2206-Ingenier-a-de-Cadena-de-Suministro-2026---19/main/perfiles_velocidades_heterogeneas.xlsx" download>Perfiles Caminos Restringidos</a>

**Solución:**
Resumen general

| Parámetro | Resultado |
|-----------|-----------|
| Distancia total recorrida | 185.66 km |
| Duración total | 598.72 min |
| Costo por distancia | 532.16 |
| Costo por duración | 822.15 |
| Costo total | 1354.31 |

Rutas asignadas

| Vehículo | Ruta | Distancia | Duración |
|----------|------|-----------|----------|
| Camion | Cliente_3 → Cliente_5 → Cliente_19 → Cliente_7 | 27.79 km | 114.63 min |
| Camion | Cliente_4 → Cliente_8 → Cliente_6 → Cliente_18 | 24.38 km | 108.80 min |
| Furgón | Cliente_10 → Cliente_1 → Cliente_13 | 26.20 km | 86.92 min |
| Furgón | Cliente_12 → Cliente_11 → Cliente_15 → Cliente_17 | 30.30 km | 99.38 min |
| Moto | Cliente_21 → Cliente_14 → Cliente_16 | 35.79 km | 80.79 min |
| Moto | Cliente_9 → Cliente_20 | 30.40 km | 67.40 min |
| Moto | Cliente_2 | 10.80 km | 40.80 min |

</details>

#### Solución 
 
Una vez construido el modelo con clientes, depósito, tipos de vehículo y arcos, se resuelve llamando al método `m.solve()`. Este método recibe obligatoriamente un criterio de parada y opcionalmente una semilla para el generador aleatorio.
 
```python
from pyvrp.stop import MaxRuntime, MaxIterations
 
result = m.solve(stop=MaxRuntime(30), seed=42)
```
 
<details>
<summary> Criterios de parada </summary>
 
PyVRP implementa un algoritmo de búsqueda iterativa que mejora la solución progresivamente. Es necesario indicarle cuándo debe detenerse mediante uno de los siguientes criterios:
 
**`MaxRuntime(segundos)`** — detiene el algoritmo cuando se alcanza el tiempo máximo de ejecución en segundos. Es el criterio más común en la práctica porque permite controlar cuánto tiempo se dedica a la exploración del espacio de busqueda.
 
```python
result = m.solve(stop=MaxRuntime(60), seed=42)   # máximo 60 segundos
```
 
**`MaxIterations(iteraciones)`** — detiene el algoritmo cuando se alcanza el número máximo de iteraciones. Útil cuando se quiere garantizar exactamente la misma cantidad de trabajo computacional entre ejecuciones.
 
```python
result = m.solve(stop=MaxIterations(10000), seed=42)   # máximo 10000 iteraciones
```
</details>
<details>
<summary> Semilla </summary>
 
El parámetro `seed` controla el generador de números aleatorios del algoritmo. Dado que PyVRP es un algoritmo heurístico con componentes aleatorios, dos ejecuciones con la misma semilla producen exactamente el mismo resultado, lo que permite reproducir soluciones. Cambiar la semilla puede dar soluciones distintas de diferente calidad.
 
```python
result = m.solve(stop=MaxRuntime(30), seed=42)    # reproducible
result = m.solve(stop=MaxRuntime(30), seed=123)   # puede dar resultado diferente
```
</details>
<details>
<summary> Resultados globales </summary>
 
La solución se accede a través de `result.best`, que contiene la mejor solución encontrada durante la ejecución.
 
```python
sol = result.best
```
 
Los principales atributos globales disponibles son:
 
```python
print(f"Factible            : {sol.is_feasible()}")
print(f"Distancia total     : {sol.distance()}")
print(f"Duración total      : {sol.duration()}")
print(f"Costo de distancia  : {sol.distance_cost()}")
print(f"Costo de duración   : {sol.duration_cost()}")
print(f"Costo fijo total    : {sol.fixed_vehicle_cost()}")
print(f"Costo total         : {sol.distance_cost() + sol.duration_cost() + sol.fixed_vehicle_cost()}")
```
 
Ejemplo de salida:
 
```
Factible            : True
Distancia total     : 8450
Duración total      : 24320
Costo de distancia  : 25350
Costo de duración   : 48640
Costo fijo total    : 1000
Costo total         : 74990
```
 
> **Nota:** Si se usó escalado, todos estos valores deben desescalarse dividiendo por el factor correspondiente antes de interpretarlos.

</details>
<details>
<summary> Resultados por ruta </summary>
 
Para acceder a los detalles de cada ruta se itera sobre `sol.routes()`. Cada ruta registra su secuencia de nodos, distancia, duración, costos y los tiempos de inicio y fin del servicio en cada nodo mediante `route.schedule()`.
 
```python
def min_a_hora(minutos):
    """Convierte minutos desde medianoche a formato HH:MM."""
    h = int(minutos) // 60
    m = int(minutos) % 60
    return f"{h:02d}:{m:02d}"
 
for route in sol.routes():
    vt      = m.vehicle_types[route.vehicle_type()]
    nombres = [m.locations[v].name for v in route.visits()]
 
    print(f"Vehículo  : {vt.name}")
    print(f"Ruta      : {' → '.join(nombres)}")
    print(f"Distancia : {route.distance()}")
    print(f"Duración  : {route.duration()} min")
    print(f"Costo dist: {route.distance_cost()}")
    print(f"Costo dur : {route.duration_cost()}")
    print(f"Inicio    : {min_a_hora(route.start_time())}")
    print(f"Fin       : {min_a_hora(route.end_time())}")
```
 
Ejemplo de salida:
 
```
Vehículo  : Furgon
Ruta      : Cliente_2 → Cliente_5 → Cliente_3
Distancia : 3120
Duración  : 8740 min
Costo dist: 9360
Costo dur : 17480
Inicio    : 08:00
Fin       : 14:33
```
 
> **Nota:** Los tiempos de `start_time()`, `end_time()`, `start_service` y `end_service` están en las mismas unidades con las que se definieron las ventanas de tiempo en el modelo. Si se usó escalado, deben desescalarse antes de convertirlos a formato `HH:MM`. La función `min_a_hora()` asume que los valores ya están en minutos desde medianoche.

</details>


