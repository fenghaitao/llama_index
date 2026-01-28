# Query Transformations

<cite>
**Referenced Files in This Document**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py)
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py)
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py)
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py)
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py)
- [types.py](file://llama-index-core/llama_index/core/question_gen/types.py)
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
This document provides comprehensive API documentation for Query Transformation systems in the repository. It focuses on:
- Transform query engines that apply query transformations before delegating to downstream query engines
- Retry mechanisms that evaluate responses and reattempt queries with transformed inputs
- Question generation components that produce structured sub-questions for multi-step retrieval
- Dynamic query modification via feedback-driven transformations
- Practical guidance for implementing custom transformers, configuring retry strategies, and generating synthetic queries
- Performance optimization and error handling patterns for transformation pipelines

## Project Structure
The query transformation ecosystem spans three primary areas:
- Query Engine Wrappers: TransformQueryEngine, RetryQueryEngine, RetrySourceQueryEngine
- Query Transform Implementations: HyDE, Decompose, StepDecompose, ImageOutput, Identity
- Question Generation: LLM-backed generator producing structured sub-questions

```mermaid
graph TB
subgraph "Query Engines"
TQE["TransformQueryEngine"]
RQE["RetryQueryEngine"]
RSQE["RetrySourceQueryEngine"]
end
subgraph "Transforms"
BQT["BaseQueryTransform"]
HYDE["HyDEQueryTransform"]
DECOMP["DecomposeQueryTransform"]
STEP["StepDecomposeQueryTransform"]
IMG["ImageOutputQueryTransform"]
ID["IdentityQueryTransform"]
FB["FeedbackQueryTransformation"]
end
subgraph "Question Gen"
QGEN["LLMQuestionGenerator"]
QTYPE["BaseQuestionGenerator/SubQuestion"]
end
TQE --> BQT
BQT --> HYDE
BQT --> DECOMP
BQT --> STEP
BQT --> IMG
BQT --> ID
RQE --> FB
RSQE --> FB
QGEN --> QTYPE
```

**Diagram sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L11-L95)
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L22-L147)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L24-L93)
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L30-L322)
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L28-L118)
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L20-L98)
- [types.py](file://llama-index-core/llama_index/core/question_gen/types.py#L26-L42)

**Section sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L1-L95)
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L1-L322)

## Core Components
- TransformQueryEngine: Applies a BaseQueryTransform to a QueryBundle before invoking retrieve/synthesize/query on a downstream query engine.
- RetryQueryEngine: Evaluates the response via a BaseEvaluator; if failing, constructs a new query using FeedbackQueryTransformation and retries up to max_retries.
- RetrySourceQueryEngine: Evaluates per-source-node relevance; filters to a new index and retries with a fresh RetrieverQueryEngine pipeline.
- BaseQueryTransform family: HyDEQueryTransform, DecomposeQueryTransform, StepDecomposeQueryTransform, ImageOutputQueryTransform, IdentityQueryTransform.
- FeedbackQueryTransformation: Augments the query with evaluation feedback; optionally resynthesizes the query using an LLM.
- LLMQuestionGenerator: Generates structured sub-questions from a QueryBundle and tool metadata.

**Section sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L11-L95)
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L22-L147)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L24-L93)
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L30-L322)
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L28-L118)
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L20-L98)

