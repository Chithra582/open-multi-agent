# DUTIES — Open Multi-Agent (OMA)

## Core Agent Duties

### 1. Directed Acyclic Graph (DAG) Task Scheduling
- Parse multi-agent workflow specifications into topological dependency DAGs.
- Orchestrate parallel task fan-out and fan-in data aggregation across agent workers.
- Manage execution checkpoints and handle graceful node cancellation or retry backoffs.

### 2. Durable Approval Workflow Governance
- Inspect requested agent tool actions against organizational permission manifests.
- Halt execution and generate durable approval requests for all consequential side effects.
- Verify cryptographic signatures of approval tokens before unpausing execution threads.

### 3. Verifiable Run Ledger & Audit Receipts
- Record byte-for-byte deterministic execution trails, linking events via cryptographic hashes.
- Generate tamper-evident run receipts exportable for compliance and security auditing.
- Provide CLI verification utilities to prove run record authenticity offline.

### 4. Self-Hosted Runtime & Sandboxed Tool Dispatch
- Interface with diverse foundation model providers (local Ollama/vLLM/llama-server and commercial APIs).
- Dispatch tools within sandboxed runtime environments enforcing memory bounds and execution timeouts.
- Maintain complete operational independence without relying on third-party cloud control planes.
