# Evidence Pack — Deliverables for Betagro Acceptance
<!-- ชุดหลักฐานที่ต้องส่งมอบเพื่อให้ผ่านการตรวจรับ -->

At delivery / pre-go-live, assemble the **NFR Evidence Pack**. Each item must be a real file or
link, captured from an actual run — not a description. Save everything under `./evidence/` in the
repo (or attach to the delivery ticket).

<!-- ทุกชิ้นต้องเป็นไฟล์/ลิงก์จริงจากการรันจริง ไม่ใช่แค่คำอธิบาย เก็บไว้ใน ./evidence/ -->

## The 10 Technical Documents

These are the core technical deliverables. Run each, capture the report, and link it.

| # | Document            | Tool / command                                          | Evidence to capture                          |
|---|---------------------|---------------------------------------------------------|----------------------------------------------|
| 1 | API Document        | OpenAPI/Swagger spec generated from code                | `openapi.json` + Swagger UI link             |
| 2 | Google PageSpeed    | Lighthouse / PageSpeed Insights on key pages            | Score report (≥ 90–100 all metrics) screenshot/JSON |
| 3 | Mozilla Observatory | `npx @mdn/mdn-http-observatory <domain>`                | Grade A report output                        |
| 4 | Load Test           | JMeter (or k6) at app + API level                       | Load report + synthetic data description     |
| 5 | Unit Test           | Jest / Vitest with coverage                             | Coverage report; tests for business logic    |
| 6 | Automate Test (E2E) | webdriver.io + BrowserStack (or Playwright)             | E2E run report across target browsers        |
| 7 | Checkmarx / audit   | Checkmarx scan + `npm audit`, `npm audit --omit=dev`    | Scan results + remediation commits           |
| 8 | DataDog             | Winston → DataDog instrumentation                       | Links to dashboards, logs, alerts            |
| 9 | Data Structure      | ERD / schema doc (PostgreSQL)                           | ERD image or document                        |
| 10| User Manual         | Short doc: usage / setup / permissions                  | User manual (TH + EN)                         |

### Command reference

```bash
# 3. Mozilla Observatory — target grade A
npx @mdn/mdn-http-observatory <domain>

# 7. Dependency audit
npm audit
npm audit --omit=dev

# 4. Knip — unused files/exports/dependencies (run before delivery)
npx knip
```

## Full Evidence Pack checklist
<!-- รายการ Evidence Pack ทั้งหมดที่ต้องส่งกลับ -->

- [ ] Pipeline YAML (`azure-pipelines.yml`) + screenshot of a run passing every stage.
- [ ] Environment / domain / DB diagram + how secrets are separated.
- [ ] Reports: PageSpeed, Mozilla Observatory, JMeter (load), Unit/E2E, `npm audit`.
- [ ] DataDog links (dashboards, logs, alerts).
- [ ] API Documentation (Swagger / OpenAPI JSON + UI).
- [ ] Data structure / ERD (image or document).
- [ ] User Manual (short: usage / install / permissions), Thai + English.
- [ ] Knip scan result + list of removed files/packages.
- [ ] a11y config (`eslint-plugin-jsx-a11y`) + passing lint result.
- [ ] Commit(s) removing `console.log` and switching to Winston.
- [ ] VAPT / Security Test report with all Critical/High/Medium cleared.
- [ ] Local-setup doc (automated, runnable on a clean machine).
- [ ] Handover assets for Betagro Support Team (troubleshooting guide, APIs, architecture).

## Suggested delivery cadence
<!-- จังหวะการส่งมอบที่แนะนำ -->

A useful rhythm for the final sprint, ending with the first UAT deployment:

- Set up pipeline + environment separation first (blocks everything else).
- Then generate the per-tool reports (PageSpeed, Observatory, load, unit/E2E, audit).
- Wire DataDog logging + alerts.
- Produce docs (API, ERD, user manual, local setup).
- Finish with the security test, then assemble everything into the Evidence Pack and do the first
  UAT deployment.

## Output format

When the user asks "what evidence is ready / missing", reply with the Evidence Pack checklist above
as a table: `Item | Status (Ready/Missing) | Location or link | Note (TH)`.
