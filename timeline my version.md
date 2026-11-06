# Investigation Timeline

## 1. Timeline Scope

This timeline reconstructs the activity visible in the supplied terminal output, Bash history, filesystem recovery records, and audit excerpts from 6 November 2026.

The timeline distinguishes between:

- Commands recorded in Bash history.
- Commands corroborated by audit records.
- Observed system states.
- Recovery actions.
- Events whose exact timestamps or outcomes remain uncertain.

**Timestamp caution:** Many Bash history entries displayed the same timestamp (`2026-11-06 10:57:39`). These values are not treated as independently verified execution times. Audit output also displayed timestamps in a different time representation. The sources have not been normalized into a single clock because their relationship has not been established.

## 2. Investigation Timeline

| Sequence | Timestamp / Period | Source | Activity / Observation | Assessment |
|---|---|---|---|---|
| 1 | Before 6 Nov 2026 | Filesystem recovery context | XFS corruption affected the `/home` logical volume | Confirmed from recovery observations |
| 2 | 6 Nov 2026, recovery period | `/etc/fstab` | `/home` mount entry was changed to prevent automatic mounting | Confirmed |
| 3 | 6 Nov 2026, recovery period | Root shell | Root filesystem was found mounted read-only | Confirmed |
| 4 | 6 Nov 2026, recovery period | Root shell | Root filesystem was remounted read-write | Confirmed command issued without reported error |
| 5 | 6 Nov 2026, recovery period | `/etc/fstab` | `/home` entry was commented out | Confirmed |
| 6 | 6 Nov 2026, recovery period | systemd | `systemctl daemon-reload` and reboot were performed | Confirmed as recorded commands |
| 7 | 6 Nov 2026, 10:57:46 | Bash history | Wazuh manager start command recorded | Confirmed as recorded command |
| 8 | 6 Nov 2026, 10:57:53 | Bash history | Wazuh indexer start command recorded | Confirmed as recorded command |
| 9 | 6 Nov 2026, 10:58:01 | Bash history | Wazuh dashboard start command recorded | Confirmed as recorded command |
| 10 | 6 Nov 2026, 11:03:29–11:04:41 | Bash history and terminal output | Root identity, hostname, date, test directory, test file, and permissions examined or created | Confirmed |
| 11 | 6 Nov 2026, 11:05–11:08 | Bash history | History searches and persistent history examination performed | Confirmed |
| 12 | 6 Nov 2026, 11:08:15 | File metadata | `/root/.bash_history` modification and change time recorded | Confirmed |
| 13 | 6 Nov 2026, 11:09:16 | Bash history | `HISTTIMEFORMAT` configured and history displayed | Confirmed |
| 14 | 6 Nov 2026, 11:11:04–11:11:46 | Bash history | Additional command searches performed | Confirmed |
| 15 | 6 Nov 2026, audit excerpt | Linux audit log | Wazuh service start commands appeared in `USER_CMD` records | Corroborated command activity |

## 3. Detailed Timeline Interpretation

### Phase 1 – Filesystem Recovery

The investigation took place after the `/home` XFS filesystem became inaccessible.

The root filesystem was initially mounted read-only, preventing changes to `/etc/fstab`. After remounting `/` read-write, the `/home` entry was commented out.

The system was then able to boot without attempting the automatic `/home` mount.

This restored access to the installed Rocky Linux environment but did not repair the damaged filesystem.

### Phase 2 – Wazuh Service Operations

