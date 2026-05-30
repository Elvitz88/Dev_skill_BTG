# Security Standards — Betagro
<!-- มาตรฐานความปลอดภัยของ Betagro -->

Read this for any security work, hardening, or pre-go-live check. Pair every fix with evidence for
the Evidence Pack.

## Web-server / TLS hardening (Security Req 11)
<!-- การ harden web server และ TLS -->

| Item   | Requirement                                                              | How to verify                          |
|--------|--------------------------------------------------------------------------|----------------------------------------|
| 11.1   | TLS ≥ 1.2, no weak cipher suites                                         | Qualys SSL Labs; `nmap --script ssl-enum-ciphers` |
| 11.2   | No self-signed certificate                                              | Cert issued by trusted CA              |
| 11.3   | CSP header enabled in web-server config                                 | Response header `Content-Security-Policy` |
| 11.4   | HSTS ≥ 12 months (31536000 s)                                           | Header `Strict-Transport-Security: max-age=31536000; includeSubDomains` |
| 11.5   | X-Content-Type-Options best practice                                    | Header `X-Content-Type-Options: nosniff` |
| 11.6   | Disable ports 80 / 8080, or IP-restrict if required                     | Firewall / ingress config              |

Target: **Mozilla Observatory grade A** and **SecurityScorecard rating A**.

### Example security headers (verify against the GT&D-approved boilerplate first)

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; ...   # tune per app, no 'unsafe-inline' if avoidable
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
```

## CORS

Start **fully closed**. Whitelist only the specific origins the frontend needs (per environment).
Never use `Access-Control-Allow-Origin: *` on authenticated or internal APIs.

<!-- เริ่มจากปิด CORS ทั้งหมด แล้ว whitelist เฉพาะ origin ที่จำเป็น -->

## Authorization (Security Req 2)

- Delete or change all default accounts/passwords before production.
- Enforce least privilege; when showing personal data, show only the data owner's information.
- Maintain an **authorization matrix** document.

## Input validation (Security Req 3)

- Reject/escape special characters (`<`, `@`, `!`, etc.) to prevent XSS and SQL injection.
- Validate type, value range, upper/lower bounds; prevent data loss/incompleteness; verify the value
  was actually stored.
- No technique that allows importing duplicate data.
- Do not store account/password/PII in cookies; set autocomplete `off` on sensitive fields.

## Password management (Security Req 4)

- Minimum 8 chars; mixed uppercase + lowercase + number + special symbol.
- Rotate every 180 days.
- Lockout for 5 minutes after 5 consecutive wrong attempts; counter resets on a correct entry.
- Forced change on first login or after admin reset.
- Masked entry; never "remember" passwords; **support MFA**.
- Deliver the admin password to Betagro once implementation is complete.

## Audit logging (Security Req 5)

Log: user identification, event type (transaction / abnormal / access logs), access time, all
admin/root activity, affected resource, and success/failure results.

- Retain logs **≥ 90 days**.
- Ship to a log server / external SIEM (Betagro DataDog for app logs).
- Betagro must have access to the logs.

## Other

- **Encryption / data masking** for confidential data (financial, PII) — Security Req 6.
- **Session timeout** configurable — Security Req 7.
- **Security Test / VAPT** before go-live referencing the latest OWASP; Betagro InfoSec runs it;
  vendor clears all discovered vulnerabilities — Security Req 8, 13.
- Works with the **Betagro WAF** on subdomains (test + prod) — Security Req 9.
- Close all **Critical / High / Medium** vulns before prod and throughout the contract — Security Req 10.
- **PDPA**: test data with personal data needs Betagro approval before import; comply with PDPA and
  integrate with Betagro's personal-data management system — Security Req 12, 13.
- Keep versions + **security patches** updated for the whole contract — Security Req 14.
- Define **application-interface security** (API security, certificate, encryption) — Security Req 15.
- Set server time to **Thailand standard time** — Security Req 16.

## Secure-code scanning

- Resolve **Checkmarx** findings within the sprint (Technical Req 11).
- Run `npm audit` and `npm audit --omit=dev`; fix reported issues.
