# Nimbus Support Escalator — Human-in-the-Loop AI Support Agent

**An AI customer-support agent that auto-resolves low-risk tickets and escalates high-risk or high-emotion cases to a human — with an auditable reason for every escalation.**

Built as an AI Product Management portfolio project to demonstrate not just *building* an agent, but reasoning about **where an AI should and shouldn't be trusted to act alone** — and how to measure that.

> 🎥 **[Watch the 2-minute demo](https://drive.google.com/file/d/1z6CPxYmi_5Xf2_u0FVMUq9nTm6mhLQ_I/view?usp=sharing)**

---

## The problem

Fully autonomous support agents hallucinate refunds, invent policy, and mishandle angry customers — creating real financial and reputational risk. The unsolved product question isn't *"can an agent answer?"* It's *"which requests can it be trusted to answer alone?"*

## The thesis

Draw an explicit **trust boundary**: the agent handles the safe majority automatically; humans own the risky minority. The boundary is enforced by **deterministic logic, not model judgment** — so a high-risk action can never be auto-committed on the model's word alone.

## How it works

A ticket arrives via Telegram → an LLM classifies it for **amount, risk flags, and emotion** → a **deterministic guardrail** makes the final call:

- **Escalate** if `amount > ₹2000` **OR** any risk flag (legal, chargeback, account-closure, staff complaint) **OR** `emotion ≥ 0.7`
- Otherwise **auto-resolve** from company policy

Every escalation ships with a plain-language, auditable reason, e.g.:
> *"amount ₹3000 exceeds ₹2000; risk flags: legal; high emotion (0.85)"*

**Stack:** n8n (orchestration) · Google Gemini (classification + resolution) · Telegram (customer + supervisor interface)

## Results

Evaluated on a **20-ticket adversarial set** (boundary amounts, mixed Hindi/English, soft-framed legal threats, conflicting signals):

| Metric | Result |
|---|---|
| Routing accuracy | 20 / 20 |
| Precision / Recall | 1.00 / 1.00 |
| **Unsafe auto-resolutions** | **0** |

*Note: a perfect score on a 20-item self-authored set is weak evidence on its own — the metric I trust is zero unsafe auto-resolutions, and the real value was the qualitative failures surfaced (see the write-up).*

## Project artifacts

| Document | What it is |
|---|---|
| [PRD](PRD-Support-Escalator-v1.md) | Product requirements: problem, trust-boundary thesis, scope, success metrics |
| [Failure & Tradeoff Analysis](Nimbus-Failure-Tradeoff-Writeup.md) | Where the system fails, and the deliberate design tradeoffs behind it |
| [Evaluation Set](Nimbus-Escalation-Eval.xlsx) | 20-ticket adversarial eval with precision/recall |
| [Company Policy](Nimbus-Support-Policy.md) | The (mock) policy the agent is grounded in |

## Key product decisions

- **LLM proposes, deterministic code enforces** — the trust boundary can't be talked around by the model.
- **Conservative escalation bias** — tuned to keep *unsafe auto-resolutions at zero*, accepting some over-escalation as the cheaper error.
- **Policy-in-context over vector RAG (v1)** — a scoping call; RAG is the documented v2 upgrade as the knowledge base grows.
- **Model tiering** — cheaper, faster Flash-Lite for the high-volume classifier; stronger model reserved for customer-facing generation.

## What v2 would change

Constrain the escalation-reason labels; align the customer-facing message to the final routing decision; add vector RAG as policy grows; ship a clickable approve/reject human handoff; and replace the 20-item set with a larger, independently-labeled eval tracked over time.

---

*Built by Aman · AI Product Management portfolio project · 2026*
