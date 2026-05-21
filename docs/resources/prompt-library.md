---
title: "Prompt Library"
description: "Reusable prompts สำหรับ Solution Architect / Software Engineer"
---

# Prompt Library 📚

ห้องเก็บ prompts สำเร็จรูปสำหรับ Solution Architect / Software Engineer ที่ใช้ Claude — copy ไปใช้แล้วปรับ

!!! tip "วิธีใช้"
    1. Copy prompt ที่ตรงงาน
    2. แทนที่ `[PLACEHOLDER]`
    3. ปรับ tone, language, depth ตามต้องการ
    4. Save ที่ดีๆ เป็น Claude.ai Project สำหรับใช้ซ้ำ

---

## 🏗️ Architecture & Design

### ADR Generator

```
คุณคือ Senior Solution Architect

เขียน Architecture Decision Record (ADR) ตาม Michael Nygard format:
1. Title
2. Status (Proposed | Accepted | Deprecated)
3. Context
4. Decision
5. Consequences (positive + negative)
6. Alternatives considered

Decision: [ระบุ]
Constraints: [budget, team skill, timeline]
Stakeholders: [คน/ทีม]

Output: Markdown + Mermaid diagram (ถ้าเหมาะ)
```

### System Design Critique

```
คุณคือ FAANG-level architect

วิเคราะห์ design ต่อไปนี้ [paste design]

Critique ตาม 4 มิติ:
1. **Scalability** — handle 10x, 100x load?
2. **Reliability** — SPOFs, failure modes, recovery
3. **Security** — threat model, attack surface
4. **Maintainability** — debug, evolve, team handoff

Output: ตาราง issues + severity + suggested fixes
```

### API Design Review

```
ตรวจสอบ API spec [paste OpenAPI / endpoints]

Checklist:
- RESTful conventions (verbs, nouns, status codes)
- Versioning strategy
- Pagination
- Filter/sort/search
- Auth & authz
- Rate limiting
- Idempotency (POST → use Idempotency-Key)
- Error format (RFC 7807?)
- Backwards compatibility
- Documentation (examples, edge cases)

Output: severity-sorted issue list + suggested改善
```

### Microservices Decomposition

```
ระบบ monolith ปัจจุบัน: [อธิบาย]
แสดง code/repo structure: [paste]

แบ่งเป็น microservices ตาม:
- Domain-Driven Design (bounded contexts)
- Independent deploy
- Data ownership
- Team Topologies

Output:
1. Service list + responsibilities
2. Data ownership matrix
3. Synchronous vs async communication
4. Mermaid diagram ของ service interaction
5. Migration strategy (Strangler Fig)
```

---

## ☁️ Cloud & DevOps

### Cloud Cost Optimization

```
AWS cost breakdown ปัจจุบัน: [paste bill summary]
Workload: [อธิบาย]

วิเคราะห์ + เสนอ optimizations เรียงตาม cost saving:
1. Right-sizing
2. Reserved Instances / Savings Plans
3. Spot instances ที่เหมาะกับ workload
4. S3 storage class transitions
5. Idle resources cleanup
6. Cross-service patterns (e.g., reduce data transfer)

Output: ตาราง finding + monthly saving estimate + risk
```

### Kubernetes Manifest Review

```
ตรวจสอบ K8s manifests: [paste YAML]

Audit:
- Resource requests/limits (sized appropriately?)
- Liveness/readiness probes
- HorizontalPodAutoscaler
- PodDisruptionBudget
- Security context (non-root, readOnly, capabilities)
- Network policies
- Secrets management (not hardcoded)
- Image pull policy + tags (no :latest)

Output: prioritized findings
```

### Terraform Refactor

```
[paste main.tf]

Refactor:
- ใช้ modules แยก concern
- ใช้ variables ที่เหมาะสม (no hardcode)
- ใช้ remote state + locking
- ตั้ง provider versions
- เพิ่ม output blocks
- ใช้ tagging convention

Output: รูปแบบ folder structure + refactored code snippets
```

### Incident Postmortem

```
Incident:
- Time: [start - end]
- Impact: [users affected, revenue, SLA breach]
- Detection: [how was it detected, how long until detected]
- Resolution: [steps taken]
- Root cause: [what failed]

เขียน blameless postmortem ตาม Google SRE format:
- Summary
- Timeline
- Root cause analysis (5 whys)
- What went well / poorly / luck
- Action items (with owners + dates)
- Lessons learned
```

---

## 💻 Coding

### Code Review

```
Review code ต่อไปนี้ [paste code]

ตรวจตาม:
1. **Correctness** — bugs, edge cases, race conditions
2. **Performance** — O(n²) loops, N+1 queries
3. **Security** — injection, auth, secrets, OWASP
4. **Readability** — naming, comments, function size
5. **Maintainability** — duplication, coupling, tests
6. **Style** — language conventions, lint

Output: severity-sorted comments with file:line refs + suggested fixes
```

### Refactor for Testability

```
Refactor function ต่อไปนี้ให้ test ง่ายขึ้น [paste code]

Apply:
- Dependency injection (no hardcoded deps)
- Pure functions where possible
- Separate side effects
- Single Responsibility
- Predictable interfaces

Output: refactored code + unit tests + before/after comparison
```

### Test Writing