## Architecture Overview
The transformation pipeline integrates query engines, transforms, and evaluators to iteratively refine queries and responses.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant TQE as "TransformQueryEngine"
participant BQT as "BaseQueryTransform"
participant QE as "Wrapped QueryEngine"
Client->>TQE : "query(QueryBundle)"
TQE->>BQT : "run(query_bundle, metadata)"
BQT-->>TQE : "QueryBundle'"
TQE->>QE : "query(QueryBundle')"
QE-->>TQE : "RESPONSE"
TQE-->>Client : "RESPONSE"
```

**Diagram sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L82-L95)
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L50-L73)

```mermaid
sequenceDiagram
participant Client as "Caller"
participant RQE as "RetryQueryEngine"
participant QE as "Wrapped QueryEngine"
participant EVAL as "BaseEvaluator"
participant FB as "FeedbackQueryTransformation"
Client->>RQE : "query(QueryBundle)"
RQE->>QE : "_query(QueryBundle)"
QE-->>RQE : "RESPONSE"
RQE->>EVAL : "evaluate_response(query, response)"
EVAL-->>RQE : "Evaluation(passing)"
alt "passing"
RQE-->>Client : "RESPONSE"
else "not passing"
RQE->>FB : "run(QueryBundle, {evaluation})"
FB-->>RQE : "QueryBundle'"
RQE->>RQE : "recurse with max_retries-1"
RQE-->>Client : "RESPONSE"
end
```

**Diagram sources**
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L50-L76)
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L60-L93)

## Detailed Component Analysis

### TransformQueryEngine
- Purpose: Apply a BaseQueryTransform to a QueryBundle prior to invoking retrieve/synthesize/query on a downstream query engine.
- Key behaviors:
  - Delegates prompt module composition to contained components.
  - Runs transform on the QueryBundle before forwarding to the underlying engine.
  - Supports sync and async query pathways.

```mermaid
classDiagram
class TransformQueryEngine {
-_query_engine
-_query_transform
-_transform_metadata
+retrieve(query_bundle)
+synthesize(query_bundle, nodes, additional_source_nodes)
+asynthesize(query_bundle, nodes, additional_source_nodes)
+_query(query_bundle)
+_get_prompt_modules()
}
class BaseQueryTransform {
<<abstract>>
+run(query_bundle_or_str, metadata)
+_run(query_bundle, metadata)
}
TransformQueryEngine --> BaseQueryTransform : "uses"
```

**Diagram sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L11-L95)
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L30-L74)

**Section sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L11-L95)

### RetryQueryEngine
- Purpose: Evaluate responses and retry with transformed queries when evaluations fail.
- Key behaviors:
  - Uses a BaseEvaluator to assess responses.
  - On failure, constructs a new query via FeedbackQueryTransformation and recurses with reduced retries.
  - Supports synchronous query; asynchronous path falls back to synchronous.

```mermaid
flowchart TD
Start(["Start _query"]) --> CallQE["_query(QueryBundle)"]
CallQE --> EvalResp["Build Response object"]
EvalResp --> EvalPass{"Evaluation passing?"}
EvalPass --> |Yes| ReturnOrig["Return original response"]
EvalPass --> |No| CheckRetries{"max_retries > 0?"}
CheckRetries --> |No| ReturnOrig
CheckRetries --> |Yes| FB["FeedbackQueryTransformation.run(...)"]
FB --> Recurse["Instantiate RetryQueryEngine(max_retries-1)"]
Recurse --> CallQE
```

**Diagram sources**
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L50-L76)
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L60-L93)

**Section sources**
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L22-L147)

### RetrySourceQueryEngine
- Purpose: Filter source nodes by evaluator quality and rebuild a retriever pipeline for retries.
- Key behaviors:
  - Evaluates each source node independently.
  - Builds a new SummaryIndex from passing nodes and a new RetrieverQueryEngine.
  - Retries with reduced max_retries until success or exhaustion.

```mermaid
flowchart TD
Start(["Start _query"]) --> CallQE["_query(QueryBundle)"]
CallQE --> BuildResp["Build Response object"]
BuildResp --> EvalPass{"Evaluation passing?"}
EvalPass --> |Yes| ReturnOrig["Return original response"]
EvalPass --> |No| CheckRetries{"max_retries > 0?"}
CheckRetries --> |No| ReturnOrig
CheckRetries --> |Yes| EvalNodes["Evaluate each source node"]
EvalNodes --> HasPass{"Any passing nodes?"}
HasPass --> |No| RaiseErr["Raise ValueError"]
HasPass --> |Yes| BuildIdx["Build SummaryIndex from passing nodes"]
BuildIdx --> NewRE["New RetrieverQueryEngine"]
NewRE --> Recurse["Instantiate RetrySourceQueryEngine(max_retries-1)"]
Recurse --> CallQE
```

**Diagram sources**
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L46-L93)

**Section sources**
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L24-L93)

### BaseQueryTransform Implementations
- HyDEQueryTransform: Generates a hypothetical document via LLM and augments embedding strings.
- DecomposeQueryTransform: Rewrites a query into a more index-friendly form using a prompt and index summary.
- StepDecomposeQueryTransform: Extends decomposition with previous reasoning context.
- ImageOutputQueryTransform: Injects instructions to format image outputs.
- IdentityQueryTransform: No-op transform returning the input unchanged.

```mermaid
classDiagram
class BaseQueryTransform {
<<abstract>>
+run(query_bundle_or_str, metadata)
+_run(query_bundle, metadata)
}
class HyDEQueryTransform {
-_llm
-_hyde_prompt
-_include_original
+_run(query_bundle, metadata)
}
class DecomposeQueryTransform {
-_llm
-_decompose_query_prompt
-verbose
+_run(query_bundle, metadata)
}
class StepDecomposeQueryTransform {
-_llm
-_step_decompose_query_prompt
-verbose
+_run(query_bundle, metadata)
}
class ImageOutputQueryTransform {
-_width
-_query_prompt
+_run(query_bundle, metadata)
}
class IdentityQueryTransform {
+_run(query_bundle, metadata)
}
BaseQueryTransform <|-- HyDEQueryTransform
BaseQueryTransform <|-- DecomposeQueryTransform
BaseQueryTransform <|-- StepDecomposeQueryTransform
BaseQueryTransform <|-- ImageOutputQueryTransform
BaseQueryTransform <|-- IdentityQueryTransform
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L30-L322)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L76-L322)

