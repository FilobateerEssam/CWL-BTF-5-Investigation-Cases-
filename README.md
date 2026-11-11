# 🛡️ Blue Team Investigations: SOC Analyst Lab Portfolio

> Five hands-on incident investigations completed during the **Blue Team Fundamentals (BTF)** course by **CyberWarFare Labs (CWL)**, covering web attacks, post-exploitation, memory forensics and network forensics.

![Focus](https://img.shields.io/badge/Focus-Blue%20Team%20%2F%20SOC-blue)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-005571)
![Forensics](https://img.shields.io/badge/Forensics-Volatility%203-orange)
![Network](https://img.shields.io/badge/Network-Wireshark-1679A7)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Lab Environment & Tools](#-lab-environment--tools)
- [Investigations at a Glance](#-investigations-at-a-glance)
- [Investigation 1: SQL Injection](#1--sql-injection-sqli)
- [Investigation 2: Cross-Site Scripting (XSS)](#2--cross-site-scripting-xss)
- [Investigation 3: Remote File Inclusion / Malicious Upload (RFI)](#3--remote-file-inclusion--malicious-upload-rfi)
- [Investigation 4: Memory Dump Analysis](#4--memory-dump-analysis)
- [Investigation 5: External Network Communication](#5--external-network-communication)
- [Consolidated IOCs](#-consolidated-iocs)
- [MITRE ATT&CK Mapping](#-mitre-attck-mapping)
- [Key Takeaways](#-key-takeaways)
- [Repository Structure](#-repository-structure)

---

## 📌 Overview

Each investigation follows a consistent SOC workflow:

1. **Define the objective:** identify the *source*, *scope* and *impact* of the incident.
2. **Collect telemetry:** web logs in Wazuh, memory images, packet captures.
3. **Analyze patterns:** URIs, status codes, process lists, network flags.
4. **Correlate across sources:** for example, SIEM logs with PCAP evidence.
5. **Identify the root cause and IOCs.**

The scenarios simulate a realistic attack chain:

```
Web attack (SQLi / XSS)  →  Malicious file upload (RFI)  →  Reverse shell  →  Endpoint compromise  →  C2-style external communication
```

---

## 🧰 Lab Environment & Tools

| Category | Tool | Purpose |
|---|---|---|
| SIEM | **Wazuh** (`wazuh-archives` index) | Log search, decoder filtering, visualizations |
| Web server logs | Apache2 `access.log` | Request, URI and status-code analysis |
| Memory forensics | **Volatility 3** | Process, command-line and network analysis of `.raw` dumps |
| Network forensics | **Wireshark** | PCAP analysis, TCP stream following, endpoint statistics |
| Threat intel | **VirusTotal** | Reputation check on the extracted binary |
| Target app | Intentionally vulnerable web app (`/prod/vulnerabilities/*`) | Attack surface |

---

## 🗂️ Investigations at a Glance

| # | Investigation | Data Source | Key Finding |
|---|---|---|---|
| 1 | SQL Injection | Wazuh / Apache logs | Attacker `192.168.117.143` targeting `/prod/vulnerabilities/sqli` |
| 2 | Cross-Site Scripting | Wazuh / Apache logs | JavaScript payloads in URL requests to `/prod/vulnerabilities/xss_r/` |
| 3 | Remote File Inclusion | Wazuh + PCAP | `rev.php` uploaded via POST; reverse shell over port **9001** |
| 4 | Memory Dump Analysis | `suspected.raw` | `shell.exe` (PID **7396**) connected to `192.168.121.242:4444` |
| 5 | External Network Communication | `external.pcapng` | Interactive shell session over port **4444** (SYN/ACK → PSH flow) |

---

## 1. 💉 SQL Injection (SQLi)

**Targeted URL:** `http://192.168.117.190/prod/vulnerabilities/sqli`

**Objective:** Identify the source, scope and impact of suspected SQL injection activity.

### Methodology

1. Open the `wazuh-archives` index and filter by the `web-accesslog` decoder.
2. Narrow to the Apache access log and the vulnerable URI:
   ```
   location : "/var/log/apache2/access.log" and data.url: /prod/vulnerabilities/sqli*
   ```
3. Review the **status codes** of the requests:
   ```
   location : "/var/log/apache2/access.log" and data.url: /prod/vulnerabilities/sqli* and data.id: *
   ```
4. Pivot on the source IP of the requests.

### Findings

- Repeated requests to the `sqli` endpoint, abnormal compared to normal traffic.
- `HTTP 200` responses show the server processed the requests successfully.
- **Attacker IP:** `192.168.117.143`

---

## 2. 🕸️ Cross-Site Scripting (XSS)

**Targeted URL:** `http://192.168.117.190/prod/vulnerabilities/xss_r/`

**Objective:** Detect reflected XSS attempts and identify the origin.

### Methodology

1. Filter Apache access logs:
   ```
   decoder.name : web-accesslog and location : "/var/log/apache2/access.log"
   ```
2. Narrow to the vulnerabilities directory, then to XSS:
   ```
   ... and data.url: /prod/vulnerabilities/*      → 19 events
   ... and data.url: /prod/vulnerabilities/xss*   → 6 events
   ```
3. Inspect status codes (`data.id: *`).
4. Identify the source IP.

### Findings

- **6 XSS-related requests** out of 19 requests to the vulnerabilities path.
- JavaScript commands inside URL parameters, which is not normal user behavior.
- Status codes observed: `200` (successful) and `404`.
- **Attacker IP:** `192.168.117.143`

---

## 3. 📤 Remote File Inclusion / Malicious Upload (RFI)

**Targeted URL:** `http://192.168.117.190/prod/vulnerabilities/upload/`

**Objective:** Determine whether an insecure upload function was abused to deploy a web shell.

### Methodology

1. Search the file-inclusion path (`/prod/vulnerabilities/f*`): 6 events, including `password` and `php` references.
2. Search the upload path:
   ```
   decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url: /prod/vulnerabilities/up*
   ```
   → 2 events.
3. Filter by `data.protocol: POST` to find the upload request.
4. Because POST bodies have low visibility in access logs, **correlate with the PCAP** (`suspected.pcapng`).
5. Investigate the uploaded file:
   ```
   decoder.name : web-accesslog and location : "/var/log/apache2/access.log" and data.url: "/prod/hackable/uploads/rev.php"
   ```
6. Confirm the root cause in the PCAP by hunting the suspicious port:
   ```
   ip.dst == 192.168.117.190 && tcp.srcport == 9001
   ```

### Findings

- A PHP reverse shell, **`rev.php`**, was uploaded and then requested from `/prod/hackable/uploads/`.
- Network traffic over **port 9001** shows the reverse-shell connection back to the attacker.
- **Attacker IP:** `192.168.117.143` (source of the upload and the request for `rev.php`)

> ⚠️ **Note:** The original lab notes label both `192.168.117.143` and `192.168.117.190` as "attacker" at different stages. `.190` is the targeted web server, so the port-9001 traffic is the compromised server's side of the shell connection. Verify against your own PCAP before publishing.

---

## 4. 🧠 Memory Dump Analysis

**Evidence:** `suspected.raw`

**Objective:** Identify the malicious process, its network activity and the root cause on a compromised Windows endpoint.

> 🔒 Always analyze memory images in an **isolated environment**.

### Methodology (Volatility 3)

| Step | Command | Purpose |
|---|---|---|
| 1 | `python3 vol.py -f suspected.raw windows.info.Info` | OS version, architecture, system configuration |
| 2 | `python3 vol.py -f suspected.raw windows.pslist.PsList` | List running processes and spot anomalies |
| 3 | `python3 vol.py -f suspected.raw windows.cmdline.CmdLine` | Cross-check command-line arguments |
| 4 | `python3 vol.py -f suspected.raw windows.pslist --pid 7396 --dump` | Extract the suspicious executable |
| 5 | `strings pid.7396.0x400000.dmp` | Look for embedded indicators |
| 6 | Upload dump to **VirusTotal** | Reputation / detection check |
| 7 | `python3 vol.py -f suspected.raw windows.netstat.NetStat` | Network connections per process |

### Findings

- Suspicious process: **`shell.exe`**, **PID 7396**.
- `CmdLine` and `strings` showed nothing obvious, so the investigation shifted to VirusTotal and network analysis.
- `NetStat` confirmed `shell.exe` **communicating with `192.168.121.242` over port 4444**, a common default reverse-shell / Metasploit listener port.

---

## 5. 🌐 External Network Communication

**Evidence:** `external.pcapng` | **Targeted port:** `4444`

**Objective:** Analyze network traffic to confirm a compromised host communicating with an attacker.

### Methodology (Wireshark)

1. Prioritize the port: **Statistics → Endpoints → TCP/UDP**, then filter `tcp.dstport == 4444`.
2. Identify the endpoints and conversations: **Statistics → Conversations**:
   ```
   tcp.dstport == 4444 && ip.src == 192.168.132.238 && ip.dst == 192.168.132.242
   ```
3. Examine the **TCP flags**:
   - `SYN / SYN-ACK / ACK` shows an established connection.
   - `PSH` shows data or commands being transmitted.
4. Follow the stream on **packet 274** (**Follow → TCP Stream**).
5. Review the **reverse direction**:
   ```
   ip.src == 192.168.132.242 && ip.dst == 192.168.132.238
   ```

### Findings

- A stable session between `192.168.132.238` and `192.168.132.242` over port **4444**.
- Multiple `PSH` packets in both directions match an **interactive shell** with command execution.
- This confirms the pattern of a compromised host with an attacker-controlled shell.

---

## 🚩 Consolidated IOCs

| Type | Value | Context |
|---|---|---|
| IP (attacker) | `192.168.117.143` | Source of SQLi, XSS and upload activity |
| IP (victim web server) | `192.168.117.190` | Target of web attacks |
| File | `rev.php` | PHP reverse shell in `/prod/hackable/uploads/` |
| Port | `9001` | Reverse shell traffic (RFI scenario) |
| Process | `shell.exe` (PID 7396) | Malicious executable in memory dump |
| IP (C2) | `192.168.121.242` | Remote endpoint contacted by `shell.exe` |
| Port | `4444` | Reverse shell / C2 port |
| IP pair | `192.168.132.238 ⇄ 192.168.132.242` | Shell session in `external.pcapng` |

> Defang IOCs (for example `192[.]168[.]121[.]242`) before sharing outside a lab.

---

## 🧭 MITRE ATT&CK Mapping

*Suggested mapping, added by me after the course material.*

| Investigation | Tactic | Technique |
|---|---|---|
| SQLi / XSS | Initial Access | T1190: Exploit Public-Facing Application |
| RFI / Upload | Persistence | T1505.003: Server Software Component: Web Shell |
| RFI / Memory / Network | Command and Control | T1571: Non-Standard Port |
| Memory / Network | Command and Control | T1095: Non-Application Layer Protocol |
| Memory | Execution | T1059: Command and Scripting Interpreter |

---

## 💡 Key Takeaways

- **Access logs show the "what", not the "payload".** POST requests need PCAP correlation.
- **Status 200 doesn't mean "safe".** It only means the server processed the request.
- **Hunt by port.** In large captures, filtering by suspicious ports (4444, 9001) is faster than reading every packet.
- **No single artifact tells the whole story.** `CmdLine` and `strings` were clean, but `NetStat` exposed the C2 connection.
- **TCP flags tell a story:** `SYN/ACK` for session setup, `PSH` for command and data transfer.

### Detection / Hardening Ideas

- WAF rules for SQLi / XSS patterns and Wazuh alerts on repeated `/vulnerabilities/` hits.
- Restrict upload types, store uploads outside the web root, and disable script execution in upload directories.
- Alert on outbound connections to uncommon ports (4444, 9001) and on unsigned binaries making external connections.

---

## 📁 Repository Structure

```
.
├── README.md
├── 01-SQL-Injection/
├── 02-Cross-Site-Scripting/
├── 03-Remote-File-Inclusion/
├── 04-Memory-Dump-Analysis/
├── 05-External-Network-Communication/
└── assets/            # screenshots & mind maps
```

---

## 🙏 Acknowledgements

Lab scenarios and course material by **CyberWarFare Labs (CWL)**, Blue Team Fundamentals. All activity was performed in an isolated, intentionally vulnerable lab environment for educational purposes only.

## 📬 Connect

**[Your Name]** | SOC / Blue Team enthusiast
🔗 [LinkedIn](https://www.linkedin.com/in/your-profile) • 📧 your.email@example.com
