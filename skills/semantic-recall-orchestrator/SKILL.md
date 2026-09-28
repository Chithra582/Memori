---
name: semantic-recall-orchestrator
description: Execute dual-stage hybrid retrieval across dense vector indices and lexical keyword stores with recency decay scoring.
version: 1.0.0
category: Developer Tools
---

# Semantic Recall Orchestrator

## Overview
The `semantic-recall-orchestrator` skill executes sub-millisecond semantic retrieval across multi-tenant vector namespaces. It combines dense mathematical cosine similarity with exact BM25 keyword matching and applies an exponential temporal decay function to prioritize fresh, relevant memories.

## Mathematical Formulation
The composite retrieval score $S(q, m)$ is evaluated for each candidate memory item $m$:

$$S(q, m) = \alpha \cdot \cos(\mathbf{e}_q, \mathbf{e}_m) + (1 - \alpha) \cdot \exp(-\lambda \cdot \Delta t)$$

- $\alpha$: Semantic weight (default: $0.85$)
- $\lambda$: Temporal decay factor (default: $0.005$)
- $\Delta t$: Age of memory in elapsed days
- Cutoff threshold: $\tau \ge 0.78$

## Operational Procedure
1. Receive incoming user prompt query vector $\mathbf{e}_q$ and entity partition filter `entity_id`.
2. Perform $k$-NN search over the vector index (default $k=20$).
3. Filter candidate records using the temporal decay threshold $\tau \ge 0.78$.
4. Sort survivors by composite score and deduplicate semantic overlaps.
5. Return ranked array of memory facts accompanied by source attribution metadata.
