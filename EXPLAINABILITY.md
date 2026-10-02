# EXPLAINABILITY.md

This document explains the internal mechanisms, data lineage, operational boundaries, and governance framework of **Autonomous System & Infrastructure Monitor Agent** (`monitor-agent`) in accordance with the **OpenGAP v0.1.0** specification for the **HiDevs GitAgent Passport** clearance pipeline.

> **Agent Name:** Autonomous System & Infrastructure Monitor Agent (`monitor-agent`)  
> **Specification:** OpenGAP v0.1.0  
> **Category / Domain:** Developer Tools / Infrastructure Monitoring & eBPF Telemetry  
> **Compliance Standard:** OpenGAP Checkpoint 2 (Explainability & Decision Governance), OWASP LLM Top 10, MITRE ATLAS  

---

## How the Agent Decides

The Autonomous System & Infrastructure Monitor Agent is an open-source Linux observability and security agent utilizing extended Berkeley Packet Filters (eBPF) to passively inspect OpenSSH sessions and host process executions. It provides security engineers and system administrators with granular visibility into active terminal sessions without requiring custom SSH daemons or wrapper shells. Its operational purpose is to detect security incidents, enforce administrative compliance, record immutable session transcripts, and stream system health metrics to enterprise SIEM and observability stacks.

### 1. Decision Architecture

The kernel probe interception, user session auditing, anomaly detection, and telemetry dispatch pipeline operates across a deterministic, five-stage architecture:

```
Linux Kernel Event / Host Syscall (SSH Auth / execve Syscall / Terminal Keystroke / Network Socket)
    │
    ▼
[Stage 1: eBPF Kernel Probe Interception]
    │  - Attaches non-intrusive eBPF kprobes and tracepoints to kernel syscall hooks
    │  - Passively intercepts OpenSSH authentication events and interactive terminal I/O
    │  - Normalizes raw kernel ring buffer events into structured user space event streams
    ▼
[Stage 2: Process Lineage & Session Tracking]
    │  - Maps process execution trees (`execve`) to originating SSH session TTYs
    │  - Correlates user credentials, IP addresses, and command line arguments
    │  - Reconstructs exact terminal session transcripts with microsecond timestamps
    ▼
[Stage 3: Rule-Based Anomaly Detection & Threat Scoring]
    │  - Evaluates commands against MITRE ATT&CK patterns and security rulesets
    │  - Flags privilege escalation attempts (`sudo`, `su`), sensitive file reads, and reverse shells
    │  - Computes deterministic session threat risk scores
    ▼
[Stage 4: Automated Policy Enforcement & Incident Dispatch]
    │  - Emits real-time alerting events to enterprise SIEM endpoints (Elastic, Splunk, Datadog)
    │  - Optionally triggers administrative session termination for certified malicious activity
    │  - Verifies that passive monitoring does not impact host operating system stability
    ▼
[Stage 5: Immutable Audit Logging & Telemetry Commit]
    │  - Commits cryptographically hashed session transcripts to local tamper-evident logs
    │  - Scrubs private user passwords and authentication tokens from keystroke records
    │  - Emits structured telemetry metrics to Prometheus / Grafana exporters
    ▼
Validated Security Audit Event & Auditable eBPF Telemetry Record
```

### 2. Decision Logic & Anomaly Scoring Formulations

The monitor agent evaluates session risk, privilege escalation probability, and system anomaly scores using deterministic mathematical models:

1. **Session Threat Risk Score ($S_{\text{threat}}$)**:
   $$S_{\text{threat}} = \min\left( 1.0, \sum_{k=1}^K w_k \cdot \mathbb{I}(\text{PatternMatch}(e_k)) \right)$$
   where $\mathbb{I}(\text{PatternMatch}(e_k))$ flags signature behaviors (reverse shell, shadow file read, sudo abuse), with weights calibrated to CVSS severity tiers. Any session reaching $S_{\text{threat}} \ge 0.85$ triggers high-priority security alerts.

