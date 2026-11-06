# Linux-Shell-History-Forensics-Lab-129
## Overview
Linux shell history forensics is the examination of command-line records to understand activities performed by users on a Linux system. These records can help investigators identify administrative operations, account changes, file modifications, service management, network activity, and commands that may be relevant to a security incident.

In Linux, the Bash shell maintains a history of commands entered during interactive sessions. This history can provide useful investigative leads, but it should not be treated as a complete or tamper-proof record of system activity.

### 1. Understanding Bash History

Bash maintains command history for the current shell session and can save it to a persistent history file.

The commonly used commands are:

```bash
history
```

Displays commands retained in the current shell session.

```bash
echo "$HISTFILE"
```

Displays the configured history file location.

```bash
cat ~/.bash_history
```

Displays the contents of the current user's persistent Bash history file.

The default history file locations are generally:

| Account | Default History File |
|---|---|
| Root | `/root/.bash_history` |
| Standard user | `/home/username/.bash_history` |

The actual location can differ depending on the user's shell configuration and environment.

### 2. Current Session History vs. Persistent History

It is important to distinguish between commands retained in the current shell session and commands written to the persistent history file.

**Current session history**

The `history` command displays commands available in the active shell's history list.

**Persistent history**

Bash commonly writes history entries to `.bash_history` when the shell exits. The command below can explicitly append the current session's history:

```bash
history -a
```

As a result, the persistent history file may not always contain every command visible in the active session.

This distinction matters during forensic collection because an investigator may find additional commands in the active shell that have not yet been written to disk.

### 3. Examining Shell History

Shell history can reveal commands associated with different types of system activity.

| Command Category | Examples | Investigative Relevance |
|---|---|---|
| Account management | `useradd`, `usermod`, `userdel` | Account creation, modification, or deletion |
| Permission changes | `chmod`, `chown` | Changes to file permissions and ownership |
| Service management | `systemctl start`, `enable`, `disable` | Service configuration and execution |
| File operations | `cp`, `mv`, `rm`, `touch` | File creation, modification, or deletion |
| Network activity | `curl`, `wget`, `ssh` | Network requests, downloads, and remote access |
| Scheduled tasks | `crontab`, `at` | Scheduled command execution |
| Log investigation | `journalctl`, `ausearch` | Inspection of system and audit records |

These commands provide investigative leads. Their presence alone does not establish malicious intent or prove that an operation completed successfully.

### 4. Understanding Bash History Timestamps

Bash can display timestamps alongside history entries when timestamp information is available.

For example:

```bash
HISTTIMEFORMAT='%F %T '
history
```

This format displays dates and times in a readable form.

However, timestamps require careful interpretation:

- Not every historical command necessarily has reliable timestamp information.
- Multiple entries may display the same timestamp.
- The displayed time depends on the available history timestamp data and system time configuration.
- System clock changes can affect timeline accuracy.
- History files can be modified by users with sufficient privileges.

An investigator should therefore avoid treating Bash history timestamps as independently verified execution times.

### 5. Examining History File Metadata

The history file itself is a forensic artifact. Its metadata can provide useful information about ownership, access permissions, size, and filesystem timestamps.

The following command can be used:

```bash
stat ~/.bash_history
```

Important metadata includes:

| Metadata | Forensic Relevance |
|---|---|
| File size | Indicates the amount of stored data |
| Owner and group | Identifies the account associated with the file |
| Permissions | Shows who can access or modify the file |
| Access time | Records the last reported access time |
| Modification time | Records the last content modification time |
| Change time | Records the last inode metadata change |
| Birth time | Indicates file creation time when supported by the filesystem |

These timestamps describe filesystem metadata events. They do not establish when every individual command was executed.

### 6. Correlating Shell History with Other Evidence

Shell history should be examined alongside independent system artifacts to strengthen investigative conclusions.

| Evidence Source | Investigative Value |
|---|---|
| Linux Audit logs | Can record audited command execution and related system activity |
| `/var/log/secure` | Provides authentication and privilege-related records |
| `/var/log/messages` | Contains system and service messages |
| `journalctl` | Provides systemd journal and service activity |
| `/etc/passwd` | Provides local account information |
| `/etc/group` | Provides group membership information |
| systemd unit files | Help identify service configuration |
| File metadata | Provides ownership, permissions, and filesystem timestamps |
| Cron configuration | Helps identify scheduled tasks |

For example, if Bash history contains a command to create a user, the investigator can examine `/etc/passwd` and relevant audit records for supporting evidence.

Similarly, a recorded `systemctl start` command can be compared with audit records and service journal entries. A successful command execution record does not automatically prove that the service remained active or became healthy.

### 7. Limitations of Shell History

Bash history has several limitations that must be considered during an investigation.

