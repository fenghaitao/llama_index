# Specialized LLM Providers

<cite>
**Referenced Files in This Document**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py)
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
This document explains specialized Large Language Model (LLM) providers integrated in LlamaIndex: Mistral AI, DeepSeek, Fireworks, Together, and Azure Inference. It focuses on provider-specific model architectures, unique features, and use-case optimizations. It also covers configuration examples, provider-specific parameters, temperature settings, output formatting, and guidance for selecting models for reasoning, code generation, and research-oriented tasks. Pricing models, usage tracking, and billing considerations are addressed conceptually, with practical steps for monitoring and cost control.

## Project Structure
The specialized providers are implemented as separate packages under the LlamaIndex integrations tree. Each provider exposes a dedicated LLM class that adheres to LlamaIndex’s LLM interface, enabling drop-in replacement and consistent usage patterns.

```mermaid
graph TB
subgraph "LlamaIndex Integrations"
MI["Mistral AI<br/>llama_index/llms/mistralai/base.py"]
DS["DeepSeek<br/>llama_index/llms/deepseek/base.py"]
FW["Fireworks<br/>llama_index/llms/fireworks/base.py"]
TG["Together<br/>llama_index/llms/together/base.py"]
AZ["Azure Inference<br/>llama_index/llms/azure_inference/base.py"]
end
MI --> |"Uses Mistral client"| MI
DS --> |"OpenAI-like adapter"| DS
FW --> |"OpenAI-like adapter"| FW
TG --> |"OpenAI-like adapter"| TG
AZ --> |"Azure AI Inference SDK"| AZ
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L178-L757)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L8-L56)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L18-L128)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L7-L52)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L147-L601)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L1-L757)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L1-L56)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L1-L128)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L1-L52)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L1-L601)

## Core Components
- Mistral AI: Provides reasoning-aware models with “thinking” output parsing, function-calling support, and multimodal content handling. Includes specialized code model support via Fill-In-the-Middle.
- DeepSeek: OpenAI-like adapter configured against DeepSeek’s API base, with function-calling detection and context window sizing.
- Fireworks: OpenAI-like adapter with configurable context windows and function-calling detection, plus custom model metadata support.
- Together: OpenAI-like adapter configured against Together’s API base.
- Azure Inference: Azure AI Inference client wrapper supporting chat, streaming, function/tool calling, and multiple authentication modes.

Each provider class exposes:
- Standard LLM methods: chat, complete, stream_chat, stream_complete, async variants.
- Metadata reporting: context window, output token limits, chat capability, function-calling capability.
- Provider-specific parameters: API keys, endpoints, base URLs, model names, and optional headers.

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L178-L757)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L8-L56)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L18-L128)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L7-L52)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L147-L601)

## Architecture Overview
The providers follow a layered pattern:
- LlamaIndex LLM interface: unified chat/completion/streaming APIs.
- Provider adapters: OpenAI-like or SDK-specific clients.
- Provider SDKs: Mistral client, Azure AI Inference SDK, or OpenAI-compatible APIs.

```mermaid
sequenceDiagram
participant App as "Application"
participant LLM as "Provider LLM Class"
participant Adapter as "Adapter Layer"
participant SDK as "Provider SDK/HTTP"
App->>LLM : chat(messages, kwargs)
LLM->>Adapter : normalize messages/format
Adapter->>SDK : send request (chat/completions/stream)
SDK-->>Adapter : response chunks/deltas/raw
Adapter-->>LLM : normalized ChatResponse
LLM-->>App : ChatResponse (text/thinking/tool_calls)
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L358-L491)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L352-L472)

## Detailed Component Analysis

### Mistral AI
- Specialized features:
  - Reasoning models with “thinking” content extraction and optional display toggle.
  - Function-calling model detection and tool call injection.
  - Multimodal content conversion (text, images, thinking).
  - Fill-In-the-Middle for code completion with a dedicated code model.
- Key parameters:
  - model, temperature, max_tokens, timeout, max_retries, random_seed, additional_kwargs, endpoint, show_thinking.
- Use cases:
  - Research-oriented tasks, reasoning chains, and multimodal Q&A.
- Configuration highlights:
  - Environment variables: MISTRAL_API_KEY, MISTRAL_ENDPOINT.
  - Metadata exposes context window and function-calling capability.

