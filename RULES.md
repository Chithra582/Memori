# RULES.md - Memori Operational Constraints & Guardrails

> **Specification:** OpenGAP Spec 0.1.0  
> **Agent Name:** Memori Agent (`memori-agent`)  
> **Enforcement Level:** Mandatory & Deterministic  

---

## 1. Tenancy Isolation & Memory Leakage Guardrails

1. **Strict Multi-Tenant Segregation**: Memory retrieval queries must be strictly partitioned by `tenant_id` and `user_id`. Cross-tenant vector similarity searches or relational queries are deterministically prohibited at the datastore bridge layer.
2. **Zero Memory Exfiltration**: Extracted user memories, personal habits, credentials, or proprietary tool outputs must never be exported to external third-party logging sinks or telemetry collectors.
3. **Attribution Binding**: Every memory write batch (`writeBatch`) must contain valid `user_id` and `session_id` attribution headers. Anonymous or un-attributed memory persistence is rejected with code `ERR_UNATTRIBUTED_MEMORY_PERSISTENCE`.

---

## 2. Privacy, PII Redaction & GDPR Standards

1. **Pre-Embedding PII Sanitization**: Candidate conversational snippets containing sensitive authentication secrets, raw credit card numbers, or passwords must have those tokens redacted prior to vector embedding generation.
2. **Deterministic Right to Erasure**: Deletion requests must execute atomic cascading deletions across both vector index namespaces and relational conversation history tables within 500ms.
3. **FERPA & GDPR Compliance**: Educational records, student interaction histories, and corporate confidential data are classified as protected internal assets with mandatory encryption at rest (AES-256) and in transit (TLS 1.3).

---

## 3. Retrieval Scoring & Relevance Constraints

1. **Minimum Similarity Threshold**: Injected episodic facts must satisfy an empirical cosine similarity cutoff ($\text{CosineSim} \ge 0.72$). Low-confidence vector matches must be discarded to prevent context window dilution.
2. **Context Window Token Budget Cap**: Memory injection payloads must not exceed the pre-allocated token ceiling (default: 15% of the active model's context window) to prevent starvation of the LLM's reasoning buffer.
3. **Temporal Supersession**: Conflicting memory updates must overwrite or deprecate obsolete episodic nodes using monotonic timestamp comparisons.

---

## 4. Human-in-the-Loop & Developer Primacy

1. **Configurable Memory Policies**: Developers maintain full administrative control to define custom retention TTLs, embedding models, and local-versus-cloud storage routing.
2. **Audit Trail Immutability**: All memory creation, recall, and purge operations must be logged to structured JSON audit streams with cryptographic operation signatures.
