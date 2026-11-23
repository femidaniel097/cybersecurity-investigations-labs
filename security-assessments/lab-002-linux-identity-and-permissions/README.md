# LINUX-SEC-002 — Linux Identity, File Permissions and Privilege Investigation

**Date:** 23 November 2026  
**Environment:** Ubuntu 26.04.1 LTS, Oracle VirtualBox  
**Investigator:** Femi Daniel Somuyiwa  
**Lab Status:** Practical exercises completed

## 1. Objective

Investigate Linux user identities, group membership, file ownership, access permissions and sudo privileges.

Demonstrate how an insecure file permission can expose information to another local account and verify remediation.

## 2. User and Group Investigation

### Commands

```bash
whoami
id
getent passwd | tail -15
getent group sudo
```

### Purpose

- `whoami`: Identify the current effective username.
- `id`: Display user ID, group ID and group memberships.
- `getent passwd`: Query the configured user account database.
- `getent group sudo`: Inspect sudo group membership.

### Findings

The system contained two regular user accounts:

| Username | UID | Primary GID |
|---|---|---|
| dansom | 1000 | 1000 |
| student | 1001 | 1001 |

Both accounts were listed as members of the `sudo` group.

The `dansom` account also belonged to several supplementary groups, including `adm`, `lpadmin` and `lxd`.

**Security interpretation:** Administrative and service-related group memberships should be reviewed according to operational need.

## 3. File Ownership and Permission Investigation

### Commands

```bash
mkdir -p ~/security-labs/day7-permissions
cd ~/security-labs/day7-permissions
echo "Confidential security investigation notes" > investigation.txt
ls -l investigation.txt
stat -c '%A %a %U %G %n' investigation.txt
```

### Initial Result

```text
-rw-rw-r-- 664 dansom dansom investigation.txt
```

### Interpretation

- Owner: Read and write
- Group: Read and write
- Others: Read only

The file permitted reading by other local users, subject to directory access controls.

## 4. Investigating Directory Traversal

### Commands

```bash
namei -l ~/security-labs/day7-permissions/investigation.txt
sudo -u student cat /home/dansom/security-labs/day7-permissions/investigation.txt
```

### Findings

The `/home/dansom` directory had permission `750`.

The `student` account could not traverse this directory using ordinary permissions.

Observed result:

```text
cat: /home/dansom/security-labs/day7-permissions/investigation.txt: Permission denied
```

**Lesson:** File access depends on both file permissions and the ability to traverse the directory path.

## 5. Applying Least Privilege

### Command

```bash
chmod 600 investigation.txt
```

### Verification

```bash
ls -l investigation.txt
stat -c '%A %a %U %G %n' investigation.txt
cat investigation.txt
```

### Results

```text
-rw------- 600 dansom dansom investigation.txt
Confidential security investigation notes
```

The file owner retained read/write access, while group and others permissions were removed.

A subsequent access attempt using `student` returned `Permission denied`.

## 6. Investigating Sudo Privileges

### Commands

```bash
whoami
id -u
sudo whoami
sudo id -u
sudo -l
```

### Results

| Command | Output |
|---|---|
| whoami | dansom |
| id -u | 1000 |
| sudo whoami | root |
| sudo id -u | 0 |

Sudo policy output:

```text
User dansom may run the following commands on dansom-VirtualBox:
    (ALL : ALL) ALL
```

### Interpretation

The `dansom` account can execute commands through sudo as root or other users and groups, subject to the applicable authentication requirements.

Ordinary commands continue to run under the user's normal privileges unless privilege elevation is explicitly requested.

## 7. Controlled File-Permission Misconfiguration

### Objective

Demonstrate how permissive file permissions can expose information to another local user.

### Step 1 — Create the test environment

```bash
mkdir -p /tmp/day7-access-test
chmod 755 /tmp/day7-access-test
echo "Training data only - no real secrets" > /tmp/day7-access-test/sample.txt
chmod 644 /tmp/day7-access-test/sample.txt
```

### Step 2 — Test access as another user

```bash
sudo -u student cat /tmp/day7-access-test/sample.txt
```

Actual output:

```text
Training data only - no real secrets
```

**Finding:** The `student` account could read the file.

### Step 3 — Remediate the misconfiguration

```bash
chmod 600 /tmp/day7-access-test/sample.txt
```

### Step 4 — Verify remediation

```bash
sudo -u student cat /tmp/day7-access-test/sample.txt
```

Actual output:

```text
cat: /tmp/day7-access-test/sample.txt: Permission denied
```

**Finding:** The permission change successfully blocked ordinary read access by `student`.

## 8. Security Assessment

**Finding SEC-003: Overly Permissive File Read Access**

- **Affected resource:** `/tmp/day7-access-test/sample.txt`
- **Initial permissions:** `644`
- **Security impact:** Other local users could read the file.
- **Remediation:** Changed permissions to `600`.
- **Verification:** Access as `student` was denied.
- **Status:** Remediated and verified.

This was an intentionally introduced misconfiguration in a controlled training environment, not a discovered production vulnerability.

## 9. Lessons Learned

1. Linux users have unique UIDs and primary groups.
2. Users may belong to multiple supplementary groups.
3. Sudo privileges do not automatically apply to ordinary commands.
4. Directory execute permissions control traversal.
5. File permissions control ordinary read, write and execute access.
6. Permission `600` restricts normal file access to its owner.
7. Sensitive files should not be unnecessarily readable by other users.
8. Security remediation should be followed by verification.

## 10. Conclusion

This lab demonstrated practical Linux identity investigation, permission analysis, sudo privilege verification and file-access hardening.

The controlled misconfiguration was successfully remediated, with access verification confirming the intended result.

**Practical Lab Status: Completed.**