```
Source code: [paste]

เขียน tests ที่ครอบคลุม:
1. Happy path (input ปกติ)
2. Edge cases (empty, null, boundary)
3. Error cases (invalid input, dependency failure)
4. Concurrency (ถ้าเกี่ยวข้อง)

Use [pytest/jest/junit] with mocking framework
Target coverage: 85%+
```

### Debug Help

```
Error: [paste stack trace + error message]

Relevant code:
[paste files]

Logs leading up to error:
[paste logs]

วิเคราะห์:
1. Root cause (ที่น่าจะใช่ที่สุด + alternative)
2. ขั้นตอนตรวจสอบเพิ่ม (commands, breakpoints)
3. Fix proposal (พร้อม code)
4. Prevention (test, lint, monitor)
```

---

## 📋 Documentation

### README Generator

```
Project: [name + 1-line description]
Repo files: [paste tree]
Key code: [paste main entry + config]

สร้าง README.md ที่มี:
- Project title + badges
- One-paragraph elevator pitch
- Features (bullet)
- Quick start (install, run, test)
- Architecture overview + mermaid diagram
- Configuration table
- API/Usage examples
- Contributing
- License

Tone: professional, concise
```

### Runbook

```
Service: [name]
Common alerts: [list]

เขียน runbook สำหรับ on-call:
สำหรับแต่ละ alert:
1. Symptoms (ที่จะเห็นใน dashboard)
2. Severity & SLA
3. Diagnosis steps (commands ที่รัน)
4. Resolution steps
5. Verification
6. Escalation path

Format: Markdown ที่ paste ลง Confluence/Notion ได้
```

### Migration Guide

```
ที่เก่า: [tech, version]
ที่ใหม่: [tech, version]
Breaking changes: [list]

เขียน migration guide:
1. Overview (what & why)
2. Prerequisites
3. Step-by-step (with code diff)
4. Common issues + troubleshooting
5. Rollback procedure
6. Verification checklist

Audience: senior engineer
```

---

## 🤖 AI & Claude-specific

### Prompt Refactor

```
Original prompt: [paste]

Issues you might find:
- คำสั่งคลุมเครือ
- ไม่มี output format spec
- ไม่มี examples
- ไม่ใช้ role/persona

Refactor ตาม CRISP framework + Anthropic best practices:
- Clear instruction
- Context
- Role
- Specifics (output format, length)
- Examples (few-shot)
- Polish format (XML tags)

Output: refactored prompt + explanation of changes
```

### Tool Schema Designer

```
Use case: AI agent ต้องทำ [task]

ออกแบบ tools:
- ระบุ tool name + description ที่ AI เข้าใจ
- input_schema (JSON Schema)
- Required fields
- Return format
- Error cases

Output: JSON Schema สำหรับ Anthropic Tool Use API
```

### Eval Dataset Builder

```
Feature: [describe AI feature]
Success criteria: [accuracy, tone, format, ...]

สร้าง eval dataset 30 cases ที่ cover:
- 60% happy path (typical inputs)
- 20% edge cases (boundary, ambiguous)
- 10% error cases (malformed, hostile)
- 10% creative / unexpected

Format: CSV — columns: input, expected_output, category, difficulty
```

---

## 🎯 Communication

### Executive Summary

```
Source: [paste long document / deck]

เขียน executive summary 200 คำสำหรับ C-level:
- Opening: business impact (number ที่สำคัญ)
- Problem & opportunity
- Recommendation (1-2 sentences)
- Expected outcome
- Asks (decision + funding)

Tone: confident, concise, business-first
ไม่ใช้: jargon เทคนิค, hedging language ("might", "perhaps")
```

### Slack Update

```
สถานการณ์: [project status]
อัปเดตล่าสุด: [accomplishments + blockers]

เขียน Slack update รายสัปดาห์:
- 🎯 Goal
- ✅ Done this week
- 🚧 In progress
- ⚠️ Blockers
- 📅 Next week

Format: ใช้ emoji, bullets สั้น, mention คนที่เกี่ยวข้อง
```

### Difficult Email

```
Recipient: [role/relationship]
Situation: [what happened]
What you need to say: [message]

เขียน email ที่:
- Diplomatic แต่ honest
- Constructive ไม่ใช่ blame
- มี action items ชัดเจน
- Tone professional

Generate 2 versions: formal และ slightly warmer
```

---

## 💡 Tips การใช้ Prompt Library

1. **Save เป็น Project** — Claude.ai → Project → ใส่ favorite prompts เป็น custom instructions
2. **Iterate** — ใช้แล้วปรับให้ตรงงาน อย่ารอ perfect
3. **Combine** — เริ่มจาก template แล้ว chain หลาย prompt
4. **Share with team** — เก็บใน wiki / GitHub repo
5. **Update regularly** — เพิ่ม prompt ใหม่ที่คุณค้นพบ

---

## 🔍 References

- 📘 [Anthropic Prompt Library](https://docs.claude.com/en/resources/prompt-library/) (Official)
- 📚 [Awesome ChatGPT Prompts (Claude-compatible)](https://github.com/f/awesome-chatgpt-prompts)
- 📺 [Anthropic — Prompt Engineering Course](https://github.com/anthropics/prompt-eng-interactive-tutorial)
