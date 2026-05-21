---
title: "Day 88: Capstone — Observability + Eval Pipeline"
description: "Connect Langfuse, build eval CI gate, dashboards"
---

# Day 88: Observability + Eval Pipeline 📊

<div class="lesson-meta">
⏱️ 4 ชั่วโมง &nbsp;|&nbsp; 📊 Project &nbsp;|&nbsp; 📋 Prerequisites: Day 75-76, 87
</div>

## 🎯 Goal

Wire up complete LLMOps pipeline:
- Langfuse tracing
- Eval CI gate
- Dashboard
- Alerts

---

## 1. Langfuse Integration

```python
# orchestrator/observability.py
from langfuse import Langfuse
from langfuse.langchain import CallbackHandler

langfuse = Langfuse(
    host=LANGFUSE_HOST,
    public_key=LANGFUSE_PK,
    secret_key=LANGFUSE_SK
)

def get_handler(user_id, session_id):
    return CallbackHandler(
        user_id=user_id,
        session_id=session_id,
        tags=["capstone"]
    )

# In API
@app.post("/api/chat")
async def chat(req, user=Depends(get_current_user)):
    handler = get_handler(user.id, req.session_id)
    config = {
        "configurable": {"thread_id": req.session_id},
        "callbacks": [handler]
    }
    ...
```

---

## 2. Custom Spans

```python
@langfuse.observe(name="retrieve_hybrid")
def hybrid_retrieve(query):
    langfuse.update_current_observation(
        input={"query": query},
        metadata={"strategy": "vector+bm25+rerank"}
    )
    results = retriever.retrieve(query)
    langfuse.update_current_observation(
        output={"num_results": len(results), "top_sources": [r["metadata"]["doc_source"] for r in results[:3]]}
    )
    return results
```

---

## 3. Quality Scoring

```python
# Async eval after response
async def score_response(trace_id, question, answer, sources):
    # LLM-as-judge
    score_data = await llm_judge(question, answer, sources)
    
    langfuse.score(
        trace_id=trace_id,
        name="llm_judge_quality",
        value=score_data["score"] / 10,  # 0-1
        comment=score_data["reasoning"]
    )
    
    # User feedback (when thumbs up/down clicked)
    # → endpoint /api/feedback that calls langfuse.score()
```

---

## 4. Production Eval Sample

```python
# Sample 5% of production requests for offline eval
import random

async def maybe_eval(request_data):
    if random.random() < 0.05:
        # Queue for async eval (don't block user)
        await eval_queue.put(request_data)

# Worker
async def eval_worker():
    while True:
        data = await eval_queue.get()
        result = await full_eval(data)
        store_eval_result(result)
```

---

## 5. CI Eval Gate

```yaml
# .github/workflows/llm-eval.yml
name: LLM Eval
on:
  pull_request:
    paths: ['orchestrator/**', 'retrieve/**', 'prompts/**']

jobs:
  eval:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: '3.12' }
      
      - run: pip install -r requirements.txt
      
      - name: Run regression eval
        run: python eval/regression.py --test-set=eval/golden.json --threshold=0.85
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          QDRANT_URL: ${{ secrets.QDRANT_TEST_URL }}
      
      - name: Cost check
        run: python eval/cost_check.py --max-cost-per-100=2.50
      
      - name: Post results to PR
        uses: actions/github-script@v7
        with:
          script: |
            const results = require('./eval/results.json');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: `## Eval Results\n- Accuracy: ${results.accuracy}\n- Cost/100 queries: $${results.cost}`
            });
