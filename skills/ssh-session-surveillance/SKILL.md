---
name: "ssh-session-surveillance"
description: "Inspects OpenSSH sessions, tracking active TTY connections, user commands, and terminal I/O streams."
---

# SSH Session Surveillance

## Overview
This skill tracks and audits interactive OpenSSH sessions across Linux hosts. It correlates authenticated users, client IP addresses, pseudo-terminals (TTYs), and executed command streams.

## Key Capabilities
- **Session Attribution**: Binds process trees to active user TTY sessions and login times.
- **Command Logging**: Records complete command execution lines and standard outputs.
- **Password Masking**: Automatically detects password prompts and scrubs sensitive inputs from logs.

## Operational Workflow
1. **Session Detection**: Detect new user logins via `utmp` and SSH daemon fork events.
2. **TTY Binding**: Map process IDs to pseudo-terminal device numbers (`/dev/pts/*`).
3. **Stream Recording**: Write sanitized terminal inputs and outputs to encrypted append-only logs.
4. **Disconnect Handling**: Record session duration and aggregate final statistics upon logout.
