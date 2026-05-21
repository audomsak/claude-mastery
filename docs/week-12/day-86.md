---
title: "Day 86: Capstone Build — UI + Auth"
description: "FastAPI + Next.js + SSO + RBAC"
---

# Day 86: Build — UI + Auth 🎨

<div class="lesson-meta">
⏱️ 4 ชั่วโมง &nbsp;|&nbsp; 📊 Project &nbsp;|&nbsp; 📋 Prerequisites: Day 84-85
</div>

## 🎯 Goal

Build:
- FastAPI backend
- Next.js chat UI (streaming)
- SSO authentication
- Role-based access control (RBAC)

---

## 1. FastAPI Backend

```python
# api/main.py
from fastapi import FastAPI, Depends, HTTPException
from fastapi.responses import StreamingResponse
from pydantic import BaseModel
import asyncio, json

app = FastAPI()

class ChatRequest(BaseModel):
    question: str
    session_id: str | None = None

@app.post("/api/chat")
async def chat(req: ChatRequest, user=Depends(get_current_user)):
    session_id = req.session_id or f"{user.id}-{int(time.time())}"
    
    # Audit log
    audit_log("chat_request", user_id=user.id, question=req.question)
    
    async def event_stream():
        async for event in stream_answer(req.question, user.id, session_id):
            yield f"data: {json.dumps(event)}\n\n"
        yield "data: [DONE]\n\n"
    
    return StreamingResponse(
        event_stream(),
        media_type="text/event-stream"
    )

@app.get("/api/sessions/{session_id}/history")
async def get_history(session_id: str, user=Depends(get_current_user)):
    config = {"configurable": {"thread_id": session_id}}
    state = await orchestrator.aget_state(config)
    
    # Authorization: session must belong to user
    if state.values.get("user_id") != user.id:
        raise HTTPException(403, "Forbidden")
    
    return state.values["messages"]
```

---

## 2. SSO with OAuth (e.g., Okta)

```python
# auth/oauth.py
from authlib.integrations.starlette_client import OAuth
from fastapi.security import OAuth2PasswordBearer

oauth = OAuth()
oauth.register(
    name="okta",
    client_id=OKTA_CLIENT_ID,
    client_secret=OKTA_CLIENT_SECRET,
    server_metadata_url="https://<org>.okta.com/.well-known/openid-configuration",
    client_kwargs={"scope": "openid profile email groups"}
)

@app.get("/login")
async def login(request: Request):
    return await oauth.okta.authorize_redirect(request, REDIRECT_URI)

@app.get("/callback")
async def callback(request: Request):
    token = await oauth.okta.authorize_access_token(request)
    user_info = token["userinfo"]
    
    # Create JWT for client
    jwt_token = create_jwt({
        "sub": user_info["sub"],
        "email": user_info["email"],
        "groups": user_info.get("groups", []),
    })
    
    response = RedirectResponse("/")
    response.set_cookie("session", jwt_token, httponly=True, secure=True)
    return response
```

---

## 3. JWT Verification

```python
from fastapi import Cookie
import jwt

def get_current_user(session: str = Cookie(None)):
    if not session:
        raise HTTPException(401, "Not authenticated")
    try:
        payload = jwt.decode(session, JWT_SECRET, algorithms=["HS256"])
        return User(
            id=payload["sub"],
            email=payload["email"],
            groups=payload.get("groups", [])
        )
    except jwt.PyJWTError:
        raise HTTPException(401, "Invalid token")
```

---

## 4. RBAC Decorator

```python
# auth/rbac.py
def require_role(role: str):
    def decorator(fn):
        async def wrapper(*args, user=Depends(get_current_user), **kwargs):
            if role not in user.groups:
                audit_log("rbac_deny", user_id=user.id, attempted=role)
                raise HTTPException(403, f"Requires role: {role}")
            return await fn(*args, user=user, **kwargs)
        return wrapper
    return decorator

@app.post("/api/admin/refresh-kb")
@require_role("admin")
async def refresh_kb(user=Depends(get_current_user)):
    # ...
    pass
```

---

## 5. Tenant Isolation (KB filter)

