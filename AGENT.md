# AgentScope — Agent Usage Guide

AgentScope is a single-page tool that analyzes an AI agent description and recommends guardrails, code-based evals, and production metrics tailored to that agent's specific risk profile.

## What It Does

1. Developer describes their agent in natural language
2. Optionally selects quick-tag categories (RAG, Coding, Agentic, Financial, Medical, etc.)
3. Clicks **Get Recommendations**
4. Claude analyzes the description and returns a structured risk assessment covering:
   - Risk profile specific to the agent
   - First guardrail to ship
   - Critical eval to run in production
   - Key production metric for early warning
5. Below the AI analysis, a curated library of guardrails, evals, and metrics is surfaced — scored and ranked by relevance to the detected agent type

## Tool Structure

```
index.html          — Single-file app (HTML + CSS + JS, no build step)
AGENT.md            — This file
```

## How the Recommendation Engine Works

**Keyword detection**: The agent description is scanned against a `KEYWORD_MAP` that maps 14 agent category tags (rag, coding, agentic, medical, financial, etc.) to keyword arrays. Detected tags combine with manually selected quick-tags.

**Scoring**: Each item in the recommendation library has a `tags[]` array. Items tagged `"always"` always score 3. Others score by tag overlap count. Results are sorted by score, then by priority (critical > high > medium > low).

**AI layer**: If a Claude API key is provided, a streaming call to `claude-sonnet-4-6` generates a bespoke analysis using the description and detected tags as input. The prompt forces structured output with four sections: Risk Profile, First Guardrail, Critical Eval, Key Metric.

## Recommendation Library

| Category | Items |
|----------|-------|
| Guardrails | 10 (prompt injection, PII, hallucination, tool validation, domain boundary, output length, rate limiting, max iterations, sensitive data filter, escalation trigger) |
| Code-based Evals | 12 (faithfulness, answer relevancy, context precision, tone, format compliance, tool accuracy, safety classifier, regression suite, multi-turn coherence, latency SLA, code quality, bias detection) |
| Dashboard Metrics | 13 (latency p50/p95/p99, token usage, error rate, tool success rate, retrieval quality, user satisfaction, cost per query, escalation rate, hallucination rate, goal completion, session length, retry rate, output drift) |

## Claude API Integration

The tool calls `POST https://api.anthropic.com/v1/messages` directly from the browser with:

```
model: claude-sonnet-4-6
max_tokens: 600
stream: true
header: anthropic-dangerous-direct-browser-access: true
```

The API key is provided by the user at runtime and stored in `localStorage` under the key `agentscope_key`. It is never sent anywhere except directly to `api.anthropic.com`.

## System Prompt (Claude)

```
You are an expert AI agent observability architect. Analyze AI agent descriptions
and give sharp, specific, actionable risk assessments. Be direct, concrete, and
specific to the agent described — never generic.
```

## User Prompt Structure

```
Agent: "{description}"
Categories detected: {tags}

Analyze this agent's observability needs in exactly this structure:

**Risk Profile**
The 2-3 highest-priority failure modes specific to this agent and why they matter.

**First Guardrail to Ship**
The single most important guardrail for this specific agent and why it's the priority.

**Critical Eval**
The one eval metric that most directly measures whether this agent is working correctly.

**Key Production Metric**
The single metric that would be the clearest early warning signal of degradation.

Be specific to this agent. 200 words max.
```

## Adding New Recommendations

Each item in `GUARDRAILS`, `EVALS`, or `METRICS` follows this schema:

```javascript
{
  id: "unique_id",
  name: "Display Name",
  desc: "One-line description shown in collapsed card",
  priority: "critical" | "high" | "medium" | "low" | "always",
  tags: ["tag1", "tag2"],   // use "always" to show for all agents
  why: "Why this matters — shown when card is expanded",
  impl: "How to implement — shown when card is expanded",
  threshold: "Optional: target thresholds e.g. ≥ 0.85 | Alert: < 0.80",
  code: `Optional: Python code snippet`
}
```

Valid tags: `rag`, `qa`, `factual`, `research`, `customer_service`, `coding`, `agentic`, `autonomous`, `multi_turn`, `financial`, `medical`, `content`, `data_analysis`, `pii`, `always`

## Live URL

[josh-berkeley.github.io/agentscope](https://josh-berkeley.github.io/agentscope/)
