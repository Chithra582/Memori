---
name: episodic-memory-indexer
description: Parse dialogue events into structured episodic memory items, sanitizing noise and extracting durable assertions.
version: 1.0.0
category: Developer Tools
---

# Episodic Memory Indexer

## Overview
The `episodic-memory-indexer` skill processes raw conversational turns, tool executions, and state transitions emitted by agents. It filters conversational noise, applies regex-based and heuristic PII redaction, extracts factual assertions, and prepares standardized memory records ready for vector representation.

## Workflow

```mermaid
flowchart LR
    A["Raw Message Stream"] --> B["Attribution Validator"]
    B --> C["PII Redaction Engine"]
    C --> D["Fact & Preference Extractor"]
    D --> E["Episodic Memory Object"]
```

1. **Attribution Validation:**
   - Verifies the presence of `entity_id` and `process_id`.
   - Rejects unauthenticated or ambiguous session traces.

2. **Sanitization & Redaction:**
   - Detects and masks credit card numbers, personal identifiers, passwords, and sensitive API keys.
   - Cleans syntactic artifacts, trailing punctuation, and conversational filler phrases.

3. **Assertion Categorization:**
   - Assigns memory categories: `preference`, `fact`, `constraint`, `rule`, `relationship`, `event`.
   - Generates confidence scores and provenance pointers referencing source turn indices.

## Output Specification
```json
{
  "memory_id": "mem_01J8F3K9X2B0",
  "entity_id": "user_49102",
  "process_id": "coding_assistant",
  "category": "preference",
  "fact": "User prefers TypeScript with strict mode enabled and async/await syntax.",
  "confidence": 0.94,
  "created_at": "2026-09-28T14:35:10Z"
}
```
