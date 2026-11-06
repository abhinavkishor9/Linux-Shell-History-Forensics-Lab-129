# Lab 129 – Linux Shell History Forensics
## Investigation Notes

## 1. Investigation Objective

The objective was to examine Bash shell history on a Rocky Linux 9 system and determine what administrative and user activity could be identified from available command-history artifacts.

The investigation also examined history timestamps, file metadata, account information, and selected Linux audit records to understand the value and limitations of shell history during a DFIR investigation.

## 2. Investigation Scope and Host Information

| Attribute | Observed Value |
|---|---|
| Operating System | Rocky Linux 9 |
| Hostname | localhost.localdomain |
| IP Address | 192.168.203.129 |
| Shell | `/bin/bash` |
| Shell Version | Bash 5.1 |
| User Examined | root |
| Home Directory | `/root` |
| History File | `/root/.bash_history` |
| Investigation Date | 6 November 2026 |

The investigation was conducted after recovery of the installed Rocky Linux system from a boot problem caused by an inaccessible XFS `/home` filesystem.

The `/home` entry was temporarily disabled in `/etc/fstab` to allow the system to boot. The filesystem was not formatted or recreated.

## 3. Analyst Methodology

The investigation followed these steps:

1. Identify the current account, shell, hostname, and history file.
2. Generate a small amount of controlled command activity.
3. Examine the current shell history and persistent history file.
4. Search for administrative and potentially suspicious commands.
5. Examine history file metadata and timestamp behavior.
6. Review account information.
7. Correlate selected command activity with Linux audit records.
8. Document confirmed observations and collection limitations.

The investigation avoided treating command names as proof of malicious activity.

## 4. Evidence 1 – Shell Environment Identification

### Commands Used

```bash
whoami
hostname
date
echo $SHELL
echo $HOME
echo $HISTFILE
hostname -I
```

### Observations

| Command | Result |
|---|---|
| `whoami` | root |
| `hostname` | localhost.localdomain |
| `date` | Friday, 6 November 2026, 11:03:41 AM IST |
| `echo $SHELL` | `/bin/bash` |
| `echo $HOME` | `/root` |
| `echo $HISTFILE` | `/root/.bash_history` |
| `hostname -I` | `192.168.203.129` |

### Assessment

The investigation was conducted from a root Bash session. The persistent history file associated with this session was `/root/.bash_history`.

This establishes the context for interpreting the commands collected during the investigation.

## 5. Evidence 2 – Controlled Command Activity

### Commands Executed

```bash
whoami
hostname
date
mkdir -p ~/lab129
touch ~/lab129/testfile.txt
chmod 600 ~/lab129/testfile.txt
```

### Observations

The following commands were visible in the supplied terminal output and history:

- User identity check using `whoami`.
- Hostname check using `hostname`.
- Date and time check using `date`.
- Creation of the `lab129` directory.
- Creation of `testfile.txt`.
- Permission change using `chmod 600`.

### Assessment

These commands formed a controlled activity sequence for the lab.

The `chmod 600` command is particularly useful for demonstrating how shell history can reveal permission-related operations.

The history entry establishes that the command was recorded. The resulting file permissions should be independently verified with `stat` or `ls -l` before treating the resulting filesystem state as confirmed.

## 6. Evidence 3 – Persistent Bash History

### History File

```text
/root/.bash_history
```

### Commands Used

```bash
history
history 20
echo $HISTFILE
cat ~/.bash_history
history -a
```

### Observations

The root user's history file was present and readable.

The following metadata was later collected:

| Attribute | Observed Value |
|---|---|
| File | `/root/.bash_history` |
| Size | 20,106 bytes |
| Blocks | 40 |
| File Type | Regular file |
| Permissions | `0600` |
| Owner | root |
| Group | root |
| Inode | 100663429 |
| Device | fd00h/64768d |
| Link Count | 1 |
| Birth Time | 6 November 2026, 10:56:31 +0530 |
| Modification Time | 6 November 2026, 11:08:15 +0530 |
| Change Time | 6 November 2026, 11:08:15 +0530 |
| Access Time | 6 November 2026, 10:57:39 +0530 |

### Assessment

The history file was owned by root and had restrictive permissions (`0600`).

Its modification time was later than its recorded birth time, consistent with subsequent changes to the file.

File metadata alone does not establish who modified the history file or whether its contents were altered. Further correlation would be required.

## 7. Evidence 4 – History Timestamp Examination

### Command Used

```bash
HISTTIMEFORMAT='%F %T '
history
```

### Observation

The displayed history contained timestamp information.

