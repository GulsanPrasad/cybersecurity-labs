# crAPI API Security Testing

**crAPI Penetration Testing — OWASP API Security Top 10 Practical Exploitation**

*Target: crAPI (Completely Ridiculous API) | Environment: Local Docker Lab*

> ⚠️ **Scope:** All testing performed against a local, self-hosted crAPI Docker lab only — an intentionally vulnerable training application built by OWASP. Not tested against any live/production system.

## What Is crAPI?

crAPI is an intentionally vulnerable, microservices-based application built by OWASP that simulates a "car marketplace" platform — users, vehicles, dealerships, a community forum, and mechanic booking. It's the API-focused equivalent of DVWA, built to demonstrate real-world flaws mapped to the **OWASP API Security Top 10**.

## Lab Setup

```bash
git clone https://github.com/OWASP/crAPI.git
cd crAPI
docker-compose pull
docker-compose up -d
```

| Service | URL |
|---|---|
| Web App | `http://localhost:8888` |
| API Gateway | `/identity`, `/community`, `/workshop` |
| Mail Catcher (OTP) | `http://localhost:8888/mailhog` |

Two test accounts (attacker + victim) were registered to demonstrate authorization-bypass attacks like BOLA, and each account's JWT was captured via the login endpoint for use in subsequent requests. All attacks below were performed both via `curl` and by proxying traffic through **Burp Suite** (Proxy → Repeater/Intruder) for manual tampering and brute-forcing.

---

## Summary of Findings

| # | Vulnerability | OWASP API Top 10 | Impact |
|---|---------------|-------------------|--------|
| 1 | Broken Object Level Authorization (BOLA/IDOR) | API1:2023 | Leaks any user's live vehicle GPS location |
| 2 | Broken Authentication (OTP brute-force) | API2:2023 | Full account takeover |
| 3 | Excessive Data Exposure | API3:2023 | Leaks PII, VIN, internal mechanic notes |
| 4 | Mass Assignment → Broken Function Level Authorization | API3 + API5:2023 | Privilege escalation to admin |

---

## Attack 1: Broken Object Level Authorization (BOLA/IDOR) — API1:2023

**Module:** Vehicle / Location API (workshop service)
**Vulnerability:** Any authenticated user can fetch another user's live vehicle location just by supplying that vehicle's ID — no ownership check is performed.

**Steps:**
1. As the attacker, register a vehicle and note the response format for `GET /identity/api/v2/vehicle`.
2. Obtain the victim's `vehicleId` (leaked via a community post, support chat, or sequential-ID guessing).
3. Call the location endpoint using the **attacker's own token** but the **victim's vehicleId**:
   ```bash
   curl -s -X GET http://localhost:8888/workshop/api/vehicle/VICTIM_VEHICLE_ID/location \
     -H "Authorization: Bearer $ATTACKER_TOKEN"
   ```

**Result:** The victim's GPS coordinates were returned despite the request using only the attacker's token — confirming no ownership validation is performed. In Burp Suite, this was reproduced by sending the request to **Repeater** and swapping the `Authorization` header, and could be scaled via **Intruder** with a vehicle-ID wordlist to enumerate multiple victims at once.

---

## Attack 2: Broken Authentication — OTP Brute-Force — API2:2023

**Module:** Forgot Password flow (identity service)
**Vulnerability:** The password-reset OTP is only 4 digits, and no rate-limiting or lockout is enforced.

**Steps:**
1. Trigger a forgot-password request for the victim's email.
2. Brute-force the full 4-digit OTP space (0000–9999) against `POST /identity/api/auth/v3/check-otp`.
3. Once the valid OTP is found, submit a new password in the same request to take over the account.

**Result:** With no rate-limiting, the entire 10,000-combination OTP space could be exhausted within seconds to minutes, leading to full account takeover. The same attack was carried out in **Burp Intruder** by marking the OTP field as the payload position (`Numbers`, range 0–9999, zero-padded) and sorting results by response length to spot the valid OTP.

---

## Attack 3: Excessive Data Exposure — API3:2023

**Module:** Workshop / Mechanic report endpoint
**Vulnerability:** The mechanic report endpoint returns the entire internal object instead of only the fields the UI displays.

**Steps:**
1. Create a service request via `POST /workshop/api/merchant/contact_mechanic` to obtain a report ID.
2. Fetch the report: `GET /workshop/api/mechanic/report/{REPORT_ID}`.

**Result:** Instead of returning only a `status` field, the raw JSON response included the user's full name, email, vehicle VIN, and the mechanic's internal notes — none of which the frontend actually renders. Comparing the raw Burp response body against the rendered UI made the over-exposure obvious.

---

## Attack 4: Mass Assignment → Broken Function Level Authorization — API3 + API5:2023

**Module:** Community forum post API
**Vulnerability:** The post-creation endpoint binds request bodies without whitelisting fields, so hidden/internal fields like a role flag are accepted from client input.

**Steps:**
1. Create a normal post via `POST /community/api/v2/community/posts` and inspect the response for internal field names.
2. Re-send the request with an injected field: `{"title":"Test","content":"hello","author":{"role":"admin"}}`.
3. Confirm via `GET /identity/api/v2/user/dashboard` whether the attacker's role changed to `admin`.

**Result:** The backend blindly bound the injected `role` field, escalating the attacker's account to admin — potentially unlocking admin-only endpoints such as coupon generation.

---

## Burp Suite Workflow (used across all four attacks)

- **Proxy** — capture requests in HTTP History while browsing the app normally.
- **Repeater** — manually tamper with a single field (token, ID, hidden parameter) and resend.
- **Intruder** — brute-force or fuzz a value (OTP, sequential ID) across many requests automatically.
- Always compare the **raw request/response** against what the UI actually renders — that gap is where most API vulnerabilities surface.

## Mitigation Recommendations

- **BOLA:** Enforce server-side ownership checks on every object-access request.
- **Broken Authentication:** Increase OTP length; add rate-limiting, account lockout, and CAPTCHA.
- **Excessive Data Exposure:** Use response DTOs — serialize only the fields actually needed, never the full DB object.
- **Mass Assignment:** Bind request bodies via strict whitelist/schema validation; never accept sensitive fields (`role`, `is_admin`) from client input.
- **Overall:** Centralize authorization, logging, and anomaly detection at the API gateway level.

## Conclusion

These four attacks show that authentication alone doesn't make an API secure — proper authorization, data minimization, and input validation are equally critical. crAPI is one of the best resources for practicing these concepts safely and legally.

---

*All testing performed in an isolated local Docker lab for educational purposes, with explicit authorization scope limited to the self-hosted environment.*

Prepared by Gulsan Prasad | 2026 | Kali Linux Lab
