# SOC-003 — Security Event Triage and Incident Classification

## 1. Investigation Overview

**Learning Day:** Day 11  
**Scheduled Date:** 27 November 2026  
**Actual Investigation Date:** 28 November 2026  
**Category:** SOC Analysis and Digital Investigations  
**Environment:** Ubuntu Linux on Oracle VirtualBox  
**Status:** Practical completed

This investigation focuses on authentication event triage, historical log recovery, evidence correlation, incident classification and the identification of detection coverage gaps in a previously developed Python security automation tool.

## 2. Investigation Objectives

- Understand the differences between security events, alerts and incidents.
- Apply SOC triage methodology.
- Investigate Linux authentication failures.
- Correlate evidence across different logging interfaces.
- Investigate Linux log rotation.
- Recover historical authentication records.
- Reconstruct a security event timeline.
- Assess severity, confidence and investigative priority.
- Determine whether an event warrants escalation.

## 3. Investigation Workspace

The investigation was conducted from:

```bash
~/security-labs/day11-soc-triage
```

## 4. Investigation Timestamp

Command:

```bash
date -Is
```

Actual output:

```text
2026-11-28T09:37:27+01:00
```

This established the investigation date, time and timezone.

Accurate time references are essential for defining investigation windows and correlating security evidence.

## 5. Automated Authentication Log Analysis

I reused the Python authentication log analyser developed during Day 9.

```bash
python3 ~/security-labs/day9-python-log-analysis/log_analyser.py
```

The analyser returned one matching authentication failure record.

Sanitised output:

```text
====== DETECTED SECURITY EVENTS ======

Event: 1
Timestamp: 2026-11-26T17:34:19.924143+01:00
Service: gdm
Username: labuser

====== SECURITY SUMMARY ======
Total authentication failures: 1

====== SERVICE BREAKDOWN ======
gdm: 1
sudo: 0
polkit: 0
other: 0

Analysis completed successfully.
```

The local username has been replaced with `labuser` throughout this documentation for privacy.

## 6. Initial System Journal Investigation

Command:

```bash
sudo journalctl --since "24 hours ago" --no-pager | grep -Ei 'authentication failure|failed password|invalid user'
```

No matching records were returned.

The authentication failure identified by the Python analyser occurred approximately 40 hours before the investigation and therefore fell outside the selected 24-hour window.

This demonstrated the importance of selecting an appropriate investigation time range.

An empty search result does not necessarily establish the absence of security events.

## 7. Log Rotation Investigation

During Day 9, the Python analyser detected seven authentication failure records. During this investigation, it detected only one.

I investigated whether log rotation could explain the difference.

Command:

```bash
ls -lh /var/log/auth.log*
```

Actual output:

```text
-rw-r----- 1 syslog adm  12K Nov 28 09:38 /var/log/auth.log
-rw-r----- 1 syslog adm 129K Nov 25 22:30 /var/log/auth.log.1
```

The system contained both an active authentication log and a historical rotated log.

| File | Size | Purpose |
|---|---|---|
| auth.log | 12 KB | Current authentication records |
| auth.log.1 | 129 KB | Historical authentication records |

Both files had permission mode 640 and were owned by the syslog user and adm group.

Log rotation manages storage consumption while retaining historical records according to the configured retention policy.

## 8. Expanded Investigation Window

I expanded the system journal investigation window from 24 hours to 72 hours.

```bash
sudo journalctl --since "72 hours ago" --no-pager | grep -Ei 'authentication failure|failed password|invalid user'
```

Sanitised output:

```text
Nov 26 17:34:19 lab-host gdm-password][4483]:
pam_unix(gdm-password:auth): authentication failure;
logname=labuser uid=0 euid=0 tty=/dev/tty1
ruser= rhost= user=labuser
```

The hostname and username have been sanitised, and the line has been wrapped for readability.

The system journal contained a matching record corresponding to the event identified by the Python analyser.

## 9. Authentication Event Field Analysis

| Field | Value | Interpretation |
|---|---|---|
| Timestamp | 26 Nov, 17:34:19 | Event occurrence time |
| Service | gdm-password | Graphical authentication service |
| PID | 4483 | Process identifier |
| PAM module | pam_unix | Unix authentication module |
| uid | 0 | Process user identity |
| euid | 0 | Effective process user identity |
| tty | /dev/tty1 | Local virtual terminal |
| ruser | Empty | Remote username not recorded |
| rhost | Empty | Remote hostname not recorded |
| user | labuser | Account undergoing authentication |

The UID and effective UID values of zero describe the authentication process's identity rather than establishing that the attempted login account was root.

The available evidence was consistent with a local graphical authentication failure.

## 10. Cross-Log Evidence Correlation

I compared the event identified by the Python analyser with the system journal record.

| Attribute | Python analyser | System journal |
|---|---|---|
| Date | 26 November | 26 November |
| Time | 17:34:19 | 17:34:19 |
| Service | GDM | gdm-password |
| Account | labuser | labuser |
| Event | Authentication failure | Authentication failure |

The matching attributes established that both logging interfaces recorded the same underlying authentication event.

These records were not counted as two separate authentication failures.

## 11. Historical Evidence Recovery

I searched the rotated authentication log.

```bash
grep -ai "authentication failure" /var/log/auth.log.1
```

I independently counted the matching records.

```bash
grep -aic "authentication failure" /var/log/auth.log.1
```

Actual output:

```text
7
```

All seven authentication failure records previously discovered during Day 9 were recovered.

The historical records consisted of:

