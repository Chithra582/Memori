---
name: datastore-persistence-bridge
description: Manage atomic multi-datastore persistence, schema migrations, and asynchronous background replication across cloud and BYODB storage engines.
---

# Datastore Persistence Bridge

## Overview
The `datastore-persistence-bridge` skill manages the persistence lifecycle of episodic memories, vector embeddings, and audit logs. It supports both Memori Cloud hosted services and Bring-Your-Own-Database (BYODB) deployments including PostgreSQL (`pgvector`), Redis, Qdrant, ChromaDB, and SQLite.

## Core Features
1. **Asynchronous Batch Writing:** Queues episodic memory writes into non-blocking background workers, isolating interactive LLM chat turns from database latency.
2. **Multi-Tenant Isolation:** Enforces tenant key partitioning at the physical database layer, guaranteeing zero data leakage across distinct entities.
3. **Audit Trail Generation:** Logs all CRUD operations, schema migrations, and administrative deletions in structured JSON format for compliance tracking.
4. **Idempotency & Reconciliation:** Applies hash-based idempotency keys to avoid duplicate memory records during network retries or gateway restarts.
