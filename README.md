# Cybersecurity  Labs

Hands-on offensive security lab write-ups covering VulnHub Boot2Root machines, DVWA web app testing, API security (crAPI), WordPress vulnerability assessment, and TryHackMe rooms — built and documented end-to-end.

**Author:** Gulsan Prasad
**Focus areas:** Penetration Testing · Web Application Security · API Security · CTF / Boot2Root · Vulnerability Assessment

---

## About this repository

Each folder below is a self-contained project with its own detailed write-up: recon steps, exploitation chain, exact commands used, and the final result. These are hands-on engagements performed in a personal lab environment, not passively copied tutorials.

## Projects

| # | Project | What it demonstrates | Tools / Stack |
|---|---------|----------------------|----------------|
| 1 | [NullByte: 1 Pentest](./projects/nullbyte-pentest) | Full VulnHub Boot2Root chain — recon, steganography, brute force, SQL injection, hash cracking, privilege escalation | nmap, nikto, gobuster, hydra, sqlmap, Kali Linux |
| 2 | [DVWA Security Testing](./projects/dvwa-security-testing) | Web application vulnerability testing across OWASP Top 10 categories | DVWA, Burp Suite, Kali Linux |
| 3 | [crAPI API Security](./projects/crapi-api-security) | API-specific vulnerability testing — broken auth, BOLA, injection in a modern API stack | crAPI, Postman, Burp Suite |
| 4 | [WordPress Vulnerability Testing](./projects/wordpress-vulnerability-testing) | CMS vulnerability assessment — plugin/theme flaws, enumeration, exploitation | WPScan, Metasploit, Kali Linux |
| 5 | [TryHackMe Web Security Labs](./projects/tryhackme-web-security-labs) | Guided and CTF-style web security rooms — practical exploitation exercises | TryHackMe, Burp Suite |

## Skills demonstrated across these labs

- **Reconnaissance & Enumeration:** nmap, nikto, gobuster, WPScan
- **Exploitation:** SQL injection, brute forcing, steganography, CMS/plugin exploits
- **Web App Security:** OWASP Top 10 testing on DVWA
- **API Security:** Authentication/authorization flaws, BOLA, injection on crAPI
- **Privilege Escalation:** SUID abuse, PATH hijacking
- **Tooling:** Kali Linux, Burp Suite, sqlmap, hydra, Metasploit, WPScan

---

*All labs were performed in isolated, self-hosted virtual lab environments for educational and skill-building purposes.*
