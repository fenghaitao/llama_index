# Context Only Synthesis Strategy

<cite>
**Referenced Files in This Document**
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py)
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py)
- [response_synthesizers.md](file://docs/src/content/docs/framework/module_guides/querying/response_synthesizers/response_synthesizers.md)
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

## Introduction
The Context Only synthesis strategy is a lightweight response synthesis mode that returns a concatenated string of all retrieved text chunks without invoking an LLM or performing iterative processing. It is ideal for straightforward retrieval tasks where the goal is to present raw context directly to the user or downstream systems, minimizing processing overhead and latency.

## Project Structure
The Context Only strategy is implemented as a dedicated synthesizer class within the response synthesizers module. It integrates with the broader synthesis framework via a factory and enumeration of response modes.

```mermaid
graph TB
subgraph "Response Synthesizers Module"
A["context_only.py<br/>Defines ContextOnly"]
B["base.py<br/>Defines BaseSynthesizer"]
C["factory.py<br/>Factory for synthesizers"]
D["type.py<br/>ResponseMode enum including CONTEXT_ONLY"]
end
E["response_synthesizers.md<br/>Usage guide and examples"]
C --> A
D --> C
B --> A
E --> C
```

**Diagram sources**
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L1-L31)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L53-L322)
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L33-L152)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L4-L58)
- [response_synthesizers.md](file://docs/src/content/docs/framework/module_guides/querying/response_synthesizers/response_synthesizers.md#L1-L69)

**Section sources**
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L1-L31)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L53-L322)
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L33-L152)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L4-L58)
- [response_synthesizers.md](file://docs/src/content/docs/framework/module_guides/querying/response_synthesizers/response_synthesizers.md#L1-L69)

## Core Components
- ContextOnly: A synthesizer that concatenates retrieved text chunks into a single response string without prompting an LLM.
- BaseSynthesizer: The abstract base class defining the synthesis interface and shared orchestration logic.
- Factory: Creates synthesizer instances based on the selected ResponseMode, including CONTEXT_ONLY.
- ResponseMode: Enumeration that includes CONTEXT_ONLY as a valid synthesis mode.

Key characteristics:
- Zero LLM calls during synthesis.
- Minimal processing overhead.
- Straightforward concatenation with a newline separator between chunks.

**Section sources**
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L8-L31)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L53-L126)
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L145-L149)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L45-L46)

## Architecture Overview
The Context Only strategy participates in the unified synthesis pipeline but bypasses LLM invocation and iterative refinement. It receives text chunks from retrieval, applies optional streaming behavior, and returns a concatenated string.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant Factory as "get_response_synthesizer"
participant Mode as "ResponseMode.CONTEXT_ONLY"
participant Synth as "ContextOnly"
participant Base as "BaseSynthesizer"
Client->>Factory : Request synthesizer with response_mode="context_only"
Factory->>Mode : Resolve mode
Factory-->>Client : ContextOnly instance
Client->>Base : synthesize(query, nodes)
Base->>Synth : get_response(query_str, text_chunks)
Synth-->>Base : Concatenated string
Base-->>Client : Response object with concatenated text
```

**Diagram sources**
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L145-L149)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L45-L46)
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L16-L30)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L193-L256)

## Detailed Component Analysis

### ContextOnly Implementation
ContextOnly extends BaseSynthesizer and implements a minimal synthesis process:
- Prompts: Returns an empty prompt dictionary (no prompts required).
- get_response: Concatenates all text chunks with a double newline separator.
- aget_response: Asynchronous variant with identical logic.

```mermaid
classDiagram
class BaseSynthesizer {
+synthesize(query, nodes, ...)
+asynthesize(query, nodes, ...)
+get_response(...)
+aget_response(...)
-_prepare_response_output(...)
}
class ContextOnly {
+_get_prompts()
+_update_prompts(prompts)
+get_response(query_str, text_chunks, **kwargs)
+aget_response(query_str, text_chunks, **kwargs)
}
BaseSynthesizer <|-- ContextOnly
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L53-L126)
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L8-L31)

**Section sources**
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L8-L31)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L108-L126)

### Synthesis Pipeline Orchestration
BaseSynthesizer orchestrates synthesis:
- Validates input nodes and handles empty-node cases.
- Converts nodes to text chunks using metadata modes.
- Invokes the synthesizer’s get_response or aget_response.
- Wraps the result into a Response object with source metadata.

```mermaid
flowchart TD
Start(["synthesize/asynthesize"]) --> CheckNodes["Check nodes length"]
CheckNodes --> |Empty| EmptyResp["Return empty response"]
CheckNodes --> |Non-empty| Prepare["Prepare text_chunks from nodes"]
Prepare --> CallSynth["Call get_response/aget_response"]
CallSynth --> Wrap["Wrap into Response object"]
Wrap --> End(["Return response"])
EmptyResp --> End
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L193-L256)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L258-L322)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L193-L256)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L258-L322)

### Factory and Mode Resolution
The factory resolves ResponseMode.CONTEXT_ONLY to a ContextOnly instance and passes through configuration like streaming and callback manager.

```mermaid
flowchart TD
A["get_response_synthesizer"] --> B{"response_mode == CONTEXT_ONLY?"}
B --> |Yes| C["Return ContextOnly(...)"]
B --> |No| D["Return other synthesizer"]
```

**Diagram sources**
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L145-L149)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L45-L46)

**Section sources**
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L33-L152)
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L4-L58)

## Dependency Analysis
ContextOnly depends on BaseSynthesizer for orchestration and shares the same configuration surface (streaming, callback manager). The factory ties the mode enumeration to the synthesizer implementation.

```mermaid
graph LR
Type["ResponseMode enum"] --> Factory["get_response_synthesizer"]
Factory --> ContextOnly["ContextOnly"]
Base["BaseSynthesizer"] --> ContextOnly
```

**Diagram sources**
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L4-L58)
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L145-L149)
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L8-L31)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L53-L126)

**Section sources**
- [type.py](file://llama-index-core/llama_index/core/response_synthesizers/type.py#L4-L58)
- [factory.py](file://llama-index-core/llama_index/core/response_synthesizers/factory.py#L33-L152)
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L8-L31)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L53-L126)

## Performance Considerations
- Speed: Context Only avoids LLM calls and iterative processing, resulting in minimal latency and reduced cost.
- Throughput: Fewer model calls improve throughput for high-volume retrieval tasks.
- Memory: Concatenation scales with the number of chunks; ensure downstream consumers can handle large concatenated strings.
- Quality: Since no LLM synthesis occurs, the response quality is determined solely by the relevance and ordering of retrieved chunks.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- Empty nodes: BaseSynthesizer handles empty node lists by returning an empty response; confirm that retrieval is functioning as expected.
- Streaming behavior: If streaming is enabled, ensure downstream consumers handle the concatenated stream appropriately.
- Chunk formatting: The concatenation uses a double newline separator; verify that this separator aligns with downstream expectations.

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L206-L226)
- [base.py](file://llama-index-core/llama_index/core/response_synthesizers/base.py#L271-L291)
- [context_only.py](file://llama-index-core/llama_index/core/response_synthesizers/context_only.py#L22-L30)

## Conclusion
Context Only synthesis provides a minimal, efficient pathway for returning retrieved context directly to users or systems. It trades iterative refinement and LLM-driven synthesis for simplicity, speed, and cost-effectiveness, making it well-suited for straightforward queries and scenarios where raw context is sufficient.

[No sources needed since this section summarizes without analyzing specific files]