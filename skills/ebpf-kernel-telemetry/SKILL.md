---
name: "ebpf-kernel-telemetry"
description: "Monitors Linux kernel syscalls and process execution passively via extended Berkeley Packet Filters."
---

# eBPF Kernel Telemetry

## Overview
This skill provides passive kernel-level observability for Linux hosts using extended Berkeley Packet Filters (eBPF). By hooking directly into tracepoints and kprobes, the agent monitors all process executions and terminal I/O with minimal CPU overhead.

## Key Capabilities
- **Non-Intrusive Monitoring**: Inspects kernel events without modifying running processes or binaries.
- **Tracepoint Attachment**: Hooks into `sys_enter_execve`, `tty_write`, and socket connect syscalls.
- **Ring Buffer Streaming**: Transfers high-throughput kernel events directly to user space without dropping packets.

## Operational Workflow
1. **Probe Loading**: Verify BPF verifier constraints and attach bytecode to kernel tracepoints.
2. **Event Consumption**: Read structured event structs from the eBPF ring buffer.
3. **Filter & Decode**: Decode command-line arguments and process parentage trees.
4. **State Forwarding**: Forward processed event streams to downstream session correlation layers.
