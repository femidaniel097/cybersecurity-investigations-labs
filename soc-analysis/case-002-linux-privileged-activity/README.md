# SOC-002 — Linux Authentication and Privileged Activity Investigation

**Date:** 24 November 2026  
**Investigator:** Femi Daniel Somuyiwa  
**Environment:** Ubuntu 26.04.1 LTS — Oracle VirtualBox  
**Hostname:** dansom-VirtualBox  
**Category:** SOC Analysis and Digital Investigations  
**Status:** Practical investigation completed

## 1. Investigation Objective

Investigate Linux authentication records and privileged command activities using the systemd journal and traditional authentication logs.

The objectives were to:

- Identify available security evidence sources.
- Investigate historical administrative activities.
- Detect failed authentication attempts.
- Normalise timestamps.
- Correlate related security events.
- Reconstruct an authentication event timeline.
- Distinguish legitimate administrative activities from potentially malicious behaviour.

## 2. Tools and Commands

| Tool | Purpose |
|---|---|
| journalctl | Retrieve systemd journal records |
| grep | Filter and count matching log records |
| tail | Display recent file contents |
| ls | Inspect file metadata and permissions |
| file | Identify file content type |
| last | Investigate login-accounting records |
| command | Check executable availability |
| sudo | Execute authorised privileged commands |
| PAM | Authentication and session-management framework |

## 3. Evidence Source Discovery

### Command

```bash
journalctl --list-boots
```

### Findings

The journal contained records associated with 12 distinct boots, covering 17–24 November 2026.

- Current boot: index 0
- Previous boot: index -1
- Oldest available boot: index -11

The availability of records across multiple boots enabled historical investigation.

### Troubleshooting

The command initially opened a terminal pager, resulting in horizontally truncated output.

An alternative command for direct terminal output is:

```bash
journalctl --list-boots --no-pager
```

### Security significance

Historical journal retention enables investigators to reconstruct activities that occurred before a system restart.

However, the presence of historical records does not guarantee complete evidence retention.

## 4. Privileged Activity Investigation

### Command

```bash
sudo journalctl _COMM=sudo --since "24 hours ago" --no-pager
```

### Command explanation

- `_COMM=sudo`: Filter journal entries associated with the sudo process.
- `--since`: Specify the investigation time window.
- `--no-pager`: Display results directly.

### Findings

Seven sudo command records were identified in the initial 24-hour investigation.

All seven were initiated by the account `dansom`.

Four targeted the `student` account, while three targeted root.

### Reconstructed timeline

All timestamps below are in WAT.

| Date | Time | PID | Target user | Activity |
|---|---|---|---|---|
| Nov 23 | 22:17:21 | 5029 | student | Investigation file access test |
| Nov 23 | 22:53:49 | 5329 | root | whoami |
| Nov 23 | 22:53:49 | 5334 | root | id -u |
| Nov 23 | 22:59:22 | 5345 | student | Investigation file access retest |
| Nov 23 | 23:02:09 | 5373 | student | Sample file access test |
| Nov 23 | 23:02:52 | 5381 | student | Sample file access retest |
| Nov 24 | 20:36:01 | 4925 | root | Journal investigation |

These activities were consistent with the known Day 7 Linux identity and permissions practical.

### Important investigative observation

Two identical file-reading commands produced different outcomes during the previous practical.

The first successfully read the sample file, while the second returned Permission denied after permission hardening.

The sudo command records alone did not establish these different outcomes.

This demonstrates the importance of correlating multiple evidence sources.

## 5. Boot-Specific Evidence Filtering

### Commands

```bash
sudo journalctl -b -1 _COMM=sudo --no-pager | grep 'COMMAND='
```

```bash
sudo journalctl -b -1 _COMM=sudo --no-pager | grep -c 'COMMAND='
```

### Actual count

```text
6
```

### Findings

Six sudo command records were identified in the previous boot.

| Target account | Command records |
|---|---:|
| student | 4 |
| root | 2 |
| Total | 6 |

### Concepts learned

- Boot-specific journal filtering.
- Linux pipe operator.
- Text pattern matching.
- Counting matching security records.

## 6. Traditional Authentication Log Investigation

### Commands

```bash
ls -lh /var/log/auth.log
tail -20 /var/log/auth.log
```

### Actual file metadata

```text
-rw-r----- 1 syslog adm 118K Nov 24 20:36 /var/log/auth.log
```

### Interpretation

- Owner: syslog
- Group: adm
- Permission: 640
- Size: 118 KB

The authentication log was successfully accessed without sudo because the investigating account belonged to the adm group.

This demonstrated the practical relationship between Linux group membership and security evidence access.

## 7. Alternative Privilege Elevation Mechanism

The authentication log revealed activity involving pkexec.

### Evidence

