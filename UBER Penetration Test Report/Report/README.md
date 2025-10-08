# 🕵‍♂ UBER PENETRATION TEST REPORT
*Prepared by:* Mwangi CyberSecurity Company LTD.  
*Date:* July 29, 2025  
*Confidential*

---

## 🔒 CONFIDENTIALITY & DISCLAIMER

### Confidentiality Notice
This document contains privileged information exclusively for UBER personnel. Unauthorized sharing, duplication, or disclosure is prohibited.

### Disclaimer
Testing occurred within predefined scope/time boundaries. While thorough, this assessment cannot guarantee identification of all vulnerabilities.

### Contact
*Mwangi CyberSecurity Company LTD*  
📧 Email: lynnette.mwangi1@student.moringaschool.com  
📞 Phone: +254 712 345 6789  
🌐 Web: [www.mwangi-cyber.com](http://www.mwangi-cyber.com)

---

## 📘 TABLE OF CONTENTS
- [Confidentiality & Disclaimer](#-confidentiality--disclaimer)
- [Executive Summary](#-executive-summary)
- [Scope](#-scope)
- [Methodology](#-methodology)
- [Risk Ranking](#-risk-ranking)
- [Findings and Evidence](#-findings-and-evidence)
- [Recommendations for Uber Security Posture](#-recommendations-for-uber-security-posture)
- [Steps to Reproduce](#-steps-to-reproduce)
- [Conclusion](#-conclusion)
- [Bibliography](#-bibliography)

---

## 🧭 EXECUTIVE SUMMARY

Mwangi Cybersecurity performed a passive security evaluation of UBER's infrastructure from *July 24–27, 2025*.

### 🔍 Critical Observations
Recent analysis uncovered severe weaknesses risking data compromise, operational disruption, and brand erosion.  
Urgent remediation is required to maintain compliance and customer trust.

### 🧨 Primary Vulnerabilities

| Issue | Business Impact |
|-------|-----------------|
| *Deprecated Protocols* | Interception of credentials/payment data via SSLv2/v3/TLS 1.0. Attackers could exploit this to intercept sensitive data. |
| *Weak Encryption* | Brute-force decryption of confidential transmissions, enabling data theft. |
| *Invalid Certificates* | Browser trust warnings ("Connection not private"), expiring soon. |
| *Hostname Mismatch* | Certificates do not match domain name, enabling impersonation attacks. |

### ✅ Immediate Actions
- Disable legacy protocols; enforce TLS 1.2+  
- Upgrade encryption standards  
- Replace self-signed certificates with CA-issued trust chain  
- Implement automated certificate renewal  

---

## ⚠ BUSINESS IMPACT

| Risk Area | Potential Consequences |
|------------|------------------------|
| *Data Breaches* | Customer data theft → lawsuits, regulatory fines, revenue loss |
| *Service Disruption* | Browser warnings or expired certificates block users |
| *Compliance Violations* | PCI DSS non-compliance could lead to penalties |
| *Reputational Damage* | Loss of customer trust and competitive edge |

---

## 🎯 SCOPE

### Permitted Assets
- All uber.com / ubereats.com subdomains  
- Public APIs (developer.uber.com)  
- Mobile apps (static analysis only)

### Approved Techniques
- Passive OSINT (Recon-ng, DNS queries)  
- Public data review (GitHub, breach databases)  
- Non-intrusive checks (banner grabbing)

### Out of Scope
- Active scanning / brute-forcing  
- Social engineering  
- DoS attacks  
- Third-party services (AWS / Cloudflare)

*Authorized Testing Boundaries:* Uber’s HackerOne Program  
*Test Conducted By:* Lynette Mwangi

---

## 🧪 METHODOLOGY

### Methodology Framework
1. *PTES Phase 1:* Scope validation via HackerOne policy review  
2. *Information Gathering:* Google Dorking  
3. *Passive OSINT:* Subdomain enumeration, DNS analysis  
4. *Public Data Mining:* Breach DBs, GitHub, SSL certs  
5. *Passive Tooling:* Recon-ng (certificate_transparency, resolve)

### Tools Used
- *Recon-ng:* hackertarget, certificate_transparency, resolve
- *OWASP OSINT:* DNS mapping, crt.sh
- *DNS Recon:* dig, nslookup
- *Wappalyzer:* Tech stack analysis
- *GitHub Scraping & Breach Repositories*

### Constraints
- Zero active interaction with UBER systems  
- Ethical boundaries: public data only  

---

## 🧮 RISK RANKING

### Risk Rating Methodology

| Risk | Description | Time to Remediate |
|------|--------------|------------------|
| *Critical* | Full system compromise or data exposure | 24–48 hours |
| *High* | Privilege escalation or significant disruption | 5–8 business days |
| *Medium* | Limited access requiring conditions to exploit | 10–15 business days |
| *Low* | Minor or low-likelihood weaknesses | 30 days |
| *Informational* | Non-exploitable but informative issues | Regular review |

### Rating Matrix

| Likelihood | Impact | Severity | Example |
|-------------|---------|-----------|----------|
| High | High | Critical | Exposed admin panels |
| Medium | High | High | Unpatched CVEs |
| Low | High | Medium | Predictable email formats |

---

## 🕸 FINDINGS AND EVIDENCE

### Infrastructure Exposure
- 415 subdomains → Low risk  
- 481 public IPs → Low risk  
- DNS hosted on UltraDNS → Informational  

### Web Technology Stack (via Wappalyzer)
| Category | Technology |
|-----------|-------------|
| *Analytics* | Facebook Pixel, Google Analytics GA4, Hotjar, Microsoft Clarity, TikTok Pixel |
| *JS Frameworks* | React Router 6, React |
| *Security* | HSTS, reCAPTCHA, Cloudflare Bot Management |
| *CDN* | Cloudflare, Amazon S3 |
| *Advertising* | LinkedIn Ads, Microsoft Advertising |
| *Tag Managers* | Google Tag Manager, Tealium |
| *Libraries* | core-js, Hammer.js, web-vitals |
| *PaaS* | Amazon Web Services |
| *Reverse Proxy* | Envoy |
| *Authentication* | Google Sign-in |
| *Performance* | Priority Hints |
| *Customer Data* | Tealium |

### Infrastructure & Operations

| Finding | Evidence | Risk |
|----------|-----------|------|
| DNS hosted on UltraDNS | dig +short NS uber.com | Informational |
| MX: Google Workspace | dig +short mx uber.com | Informational |
| Employee emails in GitHub commits | Example screenshot | Medium |

---

### 🚨 Critical Findings
| Issue | Description | Severity |
|--------|--------------|-----------|
| Employee emails in breaches | GitHub commits | Critical |
| Public S3 bucket (uber-staging-backups) | Unrestricted access | Critical |
| Apache 2.4.41 vulnerabilities | CVE-2020-11984 | High |
| Historical breach data | 2016 incident | Medium |

---

## 🛡 RECOMMENDATIONS FOR UBER SECURITY POSTURE

### Critical (0–7 Days)
- Enforce *FIDO2 MFA* and password reset  
- Secure public S3 buckets (access logging, key rotation)

### High (30 Days)
- Disable BASIC auth → migrate to *OAuth 2.0*  
- Conduct vendor security audits  

### Medium/Low (90 Days)
- Server hardening (disable Apache signatures)  
- Subdomain monitoring (ChaosDB / OWASP Amass)  
- Implement API rate limiting and mandatory keys  

### Strategic Initiatives

| Initiative | Purpose | Timeline |
|-------------|----------|-----------|
| Dark web monitoring | Alert on credential leaks | Continuous |
| Bug bounty expansion | Include mobile & cloud | Q3 2025 |
| Third-party pentests | Annual audits | Annual |

---

## ⚙ STEPS TO REPRODUCE

### Subdomain Enumeration
modules load recon/domains-hosts/hacktarget
modules load recon/domains-hosts/certificate_transparency
module load recon/hosts-hosts/resolve
set SOURCE uber.com
run

---

##Tech Stack Analysis

###Performed via Wappalyzer browser extension.

- Certificate Inspection

- Manual review via browser and SSL tools.

- Vulnerability & Risk Scan

  ---

  ##🧾 CONCLUSION
### Posture Summary

- UBER maintains robust core security but faces third-party and credential-based risks.

### Strategic Recommendations

- Establish Vendor Risk Management team

- Implement dark web monitoring

- Expand bug bounty scope

- Retesting Proposal

- Follow-up validation post-remediation is recommended.

## Prepared by: Mwangi CyberSecurity Company LTD
Date: July 29, 2025

### 📚 BIBLIOGRAPHY

- OWASP Top Ten 2021 (A01/A05/A07)

- NIST SP 800-30 Risk Framework

- UBER HackerOne Program Scope

- PCI DSS v4.0 Compliance Guidelines

- IBM 2025 Data Breach Cost Report
