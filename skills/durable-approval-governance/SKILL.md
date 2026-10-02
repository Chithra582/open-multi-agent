---
name: "durable-approval-governance"
description: "Manages tamper-evident human-in-the-loop approval workflows, cryptographic signatures, and execution holds."
license: MIT
---

# Durable Approval Governance

## Overview
This skill implements durable human-in-the-loop governance for consequential agent actions, pausing workflows until authorized operators provide cryptographically verified approval tokens.

## Key Capabilities
- **Consequential Action Classification**: Evaluates tool calls against risk policies to identify side effects requiring authorization.
- **Durable Execution Suspension**: Freezes workflow state to persistent disk, surviving process restarts and server reboots.
- **Cryptographic Approval Verification**: Validates digital signatures on incoming approval payloads using public key cryptography.
- **Audit Token Generation**: Binds approval records immutably into the permanent execution graph.

## Operational Workflow
1. **Risk Inspection**: Intercept agent tool call and evaluate `consequential` risk tag.
2. **Workflow Pause**: If risk criteria are met, trigger `approval_gate_controller` to halt DAG execution.
3. **Approval Request Dispatch**: Emit notification token with action diff and requested parameters.
4. **Signature Verification & Resume**: Upon receiving approval payload, verify cryptographic signature and resume DAG node execution.