```

---

## 6. Golden Test Set

```json
[
  {
    "id": "qa-001",
    "question": "What's our PTO policy for new employees?",
    "expected_keywords": ["15 days", "pro-rated", "first year"],
    "expected_doc_id": "hr-pto-policy-2024",
    "category": "QA",
    "difficulty": "easy"
  },
  {
    "id": "action-001",
    "question": "Schedule 30-min meeting with bob@x.com tomorrow at 2pm",
    "expected_tools": ["schedule_meeting"],
    "expected_inputs": {"attendees": ["bob@x.com"], "duration_min": 30},
    "category": "ACTION",
    "difficulty": "medium"
  },
  {
    "id": "hybrid-001",
    "question": "Find expense policy and create JIRA ticket to update",
    "expected_tools": ["create_jira_ticket"],
    "expected_keywords": ["expense"],
    "category": "HYBRID",
    "difficulty": "hard"
  }
  // ... 100+ cases
]
```

---

## 7. Eval Script

```python
# eval/regression.py
import json, time
from statistics import mean

def run_test(case):
    start = time.time()
    result = run_full_pipeline(case["question"], user_id="eval_user")
    latency_ms = (time.time() - start) * 1000
    
    # Score
    answer = result["answer"]
    score = {
        "id": case["id"],
        "latency_ms": latency_ms,
        "passed_keywords": all(k.lower() in answer.lower() for k in case.get("expected_keywords", [])),
        "passed_tools": all(t in [tr["name"] for tr in result["tool_results"]] for t in case.get("expected_tools", [])),
        "cost_usd": estimate_cost(result),
    }
    score["passed"] = score["passed_keywords"] and score["passed_tools"]
    return score

def main(test_set_path, threshold):
    cases = json.load(open(test_set_path))
    results = [run_test(c) for c in cases]
    
    pass_rate = sum(1 for r in results if r["passed"]) / len(results)
    avg_latency = mean(r["latency_ms"] for r in results)
    total_cost = sum(r["cost_usd"] for r in results)
    
    print(f"Pass rate: {pass_rate:.1%}")
    print(f"Avg latency: {avg_latency:.0f}ms")
    print(f"Total cost: ${total_cost:.2f}")
    
    json.dump({"accuracy": pass_rate, "latency_p95": p95(results), "cost": total_cost},
              open("eval/results.json", "w"))
    
    if pass_rate < threshold:
        print(f"❌ Below threshold {threshold}")
        sys.exit(1)
```

---

## 8. Dashboards

### Grafana Dashboard Panels

```json
{
  "panels": [
    {"title": "Requests/min", "datasource": "prometheus", "expr": "rate(requests_total[1m])"},
    {"title": "Latency P95", "expr": "histogram_quantile(0.95, request_latency_seconds_bucket)"},
    {"title": "Error rate", "expr": "rate(requests_total{status=~'5..'}[5m]) / rate(requests_total[5m])"},
    {"title": "Cost/day", "expr": "sum(claude_cost_usd_total[1d])"},
    {"title": "Cache hit rate", "expr": "rate(cache_hits[5m]) / rate(cache_total[5m])"},
    {"title": "Quality score (avg)", "expr": "avg(llm_judge_score)"}
  ]
}
```

---

## 9. Alerts

```yaml
# alerts.yml
groups:
  - name: llm-app
    rules:
      - alert: HighErrorRate
        expr: rate(requests_total{status=~"5.."}[5m]) / rate(requests_total[5m]) > 0.05
        for: 5m
        labels: { severity: page }
        annotations:
          summary: "Error rate > 5%"
      
      - alert: HighLatency
        expr: histogram_quantile(0.95, request_latency_seconds_bucket) > 5
        for: 5m
      
      - alert: CostSpike
        expr: rate(claude_cost_usd_total[1h]) > 5 * avg_over_time(claude_cost_usd_total[7d:1h])
      
      - alert: QualityDrop
        expr: avg_over_time(llm_judge_score[1h]) < 0.7
```

---

## 🛠️ Day 88 Deliverables

- [ ] Langfuse traces visible
- [ ] Custom spans for retrieval, generation
- [ ] LLM judge scoring (offline + sample online)
- [ ] CI eval gate active
- [ ] Golden test set (≥ 50 cases)
- [ ] Grafana dashboard with 6+ panels
- [ ] Alert rules configured
- [ ] Demo: PR with prompt change → CI runs eval → see results in comments

[ต่อไป → Day 89 :material-arrow-right:](day-89.md){ .md-button .md-button--primary }
