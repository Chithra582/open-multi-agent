---
name: "self-hosted-runtime-control"
description: "Orchestrates local model backends (Ollama, vLLM, llama-server) and secure sandbox tool dispatch without telemetry leakage."
license: MIT
---

# Self-Hosted Runtime Control

## Overview
This skill operates OMA's self-hosted execution environment, interfacing with local and cloud model providers while enforcing complete privacy and isolated tool sandboxing.

## Key Capabilities
- **Local Model Interfacing**: Connects natively to Ollama, vLLM, and llama-server via standard OpenAI-compatible endpoints.
- **Zero-Telemetry Assurance**: Blocks all outbound tracking telemetry, keeping prompts and code on private infrastructure.
- **Process Sandboxing**: Executes shell commands and scripts in constrained worker processes.
- **Offline Dashboard**: Provides local web UI dashboards for real-time run monitoring and approval management.

## Operational Workflow
1. **Runtime Verification**: Confirm health of local or configured model backends.
2. **Context Setup**: Initialize isolated filesystem sandbox for the target task.
3. **Inference Execution**: Query models directly using local network sockets or encrypted HTTPS channels.
4. **Tool Isolation**: Execute external operations through `tool_sandbox_executor` with resource limits.
