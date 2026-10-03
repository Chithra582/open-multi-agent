# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Open Multi-Agent (OMA)** (`open-multi-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Open Multi-Agent (OMA) (`open-multi-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Multi-Agent Runtime, Durable Governance & Audit Ledgers  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

Open Multi-Agent (OMA) governs multi-agent workflows through a deterministic, Directed Acyclic Graph (DAG) state machine featuring mandatory cryptographic approval gates and tamper-evident run records. Workflows progress through five discrete operational phases:

### 1. Decision Architecture

The runtime intake, state classification, evaluation, and execution tracking operate across a deterministic, five-stage pipeline:

```
[ Multi-Agent Workflow Definition ]
                 │
                 ▼
[ 1. Topological Sorting & Cycle Check ]
                 │
                 ▼
[ 2. Node Execution & Risk Evaluation ]
                 │
                 ▼
[ 3. Consequential Action Approval Gate ]
                 │
                 ▼
[ 4. Sandboxed Tool Dispatch & Merkle Logging ]
                 │
                 ▼
[ 5. Byte-for-Byte Run Record Sealing ]
```

### 2. Decision Logic & Routing Formulations

Prior to dispatching any tool invocation or external API operation, the runtime calculates an objective operational risk score $R_{\text{op}}$ to determine whether the action is consequential and requires a durable approval hold:

$$R_{\text{op}} = w_m \cdot M_{\text{mutation}} + w_e \cdot E_{\text{exfiltration}} + w_s \cdot S_{\text{scope}} + w_c \cdot C_{\text{cost}}$$

Where:
- $M_{\text{mutation}} \in \{0, 1\}$: Binary flag indicating state modification (file write, database update, process kill).
- $E_{\text{exfiltration}} \in \{0, 1\}$: Network egress flag indicating transmission of data outside the local host perimeter.
- $S_{\text{scope}} \in [0, 1]$: Blast radius measure reflecting affected system dependencies or directory depths.
- $C_{\text{cost}} \in [0, 1]$: Financial cost estimate relative to maximum single-turn token thresholds.
- Parameter weights: $w_m = 0.40$, $w_e = 0.30$, $w_s = 0.20$, $w_c = 0.10$ ($\sum w_i = 1.0$).

If $R_{\text{op}} \ge \tau_{\text{risk}} = 0.50$, the runtime halts execution, creates an immutable pending approval token, and emits a human review request.

### 3. Thresholding & Refusal Decision Criteria

Open Multi-Agent (OMA) enforces strict operational boundaries and deterministic refusal thresholds:
- **Refusal on ERR_APPROVAL_REQUIRED**: Risk score exceeds threshold ($R_{\text{op}} \ge 0.50$) without valid signature halts execution with code `ERR_APPROVAL_REQUIRED`.
- **Refusal on ERR_SIGNATURE_INVALID**: Digital signature on approval token fails public key cryptographic verification halts execution with code `ERR_SIGNATURE_INVALID`.
- **Refusal on ERR_DAG_CYCLE_DETECTED**: Workflow graph contains circular dependency edges halts execution with code `ERR_DAG_CYCLE_DETECTED`.
- **Refusal on ERR_UNAUTHORIZED_NETWORK_ACCESS**: Tool attempts outbound network request when `allow_network: false` halts execution with code `ERR_UNAUTHORIZED_NETWORK_ACCESS`.
- **Refusal on ERR_TOOL_EXECUTION_TIMEOUT**: Sandbox tool execution duration exceeds timeout limit (> 60s) halts execution with code `ERR_TOOL_EXECUTION_TIMEOUT`.

### 4. Fallback Decision Mechanism

Continuous operational stability is maintained through layered fault recovery:
- **Approval Timeout Fallback**: If an approval request remains unaddressed beyond the configured timeout window, the runtime safely cancels the dependent subgraph while preserving completed antecedent state.
- **Local Inference Failover**: If an external foundation model provider is unreachable, OMA can reroute tasks to local fallback instances (e.g., local Ollama or vLLM endpoints).
- **Ledger Integrity Fallback**: If a database crash occurs during a transaction, SQLite writeahead logging (WAL) and Merkle parenthash chaining allow full state reconstruction from the last verified block.
- **Model Fallback Cascade**: High-level reasoning and synthesis default to `gemini-2.0-flash` with automatic failover to `gpt-4o` and `claude-3-5-sonnet`.

### 5. Human-in-the-Loop Governance