```mermaid
classDiagram
class MistralAI {
+string model
+float temperature
+int max_tokens
+float timeout
+int max_retries
+int random_seed
+dict additional_kwargs
+bool show_thinking
+metadata() LLMMetadata
+chat(messages) ChatResponse
+complete(prompt) CompletionResponse
+stream_chat(messages) ChatResponseGen
+stream_complete(prompt) CompletionResponseGen
+achat(messages) ChatResponse
+acomplete(prompt) CompletionResponse
+astream_chat(messages) ChatResponseAsyncGen
+astream_complete(prompt) CompletionResponseAsyncGen
+fill_in_middle(prompt, suffix, stop) CompletionResponse
}
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L178-L757)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L178-L757)

### DeepSeek
- Specialized features:
  - OpenAI-like interface with function-calling detection and context window sizing.
  - Uses a provider-specific API base URL.
- Key parameters:
  - model, api_key, api_base, is_chat_model, is_function_calling_model, context_window.
- Use cases:
  - General-purpose chat and code tasks with OpenAI-like ergonomics.
- Configuration highlights:
  - Environment variable: DEEPSEEK_API_KEY.
  - Function-calling and context window derived from utilities.

```mermaid
classDiagram
class DeepSeek {
+string model
+string api_key
+string api_base
+bool is_chat_model
+bool is_function_calling_model
+int context_window
+metadata() LLMMetadata
+chat(messages) ChatResponse
+complete(prompt) CompletionResponse
+stream_chat(messages) ChatResponseGen
+stream_complete(prompt) CompletionResponseGen
+achat(messages) ChatResponse
+acomplete(prompt) CompletionResponse
+astream_chat(messages) ChatResponseAsyncGen
+astream_complete(prompt) CompletionResponseAsyncGen
}
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L8-L56)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L8-L56)

### Fireworks
- Specialized features:
  - OpenAI-like interface with configurable context window and function-calling detection.
  - Supports custom models and explicit context window/function-calling toggles.
- Key parameters:
  - model, temperature, max_tokens, additional_kwargs, max_retries, api_base, api_key, default_headers, system_prompt, context_window, is_function_calling.
- Use cases:
  - Multi-modal instruction-following and specialized model deployments with extended context.
- Configuration highlights:
  - Environment variables: FIREWORKS_API_KEY, FIREWORKS_API_BASE.
  - Metadata computed from model name or defaults.

```mermaid
classDiagram
class Fireworks {
+string model
+float temperature
+int max_tokens
+dict additional_kwargs
+int max_retries
+string api_base
+string api_key
+dict default_headers
+string system_prompt
+int context_window
+bool is_function_calling
+metadata() LLMMetadata
+chat(messages) ChatResponse
+complete(prompt) CompletionResponse
+stream_chat(messages) ChatResponseGen
+stream_complete(prompt) CompletionResponseGen
+achat(messages) ChatResponse
+acomplete(prompt) CompletionResponse
+astream_chat(messages) ChatResponseAsyncGen
+astream_complete(prompt) CompletionResponseAsyncGen
}
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L18-L128)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L18-L128)

### Together
- Specialized features:
  - OpenAI-like adapter configured against Together’s API base.
- Key parameters:
  - model, api_key, api_base, is_chat_model.
- Use cases:
  - Aggregating and running diverse open-weight models via Together’s platform.
- Configuration highlights:
  - Environment variable: TOGETHER_API_KEY.
  - API base preconfigured to Together’s endpoint.

```mermaid
classDiagram
class TogetherLLM {
+string model
+string api_key
+string api_base
+bool is_chat_model
+metadata() LLMMetadata
+chat(messages) ChatResponse
+complete(prompt) CompletionResponse
+stream_chat(messages) ChatResponseGen
+stream_complete(prompt) CompletionResponseGen
+achat(messages) ChatResponse
+acomplete(prompt) CompletionResponse
+astream_chat(messages) ChatResponseAsyncGen
+astream_complete(prompt) CompletionResponseAsyncGen
}
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L7-L52)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L7-L52)

### Azure Inference
- Specialized features:
  - Azure AI Inference SDK integration with chat, streaming, and function/tool calling.
  - Multiple authentication modes: API key or Azure Identity credentials.
  - Metadata retrieval from endpoint when supported.
- Key parameters:
  - endpoint, credential, temperature, max_tokens, model_name, api_version, model_kwargs.
- Use cases:
  - Enterprise-grade deployments with Azure identity and compliance controls.