```text
2026-11-24T20:31:22.118429+01:00
dansom-VirtualBox pkexec[3587]:
dansom: Executing command
[USER=root]
[TTY=unknown]
[CWD=/home/dansom]
[COMMAND=/usr/lib/update-notifier/package-system-locked]
```

### Interpretation

pkexec uses the Polkit authorisation framework to execute commands under another user identity.

The recorded activity appeared consistent with Ubuntu's update-notification functionality.

Its legitimacy was not independently verified.

### Security lesson

Monitoring sudo alone may not provide complete visibility into privileged activities.

Other mechanisms, including Polkit, should also be considered during security investigations.

## 8. Historical Authentication Failure Investigation

### Initial journal query

```bash
sudo journalctl --since "24 hours ago" --no-pager | grep -Ei 'authentication failure|failed password|incorrect password|invalid user|authentication error'
```

### Initial result

No matching output was returned.

This indicated that no retrieved journal records matched the specified patterns within the investigation window at that time.

It did not establish that no authentication failures had occurred.

### Traditional authentication log query

```bash
grep -Ei 'authentication failure|failed password|incorrect password|invalid user|authentication error' /var/log/auth.log | tail -20
```

### Error encountered

```text
grep: /var/log/auth.log: binary file matches
```

### Troubleshooting commands

```bash
file /var/log/auth.log
```

Actual output:

```text
/var/log/auth.log: ASCII text
```

The search was repeated using the -a option:

```bash
grep -aEi 'authentication failure|failed password|incorrect password|invalid user|authentication error' /var/log/auth.log | tail -20
```

The -a option instructed grep to process the input as text.

The exact cause of the original binary-file warning was not established.

### Findings

Six historical authentication failure records were discovered, all dated 17 November 2026.

| Time (WAT) | Authentication service | Event |
|---|---|---|
| 10:31:54 | gdm-password | Authentication failure |
| 10:32:05 | gdm-password | Authentication failure |
| 10:32:24 | gdm-password | Authentication failure |
| 10:32:41 | gdm-password | Authentication failure |
| 10:36:51 | sudo | Authentication failure |
| 11:25:33 | Polkit | Authentication failure |

### Timestamp normalisation

The original authentication log contained timestamps with both UTC (+00:00) and UTC+1 (+01:00) offsets.

All timestamps in the investigation timeline were normalised to West Africa Time.

### Historical event assessment

Four graphical authentication failures occurred within approximately 47 seconds.

The records were consistent with repeated unsuccessful graphical authentication attempts.

The attempted username was not established by the four GDM records.

The events may have been related to earlier VM authentication difficulties, but this explanation was not independently confirmed.

**Classification:** Historical authentication failures requiring contextual verification. Insufficient evidence of malicious activity.

## 9. Failed Login Database Investigation

### Commands

```bash
sudo lastb -n 10
```

Actual output:

```text
sudo: 'lastb': command not found
```

Further investigation:

```bash
command -v last lastb
```

Actual output:

```text
/usr/bin/last
```

```bash
ls -lh /var/log/btmp /var/log/wtmp
```

Actual output:

```text
-rw-rw---- 1 root utmp 1.5K Nov 17 10:32 /var/log/btmp
-rw-rw-r-- 1 root utmp 6.4K Nov 24 20:30 /var/log/wtmp
```

A subsequent sudo log also recorded execution of:

```bash
sudo last -f /var/log/btmp
```

However, the resulting output was not captured in this investigation report.

### Evidence collection limitation

The traditional failed login database existed, but its contents were not successfully validated for this report.

No conclusions were drawn from the unavailable database results.

## 10. Controlled Authentication Failure Simulation

### Objective

Generate a known authentication failure and investigate its corresponding digital evidence.

### Simulation procedure

The planned procedure involved invalidating cached sudo authentication and requesting execution of whoami with elevated privileges.

```bash
sudo -k
sudo whoami
```

The exercise involved an intentionally incorrect password followed by successful authentication.

### Evidence collection command

```bash
sudo journalctl _COMM=sudo --since "5 minutes ago" --no-pager
```

### Observed authentication failure

```text
Nov 24 21:40:52 dansom-VirtualBox sudo[5754]:
pam_unix(sudo:auth): authentication failure;
logname=dansom uid=1000 euid=0
tty=/dev/pts/0 ruser=dansom rhost= user=dansom
```

### Subsequent privileged session

```text
Nov 24 21:41:02 dansom-VirtualBox sudo[5754]:
pam_unix(sudo:session):
session opened for user root(uid=0)
by dansom(uid=1000)
```

### Command execution record

```text
Nov 24 21:41:02 dansom-VirtualBox sudo[5754]:
dansom : TTY=/dev/pts/0 ;
PWD=/home/dansom ;
USER=root ;
COMMAND=/usr/bin/whoami
```

