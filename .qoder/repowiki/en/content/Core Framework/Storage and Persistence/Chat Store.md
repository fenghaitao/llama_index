# Chat Store

<cite>
**Referenced Files in This Document**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py)
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py)
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md)
- [test_simple_chat_store.py](file://llama-index-core/tests/storage/chat_store/test_simple_chat_store.py)
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
This document explains the Chat Store component of LlamaIndex storage system. It focuses on the BaseChatStore interface, the SimpleChatStore implementation, and the broader ecosystem of database-backed chat stores. It covers how chat messages, session data, and conversation metadata are stored and managed, and how chat stores integrate with memory modules and chat engines. Practical examples demonstrate configuration, persistence, retrieval, and session management patterns.

## Project Structure
The Chat Store functionality resides under the core storage module and is complemented by documentation and tests. The key files include the base interface, a simple in-memory implementation, a database abstraction, and a concrete SQLAlchemy-backed implementation. A loader utility recognizes known chat store types.

```mermaid
graph TB
subgraph "Core Storage"
A["base.py<br/>BaseChatStore"]
B["simple_chat_store.py<br/>SimpleChatStore"]
C["base_db.py<br/>AsyncDBChatStore + MessageStatus"]
D["sql.py<br/>SQLAlchemyChatStore"]
E["loading.py<br/>load_chat_store"]
end
subgraph "Docs & Tests"
F["chat_stores.md<br/>Guides & Examples"]
G["test_simple_chat_store.py<br/>Unit Tests"]
end
A --> B
C --> D
E --> B
E --> D
F --> B
F --> D
G --> B
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L11-L79)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L31-L113)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L9-L102)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L35-L457)
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L4-L19)
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L1-L437)
- [test_simple_chat_store.py](file://llama-index-core/tests/storage/chat_store/test_simple_chat_store.py#L1-L77)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L1-L79)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L1-L113)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L1-L102)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L1-L457)
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L1-L19)
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L1-L437)
- [test_simple_chat_store.py](file://llama-index-core/tests/storage/chat_store/test_simple_chat_store.py#L1-L77)

## Core Components
- BaseChatStore: Defines the contract for chat history storage, including synchronous and asynchronous methods for setting, retrieving, appending, and deleting messages, as well as listing keys.
- SimpleChatStore: An in-memory chat store with optional persistence to disk via JSON serialization. It supports insertion at index and deletion semantics.
- AsyncDBChatStore and MessageStatus: Abstract base for database-backed stores and the status enumeration used to manage active vs archived messages.
- SQLAlchemyChatStore: A concrete database-backed chat store using SQLAlchemy async engine, with rich operations including counting, ordering by insertion time, archiving, and dumping data for in-memory scenarios.
- Loading utility: Recognizes known chat store types and loads them from dictionaries.

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L11-L79)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L31-L113)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L9-L102)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L35-L457)
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L4-L19)

## Architecture Overview
The chat store architecture separates concerns between in-memory and persistent storage, while enabling database-backed implementations with advanced features like status tracking and ordered retrieval.

```mermaid
classDiagram
class BaseChatStore {
+set_messages(key, messages) void
+get_messages(key) ChatMessage[]
+add_message(key, message) void
+delete_messages(key) ChatMessage[]?
+delete_message(key, idx) ChatMessage?
+delete_last_message(key) ChatMessage?
+get_keys() str[]
+aset_messages(...)
+aget_messages(...)
+async_add_message(...)
+adelete_messages(...)
+adelete_message(...)
+adelete_last_message(...)
+aget_keys() str[]
}
class SimpleChatStore {
+store : Dict~str, AnnotatedChatMessage[]~
+persist(persist_path, fs) void
+from_persist_path(persist_path, fs) SimpleChatStore
}
class AsyncDBChatStore {
<<abstract>>
+get_messages(key, status, limit, offset) ChatMessage[]
+count_messages(key, status) int
+add_message(key, message, status) void
+add_messages(key, messages, status) void
+set_messages(key, messages, status) void
+delete_message(key, idx) ChatMessage?
+delete_messages(key, status) void
+delete_oldest_messages(key, n) ChatMessage[]
+archive_oldest_messages(key, n) ChatMessage[]
+get_keys() str[]
}
class MessageStatus {
<<enum>>
ACTIVE
ARCHIVED
}
class SQLAlchemyChatStore {
+table_name : str
+async_database_uri : str
+db_schema : str?
+get_messages(...)
+count_messages(...)
+add_message(...)
+add_messages(...)
+set_messages(...)
+delete_message(...)
+delete_messages(...)
+delete_oldest_messages(...)
+archive_oldest_messages(...)
+get_keys() str[]
+dump_store() dict
}
BaseChatStore <|-- SimpleChatStore
AsyncDBChatStore <|-- SQLAlchemyChatStore
MessageStatus <.. AsyncDBChatStore
MessageStatus <.. SQLAlchemyChatStore
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L11-L79)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L31-L113)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L19-L102)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L35-L457)