### FeedbackQueryTransformation
- Purpose: Augment or resynthesize a query based on evaluation feedback.
- Key behaviors:
  - Validates presence of evaluation, response, and feedback.
  - Optionally resynthesizes the query using an LLM and a dedicated prompt.
  - Returns a new QueryBundle with augmented query string and embedding context.

```mermaid
flowchart TD
Start(["Start _run"]) --> GetEval["Ensure Evaluation present and valid"]
GetEval --> Choice{"feedback is YES/NO?"}
Choice --> |Yes| AppendFB["Append previous response context"]
Choice --> |No| Resynth{"should_resynthesize_query?"}
Resynth --> |Yes| LLMResynth["LLM resynthesize query"]
Resynth --> |No| KeepOrig["Keep original query"]
LLMResynth --> BuildQ["Build feedback+query string"]
KeepOrig --> BuildQ
AppendFB --> BuildQ
BuildQ --> Return["Return QueryBundle with custom embedding"]
```

**Diagram sources**
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L60-L118)

**Section sources**
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L28-L118)

### Question Generators and Relationship to Query Engines
- LLMQuestionGenerator: Produces a list of SubQuestion objects from a QueryBundle and tool metadata using a structured prompt and output parser.
- Relationship: Sub-queries generated by a question generator can be fed into a TransformQueryEngine with a DecomposeQueryTransform to create a multi-step retrieval pipeline. Alternatively, the generator can drive a sub-question query engine pattern where each sub-question is answered independently and synthesized.

```mermaid
sequenceDiagram
participant Client as "Caller"
participant QGEN as "LLMQuestionGenerator"
participant LLM as "LLM"
participant OParser as "Output Parser"
participant TQE as "TransformQueryEngine"
participant QE as "Downstream QueryEngine"
Client->>QGEN : "generate(tools, QueryBundle)"
QGEN->>LLM : "predict(prompt, tools_str, query_str)"
LLM-->>QGEN : "raw prediction"
QGEN->>OParser : "parse(prediction)"
OParser-->>QGEN : "List[SubQuestion]"
QGEN-->>Client : "List[SubQuestion]"
Client->>TQE : "query(sub_question)"
TQE->>QE : "query(sub_question)"
QE-->>TQE : "RESPONSE"
TQE-->>Client : "RESPONSE"
```

**Diagram sources**
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L67-L97)
- [types.py](file://llama-index-core/llama_index/core/question_gen/types.py#L11-L42)
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L82-L95)

**Section sources**
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L20-L98)
- [types.py](file://llama-index-core/llama_index/core/question_gen/types.py#L26-L42)

## Dependency Analysis
- TransformQueryEngine depends on BaseQueryTransform and a downstream BaseQueryEngine.
- RetryQueryEngine depends on a BaseEvaluator and FeedbackQueryTransformation.
- RetrySourceQueryEngine depends on a RetrieverQueryEngine, BaseEvaluator, and builds a SummaryIndex for retries.
- Question generators depend on LLMs, prompts, and output parsers to produce structured sub-questions.

```mermaid
graph LR
TQE["TransformQueryEngine"] --> BQT["BaseQueryTransform"]
TQE --> QE["BaseQueryEngine"]
RQE["RetryQueryEngine"] --> EVAL["BaseEvaluator"]
RQE --> FB["FeedbackQueryTransformation"]
RQE --> QE
RSQE["RetrySourceQueryEngine"] --> RE["RetrieverQueryEngine"]
RSQE --> EVAL
RSQE --> QE
QGEN["LLMQuestionGenerator"] --> LLM["LLM"]
QGEN --> PROMPT["PromptTemplate"]
QGEN --> OPARSER["OutputParser"]
```

**Diagram sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L11-L95)
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L22-L147)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L24-L93)
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L20-L98)

