# Betagro Dev Standard — Claude Code Skill

> **Skill สำหรับทีม vendor และทีมภายใน Betagro** ที่ใช้ Claude Code (CLI / Desktop / VS Code)
> เพื่อให้ AI ช่วย code, review, และเตรียม evidence ได้ตามมาตรฐาน GT&D โดยอัตโนมัติ

---

## มันคืออะไร?

`betagro-dev-standard` คือ **Claude Code Skill** — ชุดคำสั่งและกฎที่ฝังเข้าไปใน Claude
เพื่อให้ทุกครั้งที่คุณทำงานในโปรเจกต์ของ Betagro Claude จะ:

- บังคับใช้ tech stack ที่ GT&D อนุมัติแล้ว (Node.js + TypeScript, React, PostgreSQL, Azure DevOps, Argo CD)
- เตือนทันทีเมื่อ code ละเมิด TOR requirement (CORS, console.log, secrets in source ฯลฯ)
- ช่วยตรวจ Comply Standard Checklist และบอกว่า Gap ตรงไหน
- เตรียม NFR Evidence Pack ก่อนส่งมอบงาน
- ดูแลเรื่อง Security (Checkmarx, VAPT, Mozilla Observatory, PageSpeed 90+)

คิดง่าย ๆ ว่า: **คุณ pair-program กับ Claude ที่อ่าน TOR ของ Betagro มาแล้วทั้งเล่ม**

---

## ติดตั้ง Skill เข้า Claude Code

### วิธีที่ 1 — ติดตั้งระดับ Project (แนะนำ)

ทำครั้งเดียวต่อ repo ทุกคนในทีมได้ใช้ทันทีหลัง clone

```bash
# อยู่ใน root ของ project repo
mkdir -p .claude/skills

# clone skill นี้เข้า project
git clone https://github.com/Elvitz88/Dev_skill_BTG.git .claude/skills/betagro-dev-standard

# หรือถ้าใช้ git submodule (แนะนำสำหรับ monorepo)
git submodule add https://github.com/Elvitz88/Dev_skill_BTG.git .claude/skills/betagro-dev-standard
```

จากนั้นสร้าง (หรือเพิ่มใน) `.claude/settings.json`:

```json
{
  "skills": [
    ".claude/skills/betagro-dev-standard/betagro-dev-standard"
  ]
}
```

### วิธีที่ 2 — ติดตั้งระดับ User (ใช้ได้ทุก project บนเครื่อง)

```bash
# Windows
git clone https://github.com/Elvitz88/Dev_skill_BTG.git "$env:USERPROFILE\.claude\skills\betagro-dev-standard"

# macOS / Linux
git clone https://github.com/Elvitz88/Dev_skill_BTG.git ~/.claude/skills/betagro-dev-standard
```

แก้ `~/.claude/settings.json` (สร้างถ้ายังไม่มี):

```json
{
  "skills": [
    "~/.claude/skills/betagro-dev-standard/betagro-dev-standard"
  ]
}
```

---

## วิธีใช้งานหลังติดตั้ง

Skill จะ **trigger อัตโนมัติ** เมื่อ Claude ตรวจพบ context ที่เกี่ยวข้อง เช่น:

| พูดว่า / ทำอะไร | Skill ช่วยอะไร |
|---|---|
| "ช่วย setup CI/CD pipeline" | ดึง `azure-pipeline.md` มาแนะนำ Azure DevOps + Argo CD |
| "comply standard ผ่านไหม?" | รัน gap analysis จาก checklist ตอบเป็นตาราง Comply/Gap/Partial |
| "เตรียม evidence pack" | แสดงรายการ 10 เอกสารที่ต้องส่ง พร้อม command เก็บ evidence |
| "review security" | ตรวจ OWASP, security headers, TLS, CSP ตาม `security-standards.md` |
| เขียน code มี `console.log` | Claude เตือนและแนะนำให้ใช้ Winston แทน |
| เพิ่ม library ใหม่ | Claude แจ้งว่าต้องขออนุมัติ GT&D ก่อน |

### เรียกใช้ด้วยตัวเอง (Manual trigger)

```
/betagro-dev-standard
```

หรือพิมพ์ใน chat:

```
ช่วย comply standard check สำหรับโปรเจกต์นี้หน่อย
```

---

## โครงสร้างไฟล์ใน Skill

```
betagro-dev-standard/
├── SKILL.md                        # ตัว Skill หลัก (Claude อ่านไฟล์นี้)
├── assets/
│   └── azure-pipelines-template.yml  # Template pipeline พร้อมใช้
└── references/
    ├── compliance-checklist.md     # รายการ requirement ทั้งหมด + วิธี self-assess
    ├── evidence-pack.md            # รายการ 10 เอกสารส่งมอบ + command เก็บ evidence
    ├── security-standards.md       # Security headers, TLS, CSP, OWASP, PDPA
    └── azure-pipeline.md           # Azure DevOps + Argo CD GitOps guide
```

---

## ตัวอย่างการใช้งาน

**ถามเรื่อง compliance:**
```
ตอนนี้โปรเจกต์เรา comply standard ของ Betagro ไหม? ขาดอะไรบ้าง?
```

**ขอ setup pipeline:**
```
ช่วย setup Azure DevOps pipeline ให้ตามมาตรฐาน Betagro หน่อย
environment มี Dev, QAS, Prod
```

**ตรวจ security ก่อน go-live:**
```
ช่วยตรวจ security checklist ก่อน go-live ตามมาตรฐาน Betagro
```

**เตรียมส่งมอบงาน:**
```
ช่วยสรุปว่า evidence pack ที่ต้องส่งมี อะไรบ้าง และยังขาดอะไร
```

---

## ข้อกำหนด

- ใช้ได้เฉพาะโปรเจกต์ **Betagro Group** เท่านั้น
- License: Proprietary — Internal Betagro use only
- ดูแลโดย GT&D (Group Technology & Digital)

---

## อัปเดต Skill

```bash
# Project-level
cd .claude/skills/betagro-dev-standard && git pull

# User-level (Windows)
cd "$env:USERPROFILE\.claude\skills\betagro-dev-standard" && git pull

# User-level (macOS/Linux)
cd ~/.claude/skills/betagro-dev-standard && git pull
```
