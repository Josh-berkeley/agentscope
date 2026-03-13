# AgentScope — Agent Observability Advisor

> Describe your AI agent. Get a Claude-powered risk assessment + a tailored set of guardrails, evals, and production metrics to ship.

**Live:** [josh-berkeley.github.io/agentscope](https://josh-berkeley.github.io/agentscope/)

---

## The Problem

Building an AI agent takes days. Knowing when it's broken in production takes weeks — because most teams ship without any observability plan.

There's no standard playbook for:
- Which guardrails actually matter for your specific agent type
- What evals to run continuously vs. at deploy time
- What metrics belong on your dashboard
- Which failure modes to prioritize given your domain (medical vs. coding vs. financial)

The result: agents that hallucinate undetected, runaway tool-calling loops burning cost overnight, PII leaking into logs, and zero early-warning signals before users start complaining.

**AgentScope solves the "I don't know what to monitor" problem** — not by being a monitoring platform, but by being the expert advisor that tells you exactly what to build.

---

## How It Works

```
┌──────────────────────────────┐     ┌──────────────────────────────────────┐
│  LHS — Describe Your Agent   │     │  RHS — Your Observability Plan       │
│                              │     │                                      │
│  [textarea: agent desc]      │────▶│  🤖 Claude Analysis (streaming)      │
│  [quick-tag chips]           │     │  Risk Profile / Guardrail / Eval /   │
│  [example presets]           │     │  Key Metric — specific to your agent │
│  [Claude API key]            │     │                                      │
│  [→ Get Recommendations]     │     │  🔍 Detected Agent Profile           │
│                              │     │  🛡️  Guardrails (N)                  │
│                              │     │  🧪  Code-based Evals (N)            │
│                              │     │  📊  Dashboard Metrics (N)           │
└──────────────────────────────┘     └──────────────────────────────────────┘
```

### Two-layer recommendation engine

**Layer 1 — Keyword matching (instant, no key needed)**
The agent description is scanned against 14 category keyword maps (`rag`, `coding`, `agentic`, `medical`, `financial`, etc.). Matched tags score items from a curated library of 35 recommendations. Results render immediately.

**Layer 2 — Claude analysis (streaming, requires API key)**
If a Claude API key is provided, `claude-sonnet-4-6` analyzes the specific agent description and generates a bespoke 4-section risk assessment:
- **Risk Profile** — the 2-3 highest-priority failure modes for this exact agent
- **First Guardrail to Ship** — the single most important guardrail and why
- **Critical Eval** — the one metric that directly measures quality
- **Key Production Metric** — the earliest warning signal of degradation

---

## System Architecture

```
INPUT
  Agent description (free text)
  + Quick tags (manual selection)
  + Claude API key (optional, localStorage)
        │
        ▼
DETECTION ENGINE (pure JS)
  KEYWORD_MAP: 14 tags × N keyword arrays
  detectTags(text, manualTags) → Set<tag>
        │
   ┌────┴────────────────────────┐
   ▼                             ▼
SCORING ENGINE              CLAUDE API (browser → Anthropic)
  GUARDRAILS[10]              POST /v1/messages
  EVALS[12]                   model: claude-sonnet-4-6
  METRICS[13]                 stream: true
                              → SSE token stream
  scoreRec():                 → structured 4-section analysis
    "always" → 3
    tag overlap → count
  filterAndSort():
    score > 0, sort by
    score then priority
        │                             │
        └──────────────┬──────────────┘
                       ▼
OUTPUT (RHS)
  🤖 Claude Analysis     ← streaming, agent-specific
  🔍 Agent Profile       ← detected types + failure modes
  🛡️  Guardrails         ← collapsible cards w/ code
  🧪  Evals              ← collapsible cards w/ code
  📊  Metrics            ← collapsible cards w/ thresholds
```

---

## Recommendation Library

| Category | Count | Covers |
|---|---|---|
| 🛡️ Guardrails | 10 | Prompt injection, PII masking, hallucination guard, tool call validator, domain boundary, output length, rate limiting, max iterations, sensitive data filter, escalation trigger |
| 🧪 Code-based Evals | 12 | Faithfulness, answer relevancy, context precision, tone consistency, format compliance, tool accuracy, safety classifier, regression suite, multi-turn coherence, latency SLA, code quality, bias detection |
| 📊 Dashboard Metrics | 13 | Latency p50/p95/p99, token usage, error rate, tool success rate, retrieval quality, user satisfaction, cost per query, escalation rate, hallucination rate, goal completion, session length, retry rate, output drift |

Each item includes: priority level, why it matters, how to implement it, target thresholds, and a Python code snippet.

---

## Agent Categories

The tool detects and scores recommendations across 14 agent types:

`rag` · `qa` · `factual` · `research` · `customer_service` · `coding` · `agentic` · `autonomous` · `multi_turn` · `financial` · `medical` · `content` · `data_analysis` · `pii`

---

## Getting Started

1. Open [josh-berkeley.github.io/agentscope](https://josh-berkeley.github.io/agentscope/)
2. Describe your agent in the text box (or click an example preset)
3. Select any relevant quick-tags
4. *(Optional)* Paste your [Claude API key](https://console.anthropic.com/) for AI-powered analysis
5. Click **→ Get Recommendations**

The API key is stored only in your browser's `localStorage`. It is never sent anywhere except directly to `api.anthropic.com`.

---

## Technical Details

- **No backend, no build step** — single `index.html` file, runs entirely in the browser
- **No data retention** — nothing is logged or stored server-side
- **Claude API** — direct browser → Anthropic call with `anthropic-dangerous-direct-browser-access: true`
- **Streaming** — SSE stream parsed token-by-token for live render with cursor animation

---

## Files

```
index.html    Single-file app (HTML + CSS + JS)
AGENT.md      Technical reference for the recommendation engine
README.md     This file
```

---

## What This Is vs. What It Isn't

| AgentScope IS | AgentScope IS NOT |
|---|---|
| An observability advisor | A monitoring platform |
| An LLM-powered risk assessor | An agent runtime or framework |
| A curated failure mode library | A testing tool |
| A zero-setup static tool | A SaaS product |

---

Built by [@Josh-berkeley](https://github.com/Josh-berkeley)
