#  Delivery Route Optimization — France Network

> Graph-based route optimization system for an e-commerce delivery network across 20 French cities, built with NetworkX. Includes a custom **Delivery Prediction Engine** that turns a raw shortest path into a realistic time, cost, and risk estimate.

<p>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/NetworkX-1C1C1C?style=for-the-badge&logo=graph&logoColor=white"/>
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white"/>
  <img src="https://img.shields.io/badge/Matplotlib-11557C?style=for-the-badge&logo=plotly&logoColor=white"/>
</p>

---

##  Objective

Help an e-commerce company operating across France answer a simple operational question: **what is the fastest, cheapest, and most reliable way to get a delivery from A to B?** The system models the road network as a weighted graph, applies classic graph algorithms to find and traverse optimal routes, and layers a predictive model on top so a dispatcher can see not just distance, but realistic time, cost, and delay risk before committing to a route.

---

##  Graph Model

- **20 nodes** (French cities), **30 edges** (road connections) after deduplication
- Undirected, weighted graph built with `networkx.Graph()`
- Edge weight = road distance in kilometres
- Node attributes store latitude/longitude for geographic visualization

```python
import networkx as nx

G = nx.Graph()
G.add_node(city, lat=latitude, lon=longitude)
G.add_edge(source, destination, weight=distance_km)
```

---

##  Algorithms

| Algorithm | Purpose | Notes |
|---|---|---|
| **Dijkstra** | Shortest path between two cities | Guaranteed optimal for positive edge weights; visits the nearest unvisited node at each step |
| **BFS** | Layer-by-layer network traversal | Explores all direct neighbors before moving further out |
| **DFS** | Depth-first traversal | Follows one branch as far as possible before backtracking; useful for connectivity checks |
| **Greedy Nearest-Neighbor** | Multi-stop delivery ordering | Always visits the closest undelivered city next; fast and practical, not always globally optimal |

**Example — Dijkstra shortest paths:**

| Route | Optimal Path | Distance |
|---|---|---|
| Paris → Marseille | Paris → Lyon → Marseille | 780 km |
| Paris → Nice | Paris → Lyon → Marseille → Toulon → Nice | 995 km |
| Lille → Toulouse | Lille → Paris → Lyon → Clermont-Ferrand → Toulouse | 1,155 km |
| Nantes → Strasbourg | Nantes → Paris → Reims → Strasbourg | 875 km |

---

##  Signature Feature: Delivery Prediction Engine

*Developed by Raed Meddeb.*

Dijkstra tells you the shortest **distance** — it doesn't tell a dispatcher how long a delivery will actually take today, what it will cost, or whether it's at risk of running late. This module answers all three by layering real-world conditions on top of the graph's raw distances.

### The Model

Three independent conditions — **day of week**, **weather**, and **time period** — each carry a multiplier relative to a 90 km/h baseline speed. The three combine multiplicatively into a single traffic factor:

$$
\text{traffic\_factor} = f(\text{day}) \times f(\text{weather}) \times f(\text{period})
$$

That factor is then applied to both travel time and cost:

$$
\text{travel\_time} = \frac{\text{distance}}{90 \div \text{traffic\_factor}}
$$

$$
\text{cost} = \text{distance} \times 0.15\,\text{€/km} \times \text{traffic\_factor}
$$

$$
\text{delay\_risk} =
\begin{cases}
\text{LOW} & \text{traffic\_factor} < 1.20 \\
\text{MEDIUM} & 1.20 \leq \text{traffic\_factor} \leq 1.50 \\
\text{HIGH} & \text{traffic\_factor} > 1.50
\end{cases}
$$

### Multiplier Table

| Condition | Value | Multiplier |
|---|---|---|
| **Day** | Friday | 1.45× |
| | Sunday | 0.85× |
| **Weather** | Snow | 1.65× |
| | Rain | 1.25× |
| | Clear | 1.00× |
| **Period** | Rush hour | 1.30× |
| | Holiday | 0.85× |
| | Normal | 1.00× |

### Why it matters

