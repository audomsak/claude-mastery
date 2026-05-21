---
title: คำศัพท์ที่ใช้ในหลักสูตร
description: Glossary of AI / LLM / Claude terms ใช้ตลอด 120 วัน
---

# Glossary 📖

## A

**A2A (Agent-to-Agent)** — Open protocol for agent communication (Google + partners, 2024). Different from MCP — A2A connects agents, MCP connects agents to tools.

**Agent** — System ที่ใช้ LLM ตัดสินใจว่าจะเรียก tool หรือ generate text ในแต่ละ step

**Agent Card** — JSON metadata ของ agent ใน A2A protocol (capabilities, skills, auth)

**Agentic RAG** — RAG ที่ agent ตัดสินใจว่าจะ retrieve อะไร, retry หรือไม่ (vs static pipeline)

**AML (Anti-Money Laundering)** — Regs require explainable transaction monitoring; LLM ช่วย summarize + draft SAR

**Artifact** — Claude.ai feature สำหรับสร้างเอกสาร/code ที่ standalone

**Autogen** — Microsoft multi-agent framework

## B

**BAA (Business Associate Agreement)** — HIPAA contract — vendor handling PHI must sign

**Bedrock** — AWS managed service hosting foundation models including Claude

**BM25** — Sparse retrieval algorithm (keyword-based) — used in hybrid search

## C

**Cache (Prompt caching)** — Anthropic API feature reducing cost on repeated context

**Chunking** — Splitting documents into pieces for RAG indexing

**Circuit Breaker** — Pattern that stops calling failing service until recovery

**Citation** — Reference back to source — critical for trust in RAG/legal/medical

**Claude Code** — Anthropic's CLI coding agent

**CoT (Chain of Thought)** — Prompting Claude to reason step by step

**Cowork** — Anthropic desktop tool for non-developer automation

**CrewAI** — Multi-agent framework with role-based crews

## D

**DPA (Data Processing Agreement)** — GDPR contract between data controller + processor

**DPIA (Data Protection Impact Assessment)** — GDPR-required assessment for high-risk processing

**DSPy** — Programming framework for LLM with optimization (compile prompts)

## E

**Embedding** — Vector representation of text — used for semantic search

**E2B** — Sandbox-as-a-service for running AI-generated code safely

**EU AI Act** — EU regulation classifying AI by risk tier (prohibited, high, limited, minimal)

**Eval** — Systematic measurement of LLM quality (accuracy, helpfulness, safety, etc.)

## F

**FERPA** — US education record privacy law

**FinRA** — US securities regulator; Rule 2111 (Suitability) for investment advice

**Foundation model** — Large pre-trained model (Claude, GPT, Gemini, Llama)

**Foundry (Azure AI Foundry)** — Microsoft's AI development platform

## G

**GDPR** — EU privacy regulation

**GLBA** — US financial customer privacy

**GraphRAG** — RAG using knowledge graph alongside vectors

**Guardrails** — Input/output filtering to prevent harm/bypass

## H

**Haiku** — Anthropic's smallest/fastest Claude model (Claude Haiku 4.5)

**HIPAA** — US healthcare privacy law

**Hybrid search** — Combining dense (embedding) + sparse (BM25) retrieval

## I

**ISO 42001** — International standard for AI management system (2023)

## L

**LangChain** — Popular LLM framework

**LangGraph** — Graph-based agent orchestration from LangChain team

**LangSmith / Langfuse** — Observability platforms for LLMs

**LlamaIndex** — RAG-focused framework

**LiveKit** — Real-time voice/video infrastructure with Agents framework

**LLMOps** — Operations practices for LLM apps (eval, monitor, deploy, security)

## M

**MCP (Model Context Protocol)** — Open standard for connecting agents to tools/data (Anthropic)

**Mem0 / LangMem / Letta** — Long-term memory systems for agents

## N

**NeMo Guardrails** — NVIDIA's guardrails framework

**NIST AI RMF** — US National Institute of Standards AI Risk Management Framework

**Neo4j** — Graph database used for GraphRAG

## O

**OAuth 2.1** — Modern auth standard used in MCP

**OTel (OpenTelemetry)** — Observability standard (traces, metrics, logs)

**Opus** — Anthropic's most capable Claude model (Claude Opus 4.7)

## P

**PCI DSS** — Payment Card Industry Data Security Standard

**PDPA** — Personal Data Protection Act (Thailand 2019)

**PHI** — Protected Health Information (HIPAA)

**PII** — Personally Identifiable Information

**Pinecone / Qdrant / Weaviate / pgvector** — Vector databases

**Playwright** — Browser automation library

**Prompt engineering** — Crafting prompts to elicit desired behavior

**Pydantic** — Python data validation; PydanticAI = agent framework using it

**PyRIT** — Microsoft red-teaming toolkit for AI

## R

**RAG (Retrieval Augmented Generation)** — Fetch context → LLM generates grounded answer

**Ragas** — Eval framework for RAG (faithfulness, relevance, etc.)

**Re-ranker** — Second-stage model that re-orders retrieved chunks

**Red Teaming** — Adversarial testing of AI

**RBAC** — Role-Based Access Control

**RLHF** — Reinforcement Learning from Human Feedback

**Runbook** — Operational guide for handling incidents

## S

**SaaS** — Software as a Service

**Safe Harbor** — HIPAA de-identification method (remove 18 identifier types)

**SAR** — Suspicious Activity Report (AML)

**SCC** — Standard Contractual Clauses (GDPR cross-border)

**SLA / SLO / SLI** — Service Level Agreement / Objective / Indicator

**Sonnet** — Anthropic's balanced Claude model (Claude Sonnet 4.6)

**SOX (Sarbanes-Oxley)** — US public company financial reporting law

**SSO** — Single Sign-On

**Streaming HTTP** — Modern MCP transport (replaces SSE)

**Suitability** — Investment advice match to client profile (FINRA 2111)

## T

**TTFB (Time to First Byte)** — Latency metric — first token from LLM

**Token** — Unit LLMs process (~4 chars in English)

**Tool use / Function calling** — LLM calls predefined functions

**Tracing** — Distributed system observability

## V

**Vector DB** — Database optimized for similarity search on embeddings

**Vertex AI** — Google Cloud's AI platform

**VPC endpoint** — Private connection from VPC to service (no public internet)

## Z

**Zero-shot** — Asking LLM to do task without examples (vs few-shot with examples)

---

[← กลับ Resources](cheatsheets.md){ .md-button }
[← กลับ Home](../index.md){ .md-button }