1. **Incomplete records:** Commands may not be saved to the persistent history file.
2. **History manipulation:** Users with sufficient privileges can delete, modify, or replace history entries.
3. **Disabled history:** Shell configuration can prevent commands from being recorded.
4. **Timestamp limitations:** Historical timestamps may be missing, repeated, or unreliable.
5. **Multiple user accounts:** Each user may maintain a separate history file.
6. **Non-interactive execution:** Commands executed through scripts, services, or other mechanisms may not appear in interactive shell history.
7. **Filesystem accessibility:** Corruption or unavailable mounts may prevent access to user-level history files.
8. **Execution uncertainty:** A recorded command does not independently prove successful execution or the resulting system state.

These limitations make evidence correlation essential.

### 8. Evidence Preservation and Investigation Approach

Shell history should be collected carefully to avoid altering the evidence being examined.

A practical investigation sequence is:

1. Identify the system, current user, shell, and history file location.
2. Examine the available current-session history.
3. Locate and collect relevant persistent history files.
4. Record file metadata, ownership, permissions, and timestamps.
5. Search for commands associated with accounts, permissions, services, files, and network activity.
6. Correlate important entries with audit records and other system artifacts.
7. Identify missing evidence and timestamp inconsistencies.
8. Document confirmed findings separately from assumptions and unresolved questions.

Where possible, preserve original evidence before performing investigative actions that could modify files or system state.

### 9. Key Forensic Principle

Linux shell history is a valuable source of investigative leads, but it is not a complete record of everything performed on a system.

A reliable investigation combines command history with audit records, system logs, account information, service configuration, and filesystem metadata.

**The objective is to establish what the available evidence supports, distinguish attempted commands from corroborated outcomes, and clearly document what remains unknown.**
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

## Lab Objectives

The objectives of this lab are to:

1. Understand how Bash maintains command history during interactive shell sessions and stores it in persistent history files.

2. Locate and examine root and user-level Bash history files to identify recorded administrative and system activities.

3. Investigate commands associated with account management, privilege changes, file permissions, service operations, and scheduled tasks.

4. Examine Bash history timestamps and identify limitations that may affect the accuracy of activity timelines.

5. Inspect history file metadata, including ownership, permissions, file size, and modification timestamps.

6. Correlate recorded shell commands with Linux Audit logs, system logs, account information, and service-related artifacts.

7. Differentiate between commands that were entered, commands whose execution is corroborated, and actions whose final outcome remains unverified.

8. Identify potential evidence gaps caused by missing history entries, inaccessible user home directories, incomplete audit records, or filesystem issues.

9. Develop an evidence-based understanding of Linux administrative activity without assuming that every recorded command represents malicious behavior.

10. Document forensic findings, timestamp limitations, troubleshooting observations, and unresolved questions in a structured investigation report.

## Lab Scenario

A Rocky Linux 9.8 system was used for multiple administrative, security monitoring, and hands-on cybersecurity lab activities. During a review of the system, the need arose to examine recorded command-line activity and understand which administrative operations had been performed, which accounts were involved, and whether the available records could support a reliable activity timeline.

The investigation focused on Bash shell history, particularly the root user's persistent history file at `/root/.bash_history`. The recorded commands included account-management operations, file permission changes, Wazuh service commands, systemd service configuration, and other activities associated with previous controlled lab exercises.

### Investigation Context

The system had also experienced an XFS filesystem issue affecting the `/home` logical volume. The filesystem could not be mounted successfully, so its mount entry was disabled in `/etc/fstab` on the backup virtual machine to allow the operating system to boot.

This recovery activity introduced an important forensic limitation: user-level history files stored under `/home` were not available for examination. The investigation therefore concentrated on accessible root-level evidence and other system artifacts.

### Investigation Tasks

The analyst was required to examine the available evidence and determine what could reasonably be established.

Key activities included:

- Examine root's Bash history and identify commands associated with administrative and security-related operations.
- Review history file metadata to understand its ownership, permissions, size, and timestamps.
- Search for account creation, permission changes, service management, and file download commands.
- Correlate selected shell history entries with Linux Audit records and available system information.
- Review Wazuh service-management activity without assuming that a recorded start command proves the service became healthy.
- Examine timestamp consistency and identify limitations in reconstructing the exact order of events.
- Document the impact of the inaccessible `/home` filesystem on evidence collection.

### Forensic Considerations

The investigation must distinguish between commands recorded in shell history and actions independently corroborated by other evidence. For example, a command to create a user may be supported by account information, while a service start command may have an audit record confirming command execution without proving the service reached an active state.

Similarly, commands associated with earlier controlled lab exercises should not automatically be classified as suspicious. Their purpose and surrounding context must be considered before drawing conclusions.

### Expected Outcome

The objective is to reconstruct the available administrative activity as accurately as possible while identifying evidence gaps and uncertainties.

The final assessment should explain:

- Which activities are supported by the available evidence.
- Which findings are corroborated by independent artifacts.
- Which timestamps or command outcomes remain uncertain.
- How filesystem unavailability affected the investigation.
- Whether the collected evidence supports a conclusion of suspicious or malicious activity.

The investigation follows an evidence-first approach: **record what the artifacts establish, distinguish it from what they merely suggest, and clearly document what cannot be verified.**

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

