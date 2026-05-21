# เตรียมตัวก่อนเริ่ม (Setup)

ก่อนเริ่ม Day 1 ต้องเตรียม **บัญชี** และ **เครื่องมือ** ตามนี้ ใช้เวลา ~30 นาที

---

## 1. บัญชีที่ต้องสมัคร

### :material-account-plus: 1.1 Claude.ai (ต้องมี)

จำเป็นสำหรับทุกสัปดาห์

1. ไปที่ [https://claude.ai](https://claude.ai)
2. คลิก **Sign up** → ใช้ Google หรือ Email ก็ได้
3. ยืนยันอีเมล

!!! tip "Free vs Pro vs Max"

    - **Free** — พอเรียน Week 1 ได้ แต่มี usage limit
    - **Pro ($20/เดือน)** — แนะนำสำหรับ Week 2–4 (มี Claude Code, Cowork, รุ่น Opus)
    - **Max** — สำหรับการใช้งานหนัก (ทางเลือก)

### :material-key: 1.2 Anthropic Console + API Key (ต้องมีตั้งแต่ Week 2)

1. ไปที่ [https://console.anthropic.com](https://console.anthropic.com)
2. Sign up (ใช้บัญชี Claude.ai ได้)
3. ไปที่ **Settings → API Keys → Create Key**
4. **คัดลอก key เก็บไว้ในที่ปลอดภัย** (จะแสดงครั้งเดียวเท่านั้น)
5. เติม credit ขั้นต่ำ $5 (Anthropic มักให้ free credit ตอนสมัครใหม่)

!!! warning "เก็บ API key ให้ปลอดภัย"

    - อย่า commit ลง Git
    - ใช้ `.env` file หรือ environment variable เสมอ
    - ถ้ารั่ว ให้ revoke ทันทีและสร้างใหม่

### :material-github: 1.3 GitHub (ต้องมี)

1. สมัครที่ [https://github.com](https://github.com)
2. ตั้งค่า SSH key หรือใช้ Personal Access Token (จะใช้ใน Week 3+)

---

## 2. ซอฟต์แวร์ที่ต้องติดตั้ง

### :material-language-python: 2.1 Python 3.12+

=== "macOS"

    ```bash
    # ใช้ Homebrew
    brew install python@3.12

    # ตรวจสอบ
    python3 --version  # ต้อง >= 3.12
    ```

=== "Windows"

    1. ดาวน์โหลดจาก [https://python.org](https://python.org)
    2. ตอนติดตั้ง ✅ Add Python to PATH
    3. เปิด PowerShell:
    ```powershell
    python --version
    ```

=== "Linux (Ubuntu/Debian)"

    ```bash
    sudo apt update
    sudo apt install python3.12 python3.12-venv python3-pip
    python3 --version
    ```

### :material-nodejs: 2.2 Node.js 20+ (สำหรับ Week 2–4)

=== "macOS / Linux"

    ```bash
    # ใช้ nvm (แนะนำ)
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
    nvm install 20
    nvm use 20
    node --version
    ```

=== "Windows"

    ดาวน์โหลด installer จาก [https://nodejs.org](https://nodejs.org) (LTS version)

### :material-microsoft-visual-studio-code: 2.3 Visual Studio Code

ดาวน์โหลดจาก [https://code.visualstudio.com](https://code.visualstudio.com)

**Extensions ที่แนะนำ:**

- Python (Microsoft)
- Pylance
- ESLint
- Markdown All in One
- GitLens

### :material-git: 2.4 Git

=== "macOS"

    ```bash
    brew install git
    git --version
    ```

=== "Windows"

    ดาวน์โหลดจาก [https://git-scm.com](https://git-scm.com)

=== "Linux"

    ```bash
    sudo apt install git
    ```

### :material-console: 2.5 Claude Code (สำหรับ Week 3+)

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

> ติดตั้งครั้งแรกจะถาม login — ทำตอน Day 15 ก็ได้

---

## 3. ตรวจสอบว่าพร้อมหรือยัง

รันคำสั่งทั้งหมดนี้ — ทุกตัวต้องไม่ error

```bash
python --version       # >= 3.12
node --version         # >= 20
npm --version          # >= 10
git --version          # >= 2.40
code --version         # VS Code installed
```

---

## 4. สร้าง Workspace สำหรับคอร์ส

```bash
# สร้างโฟลเดอร์เก็บทุกอย่างของคอร์ส
mkdir -p ~/claude-mastery-workspace
cd ~/claude-mastery-workspace

# สร้าง virtual environment สำหรับ Python projects
python -m venv .venv

# Activate
# macOS/Linux
source .venv/bin/activate
# Windows PowerShell
# .venv\Scripts\Activate.ps1

# ติดตั้ง Anthropic SDK
pip install anthropic python-dotenv

# ติดตั้ง Node SDK (ทางเลือก)
npm init -y
npm install @anthropic-ai/sdk dotenv
```

สร้างไฟล์ `.env` (อย่า commit!):

```bash
ANTHROPIC_API_KEY=sk-ant-api03-xxxxxxxxxx
```

และ `.gitignore`:

```
.env
.venv/
node_modules/
__pycache__/
```

---

## 5. ทดสอบ API Key

ลองรันสคริปต์นี้ (`test_api.py`):

```python title="test_api.py"
import os
from dotenv import load_dotenv
from anthropic import Anthropic

load_dotenv()
client = Anthropic()  # auto-read ANTHROPIC_API_KEY

response = client.messages.create(
    model="claude-haiku-4-5-20251001",  # ใช้รุ่นถูกที่สุดเพื่อเทส
    max_tokens=100,
    messages=[{"role": "user", "content": "สวัสดี! บอกชื่อตัวเองหน่อย"}]
)

print(response.content[0].text)
```

```bash
python test_api.py
```

ถ้าได้ผลลัพธ์ภาษาไทยมา = พร้อมเรียน! 🎉

---

## 6. Troubleshooting

??? failure "ERROR: ImportError: No module named 'anthropic'"

    Virtual environment ไม่ได้ activate
    ```bash
    source .venv/bin/activate  # macOS/Linux
    ```

??? failure "ERROR: AuthenticationError: invalid x-api-key"

    1. ตรวจว่า `.env` มี key ที่ถูกต้อง
    2. ตรวจว่า key ยัง active อยู่ที่ console
    3. ลอง create key ใหม่

??? failure "ERROR: RateLimitError"

    เติม credit ใน [Anthropic Console → Plans & Billing](https://console.anthropic.com/settings/plans)

---

## พร้อมแล้ว!

[เริ่ม Day 1: AI และ LLM คืออะไร :material-arrow-right:](week-01/day-01.md){ .md-button .md-button--primary }
