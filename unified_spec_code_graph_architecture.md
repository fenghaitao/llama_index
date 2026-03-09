# Unified Specification-Code Graph Architecture

**A comprehensive guide to combining specifications and code in a single property graph**

---

## Table of Contents

1. [Overview](#overview)
2. [Why Unified Graph?](#why-unified-graph)
3. [Architecture Design](#architecture-design)
4. [Implementation Guide](#implementation-guide)
5. [Automatic Bridge Detection](#automatic-bridge-detection)
6. [Querying the Unified Graph](#querying-the-unified-graph)
7. [Use Cases](#use-cases)
8. [Best Practices](#best-practices)

---

## Overview

This architecture combines two distinct data types in a single property graph:

- **Specification Layer**: Requirements, features, API specs, design documents
- **Code Layer**: Classes, methods, functions, modules
- **Bridges**: Relationships connecting specs to code (IMPLEMENTED_BY, VERIFIED_BY, etc.)

### Key Benefits

✅ **Traceability** - Bidirectional spec ↔ code navigation  
✅ **Impact Analysis** - See what breaks when specs change  
✅ **Coverage Analysis** - Find unimplemented requirements  
✅ **Powerful Queries** - Multi-hop reasoning across layers  
✅ **Single Source of Truth** - One graph for everything  
✅ **Visualizations** - See the full picture in Neo4j Browser  
✅ **Compliance** - Prove requirements are met  

---

## Why Unified Graph?

### Without Edges Between Spec & Code

```
[Spec Graph]              [Code Graph]
   (isolated)              (isolated)

Requirement: Auth    vs    Class: UserService
     ❌ No connection!
```

**Problems:**
- Manual cross-referencing needed
- No automated traceability
- Can't query relationships
- Hard to find gaps

### With Edges Between Spec & Code

```
[Unified Graph Database]

(Requirement: "Auth must use OAuth") -[IMPLEMENTED_BY]-> (Class: UserService)
                                    -[VERIFIED_BY]-> (Test: test_oauth)
(Feature: "User Login") -[IMPLEMENTED_BY]-> (Method: authenticate)
(API Endpoint: "/login") -[HANDLED_BY]-> (Function: login_handler)
```

**Capabilities:**
- ✅ "Which code implements this requirement?"
- ✅ "What requirements does this class fulfill?"
- ✅ "Is this feature fully implemented?"
- ✅ "What tests verify this spec?"
- ✅ Multi-hop: "Show me the path from requirement to test"

---

## Architecture Design

### Graph Structure

```
┌─────────────────────────────────────────────────────────────┐
│                  UNIFIED PROPERTY GRAPH                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  [Specification Layer]                                       │
│    ┌──────────────┐                                          │
│    │   Feature    │──REQUIRES──┐                             │
│    └──────────────┘            │                             │
│                                v                             │
│                         ┌──────────────┐                     │
│                         │ Requirement  │                     │
│                         └──────────────┘                     │
│                                │                             │
│                                │ IMPLEMENTED_BY              │
│  ════════════════════════════╪═══════════════════════════   │
│                                │ (Bridge)                    │
│                                v                             │
│  [Implementation Layer]  ┌──────────────┐                   │
│                          │    Class     │                    │
│                          └──────────────┘                    │
│                                │                             │
│                                │ HAS_METHOD                  │
│                                v                             │
│                         ┌──────────────┐                     │
│                         │   Method     │                     │
│                         └──────────────┘                     │
│                                │                             │
│                                │ TESTED_BY                   │
│  ════════════════════════════╪═══════════════════════════   │
│                                │ (Bridge)                    │
│                                v                             │
│  [Test Layer]            ┌──────────────┐                   │
│                          │  Test Suite  │                    │
│                          └──────────────┘                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Node Types

#### Specification Nodes
- `REQUIREMENT` - Individual requirements
- `FEATURE` - High-level features
- `API_SPEC` - API endpoint specifications
- `DESIGN_DOC` - Design decisions
- `USER_STORY` - User stories

#### Code Nodes
- `CLASS` - Class definitions
- `METHOD` - Method/function definitions
- `MODULE` - Code modules/files
- `INTERFACE` - Interfaces/protocols
- `COMPONENT` - Architectural components

#### Test Nodes
- `TEST_SUITE` - Test suites
- `TEST_CASE` - Individual test cases
- `TEST_SCENARIO` - Test scenarios

### Relationship Types

#### Specification Relationships
- `REQUIRES` - Feature requires requirement
- `DEPENDS_ON` - Requirement depends on another
- `PART_OF` - Component of larger feature

#### Code Relationships
- `HAS_METHOD` - Class has method
- `INHERITS_FROM` - Class inheritance
- `IMPORTS` - Module imports
- `CALLS` - Function calls another
- `USES` - Uses a component

#### Bridge Relationships (Spec ↔ Code)
- `IMPLEMENTED_BY` - Spec is implemented by code
- `IMPLEMENTS` - Code implements spec
- `VERIFIED_BY` - Spec is verified by test
- `TESTS` - Test tests code
- `RELATES_TO` - General relationship
- `HANDLED_BY` - API handled by code

---

## Implementation Guide

### Part 1: Basic Setup

```python
from llama_index.core import PropertyGraphIndex, Document, SimpleDirectoryReader
from llama_index.core.indices.property_graph import SimpleLLMPathExtractor
from llama_index.core.graph_stores.types import EntityNode, Relation
from llama_index.graph_stores.neo4j import Neo4jPropertyGraphStore
from llama_index.llms.openai import OpenAI

# Initialize single graph database
graph_store = Neo4jPropertyGraphStore(
    username="neo4j",
    password="password",
    url="bolt://localhost:7687",
    database="unified_codebase"
)

llm = OpenAI(model="gpt-4")
```

### Part 2: Index Specifications

```python
# Load specification documents
spec_docs = SimpleDirectoryReader(
    input_dir="./docs",
    required_exts=[".md", ".txt", ".pdf"],
    file_metadata=lambda x: {
        "filepath": x,
        "document_type": "specification"
    }
).load_data()

# Create PropertyGraph for specs
spec_index = PropertyGraphIndex.from_documents(
    spec_docs,
    property_graph_store=graph_store,
    kg_extractors=[
        SimpleLLMPathExtractor(
            llm=llm,
            num_workers=4,
            max_paths_per_chunk=10
        )
    ],
    embed_kg_nodes=True,
    show_progress=True
)

# Graph now contains specification entities:
# - EntityNode(name="OAuth Authentication", label="REQUIREMENT")
# - EntityNode(name="User Login", label="FEATURE")
# - Relation("User Login" -[REQUIRES]-> "OAuth Authentication")
```

### Part 3: Index Code

```python
# Load code files
code_docs = SimpleDirectoryReader(
    input_dir="./src",
    required_exts=[".py", ".js", ".java"],
    file_metadata=lambda x: {
        "filepath": x,
        "document_type": "code"
    }
).load_data()

# Add code to SAME PropertyGraph
code_index = PropertyGraphIndex.from_documents(
    code_docs,
    property_graph_store=graph_store,  # Same graph store!
    kg_extractors=[
        SimpleLLMPathExtractor(
            llm=llm,
            num_workers=4,
            max_paths_per_chunk=10
        )
    ],
    embed_kg_nodes=True,
    show_progress=True
)

# Graph now ALSO contains code entities:
# - EntityNode(name="UserService", label="CLASS")
# - EntityNode(name="authenticate", label="METHOD")
# - Relation("UserService" -[HAS_METHOD]-> "authenticate")
```

### Part 4: Create Bridges Between Spec and Code

```python
from typing import List
from datetime import datetime

def create_spec_code_bridges(graph_store, llm):
    """
    Create relationships between spec entities and code entities.
    This is the KEY part that makes the unified graph powerful!
    """
    
    # Get all spec entities
    spec_entities = graph_store.get(properties={"document_type": "specification"})
    
    # Get all code entities  
    code_entities = graph_store.get(properties={"document_type": "code"})
    
    bridges = []
    
    # For each spec entity, find related code entities
    for spec_entity in spec_entities:
        # Use LLM to find related code
        prompt = f"""
        Given this specification entity:
        Name: {spec_entity.name}
        Type: {spec_entity.label}
        Description: {spec_entity.properties.get('description', 'N/A')}
        
        And these code entities:
        {[f"{e.name} ({e.label})" for e in code_entities[:20]]}
        
        Which code entities implement or relate to this spec entity?
        
        Return as JSON:
        {{
            "implementations": [
                {{"code_entity": "UserService", "relationship": "IMPLEMENTED_BY"}},
                {{"code_entity": "test_oauth", "relationship": "VERIFIED_BY"}}
            ]
        }}
        """
        
        response = llm.complete(prompt)
        implementations = parse_json_response(response)
        
        # Create relations
        for impl in implementations:
            relation = Relation(
                label=impl["relationship"],
                source_id=spec_entity.name,
                target_id=impl["code_entity"],
                properties={
                    "bridge_type": "spec_to_code",
                    "confidence": 0.9,
                    "created_at": datetime.now().isoformat()
                }
            )
            bridges.append(relation)
    
    # Insert all bridges
    graph_store.upsert_relations(bridges)
    print(f"Created {len(bridges)} spec-to-code bridges")
    return bridges

# Execute bridging
bridges = create_spec_code_bridges(graph_store, llm)
```

---

## Automatic Bridge Detection

For production systems, use automated bridge detection with multiple strategies:

### SpecCodeBridgeBuilder Class

```python
import re
from typing import List, Dict

class SpecCodeBridgeBuilder:
    """
    Automatically creates edges between specification and code entities
    using multiple matching strategies.
    """
    
    def __init__(self, graph_store, llm):
        self.graph_store = graph_store
        self.llm = llm
    
    def build_bridges(self) -> List[Relation]:
        """Create all spec-to-code bridges using multiple strategies."""
        bridges = []
        
        # Strategy 1: Filename matching
        bridges.extend(self._match_by_filename())
        
        # Strategy 2: Entity name matching
        bridges.extend(self._match_by_entity_name())
        
        # Strategy 3: Semantic similarity
        bridges.extend(self._match_by_similarity())
        
        # Strategy 4: Docstring references
        bridges.extend(self._match_by_docstring())
        
        # Strategy 5: LLM-based matching
        bridges.extend(self._match_by_llm())
        
        # Insert all bridges
        self.graph_store.upsert_relations(bridges)
        return bridges
    
    def _match_by_filename(self) -> List[Relation]:
        """
        Match based on file naming conventions.
        Example: auth_requirements.md → auth/user_service.py
        """
        bridges = []
        
        spec_entities = self.graph_store.get(
            properties={"document_type": "specification"}
        )
        code_entities = self.graph_store.get(
            properties={"document_type": "code"}
        )
        
        for spec in spec_entities:
            spec_file = spec.properties.get("filepath", "")
            spec_module = self._extract_module_name(spec_file)
            
            for code in code_entities:
                code_file = code.properties.get("filepath", "")
                if spec_module and spec_module in code_file:
                    bridges.append(Relation(
                        label="RELATES_TO",
                        source_id=spec.name,
                        target_id=code.name,
                        properties={
                            "match_type": "filename",
                            "confidence": 0.7
                        }
                    ))
        
        return bridges
    
    def _match_by_entity_name(self) -> List[Relation]:
        """
        Match entities with similar names.
        Example: Requirement "UserService" → Class "UserService"
        """
        bridges = []
        
        spec_entities = self.graph_store.get(
            properties={"document_type": "specification"}
        )
        code_entities = self.graph_store.get(
            properties={"document_type": "code"}
        )
        
        for spec in spec_entities:
            for code in code_entities:
                if self._names_match(spec.name, code.name):
                    bridges.append(Relation(
                        label="IMPLEMENTED_BY",
                        source_id=spec.name,
                        target_id=code.name,
                        properties={
                            "match_type": "entity_name",
                            "confidence": 0.9
                        }
                    ))
        
        return bridges
    
    def _match_by_similarity(self) -> List[Relation]:
        """Match based on embedding similarity."""
        bridges = []
        
        spec_nodes = self.graph_store.get(
            properties={"document_type": "specification"}
        )
        code_nodes = self.graph_store.get(
            properties={"document_type": "code"}
        )
        
        for spec in spec_nodes:
            if not spec.embedding:
                continue
            
            similar_code = self._find_similar_by_embedding(
                spec.embedding,
                code_nodes,
                top_k=3,
                threshold=0.8
            )
            
            for code, similarity in similar_code:
                bridges.append(Relation(
                    label="RELATES_TO",
                    source_id=spec.name,
                    target_id=code.name,
                    properties={
                        "match_type": "semantic_similarity",
                        "confidence": similarity
                    }
                ))
        
        return bridges
    
    def _match_by_docstring(self) -> List[Relation]:
        """
        Match based on docstring references.
        Example: Docstring says "Implements REQ-AUTH-001"
        """
        bridges = []
        
        code_entities = self.graph_store.get(
            properties={"document_type": "code"}
        )
        
        for code in code_entities:
            text = code.properties.get("text", "")
            req_ids = self._extract_requirement_ids(text)
            
            for req_id in req_ids:
                spec = self.graph_store.get(
                    properties={"requirement_id": req_id}
                )
                
                if spec:
                    bridges.append(Relation(
                        label="IMPLEMENTS",
                        source_id=code.name,
                        target_id=spec[0].name,
                        properties={
                            "match_type": "docstring_reference",
                            "requirement_id": req_id,
                            "confidence": 1.0
                        }
                    ))
        
        return bridges
    
    def _match_by_llm(self) -> List[Relation]:
        """Use LLM to determine relationships (most accurate, slowest)."""
        bridges = []
        
        spec_entities = self.graph_store.get(
            properties={"document_type": "specification"}
        )
        code_entities = self.graph_store.get(
            properties={"document_type": "code"}
        )
        
        # Process in batches for efficiency
        for spec_batch in self._batch(spec_entities, 5):
            for code_batch in self._batch(code_entities, 10):
                prompt = f"""
                Match specification entities to code entities:
                
                SPECIFICATIONS:
                {self._format_entities(spec_batch)}
                
                CODE:
                {self._format_entities(code_batch)}
                
                Return matches as JSON:
                {{
                    "matches": [
                        {{
                            "spec": "OAuth Requirement",
                            "code": "UserService",
                            "relationship": "IMPLEMENTED_BY",
                            "confidence": 0.95
                        }}
                    ]
                }}
                """
                
                response = self.llm.complete(prompt)
                matches = self._parse_llm_response(response)
                
                for match in matches:
                    bridges.append(Relation(
                        label=match["relationship"],
                        source_id=match["spec"],
                        target_id=match["code"],
                        properties={
                            "match_type": "llm_analysis",
                            "confidence": match["confidence"]
                        }
                    ))
        
        return bridges
    
    # Helper methods
    def _extract_module_name(self, filepath: str) -> str:
        """Extract module name from filepath."""
        match = re.search(r'(\w+)_requirements', filepath)
        return match.group(1) if match else ""
    
    def _names_match(self, name1: str, name2: str) -> bool:
        """Check if entity names match."""
        n1 = name1.lower().replace("_", "").replace(" ", "")
        n2 = name2.lower().replace("_", "").replace(" ", "")
        return n1 == n2 or n1 in n2 or n2 in n1
    
    def _extract_requirement_ids(self, text: str) -> List[str]:
        """Extract requirement IDs from text (e.g., REQ-AUTH-001)."""
        return re.findall(r'\b(REQ|SPEC)-[A-Z]+-\d+\b', text)
    
    def _batch(self, items: List, batch_size: int):
        """Yield batches of items."""
        for i in range(0, len(items), batch_size):
            yield items[i:i + batch_size]
    
    def _format_entities(self, entities):
        """Format entities for LLM prompt."""
        return "\n".join([
            f"- {e.name} ({e.label}): {e.properties.get('description', 'N/A')}"
            for e in entities
        ])
    
    def _parse_llm_response(self, response):
        """Parse JSON from LLM response."""
        import json
        # Extract JSON from response text
        # Implementation depends on your LLM response format
        return json.loads(response.text)
    
    def _find_similar_by_embedding(self, query_embedding, nodes, top_k=3, threshold=0.8):
        """Find similar nodes by embedding."""
        import numpy as np
        
        similarities = []
        for node in nodes:
            if not node.embedding:
                continue
            
            # Cosine similarity
            similarity = np.dot(query_embedding, node.embedding) / (
                np.linalg.norm(query_embedding) * np.linalg.norm(node.embedding)
            )
            
            if similarity >= threshold:
                similarities.append((node, similarity))
        
        # Sort by similarity and return top k
        similarities.sort(key=lambda x: x[1], reverse=True)
        return similarities[:top_k]


# Usage
bridge_builder = SpecCodeBridgeBuilder(graph_store, llm)
bridges = bridge_builder.build_bridges()

print(f"Created {len(bridges)} spec-to-code bridges:")
for bridge in bridges[:10]:
    print(f"  ({bridge.source_id}) -[{bridge.label}]-> ({bridge.target_id})")
    print(f"    Type: {bridge.properties['match_type']}")
    print(f"    Confidence: {bridge.properties['confidence']}")
```

---

## Querying the Unified Graph

### Query 1: Traceability Matrix

Find all requirements and their implementation status:

```cypher
MATCH (req:REQUIREMENT)
OPTIONAL MATCH (req)-[:IMPLEMENTED_BY]->(impl)
OPTIONAL MATCH (req)-[:VERIFIED_BY]->(test)
RETURN req.name AS requirement,
       req.priority AS priority,
       impl.name AS implementation,
       test.name AS test,
       CASE
         WHEN impl IS NULL THEN 'NOT STARTED'
         WHEN test IS NULL THEN 'IMPLEMENTED, NOT TESTED'
         ELSE 'COMPLETE'
       END AS status
ORDER BY req.priority DESC
```

**Example Output:**

| requirement          | priority | implementation | test       | status                   |
|---------------------|----------|----------------|------------|--------------------------|
| OAuth Authentication | HIGH     | UserService    | test_oauth | COMPLETE                 |
| Two-Factor Auth     | HIGH     | NULL           | NULL       | NOT STARTED              |
| Password Reset      | MEDIUM   | PasswordReset  | NULL       | IMPLEMENTED, NOT TESTED  |

### Query 2: Impact Analysis

If I change this spec, what code is affected?

```cypher
MATCH (spec:REQUIREMENT {name: 'OAuth Authentication'})
      -[:IMPLEMENTED_BY]->(code)
OPTIONAL MATCH (code)-[:CALLS]->(dependency)
RETURN spec.name,
       code.name AS affected_code,
       collect(dependency.name) AS dependencies
```

### Query 3: Coverage Analysis

Which requirements have no implementation?

```cypher
MATCH (req:REQUIREMENT)
WHERE NOT (req)-[:IMPLEMENTED_BY]->()
RETURN req.name, req.priority, req.description
ORDER BY req.priority DESC
```

### Query 4: Reverse Tracing

What specs does this class implement?

```cypher
MATCH (spec)-[:IMPLEMENTED_BY]->(class:CLASS {name: 'UserService'})
RETURN spec.name, spec.description, spec.filepath
```

### Query 5: Full Traceability Path

From feature to test:

```cypher
MATCH path = (feature:FEATURE {name: 'User Management'})
             -[:REQUIRES]->(req:REQUIREMENT)
             -[:IMPLEMENTED_BY]->(code)
             -[:TESTED_BY]->(test)
RETURN path
```

### Query 6: Unverified Implementations

Code that implements specs but has no tests:

```cypher
MATCH (req:REQUIREMENT)-[:IMPLEMENTED_BY]->(code)
WHERE NOT (code)-[:TESTED_BY]->()
RETURN req.name, code.name, code.filepath
```

---

## Use Cases

### Use Case 1: Feature Development

```python
# 1. Developer asks: "What does the spec say about user registration?"
spec_answer = graph_store.structured_query("""
    MATCH (req:REQUIREMENT)
    WHERE req.name CONTAINS 'registration' OR req.description CONTAINS 'registration'
    RETURN req.name, req.description
""")

# 2. Developer implements based on spec

# 3. Code review: "Does this implementation match the spec?"
code_answer = graph_store.structured_query("""
    MATCH (req:REQUIREMENT {name: 'User Registration'})
          -[:IMPLEMENTED_BY]->(impl)
    RETURN impl.name, impl.filepath
""")

# 4. Compare manually or with LLM
```

### Use Case 2: Debugging

```python
# Bug report: "Authentication fails for OAuth users"

# Check spec
spec = graph_store.structured_query("""
    MATCH (req:REQUIREMENT)
    WHERE req.name CONTAINS 'OAuth'
    RETURN req.name, req.description
""")

# Check implementation
code = graph_store.structured_query("""
    MATCH (req:REQUIREMENT)-[:IMPLEMENTED_BY]->(impl)
    WHERE req.name CONTAINS 'OAuth'
    RETURN impl.name, impl.filepath
""")

# Find mismatch
print("Spec expects:", spec)
print("Code does:", code)
```

### Use Case 3: Compliance Checking

```python
def check_compliance():
    """Check if all high-priority requirements are implemented and tested."""
    
    results = graph_store.structured_query("""
        MATCH (req:REQUIREMENT)
        WHERE req.priority = 'HIGH'
        OPTIONAL MATCH (req)-[:IMPLEMENTED_BY]->(impl)
        OPTIONAL MATCH (req)-[:VERIFIED_BY]->(test)
        RETURN req.name,
               impl.name AS implementation,
               test.name AS test,
               CASE
                 WHEN impl IS NULL THEN 'FAIL: Not implemented'
                 WHEN test IS NULL THEN 'FAIL: Not tested'
                 ELSE 'PASS'
               END AS status
    """)
    
    failures = [r for r in results if r['status'].startswith('FAIL')]
    
    if failures:
        print("❌ Compliance check FAILED:")
        for f in failures:
            print(f"  - {f['name']}: {f['status']}")
        return False
    else:
        print("✅ All high-priority requirements are implemented and tested")
        return True

# Run in CI/CD
if not check_compliance():
    sys.exit(1)
```

### Use Case 4: Documentation Generation

```python
def generate_feature_documentation(feature_name):
    """Generate documentation combining spec and code."""
    
    # Query graph
    data = graph_store.structured_query("""
        MATCH (feature:FEATURE {name: $feature_name})
              -[:REQUIRES]->(req:REQUIREMENT)
        OPTIONAL MATCH (req)-[:IMPLEMENTED_BY]->(impl)
        OPTIONAL MATCH (req)-[:VERIFIED_BY]->(test)
        RETURN feature.name,
               feature.description,
               collect({
                   requirement: req.name,
                   req_description: req.description,
                   implementation: impl.name,
                   impl_file: impl.filepath,
                   test: test.name
               }) AS details
    """, params={"feature_name": feature_name})
    
    # Generate markdown
    doc = f"""
# {data['name']}

{data['description']}

## Requirements

"""
    
    for detail in data['details']:
        doc += f"""
### {detail['requirement']}

{detail['req_description']}

**Implementation:** `{detail['implementation']}` in `{detail['impl_file']}`  
**Test Coverage:** `{detail['test']}`

"""
    
    return doc

# Usage
doc = generate_feature_documentation("User Management")
with open("docs/generated/user_management.md", "w") as f:
    f.write(doc)
```

### Use Case 5: Impact Analysis Dashboard

```python
def analyze_spec_change_impact(requirement_name):
    """Analyze the impact of changing a requirement."""
    
    impact = graph_store.structured_query("""
        MATCH (req:REQUIREMENT {name: $req_name})
              -[:IMPLEMENTED_BY]->(impl)
        OPTIONAL MATCH (impl)-[:CALLS]->(dep)
        OPTIONAL MATCH (impl)<-[:TESTS]-(test)
        OPTIONAL MATCH (req)<-[:REQUIRES]-(feature)
        RETURN req.name,
               collect(DISTINCT impl.name) AS affected_code,
               collect(DISTINCT dep.name) AS dependencies,
               collect(DISTINCT test.name) AS affected_tests,
               collect(DISTINCT feature.name) AS affected_features
    """, params={"req_name": requirement_name})
    
    print(f"\n📊 Impact Analysis for: {requirement_name}")
    print(f"Affected Code: {len(impact['affected_code'])} modules")
    for code in impact['affected_code']:
        print(f"  - {code}")
    
    print(f"\nDependent Code: {len(impact['dependencies'])} modules")
    for dep in impact['dependencies']:
        print(f"  - {dep}")
    
    print(f"\nAffected Tests: {len(impact['affected_tests'])} tests")
    for test in impact['affected_tests']:
        print(f"  - {test}")
    
    print(f"\nAffected Features: {len(impact['affected_features'])} features")
    for feature in impact['affected_features']:
        print(f"  - {feature}")
    
    return impact

# Usage
analyze_spec_change_impact("OAuth Authentication")
```

---

## Best Practices

### 1. Naming Conventions

**Use consistent naming between specs and code:**

```python
# In specification
"""
Requirement: USER_AUTH_001
The UserService must authenticate users using OAuth 2.0
"""

# In code
class UserService:
    """
    Implements: USER_AUTH_001
    
    Handles user authentication using OAuth 2.0
    """
    def authenticate(self, user):
        pass
```

### 2. Metadata Enrichment

**Add rich metadata to enable better matching:**

```python
spec_docs = SimpleDirectoryReader(
    input_dir="./docs",
    file_metadata=lambda x: {
        "filepath": x,
        "document_type": "specification",
        "requirement_id": extract_req_id(x),
        "priority": extract_priority(x),
        "category": extract_category(x),
        "version": extract_version(x)
    }
).load_data()
```

### 3. Bridge Quality Scoring

**Track confidence scores for bridges:**

```python
# Low confidence bridges need manual review
manual_review = graph_store.structured_query("""
    MATCH (spec)-[r]->(code)
    WHERE r.bridge_type = 'spec_to_code'
      AND r.confidence < 0.7
    RETURN spec.name, code.name, r.confidence, r.match_type
    ORDER BY r.confidence ASC
""")
```

### 4. Regular Validation

**Run validation checks regularly:**

```python
def validate_graph():
    """Validate graph integrity."""
    
    # Check 1: All high-priority specs have implementations
    missing_impl = graph_store.structured_query("""
        MATCH (req:REQUIREMENT)
        WHERE req.priority = 'HIGH'
          AND NOT (req)-[:IMPLEMENTED_BY]->()
        RETURN count(req) AS count
    """)
    
    # Check 2: All implementations have tests
    missing_tests = graph_store.structured_query("""
        MATCH (code)<-[:IMPLEMENTED_BY]-(req)
        WHERE NOT (code)-[:TESTED_BY]->()
        RETURN count(code) AS count
    """)
    
    # Check 3: Orphaned nodes
    orphans = graph_store.structured_query("""
        MATCH (n)
        WHERE NOT (n)--()
        RETURN count(n) AS count
    """)
    
    print(f"Missing implementations: {missing_impl['count']}")
    print(f"Missing tests: {missing_tests['count']}")
    print(f"Orphaned nodes: {orphans['count']}")
```

### 5. Version Control Integration

**Track spec-code synchronization:**

```python
# Add git commit info to metadata
code_docs = SimpleDirectoryReader(
    input_dir="./src",
    file_metadata=lambda x: {
        "filepath": x,
        "document_type": "code",
        "git_commit": get_last_commit(x),
        "git_author": get_author(x),
        "last_modified": get_modification_time(x)
    }
).load_data()
```

### 6. Incremental Updates

**Update only changed files:**

```python
def incremental_update(changed_files):
    """Update only changed files in the graph."""
    
    for file in changed_files:
        if is_spec_file(file):
            # Delete old spec entities
            graph_store.delete(properties={"filepath": file})
            
            # Re-index
            new_doc = load_document(file)
            spec_index.insert(new_doc)
            
        elif is_code_file(file):
            # Delete old code entities
            graph_store.delete(properties={"filepath": file})
            
            # Re-index
            new_doc = load_document(file)
            code_index.insert(new_doc)
    
    # Rebuild bridges for affected entities
    bridge_builder.build_bridges()
```

### 7. Visualization

**Use Neo4j Browser for visual exploration:**

```cypher
-- Visualize feature → requirement → code → test
MATCH path = (f:FEATURE)-[:REQUIRES]->(r:REQUIREMENT)
             -[:IMPLEMENTED_BY]->(c:CLASS)
             -[:TESTED_BY]->(t:TEST)
WHERE f.name = 'User Management'
RETURN path
LIMIT 50
```

---

## Project Structure

```
project/
├── src/                          # Source code
│   ├── auth/
│   │   ├── user_service.py
│   │   └── oauth_handler.py
│   └── utils/
│       └── error_handler.py
│
├── docs/                         # Specifications
│   ├── requirements/
│   │   ├── auth_requirements.md  (Contains: REQ-AUTH-001, REQ-AUTH-002)
│   │   └── api_spec.md
│   └── design/
│       └── architecture.md
│
├── tests/                        # Test suites
│   ├── test_auth.py
│   └── test_integration.py
│
├── graph_config/
│   ├── build_unified_graph.py   # Main script
│   ├── bridge_builder.py        # SpecCodeBridgeBuilder
│   └── queries.py               # Common queries
│
└── .github/workflows/
    └── graph_validation.yml     # CI check for compliance
```

---

## Summary

### Key Takeaways

✅ **Single Graph Database** - Store specs and code in the same Neo4j database  
✅ **Automatic Bridging** - Use multiple strategies to link specs to code  
✅ **Powerful Queries** - Cypher enables complex traceability queries  
✅ **Compliance Checking** - Verify all specs are implemented and tested  
✅ **Impact Analysis** - See what breaks when specs change  
✅ **Documentation Generation** - Auto-generate docs from the graph  

### When to Use This Architecture

✅ Large codebases with extensive specifications  
✅ Regulated industries (finance, healthcare) requiring traceability  
✅ Teams needing impact analysis before changes  
✅ Projects requiring compliance reporting  
✅ Documentation-heavy systems  
✅ Long-lived projects with evolving requirements  

### When NOT to Use

❌ Small projects (<1000 LOC)  
❌ Prototype/experimental code  
❌ No formal specifications  
❌ Solo developer projects  
❌ Fast-moving startups (overhead too high)  

---

## Additional Resources

- **LlamaIndex PropertyGraph Docs**: [Link]
- **Neo4j Cypher Guide**: https://neo4j.com/docs/cypher-manual/
- **Graph Visualization**: Neo4j Browser, Neo4j Bloom
- **Alternative Graph DBs**: Neptune (AWS), NebulaGraph, Memgraph

---

**Created by:** Your friendly AI assistant  
**Last Updated:** 2026-02-11  
**Version:** 1.0

