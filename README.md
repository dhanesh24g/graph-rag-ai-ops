# GraphRAG for AIOps Incident Analysis

## 📌 Overview
This project demonstrates how to use **GraphRAG (Graph-based Retrieval Augmented Generation)** for **AIOps incident analysis**.  
Traditional RAG pipelines rely on vector similarity search over text chunks. While effective, they struggle with **complex dependencies** across microservices.  

GraphRAG enhances this by building a **knowledge graph** of services, infrastructure, and incidents. This allows the system to answer **multi-hop dependency questions**, trace **root causes**, and provide **explainable results**.  

---

## ⚡ Problem Statement
Modern systems process **tens of thousands of TPS (transactions per second)** across **hundreds of microservices**.  
When incidents happen (e.g., a network switch failure), the failure **cascades across dependent services**, making **root cause analysis (RCA)** difficult.  

This project simulates such an AIOps scenario with:
- Microservice failures due to **network, DB, and cache issues**  
- Metrics like **TPS, CPU, memory, DB latency, packet loss**  
- Dependency relationships between services  

---

## 🗂 Dataset
We use a synthetic **incident log dataset** (see `data/incident_logs.txt`) that includes:
- Alerts, warnings, errors, info logs  
- Metrics (TPS, CPU, memory, latency, IOPS, packet loss)  
- Service dependencies  

Example log:

```
[2025-08-30 02:36:42] ALERT: TransactionService reporting cascading failures due to PaymentService API timeouts.
Impact: 1,200 failed transactions in the last 2 minutes.
Metrics: TPS dropped from 9,800 → 1,050. Error rate = 72%. DB query latency = 5.6s.
```

---

## 🔗 Knowledge Graph Construction
We extract **triplets (subject, relation, object)** from logs using an LLM pipeline.  
Example:

```
(TransactionService, depends_on, PaymentService)
(AuthService, failure_reason, DB_CONN_TIMEOUT)
(Network_Switch_7, affects, Cluster_A)
```

These triplets are stored in a **graph database (Neo4j)**.  
The graph enables:
- Root cause tracing  
- Dependency traversal  
- Impact analysis  

---

## 🛠 Tech Stack
- **LangGraph** → workflow orchestration  
- **OpenAI / LLM** → entity + relation extraction from logs  
- **Neo4j** → graph database for triplets  
- **Pinecone / Qdrant** → vector DB for semantic search (log similarity)  
- **Python** (FastAPI optional for API)  

---

## 📊 Example Queries
1. **Impact Analysis**
   > “Which services were impacted by Network_Switch_7 failure at 02:35?”  
   → AuthService → PaymentService → TransactionService → NotificationService  

2. **Root Cause**
   > “What caused CheckoutService failures at 03:18?”  
   → PaymentService (AuthService timeouts) + ProductCatalogService (Cache misses)  

3. **Resource Bottlenecks**
   > “Which components exceeded CPU > 80% during incidents?”  
   → DatabaseCluster_Node3, Cluster_A, AuthService  

---

## 🚀 Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/<your-username>/graph-rag.git
cd graph-rag
```

### 2. Setup environment
```bash
python -m venv venv
source venv/bin/activate   # Mac/Linux
venv\Scripts\activate      # Windows

pip install -r requirements.txt
```

### 3. Start Neo4j (local or cloud)
```bash
docker run -d --name neo4j   -p7474:7474 -p7687:7687   -e NEO4J_AUTH=neo4j/test neo4j:5.11
```

### 4. Run pipeline
```bash
python ingest.py   # Parse logs & populate graph
python query.py    # Run sample GraphRAG queries
```

---

## 📂 Project Structure
```
GraphRAG-AIOps/
│── data/
│   └── incident_logs.txt     # Synthetic dataset
│── src/
│   ├── ingest.py             # Parse logs, extract triplets
│   ├── graph_store.py        # Neo4j integration
│   ├── retriever.py          # Hybrid (graph + vector) retriever
│   └── query.py              # Example queries
│── README.md
│── requirements.txt
```

---

## 🎯 Roadmap
- [ ] Expand dataset with more incident scenarios  
- [ ] Add FastAPI endpoint for live querying  
- [ ] Add RAG evaluation with **RAGAs / DeepEval**  
- [ ] Integrate real logs from open-source datasets  

---

## 🤝 Contributing
Pull requests welcome! If you’d like to extend the dataset, add more queries, or improve the pipeline, please open an issue.  

---

## 📜 License
MIT License.  
