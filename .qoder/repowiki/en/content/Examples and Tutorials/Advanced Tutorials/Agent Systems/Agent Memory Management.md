# Agent Memory Management

<cite>
**Referenced Files in This Document**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py)
- [types.py](file://llama-index-core/llama_index/core/memory/types.py)
- [chat_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_memory_buffer.py)
- [chat_summary_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_summary_memory_buffer.py)
- [simple_composable_memory.py](file://llama-index-core/llama_index/core/memory/simple_composable_memory.py)
- [vector_memory.py](file://llama-index-core/llama_index/core/memory/vector_memory.py)
- [memory_blocks/static.py](file://llama-index-core/llama_index/core/memory/memory_blocks/static.py)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py)
- [__init__.py](file://llama-index-core/llama_index/core/memory/__init__.py)
- [test_chat_memory_buffer.py](file://llama-index-core/tests/memory/test_chat_memory_buffer.py)
- [test_chat_summary_memory_buffer.py](file://llama-index-core/tests/memory/test_chat_summary_memory_buffer.py)
- [test_memory_base.py](file://llama-index-core/tests/memory/test_memory_base.py)
- [test_memory_blocks_base.py](file://llama-index-core/tests/memory/test_memory_blocks_base.py)
- [test_memory_schema.py](file://llama-index-core/tests/memory/test_memory_schema.py)
- [test_vector_memory.py](file://llama-index-core/tests/agent/memory/test_vector_memory.py)
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
This document explains agent memory management in LlamaIndex with a focus on short-term and long-term memory systems for maintaining conversational context. It covers memory buffer implementations, summarization strategies, context preservation techniques, memory block types, composition patterns, and state persistence mechanisms. It also provides guidance on optimization, storage efficiency, retrieval strategies, custom memory implementations, memory scrubbing, and privacy considerations.

## Project Structure
The memory subsystem centers around a unified Memory orchestration component that coordinates short-term chat history with pluggable memory blocks for long-term knowledge. Supporting components include legacy buffers (deprecated), vector-backed memory, and composable memory for multi-source context.

```mermaid
graph TB
subgraph "Core Memory"
M["Memory<br/>orchestrates chat + blocks"]
MB["BaseMemoryBlock<br/>abstract interface"]
TM["SQLAlchemyChatStore<br/>persistence"]
end
subgraph "Memory Blocks"
SB["StaticMemoryBlock"]
FB["FactExtractionMemoryBlock"]
VB["VectorMemoryBlock"]
end
subgraph "Legacy Buffers (Deprecated)"
CMB["ChatMemoryBuffer"]
CSMB["ChatSummaryMemoryBuffer"]
SCM["SimpleComposableMemory"]
end
subgraph "Vector Memory"
VM["VectorMemory"]
end
M --> TM
M --> MB
MB --> SB
MB --> FB
MB --> VB
SCM --> CMB
SCM --> CSMB
VM --> TM
```

**Diagram sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L800)
- [types.py](file://llama-index-core/llama_index/core/memory/types.py#L14-L153)
- [memory_blocks/static.py](file://llama-index-core/llama_index/core/memory/memory_blocks/static.py#L8-L40)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L177)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L202)
- [chat_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_memory_buffer.py#L19-L167)
- [chat_summary_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_summary_memory_buffer.py#L26-L341)
- [simple_composable_memory.py](file://llama-index-core/llama_index/core/memory/simple_composable_memory.py#L14-L164)
- [vector_memory.py](file://llama-index-core/llama_index/core/memory/vector_memory.py#L48-L207)

**Section sources**
- [__init__.py](file://llama-index-core/llama_index/core/memory/__init__.py#L1-L27)
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L800)
- [types.py](file://llama-index-core/llama_index/core/memory/types.py#L14-L153)

## Core Components
- Memory: Central orchestrator that maintains a token-bounded short-term chat history and integrates long-term knowledge via memory blocks. It supports configurable token limits, flush sizes, insertion strategies (system vs. user message), and content truncation across blocks.
- BaseMemoryBlock: Abstract interface for memory blocks. Implementations define how to aget content and aput archived messages, with optional atruncate to reduce size.
- Memory Blocks:
  - StaticMemoryBlock: Injects fixed content into context.
  - FactExtractionMemoryBlock: Extracts and condenses factual statements from conversation history.
  - VectorMemoryBlock: Retrieves relevant past messages from a vector store.
- Legacy Buffers (Deprecated): ChatMemoryBuffer, ChatSummaryMemoryBuffer, SimpleComposableMemory.
- VectorMemory: Stores chat batches as nodes in a vector index for retrieval.

Key capabilities:
- Short-term memory: FIFO queue persisted via SQL chat store with token-aware eviction and conversation-boundary preservation.
- Long-term memory: Pluggable blocks that receive “waterfalled” messages (evicted from short-term) and produce context for injection.
- Context injection: Memory blocks’ content is inserted into the system message or the latest user message based on insert method.

**Section sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L800)
- [types.py](file://llama-index-core/llama_index/core/memory/types.py#L14-L153)
- [memory_blocks/static.py](file://llama-index-core/llama_index/core/memory/memory_blocks/static.py#L8-L40)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L177)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L202)
- [chat_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_memory_buffer.py#L19-L167)
- [chat_summary_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_summary_memory_buffer.py#L26-L341)
- [simple_composable_memory.py](file://llama-index-core/llama_index/core/memory/simple_composable_memory.py#L14-L164)
- [vector_memory.py](file://llama-index-core/llama_index/core/memory/vector_memory.py#L48-L207)

## Architecture Overview
The Memory component manages a bounded chat history and coordinates with memory blocks to preserve and inject relevant knowledge. It uses a SQL chat store for persistence and applies token estimation to enforce limits. When the queue grows beyond capacity, it evicts older messages in conversation-turn-safe chunks and “waters falls” them into memory blocks for archival and later retrieval.

```mermaid
sequenceDiagram
participant Agent as "Agent"
participant Mem as "Memory"
participant Store as "SQLAlchemyChatStore"
participant Blocks as "Memory Blocks"
Agent->>Mem : "aput(message)"
Mem->>Store : "add_message(session_id, message, ACTIVE)"
Mem->>Mem : "_manage_queue()"
Mem->>Store : "archive_oldest_messages(n)"
Mem->>Blocks : "aput(messages, from_short_term_memory=True)"
Agent->>Mem : "aget(input?)"
Mem->>Store : "get_messages(session_id, ACTIVE)"
Mem->>Blocks : "aget(messages, session_id)"
Mem->>Mem : "_truncate_memory_blocks()"
Mem->>Mem : "_format_memory_blocks()"
Mem->>Mem : "_insert_memory_content()"
Mem-->>Agent : "List[ChatMessage]"
```

**Diagram sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L655-L800)
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L246)

## Detailed Component Analysis

### Memory Orchestration
- Token management: Enforces a global token limit and reserves a minimum ratio for chat history. Flush size controls chunk eviction granularity.
- Queue management: Evicts oldest messages while preserving complete conversation turns and ensuring at least one message remains.
- Block integration: Aggregates content from memory blocks, formats via a template, and injects into the system message or latest user message.
- Persistence: Uses a SQL chat store keyed by session_id; supports async operations.

```mermaid
flowchart TD
Start(["aput(message)"]) --> AddMsg["Store.add_message(session_id, message)"]
AddMsg --> ManageQ["_manage_queue()"]
ManageQ --> CheckLimit{"Queue tokens > limit?"}
CheckLimit --> |No| End(["Done"])
CheckLimit --> |Yes| Evict["Pop oldest messages<br/>until under limit"]
Evict --> PreserveTurn["Ensure conversation boundary preserved"]
PreserveTurn --> Archive["archive_oldest_messages(n)"]
Archive --> Waterfall["async gather: block.aput(messages, from_short_term_memory=True)"]
Waterfall --> End
%% aget path
AStart(["aget(input)"]) --> Load["sql_store.get_messages(ACTIVE)"]
Load --> BlocksA["_get_memory_blocks_content()"]
BlocksA --> Trunc["_truncate_memory_blocks()"]
Trunc --> Format["_format_memory_blocks()"]
Format --> Insert["_insert_memory_content()"]
Insert --> AEnd(["Return ChatMessage[]"])
```

**Diagram sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L655-L800)
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L429-L653)

**Section sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L800)

### Memory Block Types and Composition
- StaticMemoryBlock: Provides fixed content blocks for persistent instructions or context.
- FactExtractionMemoryBlock: Extracts explicit facts from conversation segments and condenses them when exceeding a maximum count.
- VectorMemoryBlock: Encodes recent context into embeddings and retrieves semantically similar stored messages from a vector store, optionally filtered by session_id.

```mermaid
classDiagram
class BaseMemoryBlock {
+string name
+string description
+int priority
+bool accept_short_term_memory
+aget(messages, **kwargs) T
+aput(messages, from_short_term_memory, session_id) void
+atruncate(content, tokens_to_truncate) T?
}
class StaticMemoryBlock {
+ContentBlock[] static_content
+_aget(...)
+_aput(...)
}
class FactExtractionMemoryBlock {
+string[] facts
+int max_facts
+_aget(...)
+_aput(messages)
}
class VectorMemoryBlock {
+BasePydanticVectorStore vector_store
+BaseEmbedding embed_model
+int similarity_top_k
+int retrieval_context_window
+_aget(messages, session_id)
+_aput(messages)
}
BaseMemoryBlock <|-- StaticMemoryBlock
BaseMemoryBlock <|-- FactExtractionMemoryBlock
BaseMemoryBlock <|-- VectorMemoryBlock
```

**Diagram sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L94-L177)
- [memory_blocks/static.py](file://llama-index-core/llama_index/core/memory/memory_blocks/static.py#L8-L40)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L177)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L202)

**Section sources**
- [memory_blocks/static.py](file://llama-index-core/llama_index/core/memory/memory_blocks/static.py#L8-L40)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L177)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L202)

### Summarization Strategies and Context Preservation
- ChatSummaryMemoryBuffer (Deprecated): Iteratively summarizes older messages using an LLM to keep the most recent portion as full-text within the token budget. Preserves assistant/tool pairs and enforces conversation boundaries.
- Memory’s waterfall: Evicts older messages safely and passes them to memory blocks, enabling archival and later retrieval without manual summarization.

```mermaid
flowchart TD
SStart(["get() with initial_token_count"]) --> Split["_split_messages_summary_or_full_text()"]
Split --> Decide{"Has LLM and messages to summarize?"}
Decide --> |No| KeepFull["Use full-text portion"]
Decide --> |Yes| Sum["_summarize_oldest_chat_history()"]
Sum --> Combine["[summary] + full-text"]
KeepFull --> Combine
Combine --> ResetSet["reset() + set(updated_history)"]
ResetSet --> SEnd(["Return updated history"])
```

**Diagram sources**
- [chat_summary_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_summary_memory_buffer.py#L167-L341)

**Section sources**
- [chat_summary_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_summary_memory_buffer.py#L26-L341)
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L655-L800)

### Vector-Based Memory and Retrieval
- VectorMemory: Stores message batches as nodes in a vector index and retrieves relevant nodes for a query. Supports combining adjacent messages by user role to preserve function-calling contexts.
- VectorMemoryBlock: Retrieves relevant stored messages from a vector store using embeddings, applies optional postprocessors, and formats results for inclusion.

```mermaid
sequenceDiagram
participant Agent as "Agent"
participant VMB as "VectorMemoryBlock"
participant VS as "VectorStore"
participant EM as "Embedding Model"
Agent->>VMB : "aget(messages, session_id)"
VMB->>VMB : "select context window"
VMB->>EM : "aget_query_embedding(text)"
VMB->>VS : "aquery(VectorStoreQuery)"
VS-->>VMB : "nodes_with_scores"
VMB->>VMB : "postprocess nodes"
VMB-->>Agent : "formatted text"
```

**Diagram sources**
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L100-L168)

**Section sources**
- [vector_memory.py](file://llama-index-core/llama_index/core/memory/vector_memory.py#L48-L207)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L202)

### Legacy Buffers and Composability
- ChatMemoryBuffer (Deprecated): Maintains a sliding window of chat history respecting token limits and conversation boundaries.
- SimpleComposableMemory (Deprecated): Composes a primary memory with secondary sources, injecting secondary histories into the system prompt.

**Section sources**
- [chat_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_memory_buffer.py#L19-L167)
- [simple_composable_memory.py](file://llama-index-core/llama_index/core/memory/simple_composable_memory.py#L14-L164)

## Dependency Analysis
- Memory depends on:
  - SQLAlchemyChatStore for persistence and session scoping.
  - Tokenizer for token estimation across text, images, audio, and video content.
  - Memory blocks for long-term context generation.
- Memory blocks depend on:
  - Embedding models for VectorMemoryBlock.
  - Vector stores for retrieval.
  - LLMs for FactExtractionMemoryBlock.
- Legacy buffers are standalone and deprecated in favor of Memory.

```mermaid
graph LR
Mem["Memory"] --> Store["SQLAlchemyChatStore"]
Mem --> Blocks["BaseMemoryBlock*"]
Blocks --> Embed["Embedding Model"]
Blocks --> VS["Vector Store"]
Blocks --> LLM["LLM"]
SCM["SimpleComposableMemory"] --> CMB["ChatMemoryBuffer"]
SCM --> CSMB["ChatSummaryMemoryBuffer"]
```

**Diagram sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L246)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L66)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L95)
- [simple_composable_memory.py](file://llama-index-core/llama_index/core/memory/simple_composable_memory.py#L14-L61)

**Section sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L179-L246)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L66)
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L95)
- [simple_composable_memory.py](file://llama-index-core/llama_index/core/memory/simple_composable_memory.py#L14-L61)

## Performance Considerations
- Token estimation: Accurate estimation of text and multimodal content prevents overflows and reduces unnecessary truncation.
- Flush size and chat history ratio: Tune to balance latency and context retention.
- Block truncation: Prioritized truncation minimizes impact on high-priority blocks.
- Vector retrieval: Adjust similarity_top_k and context window to trade recall for speed.
- Asynchronous operations: Use async APIs for chat store and embedding operations to avoid blocking.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
Common issues and resolutions:
- Token limit errors: Validate token_limit and token_flush_size; ensure tokenizer_fn is set.
- Conversation boundary violations: The queue manager preserves user-assistant-tool pairs; confirm input messages follow expected roles.
- Memory block content type errors: Ensure blocks return supported types (str, ContentBlock[], ChatMessage[]).
- Session scoping: When using VectorMemoryBlock, ensure session_id is propagated and filters are applied consistently.
- Legacy buffer deprecation: Prefer Memory for new implementations.

**Section sources**
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L251-L273)
- [memory.py](file://llama-index-core/llama_index/core/memory/memory.py#L429-L462)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L123-L142)
- [chat_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_memory_buffer.py#L37-L50)
- [chat_summary_memory_buffer.py](file://llama-index-core/llama_index/core/memory/chat_summary_memory_buffer.py#L65-L81)

## Conclusion
LlamaIndex provides a flexible, extensible memory framework. Memory orchestrates short-term chat history with long-term knowledge via memory blocks, supporting static context, factual extraction, and vector retrieval. Its design emphasizes token-awareness, conversation integrity, and asynchronous persistence, enabling efficient and privacy-conscious agent conversations.

[No sources needed since this section summarizes without analyzing specific files]

## Appendices

### Examples and Patterns
- Chat summaries: Use FactExtractionMemoryBlock to capture explicit facts and condense them over time.
- Conversation histories: Use VectorMemoryBlock to retrieve relevant past exchanges filtered by session_id.
- Knowledge retention: Combine StaticMemoryBlock for instructions and VectorMemoryBlock for dynamic retrieval.

**Section sources**
- [memory_blocks/fact.py](file://llama-index-core/llama_index/core/memory/memory_blocks/fact.py#L66-L177)
- [memory_blocks/vector.py](file://llama-index-core/llama_index/core/memory/memory_blocks/vector.py#L29-L202)
- [memory_blocks/static.py](file://llama-index-core/llama_index/core/memory/memory_blocks/static.py#L8-L40)

### Testing References
- Memory orchestration and block integration tests validate token handling, truncation, and injection logic.
- Legacy buffer tests cover backward compatibility and behavior under token constraints.
- Vector memory tests validate retrieval and storage semantics.

**Section sources**
- [test_memory_base.py](file://llama-index-core/tests/memory/test_memory_base.py)
- [test_memory_blocks_base.py](file://llama-index-core/tests/memory/test_memory_blocks_base.py)
- [test_memory_schema.py](file://llama-index-core/tests/memory/test_memory_schema.py)
- [test_chat_memory_buffer.py](file://llama-index-core/tests/memory/test_chat_memory_buffer.py)
- [test_chat_summary_memory_buffer.py](file://llama-index-core/tests/memory/test_chat_summary_memory_buffer.py)
- [test_vector_memory.py](file://llama-index-core/tests/agent/memory/test_vector_memory.py)