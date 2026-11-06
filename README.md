# Lab 129 – Linux Shell History Forensics

## Overview

This lab explores how Linux Bash shell history can support host-based forensic investigations. Shell history provides a record of commands entered by users and can help investigators identify administrative activity, system changes, downloads, file operations, and commands that warrant further examination.

The investigation was performed on a Rocky Linux 9 system. It included examination of the root user's Bash history, history timestamps, file metadata, account information, and available audit records. The activity also occurred during recovery from an XFS `/home` filesystem problem, providing an opportunity to understand how system availability and filesystem issues can affect forensic collection.

The investigation follows an evidence-based approach: shell history is treated as an investigative artifact rather than a complete or authoritative record of all command execution.

## Lab Environment

| Component | Details |
|---|---|
| Lab Number | 129 |
| Lab Title | Linux Shell History Forensics |
| Operating System | Rocky Linux 9 |
| Shell | GNU Bash 5.1 |
| Hostname | localhost.localdomain |
| IP Address | 192.168.203.129 |
| Primary Account Examined | root |
| History File | `/root/.bash_history` |
| Additional Evidence | Linux audit records, system logs, account database |
| Investigation Type | Linux Host-Based Forensics |

## Investigation Objectives

- Understand how Bash records and stores command history.
- Identify the location of user-specific history files.
- Examine commands associated with account management, permissions, service operations, and file activity.
- Review available history timestamps and file metadata.
- Correlate selected command activity with Linux audit records.
- Identify limitations of shell history as forensic evidence.
- Document the impact of an unavailable `/home` filesystem on user-level evidence collection.

## Investigation Scenario

A Rocky Linux system was being examined after experiencing an XFS filesystem problem affecting the `/home` logical volume. The filesystem could not be mounted, so the system was recovered by temporarily disabling the `/home` mount entry in `/etc/fstab`. The existing filesystem was not formatted or recreated.

After the system became accessible, the root user's Bash history was examined to understand recent administrative activity and to identify commands executed during the recovery process.

The history also contained activity from earlier security labs, including Wazuh service operations and a controlled systemd persistence demonstration from Lab 128. These entries provided useful examples of how legitimate administrative and lab activity can appear during a forensic review.

The objective was not to classify every command as malicious or benign based on its name. Instead, the investigation focused on identifying recorded activity, correlating available evidence, and documenting what could and could not be established.

## Investigation Scope

The investigation covered:

- `/root/.bash_history`
- Bash history output using the `history` command
- History timestamps and `HISTTIMEFORMAT`
- Shell history file metadata
- Local account information from `getent passwd`
- Linux audit records containing `USER_CMD` events
- Selected systemd and Wazuh service commands
- Recovery-related commands associated with `/etc/fstab` and filesystem mounting

The Windows Task Scheduler Event ID 129 output shown separately was not included as Linux host evidence. It belongs to a different operating system and requires a separate investigation context.

## Tools and Commands Used

| Tool / Command | Purpose |
|---|---|
| `history` | Display commands recorded in the current Bash session |
| `history -a` | Append current session history to the history file |
| `cat` | Read the Bash history file |
| `grep` | Search history for selected command patterns |
| `stat` | Examine history file metadata |
| `getent passwd` | Review local and directory-backed account information |
| `ausearch` | Examine Linux audit records |
| `journalctl` | Review system and service logs |
| `systemctl` | Review and manage system services |
| `findmnt` | Examine filesystem mount information |
| `mount` | Review current mount state |

## Investigation Workflow

### 1. Identify the shell environment

The following commands were used to establish the account and shell context:

```bash
whoami
hostname
date
echo $SHELL
echo $HOME
echo $HISTFILE
```

Observed results:

- Current user: `root`
- Hostname: `localhost.localdomain`
- Shell: `/bin/bash`
- Home directory: `/root`
- History file: `/root/.bash_history`
- Host IP address: `192.168.203.129`

### 2. Generate controlled command activity

A small set of harmless commands was executed to create known history entries:

```bash
whoami
hostname
date
mkdir -p ~/lab129
touch ~/lab129/testfile.txt
chmod 600 ~/lab129/testfile.txt
```

This created a test directory and file under the root user's home directory and changed the file permissions to owner read/write only.

### 3. Examine Bash history

The current session history was reviewed using:

```bash
history
history 20
history | grep chmod
history | grep -E 'useradd|usermod|userdel'
history | grep -E 'curl|wget'
```

The history was also appended to the persistent history file:

```bash
history -a
```

The root user's history file was examined using:

```bash
cat ~/.bash_history
```

### 4. Review timestamps and metadata

History output was displayed with a timestamp format:

```bash
HISTTIMEFORMAT='%F %T '
history
```

The history file metadata was collected using:

```bash
stat ~/.bash_history
```

The file was found to be owned by root with restrictive permissions.

