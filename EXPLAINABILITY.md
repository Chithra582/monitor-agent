# EXPLAINABILITY — Autonomous System & Infrastructure Monitor Agent

> **Admissibility & Transparency Report for OpenGAP / Agent Passport**  
> *Agent Name:* Autonomous System & Infrastructure Monitor Agent (`monitor-agent`)  
> *Specification:* OpenGAP v0.1.0  
> *Domain:* Developer Tools / Infrastructure Monitoring & eBPF Telemetry  

---

## 1. Overview & Operational Purpose
The **Autonomous System & Infrastructure Monitor Agent** is an open-source Linux observability and security agent utilizing extended Berkeley Packet Filters (eBPF) to passively inspect OpenSSH sessions and host process executions. It provides security engineers and system administrators with granular visibility into active terminal sessions without requiring custom SSH daemons or wrapper shells.

Its operational purpose is to detect security incidents, enforce administrative compliance, record immutable session transcripts, and stream system health metrics to enterprise SIEM and observability stacks.

---

## 2. How the Agent Decides (Decision-Making Logic)
Autonomous System & Infrastructure Monitor Agent operates across a deterministic, multi-stage decision pipeline:

```
[Stage 1: eBPF Probe Intake] ──> [Stage 2: Session Correlation] ──> [Stage 3: Policy & Anomaly Check]
                                                                                │
                                                                                ▼
[Stage 6: Telemetry Flush] <── [Stage 5: Alert Dispatch] <── [Stage 4: Threat Assessment]
```

### 2.1 eBPF Probe Intake & Kernel Event Capture
- **Decision:** Capture raw kernel tracepoint events (sys_enter_execve, tty_write) from the Linux kernel ring buffer.
- **Rules:** Reject events originating from non-monitored daemon processes; buffer valid syscall events in locked memory queues.

### 2.2 Session Correlation & Attribution
- **Decision:** Associate captured process execution and terminal output events with a verified SSH session context (User, Client IP, TTY).
- **Rules:** Match process parentage (PPID) against the active SSHD process tree; assign session identifier and timestamp.

### 2.3 Policy & Anomaly Check
- **Decision:** Evaluate executed command strings and file access paths against security policy rulesets and suspicious binary lists.
- **Rules:** Flag unauthorized privilege escalations, reverse shell payloads, or directory traversals; mask sensitive password prompts.

### 2.4 Threat Assessment & Alert Dispatch
- **Decision:** Determine severity level (Info, Warning, Critical) and route event notifications to configured alerting backends.
- **Rules:** Throttle repeated alert occurrences; stream structured JSON events to syslog, Slack webhooks, and StatsD metric sinks.

---

## 3. Data & Privacy
| Data Category | Retention Policy | Third-Party Sharing | Storage Mechanism |
|---|---|---|---|
| SSH Session Transcripts | Configured Retention (e.g. 90 Days) | None | Local Append-Only Files (`/var/log/sshlog/`) |
| Client IP & Login Records | Audit Archive Horizon | None | Local SQLite / Syslog |
| eBPF Kernel Telemetry Events | Ephemeral (Stream Processing) | None | In-Memory Kernel Ring Buffers |
| Aggregate Performance Metrics | Permanent (Time Series) | Configured StatsD / Prometheus | External Metric Sinks |

Autonomous System & Infrastructure Monitor Agent complies with operational security and privacy standards:
- **No Cloud Data Exfiltration:** All captured terminal logs and kernel events remain strictly on the host system unless explicitly configured to forward to private enterprise syslog/StatsD endpoints.
- **Epistemic Isolation:** The daemon runs with bounded read-only inspection access to kernel tracepoints, isolated from user data directories and application databases.
- **Sanitized Model Payloads:** Automated alerts and LLM diagnostic reports are stripped of user passwords, confidential environment variables, and authentication tokens.
- **Data Minimization:** Only commands, terminal output, and session metadata relevant to security auditing are recorded; non-interactive background system processes are excluded.

---

## 4. Known Limitations & Failure Modes
Reviewers, auditors, and users should note the following operational constraints:
1. Kernel Version Compatibility
   - *Limitation:* Advanced eBPF features require Linux kernel version 4.18 or higher with BPF and tracepoints enabled.
   - *Mitigation:* The agent performs pre-flight kernel capability checks and gracefully falls back to standard auditd interfaces on legacy kernels.
2. High-Throughput Terminal Flooding
   - *Limitation:* Rapid bulk terminal output (e.g., executing `cat /dev/urandom` or large binary dumps) can saturate ring buffer queues.
   - *Mitigation:* The daemon enforces maximum throughput rate limits per TTY and drops excessive output frames with a warning flag.
3. Binary Obfuscation and Alias Evasion
   - *Limitation:* Malicious actors using encoded payloads (e.g., `base64 -d | sh`) may evade simple command-name string matching rules.
   - *Mitigation:* The eBPF probes inspect low-level `execve` syscalls directly, intercepting the true binary path and resolved arguments.
4. Containerized Host Isolation
   - *Limitation:* Running inside unprivileged Docker containers prevents the agent from attaching to host kernel tracepoints.
   - *Mitigation:* The agent documentation specifies required privileged capability flags (`--privileged` or `CAP_BPF` / `CAP_SYS_ADMIN`).

---

## 5. Verification, Safety & Human Oversight
Autonomous System & Infrastructure Monitor Agent integrates multi-layer safety rails to ensure full human accountability and system integrity:
- **Real-Time Human Approval Gate:** Terminating active sessions, disconnecting users, or injecting commands into live terminals requires explicit administrator authorization.
- **Emergency Session Interrupt:** The monitoring daemon can be cleanly stopped or unloaded (`systemctl stop sshlog`) instantly removing all kernel hooks.
- **Step Quota Guardrails:** Strict buffer memory boundaries and webhook dispatch quotas prevent denial-of-service impacts on host systems.
- **Structured Audit Logging:** Every configuration change, probe attachment, user login, and security alert is recorded in immutable, timestamped audit files.