Human operators retain sovereign authority over the multi-agent execution lifecycle:
- **Consequential Action Sign-Off**: Irreversible operations (production deployments, database migrations, financial transactions) are structurally blocked without cryptographically signed approvals.
- **Offline Ledger Auditing**: Compliance officers can run `npx oma-audit verify <run-record.jsonl>` offline to inspect every prompt and decision without contacting external servers.
- **Key Pair Ownership**: Approval signers manage their private signing keys locally (via WebCrypto or hardware security modules), ensuring non-repudiation.

---

## The Data It Uses

Open Multi-Agent (OMA) operates under strict principles of data minimization, environment isolation, and privacy protection.

### 1. Ingested Input Data

The framework processes only operational data necessary to perform its functions:
- **Workflow DAG Definitions**: JSON/TypeScript task graphs specifying agent roles, dependencies, tool bindings, and prompt templates.
- **Tool Arguments & Inputs**: Structured JSON payloads validated against typed JSON Schemas before execution.
- **Signed Approval Tokens**: Cryptographically signed JSON Web Tokens (JWT) or Ed25519 signatures issued by human approvers.

### 2. Configuration & Reference Data

- **Offline Run Ledger**: Local SQLite database storing chronological run events, parent hashes, and token expenditures.
- **Permission Policy Manifest**: Configuration files mapping tool names to risk scores and mandatory approval roles.
- **Public Key Keystore**: Public keys of authorized human approvers used to verify digital signatures.

### 3. Base Model & Inference Lineage

- **Engine Core**: Self-hosted TypeScript 5.9+ runtime built on Node.js LTS with pure native WebCrypto APIs.
- **Model Flexibility**: Compatible with local inference engines (Ollama, vLLM, llama-server) and standard cloud APIs (OpenAI, Anthropic, DeepSeek).
- **Zero-Telemetry Protocol**: Completely isolated networking with zero outbound heartbeat or analytics pings.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against indirect prompt injection, credential leakage, and unauthorized external API dispatch.
- **Local Environment Isolation**: Agent execution workspaces, intermediate scratchpads, and vector stores reside strictly within designated local project directories.
- **Automated Secret Scrubbing**: API keys, database credentials, and personal credentials are automatically redacted prior to embedding or logging.
- **Zero Commercial Monetization**: Prompts, intermediate reasoning trajectories, and task deliverables are never commercialized or shared with third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Open Multi-Agent (OMA) is essential for effective deployment.

### 1. Human Approval Latency
- **Limitation**: Requiring human cryptographic signatures on consequential actions introduces asynchronous waiting delays into otherwise rapid agent loops.
- **Mitigation**: Support webhook notifications (Slack, Discord, Email) to alert approvers instantly, and permit pre-approved policy whitelists for safe development environments.

### 2. Complex Merkle Tree Overhead on Long Workflows
- **Limitation**: Continuously hashing large JSON artifacts in multi-thousand-step workflows increases CPU utilization and ledger storage size.
- **Mitigation**: Hash large tool outputs by streaming content digests rather than loading full payloads into memory, and compress historical ledger segments.

### 3. Strict DAG Acyclicity vs. Open-Ended Exploration
- **Limitation**: Enforcing pure acyclic graphs prevents unbounded iterative conversation loops between agents.
- **Mitigation**: Support explicit iterative sub-graphs with deterministic bounded loop counters rather than arbitrary cycles.

### 4. Local Model Compute Requirements
- **Limitation**: Self-hosted execution with local models (Ollama/vLLM) requires substantial GPU/RAM resources compared to cloud APIs.
- **Mitigation**: Allow hybrid routing where low-risk reasoning runs on local hardware while compute-heavy tasks leverage private cloud endpoints.

### 5. Private Key Custody Responsibility
- **Limitation**: The security of the approval mechanism depends entirely on users safeguarding their private signing keys.
- **Mitigation**: Provide integration guides for hardware security tokens (YubiKeys) and standard WebCrypto credential stores.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & routing formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested input data & query streams | Section 1 | Verified |
| - Configuration & reference schemas | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Human Approval Latency | Section 1 | Verified |
| - Complex Merkle Tree Overhead on Long Workflows | Section 2 | Verified |
| - Strict DAG Acyclicity vs. Open-Ended Exploration | Section 3 | Verified |
| - Local Model Compute Requirements | Section 4 | Verified |
| - Private Key Custody Responsibility | Section 5 | Verified |