```python
def rag_step_with_tenant(state):
    user_dept = get_user_department(state["user_id"])
    
    # Filter retrieval by dept access
    retrieved = retriever.retrieve(
        state["question"],
        filters={
            "allowed_groups": {"any_of": [user_dept, "all"]}
        }
    )
    return {"retrieved": retrieved, ...}
```

→ User เห็นเฉพาะ docs ที่มี permission

---

## 6. Next.js Chat UI

```tsx
// app/chat/page.tsx
"use client";
import { useState, useRef } from "react";

export default function Chat() {
  const [messages, setMessages] = useState<Message[]>([]);
  const [input, setInput] = useState("");
  const sessionIdRef = useRef<string | null>(null);

  async function send() {
    const userMsg = { role: "user", content: input };
    setMessages(m => [...m, userMsg]);
    setInput("");
    
    const resp = await fetch("/api/chat", {
      method: "POST",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify({
        question: input,
        session_id: sessionIdRef.current
      })
    });
    
    const reader = resp.body!.getReader();
    const decoder = new TextDecoder();
    let assistantMsg = { role: "assistant", content: "", citations: [] };
    setMessages(m => [...m, assistantMsg]);
    
    while (true) {
      const { value, done } = await reader.read();
      if (done) break;
      const chunk = decoder.decode(value);
      const lines = chunk.split("\n\n");
      
      for (const line of lines) {
        if (!line.startsWith("data: ")) continue;
        const data = line.slice(6);
        if (data === "[DONE]") return;
        
        const event = JSON.parse(data);
        if (event.type === "text") {
          assistantMsg.content += event.content;
          setMessages(m => [...m.slice(0, -1), {...assistantMsg}]);
        } else if (event.type === "citations") {
          assistantMsg.citations = event.data;
          setMessages(m => [...m.slice(0, -1), {...assistantMsg}]);
        }
      }
    }
  }

  return (
    <div>
      {messages.map((m, i) => (
        <div key={i} className={m.role}>
          <p>{m.content}</p>
          {m.citations?.map(c => (
            <a key={c.id} href={c.url}>[{c.id}] {c.title}</a>
          ))}
        </div>
      ))}
      <input value={input} onChange={e => setInput(e.target.value)} 
             onKeyDown={e => e.key === "Enter" && send()} />
    </div>
  );
}
```

---

## 7. Audit Log

```python
# logging/audit.py
import structlog
from datetime import datetime

logger = structlog.get_logger()

def audit_log(action, **kwargs):
    logger.info("audit", 
        action=action,
        timestamp=datetime.utcnow().isoformat(),
        **kwargs
    )
```

→ Pipe to CloudWatch / Datadog / etc. — retain for compliance

---

## 8. Rate Limiting

```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=lambda req: req.state.user.id if hasattr(req.state, "user") else get_remote_address(req))

@app.post("/api/chat")
@limiter.limit("60/minute")
async def chat(...):
    pass
```

---

## 9. Health + Metrics Endpoints

```python
from prometheus_client import Counter, Histogram, generate_latest

REQUEST_COUNT = Counter("requests_total", "Total requests", ["endpoint", "status"])
LATENCY = Histogram("request_latency_seconds", "Latency", ["endpoint"])

@app.get("/health")
async def health():
    # Check deps: DB, vector DB, Claude API
    return {
        "status": "ok",
        "db": check_db(),
        "vector_db": check_qdrant(),
        "claude": check_claude(),
        "timestamp": datetime.utcnow().isoformat()
    }

@app.get("/metrics")
async def metrics():
    return Response(generate_latest(), media_type="text/plain")
```

---

## 🛠️ Day 86 Deliverables

- [ ] FastAPI backend with chat + history endpoints
- [ ] SSO login flow (Okta / Auth0 / Azure AD)
- [ ] RBAC enforcement
- [ ] Tenant isolation in RAG
- [ ] Streaming chat UI (Next.js)
- [ ] Audit log
- [ ] Rate limit
- [ ] Health + metrics endpoints
- [ ] Demo: complete user flow login → chat → see citations → logout

[ต่อไป → Day 87 :material-arrow-right:](day-87.md){ .md-button .md-button--primary }
