# PART 1 — How Linux Logging Actually Works (Core Knowledge)

Linux logging is NOT one thing.  
It is a **pipeline**:

`Application → syscalls → systemd-journald → rsyslog/syslog-ng → log files → rotation → archive → SIEM`

## 1. journald (The Heart of Modern Linux Logs)

Location:   `/var/log/journal/`
View logs:   `journalctl`
Real-time:  `journalctl -f`
Boot logs:  `journalctl -b`
Last 1 hour:  `journalctl --since "1 hour ago"`
By service:  `journalctl -u ssh journalctl -u nginx journalctl -u docker`
By PID:  `journalctl _PID=1234`
By user:  `journalctl _UID=1000`
### Why journald is powerful

Unlike plain files, journald logs contain metadata:
- PID
- user
- command
- cgroup
- container id
- boot id
- SELinux context

This makes hiding actions VERY hard.
## 2. Classic Log Files (rsyslog)

Location:  `/var/log/`
Important ones:

|File|Meaning|
|---|---|
|auth.log / secure|logins, sudo, ssh|
|syslog / messages|system events|
|kern.log|kernel activity|
|dmesg|hardware|
|boot.log|startup|
|apt/history.log|installed packages|

Example:  `sudo cat /var/log/auth.log`
# PART 2 — The Skill of a Real Log Expert (What companies want)

## Skill 1: Investigate a Login Attack

Find failed SSH attempts:   `grep "Failed password" /var/log/auth.log`
Count attackers:   `grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr`
Find successful logins:   `grep "Accepted password" /var/log/auth.log`
## Skill 2: Timeline Reconstruction (VERY IMPORTANT)

Security engineers reconstruct incidents like this:  `journalctl --since "2026-02-15 10:00" --until "2026-02-15 11:00"`
Find what a user executed:  `journalctl _UID=1000 | grep sudo`
Find suspicious commands:  `grep "COMMAND=" /var/log/auth.log`

## Skill 3: Detect Log Tampering (This makes you pro)

A pro doesn’t delete logs —  
A pro detects when someone tried to.

Signs of tampering:

### 1) Time gaps

`journalctl --list-boots`

### 2) Journal corruption

`journalctl --verify`

### 3) Missing rotation sequence

`auth.log auth.log.1 auth.log.2.gz auth.log.3.gz`

If numbers skip → edited.
### 4) Compare kernel vs auth timestamps

Attackers forget kernel logs.
# PART 3 — Log Rotation (Legitimate Deletion)

Companies don’t keep logs forever — they rotate.
Config:  `/etc/logrotate.conf /etc/logrotate.d/`
Test rotation:   `logrotate -d /etc/logrotate.conf`
Force rotate:  `sudo logrotate -f /etc/logrotate.conf`
Retention example:   `rotate 7 daily compress`
Meaning:  keep 7 days only → older logs automatically removed
This is the **correct professional way** to remove logs.
# PART 4 — journald Retention (Real world cleanup)

Admins control storage, not manually edit logs.
Config:  `/etc/systemd/journald.conf`
Examples:   `SystemMaxUse=500M MaxRetentionSec=7day`
Apply:   `sudo systemctl restart systemd-journald`
# PART 5 — How Professionals Practice (IMPORTANT)

Create your own lab:  `sudo apt install openssh-server`
Generate fake attacks:  `for i in {1..50}; do ssh fake@localhost; done`