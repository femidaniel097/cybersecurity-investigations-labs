# LAB-001 — Ubuntu Host Security Baseline Assessment

**Investigation Date:** 21–22 November 2026  
**Investigator:** Femi Daniel Somuyiwa  
**Hostname:** `dansom-VirtualBox`  
**Operating System:** Ubuntu 26.04.1 LTS  
**Kernel:** `7.0.0-43-generic`  
**Environment:** Oracle VirtualBox  
**Network Mode:** NAT (after troubleshooting)

---

## 1. Lab Objective

To conduct a practical Linux host security assessment by investigating:

- Operating system information and network configuration.
- Running processes and their resource consumption.
- Listening network ports and associated services.
- Firewall and SSH security configurations.
- Unnecessary network services and attack surface reduction.

The exercise also involved troubleshooting a DHCP connectivity failure.

---

## 2. System Reconnaissance

### Commands Used

```bash
hostnamectl
uname -r
uptime
ip -br addr
```

### Command Explanations

| Command | Purpose |
|---|---|
| `hostnamectl` | Displays hostname, operating system, kernel and virtualisation information |
| `uname -r` | Displays the running Linux kernel version |
| `uptime` | Shows system uptime, login sessions and load averages |
| `ip -br addr` | Displays network interfaces and assigned IP addresses in brief format |

### Observed Results

```text
Hostname: dansom-VirtualBox
OS: Ubuntu 26.04.1 LTS
Kernel: 7.0.0-43-generic
Virtualization: Oracle
Architecture: x86-64
```

The system was running inside Oracle VirtualBox.

**Security lesson:** System reconnaissance establishes a baseline for identifying unexpected configuration changes, outdated software and potential vulnerabilities.

---

## 3. Network Troubleshooting — DHCP Failure

### Problem Identified

During reconnaissance, the network interface `enp0s3` was UP but had no IPv4 address.

Observed output:

```text
lo       UNKNOWN  127.0.0.1/8 ::1/128
enp0s3   UP       fe80::a00:27ff:fe7c:2cba/64
```

The `fe80::` address is an IPv6 link-local address.

### Commands Used

```bash
nmcli device status
```

**Purpose:** Shows NetworkManager's connection state for each network interface.

```bash
nmcli connection show
```

**Purpose:** Lists saved NetworkManager connection profiles.

```bash
ip route
```

**Purpose:** Displays the IPv4 routing table, including default gateway information.

The routing table was empty.

### Inspecting IPv4 Configuration

```bash
nmcli connection show "netplan-enp0s3" | grep -E 'ipv4.method|ipv4.addresses|ipv4.gateway|ipv4.dns'
```

**Purpose:** Examines the connection's IPv4 settings.

**Finding:** `ipv4.method` was set to `auto`, indicating automatic IPv4 configuration.

### Investigating NetworkManager Logs

```bash
sudo journalctl -u NetworkManager -b --no-pager -n 35
```

**Command breakdown:**

| Option | Meaning |
|---|---|
| `-u NetworkManager` | Select NetworkManager logs |
| `-b` | Current boot |
| `--no-pager` | Print output directly |
| `-n 35` | Show the latest 35 entries |

The logs indicated that DHCP negotiation began but no lease was obtained.

### Resolution

The VirtualBox network adapter was initially configured as **Bridged Adapter**.

Changing the adapter to **NAT** restored network connectivity.

**Root cause assessment:** DHCP lease acquisition failed through the bridged network path. The precise upstream cause was not confirmed.

### Verification Commands

```bash
ip -br addr
ip route
ping -c 4 1.1.1.1
getent hosts example.com
```

These commands can verify IP addressing, routing, connectivity and DNS resolution.

The network was reported working after switching to NAT; individual verification outputs were not retained in this report.

**Lesson learned:** A network interface can be UP without having a usable IP configuration. Virtualisation networking modes influence how a guest obtains network access.

---

## 4. Linux Process Investigation

### Objective

Identify running processes, examine resource consumption and investigate process legitimacy.

### CPU Investigation

```bash
ps aux --sort=-%cpu | head -15
```

**Explanation:**

- `ps aux` — Display processes with detailed information.
- `--sort=-%cpu` — Sort by CPU usage in descending order.
- `|` — Pipe output to another command.
- `head -15` — Display the first 15 lines.

### Memory Investigation

```bash
ps aux --sort=-%mem | head -15
```

**Purpose:** Identify processes consuming the most memory.

### Observed Processes

| Process | Function |
|---|---|
| `gnome-shell` | GNOME graphical desktop |
| `gjs` | GNOME JavaScript interpreter |
| `nautilus` | File manager |
| `terminator` | Terminal emulator |
| `systemd` | System and service manager |
| `fwupd` | Firmware update service |

### Detailed Process Investigation

Selected process: `gnome-shell`

```bash
ps -p 1906 -o pid,ppid,user,%cpu,%mem,etime,cmd
```

**Purpose:** Display detailed information about a specific process.

**Important fields:**

| Field | Meaning |
|---|---|
| PID | Process identifier |
| PPID | Parent process identifier |
| USER | Process owner |
| %CPU | CPU usage |
| %MEM | Memory usage |
| ETIME | Elapsed running time |
| CMD | Command line |

### Identifying the Executable

```bash
readlink -f /proc/1906/exe
```

Observed output:

```text
/usr/bin/gnome-shell
```

This resolved the process executable to the expected GNOME Shell location.

### Checking Process State

```bash
ps -p 1906 -o pid,stat,comm
```

Observed output:

```text
1906 Rsl gnome-shell
```

**Process state interpretation:**

- `R` — Running or runnable.
- `s` — Session leader.
- `l` — Multithreaded.

