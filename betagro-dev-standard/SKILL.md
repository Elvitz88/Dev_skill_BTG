---
name: betagro-dev-standard
description: >-
  Betagro (เบทาโกร) software development standard for vendor/in-house teams building
  web and backend applications. Use this skill WHENEVER you are working inside a Betagro
  project repo, OR the user mentions Betagro, GT&D, AGRO project codes, "comply standard",
  "TOR requirement", "NFR evidence", "lab results examination", or asks whether code/architecture
  meets Betagro standards. Also trigger when the user asks to set up CI/CD, security headers,
  load tests, PageSpeed, Mozilla Observatory, Checkmarx, DataDog logging, i18n (Thai/English),
  WCAG a11y, environment separation (Dev/QAS/Prod), or to prepare deliverables/evidence for
  Betagro acceptance. This skill defines the organization scope so the AI agent steers the
  whole project in the same direction. Do NOT trigger for unrelated, non-Betagro projects.
license: Proprietary - internal Betagro use only
---

# Betagro Development Standard
<!-- มาตรฐานการพัฒนาซอฟต์แวร์ของ Betagro สำหรับทีม vendor และทีมภายใน -->

This skill encodes the Betagro Group Technology & Digital (GT&D) acceptance standard. When it is
active, treat **every coding, architecture, testing, security, and deployment decision** as bound
by these rules. The goal is that the application passes Betagro's "Comply Standard Checklist" and
ships a complete **NFR Evidence Pack** at delivery.

<!-- เป้าหมาย: ทำให้แอปผ่าน Checklist ของ Betagro และส่งมอบ Evidence Pack ครบถ้วน -->

## How to use this skill

This is a **steering skill**, not a one-shot generator. Keep its rules in mind across the whole
session. Concretely:

1. **At project start / architecture stage** → read `references/azure-pipeline.md` and confirm
   the environment + tech-stack choices below before scaffolding anything.
2. **When writing or reviewing code** → apply the "Always / Never" rules in this file and check
   `references/compliance-checklist.md` for the relevant requirement number.
3. **When the user asks "are we compliant?" / "what's missing?"** → run the gap analysis described
   in `references/compliance-checklist.md` and produce an Action Plan table.
4. **At delivery / pre-go-live** → assemble the deliverables listed in `references/evidence-pack.md`
   and run the security checks in `references/security-standards.md`.

Read a reference file only when the task touches it (progressive disclosure). Do not dump all
references at once.

<!-- อ่าน reference เฉพาะตอนที่งานเกี่ยวข้อง ไม่ต้องโหลดทั้งหมดพร้อมกัน -->

## Approved tech stack (mandatory unless GT&D writes otherwise)
<!-- Tech stack ที่อนุมัติแล้ว ห้ามเปลี่ยนโดยไม่ได้รับอนุญาตเป็นลายลักษณ์อักษรจาก GT&D -->

| Layer     | Technology                                  |
|-----------|---------------------------------------------|
| Backend   | Node.js + TypeScript                        |
| Frontend  | React + TypeScript (GT&D-approved boilerplate) |
| Database  | PostgreSQL                                  |
| Deploy    | Docker, Kubernetes                          |
| Pipeline  | Azure DevOps Pipelines → Argo CD (GitOps)   |
| Logging   | Winston → Betagro DataDog                   |
| i18n      | Thai + English (i18next / formatjs)         |

**Rule:** You must use ONLY tools/libraries on the GT&D-approved list. If a new tool is genuinely
needed, STOP and tell the user that written consent from the Betagro Head of Application Development
is required first — do not silently add it. Always use **Betagro accounts/licenses**, never personal
ones.

<!-- ห้ามใช้ tool/library นอกรายการที่อนุมัติ ถ้าจำเป็นต้องขออนุมัติเป็นลายลักษณ์อักษรก่อน -->

## Core "Always / Never" rules

These come straight from the TOR and apply to all code you write or review:

**Always**
- Separate `frontend`, `backend`, and `device` into different repos, each with its own README + pipeline.
- Separate environments: **Dev / QAS / Prod** with isolated config (`.env`, secrets store) and separate DB/domain.
- Keep build & deploy steps as code (Infrastructure as Code). No secrets in source.
- Use a structured logger (**Winston**) with log levels + rotation, shipping to DataDog.
- Make the app **multilingual (Thai + English)** for both text and images.
- Add loading indicators (>100ms actions) and proper error/empty-state messages.
- Provide automatic failure recovery; if impossible, expose a super-admin-only recovery API.
- Build for **horizontal scalability** (multiple replicas behind a load balancer).
- Use bigint/uuid (not plain int) for high-write tables; plan archiving to the Betagro datalake.
- Add a maintenance page toggled by an environment variable.

**Never**
- ❌ `console.log` in shipped code → use Winston instead.
- ❌ Leave build/lint/type warnings or deprecation warnings unresolved.
- ❌ Open CORS to `*` → start fully closed, then whitelist only required origins.
- ❌ Leave dev/qa/uat apps publicly accessible → require BTG network, BTG-only auth, or a login screen first.
- ❌ Keep unused files/packages/comments → run **Knip**, remove dead code, confirm with the scan.
- ❌ Use a self-signed certificate, weak cipher suites, or TLS < 1.2.
- ❌ Retain copies of source/assets after handover.

## Performance & quality targets (acceptance criteria)
<!-- เกณฑ์ที่ใช้ตรวจรับงาน -->

- Pages load within **3 seconds**; APIs respond within **200 ms** under baseline network.
- Google PageSpeed / Lighthouse: **90–100** on every metric (Performance, Accessibility, Best Practices, SEO).
- Mozilla Observatory (observatory.mozilla.org): grade **A** (HTTP, TLS, SSH tests).
- HSTS header in effect **≥ 12 months** (31536000 s).
- WCAG **2.1 Level A** accessibility (use `eslint-plugin-jsx-a11y`, fix all lint errors).
- Test cases must exist for important business logic; manual testing each sprint + final delivery.
- Checkmarx (and `npm audit`, `npm audit --omit=dev`) issues resolved within the sprint.
- VAPT / Security Test passed before go-live; all Critical/High/Medium vulns cleared.

## Reference files — read when relevant

- **`references/compliance-checklist.md`** — Full requirement list (Technical, Security, Infra,
  Design) with item numbers + how to self-assess Comply / Gap / Partial. Read this for any
  "are we compliant / what's missing" question.
- **`references/evidence-pack.md`** — The 10 technical documents and the full Evidence Pack to
  hand back at delivery, with the exact tool/command for each. Read this at delivery time.
- **`references/security-standards.md`** — Security headers, TLS, CSP, OWASP, password policy,
  audit logging, PDPA. Read this for any security or pre-go-live work.
- **`references/azure-pipeline.md`** — Azure DevOps pipeline + Argo CD GitOps + environment
  separation. Read this when setting up or reviewing CI/CD.

## Output discipline

When asked for a compliance check or evidence status, always answer with a **table** keyed by the
official requirement number (e.g. "Technical Req 4", "Security Req 11.4") and one of:
`Comply` / `Gap` / `Partial`, plus a short "what to do" note in Thai. This matches the format of
Betagro's `Comply_Standard_Checklist`.
