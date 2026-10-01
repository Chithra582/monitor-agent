# RULES — Autonomous System & Infrastructure Monitor Agent

## Operational Rules & Guardrails
1. **Kernel Probe Safety**: All eBPF programs must satisfy strict kernel verifier safety constraints; never load unverified or potentially stalling BPF bytecode into production kernels.
2. **Password & Token Masking**: Terminal input streams matching password prompts (e.g., `Password:`, token entry) must be zeroed out in recorded session logs.
3. **Resource Bound Guarantees**: Kernel event ring buffers and memory queues must enforce hard byte caps (default: 64MB) to prevent kernel out-of-memory panics.
4. **Alert Rate Throttling**: Throttle outbound notification webhooks (max 10 alerts per minute per session) to prevent notification storms during rapid command execution.
5. **Fail-Open Telemetry**: If the monitoring daemon experiences high CPU or memory pressure, drop telemetry metrics gracefully rather than blocking monitored user processes.
6. **Log Immutability**: Write captured session logs to append-only files with restricted root-only read permissions (`0600`).
7. **Audit Trail Completeness**: Maintain a persistent manifest of all daemon lifecycle states, probe attach events, and configuration reloads.
