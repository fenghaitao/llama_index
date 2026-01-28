# Audio Processing

<cite>
**Referenced Files in This Document**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/__init__.py)
- [test_readers_whisper.py](file://llama-index-integrations/readers/llama-index-readers-whisper/tests/test_readers_whisper.py)
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
This document explains audio processing capabilities in LlamaIndex multi-modal systems with a focus on speech-to-text conversion, audio transcription, voice activity detection, integration with ASR models, speaker diarization, audio feature extraction, audio embedding generation, acoustic modeling, and noise reduction. It also covers practical examples such as voice-enabled retrieval-augmented generation (RAG), audio question answering, and speech synthesis integration, along with real-time audio processing, latency optimization, multi-language support, audio quality assessment, background noise filtering, and audio format compatibility.

## Project Structure
The repository organizes audio-related functionality across:
- Core voice agent abstractions under the voice agents module
- Integration-specific audio interfaces for providers (OpenAI and Gemini Live)
- A Whisper-based reader for audio transcription
- Tests validating Whisper reader behavior

```mermaid
graph TB
subgraph "Core Voice Agents"
VA_Base["BaseVoiceAgent<br/>(abstract)"]
VA_Interface["BaseVoiceAgentInterface<br/>(abstract)"]
end
subgraph "Provider Integrations"
OA_Int["OpenAIVoiceAgentInterface<br/>(PyAudio)"]
GL_Int["GeminiLiveVoiceAgentInterface<br/>(AsyncSession)"]
end
subgraph "Audio Transcription"
WR["WhisperReader<br/>(OpenAI Whisper)"]
end
VA_Base --> VA_Interface
OA_Int --> VA_Interface
GL_Int --> VA_Interface
WR --> |"reads audio files"| VA_Base
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L11-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L5-L116)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L21-L126)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L19-L150)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L15-L175)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L1-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L1-L116)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L1-L126)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L1-L150)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L1-L175)

## Core Components
- Voice Agent Abstractions
  - BaseVoiceAgent defines asynchronous lifecycle methods for starting, sending audio, handling messages, interrupting, and stopping, plus utilities to export messages and events.
  - BaseVoiceAgentInterface defines the contract for audio input/output, including callbacks, start/stop/interrupt, output processing, and receiving audio data.

- Provider Interfaces
  - OpenAIVoiceAgentInterface integrates PyAudio for real-time capture/playback, buffering, and callback-driven streaming with configurable sample rate and chunk size.
  - GeminiLiveVoiceAgentInterface integrates with Google GenAI live sessions, managing separate input/output queues and sample rates for sending/receiving audio.

- WhisperReader
  - Provides synchronous and asynchronous audio transcription via the OpenAI Whisper API, supporting local files, bytes, and file-like objects, with configurable language and prompt.

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L11-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L5-L116)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L21-L126)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L19-L150)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L15-L175)

## Architecture Overview
The audio pipeline combines voice agent interfaces with transcription and optional downstream RAG or synthesis.

```mermaid
sequenceDiagram
participant Mic as "Microphone"
participant OA as "OpenAIVoiceAgentInterface"
participant GL as "GeminiLiveVoiceAgentInterface"
participant WR as "WhisperReader"
participant VA as "BaseVoiceAgent"
Mic->>OA : "Captures audio chunks"
OA-->>VA : "send(audio_chunk)"
VA->>WR : "load_data(bytes or file)"
WR-->>VA : "Document(text=transcription)"
VA-->>GL : "Optional downstream processing"
GL-->>Mic : "Plays synthesized audio"
```

**Diagram sources**
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L40-L121)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L41-L149)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L148-L174)
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L56-L112)

## Detailed Component Analysis

### Voice Agent Abstractions
- BaseVoiceAgent
  - Responsibilities: orchestrate websocket-backed voice services, manage tool use, maintain conversation history and events, and expose lifecycle hooks for async operation.
  - Key methods: start, send, handle_message, interrupt, stop, export_messages, export_events.

- BaseVoiceAgentInterface
  - Responsibilities: define the audio I/O contract for input/output devices, including callbacks, streaming control, and data reception.

```mermaid
classDiagram
class BaseVoiceAgent {
+start(*args, **kwargs) void
+send(audio, *args, **kwargs) void
+handle_message(message, *args, **kwargs) Any
+interrupt() void
+stop() void
+export_messages(limit, filter) List
+export_events(limit, filter) List
}
class BaseVoiceAgentInterface {
+start(*args, **kwargs) void
+stop() void
+interrupt() void
+output(*args, **kwargs) Any
+receive(data, *args, **kwargs) Any
+_speaker_callback(*args, **kwargs) Any
+_microphone_callback(*args, **kwargs) Any
}
BaseVoiceAgent --> BaseVoiceAgentInterface : "uses"
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L11-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L5-L116)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L11-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L5-L116)

### OpenAI Voice Agent Interface
- Real-time audio capture and playback using PyAudio
- Microphone callback enqueues captured chunks; speaker callback drains buffers for playback
- Configurable sample rate, chunk size, and format
- Thread-safe queue usage and re-engagement delay handling

```mermaid
flowchart TD
Start(["Start Streams"]) --> MicOpen["Open Microphone Stream"]
MicOpen --> SpkOpen["Open Speaker Stream"]
SpkOpen --> Loop{"Main Loop"}
Loop --> |Audio Available| Enqueue["Enqueue Mic Chunk"]
Enqueue --> Callback["Invoke on_audio_callback"]
Callback --> Loop
Loop --> |Playback Needed| Dequeue["Dequeue to Buffer"]
Dequeue --> Play["Play Audio Chunk"]
Play --> Loop
Loop --> |Stop| Close["Stop & Close Streams"]
```

