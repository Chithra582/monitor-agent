---
name: "anomaly-alert-triggering"
description: "Dispatches automated Slack, webhook, and syslog alerts upon detecting suspicious shell operations."
---

# Anomaly Alert Triggering

## Overview
This skill evaluates real-time SSH session activity against security policies, flagging unauthorized privilege escalations, suspicious network connections, and data exfiltration attempts.

## Key Capabilities
- **Pattern Matching**: Matches command strings against known attack patterns and suspicious binaries.
- **Severity Scoring**: Classifies events into informational, warning, and critical threat levels.
- **Multi-Channel Dispatch**: Sends immediate notifications to Slack, remote syslog servers, or incident response webhooks.

## Operational Workflow
1. **Rule Evaluation**: Scan executed commands and file paths against threat rulesets.
2. **Threat Assessment**: Calculate event severity and determine required alerting actions.
3. **Rate Throttling**: Apply rate limits to prevent alert storms during repetitive commands.
4. **Notification Dispatch**: Push structured JSON alert payloads to configured destination sinks.
