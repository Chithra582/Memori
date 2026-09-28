# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Memori Agent** (`memori-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Memori Agent (`memori-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Long-Term Agent Memory, Vector Search & Retrieval  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), FERPA, GDPR  

---

## How the Agent Decides

Memori Agent is an autonomous long-term agent memory orchestration, action-grounded recall, cognitive episodic indexing, and multi-datastore context synthesis agent designed for **Memori Labs**. It bridges LLM frameworks, local storage engines, and cloud persistence with sub-millisecond semantic retrieval, zero-data-leakage tenancy boundaries, and cross-framework memory attribution.

### 1. Decision Architecture

The episodic memory ingestion, semantic retrieval, and prompt context synthesis pipeline operates across a deterministic, five-stage architecture:

```
Conversational Dialogue / Tool Execution Stream (User Query + Assistant Output + Tool Result)
    │
    ▼
[Stage 1: Multi-Format Ingestion & Attribution Validation]
    │  - Intercepts raw interaction turns via SDK middleware and gateway hooks
    │  - Enforces mandatory attribution: validates entity_id and process_id
    │  - Normalizes timestamps, session identifiers, and conversational metadata
    │  - Halts unauthenticated or unattributed payloads to prevent cross-tenant leakage
    ▼
[Stage 2: PII Redaction & Episodic Fact Extraction]
    │  - Applies automated regex and semantic filters to redact PII (SSNs, tokens, credentials)
    │  - Strips conversational pleasantries, stop words, and filler syntax
    │  - Extracts durable assertions, user preferences, system constraints, and entity relationships
    │  - Classifies memory chunks into canonical categories (preference, fact, constraint, rule)
    ▼
[Stage 3: Dual-Stage Semantic Recall & Decay Scoring]
    │  - Generates dense vector representations (e_q) of the active prompt or user query
    │  - Executes sub-millisecond k-NN vector search across tenant-isolated index namespaces
    │  - Combines dense cosine similarity with exact BM25 keyword matching
    │  - Evaluates temporal decay scoring S(q, m) to balance semantic relevance with recency
    ▼
[Stage 4: Admission Thresholding & Context Budgeting]
    │  - Enforces strict admission threshold: filters candidates where S(q, m) < 0.78
    │  - Deduplicates overlapping facts and reconciles temporal contradictions
    │  - Dynamic token budgeting: caps injected context to <= 1,294 tokens (< 5% of prompt window)
    │  - Encloses verified memories within secure XML/Markdown containment tags (<memori_context>)
    ▼
[Stage 5: Asynchronous Datastore Persistence & Audit Logging]
    │  - Queues non-blocking batch writes to cloud storage and BYODB engines (pgvector, Redis, Qdrant)
    │  - Commits atomic transactions using unique idempotency keys to eliminate duplicate writes
    │  - Generates tamper-evident structured JSON audit logs for every recall, write, and purge event
    ▼
Augmented Prompt Dispatched to Downstream Foundation Model with Zero Latency Penalty
```

### 2. Scoring Methodology & Rubric Formulations

Memori Agent computes candidate relevance and admission metrics through two deterministic, mathematically rigorous scoring models:

1. **Composite Semantic & Temporal Decay Score ($S(q, m)$)**:
   $$S(q, m) = \alpha \cdot \cos(\mathbf{e}_q, \mathbf{e}_m) + (1 - \alpha) \cdot \exp(-\lambda \cdot \Delta t)$$
   where:
   - $\cos(\mathbf{e}_q, \mathbf{e}_m) = \frac{\mathbf{e}_q \cdot \mathbf{e}_m}{\|\mathbf{e}_q\| \|\mathbf{e}_m\|}$: Cosine similarity between query vector $\mathbf{e}_q$ and memory vector $\mathbf{e}_m$.
   - $\alpha \in [0, 1]$: Relative semantic weighting coefficient ($\alpha = 0.85$).
   - $\lambda \ge 0$: Exponential temporal decay rate factor ($\lambda = 0.005 \, \text{day}^{-1}$).
   - $\Delta t = t_{\text{current}} - t_{\text{memory}}$: Elapsed time interval in days since memory assertion creation.
   - Admission rule: $S(q, m) \ge \tau$ where baseline threshold $\tau = 0.78$.

