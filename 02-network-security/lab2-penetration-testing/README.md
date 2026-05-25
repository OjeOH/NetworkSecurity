# 🔓 Lab 2 — Penetration Testing: FTP Exploit & RSA SSH Authentication

Network Security II
**Tools:** CentOS 7 Linux VM, Wireshark, PuTTY, Windows Server 2019

> ⚠️ **Ethical Notice:** All activities in this lab were performed in an isolated, controlled lab environment on systems I own and operate for educational purposes. This lab is documented to demonstrate understanding of vulnerabilities — not to enable unauthorized access.

---

## Objective

This lab has two parts with a clear narrative arc:

1. **Attack an FTP server** to demonstrate how easily plaintext credentials can be captured over the network
2. **Implement RSA key-based SSH authentication** to show the secure alternative

---

## Part A — FTP Server Penetration Test

### Lab Setup

| Component | Details |
|---|---|
| Target | CentOS 7 Linux VM running vsftpd (FTP service) |
| Attacker | Windows workstation on same network segment |
| Tool | Wireshark (packet capture and analysis) |

### Attack Methodology

**Step 1 — Deploy the FTP Target**

Provisioned a CentOS 7 VM in the lab environment, installed vsftpd (FTP daemon), created a test user account, and started the FTP service.

```bash
# On CentOS FTP server:
yum install vsftpd -y
useradd ftpuser
systemctl start vsftpd

# Temporarily disabled firewall for the demonstration:
systemctl stop firewalld
```

**Step 2 — Launch Wireshark**

Started Wireshark on the workstation, capturing on the interface connected to the FTP server's network segment.

**Step 3 — Authenticate to the FTP Server**

From the Windows workstation command prompt:

```cmd
ftp <ftp-server-ip>
Name: ftpuser
Password: <password>
```

Authentication was successful — FTP connected and the session was active.

**Step 4 — Capture and Analyze Credentials**

In Wireshark, filtered for FTP traffic:
```
Filter: ftp
```

The captured packets clearly showed:
- **Username transmitted in plaintext** — `USER ftpuser`
- **Password transmitted in plaintext** — `PASS <password>`

No decryption required. Any user on the same network segment (or a compromised switch with port mirroring) can read these credentials directly.

### Why FTP is Dangerous

| Vulnerability | Impact |
|---|---|
| Plaintext authentication | Credentials captured in any packet capture tool |
| No transport encryption | All data files transferred are also readable in plaintext |
| Basic authentication only | No support for certificate-based or MFA login |
| No session integrity | Session hijacking attacks are possible |

### Recommendation

Organizations should replace FTP with **SFTP (SSH File Transfer Protocol)** or **FTPS (FTP over SSL/TLS)**. SFTP encrypts both the authentication and data channel, making credential capture impossible with packet sniffing alone.

---

## Part B — RSA Key Authentication via SSH

### Objective

After demonstrating the vulnerability of password-based authentication in Part A, this section implements **RSA public key authentication** for SSH — eliminating passwords entirely from the login process.

### How RSA Key Authentication Works

```
┌─────────────────────────────────────────────────────┐
│  Key Generation (one-time setup)                    │
│                                                      │
│  PuTTYgen → [Private Key]  +  [Public Key]          │
│                 (stays on       (copied to          │
│                  client)         server)            │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│  Authentication (every login)                       │
│                                                      │
│  Client sends: "I have the private key for          │
│                 this public key"                    │
│                                                      │
│  Server verifies: cryptographic challenge —         │
│                   no password ever transmitted     │
└─────────────────────────────────────────────────────┘
```

### Configuration Steps

**Step 1 — Generate RSA Key Pair (Windows/PuTTY)**

Using PuTTYgen:
- Key type: RSA, 2048 bits
- Generated the key pair
- Saved the **private key** (`.ppk` file) to the local PuTTY directory
- Copied the **public key** string for use on the server

**Step 2 — Configure the CentOS Server**

```bash
# Create a user account
useradd sshuser

# Create .ssh directory with correct permissions
mkdir /home/sshuser/.ssh
chmod 700 /home/sshuser/.ssh

# Add the public key to authorized_keys
nano /home/sshuser/.ssh/authorized_keys
# (Paste the public key from PuTTYgen)

# Set correct permissions — critical!
chmod 600 /home/sshuser/.ssh/authorized_keys
chown -R sshuser:sshuser /home/sshuser/.ssh
```

> **Permission note:** SSH will silently refuse to use `authorized_keys` if permissions are too permissive. `700` on `.ssh/` and `600` on `authorized_keys` are strict requirements.

**Step 3 — Load Private Key in PuTTY**

Configured PuTTY to use the saved `.ppk` private key under Connection > SSH > Auth.

**Step 4 — Verify Passwordless Authentication**

Connecting to the CentOS server with PuTTY:
- ✅ Authentication succeeded **without entering a password**
- The private key on the client was cryptographically matched against the public key on the server

---

## Comparison: Password Auth vs. RSA Key Auth

| Factor | Password Authentication | RSA Key Authentication |
|---|---|---|
| Credential in transit | Password sent (even over SSH) | Private key never leaves client |
| Brute force risk | Yes | No — private key is never guessed |
| Phishing risk | Yes | No |
| Key management | Simple | Requires key distribution |
| Automation-friendly | No | Yes — scripts can auth without prompts |

---

## Key Takeaways

> **Real-world relevance:** FTP is still running on thousands of legacy systems in production. Understanding exactly *how* credentials are exposed is what drives organizations to modernize. RSA key authentication is the standard for SSH access to servers and network devices in any mature security environment — it's what cloud providers like AWS and Azure use by default for Linux VMs.

- Any protocol transmitting credentials in cleartext is unacceptable in a production environment
- Wireshark is an essential tool for both attack and defense — defenders use it to verify encryption is working
- RSA key length of 2048 bits is the current minimum; 4096 bits is recommended for new deployments
- The `authorized_keys` permission model is a common misconfiguration point — always verify with `ls -la`

---

## Files

| File | Description |
|---|---|
| `Ooje7862-INF08501-Sec1-24S-Portfolio2__2_.docx` | Full lab report — FTP attack, Wireshark analysis, RSA SSH setup with screenshots |
