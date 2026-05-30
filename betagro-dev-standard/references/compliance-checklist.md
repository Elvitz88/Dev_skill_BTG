# Compliance Checklist — Betagro Comply Standard
<!-- รายการตรวจสอบมาตรฐาน เทียบกับ Checklist ของ Betagro -->

This file mirrors Betagro's `Comply_Standard_Checklist`. Use it to self-assess a project. For each
item, classify as:

- **Comply** — requirement fully covered, evidence exists.
- **Partial** — topic mentioned but missing detail (steps, tool, acceptance criteria, or evidence).
- **Gap** — not addressed; must add to spec + prepare evidence (Design + Test).

<!-- Comply = ครบ มีหลักฐาน / Partial = มีแต่ไม่ครบ / Gap = ยังไม่ทำ ต้องเพิ่ม -->

## How to run a gap analysis

1. Inventory what the repo/spec actually has (code, configs, tests, docs, pipeline).
2. Walk each requirement number below; assign Comply / Partial / Gap.
3. Output a table: `No. | Standard Detail | Status | What to do (TH)`.
4. For every Gap/Partial, give a concrete remediation: which file/config to add, which tool to run,
   what evidence to capture. Reference `evidence-pack.md` and `security-standards.md` for specifics.

Standard remediation note (Thai) for a Gap:
> เพิ่ม requirement นี้ใน Specification และเตรียมหลักฐาน (Design, Test) เพื่อให้ผ่านมาตรฐาน

For a Partial:
> ควรจัดทำรายละเอียดให้ครบ (ขั้นตอน, เครื่องมือ, Acceptance Criteria) พร้อมหลักฐานทดสอบ/เอกสารอ้างอิง

---

## A. Technical Requirements

1. Architecture (dev/deploy/data/security) presented & approved by GT&D before development; final
   delivery assessed against the approved architecture.
2. Use only GT&D-approved tools/technologies/libraries. New tool → written consent from Head of
   Application Development; Betagro accounts only.
3. Code quality & maintainability: accept Betagro code-review recommendations (clarity, readability,
   organization, maintainability, structure, general practices).
4. Performance: pages < 3 s, API < 200 ms (baseline net); PageSpeed 90–100 all metrics; Mozilla
   Observatory grade A; HSTS ≥ 12 months; SecurityScorecard rating A; good Qualys SSL Labs score.
5. Static assets served via CDN (Betagro provides). Team handles upload/update/invalidation/caching;
   cache public API responses where possible.
6. Load-test the full app and at API level; share report. Synthetic data generated programmatically;
   data volume agreed with Product Owner + Betagro tech team.
7. Hand back ALL assets (source, source maps, DB schema, server scripts, images/video, UI designs,
   OpenAPI spec). Retain no copies. Delete stale git branches (keep env-deploy + main).
8. Dev/QA/UAT apps not publicly accessible — BTG network OR BTG-only OR login on first screen.
9. Test cases must exist for important functionality and business logic.
10. Team-lead ensures manual testing before each sprint delivery and final delivery (UI, features,
    functionality, translations).
11. Resolve Checkmarx (secure-code scanning) findings within the sprint.
12. VAPT before go-live by a reputable certified auditor; share final report; consult Betagro Head
    of Security via Product Owner before starting.
13. Clear vulnerabilities discovered by Betagro Security Team's VAPT.
14. Document important workflows with screenshots and video.
15. Document local setup; automate it (very few manual commands); whole app runnable locally.
16. Responsive across listed resolutions (confirmed by Product Owner).
17. Browser/OS support: Windows 10, macOS Monterey+, Android 12+, iOS 15+; Firefox ESR, Chrome LTS,
    Safari; vendors Samsung/Apple; form factors laptop/mobile/tablet (ESR/LTS as of project start).
18. No unnecessary warnings/errors in build phase or logs (console logs, TS errors, outdated/deprecation).
19. Loading icons (>100 ms) + proper error/empty-state notifications.
20. Automatic failure recovery; if impossible, super-admin-only recovery API.
21. Logging & monitoring in Betagro DataDog; proactive alerting before users complain.
22. Multilingual: Thai + English in text AND image; Product Owner provides copy/images; team-lead
    ensures it's in scope.
23. ≥ 1 month post-go-live support for critical/high bugs.
24. Be available to explain to Betagro technical/business teams.
25. Involve Betagro Support Team 1–2 months before go-live; provide handover assets (troubleshooting
    guide, APIs, architecture).