However, many older entries displayed the same timestamp:

```text
2026-11-06 10:57:39
```

Later entries included different displayed times, such as:

```text
2026-11-06 10:57:46
2026-11-06 10:57:53
2026-11-06 10:58:01
2026-11-06 11:03:29
2026-11-06 11:11:04
```

### Assessment

The repeated timestamp values should not be interpreted as proof that all associated commands occurred at exactly the same moment.

`HISTTIMEFORMAT` controls how stored history timestamps are displayed. It does not retroactively establish accurate timestamps for commands that were not originally recorded with reliable timing information.

The raw history file and independent audit records should be examined when precise event ordering is important.

## 8. Evidence 5 – Administrative Command Searches

The history was searched for account-management, privilege-related, and administrative commands.

### Account Management

```bash
history | grep -Ei 'sudo|su|passwd|useradd|userdel|usermod'
```

A recorded entry included:

```bash
sudo useradd -m lab129user
```

The account database also contained:

```text
lab129user:x:1001:1001::/home/lab129user:/bin/bash
```

The existing account `abhinavmathur` was also present:

```text
abhinavmathur:x:1000:1000:abhinav Mathur:/home/abhinavmathur:/bin/bash
```

### Assessment

The account-creation command was recorded in shell history, and the `lab129user` account entry was visible in the account database.

Because `/home` was unavailable during recovery, the actual state and accessibility of `/home/lab129user` could not be established from the supplied evidence.

### Permission Changes

```bash
history | grep chmod
```

Relevant entries included:

```bash
sudo chmod u+s /var/tmp/lab126/suid-test
sudo chmod g+s /var/tmp/lab126/sgid-test
sudo chmod 0644 /etc/cron.d/lab127_heartbeat
sudo chmod 0644 /etc/systemd/system/lab128-persistence-demo.service
chmod 600 ~/lab129/testfile.txt
```

### Assessment

The history contained permission changes from earlier controlled security labs and the current Lab 129 exercise.

SUID and SGID permission changes can be security-relevant, but their presence in this history is consistent with earlier lab activity. The commands should not be classified as malicious without examining the associated files, ownership, timestamps, and scenario.

### Download and Execution Searches

```bash
history | grep -E 'curl|wget'
```

The output included a truncated entry:

```bash
curl -sS http://127.0.0
```

A broader search returned:

```bash
sudo bash -c 'nohup /usr/bin/python3 /opt/wazuh-ticket-lab/mock_ticket_api.py > /opt/wazuh-ticket-lab/receiver.log 2>&1 & echo $! > /run/wazuh-ticket-api.pid'
```

### Assessment

The `curl` entry is incomplete in the supplied output. Its full URL, purpose, response, and result cannot be established.

The Python command indicates an attempt to launch a local mock ticket API in the background and record its process ID. The command is consistent with a controlled lab service, but the supplied history alone does not establish whether the process remained running or successfully served requests.

## 9. Evidence 6 – systemd and Wazuh Activity

### Relevant History Entries

The history contained repeated commands involving Wazuh services:

```bash
sudo systemctl restart wazuh-manager
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
sudo systemctl is-active wazuh-manager
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-dashboard --no-pager
```

The supplied history also showed commands executed during the recovery process:

```bash
mount -o remount,rw /
vi /etc/fstab
systemctl daemon-reload
reboot
```

### Audit Correlation

The supplied audit output contained `USER_CMD` records for the following commands:

```text
systemctl start wazuh-manager
systemctl start wazuh-indexer
systemctl start wazuh-dashboard
```

The records showed:

```text
exe="/usr/bin/sudo" ... res=success
```

The audit records provide corroboration that the commands were submitted through `sudo` and that the recorded command execution was successful at the audit level.

### Assessment

The history and audit excerpts support that Wazuh service start operations were issued.

They do not independently prove that the manager, indexer, or dashboard reached the `active` state. Service status output, process information, and application logs would be required to establish service health.

## 10. Evidence 7 – Lab 128 systemd Persistence Demonstration

The root history contained commands associated with an earlier controlled systemd persistence demonstration.

Relevant operations included:

```bash
sudo systemctl daemon-reload
sudo systemctl enable lab128-persistence-demo.service
sudo systemctl start lab128-persistence-demo.service
sudo systemctl status lab128-persistence-demo.service --no-pager
sudo systemctl restart lab128-persistence-demo.service
sudo systemctl disable --now lab128-persistence-demo.service
sudo rm -f /etc/systemd/system/lab128-persistence-demo.service
sudo systemctl reset-failed lab128-persistence-demo.service
```

