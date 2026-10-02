---
name: "verifiable-run-auditing"
description: "Generates byte-for-byte verifiable run records, merkle ledgers, and deterministic execution logs."
license: MIT
---

# Verifiable Run Auditing

## Overview
This skill constructs immutable, cryptographically verifiable audit trails of multi-agent workflows, allowing internal security teams and external auditors to verify run validity byte for byte.

## Key Capabilities
- **Merkle State Chaining**: Links sequential execution events via parent SHA-256 hashes into a tamper-evident ledger.
- **Byte-for-Byte Reproducibility**: Serializes model prompts, tool arguments, outputs, and timestamps in canonical JSON format.
- **Offline Proof Generation**: Produces standalone cryptographic proof bundles that can be verified without network access.
- **Audit Export**: Exports standardized compliance artifacts (`run-record.jsonl`, `merkle-tree.json`).

## Operational Workflow
1. **Event Capture**: Intercept execution events from the scheduler and tool sandbox.
2. **Canonical Serialization**: Hash normalized JSON payloads via `verifiable_record_generator`.
3. **Ledger Commit**: Append hashed record to the local SQLite state ledger.
4. **Receipt Generation**: Compute final Merkle root upon workflow completion and export signed audit receipt.
