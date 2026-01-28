# Custom Component Development

<cite>
**Referenced Files in This Document**
- [llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py)
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py)
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py)
- [llama_index/core/base/base_auto_retriever.py](file://llama-index-core/llama_index/core/base/base_auto_retriever.py)
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py)
- [llama_index/core/schema.py](file://llama-index-core/llama_index/core/schema.py)
- [llama_index/core/readers/__init__.py](file://llama-index-core/llama_index/core/readers/__init__.py)
- [llama_index/core/postprocessor/types.py](file://llama-index-core/llama_index/core/postprocessor/types.py)
- [llama_index/core/data_structs/registry.py](file://llama-index-core/llama_index/core/data_structs/registry.py)
- [llama_index/core/service_context.py](file://llama-index-core/llama_index/core/service_context.py)
- [llama-index-core/llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py)
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
This document explains how to develop custom components in LlamaIndex. It focuses on the base classes and interfaces that enable extension across readers, indices, retrievers, query engines, selectors, and post-processors. It also covers component registration, configuration patterns, integration with existing systems, plugin architecture, factory patterns, dependency injection, testing strategies, and best practices for maintainable extensions.

## Project Structure
LlamaIndex organizes core abstractions under a central namespace. Key areas for extensibility include:
- Readers: Data ingestion via BaseReader and related utilities
- Indices: Index types and loading utilities
- Retrievers and Query Engines: Retrieval and querying abstractions
- Post-processors: Node ranking and filtering
- Selectors: Choice selection for agents and workflows
- Schema and Registry: Shared data models and component registration

```mermaid
graph TB
subgraph "Core Abstractions"
SCHEMA["schema.py<br/>BaseComponent, Node types"]
BASE_RETRIEVER["base_retriever.py<br/>BaseRetriever"]
BASE_QUERY_ENGINE["base_query_engine.py<br/>BaseQueryEngine"]
BASE_AUTO_RETRIEVER["base_auto_retriever.py<br/>BaseAutoRetriever"]
BASE_SELECTOR["base_selector.py<br/>BaseSelector"]
READERS["readers/__init__.py<br/>Reader utilities"]
POSTPROCESSOR["postprocessor/types.py<br/>BaseNodePostprocessor"]
end
subgraph "Integration"
INIT_CORE["core/__init__.py<br/>Public exports"]
REGISTRY["data_structs/registry.py<br/>Component registry"]
SERVICE_CONTEXT["service_context.py<br/>Global context"]
end
SCHEMA --> BASE_RETRIEVER
SCHEMA --> BASE_QUERY_ENGINE
SCHEMA --> BASE_AUTO_RETRIEVER
SCHEMA --> BASE_SELECTOR
SCHEMA --> POSTPROCESSOR
READERS --> SCHEMA
INIT_CORE --> SCHEMA
INIT_CORE --> READERS
INIT_CORE --> BASE_RETRIEVER
INIT_CORE --> BASE_QUERY_ENGINE
INIT_CORE --> BASE_AUTO_RETRIEVER
INIT_CORE --> BASE_SELECTOR
INIT_CORE --> POSTPROCESSOR
REGISTRY --> INIT_CORE
SERVICE_CONTEXT --> INIT_CORE
```

**Diagram sources**
- [llama_index/core/schema.py](file://llama-index-core/llama_index/core/schema.py#L80-L188)
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py#L34-L275)
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py#L22-L94)
- [llama_index/core/base/base_auto_retriever.py](file://llama-index-core/llama_index/core/base/base_auto_retriever.py#L9-L44)
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py#L72-L104)
- [llama_index/core/readers/__init__.py](file://llama-index-core/llama_index/core/readers/__init__.py#L1-L33)
- [llama_index/core/postprocessor/types.py](file://llama-index-core/llama_index/core/postprocessor/types.py#L12-L80)
- [llama-index-core/llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py#L1-L162)
- [llama-index-core/llama_index/core/data_structs/registry.py](file://llama-index-core/llama_index/core/data_structs/registry.py)
- [llama-index-core/llama_index/core/service_context.py](file://llama-index-core/llama_index/core/service_context.py)

**Section sources**
- [llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py#L1-L162)

## Core Components
This section outlines the foundational base classes and shared patterns for building custom components.

- BaseComponent and Node Types
  - BaseComponent provides serialization hooks and a class_name mechanism used by the registry and persistence.
  - TransformComponent defines the callable interface for node transformations.
  - Node types (BaseNode, TextNode, Node) define the canonical data model for documents and nodes.

- Retrieval and Query Engines
  - BaseRetriever defines synchronous and asynchronous retrieval APIs, recursive retrieval handling, and instrumentation hooks.
  - BaseQueryEngine defines query and async query entry points, with optional retrieve and synthesize capabilities.
  - BaseAutoRetriever extends BaseRetriever to generate retrieval specs and build dynamic retrievers per query.

- Selectors
  - BaseSelector defines selection APIs for choosing among choices, wrapping inputs and dispatching to abstract selection methods.

- Readers and Post-processors
  - Readers expose public entry points via readers/__init__.py.
  - Post-processors extend BaseNodePostprocessor with synchronous and asynchronous node postprocessing.

**Section sources**
- [llama_index/core/schema.py](file://llama-index-core/llama_index/core/schema.py#L80-L188)
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py#L34-L275)
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py#L22-L94)
- [llama_index/core/base/base_auto_retriever.py](file://llama-index-core/llama_index/core/base/base_auto_retriever.py#L9-L44)
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py#L72-L104)
- [llama_index/core/readers/__init__.py](file://llama-index-core/llama_index/core/readers/__init__.py#L1-L33)
- [llama_index/core/postprocessor/types.py](file://llama-index-core/llama_index/core/postprocessor/types.py#L12-L80)

## Architecture Overview
The architecture centers around a small set of base classes and mixins that standardize:
- Instrumentation and tracing via DispatcherSpanMixin
- Callback management via CallbackManager
- Prompt mixin integration for configurable prompts
- Serialization and deserialization through BaseComponent

```mermaid
classDiagram
class BaseComponent {
+class_name() str
+to_dict() Dict
+to_json() str
+from_dict(data) Self
+from_json(data_str) Self
}
class TransformComponent {
+__call__(nodes, **kwargs) Sequence[BaseNode]
+acall(nodes, **kwargs) Sequence[BaseNode]
}
class BaseRetriever {
+retrieve(str_or_query_bundle) List[NodeWithScore]
+aretrieve(str_or_query_bundle) List[NodeWithScore]
-_retrieve(query_bundle) List[NodeWithScore]
-_aretrieve(query_bundle) List[NodeWithScore]
}
class BaseQueryEngine {
+query(str_or_query_bundle) RESPONSE_TYPE
+aquery(str_or_query_bundle) RESPONSE_TYPE
-_query(query_bundle) RESPONSE_TYPE
-_aquery(query_bundle) RESPONSE_TYPE
}
class BaseAutoRetriever {
+generate_retrieval_spec(query_bundle, **kwargs) BaseModel
+agenerate_retrieval_spec(query_bundle, **kwargs) BaseModel
+_build_retriever_from_spec(spec) (BaseRetriever, QueryBundle)
}
class BaseSelector {
+select(choices, query) SelectorResult
+aselect(choices, query) SelectorResult
-_select(choices, query) SelectorResult
-_aselect(choices, query) SelectorResult
}
class BaseNodePostprocessor {
+postprocess_nodes(nodes, query_bundle, query_str) List[NodeWithScore]
+apostprocess_nodes(nodes, query_bundle, query_str) List[NodeWithScore]
-_postprocess_nodes(nodes, query_bundle) List[NodeWithScore]
-_apostprocess_nodes(nodes, query_bundle) List[NodeWithScore]
}
TransformComponent --|> BaseComponent
BaseAutoRetriever --|> BaseRetriever
BaseSelector .. BaseComponent
BaseNodePostprocessor .. BaseComponent
```

**Diagram sources**
- [llama_index/core/schema.py](file://llama-index-core/llama_index/core/schema.py#L80-L204)
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py#L34-L275)
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py#L22-L94)
- [llama_index/core/base/base_auto_retriever.py](file://llama-index-core/llama_index/core/base/base_auto_retriever.py#L9-L44)
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py#L72-L104)
- [llama_index/core/postprocessor/types.py](file://llama-index-core/llama_index/core/postprocessor/types.py#L12-L80)

## Detailed Component Analysis

### Custom Reader Development
- Purpose: Load Documents from external sources.
- Base pattern: Extend the reader base and implement the required loading interface. Reader utilities are exposed via readers/__init__.py.
- Registration and discovery: Use the loader download utility to fetch readers dynamically.
- Integration: Documents produced by readers feed into ingestion pipelines and indices.

```mermaid
sequenceDiagram
participant User as "User Code"
participant Reader as "CustomReader(BaseReader)"
participant Loader as "download_loader()"
participant Pipeline as "Ingestion/Index"
User->>Loader : "download_loader(...)"
Loader-->>User : "Reader class"
User->>Reader : "__init__(...)"
User->>Reader : "load_data(file_path)"
Reader-->>User : "List[Document]"
User->>Pipeline : "insert(nodes)"
Pipeline-->>User : "Index ready"
```

**Diagram sources**
- [llama_index/core/readers/__init__.py](file://llama-index-core/llama_index/core/readers/__init__.py#L1-L33)

**Section sources**
- [llama_index/core/readers/__init__.py](file://llama-index-core/llama_index/core/readers/__init__.py#L1-L33)

### Custom Index Creation
- Purpose: Build and persist index structures.
- Base pattern: Use index constructors and loading utilities exported from core/__init__.py. Indices are registered and discoverable via the top-level exports.
- Integration: Indices integrate with retrievers and query engines through shared schemas and node types.

```mermaid
flowchart TD
Start(["Start"]) --> ChooseIdx["Choose Index Type"]
ChooseIdx --> BuildCfg["Configure Index Settings"]
BuildCfg --> Ingest["Ingest Documents"]
Ingest --> Persist["Persist to Storage"]
Persist --> Load["Load from Storage"]
Load --> Ready(["Index Ready"])
```

**Diagram sources**
- [llama-index-core/llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py#L24-L48)

**Section sources**
- [llama-index-core/llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py#L24-L48)

### Custom Retriever Implementation
- Purpose: Retrieve relevant nodes for a query.
- Base pattern: Subclass BaseRetriever and implement _retrieve. Optionally implement _aretrieve for async support. Leverage recursive retrieval helpers and instrumentation.
- Auto-retrievers: For dynamic retrieval strategies, subclass BaseAutoRetriever and implement spec generation and builder methods.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant Retriever as "CustomRetriever(BaseRetriever)"
participant Rec as "_handle_recursive_retrieval"
participant Builder as "Optional Builder/AutoRetriever"
Client->>Retriever : "retrieve(QueryType)"
Retriever->>Retriever : "_retrieve(QueryBundle)"
alt "Recursive nodes"
Retriever->>Rec : "handle nodes"
Rec-->>Retriever : "deduplicated nodes"
end
Retriever-->>Client : "List[NodeWithScore]"
opt "Auto-retriever path"
Client->>Builder : "generate_retrieval_spec"
Builder-->>Client : "spec"
Client->>Builder : "_build_retriever_from_spec"
Builder-->>Client : "(BaseRetriever, QueryBundle)"
end
```

**Diagram sources**
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py#L185-L254)
- [llama_index/core/base/base_auto_retriever.py](file://llama-index-core/llama_index/core/base/base_auto_retriever.py#L33-L43)

**Section sources**
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py#L34-L275)
- [llama_index/core/base/base_auto_retriever.py](file://llama-index-core/llama_index/core/base/base_auto_retriever.py#L9-L44)

### Custom Query Engine Implementation
- Purpose: Execute queries and optionally synthesize responses from retrieved nodes.
- Base pattern: Subclass BaseQueryEngine and implement _query and _aquery. Use the provided query wrappers and instrumentation.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant QEng as "CustomQueryEngine(BaseQueryEngine)"
Client->>QEng : "query(QueryType)"
QEng->>QEng : "_query(QueryBundle)"
QEng-->>Client : "RESPONSE_TYPE"
Client->>QEng : "aquery(QueryType)"
QEng->>QEng : "_aquery(QueryBundle)"
QEng-->>Client : "RESPONSE_TYPE"
```

**Diagram sources**
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py#L38-L60)

**Section sources**
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py#L22-L94)

### Custom Selector Implementation
- Purpose: Select among choices given a query.
- Base pattern: Subclass BaseSelector and implement _select and _aselect. Inputs are normalized to QueryBundle and ToolMetadata.

```mermaid
flowchart TD
A["select(choices, query)"] --> Wrap["Wrap choices and query"]
Wrap --> Call["_select(choices, QueryBundle)"]
Call --> Result["SelectorResult"]
```

**Diagram sources**
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py#L79-L97)

**Section sources**
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py#L72-L104)

### Custom Post-processor Development
- Purpose: Filter, reorder, or otherwise transform ranked nodes.
- Base pattern: Subclass BaseNodePostprocessor and implement _postprocess_nodes. Async variant provided via apostprocess_nodes.

```mermaid
flowchart TD
Start(["postprocess_nodes(nodes, query)"]) --> Normalize["Normalize query"]
Normalize --> Apply["_postprocess_nodes(nodes, QueryBundle)"]
Apply --> Return(["Return transformed nodes"])
```

**Diagram sources**
- [llama_index/core/postprocessor/types.py](file://llama-index-core/llama_index/core/postprocessor/types.py#L35-L56)

**Section sources**
- [llama_index/core/postprocessor/types.py](file://llama-index-core/llama_index/core/postprocessor/types.py#L12-L80)

## Dependency Analysis
LlamaIndex exposes a curated set of public APIs through core/__init__.py, including indices, readers, prompts, and utilities. Component registration and persistence leverage BaseComponent’s class_name and serialization hooks. ServiceContext provides a global configuration surface for components.

```mermaid
graph LR
CORE_INIT["core/__init__.py"] --> READERS_API["readers/__init__.py"]
CORE_INIT --> SCHEMA_API["schema.py"]
CORE_INIT --> RETRIEVER_API["base/base_retriever.py"]
CORE_INIT --> QUERY_API["base/base_query_engine.py"]
CORE_INIT --> AUTO_API["base/base_auto_retriever.py"]
CORE_INIT --> SELECTOR_API["base/base_selector.py"]
CORE_INIT --> POST_API["postprocessor/types.py"]
REGISTRY["data_structs/registry.py"] --> CORE_INIT
SERVICE_CTX["service_context.py"] --> CORE_INIT
```

**Diagram sources**
- [llama-index-core/llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py#L24-L150)
- [llama_index/core/readers/__init__.py](file://llama-index-core/llama_index/core/readers/__init__.py#L1-L33)
- [llama-index-core/llama_index/core/data_structs/registry.py](file://llama-index-core/llama_index/core/data_structs/registry.py)
- [llama-index-core/llama_index/core/service_context.py](file://llama-index-core/llama_index/core/service_context.py)

**Section sources**
- [llama-index-core/llama_index/core/__init__.py](file://llama-index-core/llama_index/core/__init__.py#L24-L150)

## Performance Considerations
- Prefer async variants (_aretrieve, _aquery, apostprocess_nodes) when underlying operations are I/O bound.
- Deduplicate nodes early to reduce downstream processing cost.
- Use instrumentation and callback managers to profile bottlenecks without modifying core logic.
- Keep prompt templates minimal and cache heavy computations where appropriate.

## Troubleshooting Guide
- Retrieval failures: Verify that _retrieve returns NodeWithScore objects and that recursive retrieval is not causing duplicates. Inspect callback events and spans for visibility.
- Serialization issues: Ensure custom components implement class_name consistently and avoid unpickleable attributes. Use BaseComponent’s serialization helpers.
- Query engine errors: Confirm that _query and _aquery handle QueryBundle properly and that unsupported operations (synthesize, retrieve) are not invoked when not implemented.
- Selector mismatches: Validate that choices are either ToolMetadata or strings and that queries are normalized to QueryBundle.

**Section sources**
- [llama_index/core/base/base_retriever.py](file://llama-index-core/llama_index/core/base/base_retriever.py#L185-L254)
- [llama-index/core/schema.py](file://llama-index-core/llama_index/core/schema.py#L80-L188)
- [llama_index/core/base/base_query_engine.py](file://llama-index-core/llama_index/core/base/base_query_engine.py#L62-L85)
- [llama_index/core/base/base_selector.py](file://llama-index-core/llama_index/core/base/base_selector.py#L79-L97)

## Conclusion
By extending the provided base classes and adhering to the shared schema and registry patterns, developers can build robust, interoperable custom components for readers, indices, retrievers, query engines, selectors, and post-processors. The plugin architecture, combined with instrumentation, callback management, and serialization hooks, enables scalable integration and maintainability.

## Appendices

### Plugin Architecture and Factory Patterns
- Component registration: Use BaseComponent’s class_name and serialization to register components in the registry.
- Factories: Construct components via from_dict/from_json or factory-style constructors exported by modules.

**Section sources**
- [llama_index/core/schema.py](file://llama-index-core/llama_index/core/schema.py#L80-L188)
- [llama-index-core/llama_index/core/data_structs/registry.py](file://llama-index-core/llama_index/core/data_structs/registry.py)

### Dependency Injection Mechanisms
- Global ServiceContext: Configure shared dependencies (LLMs, embedders, tokenizers) globally and reference them in components.
- Constructor injection: Pass dependencies explicitly to component constructors for testability.

**Section sources**
- [llama-index-core/llama_index/core/service_context.py](file://llama-index-core/llama_index/core/service_context.py)

### Testing Strategies for Custom Components
- Unit tests: Mock callback managers and instrumentation; assert that _retrieve/_query/_postprocess_nodes return expected outputs.
- Integration tests: Wire components into retrieval/query pipelines; validate end-to-end flows with real data loaders and indices.
- Async tests: Ensure async methods behave consistently with sync counterparts.

[No sources needed since this section provides general guidance]