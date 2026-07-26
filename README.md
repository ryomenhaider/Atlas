Atlas — Knowledge Graph & Relationship Intelligence Platform

A standalone knowledge graph platform for geopolitical, economic, financial, and intelligence relationship analysis. Part of the AEGIS · HERMES · ATLAS ecosystem.

---

Overview

Atlas is a comprehensive knowledge graph platform that transforms raw data into structured, queryable intelligence. It tracks entities (countries, organizations, people, commodities, events, treaties, locations) and their temporal relationships across global systems.

Key Capabilities:

· Graph-based entity and relationship storage (Neo4j)
· Temporal queries with valid-time tracking
· Network analysis and community detection
· Graph neural networks for link prediction
· Entity resolution and canonicalization
· Interactive web-based graph visualization
· Python SDK for programmatic graph operations

---

Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                         USER LAYER                          │
│  ┌──────────────────┐    ┌──────────────────────────────┐  │
│  │   Atlas WebApp   │    │       Atlas SDK              │  │
│  │  (React + D3)    │    │   pip install atlas          │  │
│  └────────┬─────────┘    └──────────┬───────────────────┘  │
└───────────┼─────────────────────────┼──────────────────────┘
            │                         │
┌───────────┼─────────────────────────┼──────────────────────┐
│           │     PLATFORM LAYER      │                      │
│  ┌────────▼─────────────────────────▼───────────────────┐  │
│  │                    Atlas Backend                      │  │
│  │   ┌────────────┐  ┌────────────┐  ┌────────────┐   │  │
│  │   │ Graph API  │  │ Analytics  │  │ Temporal   │   │  │
│  │   │ (Cypher)   │  │ Engine     │  │ Queries    │   │  │
│  │   └────────────┘  └────────────┘  └────────────┘   │  │
│  └─────────────────────────┬────────────────────────────┘  │
│                            │                               │
│  ┌─────────────────────────▼────────────────────────────┐  │
│  │                 Graph Database                       │  │
│  │   ┌────────────┐  ┌────────────┐  ┌────────────┐   │  │
│  │   │  Neo4j     │  │ pgvector   │  │ Embedding  │   │  │
│  │   │ (Primary)  │  │ (Vector)   │  │ Store      │   │  │
│  │   └────────────┘  └────────────┘  └────────────┘   │  │
│  └─────────────────────────┬────────────────────────────┘  │
│                            │                               │
│  ┌─────────────────────────▼────────────────────────────┐  │
│  │              Hermes SDK (Data Ingestion)            │  │
│  │          from hermes import Hermes                   │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

---

Quick Start

Installation

```bash
# Install with core functionality
pip install atlas

# Install with GNN support
pip install "atlas[gnn]"

# Install with all features
pip install "atlas[all]"
```

Basic Usage (SDK)

```python
from atlas import Atlas
from hermes import Hermes

# Initialize Atlas
atlas = Atlas(graph_uri="bolt://localhost:7687")
hr = Hermes()

# Ingest data from Hermes
trade_data = hr.un_comtrade.get_bilateral_trade(
    origin='CHL', 
    destination='CHN'
)
atlas.ingest_trade_data(trade_data)

# Query the graph
results = atlas.query("""
    MATCH (c:Country {name: 'China'})-[r:TRADES_WITH]->(other)
    RETURN other.name, other.risk_score
    ORDER BY r.volume DESC
""")

# Run network analysis
communities = atlas.detect_communities(algorithm='louvain')
influence = atlas.calculate_influence(node='China', method='pagerank')
predicted_links = atlas.predict_links(node='Ukraine', top_k=10)

# Temporal queries
past_state = atlas.at_time('2022-02-01')
relations = past_state.query("""
    MATCH (p:Person)-[:CONTROLS]->(o:Organization)
    RETURN p.name, o.name
""")

# Entity resolution
atlas.resolve_entities(
    entity_type='Person',
    matches=[
        {'name': 'Vladimir Putin', 'source': 'newsapi'},
        {'name': 'V. Putin', 'source': 'gdelt'},
        {'name': 'Putin, Vladimir', 'source': 'wikidata'}
    ]
)
```

---

Entity & Relationship Model

Entity Types

Entity Properties Example Source
Country ISO code, region, GDP, population, risk score Ukraine, China, USA Hermes WB/IMF
Organization Type, sector, founding date, status Rosatom, BlackRock Hermes NewsAPI
Person Role, nationality, sanctions status Vladimir Putin, Elon Musk Hermes NewsAPI
Commodity Type, price series, reserves, criticality Lithium, Uranium Hermes USGS/EIA
Event Type, date, location, severity Invasion, Election Hermes GDELT
Treaty Type, signatories, status, clauses NATO, JCPOA Hermes Wikidata
Location Type, coordinates, jurisdiction Salar de Atacama Hermes OSM