**Diagram sources**
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L74-L121)

**Section sources**
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L21-L126)

### Gemini Live Voice Agent Interface
- Asynchronous integration with Google GenAI live sessions
- Separate queues for outgoing and incoming audio
- Dedicated input/output sample rates and chunk sizes
- Non-blocking reads/writes using asyncio threads

```mermaid
sequenceDiagram
participant GL as "GeminiLiveVoiceAgentInterface"
participant Mic as "Microphone Stream"
participant QOut as "Outgoing Queue"
participant Session as "AsyncSession"
participant QIn as "Incoming Queue"
GL->>Mic : "open(input)"
loop Capture
Mic-->>QOut : "put({data, mime_type})"
end
GL->>Session : "bind session"
loop Playback
QIn-->>GL : "get(bytestream)"
GL-->>Speaker : "write(bytestream)"
end
```

**Diagram sources**
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L41-L149)

**Section sources**
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L19-L150)

### WhisperReader: Speech-to-Text and Transcription
- Supports local file paths, bytes, and file-like objects
- Synchronous and asynchronous transcription via OpenAI Whisper API
- Configurable model, language, prompt, and additional transcribe arguments
- Returns a single Document containing the transcribed text and optional metadata

```mermaid
flowchart TD
A["Input: file path | bytes | file-like"] --> B["Resolve to path or BytesIO"]
B --> C{"Path or BytesIO?"}
C --> |Path| D["Open file and call API"]
C --> |BytesIO| E["Seek to start and call API"]
D --> F["Receive text"]
E --> F
F --> G["Wrap as Document(text, metadata)"]
```

**Diagram sources**
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L59-L174)

**Section sources**
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L15-L175)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/__init__.py#L1-L4)
- [test_readers_whisper.py](file://llama-index-integrations/readers/llama-index-readers-whisper/tests/test_readers_whisper.py)

## Dependency Analysis
- Voice Agent Core depends on:
  - BaseVoiceAgentInterface for audio I/O contracts
  - WebSocket transport abstraction (external to this module)
- Provider Interfaces depend on:
  - PyAudio for real-time capture/playback
  - Google GenAI live session bindings for Gemini integration
- WhisperReader depends on:
  - OpenAI client for audio.transcriptions API
  - fsspec and pathlib for flexible input handling

```mermaid
graph LR
VA["BaseVoiceAgent"] --> IF["BaseVoiceAgentInterface"]
OA["OpenAIVoiceAgentInterface"] --> IF
GL["GeminiLiveVoiceAgentInterface"] --> IF
WR["WhisperReader"] --> |"OpenAI client"| OA
WR --> |"OpenAI client"| GL
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L11-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L5-L116)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L21-L126)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L19-L150)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L48-L51)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L1-L163)
- [interface.py](file://llama-index-core/llama_index/core/voice_agents/interface.py#L1-L116)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L1-L126)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L1-L150)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L1-L53)

## Performance Considerations
- Real-time audio processing
  - Use appropriate chunk sizes and sample rates to balance latency and throughput.
  - Minimize blocking operations; leverage queues and callbacks for non-blocking I/O.
- Latency optimization
  - Reduce queue depths and buffer sizes for lower latency.
  - Use asynchronous patterns (asyncio) for network-bound steps (e.g., Whisper API).
- Multi-language support
  - Configure language and prompt parameters in the transcription reader to improve accuracy for target languages.
- Noise reduction and quality
  - Apply pre-processing filters before sending audio to ASR.
  - Use voice activity detection to gate capture and reduce background noise impact.
- Audio format compatibility
  - Ensure input formats align with provider requirements (e.g., PCM, sample rates).
  - Convert formats as needed before transcription or playback.

[No sources needed since this section provides general guidance]

## Troubleshooting Guide
- Audio device errors
  - Verify device availability and permissions; check PyAudio initialization and stream callbacks.
- Transcription failures
  - Confirm API keys and endpoint reachability; validate input file types and sizes.
- Real-time interruptions
  - Ensure interrupt handlers stop active streams and reset internal states.
- Event/message export
  - Use export utilities to inspect conversation state for debugging.

**Section sources**
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-openai/llama_index/voice_agents/openai/audio_interface.py#L95-L121)
- [audio_interface.py](file://llama-index-integrations/voice_agents/llama-index-voice-agents-gemini-live/llama_index/voice_agents/gemini_live/audio_interface.py#L84-L110)
- [base.py](file://llama-index-integrations/readers/llama-index-readers-whisper/llama_index/readers/whisper/base.py#L82-L113)
- [base.py](file://llama-index-core/llama_index/core/voice_agents/base.py#L114-L162)

## Conclusion
LlamaIndex provides a modular foundation for audio processing in multi-modal systems:
- Voice agent abstractions enable provider-agnostic real-time audio I/O
- Integration-specific interfaces deliver robust capture/playback experiences
- WhisperReader offers reliable speech-to-text via OpenAI’s Whisper API
- These components can be combined to build voice-enabled RAG, audio QA, and speech synthesis workflows with attention to latency, quality, and multi-language support.

[No sources needed since this section summarizes without analyzing specific files]

## Appendices

### Practical Examples and Workflows
- Voice-enabled RAG
  - Capture microphone audio via the OpenAI or Gemini Live interface
  - Transcribe using WhisperReader
  - Feed transcription into a RAG pipeline to generate grounded answers
- Audio question answering
  - Stream audio to a live session interface
  - Transcribe and route queries to a chat engine or query engine
- Speech synthesis integration
  - Synthesize responses and play them back through the speaker callback

[No sources needed since this section provides general guidance]