2. **System Health Stability Metric ($H_{\text{health}}$)**:
   $$H_{\text{health}} = 1 - \max\left( \frac{\text{CPU}_{\text{agent}}}{5\%}, \frac{\text{RAM}_{\text{agent}}}{128\text{MB}} \right)$$
   Ensuring that eBPF probe execution incurs $<5\%$ CPU overhead and zero kernel ring buffer drops.

### 3. Thresholding & Refusal Decision Criteria

Autonomous System & Infrastructure Monitor Agent enforces strict operational safety and legal compliance boundaries:
- **Refusal to Execute Remote Destructive Actions**: The agent is designed as an observability sensor; requests to execute arbitrary destructive commands on monitored hosts are rejected (`ERR_HOST_MUTATION_PROHIBITED`).
- **Refusal to Record Plaintext Passwords**: Keystroke buffers matching authentication password prompts (`Password:`, `PIN:`) are automatically masked at the kernel boundary (`WARN_PASSWORD_MASKED`).
- **Turn Ceiling Enforcement**: Analysis and alert dispatch cycles enforce bounded execution turns to eliminate infinite alert generation (`WARN_ALERT_RATE_LIMITED`).
- **Kernel Safety Boundary Confinement**: eBPF bytecode is validated through the Linux kernel BPF in-kernel verifier, ensuring zero kernel crash risks (`ERR_BPF_VERIFICATION_REQUIRED`).

### 4. Fallback Decision Mechanism

Continuous system observability is guaranteed through multi-tier fault recovery:
- **Model Cascade Failover**: When the primary foundation model experiences latency spikes or HTTP 429 rate limits, the orchestrator cascades automatically between `claude-3-5-sonnet`, `gpt-4o`, and `gemini-2.0-flash`.
- **Pure-eBPF Heuristic Fallback**: If LLM incident analysis fails, the agent operates entirely on deterministic rule-based eBPF kernel event filters.
- **Ring Buffer Overflow Protection**: If event generation rates exceed ingestion throughput, non-critical metrics are dropped while security audit events are prioritized.

### 5. Human-in-the-Loop Governance

Human security engineers and system administrators retain full operational command:
- **Administrator Review Primacy**: All incident alerts, compliance findings, and session audits are routed to security operation centers (SOC) for human validation.
- **Emergency Monitor Disconnect**: Operators can detach eBPF probes and stop monitoring instantly via `systemctl stop monitor-agent` or standard kill signals.
- **Transparent Audit Trail**: Every intercepted command, kernel timestamp, user UID, and TTY identifier is recorded in append-only logs for forensic review.

---

## The Data It Uses

Autonomous System & Infrastructure Monitor Agent operates under strict privacy, data minimization, and least-privilege standards.

### 1. Ingested Input Data

The agent processes only operational assets necessary to fulfill infrastructure observability:
- **eBPF Syscall Intercepts**: Kernel events corresponding to process executions (`execve`), network sockets, and file opens.
- **OpenSSH Session Events**: Authentication timestamps, client IP addresses, username identifiers, and TTY assignments.
- **Interactive Terminal Streams**: Standard I/O character streams passing through pseudo-terminal (PTY) devices.

### 2. Configuration & Reference Data

- **MITRE ATT&CK Threat Signatures**: Pattern rulesets defining suspicious administrative behaviors and exploit primitives.
- **eBPF Kernel Probe Manifests**: Bytecode configurations and kernel verifier constraints for supported Linux kernels.
- **SIEM Export Schemas**: Structured JSON formatting schemas for Syslog, Elasticsearch, and Prometheus.

### 3. Base Model & Inference Lineage

- **Deterministic Kernel Engines**: eBPF kernel filters, C/Go user-space collectors, and cryptographic hash formatters executed natively (100% deterministic with zero LLM variance).
- **Foundation LLMs**: High-capability frontier models (`claude-3-5-sonnet`, `gpt-4o`, `gemini-2.0-flash`) utilized exclusively for natural-language incident summarization and forensic report authoring.
- **Zero Training on Infrastructure Data**: Monitored terminal sessions, proprietary server topologies, and host telemetry are never transmitted to external cloud servers or used for model training.

