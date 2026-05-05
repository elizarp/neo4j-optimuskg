# OptimusKG — Biomedical Knowledge Graph · Neo4j Ingestion

This project demonstrates how to load the **[OptimusKG](https://optimuskg.ai/)** multimodal
biomedical knowledge graph into [Neo4j](https://neo4j.com), enabling graph-native analysis
across genes, diseases, drugs, pathways, phenotypes, and more.

By representing biomedical knowledge as a graph, researchers and engineers can move beyond
flat table joins to discover complex multi-hop associations — such as which genes are involved
in a disease pathway, which drugs target those genes, and which phenotypes they share.

> **Data source:** Vittor, L. et al. *OptimusKG: Unifying biomedical knowledge in a modern
> multimodal graph.* In Review (2026). — [optimuskg.ai](https://optimuskg.ai/)

---

## Notebooks

| Notebook | Description |
|---|---|
| [`neo4j_optimuskg.ipynb`](neo4j_optimuskg.ipynb) | Full ingestion pipeline: load OptimusKG, create constraints, and ingest all nodes and edges into Neo4j |

---

## Prerequisites

### 1. Neo4j Desktop

Download and install **Neo4j Desktop** to run a local Neo4j instance:

**[https://neo4j.com/download/](https://neo4j.com/download/)**

After installing:

1. Open Neo4j Desktop and click **New** → **Create project**
2. Inside the project, click **Add** → **Local DBMS**
3. Set a name and password (e.g. `optimuskg@123456`), select version **5.x**, then click **Create**
4. Click **Start** to launch the database
5. The default connection URI is `neo4j://127.0.0.1:7687`

### 2. Python packages

| Requirement | Version |
|---|---|
| Python | 3.10+ |
| `neo4j-rust-ext` | ≥ 6.1 |
| `optimuskg` | ≥ 1.0 |

```bash
pip install neo4j-rust-ext optimuskg
```

---

## Quick Start

1. **Start your Neo4j Desktop database** (see above)
2. **Set your credentials** in the notebook — update `NEO4J_URI`, `NEO4J_USERNAME`, and `NEO4J_PASSWORD` in the configuration cell
3. **Run all cells** — the notebook handles constraint creation, node ingestion, and edge ingestion end-to-end

---

## Graph Schema

OptimusKG integrates ten biomedical entity types sourced from curated public databases
(OMIM, DrugBank, Reactome, Uberon, Gene Ontology, and others).

### Nodes — ~190,500 entities

| Label | Short Code | Description | Example sources |
|---|---|---|---|
| `Gene` | `GEN` | Protein-coding and non-coding genes | Ensembl, NCBI Gene |
| `Disease` | `DIS` | Human diseases and disorders | OMIM, Disease Ontology |
| `Drug` | `DRG` | Approved and experimental compounds | DrugBank, ChEMBL |
| `Phenotype` | `PHE` | Clinical and molecular phenotypes | HPO |
| `Pathway` | `PWY` | Biological pathways | Reactome |
| `BiologicalProcess` | `BPO` | GO biological process terms | Gene Ontology |
| `MolecularFunction` | `MFN` | GO molecular function terms | Gene Ontology |
| `CellularComponent` | `CCO` | GO cellular component terms | Gene Ontology |
| `Anatomy` | `ANA` | Anatomical structures | Uberon |
| `Exposure` | `EXP` | Environmental and chemical exposures | CTD |

### Edges — ~21.8 million relationships

Relationships are fully typed and connect entities across all node types. Each edge carries
a `properties` field with source provenance indicating which database(s) support that association.

Edge labels follow the `SRC-TGT` convention (e.g., `GEN-DIS` for Gene → Disease), which the
ingestion pipeline uses to resolve source and target node types dynamically.

---

## Ingestion Architecture

```
optimuskg Python package
        │
        ▼
  Public Data Lake (Parquet)
  ┌─────────────┬──────────────┐
  │  nodes.pq   │  edges.pq    │
  └─────────────┴──────────────┘
        │
        ▼
  Polars DataFrames (in-memory)
        │
        ▼
  Neo4j Python Driver
  ┌────────────────────────────────┐
  │  chunked CREATE/MERGE          │  ← memory-safe, concurrent
  │  ThreadPoolExecutor workers    │
  │  TransientError retry logic    │
  └────────────────────────────────┘
        │
        ▼
  Neo4j local instance (Desktop)
```

The pipeline processes nodes label-by-label and edges grouped by
`(source type, target type, relationship type)` to keep transaction size
predictable and avoid heap exhaustion on the ~21 M edge set.

---

## Why Neo4j for Biomedical Knowledge Graphs?

Biomedical knowledge is inherently relational. A flat table can store that
*Drug X targets Gene Y*, but a graph can answer:

- Which drugs target genes associated with Disease Z?
- What is the shortest path between a drug compound and a phenotype, through pathways?
- Which diseases share the most genetic overlap with a given condition?

Neo4j's native graph storage and Cypher query language make these multi-hop traversals
fast and expressive — without the complexity of multiple joins or graph-specific ETL.

---

## Dashboard

A pre-built **dashboard** is included at [`OptimusKG_Dashboard.json`](OptimusKG_Dashboard.json).

### Importing the dashboard

1. Open **Dashboard** menu and connect to your running database
2. On the home screen, click **Import**
3. Select **From File** and open `OptimusKG_Dashboard.json`
4. The OptimusKG dashboard will load with all pre-configured visualizations
---

## Data Source & Citation

OptimusKG is published at **[https://optimuskg.ai/](https://optimuskg.ai/)**.

```
Vittor, L. et al. OptimusKG: Unifying biomedical knowledge in a modern multimodal graph.
In Review (2026).
```

The `optimuskg` Python package provides programmatic access to the gold-layer Parquet files.
No manual data download is required.