**Section sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L1-L95)
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L1-L147)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L1-L93)
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L1-L98)

## Performance Considerations
- Minimize redundant transformations: Prefer identity transforms only when necessary; avoid repeated HyDE generations for identical queries.
- Control retry depth: Set max_retries conservatively to prevent exponential cost growth; consider early exit conditions.
- Filter sources efficiently: In RetrySourceQueryEngine, pre-filter nodes and limit the number of evaluated contexts.
- Batch sub-questions: When using question generators, batch and parallelize downstream query executions where supported.
- Cache embeddings: For repeated queries, reuse custom embedding strings from HyDE to reduce LLM calls.
- Streaming vs. blocking: Avoid unnecessary async paths; synchronous query paths are simpler and often sufficient.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- TransformQueryEngine
  - Symptom: No effect on query.
    - Cause: Identity transform or missing metadata.
    - Action: Verify transform type and metadata passed to TransformQueryEngine.
  - Symptom: Unexpected embedding strings.
    - Cause: HyDE include_original flag or custom embedding_strs.
    - Action: Inspect custom_embedding_strs after transform run.

- RetryQueryEngine
  - Symptom: Infinite recursion or excessive retries.
    - Cause: max_retries not decreasing or evaluation always failing.
    - Action: Increase success criteria in evaluator; ensure FeedbackQueryTransformation receives a valid Evaluation.
  - Symptom: Async path not supported.
    - Action: Use synchronous query; async fallback invokes synchronous logic.

- RetrySourceQueryEngine
  - Symptom: No passing nodes found.
    - Cause: Evaluator too strict or poor-quality sources.
    - Action: Relax evaluation thresholds or adjust source selection strategy.

- FeedbackQueryTransformation
  - Symptom: Missing evaluation or invalid feedback.
    - Cause: Evaluation not provided or missing response/feedback fields.
    - Action: Ensure metadata contains a proper Evaluation object with response and feedback.

- Question Generators
  - Symptom: Parsing errors in sub-questions.
    - Cause: Output parser mismatch or prompt misconfiguration.
    - Action: Validate prompt template and output parser compatibility.

**Section sources**
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L11-L95)
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L50-L76)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L46-L93)
- [feedback_transform.py](file://llama-index-core/llama_index/core/indices/query/query_transform/feedback_transform.py#L60-L118)
- [llm_generators.py](file://llama-index-core/llama_index/core/question_gen/llm_generators.py#L67-L97)

## Conclusion
The query transformation system offers a modular framework to enhance retrieval accuracy and reliability:
- TransformQueryEngine enables pluggable query-time modifications.
- Retry mechanisms with evaluators and feedback-driven transformations improve robustness.
- Question generators supply structured sub-questions for complex tasks.
Adopting best practices around caching, batching, and conservative retry limits yields significant performance gains while maintaining correctness.

[No sources needed since this section summarizes without analyzing specific files]

## Appendices

### Implementing a Custom Query Transformer
- Extend BaseQueryTransform and implement _run to modify QueryBundle.
- Optionally override _get_prompts and _update_prompts to integrate with prompt management.
- Example reference paths:
  - [Base class definition](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L30-L74)
  - [Example implementations](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L76-L322)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L30-L322)

### Configuring Retry Strategies
- Choose RetryQueryEngine for evaluator-based feedback loops.
- Choose RetrySourceQueryEngine for source-node filtering and re-retrieval.
- Configure max_retries and evaluator thresholds to balance quality and latency.
- Example reference paths:
  - [RetryQueryEngine](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L22-L147)
  - [RetrySourceQueryEngine](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L24-L93)

**Section sources**
- [retry_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_query_engine.py#L22-L147)
- [retry_source_query_engine.py](file://llama-index-core/llama_index/core/query_engine/retry_source_query_engine.py#L24-L93)

### Generating Synthetic Queries
- Use HyDEQueryTransform to generate hypothetical documents and augment embeddings.
- Combine with TransformQueryEngine to inject synthetic queries into downstream engines.
- Example reference paths:
  - [HyDE implementation](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L96-L151)
  - [Transform wrapper](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L46-L65)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/indices/query/query_transform/base.py#L96-L151)
- [transform_query_engine.py](file://llama-index-core/llama_index/core/query_engine/transform_query_engine.py#L46-L65)