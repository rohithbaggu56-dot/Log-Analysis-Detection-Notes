# 📊 Log Analysis & Detection

![Linux](https://img.shields.io/badge/Linux_Log_Analysis-333333?style=for-the-badge&logo=linux&logoColor=white)
![Windows](https://img.shields.io/badge/Windows_Event_Viewer-2F7DE1?style=for-the-badge)
![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?style=for-the-badge&logo=wireshark&logoColor=white)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE_ATT%26CK-FF0000?style=for-the-badge)

Raw Windows and Linux log analysis focused on reading event data directly and identifying suspicious activity without relying on SIEM alerts. Every finding here came from reading actual log files and packet captures.

---

## 📂 What Was Analyzed

---

### 🔴 SSH Brute Force Detection — Raw Log Analysis

Analyzed OpenSSH_2k.log directly in the terminal using grep filtering to identify brute force patterns across multiple source IPs.

**Raw terminal analysis — head command to read the log, grep -Ei filter to extract failed password, invalid user, and authentication failure events:**

<img width="1910" height="922" alt="Ubuntu-SSH-log analysis" src="https://github.com/user-attachments/assets/8650a691-144b-40d9-8680-860f290ee371" />

<br>

**Findings from the raw log:**

| Source IP | Username Targeted | Pattern |
|---|---|---|
| 173.234.31.186 | webmaster | Repeated invalid user attempts, connection closed preauth |
| 52.80.34.196 | test9 | Invalid user, resolves to AWS cn-north-1 compute |
| 202.100.179.208 | chen | Failed password for invalid user |
| 5.36.59.76 | root | 5 repeated failures, PAM lockout triggered, disconnected |
| 112.95.230.3 | root | Failed password for root on ports 45378 and 47068 |

- Server logged "POSSIBLE BREAK-IN ATTEMPT" for reverse mapping failure on 173.234.31.186
- 5.36.59.76 triggered PAM lockout — "Too many authentication failures for root" — showing the brute force threshold was hit
- Multiple source IPs targeting different usernames simultaneously indicating a distributed or scripted attack
- 📌 MITRE: `T1110` Brute Force · `T1110.001` Password Guessing · `T1078` Valid Accounts

---

### 🔴 DNS Traffic Analysis — Wireshark Packet Capture

Analyzed a DNS packet capture file (primary_capture.pcapng) in Wireshark, filtering for DNS responses to understand the resolution patterns and identify any unusual behavior.

**Wireshark DNS response analysis — filter dns.flags.response == 1 applied, showing 48 displayed packets out of 4,566 total captured:**

<img width="1903" height="975" alt="Wireshark-Dsn analysis" src="https://github.com/user-attachments/assets/0b30d074-55cd-40be-88db-4cfcf6067a3f" />

<br>

- Filtered DNS responses using `dns.flags.response == 1` to isolate server replies from client queries
- Observed standard A record responses for domains including example.org, google.com, gstatic.com
- CNAME chain visible for detectportal.firefox.com resolving through mozaws.net — normal browser behavior
- IPv6 AAAA records present alongside IPv4 A records for major domains
- Packet inspection of Frame 8 shows standard query response, no error, recursion available — normal DNS behavior
- 📌 MITRE: `T1071.004` DNS · `T1046` Network Service Scanning

---

### 🔴 Windows Log Analysis — Event ID Investigation

Analyzed Windows Security Event logs to identify authentication anomalies and suspicious activity patterns.

**Key Event IDs investigated:**

| Event ID | Description | What it indicates |
|---|---|---|
| 4625 | Failed logon | Brute force or credential stuffing attempt |
| 4624 | Successful logon | Baseline for detecting unusual successful access |
| 4648 | Logon using explicit credentials | Lateral movement indicator |
| 4740 | Account locked out | Brute force threshold reached |

- Identified repeated 4625 failures followed by 4740 account lockout — confirmed brute force sequence
- Correlated 4624 success events against 4625 failure patterns to identify which accounts were eventually accessed
- 📌 MITRE: `T1110` Brute Force · `T1078` Valid Accounts · `T1021` Remote Services

---

### 🔴 Linux Authentication Log Analysis

Analyzed Linux auth.log for privilege escalation attempts and abnormal authentication patterns.

**What was identified:**
- Sudo escalation attempts logged with timestamps and executing user
- PAM session open and close events used to track user session duration
- SSH login anomalies identified by correlating successful vs failed login ratios per source IP
- Root login attempts via SSH flagged as high priority since direct root SSH access should typically be disabled

- 📌 MITRE: `T1078` Valid Accounts · `T1548.003` Sudo and Sudo Caching

---

## 🔎 IOC Extraction Workflow

Every suspicious event was processed through this workflow:

```
Raw log entry identified
    ↓
Event ID or log field extracted
    ↓
Source IP, username, timestamp documented
    ↓
Pattern classified: brute force / escalation / lateral movement / anomaly
    ↓
Severity assigned: High / Medium / Low
    ↓
Finding documented with evidence
```

---

## 📁 Repository Structure

```
Log-Analysis-Detection-Notes/
├── linux/
│   ├── ssh-brute-force-analysis.png
│   └── auth-log-investigation.md
├── windows/
│   └── event-id-investigation.md
└── network/
    └── wireshark-dns-analysis.png
```

---

⬅️ [Back to SOC Home Lab](https://github.com/rohithbaggu56-dot/Home-SOC-Lab-Detection-Log-Analysis) · [Back to Portfolio](https://github.com/rohithbaggu56-dot)
