# RULES — Open Multi-Agent (OMA)

## Operational Rules & Guardrails
1. **Mandatory Consequential Action Pauses**: Any tool or DAG node classified with `consequential: true` must freeze execution and await a cryptographically signed approval token before proceeding; automated bypass is strictly forbidden.
2. **Deterministic Merkle Run Ledger**: All execution events (prompts, tool calls, return values, approvals) must be recorded chronologically into the offline state ledger with SHA-256 parent-hash chaining to guarantee tamper evidence.
3. **Strict Zero-Telemetry Mandate**: OMA must never transmit runtime telemetry, agent performance metrics, or prompt payloads to external hosted telemetry servers; all telemetry stays on the user's host infrastructure.
4. **DAG Cycle Prohibition**: Task graphs must be strictly acyclic; dependency resolution engines must detect and reject circular references prior to workflow instantiation.
5. **Tool Sandbox Isolation**: Script execution and shell commands must run within sandboxed child processes with isolated working directories and configurable execution timeouts (default: 60 seconds).
6. **Local Key Sovereignty**: Model vendor API keys and cryptographic signing keys must be read strictly from environment variables or local secret vaults; keys must never appear in plaintext run ledgers.
7. **Offline Verification Completeness**: The runtime must provide self-contained offline verification tooling ensuring that any exported run record can be audited without network connectivity.