2. **Dynamic Context Token Allocation Budget ($B_{\text{inject}}$)**:
   $$B_{\text{inject}} = \min\left(B_{\text{max}}, \; \mu \cdot W_{\text{model}}, \; \sum_{i=1}^{k} \text{Tokens}(m_i) \cdot \mathbb{I}(S(q, m_i) \ge \tau)\right)$$
   where:
   - $B_{\text{max}} = 1,294$: Benchmark token allocation ceiling.
   - $\mu = 0.05$: Maximum proportion of active LLM context window $W_{\text{model}}$.
   - $\mathbb{I}(\cdot)$: Indicator function admitting only memories satisfying confidence thresholding.

### 3. Thresholding & Refusal Decision Criteria

Memori Agent enforces strict deterministic refusal and safety boundaries:
- **Refusal on Missing Attribution**: Memory events lacking explicit `entity_id` and `process_id` are deterministically rejected with code `ERR_UNATTRIBUTED_MEMORY_REFUSED` to prevent multi-tenant data contamination.
- **Refusal of Sensitive PII Ingestion**: Detection of unmasked financial credit card numbers, national identification numbers, or raw API authentication secrets halts storage with code `ERR_SENSITIVE_PII_DETECTED`.
- **Refusal on Sub-Threshold Similarity**: Candidate memories with composite relevance scores $S(q, m) < 0.78$ are excluded from prompt injection under code `ERR_SUBTHRESHOLD_SIMILARITY_EXCLUDED`.
- **Refusal to Execute Injected Instructions**: Ingested dialogue containing embedded prompt injection vectors (e.g., *"Ignore all previous instructions"*) is classified as inert data and stripped of directive status (`ERR_PROMPT_INJECTION_DEFLECTED`).
- **Refusal of Autonomous Self-Modification**: The agent deterministically rejects attempts to alter core configuration in `agent.yaml` or security directives in `RULES.md` (`ERR_SELF_MODIFICATION_PROHIBITED`).

### 4. Fallback Decision Mechanism

Memori Agent guarantees continuous, resilient agent operations through a multi-tier fallback architecture:
- **Stateless Prompt Passthrough Fallback**: If the vector datastore or remote retrieval API experiences network timeouts ($> 120\text{ ms}$), the agent instantly degrades to passing unaugmented system prompts with zero conversational blocking.
- **Local In-Memory Cache Fallback**: When cloud datastores are temporarily unreachable, recently indexed memories are served directly from local LRU in-memory session stores.
- **Lexical Keyword Fallback**: If dense embedding generation fails or is rate-limited, the system falls back to BM25 lexical keyword matching across cached conversational transcripts.
- **Model Fallback Cascade**: High-level memory synthesis, fact consolidation, and query interpretation default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Memori Agent maintains developer and administrative oversight at all operational layers:
- **Developer Memory Inspection**: Developers can query, review, edit, or invalidate any stored episodic assertion through the Memori CLI (`python -m memori`) or Web Console.
- **Selective Memory Purging**: End users and tenant administrators can issue granular deletion requests for specific entities, sessions, or time ranges with immediate verifiable purging.
- **Emergency Kill-Switch**: System administrators can trigger an instant kill-switch via environment variable or API flag, halting all background indexing and dynamic context injection immediately.
- **Audit Logging**: Every memory creation, similarity query, context injection, and manual deletion is recorded in tamper-evident structured JSON logs for audit review.

---

## The Data It Uses

Memori Agent operates under enterprise-grade data governance, strict multi-tenant isolation, and privacy standards.

### 1. Ingested Input Data

The agent processes only authorized conversational interactions and metadata:
- **Dialogue Turns**: User prompts, assistant completions, and tool execution outputs intercepted via SDK client middleware.
- **Attribution Metadata**: Tenant identifier (`entity_id`), calling routine/bot identifier (`process_id`), session key (`session_id`), and UTC timestamps.
- **Extracted Assertions**: Categorized factual statements, stated user preferences, coding standards, and project constraints.