Relationship Types

Relationship From → To Example
TRADES_WITH Country → Country China TRADES_WITH Chile
EXPORTS Country → Commodity Chile EXPORTS Lithium
OWNS Organization → Organization Albemarle OWNS Mine
CONTROLS Person → Organization Person CONTROLS Rosatom
PARTICIPATED_IN Country → Event Russia PARTICIPATED_IN Invasion
SIGNATORY_TO Country → Treaty USA SIGNATORY_TO NATO
ALLIED_WITH Country → Country USA ALLIED_WITH UK
HOSTILE_TO Country → Country Russia HOSTILE_TO Ukraine
LOCATED_IN Location → Country Salar LOCATED_IN Chile
SUPPLIES Location → Organization Mine SUPPLIES BatteryMaker
SANCTIONS Country → Country/Org/Person USA SANCTIONS Person
INFLUENCES Person/Org → Person/Org/Country LobbyGroup INFLUENCES Policy

Temporal Properties

Every relationship includes temporal metadata:

· valid_from: Date relationship began (required)
· valid_to: Date relationship ended (null = ongoing)
· confidence: 0.0-1.0 based on source quality
· source: Origin of relationship
· evidence: Supporting document IDs or URLs

---

Features

1. Graph Exploration

Interactive web-based graph visualization with:

· Pan, zoom, and click interactions
· Expand neighbors from any node
· Filter by relationship type and date
· Search by fuzzy text matching
· Auto-layout algorithms (force-directed, hierarchical)

2. Network Analysis

Algorithm Purpose Implementation
PageRank Node influence measurement Neo4j GDS
Betweenness Centrality Bridge/connector identification Neo4j GDS
Community Detection Cluster discovery (Louvain, Leiden) Neo4j GDS
Shortest Path Connection finding Neo4j GDS
All-Pairs Shortest Path Graph diameter and distances Neo4j GDS

3. Temporal Intelligence

· Time-travel queries: View graph state at any historical date
· Trend analysis: Track relationship evolution over time
· Event impact: Measure effects of events on graph structure
· Forecasting: Predict future relationship emergence

4. Graph Neural Networks

Built with PyTorch Geometric and DGL:

Task Method Output
Link Prediction GCN/GAT + bilinear decoder New relationship probabilities
Node Classification GraphSAGE + MLP Entity type/category prediction
Graph Embeddings Node2Vec, GCN, GraphSAGE 128-256 dim embeddings
Similarity Search Cosine similarity on embeddings Nearest neighbors

5. Entity Resolution

· Record linkage: Blocking + similarity scoring
· Canonicalization: Merge duplicate entities
· Entity linking: Connect to external knowledge bases
· Confidence scoring: Quality metrics for matches

6. WebApp Interface

View Purpose
Graph Explorer Visual network navigation
Entity Search Find entities with faceted filters
Path Finder Discover connections between nodes
Temporal Timeline Slider-based history exploration
Community Detection Cluster visualization and stats
Influence Analysis Top-ranked nodes and metrics
Embedding Explorer 2D/3D projection + similarity search

---

Data Ingestion

Atlas ingests data exclusively through the Hermes SDK:

```python
from atlas import Atlas
from hermes import Hermes

atlas = Atlas()
hr = Hermes()

# Economic data
gdp_data = hr.world_bank.get_indicator('NY.GDP.MKTP.CD')
atlas.ingest_economic_data(gdp_data)

# Trade data
trade_data = hr.un_comtrade.get_bilateral_trade()
atlas.ingest_trade_data(trade_data)

# Conflict data
conflict_data = hr.gdelt.query_events(countries=['UKR', 'RUS'])
atlas.ingest_conflict_events(conflict_data)

# News data
news_data = hr.newsapi.get_headlines(country='us')
atlas.ingest_news_data(news_data)

# Extract entities, relationships, and create graph
atlas.build_graph()
```

---

Development

Project Structure

```
atlas/
├── atlas/
│   ├── __init__.py
│   ├── core/
│   │   ├── graph.py          # Neo4j connection and queries
│   │   ├── entities.py       # Entity CRUD operations
│   │   ├── relationships.py  # Relationship management
│   │   └── temporal.py       # Temporal queries
│   ├── analytics/
│   │   ├── network.py        # Network analysis
│   │   ├── communities.py    # Community detection
│   │   └── influence.py      # Influence measurement
│   ├── ml/
│   │   ├── embeddings.py     # Graph embeddings
│   │   ├── link_prediction.py # GNN link prediction
│   │   └── node_classification.py
│   ├── ingestion/
│   │   ├── trade.py          # Trade data ingestion
│   │   ├── economic.py       # Economic data ingestion
│   │   └── conflict.py       # Conflict data ingestion
│   ├── resolution/
│   │   ├── dedupe.py         # Entity deduplication
│   │   ├── canonicalize.py   # Canonicalization
│   │   └── linking.py        # Entity linking
│   └── webapp/
│       ├── backend/
│       │   └── app.py        # FastAPI application
│       └── frontend/
│           └── src/          # React application
├── tests/
├── examples/
├── docker-compose.yml
├── pyproject.toml
└── README.md
```