26. Workshops/discussions in both English and Thai (bring translators as needed).
27. Build/deploy as code (IaC); no secrets in code.
28. High-write DB tables: avoid int keys → use bigint/uuid; plan archiving.
29. Meet the Definition of Done for release.
30. Horizontally scalable (multiple replicas behind a load balancer).
31. WCAG 2.1 Level A.
32. Maintenance page toggled by environment variable (if app has a UI).
33. Support data archiving to Betagro datalake.
34. Hypercare: 15 days post-go-live; restarts on each fix; closure only when no bugs reported in window.
35. Network/connection costs during install borne by vendor; enterprise-grade switches (Cisco/HP/
    Aruba or equivalent); ports support current + future expansion.
36. Tech & tools: Backend Node.js+TS, Frontend React+TS, DB Postgres, Deploy Docker/Kubernetes.

## B. Security Requirements (summary — full detail in security-standards.md)

1. Secure by Design / Privacy by Design, incl. secure source code.
2. Authorization: delete/change default accounts before prod; least-privilege; authorization matrix doc.
3. Input validation: block special chars (XSS, SQLi); type/range/bounds checks; no duplicate import;
   no sensitive autocomplete/cookie storage.
4. Password management: ≥ 8 chars, mixed case+number+symbol, rotate 180 days, lockout after 5 fails
   (5 min), forced change on first login, masked entry, no remembered passwords, MFA support.
5. Audit logging: user id, event type, timestamp, admin/root activity, results; retain ≥ 90 days;
   ship to log server / SIEM; Betagro has access.
6. Encryption / data masking for confidential data (financial, PII).
7. Session timeout configurable.
8. Security Test before go-live (OWASP latest); Betagro InfoSec runs it; vendor clears vulns.
9. Betagro subdomain works with Betagro WAF (test + prod).
10. Close Critical/High/Medium vulns before prod and throughout the contract.
11. Web-server hardening: TLS ≥ 1.2 + no weak ciphers; no self-signed cert; CSP enabled; HSTS ≥ 12mo;
    X-Content-Type-Options; disable ports 80/8080 (or IP-restrict).
12. PDPA-related test data needs Betagro approval before import.
13. PDPA compliance; integrate with Betagro personal-data management system.
14. Keep versions + security patches updated through the contract.
15. Define application-interface security (API security, certificate, encryption).
16. Set time to Thailand standard time.

## C. Technical Infrastructure Requirements

1. Deployable on Betagro cloud (IaaS or PaaS).
2. Scale-up (CPU/memory/storage) and scale-out supported.
3. Integrate with RDBMS or NoSQL.
4. Three separate, independent environments: Development, QAS, Production.
5. Recommend system sizing; each environment handles its workload.
6. Backup with full + partial recovery; defined recovery time from failure.
7. Maintenance page with custom message + scheduling by admin.
8. Log application config changes; admin can enable/disable logging.
9. Archive data to backup DB/storage; report from archive.
10. Support team able to resolve app + DB issues.
11. Condition-based alerts (e.g. abnormal batch job runtime).
12. Provide documentation of SQL statements run.

## D. Design Requirements (UX / WCAG)

1. Intuitive navigation, clear hierarchy (titles, headings, labels) — WCAG 2.4.2 & 2.4.6.
2. Minimize cognitive load; clear concise content.
3. Cohesive design system with standardized UI components.
4. Fast load 0–2 s with smooth interactions.

---

## Known gaps from prior assessment (history note)
<!-- ปัญหาที่เคยพบในการประเมินก่อนหน้า ใช้เป็นจุดเริ่มตรวจ -->

The most common Gaps/Partials seen on Betagro projects — check these first:

1. No CI/CD pipeline → set up Azure Pipelines (see `azure-pipeline.md`).
2. Environments not separated → split Dev/QA/UAT/Prod with isolated config + DB + domain.
3. No test/documentation evidence → run all 10 technical docs (see `evidence-pack.md`).
4. Knip not installed → install, scan unused files/packages, remove them.
5. CORS too open → close fully, whitelist required origins only.
6. Frontend not on GT&D boilerplate → migrate to the approved React build template.
7. `console.log` present → replace with Winston (levels, rotation, DataDog export).
8. Build/logging warnings → fix dependency/config; fail-on-warnings if possible.
9. No i18n → add i18next/formatjs with en/th files.
10. WCAG/a11y not done → add `eslint-plugin-jsx-a11y`, fix lint errors.
11. Dead code/comments → remove all (verify with Knip scan).
12. Single repo → split frontend/backend/device repos, each with README + pipeline.
