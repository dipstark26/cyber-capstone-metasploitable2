# Web Vulnerability Assessment of Metasploitable2

Capstone project (Milestone 7) for the **Cyber Security Foundation** program.
A full web vulnerability assessment of an intentionally vulnerable target, from lab setup to a professional security report.

> **Disclaimer:** All testing was done in an isolated lab on intentionally vulnerable machines that I own. Nothing here was tested against real or third-party systems. For educational purposes only.

**Author:** Dipak Sopan Kakade, B.E. Artificial Intelligence and Data Science
**Date:** 20 November 2026

📄 **Full report:** [`report/Capstone_Report_Dipak_Kakade.pdf`](report/Capstone_Report_Dipak_Kakade.pdf)

---

## Lab setup

```
 [Kali Linux 2026.3] ---- VMware virtual network ----> [Metasploitable2]
   192.168.68.129            192.168.68.0/24             192.168.68.130
```

| Machine | OS | IP | Role |
|---|---|---|---|
| Attacker | Kali Linux 2026.3 (VMware Workstation) | 192.168.68.129 | Scanner / analysis |
| Target | Metasploitable2 | 192.168.68.130 | Intentionally vulnerable web server |

## Methodology

1. **Network discovery:** ARP host discovery with `netdiscover`
2. **Enumeration:** crawl of the web server and its apps (DVWA, Mutillidae, TWiki, TikiWiki, phpMyAdmin)
3. **Automated vulnerability assessment:** full web scan with Acunetix
4. **Analysis:** findings grouped by severity, rated with CVSS, mapped to OWASP and MITRE ATT&CK
5. **Remediation plan:** prioritised fixes for each finding group

## Scan summary

| Metric | Value |
|---|---|
| Target web server | Apache/2.2.8 (Ubuntu) DAV/2, PHP |
| Locations crawled | 732 |
| Requests sent | 353,768 |
| Scan duration | 403 min (stopped manually at 99% because of the long run time) |
| Total findings | 559 |

| Severity | Count |
|---|---|
| High | 80 |
| Medium | 319 |
| Low | 82 |
| Informational | 78 |

Most frequent finding types: HTML form without CSRF protection (173), cross-site scripting (71), application error messages (55), directory listing (25), credentials sent in clear text (20).

## High-severity findings (80)

| Finding | Count | Example location |
|---|---|---|
| Cross-site scripting | 71 | TWiki, TikiWiki, phpMyAdmin setup, Mutillidae |
| Code execution | 2 | `/twiki/bin/viewfile/...` (`rev` parameter) |
| PHP-CGI remote code execution | 2 | `/cgi-bin/php`, `/dav/` |
| SQL injection | 1 | `/mutillidae/index.php` (`author` parameter) |
| File inclusion and directory traversal | 2 | `/mutillidae/index.php` (`page` parameter) |
| Config file source disclosure | 1 | `/dvwa/config/config.inc.php~` |
| MySQL/MariaDB `password.c` vulnerability | 1 | detected via `/phpinfo.php` |

### Evidence

| Finding | CWE | CVSS 3.0 | Result |
|---|---|---|---|
| SQL injection | CWE-89 | 10.0 | A quote in the `author` field caused a MySQL syntax error and exposed the SQL query, file path and table names |
| Code execution | CWE-94 | 9.1 | A harmless test string sent through `rev` was echoed back, showing input reaching a shell command (scanner reports it as *possible* execution) |

Screenshots are in [`screenshots/`](screenshots/).

## MITRE ATT&CK mapping

| Activity | Technique |
|---|---|
| Host discovery and web scanning | T1595.002 Active Scanning: Vulnerability Scanning |
| Server banner, phpinfo, path disclosure | T1592 Gather Victim Host Information |
| SQLi, file inclusion, XSS, code execution | T1190 Exploit Public-Facing Application |
| Clear-text credentials | T1040 Network Sniffing / T1557 Adversary-in-the-Middle |
| Login pages without lockout | T1110 Brute Force |

## Key recommendations

- **Critical:** remove or upgrade TWiki, fix PHP-CGI exposure, use parameterized queries, whitelist `page` values
- **High:** output encoding and a Content-Security-Policy for XSS, upgrade Apache and PHP, remove backup and config files, enable HTTPS
- **Medium:** disable directory listing, remove `phpinfo.php` and install/setup pages, add anti-CSRF tokens and login lockout
- **Low:** set `HttpOnly` and `Secure` cookie flags, add `X-Frame-Options`, disable the TRACE method

The fixes are recommendations only and were not applied or re-tested, since the target is a deliberately vulnerable lab machine.

## Limitations

- The scan was stopped manually at 99%, so a small part of the crawl may be incomplete
- Results come from an automated scanner and were not all manually verified, so some may be false positives
- Metasploitable2 is deliberately vulnerable, so the counts are far higher than a real system would show
- Testing covered web services only

## What I learned

Planning scan scope and time, reading scanner results as real weaknesses (SQL injection, XSS, code execution, misconfiguration), using CVSS to prioritise fixes, and writing a professional security report.

## Future work

- Re-run the assessment with **OWASP ZAP** and compare results with the first scan
- Manually verify selected findings in the lab and re-test after applying fixes
- Add an Nmap service scan for the full port and version list

## Tools

Kali Linux, VMware Workstation, Metasploitable2, netdiscover, Acunetix (web vulnerability scanner), MITRE ATT&CK, OWASP Top 10
