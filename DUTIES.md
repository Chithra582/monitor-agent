# DUTIES — Autonomous System & Infrastructure Monitor Agent

## Primary Duties
1. **eBPF Probe Lifecycle & Kernel Monitoring**:
   - Compile and load eBPF bytecode targeting kernel tracepoints and kprobes.
   - Stream kernel ring buffer events into user-space daemon consumers without dropped packets.
   - Maintain kernel probe health and re-attach upon SSH daemon restarts.
2. **SSH Session Tracking & Attribution**:
   - Monitor `/var/log/btmp`, `/var/run/utmp`, and kernel tty write events to register new sessions.
   - Correlate connecting client IP addresses, authentication methods, and allocated TTY numbers.
   - Track session duration, idle times, and terminal disconnect events.
3. **Anomaly Detection & Alerting**:
   - Evaluate executed shell commands against known security policy violation rules.
   - Identify unauthorized attempts to alter system binaries, tamper with logs, or download payloads.
   - Dispatch formatted alerts to Slack channels, remote syslog servers, and custom webhooks.
4. **Telemetry & Metrics Export**:
   - Aggregate per-user and per-host statistics into standard StatsD metric formats.
   - Expose Prometheus-compatible metrics endpoints for host monitoring dashboards.
   - Provide command-line tools to list active sessions and historical activity records.