### 2. Configuration & Reference Data

- **Vector Indices & Namespaces**: Tenant-partitioned dense vector indices storing normalized embedding vectors with associated cosine distances.
- **Semantic Taxonomy Rules**: Standardized category schemas distinguishing `preference`, `fact`, `constraint`, `rule`, `relationship`, and `event`.
- **Temporal Decay Configurations**: Configurable decay half-lives ($\lambda$) tailored to specific domain lifecycles (ephemeral sessions vs. durable project guidelines).

### 3. Base Model & Inference Lineage

- **Deterministic Embedding Algorithms**: Canonical dense mathematical vector models yielding standard dimensions (768 to 1536) executed in deterministic mode.
- **Foundation Reasoning Models**: High-capability foundation models (`gemini-2.0-flash`, `gpt-4o`, `claude-3-5-sonnet`) utilized strictly for semantic fact extraction and summarization at low temperature ($0.2$).
- **Zero Training on User Data**: User conversational sessions, private source code, and extracted memory facts are never retained or utilized for training public foundation models.

### 4. Data Privacy, Storage, and Retention

- **FERPA & GDPR Compliance**: Full compliance with FERPA and GDPR (Articles 5, 17, and 28). All episodic memory stores are classified as `internal` with AES-256 encryption at rest and TLS 1.3 in transit.
- **Right to Be Forgotten & 0-Byte Purging**: Memory records can be permanently deleted upon user or administrator request with verified 0-byte database purging.
- **Strict Multi-Tenant Isolation**: Physical and logical namespace partitioning guarantees that no memory data is ever exposed or recalled across disparate `entity_id` boundaries.

---

## Limitations

Understanding the operational boundaries and technical constraints of Memori Agent is essential for optimal integration.

### 1. Context Allocation Caps & Token Budget Saturation
- **Limitation**: In conversational sessions spanning hundreds of turns, accumulating memory facts could saturate the host model's context window.
- **Mitigation**: The agent enforces a strict budget ceiling ($1,294$ tokens, representing $< 5\%$ of standard context) using dynamic top-$k$ relevance pruning.

### 2. Semantic Embedding Drift & Domain Specificity
- **Limitation**: Proprietary code symbols, custom variable names, or emerging acronyms may initially yield lower cosine similarity scores.
- **Mitigation**: Dual-stage hybrid retrieval combines dense vector embeddings with exact BM25 keyword matching to preserve specialized identifiers.

### 3. Asynchronous Write-Behind Synchronization Lag
- **Limitation**: To prevent blocking real-time user dialogue, memory persistence is performed asynchronously, introducing a $50\text{ ms} - 200\text{ ms}$ replication window.
- **Mitigation**: An active in-memory session buffer enables immediate same-session recall prior to persistence layer flushing.

### 4. Indirect Adversarial Prompt Injection via Memories
- **Limitation**: Untrusted user inputs could attempt to plant adversarial directives into memory to manipulate future interactions.
- **Mitigation**: Recalled context is wrapped in strict structural tags (`<memori_context>`) with meta-instructions commanding the LLM to treat memories strictly as factual data.

### 5. Multi-Tenant Namespace Boundary Isolation
- **Limitation**: Misconfigured upstream client SDKs omitting `entity_id` could theoretically risk cross-user context contamination.
- **Mitigation**: The ingestion pipeline enforces hard attribution gates, deterministically rejecting any interaction trace missing verified partition keys.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Scoring methodology & retrieval formulas | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested conversational turns & attribution | Section 1 | Verified |
| - Configuration, vector index & schemas | Section 2 | Verified |
| - Base model lineage & deterministic embeddings | Section 3 | Verified |
| - Data privacy, zero-retention & FERPA/GDPR | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Context allocation caps & token budget | Section 1 | Verified |
| - Semantic embedding drift & domain vocabulary | Section 2 | Verified |
| - Asynchronous write-behind latency | Section 3 | Verified |
| - Indirect adversarial prompt injection | Section 4 | Verified |
| - Multi-tenant namespace boundary isolation | Section 5 | Verified |
