# Vulnerability Assessment & Penetration Testing — DVWA

**DVWA — Damn Vulnerable Web Application**

*Environment: Kali Linux Local Lab | Security Level: LOW*

| **Prepared By** | **Target** |
|---|---|
| Gulsan Prasad | DVWA Web Application |

| **Report Date** | **Testing Type** |
|---|---|
| May 2026 | Manual Black-Box VAPT |

**Severity Distribution:** Critical: 0 · High: 3 · Medium: 1 · Low: 0

---

## Table of Contents

1. [Executive Summary](#executive-summary)
2. [Summary of Vulnerabilities](#summary-of-vulnerabilities)
3. [Severity Distribution](#severity-distribution)
4. [Vulnerability Details](#vulnerability-details)
   - [V-01: SQL Injection \[HIGH\]](#v-01-sql-injection-high)
   - [V-02: Reflected XSS \[HIGH\]](#v-02-reflected-cross-site-scripting-xss-high)
   - [V-03: Stored XSS \[HIGH\]](#v-03-stored-cross-site-scripting-xss-high)
   - [V-04: IDOR \[MEDIUM\]](#v-04-insecure-direct-object-reference-idor-medium)
5. [Conclusion](#conclusion)
6. [Learning Outcomes](#learning-outcomes)

---

## Executive Summary

This report presents the findings from a Vulnerability Assessment and Penetration Testing (VAPT) exercise conducted on DVWA (Damn Vulnerable Web Application) hosted in a Kali Linux local environment. The objective was to identify and analyze common web application vulnerabilities, understand their real-world impact, and provide actionable remediation guidance.

The assessment was conducted manually through a web browser with DVWA security level set to **LOW**. The scope covered four vulnerability categories: SQL Injection, Reflected XSS, Stored XSS, and Insecure Direct Object Reference (IDOR).

## Summary of Vulnerabilities

| ID | Vulnerability | Severity | OWASP |
|----|---------------|----------|-------|
| V-01 | SQL Injection | HIGH | A03 — Injection |
| V-02 | Reflected Cross-Site Scripting | HIGH | A03 — Injection |
| V-03 | Stored Cross-Site Scripting | HIGH | A03 — Injection |
| V-04 | IDOR — Broken Access Control | MEDIUM | A01 — Broken Access Control |

## Severity Distribution

| Severity | Count | Vulnerabilities |
|----------|-------|------------------|
| 🔴 CRITICAL | 0 | — |
| 🟠 HIGH | 3 | SQL Injection, Reflected XSS, Stored XSS |
| 🟡 MEDIUM | 1 | IDOR |
| 🟢 LOW | 0 | — |

---

## Vulnerability Details

### V-01: SQL Injection [HIGH]

| OWASP | A03 — Injection | **Severity** | **HIGH** |
|---|---|---|---|

**Summary:** SQL Injection occurs when user input is directly embedded in SQL queries without proper sanitization. An attacker can manipulate query logic to retrieve, modify, or delete data from the database.

**Risk:** An attacker can bypass authentication, exfiltrate confidential records, modify or delete database content.

**Impact:** Successful exploitation compromises confidentiality, integrity, and availability of the entire application database.

**Reference:** https://owasp.org/Top10/A03_2021-Injection/

#### Proof of Concept (PoC)

1. Navigate to the SQL Injection module in DVWA.
2. Enter the payload `' OR '1'='1` in the User ID field and click Submit.
3. Observe that all user records are returned — authentication is bypassed.
4. Escalate with `' UNION SELECT user, password FROM users#` to dump credentials.

**Observation:** All 5 user records including admin credentials were exposed. The application returned hashed passwords for all accounts, confirming the injection was successful.

**Vulnerable URL:** `http://localhost/DVWA/vulnerabilities/sqli/`

**Remediation:**
- Use prepared statements and parameterized queries exclusively.
- Validate and whitelist all user-supplied inputs.
- Apply the principle of least privilege to database service accounts.
- Deploy a Web Application Firewall (WAF) for additional protection.
- Enable database-level audit logging to detect anomalous queries.

---

### V-02: Reflected Cross-Site Scripting (XSS) [HIGH]

| OWASP | A03 — Injection | **Severity** | **HIGH** |
|---|---|---|---|

**Summary:** Reflected XSS occurs when user input is immediately reflected back in the HTTP response without sanitization, causing the browser to execute injected JavaScript.

**Risk:** Attackers can steal session cookies, hijack user sessions, redirect victims to malicious sites, or perform actions on behalf of authenticated users.

**Impact:** Successful exploitation compromises user session security, browser integrity, and may lead to credential theft or account takeover.

**Reference:** https://owasp.org/Top10/A03_2021-Injection/

#### Proof of Concept (PoC)

1. Navigate to the XSS (Reflected) module in DVWA.
2. Enter the payload `<script>alert('XSS')</script>` in the name field.
3. Submit the form and observe the JavaScript alert popup.
4. Verify in the URL bar that the script is reflected in the query parameter.

**Observation:** A JavaScript alert popup appeared in the browser confirming the script executed. The injected payload was reflected verbatim in the server response without encoding.

**Vulnerable URL:** `http://localhost/DVWA/vulnerabilities/xss_r/`

**Remediation:**
- Encode all output before rendering HTML — use `htmlspecialchars()` or equivalent.
- Implement a strict Content Security Policy (CSP) to block inline scripts.
- Validate and whitelist input on the server side.
- Set the `HttpOnly` flag on session cookies to prevent script-based theft.

---

### V-03: Stored Cross-Site Scripting (XSS) [HIGH]

| OWASP | A03 — Injection | **Severity** | **HIGH** |
|---|---|---|---|

**Summary:** Stored XSS occurs when malicious scripts are persisted to the server (e.g., in a database) and executed each time a victim loads the affected page — no interaction with the attacker required.

**Risk:** Any user viewing the infected page will execute the payload. Attackers can mass-harvest sessions, perform drive-by attacks, or create persistent backdoors in the application.

**Impact:** Every visitor to the compromised page is affected. The impact is broader and more severe than Reflected XSS because the payload survives across sessions.

**Reference:** https://owasp.org/Top10/A03_2021-Injection/

#### Proof of Concept (PoC)

1. Navigate to the XSS (Stored) module — Guestbook in DVWA.
2. In the Message field, enter: `<script>alert('stored XSS')</script>`
3. Click "Sign Guestbook" to submit.
4. Reload the page — the alert fires automatically for every visitor.

**Observation:** The stored payload executed on every page load without any further attacker action, confirming persistent XSS. The guestbook stored the raw script tag in the database.

**Vulnerable URL:** `http://localhost/DVWA/vulnerabilities/xss_s/`

**Remediation:**
- Sanitize and strip dangerous HTML tags before storing user input.
- Use an allowlist-based HTML sanitization library (e.g., DOMPurify).
- Implement CSP to prevent execution of unauthorized scripts.
- Apply output encoding when rendering stored content.

---

### V-04: Insecure Direct Object Reference (IDOR) [MEDIUM]

| OWASP | A01 — Broken Access Control | **Severity** | **MEDIUM** |
|---|---|---|---|

**Summary:** IDOR vulnerabilities allow attackers to access resources belonging to other users by manipulating object identifiers (e.g., numeric IDs in URL parameters) without authorization checks.

**Risk:** Attackers can enumerate user records, access private data, or perform unauthorized operations by simply changing an ID value in the request.

**Impact:** Unauthorized data exposure, privacy violations, and potential for mass data exfiltration affecting all users of the application.

**Reference:** https://owasp.org/Top10/A01_2021-Broken_Access_Control/

#### Proof of Concept (PoC)

1. Identify a URL parameter controlling resource access, e.g. `?id=1`.
2. Intercept the request in Burp Suite Repeater.
3. Modify the parameter: change `id=1` to `id=2`, then `id=3`, etc.
4. Observe that a different user's data is returned without any authorization check.

**Observation:** The application returned data for User ID 2 while still authenticated as User ID 1. No authorization check was performed — the session cookie of User 1 was accepted for accessing resources of User 2.

**Vulnerable URL:** `http://localhost/DVWA/`

**Remediation:**
- Implement server-side authorization checks before serving any resource.
- Use indirect object references (e.g., GUIDs or tokens) instead of sequential integers.
- Verify that the authenticated user has permission to access the requested resource.
- Log and alert on abnormal sequential ID enumeration patterns.

---

## Conclusion

The VAPT assessment of DVWA successfully identified four critical and medium severity vulnerabilities: SQL Injection, Reflected XSS, Stored XSS, and Insecure Direct Object Reference. These vulnerabilities stem from a common root cause — the absence of server-side input validation, output encoding, and proper access control enforcement.

Immediate remediation is strongly recommended. Adoption of secure coding standards, OWASP Top 10 compliance, and regular security assessments will significantly reduce the attack surface of real-world web applications.

## Learning Outcomes

- Gained practical understanding of SQL Injection, XSS (Reflected & Stored), and IDOR vulnerabilities.
- Learned to use Burp Suite for intercepting, replaying, and manipulating HTTP requests.
- Understood how poor input validation and missing access control lead to real-world breaches.
- Developed skills in professional vulnerability reporting aligned with OWASP Top 10.
- Acquired knowledge of remediation techniques including parameterized queries, CSP, and output encoding.
- Improved understanding of how attackers enumerate and exploit web application weaknesses.

---

*Prepared by Gulsan Prasad | May 2026 | Kali Linux Lab*