### Security Assessment

The examined process was consistent with expected Ubuntu desktop behaviour.

No suspicious behaviour was established from the collected evidence.

**Lesson learned:** A process should be assessed using its executable location, ownership, command line, parent process and behaviour—not resource consumption alone.

---

## 5. Listening Port and Network Service Investigation

### Command Used

```bash
sudo ss -tulnp
```

### Command Breakdown

| Option | Meaning |
|---|---|
| `-t` | TCP sockets |
| `-u` | UDP sockets |
| `-l` | Listening sockets |
| `-n` | Numeric addresses and ports |
| `-p` | Associated processes |

### Observed Services

| Port | Process | Purpose |
|---|---|---|
| 22/TCP | systemd / SSH socket | Remote SSH access |
| 53/TCP/UDP | systemd-resolved | DNS resolution |
| 323/UDP | chronyd | Time synchronisation control |
| 5353/UDP | avahi-daemon | Multicast DNS |
| 631/TCP | cupsd | Printing |

### Understanding Listening Addresses

**127.0.0.1**

Loopback address, generally accessible only from the local machine.

**0.0.0.0**

Wildcard IPv4 address, allowing the socket to listen on all local IPv4 interfaces.

**[::]**

IPv6 wildcard listening address.

### Security Finding

SSH was listening on:

```text
0.0.0.0:22
[::]:22
```

This required further investigation because the service was available on all local network interfaces.

**Lesson learned:** Listening ports reveal potential network entry points, but reachability also depends on routing, firewalls and virtual network configuration.

---

## 6. Firewall Investigation

### Command Used

```bash
sudo ufw status verbose
```

### Observed Output

```text
Status: inactive
```

**Interpretation:** UFW was not actively enforcing firewall rules.

This did not prove that all incoming connections were unrestricted because other network controls could still apply.

**Security recommendation:** Review firewall requirements before exposing network services.

---

## 7. SSH Service Investigation

### Checking SSH Service Status

```bash
systemctl is-active ssh
```

Observed output:

```text
inactive
```

### Checking SSH Socket Activation

```bash
systemctl status ssh.socket --no-pager
```

Observed result:

```text
Active: active (listening)
Listen: 0.0.0.0:22
        [::]:22
Triggers: ssh.service
```

### Explanation

Ubuntu was using systemd socket activation.

This allows `ssh.socket` to listen for connections even when `ssh.service` is inactive.

An incoming connection can trigger the SSH service.

### Inspecting SSH Configuration

```bash
sudo sshd -T | grep -E '^(permitrootlogin|passwordauthentication|pubkeyauthentication|allowusers|port) '
```

Observed output:

```text
port 22
permitrootlogin prohibit-password
pubkeyauthentication yes
passwordauthentication yes
```

### Interpretation

- SSH used port 22.
- Root password authentication was prohibited.
- Public-key authentication was enabled.
- Password authentication was enabled.
- No global `AllowUsers` restriction appeared in the filtered output.

These were effective global settings; connection-specific SSH `Match` rules may require additional evaluation.

---

## 8. Security Finding SEC-001 — Unnecessary SSH Service

### Finding

The SSH listening socket was enabled even though Ubuntu was accessed through the VirtualBox desktop.

### Potential Risk

An unnecessary listening service increases attack surface if the port becomes reachable.

The VM's NAT networking reduced ordinary inbound reachability, but did not eliminate every possible access path.

### Remediation Command

```bash
sudo systemctl disable --now ssh.socket
```

**Explanation:**

- `systemctl` — Controls systemd units.
- `disable` — Prevents the socket from automatically starting through its enablement configuration.
- `--now` — Stops the socket immediately.
- `ssh.socket` — The SSH listening socket unit.

### Verification Commands

```bash
systemctl is-active ssh.service
sudo ss -tlnp | grep ':22 '
systemctl is-enabled ssh.socket
systemctl is-active ssh.socket
```

### Actual Verification Results

```text
ssh.service: inactive
Port 22: no matching TCP listener
ssh.socket enabled state: disabled
ssh.socket active state: inactive
```

### Final Assessment

**Finding:** Unnecessary SSH listening service.

**Remediation:** SSH socket disabled and stopped.

**Verification:** No TCP port 22 listener detected.

**Status:** Remediated and verified.

### Restoring SSH for Future Labs

```bash
sudo systemctl enable --now ssh.socket
```

This command can restore SSH socket activation when remote-access exercises are required.

---

## 9. Key Security Lessons

1. Linux system reconnaissance is essential before performing security analysis.
2. Network interfaces can be active without working IPv4 connectivity.
3. DHCP and VirtualBox network configurations can affect connectivity.
4. High CPU or memory usage does not automatically indicate malicious activity.
5. Process executable paths and ownership provide useful investigative evidence.
6. Systemd socket activation can expose a listening port even when the associated service is inactive.
7. A listening port does not automatically establish external reachability.
8. Unnecessary services should be reviewed and disabled where appropriate.
9. Security changes must be verified after implementation.
10. Documenting commands, outputs and reasoning makes investigations reproducible.

---

## 10. Final Assessment

The Ubuntu virtual machine was examined for operating system information, running processes, listening services, firewall configuration and SSH exposure.

A DHCP connectivity problem was investigated and connectivity restored using VirtualBox NAT.

An unnecessary SSH listening service was identified and successfully disabled.

No confirmed malicious activity was identified in the reviewed evidence.

**Lab Status:** Completed  
**Security Finding SEC-001:** Remediated and verified

---

*Cybersecurity & DevSecOps Practical Learning Portfolio — Femi Daniel Somuyiwa*
