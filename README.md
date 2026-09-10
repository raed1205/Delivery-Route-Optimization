# Delivery Route Optimization - France Network

Graph-based route optimization system for an e-commerce delivery network across 20 French cities, built with NetworkX. Features a Delivery Prediction Engine that converts graph shortest paths into estimated travel time, cost, and delay risk based on traffic factors.

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/NetworkX-1C1C1C?style=for-the-badge&logo=graph&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge&logo=plotly&logoColor=white"/>
</p>

---

## Documentation & Code

📄 **[View Full Project Report (PDF)](PROJECT+REPORT.pdf)**  
📓 **[Open Jupyter Notebook](DELIVERY%20route%20optimization%20code.ipynb)**

---

## Objective

This system models a road network across 20 French cities as a weighted graph. It calculates optimal paths using standard routing algorithms and applies dynamic traffic condition multipliers (weather, day of week, rush hour) to forecast realistic delivery times, fuel costs, and scheduling risk before dispatching vehicles.

---

## Graph Model

- **20 nodes** (French cities), **30 edges** (road connections) after deduplication
- Undirected, weighted graph initialized with `networkx.Graph()`
- Edge weights represent distance in kilometers
- Node attributes contain latitude and longitude coordinates for geographical mapping

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
| **BFS** | Breadth-First Search | Explores network layer by layer to evaluate minimum hop count |
| **DFS** | Depth-First Search | Traverses paths to maximum depth to assess network connectivity |
| **Greedy Nearest-Neighbor** | Multi-stop order routing | Iteratively visits the closest unvisited city for multi-destination delivery sequences |

**Dijkstra Shortest Path Benchmarks:**

| Route | Optimal Path | Distance |
|---|---|---|
| Paris -> Marseille | Paris -> Lyon -> Marseille | 780 km |
| Paris -> Nice | Paris -> Lyon -> Marseille -> Toulon -> Nice | 995 km |
| Lille -> Toulouse | Lille -> Paris -> Lyon -> Clermont-Ferrand -> Toulouse | 1,155 km |
| Nantes -> Strasbourg | Nantes -> Paris -> Reims -> Strasbourg | 875 km |

---

## Delivery Prediction Engine

*Developed by Raed Meddeb*

While standard graph algorithms calculate minimum physical distance, real-world delivery schedules depend on external conditions. This module applies environmental multipliers to raw distances to estimate actual transit duration, operational cost, and delivery risk.

### Mathematical Formulation

Three environmental parameters (day of week, weather condition, and time period) combine multiplicatively into a single traffic factor relative to a 90 km/h base speed:

$$
\text{Traffic Factor} = f(\text{Day}) \times f(\text{Weather}) \times f(\text{Period})
$$

Travel time and fuel cost are updated using the resulting traffic factor:

$$
\text{Travel Time} = \frac{\text{Distance}}{90 \div \text{Traffic Factor}}
$$

$$
\text{Cost} = \text{Distance} \times 0.15\,\text{€/km} \times \text{Traffic Factor}
$$

$$
\text{Delay Risk} =
\begin{cases}
\text{LOW} & \text{Traffic Factor} < 1.20 \\
\text{MEDIUM} & 1.20 \le \text{Traffic Factor} \le 1.50 \\
\text{HIGH} & \text{Traffic Factor} > 1.50
\end{cases}
$$

### Traffic Multipliers

| Category | Parameter | Multiplier |
|---|---|---|
| **Day** | Friday | 1.45x |
| | Sunday | 0.85x |
| **Weather** | Snow | 1.65x |
| | Rain | 1.25x |
| | Clear | 1.00x |
| **Period** | Rush hour | 1.30x |
| | Holiday | 0.85x |
| | Normal | 1.00x |

### Scenario Predictions

| Route | Test Conditions | Traffic Factor | Time | Cost | Risk Level |
|---|---|---|---|---|---|
| Paris -> Marseille | Monday, Clear, Rush hour | 1.76x | 15h15 | 206 € | 🔴 HIGH |
| Paris -> Marseille | Sunday, Clear, Normal | 0.85x | 7h21 | 99 € | 🟢 LOW |
| Lille -> Toulouse | Friday, Rain, Rush hour | 2.36x | 30h17 | 409 € | 🔴 HIGH |
| Nantes -> Nice | Wednesday, Snow, Normal | 1.90x | 25h32 | 345 € | 🔴 HIGH |

---

## Installation & Usage

### 1. Clone the repository

```bash
git clone [https://github.com/RaedMeddeb/delivery-route-optimization.git](https://github.com/RaedMeddeb/delivery-route-optimization.git)
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
{'traffic_factor': 1.76, 'travel_time': '15h15', 'cost': '206 €', 'delay_risk': 'HIGH'}
```

---

## Repository Structure

```text
delivery-route-optimization/
├── DELIVERY route optimization code.ipynb  # Primary Jupyter notebook implementation
├── PROJECT+REPORT.pdf                      # Complete technical project report
├── cities.pdf                              # Node coordinate dataset
├── distance.pdf                            # Edge weight dataset
├── delivery.pdf                            # Delivery window constraints dataset
└── README.md                               # Repository documentation
```

---

## Team

Academic project developed for the Graph Theory course (Academic Year 2025-2026).

| Member | Module Contribution |
|---|---|
| **Raed Meddeb** | Delivery Prediction Engine (time, cost, and delay risk) |
| Sadok Tlili | Interactive Chatbot Interface |
| Ibrahim Grira | Driver Clustering (K-Means) |
| Ahmed Frouja | Dynamic Road Weight Modifier |
| Koussay Ibn Haj Kacem | Geographical Network Visualization |
