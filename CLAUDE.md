# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is a **content repository**, not a software project. It holds study guides for Microsoft exam **AI-103 — "Developing AI Apps and Agents on Azure"** (the successor to AI-102, which retires 2026-06-30). There is no build, lint, or test tooling — the deliverables are the Markdown files themselves.

Each numbered file covers one official exam domain. The files are numbered in **suggested study order** (weight-first), not in official exam-domain order; treat this table as the file-to-domain contract:

| File | Exam domain | Weight |
|---|---|---|
| `1.md` | Implement generative AI and agentic solutions | 30–35% |
| `2.md` | Plan and manage an Azure AI solution | 25–30% |
| `3.md` | Implement information extraction solutions | 10–15% |
| `4.md` | Implement text analysis solutions (speech is a **subsection inside** this domain, not its own) | 10–15% |
| `5.md` | Implement computer vision solutions | 10–15% |

`README.md` documents the guide set, weighting, and high-leverage facts.

## Core constraint: factual accuracy against Microsoft Learn

These guides make dense, specific, drift-prone claims about Microsoft Foundry, Azure AI Search, deployment types, RBAC roles, quotas, and guardrails. Microsoft renames and re-scopes these rapidly through 2026. When editing or extending a guide, every factual claim must be verifiable against current Microsoft Learn docs. High-leverage facts that are easy to get wrong:

- The platform is now **Microsoft Foundry** (formerly *Azure AI Foundry*, earlier *Azure AI Studio*). Don't reintroduce the old names except when explicitly contrasting legacy vs. current.
- The exam is **AI-103**, not AI-102. Do not "correct" it back to AI-102.
- Keyless model inference in **new Foundry uses the `Foundry User` role, not `Cognitive Services User`** (the latter was the classic Azure OpenAI answer). Foundry RBAC roles were renamed from the old `Azure AI *` names.
- Runtime model is the **Responses API (Agents v2)** — conversations/items/responses/agent-versions — not the legacy **Assistants API** (threads/messages/runs/assistants), which is reported to sunset 2026-08-26.

## Verification workflow

The guides are fact-checked per file. The intended command is:

```
/deep-research @N.md for correctness and completeness
```

**Caveat (learned, important):** the `deep-research` verification stage is fragile under API rate limiting. Failed verifier votes get recorded as `0-0 (3 abstain)`, then mislabeled "killed," which produces a bogus "all claims refuted" headline. Distrust that final verdict. The search/fetch stage and its extracted source claims are the real signal. More reliable for spot-checking: manually `WebFetch` the primary Microsoft Learn pages in one batch of parallel fetches and verify the high-value claims directly.

## Formatting rules

The guides contain **no citations and no em dashes**. Both were stripped out; do not reintroduce either when editing or extending a guide.

- **No citations of any kind.** No inline citation tokens (the old `citeturnNviewM` / `citeturnNsearchM` research-tool artifacts), no Markdown footnote markers (`[^sg]`, `[^foundry]`, …), and no **Sources** / **References** / footnote-definition sections at the bottom of a file. State facts directly in prose. (Facts must still be verifiable against current Microsoft Learn docs per the constraint above; just do not cite them inline.)
- **No em dashes (`—`, U+2014).** Use a comma, colon, or parentheses, or split the sentence instead. Do **not** touch en dashes (`–`, U+2013); they are load-bearing in numeric ranges such as `25–30%`.

Several guides open with a dated **"Accuracy note (verified <month year>)"** block. When you re-verify content, update that date to reflect when the check actually happened.