The Bash history contained the following commands:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
```

The displayed history timestamps were:

```text
10:57:46  wazuh-manager
10:57:53  wazuh-indexer
10:58:01  wazuh-dashboard
```

The supplied audit excerpt also contained corresponding `USER_CMD` records.

These records corroborate that the service start commands were issued through `sudo`.

The final active state and application health of the services were not established by the supplied evidence.

### Phase 3 – Controlled Lab Activity

The following commands were executed during the Lab 129 exercise:

```bash
whoami
hostname
date
mkdir -p ~/lab129
touch ~/lab129/testfile.txt
chmod 600 ~/lab129/testfile.txt
```

The terminal output showed:

```text
root
localhost.localdomain
Friday 06 November 2026 11:03:41 AM IST
```

The commands created a controlled directory and file under `/root`.

The permission command was recorded in history as:

```bash
chmod 600 ~/lab129/testfile.txt
```

The command history supports the recorded activity. Independent file metadata would be needed to verify the resulting file state.

### Phase 4 – Shell History Examination

The following commands were used to examine the root user's shell history:

```bash
history
history 20
history | grep chmod
history | grep -E 'useradd|usermod|userdel'
history | grep -E 'curl|wget'
cat ~/.bash_history
history -a
```

The history searches identified activity associated with:

- Account creation.
- Permission changes.
- Wazuh service operations.
- systemd persistence testing.
- Controlled lab execution.
- Filesystem recovery.

The search results provided investigative leads rather than proof of malicious behavior.

### Phase 5 – History File Metadata

The root history file was examined using:

```bash
stat ~/.bash_history
```

The supplied output showed:

```text
File: /root/.bash_history
Size: 20106
Inode: 100663429
Access: (0600/-rw-------)
Uid: (0/root)
Gid: (0/root)
Birth: 2026-11-06 10:56:31.574020943 +0530
Modify: 2026-11-06 11:08:15.262895818 +0530
Change: 2026-11-06 11:08:15.262895818 +0530
```

The file was owned by root and had restrictive permissions.

The metadata establishes the observed file attributes and timestamps. It does not prove that the history contents were complete or unmodified.

### Phase 6 – Timestamp Review

The history was displayed using:

```bash
HISTTIMEFORMAT='%F %T '
history
```

Many earlier entries displayed the same timestamp, while later entries showed different values.

This limited the reliability of a strict chronological reconstruction based solely on Bash history.

The audit records were retained as a separate source rather than silently merging their timestamps with the history output.

## 4. Relevant Historical Activity

The root history also contained commands from earlier security labs.

### Lab 128 – systemd Persistence Demonstration

The history showed service operations involving:

```text
lab128-persistence-demo.service
```

Recorded actions included:

- Service file inspection.
- Syntax verification.
- systemd daemon reload.
- Service enablement and execution.
- Service modification and restart.
- Journal inspection.
- Service disablement and removal.
- systemd failed-state cleanup.

These commands are consistent with the controlled Lab 128 exercise.

Their appearance in the history does not establish malicious persistence.

### Account Management

The history contained:

```bash
sudo useradd -m lab129user
```

The account database also contained the `lab129user` account with UID 1001 and home directory `/home/lab129user`.

Because `/home` was unavailable during the investigation, the accessibility and contents of that home directory were not established.

### Mock Ticket API

The history contained a command that launched:

```text
/opt/wazuh-ticket-lab/mock_ticket_api.py
```

using `nohup` and Python, with output redirected to a log file.

This is consistent with a controlled mock ticket API lab component. The supplied evidence does not establish its final process state.

## 5. Evidence Source Reliability

| Evidence Source | Reliability / Value | Limitation |
|---|---|---|
| Terminal output | Direct observation of displayed command results | Covers only the captured session |
| Bash history | Records commands retained by the shell | Can be incomplete or manipulated |
| History timestamps | May help establish command timing | Repeated timestamps require caution |
| `stat` output | Provides filesystem metadata | Does not establish content authenticity |
| Account database | Establishes account entries at collection time | Does not prove home directory availability |
| Audit `USER_CMD` records | Independent command-activity evidence | Limited to captured events and configured audit rules |
| systemd status output | Can establish service state at collection time | Relevant status evidence was not included for all services |
| `/etc/fstab` | Establishes the configured mount entry | Does not establish filesystem recovery |

## 6. Events Excluded from This Timeline

A separate screenshot showed Windows Task Scheduler Operational Event ID 129 for:

```text
\Microsoft\Windows\Flighting\FeatureConfig\ReconcileFeatures
```

This was generated on a Windows endpoint and was not included in the Rocky Linux timeline.

It requires a separate Windows investigation and cannot be correlated with Linux shell commands from the supplied evidence.