### 4. Data Privacy, Storage, and Retention

- **OWASP LLM & MITRE ATLAS Hardened**: Defended against telemetry tampering, command injection, and unauthorized privilege escalation.
- **Local-Only Audit Storage**: All raw session transcripts and eBPF logs reside exclusively on the monitored host or designated enterprise SIEM sinks.
- **Automated Password Masking**: Password prompts and sensitive environment secrets are redacted before event serialization.
- **Zero Commercial Monetization**: Monitored telemetry, administrator transcripts, and security findings are never shared, monetized, or sold to third parties.

---

## Limitations

Understanding the operational boundaries and technical constraints of Autonomous System & Infrastructure Monitor Agent is essential for production deployment.

### 1. Linux Kernel Compatibility Requirements
- **Limitation**: eBPF monitoring requires modern Linux kernels (>=5.4) with BTF (BPF Type Format) enabled, excluding legacy kernel distributions.
- **Mitigation**: The agent runs pre-flight kernel checks, advising administrators on required kernel configurations before deployment.

### 2. Encrypted Out-of-Band Channel Evasion
- **Limitation**: Non-standard administrative sessions that bypass OpenSSH (e.g., direct serial console cables) require specialized probe attachments.
- **Mitigation**: The agent instruments core kernel syscalls (`sys_execve`, `sys_openat`) to track process execution regardless of entry channel.

### 3. High-Throughput Terminal Streaming CPU Overhead
- **Limitation**: Applications dumping massive binary streams (e.g., `cat /dev/urandom`) into terminal sessions can generate heavy eBPF ring buffer traffic.
- **Mitigation**: The agent implements dynamic rate limiting and skips binary non-ASCII payload streams to preserve host CPU headroom.

### 4. False-Positive Anomaly Triggers in DevOps Automation
- **Limitation**: Routine deployment scripts running administrative commands (e.g., Ansible, Puppet) can trigger naive anomaly detection rules.
- **Mitigation**: The system supports process whitelist tagging, recognizing trusted CI/CD service account service principals.

### 5. Multi-Tenant Cloud Container Ephemeral PID Namespace Shifting
- **Limitation**: Short-lived Docker containers can rapidly cycle process PIDs, challenging long-term historical correlation.
- **Mitigation**: The agent captures Linux cgroup IDs and container runtime metadata alongside process PIDs for durable correlation.

---

## Summary & Compliance Checklist

| Checkpoint 2 Requirement | Corresponding Section | Status |
| :--- | :--- | :---: |
| **How the agent decides** | [How the Agent Decides](#how-the-agent-decides) | **Covered** |
| - Decision architecture & 5-stage pipeline | Section 1 | Verified |
| - Decision logic & anomaly scoring formulations | Section 2 | Verified |
| - Thresholding & refusal decision criteria | Section 3 | Verified |
| - Fallback decision mechanism | Section 4 | Verified |
| - Human-in-the-loop governance & oversight | Section 5 | Verified |
| **The data it uses** | [The Data It Uses](#the-data-it-uses) | **Covered** |
| - Ingested eBPF intercepts, SSH events & terminal streams | Section 1 | Verified |
| - Configuration, threat signatures & eBPF manifests | Section 2 | Verified |
| - Base model lineage & deterministic engines | Section 3 | Verified |
| - Data privacy, retention lifecycle & MITRE/OWASP | Section 4 | Verified |
| **Its limitations** | [Limitations](#limitations) | **Covered** |
| - Linux kernel compatibility requirements | Section 1 | Verified |
| - Encrypted out-of-band channel evasion | Section 2 | Verified |
| - High-throughput terminal streaming CPU overhead | Section 3 | Verified |
| - False-positive anomaly triggers in DevOps automation | Section 4 | Verified |
| - Multi-tenant cloud container PID namespace shifting | Section 5 | Verified |
