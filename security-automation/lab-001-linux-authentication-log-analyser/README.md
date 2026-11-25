# AUTO-001 — Linux Authentication Log Analyser

## 1. Project Overview

**Date:** 25 November 2026  
**Learning Day:** Day 9  
**Category:** Cybersecurity Automation and SOC Analysis  
**Status:** Practical completed  
**Environment:** Ubuntu Linux running on Oracle VirtualBox  
**Programming Language:** Python 3  
**Evidence Source:** `/var/log/auth.log`

This project focuses on developing a Python-based security automation tool for detecting, categorising, counting and extracting authentication failure records from Linux system logs.

The project builds upon my previous manual Linux authentication and privileged activity investigations by automating repetitive security analysis tasks.

## 2. Project Objectives

- Read and process Linux authentication logs using Python.
- Detect authentication failure records.
- Automatically count matching security events.
- Categorise events by authentication service.
- Extract usernames and timestamps.
- Handle missing information without making unsupported assumptions.
- Implement basic exception handling.
- Independently validate automated findings.
- Generate a structured security investigation report.

## 3. Environment Preparation

I prepared a dedicated working directory for the project.

```bash
mkdir -p ~/security-labs/day9-python-log-analysis
cd ~/security-labs/day9-python-log-analysis
```

I inspected the authentication log and verified that it was readable.

```bash
ls -lh /var/log/auth.log
test -r /var/log/auth.log && echo "Log file is readable"
```

Observed output:

```text
-rw-r----- 1 syslog adm 127K Nov 25 16:51 /var/log/auth.log
Log file is readable
```

The authentication log was accessible without elevated privileges. Membership of the `adm` group provided the necessary read access.

This allowed the investigation to follow the principle of least privilege.

## 4. Reading Linux Authentication Logs

I created the initial Python script.

```bash
nano log_analyser.py
```

Initial implementation:

```python
log_file = "/var/log/auth.log"

with open(log_file, "r", encoding="utf-8", errors="replace") as file:
    for line in file:
        print(line.strip())
```

I executed the script and inspected the last ten records.

```bash
python3 log_analyser.py | tail -10
```

The output contained records associated with:

- Polkit privileged operations.
- CRON session activity.
- GDM login keyring activity.

This exercise demonstrated Python file handling, context managers, iteration and string manipulation.

Processing the file line by line also avoids loading the entire log file into memory at once.

## 5. Authentication Failure Detection

I introduced conditional filtering to identify records containing the phrase `authentication failure`.

```python
if "authentication failure" in line.lower():
    print(line.strip())
```

The `lower()` method makes the phrase comparison case-insensitive.

Execution:

```bash
python3 log_analyser.py
```

The script identified seven matching authentication failure records.

## 6. Automated Event Counting

I introduced a counter to automatically calculate the total number of matching records.

```python
failure_count = 0
```

For each matching record:

```python
failure_count += 1
```

The resulting output was:

```text
====== SECURITY SUMMARY ======
Total authentication failures: 7
```

This eliminated the need to manually count the detected records.

## 7. Security Event Categorisation

I introduced a Python dictionary to aggregate authentication failures by service.

```python
services = {
    "gdm": 0,
    "sudo": 0,
    "polkit": 0,
    "other": 0
}
```

Conditional statements were used to classify matching records according to their PAM authentication service.

Actual output:

```text
====== SECURITY SUMMARY ======
Total authentication failures: 7

====== SERVICE BREAKDOWN ======
gdm: 4
sudo: 2
polkit: 1
other: 0
```

This demonstrates a basic form of security event aggregation, a concept used in Security Information and Event Management (SIEM) systems.

## 8. Regular Expression Practice

I created a separate practice script to understand how regular expressions can extract usernames from authentication logs.

```bash
nano regex_practice.py
```

Example implementation using a sanitised username:

```python
import re

line = "logname=labuser uid=1000 ruser=labuser rhost= user=labuser"

match = re.search(r"\buser=(\S+)", line)

if match:
    username = match.group(1)
    print(f"Username: {username}")
else:
    print("Username not recorded")
```

Execution:

```bash
python3 regex_practice.py
```

