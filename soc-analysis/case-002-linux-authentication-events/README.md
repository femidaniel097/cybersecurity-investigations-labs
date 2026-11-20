# Case SOC-001 — Linux Authentication Event Analysis

**Date:** 20 November 2026  
**System:** dansom-VirtualBox  
**Operating System:** Ubuntu Linux  
**Investigating Account:** student  
**Investigation Type:** Host Authentication Log Analysis

## 1. Objective

To examine Linux authentication and privilege-related events, identify potentially suspicious activity and practise basic SOC alert triage.

## 2. Tools Used

- Ubuntu Terminal
- journalctl
- grep
- sudo

## 3. Investigation Commands

```bash
sudo journalctl --since "24 hours ago" --no-pager | grep -Ei "authentication failure|failed password|session opened|sudo"

sudo journalctl --since "24 hours ago" --no-pager | grep -Eic "authentication failure|failed password|invalid user|incorrect password"
```

## 4. Findings

### Authentication Events

The system journal contained entries relating to:

- CRON sessions running under the root account.
- Sudo commands executed by the student account.
- PAM session opening and closing events.
- SSSD sudo socket dependency errors.

### Failed Authentication

The targeted search returned **0 matching log lines** within the previous 24 hours.

This result does not exclude authentication failures recorded under other message patterns or outside the available journal.

## 5. Security Assessment

The observed CRON and sudo activities were consistent with normal Linux system administration.

The SSSD messages indicated service dependency issues rather than confirmed malicious activity.

No confirmed security incident was identified from the reviewed evidence.

## 6. Simulated Alert Triage

A scenario involving four failed SSH logins followed by one successful login was classified as suspicious activity requiring further investigation.

Recommended checks include:

- Verify whether the login was authorised.
- Investigate the source IP address.
- Review subsequent privileged commands.
- Examine related authentication events.

## 7. Conclusion

The investigation demonstrated basic Linux log analysis, event classification and SOC alert triage.

**Final Assessment:** No confirmed malicious activity in the reviewed logs.

**Case Status:** Closed for training purposes, with limited evidence scope.