### Session termination

```text
Nov 24 21:41:02 dansom-VirtualBox sudo[5754]:
pam_unix(sudo:session):
session closed for user root
```

## 11. Security Event Correlation

The authentication failure, subsequent privileged session, command record and session closure shared the same process ID: 5754.

### Reconstructed timeline

| Time | PID | Event |
|---|---|---|
| 21:40:06 | 5735 | Privileged whoami command |
| 21:40:33 | 5747 | Privileged whoami command |
| 21:40:52 | 5754 | Authentication failure |
| 21:41:02 | 5754 | Root session opened |
| 21:41:02 | 5754 | whoami command recorded |
| 21:41:02 | 5754 | Root session closed |
| 21:41:16 | 5762 | Evidence collection |

### Extracted evidence

| Attribute | Value |
|---|---|
| Host | dansom-VirtualBox |
| Initiating account | dansom |
| Initiating UID | 1000 |
| Target account | root |
| Target UID | 0 |
| Correlated PID | 5754 |
| Terminal | /dev/pts/0 |
| Recorded failures | 1 |
| Time between failure and privileged session | 10 seconds |
| Command | /usr/bin/whoami |

## 12. Detection Queries and Investigative Pivoting

### Broad detection query

```bash
sudo journalctl _COMM=sudo --since today --no-pager | grep -E 'authentication failure|COMMAND='
```

### Actual findings

- Total matching records: 11
- Sudo command records: 10
- Authentication failure records: 1
- Initiating accounts: 1
- Commands targeting root: 10

Seven command records related to evidence collection, while three involved privilege verification using whoami.

### Targeted investigation query

```bash
sudo journalctl _COMM=sudo _PID=5754 -b 0 --no-pager
```

### Actual output

```text
Nov 24 21:40:52 dansom-VirtualBox sudo[5754]: pam_unix(sudo:auth): authentication failure; logname=dansom uid=1000 euid=0 tty=/dev/pts/0 ruser=dansom rhost= user=dansom

Nov 24 21:41:02 dansom-VirtualBox sudo[5754]: pam_unix(sudo:session): session opened for user root(uid=0) by dansom(uid=1000)

Nov 24 21:41:02 dansom-VirtualBox sudo[5754]: dansom : TTY=/dev/pts/0 ; PWD=/home/dansom ; USER=root ; COMMAND=/usr/bin/whoami

Nov 24 21:41:02 dansom-VirtualBox sudo[5754]: pam_unix(sudo:session): session closed for user root
```

The query successfully isolated four related records.

### Security significance

Investigative pivoting enables analysts to narrow broad searches using specific evidence attributes, including process IDs, usernames, hostnames and timestamps.

Restricting the search to the current boot reduced the risk of correlating unrelated records involving reused process IDs across different boots.

## 13. Incident Assessment

**Case ID:** SOC-002-EVENT-001

**Detection:** Failed sudo authentication followed by successful privileged command execution.

**Affected host:** dansom-VirtualBox

**Account:** dansom

**Observed command:** /usr/bin/whoami

**Disposition:** Benign — authorised security testing.

### Assessment rationale

The activity originated from a known controlled authentication experiment.

The recorded authentication failure and subsequent privileged session were correlated through their process ID, timestamp and account identity.

No evidence of compromise was established by this experiment.

In an unknown operational environment, similar activity would require contextual investigation before classification.

## 14. Key Security Lessons

1. Linux journal records can support historical event reconstruction.
2. Authentication and authorisation are distinct security processes.
3. Successful sudo sessions do not necessarily establish successful command outcomes.
4. Privileged activities can occur through mechanisms other than sudo.
5. Regular expressions support efficient security evidence filtering.
6. Timestamp normalisation is essential for accurate timeline reconstruction.
7. Process IDs can support security event correlation.
8. Failed authentication followed by successful access is not automatically malicious.
9. Evidence collection limitations must be documented.
10. Investigative activities can themselves generate additional security logs.
11. Absence of matching records does not prove absence of malicious activity.

## 15. Skills Developed

- Linux security log analysis
- Authentication monitoring
- Privileged activity investigation
- Bash command pipelines
- Regular expression filtering
- Evidence collection and validation
- Timestamp normalisation
- Security event correlation
- Investigative pivoting
- Incident timeline reconstruction
- Incident classification
- Technical security reporting

## 16. Conclusion

This practical investigation demonstrated the use of Linux authentication logs and the systemd journal for SOC analysis.

Historical authentication failures were identified, previous administrative activities were reconstructed, and a controlled authentication failure was successfully correlated with subsequent privileged command execution.

The exercise demonstrated how raw system logs can be transformed into meaningful security intelligence through systematic evidence collection, filtering, correlation and contextual analysis.

**Practical investigation status: Completed.**