The test successfully extracted the username from the `user=` field.

The word boundary prevented the expression from incorrectly matching the `ruser=` field in the tested records.

## 9. Independent Evidence Validation

To verify the Python counter, I independently counted matching records using the Linux grep utility.

```bash
grep -aic "authentication failure" /var/log/auth.log
```

Actual output:

```text
7
```

| Analysis Method | Result |
|---|---:|
| Python log analyser | 7 |
| Linux grep utility | 7 |

Both methods returned identical results.

This validated the counting logic for the selected detection phrase, although it does not establish complete authentication failure detection coverage.

## 10. Timestamp and Username Extraction

I integrated timestamp extraction using Python string manipulation.

```python
timestamp = line.split()[0]
```

Username extraction was implemented using regular expressions.

```python
match = re.search(r"\buser=(\S+)", line)

if match:
    username = match.group(1)
else:
    username = "Not recorded"
```

The analyser successfully extracted seven timestamps and three recorded usernames.

Four GDM authentication failure records did not contain the required username field.

Rather than assigning an unsupported identity, the analyser reported the username as `Not recorded`.

## 11. Exception Handling

I created a separate Python script to investigate how file access errors can be handled.

```bash
nano exception_practice.py
```

The practice script used the following exception handlers:

```python
except FileNotFoundError:
    print(f"ERROR: File not found: {log_file}")

except PermissionError:
    print(f"ERROR: Permission denied: {log_file}")
```

### Test 1 — Nonexistent File

Execution:

```bash
python3 exception_practice.py
```

Actual output:

```text
ERROR: File not found: /var/log/nonexistent.log
Program finished.
```

### Test 2 — Restricted File Access

I changed the target path to `/etc/shadow` and executed the practice script without sudo.

Actual output:

```text
ERROR: Permission denied: /etc/shadow
Program finished.
```

The script correctly handled both runtime exceptions.

## 12. Troubleshooting Experience

While modifying the exception handling script, I accidentally introduced an additional quotation mark.

Incorrect code:

```python
log_file = "/etc/shadow""
```

Python returned:

```text
SyntaxError: unterminated string literal (detected at line 1)
```

I identified and corrected the additional quotation mark.

Corrected code:

```python
log_file = "/etc/shadow"
```

The script subsequently executed successfully.

This demonstrated the distinction between syntax errors detected before execution and runtime exceptions encountered during execution.

## 13. Final Python Implementation

```python
import re
import sys

log_file = "/var/log/auth.log"

failure_count = 0

services = {
    "gdm": 0,
    "sudo": 0,
    "polkit": 0,
    "other": 0
}

print("====== DETECTED SECURITY EVENTS ======")

try:
    with open(log_file, "r", encoding="utf-8", errors="replace") as file:
        for line in file:
            if "authentication failure" in line.lower():
                failure_count += 1

                # Identify authentication service
                if "pam_unix(gdm-password:auth)" in line:
                    service = "gdm"

                elif "pam_unix(sudo:auth)" in line:
                    service = "sudo"

                elif "pam_unix(polkit-1:auth)" in line:
                    service = "polkit"

                else:
                    service = "other"

                services[service] += 1

                # Extract timestamp
                timestamp = line.split()[0]

                # Extract username
                match = re.search(r"\buser=(\S+)", line)

                if match:
                    username = match.group(1)
                else:
                    username = "Not recorded"

                # Display extracted evidence
                print(f"\nEvent: {failure_count}")
                print(f"Timestamp: {timestamp}")
                print(f"Service: {service}")
                print(f"Username: {username}")

except FileNotFoundError:
    print(f"ERROR: File not found: {log_file}", file=sys.stderr)
    sys.exit(1)

except PermissionError:
    print(f"ERROR: Permission denied: {log_file}", file=sys.stderr)
    sys.exit(1)

except OSError as error:
    print(f"ERROR: {error}", file=sys.stderr)
    sys.exit(1)

print("\n====== SECURITY SUMMARY ======")
print(f"Total authentication failures: {failure_count}")

print("\n====== SERVICE BREAKDOWN ======")

for service, count in services.items():
    print(f"{service}: {count}")

print("\nAnalysis completed successfully.")
```

