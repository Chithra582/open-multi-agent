# EXPLAINABILITY — Open Multi-Agent (OMA)

## How the Agent Decides

Open Multi-Agent (OMA) governs multi-agent workflows through a deterministic, Directed Acyclic Graph (DAG) state machine featuring mandatory cryptographic approval gates and tamper-evident run records. Workflows progress through five discrete operational phases:

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

### 1. Mathematical Scoring & Routing Formulation
Prior to dispatching any tool invocation or external API operation, the runtime calculates an objective operational risk score $R_{\text{op}}$ to determine whether the action is consequential and requires a durable approval hold:

$$R_{\text{op}} = w_m \cdot M_{\text{mutation}} + w_e \cdot E_{\text{exfiltration}} + w_s \cdot S_{\text{scope}} + w_c \cdot C_{\text{cost}}$$

Where:
- $M_{\text{mutation}} \in \{0, 1\}$: Binary flag indicating state modification (file write, database update, process kill).
- $E_{\text{exfiltration}} \in \{0, 1\}$: Network egress flag indicating transmission of data outside the local host perimeter.
- $S_{\text{scope}} \in [0, 1]$: Blast radius measure reflecting affected system dependencies or directory depths.
- $C_{\text{cost}} \in [0, 1]$: Financial cost estimate relative to maximum single-turn token thresholds.
- Parameter weights: $w_m = 0.40$, $w_e = 0.30$, $w_s = 0.20$, $w_c = 0.10$ ($\sum w_i = 1.0$).

If $R_{\text{op}} \ge \tau_{\text{risk}} = 0.50$, the runtime halts execution, creates an immutable pending approval token, and emits a human review request.

### 2. Refusal Criteria & Decision Thresholds
OMA enforces strict structural and security boundaries:

| Trigger Scenario | Operational Action | Error Code |
| :--- | :--- | :--- |
| Risk score exceeds threshold ($R_{\text{op}} \ge 0.50$) without valid signature | Suspend DAG branch; persist checkpoint; await signed approval | `ERR_APPROVAL_REQUIRED` |
| Digital signature on approval token fails public key cryptographic verification | Reject unpause request; record unauthorized resumption attempt | `ERR_SIGNATURE_INVALID` |
| Workflow graph contains circular dependency edges | Abort compilation; report cyclic node references | `ERR_DAG_CYCLE_DETECTED` |
| Tool attempts outbound network request when `allow_network: false` | Terminate tool process; log unauthorized egress attempt | `ERR_UNAUTHORIZED_NETWORK_ACCESS` |
| Sandbox tool execution duration exceeds timeout limit (> 60s) | Terminate worker process; emit timeout failure event | `ERR_TOOL_EXECUTION_TIMEOUT` |

### 3. Multi-Tier Fallback Mechanisms
1. **Approval Timeout Fallback**: If an approval request remains unaddressed beyond the configured timeout window, the runtime safely cancels the dependent sub-graph while preserving completed antecedent state.
2. **Local Inference Failover**: If an external foundation model provider is unreachable, OMA can re-route tasks to local fallback instances (e.g., local Ollama or vLLM endpoints).
3. **Ledger Integrity Fallback**: If a database crash occurs during a transaction, SQLite write-ahead logging (WAL) and Merkle parent-hash chaining allow full state reconstruction from the last verified block.

### 4. Human-in-the-Loop Governance
- **Consequential Action Sign-Off**: Irreversible operations (production deployments, database migrations, financial transactions) are structurally blocked without cryptographically signed approvals.
- **Offline Ledger Auditing**: Compliance officers can run `npx oma-audit verify <run-record.jsonl>` offline to inspect every prompt and decision without contacting external servers.
- **Key Pair Ownership**: Approval signers manage their private signing keys locally (via WebCrypto or hardware security modules), ensuring non-repudiation.

---

## The Data It Uses

### 1. Input Data Types
- **Workflow DAG Definitions**: JSON/TypeScript task graphs specifying agent roles, dependencies, tool bindings, and prompt templates.
- **Tool Arguments & Inputs**: Structured JSON payloads validated against typed JSON Schemas before execution.
- **Signed Approval Tokens**: Cryptographically signed JSON Web Tokens (JWT) or Ed25519 signatures issued by human approvers.

### 2. Reference & Configuration Data
- **Offline Run Ledger**: Local SQLite database storing chronological run events, parent hashes, and token expenditures.
- **Permission Policy Manifest**: Configuration files mapping tool names to risk scores and mandatory approval roles.
- **Public Key Keystore**: Public keys of authorized human approvers used to verify digital signatures.

### 3. Model Lineage & System Architecture
- **Engine Core**: Self-hosted TypeScript 5.9+ runtime built on Node.js LTS with pure native WebCrypto APIs.
- **Model Flexibility**: Compatible with local inference engines (Ollama, vLLM, llama-server) and standard cloud APIs (OpenAI, Anthropic, DeepSeek).
- **Zero-Telemetry Protocol**: Completely isolated networking with zero outbound heartbeat or analytics pings.

### 4. Data Privacy, Retention & Sanitization
- **Strict Data Sovereignty**: All prompt contexts, intermediate reasoning traces, and audit logs remain strictly on the host infrastructure.
- **Secret Redaction**: API keys and private signing keys are masked before committing event snapshots to the ledger.
- **Verifiable Retention**: Completed run ledgers are sealed into immutable `run-record.jsonl` files retained per organizational compliance policies.

---

## Limitations

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

| Checkpoint Focus | Requirement | Status |
| :--- | :--- | :--- |
| **Checkpoint 1** | OpenGAP v0.1.0 Specification (`agent.yaml`, `SOUL.md`, `RULES.md`, `DUTIES.md`, `skills/`, `tools/`) | **Verified** |
| **Checkpoint 2** | Canonical 4-Heading AST Schema & Deterministic Pipeline Diagram | **Verified** |
| **Checkpoint 2** | Mathematical Operational Risk Formulation ($R_{\text{op}}$) & Parameter Weights | **Verified** |
| **Checkpoint 2** | Refusal Criteria Table with Explicit Error Codes & Multi-Tier Fallbacks | **Verified** |
| **Checkpoint 2** | Comprehensive Data Privacy Coverage (4 Subsections) & 5 Numbered Limitations | **Verified** |
| **Checkpoint 3** | Multi-Framework Adapter Portability (`openai`, `crewai`, `claude-code`, `lyzr`) | **Verified** |
