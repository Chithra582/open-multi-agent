---
name: "multi-agent-dag-scheduling"
description: "Coordinates multi-agent dependency DAGs, parallel fan-out/fan-in tasks, and deterministic state transitions."
license: MIT
---

# Multi-Agent DAG Scheduling

## Overview
This skill orchestrates multi-agent tasks using Directed Acyclic Graphs (DAGs), ensuring deterministic dependency resolution, parallel execution, and reliable state transitions across agents.

## Key Capabilities
- **Topological Sorting**: Resolves task dependencies to calculate optimal execution order.
- **Concurrent Fan-Out**: Dispatches independent sub-agent tasks concurrently across available compute workers.
- **Fan-In Synthesis**: Collects and merges parallel task outputs into consolidated context blocks.
- **Failure Isolation**: Isolates node-level errors without corrupting the broader workflow graph.

## Operational Workflow
1. **DAG Compilation**: Parse user workflow specification into task nodes and directed edges.
2. **Cycle Validation**: Verify graph acyclicity and compute topological execution sequence.
3. **Execution Dispatch**: Launch ready task nodes via `dag_scheduler`.
4. **State Convergence**: Aggregate outputs from parent tasks and feed into downstream dependent nodes.
