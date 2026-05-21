---
title: "Day 83: Capstone Design — Phase 2 (ADR + Risk + Cost)"
description: "Finalize ADRs, risk register, cost projection"
---

# Day 83: Capstone Design — Phase 2 ⚖️

<div class="lesson-meta">
⏱️ 3 ชั่วโมง &nbsp;|&nbsp; 📊 Project &nbsp;|&nbsp; 📋 Prerequisites: Day 82
</div>

## 🎯 Goal

ทำ design รอบสุดท้าย — ADRs ครบ + risk register + cost projection

---

## 1. Finalize ADRs

ขยาย ADR 8 อันจาก Day 82 ให้สมบูรณ์ + เพิ่มอีก 4:

- ADR-009: Caching strategy
- ADR-010: Multi-tenancy isolation
- ADR-011: PII handling & retention
- ADR-012: Disaster recovery plan

---

## 2. Risk Register

```markdown
# Risk Register

| ID | Risk | Likelihood | Impact | Score | Mitigation |
|----|------|-----------|--------|-------|------------|
| R1 | Prompt injection bypass guards | Medium | High | 12 | Layered defense (Day 78), red team CI |
| R2 | RAG accuracy < 75% | Medium | High | 12 | Hybrid retrieval, eval CI gate |
| R3 | Cost runaway | Medium | High | 12 | Budget alerts, rate limit, caching |
| R4 | Vendor outage | Low | High | 8 | Multi-region failover (Day 55) |
| R5 | PII leakage | Low | Critical | 16 | Output guard + audit + masking |
| R6 | Knowledge drift | High | Medium | 12 | Quarterly KB refresh + eval |
| R7 | Slow adoption | Medium | Medium | 9 | UX research + onboarding |
| R8 | Team skill gap | Medium | Medium | 9 | Training + pair programming |
```

Likelihood × Impact (1-4 each) = score. ≥ 12 → must mitigate before launch.

---

## 3. Cost Projection

```markdown
# Cost Projection — Year 1

## Assumptions
- 5,000 DAU (avg)
- 8 queries/user/day
- = 40K queries/day = 1.2M/month
- Avg input: 4K tokens (with RAG context)
- Avg output: 300 tokens

## Monthly Costs

### Inference (with model routing)
- 70% Haiku: 840K × (4K input + 300 output) = ~3.3B input + 252M output
  - $0.80/M × 3300 + $4/M × 252 = $2,640 + $1,008 = **$3,648**
- 25% Sonnet: 300K × ... = ~$3,600 + $1,350 = **$4,950**
- 5% Opus: 60K × ... = ~$3,600 + $2,250 = **$5,850**
- **Total inference: ~$14,448/mo**

Note: Verify current Anthropic pricing — numbers above are illustrative; actual rates vary.

### Cache savings
- Assume 35% cache hit rate
- Savings: ~$2,500/mo
- **Net inference: ~$12,000/mo**

### Infrastructure (AWS)
- ECS Fargate: $400
- RDS Postgres: $200
- Qdrant on EC2: $300
- Vector DB storage: $100
- S3: $50
- CloudWatch: $150
- ElastiCache: $150
- **Total infra: ~$1,350/mo**

### 3rd Party
- Cohere Rerank: $300
- Langfuse cloud: $200
- **Total: ~$500/mo**

## Grand Total
**~$13,850/mo = $166K/year**

## Per-user
- ~$2.77/user/month
- ~$0.09/user/day (acceptable)
```

---

## 4. Capacity Planning

```python
# Forecast over 12 months
GROWTH_RATE = 0.08  # 8% monthly

for month in range(1, 13):
    dau = 5000 * (1 + GROWTH_RATE) ** month
    queries = dau * 8 * 30
    cost = queries * 0.012  # $0.012/query avg
    print(f"M{month}: DAU={dau:,.0f}, Queries/mo={queries:,.0f}, Cost=${cost:,.0f}")
```

→ Plan ROI breakeven, scaling milestones

---

## 5. Security & Compliance Worksheet

```markdown
# Security & Compliance — Project X

## Threat Model
| Threat | Vector | Mitigation |
|--------|--------|-----------|
| Prompt injection | User input | Input filter + hardened prompt + guards |
| Data exfil | LLM output | Output PII filter + audit |
| Tool abuse | Agent | Tool permissions + approval workflow |
| Indirect injection | RAG corpus | Untrusted-content tagging |

## Privacy
- PII in inputs: detect + tokenize before LLM
- PII in outputs: filter before user
- Retention: 30 days (raw), aggregated forever
- Right to be forgotten: API endpoint per user

## Compliance
- PDPA Thailand: ✅ (data in-region, consent flow)
- GDPR (for EU users): ✅ (DPA, lawful basis, right to forget)
- SOC 2 Type II: Year 1 target
- HIPAA: not applicable
```

---

## 6. Operations Plan

```markdown
# Operations Plan

## Team
- AI Engineer × 2
- SRE × 1
- Data Engineer × 1 (RAG corpus)
- PM × 0.5
- Designer × 0.5

## On-call
- 24/7 with 4 engineers rotating weekly
- Backup escalation: Manager

## SLOs
[link to slos.md]

## Maintenance windows
- KB refresh: Sundays 2-4am
- Deploy: Tuesdays/Thursdays 14:00-15:00

## Postmortem
- Required for Sev1, Sev2
- Within 1 week
- Public-blameless template
```

---

## 7. Decision Gate

Before proceeding to build:

- [ ] PRD reviewed by stakeholders
- [ ] Architecture diagram approved
- [ ] ADRs ≥ 12 written
- [ ] Risk register approved (no score > 16 unmitigated)
- [ ] Cost projection approved by finance
- [ ] Compliance signed off
- [ ] Team capacity confirmed
- [ ] Timeline realistic

→ All checked → start building (Day 84-86)

---

## 🛠️ Deliverables (Day 83)

1. **12 ADRs** complete
2. **Risk register** (≥ 10 risks)
3. **Cost projection** (12 months)
4. **Security worksheet**
5. **Operations plan**
6. **Decision gate** signed

---

## ✅ Self-Check Quiz

<div class="quiz">

**Q1:** ทำไม Risk Register สำคัญก่อน build?

??? success "ดูคำตอบ"
    - บังคับให้ฉลาดก่อน start coding
    - Mitigation มี cost / time → ต้อง plan
    - Catches showstoppers ก่อนเสีย sprints
    - Compliance / leadership review needs

**Q2:** Cost-per-user metric ทำไมสำคัญ?

??? success "ดูคำตอบ"
    - Unit economics ของ business
    - Compare ROI vs ค่าคน
    - Predict burn rate ตามการเติบโต
    - Threshold for "free tier" decisions

</div>

[ต่อไป → Day 84 :material-arrow-right:](day-84.md){ .md-button .md-button--primary }
