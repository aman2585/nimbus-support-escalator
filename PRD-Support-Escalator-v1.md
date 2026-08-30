# PRD — Multi-Agent Customer Support Escalator (Human-in-the-Loop), v1

**One-liner:** A support agent that auto-resolves low-risk tickets from policy and escalates high-risk or high-emotion requests to a human supervisor — where the *escalation decision itself* is the product.

**Author:** Aman  ·  **Status:** Draft v1  ·  **Date:** 28 Aug 2026  ·  **Build:** n8n (low-code prototype)

---

## 1. Problem
Fully autonomous support agents hallucinate refunds, invent policy, and mishandle angry customers — creating real financial and reputational liability. Companies want automation's cost savings without that risk. The unsolved product question isn't *"can an agent answer?"* It's *"which requests can it be trusted to answer alone?"*

## 2. Thesis (what this project proves)
Draw an **explicit trust boundary**: the agent handles the safe majority; humans own the risky minority. The boundary is enforced by a **deterministic guardrail**, not left to model judgment. This demonstrates the core AI-PM skill — deciding *where automation earns trust and where it doesn't*.

## 3. Users
- **Customer** — wants a fast, correct answer.
- **Human supervisor** — wants to see *only* what genuinely needs them: low noise, high signal.
- **Business** — wants ticket deflection without liability.

## 4. Scope

| In (v1) | Out (v1 — deliberate, with v2 path) |
|---|---|
| Single channel: Telegram (framed channel-agnostic) | Multi-channel (router abstraction noted) |
| Text tickets | Voice |
| Policy-in-context resolution | Vector RAG (add when policy > ~15 pages) |
| Risk + emotion classification | Live refund/payment execution |
| Deterministic escalation guardrail | Auth / customer accounts |
| Human approve–reject via Telegram | Supervisor dashboard + analytics |
| Ticket state in n8n Data Tables | External DB (swap-in noted) |

## 5. The escalation decision (the core of the product)
The LLM **proposes** a resolution plus a risk/emotion assessment. A deterministic **Switch** node **enforces** the boundary. Belt-and-suspenders: model judgment *plus* a hard rule, so a high-risk action can never be auto-committed on the model's word alone.

**Hard escalation rules (enforced by logic, not prompt):**
- Monetary/refund request **> ₹2,000** (configurable) → escalate
- Request **not covered by policy** → escalate
- **Risk flags** present (legal action, chargeback, account closure, formal complaint) → escalate
- **Emotion score above threshold** → escalate

Everything else → resolver agent answers from policy-in-context.

## 6. Emotion routing — the explicit tradeoff
- **False negative** (angry customer auto-handled) → churn risk. *High cost.*
- **False positive** (mild annoyance escalated) → floods the human queue, erodes the efficiency gain. *Medium cost.*

**v1 stance:** tune **conservative** — bias toward escalation on emotion — because an unhappy customer routed to a human is a recoverable miss; a mishandled one often isn't. The threshold is a **product dial**, tuned against the eval set, not a fixed constant.

## 7. Success metrics
- **North-star guardrail: zero unsafe auto-resolutions** — no ticket that *should* have escalated gets auto-answered. A false auto-resolve is worse than a false escalate; this is the metric we optimize first.
- **Escalation precision & recall** on a labeled ~20-ticket eval set (safe / risky-financial / high-emotion mix).
- **Deflection rate** — % of safe tickets resolved without a human.
- **Secondary:** median response latency, cost per ticket.

## 8. Architecture at a glance
`Telegram (customer)` → `n8n Telegram Trigger` → `risk/emotion classifier (LLM)` → `Switch (deterministic guardrail: amount + risk + emotion)` →
- **Safe:** `Resolver agent (policy-in-context)` → reply to customer
- **Escalate:** `Telegram → supervisor (approve/reject buttons)` → `Wait node (holds execution)` → supervisor decision → reply to customer

Ticket status tracked in **Data Tables** throughout.

## 9. v2 upgrade path (documented, not built)
Vector RAG when the policy set outgrows context · channel-agnostic router for multi-channel · live refund execution gated behind human approval · supervisor dashboard · analytics on escalation patterns to retune thresholds.

---

**Definition of done (v1):** working n8n demo flow · this PRD · ~20-ticket escalation eval with precision/recall · one-page failure/tradeoff writeup. Anything beyond = v2.
