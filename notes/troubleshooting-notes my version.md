# Troubleshooting Notes
## 1. `/home` Filesystem Failed to Mount

### Problem

The Rocky Linux system experienced XFS metadata corruption on the `/home` logical volume.

The filesystem produced errors indicating internal XFS corruption and could not be mounted normally.

The mount failure also affected access to user home directories.

### Impact

- Normal access to `/home` was unavailable.
- The account `abhinavmathur` referenced `/home/abhinavmathur` as its home directory.
- The `lab129user` account referenced `/home/lab129user`.
- User-level Bash history files under `/home` could not be collected through their normal paths.
- The root user's history remained accessible under `/root`.

### Recovery Action

The existing `/home` entry was temporarily commented out in `/etc/fstab`:

```text
#/dev/mapper/rl-home     /home                   xfs     defaults,nofail 0 0
```

The system was then able to boot without automatically attempting to mount the damaged filesystem.

### Assessment

This action bypassed the mount failure; it did not repair XFS or recover the data.

The original filesystem and logical volume were not formatted or recreated.

The `/home` filesystem must remain outside the normal collection scope until its recovery is handled separately.

## 2. `/etc/fstab` Could Not Be Saved

### Problem

An attempt to edit `/etc/fstab` using `vi` returned an error indicating that the file could not be opened for writing.

### Validation

The root filesystem mount state was examined:

```bash
mount | grep ' on / '
```

The output showed:

```text
/dev/mapper/rl-root on / type xfs (ro,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
```

The root filesystem was mounted read-only.

### Resolution

The root filesystem was remounted read-write:

```bash
mount -o remount,rw /
```

After the remount completed without an error, `/etc/fstab` was edited successfully.

### Assessment

The inability to save the configuration file was caused by the read-only mount state of `/`.

The successful remount allowed the recovery configuration change to be saved. It did not repair the separate `/home` filesystem.

## 3. Graphical Login Loop

### Problem

After bypassing the initial emergency boot failure, the system displayed the graphical login screen. Attempting to log in returned to the same screen.

### Investigation Context

The affected user account referenced a home directory under `/home`, which was unavailable because of the XFS mount problem.

A text-mode boot attempt was also made using a temporary GRUB kernel parameter:

```text
systemd.unit=multi-user.target
```

The system did not provide a usable text login during that attempt.

### Recovery Approach

The GRUB menu was accessed, and the installed system was booted into an emergency target for recovery.

The `/home` mount entry was then disabled in `/etc/fstab`.

### Assessment

The inaccessible home directory was a relevant recovery constraint, but the exact cause of the graphical login loop was not independently established.

The successful boot after disabling the `/home` mount allowed work to continue without repairing or formatting the filesystem.

## 4. Distinguishing the initramfs Shell from the Installed System

### Problem

A boot attempt using:

```text
rd.break
```

entered the early boot recovery environment.

The prompt appeared as:

```text
switch_root:/#
```

A mount inspection showed:

```text
TARGET SOURCE FSTYPE OPTIONS
/      none   rootfs rw
```

### Explanation

This was the initramfs environment, not the normal installed Rocky Linux root filesystem.

The `rootfs` mount output therefore did not represent the mounted installed root logical volume.

### Resolution

The boot process was allowed to continue into the installed system's emergency environment.

The installed-system prompt was:

```text
[root@localhost ~]#
```

### Assessment

The distinction matters because recovery commands must be executed against the intended installed system and not mistakenly interpreted from the temporary initramfs environment.

## 5. Bash History Did Not Initially Show All Current Commands

### Problem

The persistent history file did not immediately contain all commands entered during the current shell session.

### Explanation

Bash commonly writes the current session's history to the history file when the shell exits. The timing depends on shell configuration and history settings.

### Command Used

```bash
history -a
```

The command appends the current session's history to the history file.

The file was then examined using:

```bash
cat ~/.bash_history
```

### Assessment

A forensic collection should distinguish between the current in-memory history list and the persistent history file.

Collecting only the file may omit commands that have not yet been written.

## 6. Repeated History Timestamps

### Problem

Many history entries displayed the same timestamp:

```text
2026-11-06 10:57:39
```

Other entries displayed later times.

### Explanation

`HISTTIMEFORMAT` controls timestamp display, but it does not create reliable historical timestamps for commands that were not originally recorded with timestamp information.

Repeated timestamps should not be interpreted as proof of simultaneous execution.

### Command Used

```bash
HISTTIMEFORMAT='%F %T '
history
```

### Assessment

History timestamps must be evaluated alongside raw history data, shell configuration, system time, and independent audit records.

Do not construct a precise timeline solely from repeated or potentially incomplete history timestamps.

## 7. Root History Was Accessible but User History Was Not

### Problem

The account database showed:

```text
abhinavmathur:x:1000:1000:abhinav Mathur:/home/abhinavmathur:/bin/bash
lab129user:x:1001:1001::/home/lab129user:/bin/bash
```

However, `/home` was not mounted.

### Impact

The normal paths for these users' Bash history files were unavailable.

### Available Evidence

The root user's history remained accessible:

```text
/root/.bash_history
```

### Assessment

The presence of an account entry does not prove that its home directory is accessible.

The user-level history files were not treated as collected evidence because the filesystem containing them was unavailable.

## 8. Interpreting `systemctl start` Commands

### Problem

The shell history contained multiple commands such as:

```bash
sudo systemctl start wazuh-manager
sudo systemctl start wazuh-indexer
sudo systemctl start wazuh-dashboard
```

The audit output also contained corresponding `USER_CMD` records with `res=success`.

### Explanation

An audit result of `res=success` supports successful execution of the recorded command through the audited interface. It does not independently establish that the service started successfully or remained healthy.

### Required Correlation

Service state should be established from service status output, for example:

```bash
systemctl is-active wazuh-manager
systemctl is-active wazuh-indexer
systemctl is-active wazuh-dashboard
```

Service logs and application health information provide additional evidence.

### Assessment

The available evidence confirms that service start commands were issued. It does not, by itself, establish that all three services reached a healthy active state.

## 9. Interpreting the Lab 128 Persistence Commands

### Problem

The root history contained commands to create, enable, start, modify, disable, and remove:

```text
lab128-persistence-demo.service
```

### Explanation

Systemd service creation and modification can be associated with persistence techniques. However, the same actions can also occur during authorized security testing and controlled labs.

The supplied history included service verification, journal review, Wazuh alert searches, and cleanup commands.

### Assessment

The entries are consistent with the previously conducted Lab 128 demonstration.

They should be interpreted in that context rather than automatically classified as malicious persistence.

The history alone does not prove that every cleanup action succeeded or that no residual artifacts remained.

## 10. Truncated `curl` Command

### Problem

The supplied history output contained:

```bash
curl -sS http://127.0.0
```

The displayed command was incomplete.

### Impact

The full destination, request path, purpose, response, and result could not be established.

### Assessment

The entry is an investigative lead only.

Do not reconstruct the missing URL or claim that a remote connection or successful download occurred based on the truncated output.

## 11. Separate Windows Event ID 129 Output

### Problem

A separate screenshot showed Windows Task Scheduler Operational Event ID 129.

### Explanation

The event referred to the Windows task:

```text
\Microsoft\Windows\Flighting\FeatureConfig\ReconcileFeatures
```

It was generated on a Windows host and was not part of the Rocky Linux evidence collection.

### Assessment

The event was excluded from the Linux investigation timeline.

It requires separate Windows endpoint context and should not be correlated with Linux shell activity without an independently established relationship.