The same route can swing dramatically depending on conditions  the gap between best- and worst-case isn't cosmetic, it's the difference between a route that's safe to promise a customer and one that isn't.

| Route | Conditions | Factor | Time | Cost | Risk |
|---|---|---|---|---|---|
| Paris → Marseille | Monday, clear, rush hour | 1.76× | 15h15 | 206 € | 🔴 HIGH |
| Paris → Marseille | Sunday, clear, normal | 0.85× | 7h21 | 99 € | 🟢 LOW |
| Lille → Toulouse | Friday, rain, rush hour | 2.36× | 30h17 | 409 € | 🔴 HIGH |
| Nantes → Nice | Wednesday, snow, normal | 1.90× | 25h32 | 345 € | 🔴 HIGH |

That **107 € gap** between the best- and worst-case Paris–Marseille run is a concrete, actionable signal for when it's worth rescheduling a delivery rather than dispatching it into rush hour.

---

##  Installation & Usage

### 1. Clone the repository

```bash
git clone https://github.com/RaedMeddeb/delivery-route-optimization.git
cd delivery-route-optimization
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv
source venv/bin/activate   # on Windows: venv\Scripts\activate
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Run the main script

```bash
python src/main.py
```

### 5. (Optional) Run the interactive chatbot interface

```bash
python src/chatbot.py
```

---

##  Example Usage

```python
from src.graph_model import build_graph
from src.routing import dijkstra_shortest_path
from src.prediction import predict_delivery

G = build_graph("data/cities.csv", "data/distance.csv")

# Shortest path
path, distance = dijkstra_shortest_path(G, "Paris", "Marseille")
print(f"Path: {' -> '.join(path)} | Distance: {distance} km")

# Delivery prediction under specific conditions
result = predict_delivery(
    distance_km=distance,
    day="Monday",
    weather="Clear",
    period="Rush hour"
)
print(result)
```

**Output:**

```
Path: Paris -> Lyon -> Marseille | Distance: 780 km
{'traffic_factor': 1.76, 'travel_time': '15h15', 'cost': '206 €', 'delay_risk': 'HIGH'}
```

---

##  requirements.txt

```txt
networkx>=3.0
pandas>=2.0
numpy>=1.24
matplotlib>=3.7
```

---

##  Project Structure

```
delivery-route-optimization/
├── data/
│   ├── cities.csv              # 20 cities: name, latitude, longitude
│   ├── distance.csv            # 31 road connections: source, destination, distance_km
│   └── delivery.csv            # delivery constraints: city, priority, time window
├── src/
│   ├── graph_model.py          # graph construction (NetworkX)
│   ├── routing.py              # Dijkstra, BFS, DFS implementations
│   ├── greedy_delivery.py      # nearest-neighbor delivery ordering
│   ├── prediction.py           # Delivery Prediction Engine (traffic multipliers)
│   ├── road_weight_modifier.py # dynamic edge weight updates (closures, detours)
│   ├── visualization.py        # geographic network + route-highlight maps
│   ├── chatbot.py              # text-command interface
│   └── main.py                 # entry point
├── tests/
│   ├── test_routing.py
│   ├── test_prediction.py
│   └── test_graph_model.py
├── docs/
│   └── project_report.pdf
├── requirements.txt
└── README.md
```

---

##  Team

Built for the Graph Theory course — Academic Year 2025–2026.

| Member | Signature Feature |
|---|---|
| **Raed Meddeb** | Delivery Prediction Module (time, cost, delay risk) |
| Sadok Tlili | Interactive Chatbot |
| Ibrahim Grira | K-Means Driver Assignment |
| Ahmed Frouja | Dynamic Road Weight Modifier |
| Koussay Ibn Haj Kacem | Geographic Network Visualization |

The core graph model (Dijkstra, BFS, DFS, greedy delivery ordering) was built collaboratively by the team; each member then extended it with an individual signature module.

---

##  Possible Next Steps

- Replace the greedy delivery optimizer with a full Travelling Salesman Problem (TSP) solver
- Integrate a live traffic API for real-time edge weights instead of static multipliers
- Build a web interface around the chatbot for non-technical users