- Configuration highlights:
  - Environment variables: AZURE_INFERENCE_ENDPOINT, AZURE_INFERENCE_CREDENTIAL.
  - Tool choice presets for required vs auto tool selection.

```mermaid
classDiagram
class AzureAICompletionsModel {
+string model_name
+float temperature
+int max_tokens
+string seed
+dict model_kwargs
+metadata() LLMMetadata
+chat(messages) ChatResponse
+complete(prompt) CompletionResponse
+stream_chat(messages) ChatResponseGen
+stream_complete(prompt) CompletionResponseGen
+achat(messages) ChatResponse
+acomplete(prompt) CompletionResponse
+astream_chat(messages) ChatResponseAsyncGen
+astream_complete(prompt) CompletionResponseAsyncGen
+chat_with_tools(tools, user_msg, chat_history, tool_required) ChatResponse
+achat_with_tools(tools, user_msg, chat_history, tool_required) ChatResponse
+get_tool_calls_from_response(response) ToolSelection[]
}
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L147-L601)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L147-L601)

## Dependency Analysis
- Mistral AI depends on the Mistral client and supports function-calling and multimodal inputs.
- DeepSeek, Fireworks, and Together rely on OpenAI-like adapters, enabling a uniform interface while pointing to provider-specific bases.
- Azure Inference integrates the Azure AI Inference SDK and supports both sync and async flows.

```mermaid
graph LR
MA["MistralAI"] --> MC["Mistral client"]
DS["DeepSeek"] --> OAI["OpenAI-like adapter"]
FW["Fireworks"] --> OAI
TG["TogetherLLM"] --> OAI
AZ["AzureAICompletionsModel"] --> ASDK["Azure AI Inference SDK"]
```

**Diagram sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L56-L68)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L4-L5)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L11)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L4)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L46-L51)

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L56-L68)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-deepseek/llama_index/llms/deepseek/base.py#L4-L5)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-fireworks/llama_index/llms/fireworks/base.py#L11)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-together/llama_index/llms/together/base.py#L4)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L46-L51)

## Performance Considerations
- Temperature tuning:
  - Lower values (e.g., near zero) increase determinism for reasoning and code tasks.
  - Higher values (up to 1.0) increase creativity for open-ended tasks.
- Context window:
  - Respect provider context limits to avoid truncation and extra round-trips.
  - For Fireworks and DeepSeek, explicitly set context_window when using custom models.
- Streaming:
  - Prefer streaming for long responses to reduce perceived latency and enable early termination.
- Retries and timeouts:
  - Configure max_retries and timeout to balance reliability and responsiveness.
- Function-calling:
  - Enable function-calling models to reduce hallucinations and improve tool use accuracy.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- Authentication errors:
  - Ensure environment variables are set or passed explicitly: MISTRAL_API_KEY, DEEPSEEK_API_KEY, FIREWORKS_API_KEY, TOGETHER_API_KEY, AZURE_INFERENCE_ENDPOINT, AZURE_INFERENCE_CREDENTIAL.
- Endpoint configuration:
  - Mistral AI allows overriding the endpoint via environment or constructor parameter.
  - Azure Inference requires endpoint and credential; missing either raises an error.
- Function-calling mismatches:
  - Verify model supports function-calling; providers expose metadata indicating capability.
- Tool call validation:
  - Some providers enforce single tool call per response; ensure allow_parallel_tool_calls is aligned with provider behavior.
- Pricing and usage tracking:
  - Use provider dashboards to monitor usage and costs.
  - Track tokens consumed per request/response and aggregate across runs.
  - Implement cost-aware batching and retry policies to minimize wasted compute.

**Section sources**
- [base.py](file://llama-index-integrations/llms/llama-index-llms-mistralai/llama_index/llms/mistralai/base.py#L262-L273)
- [base.py](file://llama-index-integrations/llms/llama-index-llms-azure-inference/llama_index/llms/azure_inference/base.py#L243-L266)

## Conclusion
These specialized providers bring distinct strengths to LlamaIndex:
- Mistral AI excels in reasoning and multimodal tasks with thinking support and function-calling.
- DeepSeek and Together offer OpenAI-like ergonomics with provider-specific bases and function-calling detection.
- Fireworks enables flexible context windows and function-calling for specialized models.
- Azure Inference integrates enterprise-grade authentication and compliance with robust streaming and tool-calling support.

Select models based on task type (reasoning, code, research), environment constraints (context, latency), and provider features. Combine temperature tuning, streaming, and function-calling to optimize performance and cost.