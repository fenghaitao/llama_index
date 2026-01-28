# Document Formats

<cite>
**Referenced Files in This Document**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [base.py](file://llama-index-core/llama_index/core/readers/base.py)
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-legacy-office/llama_index/readers/legacy_office/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)
- [test_readers_pdf_table.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/tests/test_readers_pdf_table.py)
- [generate_test_ppt.py](file://llama-index-integrations/readers/llama-index-readers-file/tests/generate_test_ppt.py)
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
This document explains how LlamaIndex supports reading various document formats, focusing on PDF, DOC/DOCX, XLS/XLSX, and PPTX. It covers configuration options such as password protection, page ranges, table extraction, metadata handling, and layout preservation. It also describes file-specific parsing techniques, text extraction methods, structured data recovery, performance optimization for large documents, memory management strategies, error handling for corrupted files, and guidance on customizing extraction parameters and integrating with external document processing libraries.

## Project Structure
LlamaIndex organizes readers primarily under the core readers module and extends functionality via integrations. The file-based readers expose a broad set of format-specific readers, while specialized readers (e.g., PDF table extraction, PDF marker, smart PDF loader, legacy office) live in dedicated integration packages.

```mermaid
graph TB
subgraph "Core Readers"
CR["llama_index.core.readers"]
CR --> CRBase["base.py"]
CR --> CRLoad["loading.py"]
end
subgraph "Integrations - File Readers"
FR["llama_index.readers.file.*"]
FR --> FRDocs["docs.py"]
FR --> FRSlds["slides.py"]
FR --> FRTab["tabular.py"]
FR --> FRPDF["pymu_pdf.py"]
end
subgraph "Integrations - Specialized Readers"
PR["PDF Table Reader"]
MR["PDF Marker Reader"]
SR["Smart PDF Loader"]
LOR["Legacy Office Reader"]
end
CR --> FR
CR --> PR
CR --> MR
CR --> SR
CR --> LOR
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/readers/base.py)
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-legacy-office/llama_index/readers/legacy_office/__init__.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [base.py](file://llama-index-core/llama_index/core/readers/base.py)
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)

## Core Components
- File-based readers: Provide format-specific loaders for PDF, DOC/DOCX, XLS/XLSX, PPTX, CSV, HTML, images, notebooks, EPUB, RTF, XML, and more.
- Specialized readers:
  - PDF Table Reader: Extracts tabular data from PDFs.
  - PDF Marker Reader: Preserves layout and marks regions for downstream processing.
  - Smart PDF Loader: Advanced PDF parsing with configurable strategies.
  - Legacy Office Reader: Handles older Microsoft Office formats.

These components are exported via their respective package __init__.py files and are discoverable through the unified readers namespace.

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-legacy-office/llama_index/readers/legacy_office/__init__.py)

## Architecture Overview
The reader ecosystem follows a layered architecture:
- Core readers define the base interface and loading utilities.
- Integrations encapsulate format-specific logic and third-party dependencies.
- Specialized readers augment capabilities for advanced scenarios (tables, layout, OCR).

```mermaid
graph TB
Client["Application"]
Loader["Reader Loader<br/>loading.py"]
Base["Base Reader Interface<br/>base.py"]
FileReaders["File Readers<br/>file/*"]
PDFTable["PDF Table Reader"]
PDFMarker["PDF Marker Reader"]
SmartPDF["Smart PDF Loader"]
LegacyOffice["Legacy Office Reader"]
Client --> Loader
Loader --> Base
Base --> FileReaders
Base --> PDFTable
Base --> PDFMarker
Base --> SmartPDF
Base --> LegacyOffice
```

**Diagram sources**
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)
- [base.py](file://llama-index-core/llama_index/core/readers/base.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-legacy-office/llama_index/readers/legacy_office/__init__.py)

## Detailed Component Analysis

### PDF Reader
- Supported via file-based readers and specialized PDF tools.
- Capabilities:
  - Text extraction with optional page range selection.
  - Password-protected PDF handling through underlying libraries.
  - Layout preservation and region marking via PDF Marker.
  - Structured table extraction via PDF Table Reader.
  - Advanced parsing strategies via Smart PDF Loader.

```mermaid
sequenceDiagram
participant App as "Application"
participant Loader as "Reader Loader"
participant PDFR as "PDF Reader"
participant TableR as "PDF Table Reader"
participant MarkerR as "PDF Marker Reader"
App->>Loader : Request load(document)
Loader->>PDFR : Initialize with config (password, pages)
PDFR-->>Loader : Extracted text/layout
App->>TableR : Extract tables from PDF
TableR-->>App : Tabular data
App->>MarkerR : Mark regions/layout
MarkerR-->>App : Layout-aware segments
```

**Diagram sources**
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)

### DOC/DOCX Reader
- Provided by file-based readers.
- Capabilities:
  - Text extraction from Microsoft Word documents.
  - Metadata handling (author, title, subject, creation date).
  - Embedded object extraction (images, charts) via underlying libraries.
  - Page-range selection for large documents.

```mermaid
flowchart TD
Start(["Open DOC/DOCX"]) --> Detect["Detect format and encryption"]
Detect --> Encrypted{"Encrypted?"}
Encrypted --> |Yes| Prompt["Prompt for password"]
Encrypted --> |No| Parse["Parse document structure"]
Prompt --> Parse
Parse --> ExtractText["Extract text content"]
Parse --> ExtractMeta["Extract metadata"]
Parse --> ExtractEmbeds["Extract embedded objects"]
ExtractText --> End(["Return nodes"])
ExtractMeta --> End
ExtractEmbeds --> End
```

**Diagram sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)

### XLS/XLSX Reader
- Provided by file-based readers with pandas-backed support.
- Capabilities:
  - Cell-level extraction with row/column indexing.
  - Sheet selection and iteration.
  - Table extraction with headers and alignment.
  - Metadata handling (creator, last modified, sheet names).
  - Handling of formulas and rich formatting.

```mermaid
flowchart TD
Open(["Open XLS/XLSX"]) --> Sheets["List sheets"]
Sheets --> Select["Select target sheets"]
Select --> Iterate["Iterate rows/columns"]
Iterate --> Cells["Extract cell values"]
Cells --> Headers{"Has header row?"}
Headers --> |Yes| Table["Build structured table"]
Headers --> |No| Raw["Raw matrix"]
Table --> Meta["Collect metadata"]
Raw --> Meta
Meta --> Output(["Return nodes"])
```

**Diagram sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)

### PPTX Reader
- Provided by file-based readers.
- Capabilities:
  - Slide-level extraction with page-range selection.
  - Text extraction from shapes and text frames.
  - Metadata handling (title, author, slide count).
  - Embedded media extraction (images, videos) via underlying libraries.

```mermaid
sequenceDiagram
participant App as "Application"
participant Loader as "Reader Loader"
participant PPTXR as "PPTX Reader"
App->>Loader : Load presentation with page range
Loader->>PPTXR : Initialize with config
PPTXR->>PPTXR : Iterate slides
PPTXR-->>Loader : Extract text per slide
Loader-->>App : Nodes with metadata and text
```

**Diagram sources**
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [generate_test_ppt.py](file://llama-index-integrations/readers/llama-index-readers-file/tests/generate_test_ppt.py)

### PDF Table Extraction
- Dedicated reader for robust table detection and recovery.
- Capabilities:
  - Table detection and segmentation.
  - Header inference and column alignment.
  - Export to structured formats (e.g., DataFrame-ready).
  - Integration with downstream parsers and node parsers.

```mermaid
flowchart TD
PDFIn(["PDF Input"]) --> Detect["Detect tables"]
Detect --> Segment["Segment table regions"]
Segment --> Normalize["Normalize cells and headers"]
Normalize --> Export["Export structured data"]
Export --> Done(["Structured output"])
```

**Diagram sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [test_readers_pdf_table.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/tests/test_readers_pdf_table.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [test_readers_pdf_table.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/tests/test_readers_pdf_table.py)

### PDF Marker and Smart PDF Loader
- PDF Marker preserves layout and marks regions for downstream processing.
- Smart PDF Loader offers advanced parsing strategies and configuration options.

```mermaid
sequenceDiagram
participant App as "Application"
participant Marker as "PDF Marker Reader"
participant Smart as "Smart PDF Loader"
App->>Marker : Load with layout preservation
Marker-->>App : Region-marked segments
App->>Smart : Configure parsing strategy
Smart-->>App : Optimized text and structure
```

**Diagram sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)

### Legacy Office Reader
- Supports older Microsoft Office formats.
- Capabilities:
  - Text extraction from legacy formats.
  - Metadata handling and basic layout awareness.

**Section sources**
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-legacy-office/llama_index/readers/legacy_office/__init__.py)

## Dependency Analysis
- Readers depend on the core base interface and loading utilities.
- File-based readers aggregate multiple format-specific loaders.
- Specialized readers depend on external libraries for advanced parsing and table extraction.
- There are no circular dependencies among the readers; they are decoupled and composable.

```mermaid
graph LR
Base["Base Reader Interface"] --> FileReaders["File Readers"]
Base --> PDFTable["PDF Table Reader"]
Base --> PDFMarker["PDF Marker Reader"]
Base --> SmartPDF["Smart PDF Loader"]
Base --> LegacyOffice["Legacy Office Reader"]
```

**Diagram sources**
- [base.py](file://llama-index-core/llama_index/core/readers/base.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/llama_index/readers/pdf_table/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-pdf-marker/llama_index/readers/pdf_marker/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-smart-pdf-loader/llama_index/readers/smart_pdf_loader/__init__.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-legacy-office/llama_index/readers/legacy_office/__init__.py)

**Section sources**
- [base.py](file://llama-index-core/llama_index/core/readers/base.py)
- [loading.py](file://llama-index-core/llama_index/core/readers/loading.py)
- [__init__.py](file://llama-index-integrations/readers/llama-index-readers-file/llama_index/readers/file/__init__.py)

## Performance Considerations
- Memory management:
  - Process documents in chunks (page ranges, sheet subsets).
  - Stream large files when possible to reduce peak memory usage.
  - Dispose of intermediate objects promptly after parsing.
- Parallelism:
  - Use concurrent readers for independent documents.
  - Batch process multiple pages/sheets where supported.
- Caching:
  - Cache extracted tables and layout markers for repeated queries.
- I/O optimization:
  - Prefer local copies for frequent access.
  - Use efficient serialization formats for intermediate results.

## Troubleshooting Guide
- Corrupted or encrypted files:
  - Verify password-protection and supply correct credentials.
  - Attempt fallback extraction strategies (e.g., layout-preserving vs. plain text).
- Large documents:
  - Limit page/sheet ranges to reduce memory footprint.
  - Enable incremental processing and streaming.
- Table extraction failures:
  - Adjust detection sensitivity and header inference settings.
  - Validate table boundaries and re-run extraction.
- Metadata loss:
  - Confirm metadata extraction is enabled for the specific reader.
  - Use readers that preserve richer metadata when available.

## Conclusion
LlamaIndex provides a comprehensive, extensible set of document readers covering PDF, DOC/DOCX, XLS/XLSX, and PPTX. With specialized readers for tables, layout preservation, and smart parsing, users can tailor extraction to their needs. By leveraging configuration options, performance strategies, and robust error handling, teams can reliably process diverse document formats at scale.

## Appendices
- Practical examples:
  - PDF table extraction and layout-aware processing are demonstrated in the integration tests and examples.
  - PPTX generation utilities help create test slides for validation.

**Section sources**
- [test_readers_pdf_table.py](file://llama-index-integrations/readers/llama-index-readers-pdf-table/tests/test_readers_pdf_table.py)
- [generate_test_ppt.py](file://llama-index-integrations/readers/llama-index-readers-file/tests/generate_test_ppt.py)