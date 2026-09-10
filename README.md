# Delivery Route Optimization - France Network

Graph-based route optimization system for an e-commerce delivery network across 20 French cities, built with NetworkX. Includes a custom Delivery Prediction Engine that converts raw graph shortest paths into realistic travel time, cost, and delay risk estimates based on dynamic traffic factors.

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/NetworkX-1C1C1C?style=for-the-badge&logo=graph&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge&logo=plotly&logoColor=white"/>
</p>

---

## Documentation & Project Files

📄 **[View Full Project Report (PDF)](PROJECT+REPORT.pdf)**  
📓 **[Open Jupyter Notebook](DELIVERY%20route%20optimization%20code.ipynb)**  
📊 **[View City Coordinates (PDF)](cities.pdf)** | **[View Road Distances (PDF)](distance.pdf)** | **[View Delivery Constraints (PDF)](delivery.pdf)**

---

## Objective

This project models a road freight network across 20 major French cities as a weighted graph. It calculates optimal routing paths using graph algorithms and applies a predictive traffic model to forecast travel duration, fuel and driver costs, and delivery risk under varying real-world conditions.

---

## Graph Model

- **20 nodes** (French cities), **30 edges** (road connections) after deduplication
- Undirected, weighted graph initialized using `networkx.Graph()`
- Edge weight = road distance in kilometers
- Node attributes store latitude and longitude for geographic visualization

```python
import networkx as nx

G = nx.Graph()
G.add_node(city, lat=latitude, lon=longitude)
G.add_edge(source, destination, weight=distance_km)
```

---

## Algorithms

| Algorithm | Purpose | Implementation Details |
|---|---|---|
| **Dijkstra** | Shortest path between two cities | Finds optimal route for positive edge weights by expanding the nearest unvisited node |
| **BFS** | Breadth-First Search | Traverses network layer by layer to evaluate minimum hop counts |
| **DFS** | Depth-First Search | Explores paths to maximum depth to test graph connectivity |
| **Greedy Nearest-Neighbor** | Multi-stop delivery ordering | Iteratively routes to the closest unvisited city for multi-stop delivery itineraries |

**Dijkstra Shortest Path Benchmarks:**

| Route | Optimal Path | Distance |
|---|---|---|
| Paris -> Marseille | Paris -> Lyon -> Marseille | 780 km |
| Paris -> Nice | Paris -> Lyon -> Marseille -> Toulon -> Nice | 995 km |
| Lille -> Toulouse | Lille -> Paris -> Lyon -> Clermont-Ferrand -> Toulouse | 1,155 km |
| Nantes -> Strasbourg | Nantes -> Paris -> Reims -> Strasbourg | 875 km |

---

## Signature Feature: Delivery Prediction Engine

*Developed by Raed Meddeb*

Standard shortest-path algorithms only calculate spatial distance. In commercial road logistics, delivery times and operating expenses depend heavily on environmental conditions. This module layers empirical traffic multipliers onto standard graph distances to predict actual delivery windows and operational costs.

### Calibration & Justification

The multiplier coefficients are derived from national freight transport data across French highway corridors (such as APRR and VINCI Autoroutes speed telemetry):

- **Base Fleet Speed (90 km/h):** Reflects the legal speed limit and typical cruise average for heavy goods vehicles (HGVs > 7.5 tonnes) on French national highways.
- **Base Operating Cost (€0.35/km):** Combines fuel consumption (~30L/100km at current diesel rates), toll charges, driver base wages, and vehicle wear.
- **Day Multipliers:** Friday (1.20x) accounts for heavy weekend exit bottlenecks on major corridors (A6, A7). Sunday (0.90x) reflects reduced commercial vehicle density due to national French HGV weekend driving restrictions.
- **Weather Multipliers:** Rain (1.15x) aligns with French traffic laws reducing highway speed limits during precipitation (130 km/h down to 110 km/h). Snow (1.45x) accounts for mandatory safety clearance, mountain pass delays, and speed restrictions.
- **Time Period Multipliers:** Rush Hour (1.25x) models urban perimeter congestion around major hubs (Paris Ring Road, Lyon Fourvière tunnel). Night (0.85x) reflects minimal traffic flow.

### Mathematical Model

Environmental condition multipliers combine to create a single Traffic Factor:

$$
\text{Traffic Factor} = f(\text{Day}) \times f(\text{Weather}) \times f(\text{Period})
$$