## Detailed Component Analysis

### BaseChatStore Interface
- Purpose: Defines a uniform interface for chat history management across implementations.
- Core operations:
  - Set/get messages by key
  - Append single message with optional index insertion
  - Delete entire history, a specific message by index, or the last message
  - List all keys
- Asynchronous variants: Provided for convenience and compatibility with async contexts.

```mermaid
flowchart TD
Start(["Call get_messages(key)"]) --> CheckKey["Key exists?"]
CheckKey --> |No| ReturnEmpty["Return []"]
CheckKey --> |Yes| ReturnList["Return stored messages"]
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L17-L35)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L11-L79)

### SimpleChatStore Implementation
- In-memory storage keyed by strings with optional persistence to JSON.
- Message serialization:
  - Uses a Pydantic serializer wrapper to ensure additional kwargs are serializable.
  - Validates that nested additional kwargs can be represented as JSON-compatible types.
- Persistence:
  - Persist to a file path using fsspec filesystem abstraction.
  - Load from path or raw JSON string.
- Operations:
  - Append or insert at index
  - Delete by key, by index, or last element
  - List keys

```mermaid
sequenceDiagram
participant App as "Application"
participant Store as "SimpleChatStore"
participant FS as "fsspec FileSystem"
App->>Store : persist(persist_path)
Store->>FS : open(persist_path, "w")
Store->>Store : json()
Store-->>FS : write serialized data
FS-->>Store : close()
App->>Store : from_persist_path(persist_path)
Store->>FS : open(persist_path, "r")
FS-->>Store : read data
Store-->>App : model_validate(...) or model_validate_json(...)
```

**Diagram sources**
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L82-L113)

**Section sources**
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L31-L113)
- [test_simple_chat_store.py](file://llama-index-core/tests/storage/chat_store/test_simple_chat_store.py#L1-L77)

### Database Abstractions and Status Tracking
- AsyncDBChatStore: Abstract base for database-backed stores with operations for counting, ordering, and status-aware management.
- MessageStatus: Enumerates ACTIVE and ARCHIVED statuses to support FIFO short-term memory and archival workflows.

```mermaid
classDiagram
class AsyncDBChatStore {
<<abstract>>
+get_messages(key, status, limit, offset) ChatMessage[]
+count_messages(key, status) int
+add_message(key, message, status) void
+add_messages(key, messages, status) void
+set_messages(key, messages, status) void
+delete_message(key, idx) ChatMessage?
+delete_messages(key, status) void
+delete_oldest_messages(key, n) ChatMessage[]
+archive_oldest_messages(key, n) ChatMessage[]
+get_keys() str[]
}
class MessageStatus {
<<enum>>
ACTIVE
ARCHIVED
}
AsyncDBChatStore --> MessageStatus : "uses"
```

**Diagram sources**
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L19-L102)

**Section sources**
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L9-L102)

### SQLAlchemyChatStore (Database-backed)
- Provides a robust, async-first implementation backed by SQLAlchemy.
- Features:
  - Ordered insertion via nanosecond-precision timestamps
  - Status-aware queries and updates (ACTIVE vs ARCHIVED)
  - Batch operations for adding messages
  - Archival and deletion of oldest messages
  - Schema support for databases that allow it
  - In-memory dump/restore for migration or testing
- Methods mirror AsyncDBChatStore with async implementations.

```mermaid
sequenceDiagram
participant App as "Application"
participant Store as "SQLAlchemyChatStore"
participant Engine as "AsyncEngine"
participant Table as "messages Table"
App->>Store : add_messages(key, messages, status)
Store->>Store : _initialize()
Store->>Engine : create_async_engine(if needed)
Store->>Engine : metadata.create_all()
Store->>Table : insert(values with timestamps)
Engine-->>Store : commit
Store-->>App : done
```

**Diagram sources**
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L91-L169)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L243-L268)

**Section sources**
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L35-L457)

### Loading Utility
- Recognizes known chat store types and reconstructs them from dictionaries.
- Enables dynamic loading of persisted stores.

```mermaid
flowchart TD
A["load_chat_store(data)"] --> B{"class_name present?"}
B --> |No| E["Raise ValueError"]
B --> |Yes| C{"Known store?"}
C --> |No| F["Raise ValueError"]
C --> |Yes| D["Instantiate via from_dict"]
```

**Diagram sources**
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L4-L19)

**Section sources**
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L1-L19)

### Integration with Memory and Engines (Documentation Examples)
- The documentation demonstrates how to wire SimpleChatStore with ChatMemoryBuffer and then use the memory in agents or chat engines.
- It also shows persistence via JSON and string serialization.

```mermaid
sequenceDiagram
participant User as "User"
participant Mem as "ChatMemoryBuffer"
participant Store as "SimpleChatStore"
participant Agent as "Agent/ChatEngine"
User->>Mem : Provide query
Mem->>Store : get_messages(key)
Store-->>Mem : List of ChatMessage
Mem->>Agent : Run with context
Agent-->>User : Response
User->>Store : persist()/from_persist_path()
```

**Diagram sources**
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L15-L51)

**Section sources**
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L1-L437)

## Dependency Analysis
- SimpleChatStore depends on BaseChatStore and uses Pydantic serialization with a custom serializer for ChatMessage additional kwargs.
- SQLAlchemyChatStore depends on AsyncDBChatStore and SQLAlchemy async primitives.
- The loader utility depends on recognized class names to instantiate stores.

```mermaid
graph LR
Base["BaseChatStore"] --> Simple["SimpleChatStore"]
BaseDB["AsyncDBChatStore"] --> SQL["SQLAlchemyChatStore"]
Loader["load_chat_store"] --> Simple
Loader --> SQL
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L11-L79)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L31-L113)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L19-L102)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L35-L457)
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L4-L19)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/storage/chat_store/base.py#L11-L79)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L31-L113)
- [base_db.py](file://llama-index-core/llama_index/core/storage/chat_store/base_db.py#L19-L102)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L35-L457)
- [loading.py](file://llama-index-core/llama_index/core/storage/chat_store/loading.py#L4-L19)

## Performance Considerations
- SimpleChatStore:
  - In-memory operations are fast but not durable; use persist() judiciously.
  - Serialization validates additional kwargs to ensure safe JSON storage.
- SQLAlchemyChatStore:
  - Timestamp-based ordering ensures deterministic retrieval order.
  - Batch insert reduces round-trips for bulk operations.
  - Status filtering enables efficient archival and pruning of old messages.
  - Schema support allows separation of data for supported databases.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- Serialization errors:
  - Additional kwargs must be JSON-serializable; otherwise, serialization raises an error. Ensure custom fields are basic types or convertible to JSON-compatible structures.
- Missing keys:
  - get_messages returns an empty list for unknown keys; ensure the correct key is used when persisting/retrieving.
- Index out of bounds:
  - Deleting by index returns None if the index is invalid; check message length before deletion.
- Persistence:
  - Ensure the target directory exists before persist(); the implementation creates directories if missing.
- Database initialization:
  - The async engine and session factory are lazily initialized; ensure a valid async database URI or pre-provided engine is supplied.

**Section sources**
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L12-L28)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L64-L76)
- [simple_chat_store.py](file://llama-index-core/llama_index/core/storage/chat_store/simple_chat_store.py#L82-L94)
- [sql.py](file://llama-index-core/llama_index/core/storage/chat_store/sql.py#L110-L133)

## Conclusion
The Chat Store system provides a flexible, extensible foundation for managing conversation history. BaseChatStore defines a consistent interface, SimpleChatStore offers a practical in-memory option with persistence, and AsyncDBChatStore plus SQLAlchemyChatStore enable scalable, status-aware, and ordered storage. Together with the loader utility and documented integration patterns, developers can build robust chat experiences with reliable session management and durable persistence.

[No sources needed since this section summarizes without analyzing specific files]

## Appendices

### Practical Examples and Patterns
- Configure SimpleChatStore with ChatMemoryBuffer and persist to disk or serialize to string.
- Use database-backed stores for multi-instance deployments and advanced lifecycle management.
- Leverage status-aware operations to maintain short-term memory and archive older messages.

**Section sources**
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L15-L51)
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L53-L111)
- [chat_stores.md](file://docs/src/content/docs/framework/module_guides/storing/chat_stores.md#L231-L248)