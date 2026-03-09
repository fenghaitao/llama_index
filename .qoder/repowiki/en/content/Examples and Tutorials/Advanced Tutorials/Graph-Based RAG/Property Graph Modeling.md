# Property Graph Modeling

<cite>
**Referenced Files in This Document**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py)
- [simple.py](file://llama-index-core/llama_index/core/graph_stores/simple.py)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py)
- [__init__.py](file://llama-index-core/llama_index/core/graph_stores/__init__.py)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py)
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py)
- [analytics_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/analytics_property_graph.py)
- [property_graph_basic.ipynb](file://docs/examples/property_graph/property_graph_basic.ipynb)
- [property_graph_advanced.ipynb](file://docs/examples/property_graph/property_graph_advanced.ipynb)
</cite>

## Table of Contents
1. [Introduction](#introduction)
2. [Project Structure](#project-structure)
3. [Core Components](#core-components)
4. [Architecture Overview](#architecture-overview)
5. [Detailed Component Analysis](#detailed-component-analysis)
6. [Dependency Analysis](#dependency-analysis)
7. [Performance Considerations](#performance-considerations)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [Conclusion](#conclusion)
10. [Appendices](#appendices)

## Introduction
This document explains property graph modeling in LlamaIndex, focusing on data structures, schema definition, graph traversal, Cypher-like queries, and integration with property graph databases such as Neo4j and Amazon Neptune. It covers how LlamaIndex constructs labeled property graphs from documents, how to query and filter nodes and edges, and how to evolve schemas over time. Guidance is also provided for optimization, indexing, visualization, and best practices.

## Project Structure
LlamaIndex organizes property graph capabilities across core graph store abstractions and multiple integrations:
- Core graph store interfaces and simple in-memory implementations
- Integrations with Neo4j and Amazon Neptune (both database-backed and analytics)
- Example notebooks demonstrating construction, querying, and visualization

```mermaid
graph TB
subgraph "Core"
T["types.py<br/>Interfaces & Models"]
S["simple.py<br/>SimpleGraphStore"]
SL["simple_labelled.py<br/>SimplePropertyGraphStore"]
end
subgraph "Integrations"
N4["neo4j_property_graph.py<br/>Neo4jPropertyGraphStore"]
NB["base_property_graph.py<br/>NeptuneBasePropertyGraph"]
NA["analytics_property_graph.py<br/>NeptuneAnalyticsPropertyGraphStore"]
end
T --> S
T --> SL
SL --> N4
SL --> NB
NB --> NA
```

**Diagram sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L215-L528)
- [simple.py](file://llama-index-core/llama_index/core/graph_stores/simple.py#L72-L187)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L20-L311)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L94-L800)
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py#L22-L387)
- [analytics_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/analytics_property_graph.py#L20-L220)

**Section sources**
- [__init__.py](file://llama-index-core/llama_index/core/graph_stores/__init__.py#L1-L22)
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L1-L528)
- [simple.py](file://llama-index-core/llama_index/core/graph_stores/simple.py#L1-L187)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L1-L311)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L1-L800)
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py#L1-L387)
- [analytics_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/analytics_property_graph.py#L1-L220)

## Core Components
- Property graph data structures
  - LabelledNode: base node with label, optional embedding, and arbitrary properties
  - EntityNode: labeled entity with name and properties
  - ChunkNode: labeled text chunk with text content and optional id
  - Relation: typed relationship with source/target ids and properties
  - LabelledPropertyGraph: in-memory graph holding nodes, relations, and triplets
- Graph store interfaces
  - GraphStore: generic triple-store interface for simple adjacency graphs
  - PropertyGraphStore: labeled property graph interface with get/get_triplets/get_rel_map/upsert/delete plus vector and structured query support
- Simple implementations
  - SimpleGraphStore: in-memory triple store persisted as JSON
  - SimplePropertyGraphStore: in-memory labeled property graph with filtering and persistence

Key capabilities:
- Node and edge property filtering
- Depth-aware relational map extraction
- Upsert and delete semantics for nodes and relations
- Persistence to JSON for simple stores
- Vector similarity queries against node embeddings (when supported)

**Section sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L36-L214)
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L215-L528)
- [simple.py](file://llama-index-core/llama_index/core/graph_stores/simple.py#L72-L187)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L20-L311)

## Architecture Overview
LlamaIndex’s property graph architecture separates concerns between:
- Construction: extractors produce labeled nodes and relations from documents
- Storage: PropertyGraphStore implementations persist and query graphs
- Retrieval: retrievers select seed nodes and traverse to collect relevant triplets
- Querying: PropertyGraphStore supports Cypher-like structured queries and vector similarity

```mermaid
sequenceDiagram
participant U as "User"
participant IDX as "PropertyGraphIndex"
participant EX as "KG Extractors"
participant GS as "PropertyGraphStore"
participant DB as "Graph DB (Neo4j/Neptune)"
U->>IDX : "from_documents(documents, ...)"
IDX->>EX : "extract triplets from chunks"
EX-->>IDX : "LabelledNode, Relation"
IDX->>GS : "upsert_nodes(nodes)"
IDX->>GS : "upsert_relations(relations)"
GS->>DB : "persist (when integrated)"
U->>IDX : "as_retriever()/as_query_engine()"
IDX->>GS : "get()/get_triplets()/get_rel_map()"
GS->>DB : "structured_query()/vector_query()"
DB-->>GS : "results"
GS-->>IDX : "triplets/nodes"
IDX-->>U : "retrieved nodes"
```

**Diagram sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L276-L528)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L133-L142)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L329-L408)
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py#L264-L359)

## Detailed Component Analysis

### Data Model and Schema Definition
- Nodes
  - LabelledNode: base with label, optional embedding, properties
  - EntityNode: labeled entity with name and properties
  - ChunkNode: labeled text chunk with text and optional id
- Edges
  - Relation: typed edge with source_id, target_id, label, properties
- Triplets
  - LabelledPropertyGraph maintains sets of triplets and ensures uniqueness

```mermaid
classDiagram
class LabelledNode {
+string label
+float[] embedding
+Dict properties
+id() str
}
class EntityNode {
+string name
+string label
+Dict properties
+id() str
}
class ChunkNode {
+string text
+string id_
+string label
+Dict properties
+id() str
}
class Relation {
+string label
+string source_id
+string target_id
+Dict properties
}
class LabelledPropertyGraph {
+Dict~str,LabelledNode~ nodes
+Dict~str,Relation~ relations
+Set triplets
+add_node(node)
+add_relation(relation)
+add_triplet(triplet)
+get_triplets() List
}
LabelledNode <|-- EntityNode
LabelledNode <|-- ChunkNode
LabelledPropertyGraph --> LabelledNode : "contains"
LabelledPropertyGraph --> Relation : "contains"
```

**Diagram sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L36-L214)
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L119-L214)

**Section sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L36-L214)

### Property Graph Store Interfaces
- GraphStore
  - Provides get, get_rel_map, upsert_triplet, delete, persist, get_schema, query
- PropertyGraphStore
  - Adds get, get_triplets, get_rel_map, upsert_nodes, upsert_relations, delete, structured_query, vector_query
  - Async variants included
  - Supports vector and structured query flags

```mermaid
classDiagram
class GraphStore {
+client
+get(subj) List
+get_rel_map(subjs,depth,limit) Dict
+upsert_triplet(subj,rel,obj)
+delete(subj,rel,obj)
+persist(path,fs)
+get_schema(refresh) str
+query(q,param_map) Any
}
class PropertyGraphStore {
+supports_structured_queries : bool
+supports_vector_queries : bool
+text_to_cypher_template
+get(properties,ids) LabelledNode[]
+get_triplets(entities,rels,props,ids) Triplet[]
+get_rel_map(nodes,depth,limit,ignore) Triplet[]
+upsert_nodes(nodes)
+upsert_relations(relations)
+delete(entities,rels,props,ids)
+structured_query(q,param_map) Any
+vector_query(query,**kwargs) (LabelledNode[], float[])
+persist(path,fs)
+get_schema(refresh) Any
}
GraphStore <|-- PropertyGraphStore
```

**Diagram sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L215-L528)

**Section sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L215-L528)

### Simple Implementations
- SimpleGraphStore
  - In-memory adjacency map persisted as JSON
  - get_rel_map supports depth-limited traversal with limit
- SimplePropertyGraphStore
  - In-memory LabelledPropertyGraph
  - Filtering by properties and ids for nodes/triplets
  - get_rel_map performs iterative depth expansion
  - save_networkx_graph and show_jupyter_graph helpers for visualization

```mermaid
flowchart TD
Start(["get_rel_map(graph_nodes, depth, limit)"]) --> Init["Init: triplets=[], cur_depth=0<br/>graph_triplets=get_triplets(ids)"]
Init --> Loop{"len(graph_triplets)>0 AND cur_depth<depth?"}
Loop --> |Yes| Append["Append current triplets"]
Append --> Expand["Expand to neighbors via get_triplets(entity_names)"]
Expand --> Dedup["Deduplicate triplets"]
Dedup --> IncDepth["cur_depth += 1"]
IncDepth --> Loop
Loop --> |No| Filter["Filter ignore_rels"]
Filter --> Slice["Return triplets[:limit]"]
Slice --> End(["Done"])
```

**Diagram sources**
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L103-L131)

**Section sources**
- [simple.py](file://llama-index-core/llama_index/core/graph_stores/simple.py#L72-L187)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L20-L311)

### Neo4j Integration
Neo4jPropertyGraphStore implements PropertyGraphStore with:
- Structured schema discovery and enhancement
- Upsert of nodes and relations with batching and chunking
- Cypher-based get, get_triplets, get_rel_map
- Vector similarity via vector index or fallback cosine similarity
- Constraints and indexes for performance

```mermaid
sequenceDiagram
participant APP as "Application"
participant PG as "Neo4jPropertyGraphStore"
participant DRV as "Neo4j Driver"
APP->>PG : "upsert_nodes(nodes)"
PG->>DRV : "MERGE/SET/CALL db.create.setNodeVectorProperty"
DRV-->>PG : "status"
APP->>PG : "get_rel_map(ids, depth, limit)"
PG->>DRV : "MATCH p=(e)-[r*1..depth]-()..."
DRV-->>PG : "triples"
PG-->>APP : "triplets"
```

**Diagram sources**
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L329-L408)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L543-L610)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L659-L720)

**Section sources**
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L94-L800)

### Amazon Neptune Integration
NeptuneBasePropertyGraph and NeptuneAnalyticsPropertyGraph provide:
- Base Cypher query support and schema introspection
- Upsert and deletion of nodes and relations
- Vector similarity via Neptune algorithms
- Schema caching and refresh

```mermaid
sequenceDiagram
participant APP as "Application"
participant NP as "NeptuneAnalyticsPropertyGraphStore"
participant CL as "Neptune Analytics Client"
APP->>NP : "vector_query(VectorStoreQuery)"
NP->>CL : "execute_query(language=OPEN_CYPHER)"
CL-->>NP : "results"
NP-->>APP : "nodes, scores"
```

**Diagram sources**
- [analytics_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/analytics_property_graph.py#L75-L128)

**Section sources**
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py#L22-L387)
- [analytics_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/analytics_property_graph.py#L20-L220)

### Construction, Querying, and Visualization
- Construction
  - PropertyGraphIndex.from_documents builds nodes and relations from documents
  - Extractors (e.g., SchemaLLMPathExtractor) can constrain schema
- Querying
  - Retrievers combine synonym expansion and vector context retrieval
  - Query engines assemble results from retrieved triplets
- Visualization
  - SimplePropertyGraphStore.save_networkx_graph writes HTML for inspection

**Section sources**
- [property_graph_basic.ipynb](file://docs/examples/property_graph/property_graph_basic.ipynb#L103-L192)
- [property_graph_advanced.ipynb](file://docs/examples/property_graph/property_graph_advanced.ipynb#L282-L293)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L262-L277)

## Dependency Analysis
- Core abstractions in types.py underpin all implementations
- Simple implementations are standalone and useful for development/testing
- Neo4j and Neptune integrations depend on their respective clients and expose Cypher-based APIs
- Retrievers and query engines consume PropertyGraphStore interfaces

```mermaid
graph LR
Types["types.py"] --> Simple["simple.py"]
Types --> SimpleLabelled["simple_labelled.py"]
Types --> Neo4j["neo4j_property_graph.py"]
Types --> NeptuneBase["base_property_graph.py"]
NeptuneBase --> NeptuneAnalytics["analytics_property_graph.py"]
```

**Diagram sources**
- [types.py](file://llama-index-core/llama_index/core/graph_stores/types.py#L1-L528)
- [simple.py](file://llama-index-core/llama_index/core/graph_stores/simple.py#L1-L187)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L1-L311)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L1-L800)
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py#L1-L387)
- [analytics_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/analytics_property_graph.py#L1-L220)

**Section sources**
- [__init__.py](file://llama-index-core/llama_index/core/graph_stores/__init__.py#L1-L22)

## Performance Considerations
- Indexing and constraints
  - Neo4jPropertyGraphStore creates unique constraints and vector indexes when supported
- Batching and chunking
  - Neo4j upserts are chunked to reduce transaction overhead
- Vector similarity
  - Prefer vector index when available; otherwise compute cosine similarity
- Traversal limits
  - get_rel_map enforces depth and limit to bound result sizes
- Schema caching
  - Neptune stores cache schema to avoid repeated expensive introspection

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- Neo4j errors
  - Structured queries handle specific Neo4j exceptions and fall back to session-based execution
- Missing client or schema support
  - SimplePropertyGraphStore raises NotImplementedError for unsupported operations
- Vector and structured query availability
  - Check supports_structured_queries and supports_vector_queries flags on stores
- Parameter sanitization
  - Neo4j store can sanitize query output values when enabled

**Section sources**
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L612-L657)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L233-L253)

## Conclusion
LlamaIndex provides a robust, extensible property graph framework:
- Clear data models for nodes, edges, and triplets
- Flexible storage backends with Cypher-like query support
- Practical construction and retrieval patterns
- Visualization aids for debugging
Adopting schema constraints, leveraging vector indexes, and carefully managing traversal depth yields scalable and efficient graph applications.

## Appendices

### Property Filtering and Graph Traversal Patterns
- Property filtering
  - Filter nodes by ids and properties; filter triplets by entity/relation names and properties
- Relational map extraction
  - Iterative expansion from seed nodes with configurable depth and limit
- Cypher-like queries
  - Neo4j and Neptune stores translate get/get_triplets/get_rel_map into Cypher statements

**Section sources**
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L40-L101)
- [neo4j_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neo4j/llama_index/graph_stores/neo4j/neo4j_property_graph.py#L409-L541)
- [base_property_graph.py](file://llama-index-integrations/graph_stores/llama-index-graph-stores-neptune/llama_index/graph_stores/neptune/base_property_graph.py#L108-L182)

### Schema Evolution and Best Practices
- Define explicit schemas during extraction to constrain entity and relation types
- Use vector indexes in Neo4j for efficient similarity search
- Persist and reload graph stores to disk for reuse
- Visualize graphs for debugging and validation

**Section sources**
- [property_graph_advanced.ipynb](file://docs/examples/property_graph/property_graph_advanced.ipynb#L128-L142)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L164-L196)
- [simple_labelled.py](file://llama-index-core/llama_index/core/graph_stores/simple_labelled.py#L262-L310)