The history also contained commands to inspect the service file, verify its syntax, review its journal entries, and inspect Wazuh alerts.

### Assessment

These entries are consistent with the creation, execution, modification, and cleanup of a controlled systemd persistence demonstration.

The recorded cleanup commands indicate an attempt to disable and remove the demonstration service. The history alone does not prove that every cleanup operation completed or that no related artifacts remained.

This is an example of why persistence-related commands must be interpreted in their original lab context.

## 11. Evidence 8 – Linux Audit Records

The supplied audit output included records of the following type:

```text
type=USER_CMD
```

The records contained command information, user identifiers, session identifiers, process IDs, terminal information, and command execution results.

Examples included:

- Wazuh manager start operation.
- Inspection of system files.
- Account-management activity.
- Filesystem and journal inspection.
- Recovery-related system commands.

### Assessment

Audit records can provide useful corroboration because they may preserve command activity independently of Bash history.

However, the available audit excerpt is incomplete. Some decoded command strings in the supplied extraction are malformed or truncated and should not be reconstructed by guessing.

The audit evidence should be treated as a separate source with its own timestamps and collection limitations.

## 12. Evidence 9 – `/home` Filesystem Recovery Context

The investigation took place after the `/home` XFS filesystem became inaccessible.

The system's `/etc/fstab` originally contained:

```text
/dev/mapper/rl-home     /home                   xfs     defaults,nofail 0 0
```

During recovery, the root filesystem was remounted read-write and the `/home` entry was commented out:

```text
#/dev/mapper/rl-home     /home                   xfs     defaults,nofail 0 0
```

The system was then able to boot without attempting the automatic `/home` mount.

### Assessment

This was a boot-recovery measure, not a filesystem repair.

The existing `/home` logical volume and its contents were not formatted or recreated. The filesystem remains outside the accessible mounted paths, and user-level evidence stored under `/home` was not available for this investigation.

The root user's history remained accessible because it was stored under `/root`.

## 13. Evidence Correlation Summary

| Evidence Source | What It Supports | What It Does Not Prove |
|---|---|---|
| Bash history | Commands recorded by the root shell | Successful completion of every command |
| History file metadata | File ownership, permissions, and timestamps | Authenticity or completeness of history contents |
| Account database | Presence of account entries | Successful creation of accessible home directories |
| Linux audit records | Selected command activity and execution context | Final state of services or filesystem changes |
| systemd commands | Service-management operations were recorded | Service health without status and application logs |
| `/etc/fstab` | `/home` mount was disabled during recovery | Recovery of the damaged filesystem |
| Windows Event ID 129 | A separate Windows Task Scheduler event | Any activity on the Rocky Linux host |

## 14. Confirmed Observations

- The investigation used the root account and Bash shell.
- `/root/.bash_history` was the configured history file.
- The history file existed and was owned by root with mode `0600`.
- Controlled Lab 129 directory, file, and permission commands appeared in history.
- Account-management commands and earlier lab activity appeared in history.
- Wazuh service start and restart commands appeared in history.
- Selected Wazuh service start commands were also present in the supplied audit records.
- The `/home` mount was disabled in `/etc/fstab` to restore system boot access.

## 15. Not Established

- Whether every command in shell history completed successfully.
- Whether all history timestamps accurately represent command execution times.
- Whether the `curl` command completed successfully or what its full destination was.
- Whether the mock ticket API process remained active.
- Whether all Wazuh services reached an active and healthy state.
- Whether the `lab129user` home directory was successfully created and accessible.
- Whether the `/home` filesystem data is intact or recoverable.
- Whether the shell history is complete and unmodified.

## 16. Investigation Assessment

The root Bash history provided useful visibility into administrative operations, controlled lab activity, and filesystem recovery actions.

Selected audit records strengthened the investigation by providing independent evidence of some command activity. However, differences in timestamp presentation, incomplete command output, and the unavailable `/home` filesystem limited the reconstruction.

The evidence does not establish malicious activity. The observed commands are consistent with the system administration, Wazuh operations, and controlled security labs documented in the available context.

## 17. Conclusion

Lab 129 demonstrated how Linux shell history can be used to reconstruct aspects of user and administrative activity.

The investigation also highlighted the importance of independent evidence sources. Bash history, file metadata, account information, audit records, and system logs each provide different levels of visibility.

A reliable DFIR conclusion must distinguish between a recorded command, a successful command execution, and a verified resulting system state.

**Final assessment: Shell history is a valuable investigative artifact, but it must be corroborated before drawing conclusions about user actions or system compromise.**
