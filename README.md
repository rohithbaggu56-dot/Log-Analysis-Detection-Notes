## 📊 Log Analysis & Detection Notes (SOC Investigation Lab)

This section documents my hands-on practice with **Windows and Linux log analysis** to understand how malicious and suspicious activity appears in logs and how SOC analysts detect, investigate, and escalate incidents.

The focus was on **event correlation, identifying Indicators of Compromise (IOCs), and building a structured investigation mindset**.

---

## 🪟 Windows Log Analysis

### 🔐 Windows Security Event Logs (Event Viewer)

I analyzed Windows Security logs to understand authentication behavior, account activity, and system changes.

#### Key Event IDs Explored

| Event ID | Description | SOC Relevance |
|--------|-------------|---------------|
| **4624** | Successful logon | Baseline user behavior |
| **4625** | Failed logon attempt | Brute force / password spray |
| **4634** | Logoff | Session tracking |
| **4672** | Special privileges assigned | Admin access detection |
| **4688** | Process creation | Malware execution tracking |
| **4697** | Service installed | Persistence mechanism |
| **4720** | New user created | Unauthorized account creation |
| **4728 / 4732** | User added to group | Privilege escalation |
| **4799** | Local group membership queried | Reconnaissance activity |

---

### 🧠 Investigation Approach (Windows)

1. Established **baseline behavior** using successful logons (4624)
2. Identified abnormal patterns:
   - Repeated **4625** failures from same IP/user
   - Logons at unusual hours
3. Correlated **privilege-related events**:
   - 4672 + 4688 → possible admin misuse
4. Investigated suspicious processes:
   - Unknown executables
   - Unexpected parent-child process chains
5. Flagged potential IOCs:
   - Source IPs
   - Suspicious usernames
   - Unusual process names

---

## 🧩 Sysmon Analysis (Advanced Windows Telemetry)

Configured and explored **Sysmon logs** for deeper visibility beyond default Windows logging.

#### Key Sysmon Event IDs

| Sysmon ID | Description | Use Case |
|---------|-------------|---------|
| **1** | Process creation | Detect suspicious execution |
| **3** | Network connection | Outbound C2 detection |
| **7** | Image loaded | DLL hijacking |
| **10** | Process access | Credential dumping |
| **11** | File creation | Malware drops |
| **13** | Registry modification | Persistence |

---

### 🔍 Detection Logic with Sysmon

- Identified suspicious behavior such as:
  - Command-line abuse (`powershell`, `cmd`, encoded commands)
  - Processes making unexpected network connections
- Correlated:
  - Process creation → network activity → persistence
- Practiced **alert-style thinking**, not just log reading

---

## 🐧 Linux Log Analysis

Analyzed Linux authentication and system logs to detect brute-force attacks and unauthorized access attempts.

---

## 📁 Linux Log Sources Analyzed

| Log File | Description | SOC Use Case |
|-------|-------------|-------------|
| `/var/log/auth.log` | Authentication & SSH activity | Brute-force, unauthorized access |
| `/var/log/syslog` | System-wide messages | Service abuse, system anomalies |
| `/var/log/secure` | Security logs (RHEL-based) | Auth failures, privilege misuse |
| SSH daemon logs | SSH session details | Remote attack detection |

---

## 🔐 SSH Authentication Analysis

I analyzed SSH logs to detect unauthorized access attempts and attacker behavior.

### Common Log Indicators Identified

- `Failed password`
- `Invalid user`
- `authentication failure`
- `Connection closed`
- `Too many authentication failures`

Example log evidence:
- Repeated login failures against non-existent users
- Multiple attempts from the same external IP
- Attacks targeting `root` and common usernames

---

### 🔎 Investigation Commands Used

```bash
grep "Failed password" auth.log
grep "Invalid user" auth.log
grep "authentication failure" auth.log
grep "sshd" auth.log