### 5. Identify administrative activity

Searches were performed for account management, service operations, permission changes, downloads, and history manipulation.

Examples included:

```bash
history | grep -Ei 'sudo|su|passwd|useradd|userdel|usermod'
history | grep -Ei 'systemctl|firewall-cmd|iptables'
history | grep -Ei 'curl|wget|bash -c|chmod \+x'
history | grep -Ei 'rm |shred|history -c|unset HISTFILE'
```

These searches were used to identify areas requiring further examination, not to establish malicious intent.

### 6. Correlate with audit evidence

Linux audit records were reviewed for selected `USER_CMD` events.

The supplied audit output included records associated with Wazuh service start commands and earlier administrative activity.

Audit records provided an additional source for command attribution and execution context. However, a successful audit result for a `sudo` command does not independently establish the final state of the requested service or operation.

## Key Findings

| Area | Observation | Assessment |
|---|---|---|
| Shell identity | Root account and Bash shell were identified | Confirmed |
| History location | `/root/.bash_history` was identified | Confirmed |
| History file permissions | File owned by root with mode `0600` | Confirmed |
| Controlled file activity | Lab directory and test file commands appeared in history | Confirmed as recorded commands |
| Permission change | `chmod 600` appeared in history | Confirmed as recorded command |
| Account management | `useradd -m lab129user` appeared in history | Confirmed as recorded command |
| Wazuh service operations | Start and restart commands appeared in history | Confirmed as recorded commands |
| Lab 128 activity | systemd persistence demonstration and cleanup commands appeared in history | Confirmed as recorded commands |
| Audit correlation | Selected `USER_CMD` records were available | Confirmed |
| `/home` filesystem | Mount was disabled during recovery because of the XFS problem | Confirmed |
| User-level history | `/home` was unavailable during this investigation | Collection limitation |

## Important Forensic Observations

### Shell history is not a complete execution record

Bash history can be modified, cleared, or disabled. Commands may also remain in memory until written to the history file. Non-interactive command execution may not be recorded in the same way as interactive shell activity.

### Timestamps require careful interpretation

The supplied history output showed many entries with the same timestamp, while later commands displayed different times. This does not establish that all commands were executed simultaneously.

Timestamp behavior, shell-session history, and the system clock must be considered before constructing a definitive timeline.

### Audit records provide additional context

The audit output included `USER_CMD` records for Wazuh service start operations. These records help corroborate command activity, but do not prove that the Wazuh services reached an active state.

### The `/home` problem affected collection scope

The `/home` logical volume could not be mounted because of XFS corruption. The mount entry was temporarily commented out in `/etc/fstab` to allow the installed system to boot.

The root user's history remained accessible at `/root/.bash_history`. However, history files belonging to users whose home directories reside under `/home` could not be examined through the normal mounted paths.

## Evidence Limitations

- Shell history does not prove that a command completed successfully.
- History entries may be deleted, altered, or absent.
- The timestamp output contains repeated values that require cautious interpretation.
- Audit records cover only activity captured by the configured audit rules.
- The supplied audit excerpts do not establish the final state of Wazuh services.
- The `/home` filesystem was not mounted, so user-level artifacts under `/home` were outside the accessible collection scope.
- The separate Windows Task Scheduler Event ID 129 output was excluded from the Linux investigation.

## Related Documentation

- `investigation-notes.md` – Detailed examination and evidence assessment.
- `troubleshooting-notes.md` – Collection issues, recovery observations, and forensic limitations.
- `timeline.md` – Chronological reconstruction of the observed activity.

## MITRE ATT&CK Relevance

The following techniques may be relevant when investigating shell activity. Their presence in this mapping does not mean that the corresponding behavior was malicious in this lab.

| Technique | ID | Relevance |
|---|---|---|
| Command and Scripting Interpreter: Unix Shell | T1059.004 | Bash command execution |
| Account Manipulation | T1098 | Investigation of account creation and modification commands |
| Indicator Removal | T1070 | Investigation of attempts to clear or manipulate shell history |
| System Services | T1569.002 | Investigation of service execution through systemd |

## Conclusion

This lab demonstrated how Bash shell history can provide useful investigative context during a Linux host-based forensic examination.

The review identified administrative commands, controlled file operations, account-management activity, Wazuh service operations, and commands associated with earlier security labs. Selected audit records provided additional evidence of command activity.

The investigation also demonstrated that filesystem availability directly affects forensic collection. Although the root user's history was accessible, the damaged `/home` filesystem limited access to user-level artifacts.

The central lesson is to treat shell history as one source of evidence, correlate it with independent telemetry, and clearly document uncertainty rather than drawing conclusions from command names alone.

**Investigation principle: Follow the recorded activity, validate it against independent evidence, and distinguish what is confirmed from what remains unknown.**
