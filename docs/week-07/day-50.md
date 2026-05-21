---
title: "Day 50: Pydantic + Structured Outputs"
description: "Type-safe LLM responses — เลิกใช้ try/except parse JSON"
---

# Day 50: Pydantic + Structured Outputs 🛡️

<div class="lesson-meta">
⏱️ 3 ชั่วโมง &nbsp;|&nbsp; 📊 Intermediate &nbsp;|&nbsp; 📋 Prerequisites: Day 11 (API)
</div>

## 🎯 Learning Objectives

<ul class="objectives">
<li>เข้าใจว่าทำไม structured outputs สำคัญใน production</li>
<li>ใช้ Pydantic models กับ Claude</li>
<li>Validation + retry pattern</li>
<li>Migrate "JSON-in-prompt" → tool use + Pydantic</li>
</ul>

---

## 1. ปัญหา: LLM JSON ที่ไม่ valid

```python
prompt = "Return JSON: {name, age, email}"
resp = client.messages.create(...)
data = json.loads(resp.content[0].text)  # 💥 อาจ crash
```

ที่เจอบ่อย:
- Trailing comma
- Single quotes แทน double
- Markdown fence ใน response (` ```json `)
- Missing required field
- Wrong type (string แทน int)

---

## 2. Pydantic — Type Validation

```bash
pip install pydantic
```

```python
from pydantic import BaseModel, Field, EmailStr
from typing import Literal

class User(BaseModel):
    name: str
    age: int = Field(ge=0, le=150)
    email: EmailStr
    role: Literal["admin", "user", "guest"]

# Validate
try:
    user = User(name="Alice", age=30, email="alice@x.com", role="admin")
except ValidationError as e:
    print(e)
```

---

## 3. With Claude Tool Use (recommended way)

```python
from anthropic import Anthropic
import json

client = Anthropic()

# 1. Pydantic schema → JSON schema
user_schema = User.model_json_schema()

# 2. Pass as tool
tools = [{
    "name": "save_user",
    "description": "Save extracted user info",
    "input_schema": user_schema
}]

# 3. Force tool use
resp = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=500,
    tools=tools,
    tool_choice={"type": "tool", "name": "save_user"},  # force this tool
    messages=[{"role": "user", "content": "Extract user from: Alice, 30, alice@x.com, admin"}]
)

# 4. Validate
for block in resp.content:
    if block.type == "tool_use":
        user = User(**block.input)  # Pydantic validate
        print(user)
```

---

## 4. Helper Function — Clean Pattern

```python
def extract_structured(prompt: str, schema: type[BaseModel]) -> BaseModel:
    tools = [{
        "name": "respond",
        "description": f"Provide a structured {schema.__name__}",
        "input_schema": schema.model_json_schema()
    }]
    
    resp = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=2000,
        tools=tools,
        tool_choice={"type": "tool", "name": "respond"},
        messages=[{"role": "user", "content": prompt}]
    )
    
    for block in resp.content:
        if block.type == "tool_use":
            return schema(**block.input)
    raise ValueError("No tool use in response")

# ใช้
user = extract_structured("...", User)
```

---

## 5. Nested Models

```python
class Address(BaseModel):
    street: str
    city: str
    zip: str

class Employee(BaseModel):
    name: str
    title: str
    address: Address
    skills: list[str]
    reports_to: str | None = None

emp = extract_structured("""
John Smith is a Senior Engineer based at 123 Main St, NYC, 10001.
Skills: Python, AWS, K8s. He reports to Sarah Lee.
""", Employee)

print(emp.address.city)  # "NYC"
```

---

## 6. Validators

```python
from pydantic import field_validator

class Ticket(BaseModel):
    severity: Literal["low", "medium", "high", "critical"]
    summary: str = Field(min_length=10, max_length=200)
    tags: list[str]
    
    @field_validator("tags")
    @classmethod
    def lowercase_tags(cls, v):
        return [t.lower().strip() for t in v]
    
    @field_validator("summary")
    @classmethod
    def no_pii(cls, v):
        if "@" in v:  # crude email check
            raise ValueError("Summary should not contain emails")
        return v
```

---

## 7. Retry on Validation Failure

```python
def extract_with_retry(prompt: str, schema: type[BaseModel], max_retries=3):
    last_error = None
    for attempt in range(max_retries):
        try:
            return extract_structured(prompt, schema)
        except ValidationError as e:
            last_error = e
            # Add error to prompt for next attempt
            prompt = f"{prompt}\n\nPrevious attempt error: {e}\nFix and try again."
    raise last_error
```

---

## 8. Combine with Instructor (recommended library)

```bash
pip install instructor
```

```python
import instructor
from anthropic import Anthropic

client = instructor.from_anthropic(Anthropic())

user = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=500,
    response_model=User,  # Pydantic auto-handles
    messages=[{"role": "user", "content": "Extract: Alice, 30..."}]
)
```

Instructor handles tool use + retry + parsing internally — 1-liner

---

## 9. Use Cases ใน Enterprise

| Use case | Schema |
|---------|--------|
| Form extraction | Form fields → Pydantic |
| Email triage | severity, category, action |
| Ticket creation | summary, severity, tags, assignee |
| Data ETL | rows → typed records |
| API response gen | Match downstream API contract |

---

## 🛠️ Hands-on Exercise

!!! example "Exercise 1: Form Extractor"
    Design Pydantic model สำหรับ "Resume" → extract จาก paste text 5 ตัวอย่าง

!!! example "Exercise 2: Validation"
    เพิ่ม validators (e.g., phone format, date range) — ทดสอบกับ invalid input

!!! example "Exercise 3: Instructor"
    Migrate code Day 9 (Data Analysis) ให้ใช้ Instructor + Pydantic

---

## ✅ Self-Check Quiz

<div class="quiz">

**Q1:** ทำไม structured output > "JSON in prompt"?

??? success "ดูคำตอบ"
    - Tool use schema enforce structure ก่อน LLM generate
    - Pydantic validate ทันทีหลัง parse
    - Type-safe → IDE auto-complete, less bugs
    - Easier to test

**Q2:** เมื่อไหร่ใช้ Instructor vs raw tool use?

??? success "ดูคำตอบ"
    - Instructor: เร็ว, retry built-in, multi-provider support
    - Raw tool use: full control, no extra dependency

</div>

---

## 🔍 Cross-check & References

- 📘 [Pydantic docs](https://docs.pydantic.dev/)
- 📦 [Instructor library](https://github.com/jxnl/instructor)
- 📺 [Pydantic for LLM Workflows (DLAI)](https://www.deeplearning.ai/courses/pydantic-for-llm-workflows)
- 📺 [Getting Structured LLM Output (DLAI)](https://www.deeplearning.ai/courses/getting-structured-llm-output)

[ต่อไป → Day 51: Framework Decision :material-arrow-right:](day-51.md){ .md-button .md-button--primary }
