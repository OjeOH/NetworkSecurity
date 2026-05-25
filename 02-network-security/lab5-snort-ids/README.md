# 🚨 Lab 5 — Intrusion Detection System: SNORT on CentOS 7

Network Security II
**Tools:** SNORT 3, PulledPork, CentOS 7, VMware vSphere

---

## Objective

Deploy a **network-based Intrusion Detection System (IDS)** using **SNORT** — the world's most widely deployed open-source IDS/IPS — on a CentOS 7 Linux server. Configure it to monitor live network traffic, detect malicious patterns using rule sets, and validate it is capturing ICMP and other traffic as expected.

---

## What Is an IDS?

An **Intrusion Detection System** passively monitors network traffic and generates alerts when traffic matches a known attack signature or anomalous behavior pattern.

```
[Network Traffic]
       │
       ▼
  [SNORT IDS]
  ├─ Rules Engine  ←── Rule Sets (from PulledPork)
  └─ Alert Engine  ──► [Log File / Alert Console]
```

**IDS vs. IPS:**
- **IDS** (Intrusion Detection System) — detects and alerts; does not block
- **IPS** (Intrusion Prevention System) — detects and actively blocks traffic

SNORT can operate in both modes. This lab implements IDS mode.

---

## Architecture

```
[Network Segment to Monitor]
           │
     [Mirror/SPAN Port]
           │
     [SNORT Interface]
     CentOS 7 Server
     ├─ /etc/snort/snort.conf
     ├─ /etc/snort/rules/     ← Rule sets managed by PulledPork
     └─ /var/log/snort/       ← Alert logs
```

---

## Part 1 — Package Installation

SNORT has several dependencies that must be installed before SNORT itself. The installation was performed via the CentOS package manager:

```bash
# Step 1 — Enable EPEL repository
yum install epel-release -y

# Step 2 — Install DNF (modern package manager)
yum install dnf -y

# Step 3 — Install SNORT and dependencies
dnf install snort -y
```

**Key dependencies include:** `libpcap` (packet capture), `pcre` (pattern matching), `libdnet`, `daq` (Data Acquisition module).

---

## Part 2 — SNORT Configuration

### Directory Structure

SNORT requires a specific directory structure for rules, logs, and configuration:

```bash
# Create required directories
mkdir -p /etc/snort/rules
mkdir -p /var/log/snort
mkdir -p /usr/local/lib/snort_dynamicrules

# Set correct permissions
chmod -R 5775 /etc/snort
chmod -R 5775 /var/log/snort

# Create required empty files
touch /etc/snort/rules/white_list.rules
touch /etc/snort/rules/black_list.rules
touch /etc/snort/rules/local.rules
```

### snort.conf Configuration

Key configuration sections in `/etc/snort/snort.conf`:

```bash
# Define your protected network
ipvar HOME_NET 172.17.200.0/24   # Inside network
ipvar EXTERNAL_NET !$HOME_NET    # Everything else

# Rule paths
var RULE_PATH /etc/snort/rules
var SO_RULE_PATH /etc/snort/so_rules
var PREPROC_RULE_PATH /etc/snort/preproc_rules

# Log format
output alert_fast: stdout
```

### PulledPork — Automated Rule Management

**PulledPork** automates the download and management of SNORT rule sets — equivalent to running Windows Update for your IDS signatures.

```bash
# Install Perl (required by PulledPork)
yum install perl -y

# Install PulledPork
# (downloaded from GitHub, installed to /usr/local/bin/)
perl pulledpork.pl -c /etc/snort/pulledpork.conf -l

# Verify PulledPork
perl pulledpork.pl -V
```

PulledPork merges all downloaded rule sets into a single `snort.rules` file, handles rule conflicts, and can enable/disable specific rules via `enablesid.conf` and `disablesid.conf`.

---

## Part 3 — Testing SNORT

### Disable SELinux (Lab Environment)

SELinux (Security-Enhanced Linux) in CentOS 7 restricts SNORT's network access. For the lab, it was temporarily disabled:

```bash
setenforce 0  # Temporarily disable
# For permanent disable: edit /etc/selinux/config → SELINUX=disabled
```

### Verify SNORT Configuration

```bash
snort -T -i eth0 -c /etc/snort/snort.conf
# -T = test config, -i = interface, -c = config file
# Should end with: "Snort successfully validated the configuration!"
```

### Run SNORT in IDS Mode

```bash
snort -A console -i eth0 -c /etc/snort/snort.conf
# -A console = print alerts to console in real-time
```

### Test with ICMP Traffic

Pinged the SNORT server from another host:

```bash
ping <snort-server-ip>
```

SNORT generated ICMP detection alerts in real-time on the console — confirming:
- ✅ SNORT was actively monitoring the network interface
- ✅ Rule sets were loaded and pattern-matching was functional
- ✅ ICMP packets were captured and matched against ICMP rules

---

## Sample SNORT Alert Format

```
[**] [1:408:5] ICMP Echo Request [**]
[Classification: Misc activity] [Priority: 3]
05/24-14:32:11.456789 172.17.200.50 -> 172.17.200.100
ICMP TTL:64 TOS:0x0 ID:1234 IpLen:20 DgmLen:84
```

Breaking down the alert:
- `1:408:5` → Generator ID : Signature ID : Revision
- `ICMP Echo Request` → Rule message
- Priority 3 → Informational (low severity)
- Source → Destination mapping

---

## Key Takeaways

> **Real-world relevance:** IDS/IPS systems are a core component of any security monitoring stack. SNORT powers many commercial security appliances and is the detection engine behind countless SOC environments. Understanding how to deploy, configure, and tune SNORT — and how PulledPork keeps rules current — is directly applicable to SOC analyst, security engineer, and network defender roles.

- SELinux and firewall configurations are common deployment blockers — always verify permissions first
- PulledPork is essential for maintaining current signatures — manual rule management doesn't scale
- SNORT in IDS mode is passive — it generates alerts but doesn't block traffic; IPS mode (inline) does block
- Rule tuning is critical in production — an untuned SNORT generates thousands of false positives that overwhelm analysts
- The `snort -T` test mode should always be run after any configuration change before restarting in production

---

## Files

| File | Description |
|---|---|
| `Ooje7862-INFO8501-24S-SEC1-PORTFOLIO_3__3_.docx` | Full lab report — SNORT installation, configuration, and testing with screenshots |