- Four GDM authentication failures.
- Two sudo authentication failures.
- One Polkit authentication failure.

This confirmed that the historical records remained available in the rotated authentication log.

## 12. Combined Authentication Findings

| Authentication service | Historical log | Current log | Total |
|---|---:|---:|---:|
| GDM | 4 | 1 | 5 |
| Sudo | 2 | 0 | 2 |
| Polkit | 1 | 0 | 1 |
| Total | 7 | 1 | 8 |

The Python analyser detected only one of the eight matching records available across the two log files.

This exposed a detection coverage limitation in the existing automation tool.

## 13. Temporal Evidence Correlation

I examined a ten-minute window surrounding the recent authentication failure.

```bash
sudo journalctl --since "2026-11-26 17:30:00" --until "2026-11-26 17:40:00" --no-pager | grep -Ei 'gdm|pam_unix'
```

The investigation returned four relevant records.

| Time (WAT) | PID | Activity |
|---|---|---|
| 17:30:01 | 4426 | CRON root session opened |
| 17:30:01 | 4426 | CRON root session closed |
| 17:34:19 | 4483 | GDM authentication failure |
| 17:34:30 | 4517 | GNOME login keyring unlocked |

The login keyring was unlocked 11 seconds after the authentication failure.

This supported the hypothesis that an unsuccessful authentication attempt was followed by successful authentication.

The CRON activity was recorded as contextual information. No evidence established a connection between the scheduled process and the authentication failure.

## 14. Investigative Pivoting

I used the process identifier associated with the keyring activity to retrieve additional evidence.

```bash
sudo journalctl _PID=4517 --since "2026-11-26 17:34:00" --until "2026-11-26 17:36:00" --no-pager
```

Sanitised output:

```text
Nov 26 17:34:30 lab-host gdm-password][4517]:
gkr-pam: unlocked login keyring
```

The process-specific investigation confirmed the keyring unlock message.

No additional records establishing the identity of the successfully authenticated user were returned.

## 15. Contextual Evidence

I recalled entering an incorrect password while attempting to authenticate to the Ubuntu virtual machine.

This recollection provided contextual support for the benign authentication error hypothesis.

User recollection was treated separately from technical log evidence.

## 16. Investigative Hypotheses

| Hypothesis | Assessment |
|---|---|
| Incorrect password entered by legitimate user | Strongly supported |
| Subsequent successful authentication | Supported by keyring activity |
| Remote authentication attempt | Not supported by available evidence |
| Brute-force attack | Not supported by observed activity |
| Unauthorised system access | Not established |
| Authentication configuration problem | No supporting evidence identified |

## 17. Final SOC Triage Assessment

| Attribute | Assessment |
|---|---|
| Event type | Local authentication failure |
| Detection validity | True detection |
| Activity classification | Benign |
| Severity | Low |
| Investigation priority | Routine |
| Confidence | High |
| Evidence of compromise | None identified |
| Escalation required | No |
| Disposition | Close and document |

The technical evidence and user recollection strongly supported the conclusion that the event resulted from a legitimate user's authentication error.

The available evidence did not establish a security incident.

## 18. True Positive and False Positive Distinction

The Python analyser correctly detected an authentication failure.

Therefore, it produced a true positive relative to its intended detection objective.

However, if a security detection rule had classified the same benign activity as malicious, that classification would represent a false positive.

Detection accuracy must be evaluated against the specific objective of the detection rule.

## 19. Security Automation Coverage Gap

The existing Python analyser reads only:

```python
log_file = "/var/log/auth.log"
```

Consequently, historical records stored in rotated authentication logs were excluded from its analysis.

| Metric | Result |
|---|---:|
| Matching records across both files | 8 |
| Records detected by Python | 1 |
| Historical records excluded | 7 |
| Coverage of observed matching records | 12.5% |

This percentage describes the observed records in the two available files. It does not represent the tool's overall security detection accuracy.

Recommended future improvements include:

- Multiple-file processing.
- Rotated and compressed log support.
- Configurable log paths.
- Time-based filtering.
- Duplicate event detection.
- Additional authentication detection patterns.
- Structured report generation.

## 20. Investigative Limitations

- The investigation was limited to the available authentication logs and system journal.
- The keyring unlock message did not independently establish the authenticated user's identity.
- User recollection provided contextual rather than independently verified technical evidence.
- The exact log rotation schedule and configuration were not examined.
- The investigation did not include network traffic or endpoint forensic analysis.
- No comprehensive assessment of system compromise was performed.
- The Python analyser remains limited to a single log file and selected detection phrase.

## 21. Skills Developed

- SOC event triage.
- Authentication event classification.
- Linux log rotation analysis.
- Historical evidence recovery.
- Cross-log correlation.
- Temporal correlation.
- Process identifier investigation.
- Hypothesis-driven investigation.
- Severity and priority assessment.
- Detection coverage analysis.
- Security automation evaluation.
- Evidence-based incident reporting.

## 22. Personal Reflection

This practical helped me understand that detecting a security event is only the beginning of an investigation.

I initially observed a difference between the authentication records detected during two separate practical sessions. By investigating log rotation, I recovered the historical records and identified a limitation in my Python automation tool.

I also correlated authentication records, reconstructed an event timeline and used contextual information to assess whether the observed activity represented a security incident.

The investigation reinforced the importance of selecting appropriate time windows, validating evidence, avoiding unsupported conclusions and distinguishing genuine security events from malicious activity.

It also demonstrated how practical SOC investigations can reveal opportunities to improve security automation tools.