Local Development Setup

```bash
# Clone repository
git clone https://github.com/your-org/atlas.git
cd atlas

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -e ".[dev,all]"

# Start Neo4j with Docker
docker run -d \
  --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/password \
  neo4j:latest

# Run tests
pytest

# Start development web server
uvicorn atlas.webapp.backend.app:app --reload

# Start frontend
cd atlas/webapp/frontend
npm install
npm start
```

Build Timeline

Atlas is built in parallel with Aegis, starting at Week 13:

Phase Weeks Module Key Learnings
4.1 W13-16 Core SDK Neo4j, Cypher, graph modeling
4.2 W17-18 Network Analysis NetworkX, graph algorithms
4.3 W19-20 Temporal Layer Temporal databases, bitemporal design
4.4 W21-22 GNN + Embeddings PyTorch Geometric, DGL
4.5 W23-24 WebApp D3.js, Cytoscape.js
4.6 W25-26 Entity Resolution Record linking, fuzzy matching
4.7 W27-28 Production K8s, monitoring, scaling

---

Technology Stack

Layer Technology Purpose
Graph DB Neo4j 5 Primary graph storage
Vector Store pgvector Graph embeddings
Backend FastAPI, Python 3.11+ REST API
Frontend React, TypeScript, D3.js Graph visualization
Graph Analytics NetworkX, igraph Network algorithms
GNN PyTorch Geometric, DGL Deep learning on graphs
Orchestration Docker, Kubernetes Deployment
Monitoring Prometheus, Grafana Metrics

---

Integration with Ecosystem

Hermes → Atlas

Atlas ingests raw data from Hermes connectors:

· Economic indicators (World Bank, IMF, FRED)
· Trade flows (UN Comtrade)
· Conflict events (GDELT, UCDP)
· News and media (NewsAPI)
· Entity data (Wikidata)

Aegis ↔ Atlas (Future)

· Aegis queries Atlas for relationship context
· Atlas consumes Aegis risk scores as node properties
· Example: "Show all subsidiaries of sanctioned entity X with exposure to Y"

---

API Reference

Core SDK

```python
class Atlas:
    def __init__(self, graph_uri: str, vector_uri: Optional[str] = None):
        """Initialize Atlas with graph and optional vector store."""
    
    def query(self, cypher_query: str) -> List[Dict]:
        """Execute a Cypher query and return results."""
    
    def at_time(self, date: str) -> 'Atlas':
        """Return graph state at a specific date."""
    
    def ingest_economic_data(self, data: pd.DataFrame) -> None:
        """Ingest economic indicators from Hermes."""
    
    def ingest_trade_data(self, data: pd.DataFrame) -> None:
        """Ingest trade flow data from Hermes."""
    
    def ingest_conflict_events(self, data: pd.DataFrame) -> None:
        """Ingest conflict events from Hermes."""
    
    def build_graph(self) -> None:
        """Build graph from ingested data."""
```

Analytics API

```python
class GraphAnalytics:
    def detect_communities(self, algorithm: str = 'louvain') -> Dict:
        """Detect communities in the graph."""
    
    def calculate_influence(self, node: str, method: str = 'pagerank') -> float:
        """Calculate node influence score."""
    
    def shortest_path(self, source: str, target: str) -> List[str]:
        """Find shortest path between nodes."""
    
    def node_centrality(self, method: str = 'betweenness') -> Dict:
        """Calculate centrality scores for all nodes."""
```

GNN API

```python
class GraphLearning:
    def train_link_prediction(self, data: GraphData) -> Model:
        """Train link prediction model."""
    
    def predict_links(self, node: str, top_k: int = 10) -> List[Tuple[str, float]]:
        """Predict missing connections for a node."""
    
    def get_embeddings(self, nodes: Optional[List[str]] = None) -> np.ndarray:
        """Get graph embeddings for nodes."""
    
    def find_similar(self, node: str, k: int = 10) -> List[Tuple[str, float]]:
        """Find similar nodes by embedding distance."""
```

---

License

Copyright © 2026 Finance-Intelligence-Defense

This project is part of the AEGIS · HERMES · ATLAS ecosystem. All rights reserved.

---

Contributors

· Architecture: Haider Ali 
· Graph Engineering: Haider Ali 
· Frontend: Haider Ali 

---

Document Version 1.0 — July 2026