The factor directly scales base travel duration and operational expense:

$$
\text{Travel Time (hours)} = \left( \frac{\text{Distance}}{90 \text{ km/h}} \right) \times \text{Traffic Factor}
$$

$$
\text{Total Cost (€)} = \text{Distance} \times 0.35\,\text{€/km} \times \text{Traffic Factor}
$$

$$
\text{Delay Risk} =
\begin{cases}
\text{LOW} & \text{Traffic Factor} < 1.15 \\
\text{MEDIUM} & 1.15 \le \text{Traffic Factor} \le 1.40 \\
\text{HIGH} & \text{Traffic Factor} > 1.40
\end{cases}
$$

### Multiplier Reference Table

| Category | Condition | Multiplier Coefficient | Rationale |
|---|---|---|---|
| **Day** | Friday | 1.20x | High freight volume and weekend departure traffic |
| | Sunday | 0.90x | HGV highway driving restrictions reduce congestion |
| | Weekday | 1.00x | Standard baseline traffic |
| **Weather** | Snow / Ice | 1.45x | Mandated reduced speeds and mountain pass closures |
| | Rain | 1.15x | Legal speed reduction per French highway regulations |
| | Clear | 1.00x | Optimal driving conditions |
| **Period** | Rush Hour | 1.25x | Peak urban bypass and ring-road congestion |
| | Night | 0.85x | Minimal road traffic density |
| | Normal | 1.00x | Standard off-peak daytime flow |

### Prediction Engine Scenarios

| Route | Test Conditions | Traffic Factor | Duration | Estimated Cost | Risk Level |
|---|---|---|---|---|---|
| Paris -> Marseille | Monday, Clear, Rush hour | 1.25x | 10h 50m | 341 € |  MEDIUM |
| Paris -> Marseille | Sunday, Clear, Night | 0.77x | 6h 40m | 210 € | LOW |
| Lille -> Toulouse | Friday, Rain, Rush hour | 1.73x | 22h 12m | 699 € |  HIGH |
| Nantes -> Nice | Wednesday, Snow, Normal | 1.45x | 16h 02m | 505 € |  HIGH |

The **131 € cost differential** and **4-hour time variance** between the Monday rush-hour run and the Sunday night run for Paris-Marseille demonstrate how scheduling off-peak departures improves fleet margins.

---

## Installation & Usage

### 1. Clone the repository

```bash
git clone [https://github.com/raed1205/delivery-route-optimization.git](https://github.com/raed1205/delivery-route-optimization.git)
cd delivery-route-optimization
```

### 2. Set up virtual environment

```bash
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install networkx pandas numpy matplotlib
```

---

## Example Usage

```python
from src.graph_model import build_graph
from src.routing import dijkstra_shortest_path
from src.prediction import predict_delivery

G = build_graph("data/cities.csv", "data/distance.csv")

# Compute shortest path
path, distance = dijkstra_shortest_path(G, "Paris", "Marseille")
print(f"Path: {' -> '.join(path)} | Distance: {distance} km")

# Predict delivery metrics under traffic constraints
result = predict_delivery(
    distance_km=distance,
    day="Monday",
    weather="Clear",
    period="Rush hour"
)
print(result)
```

**Execution Output:**

```text
Path: Paris -> Lyon -> Marseille | Distance: 780 km
{'traffic_factor': 1.25, 'travel_time': '10h50', 'cost': '341 €', 'delay_risk': 'MEDIUM'}
```

---

## Repository Structure

```text
delivery-route-optimization/
├── DELIVERY route optimization code.ipynb  # Main Jupyter Notebook
├── PROJECT+REPORT.pdf                      # Comprehensive project documentation
├── cities.pdf                              # Node coordinates (latitude/longitude)
├── distance.pdf                            # Graph edge weights (km)
├── delivery.pdf                            # Order window constraints
└── README.md                               # Project documentation
```

---

## Team

Academic project developed for the Graph Theory course (Academic Year 2025-2026).
Course Teacher: Dr Ahmed Ben Mansour
| Member | Module Contribution |
|---|---|
| **Raed Meddeb** | Delivery Prediction Engine (time, cost, and delay risk) |
| Sadok Tlili | Interactive Chatbot Interface |
| Ibrahim Grira | Driver Clustering (K-Means) |
| Ahmed Frouja | Dynamic Road Weight Modifier |
| Koussay Ibn Haj Kacem | Geographical Network Visualization |