## 14. Investigation Findings

The following table summarises the seven detected records. The local username has been replaced with `labuser` for privacy, and timestamps have been manually normalised to West Africa Time.

| Event | Date | Time (WAT) | Service | Username |
|---|---|---|---|---|
| 1 | 17 Nov 2026 | 10:31:54 | GDM | Not recorded |
| 2 | 17 Nov 2026 | 10:32:05 | GDM | Not recorded |
| 3 | 17 Nov 2026 | 10:32:24 | GDM | Not recorded |
| 4 | 17 Nov 2026 | 10:32:41 | GDM | Not recorded |
| 5 | 17 Nov 2026 | 10:36:51 | Sudo | labuser |
| 6 | 17 Nov 2026 | 11:25:33 | Polkit | labuser |
| 7 | 24 Nov 2026 | 21:40:52 | Sudo | labuser |

### Security Interpretation

Four GDM authentication failure records occurred within approximately 48 seconds.

Two failure records were associated with sudo authentication, while one was associated with Polkit authentication.

The sudo failure recorded on 24 November was associated with a controlled authentication simulation conducted during my previous SOC investigation lab.

The remaining records require additional contextual investigation before attributing them to specific causes.

The available evidence does not independently establish a brute-force attack, unauthorised access or system compromise.

## 15. Exit Status Validation

I executed the final analyser and checked its exit status.

```bash
python3 log_analyser.py
echo $?
```

Actual output:

```text
====== SECURITY SUMMARY ======
Total authentication failures: 7

====== SERVICE BREAKDOWN ======
gdm: 4
sudo: 2
polkit: 1
other: 0

Analysis completed successfully.
0
```

The exit status of zero confirmed successful program execution.

Exit statuses are particularly useful when integrating security scripts into shell automation, monitoring systems and CI/CD pipelines.

## 16. Automated Report Generation

I redirected the standard output of the analyser into a text file.

```bash
python3 log_analyser.py > security_report.txt
```

I verified the generated report.

```bash
ls -lh security_report.txt
```

The report was successfully created with a size of 822 bytes.

I inspected the project directory.

```bash
ls -lh
```

Observed project files:

| File | Size |
|---|---:|
| exception_practice.py | 275 B |
| log_analyser.py | 2.0 KB |
| regex_practice.py | 257 B |
| security_report.txt | 822 B |

This demonstrated the ability to automatically generate investigation reports without manually copying terminal output.

## 17. Security Relevance

This project demonstrates foundational concepts applicable to:

- SOC analysis and security monitoring.
- Automated log analysis.
- Digital investigations.
- Authentication event monitoring.
- Security event aggregation.
- Evidence extraction and validation.
- Linux security administration.
- DevSecOps automation.

## 18. Current Limitations

The analyser is a learning prototype rather than a production-ready security monitoring solution.

Its current limitations include:

- Analysis of a single authentication log file.
- Detection based on one authentication failure phrase.
- Hardcoded authentication service categories.
- No analysis of rotated or compressed logs.
- No automatic timezone normalisation.
- No source IP extraction or event correlation.
- No real-time monitoring or alerting.
- No automatic distinction between benign and malicious authentication failures.
- Integrated error-handling branches have not been independently tested, although the separate exception handling exercise was successful.

## 19. Future Improvements

Future development may include:

- Configurable log file paths.
- Additional authentication failure detection patterns.
- Dynamic authentication service extraction.
- Username-based event aggregation.
- Source IP extraction.
- Timestamp normalisation.
- JSON and CSV report generation.
- Automated unit testing.
- Threshold-based security alerts.
- Scheduled execution.
- Integration with security monitoring platforms.

## 20. Personal Reflection

This practical helped me understand how Python can be applied to real cybersecurity investigations.

Rather than manually searching and counting authentication failure records, I developed a tool capable of processing logs, extracting relevant evidence, categorising events and generating a structured report.

I also learnt that security automation must go beyond producing results. It must handle errors, preserve evidence accurately, validate findings and communicate its limitations.

This project represents a practical step in my development across cybersecurity, SOC analysis, digital investigations and DevSecOps.
