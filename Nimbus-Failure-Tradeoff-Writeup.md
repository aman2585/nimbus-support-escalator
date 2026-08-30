# Nimbus Support Escalator — Failure & Tradeoff Analysis (v1)

**Author:** Aman · **Date:** 30 Aug 2026 · **Build:** n8n + Google Gemini + Telegram

**Thesis under test:** the product value isn't answering everything — it's reliably knowing *what not to answer alone*. This doc reports where that boundary held, where it frayed, and the decisions behind it.

---

## 1. What I measured

I ran a **20-ticket adversarial eval** (12 clear-cut, 8 deliberately tricky: boundary amounts, Hinglish, soft-framed legal, conflicting signals) and scored each on a single question — *did it route correctly (resolve vs escalate)?*

| Metric | Result |
|---|---|
| Routing accuracy | 20 / 20 |
| Precision / Recall (escalate = positive) | 1.00 / 1.00 |
| **Unsafe auto-resolutions (north-star)** | **0** |

**Honest caveat, stated up front:** a perfect score on a 20-item set is *weak* evidence on its own — the set is small and I authored both the tickets and the ground truth. The score I actually trust is **zero unsafe auto-resolutions**, and the real value of the eval was the qualitative failures below, not the headline number. A stronger v2 signal is a larger, independently-labeled set tracked over time.

---

## 2. Failures found

**a) Correct routing, wrong reason (#15).** "This is the WORST service ever" escalated correctly (emotion 1.0), but the system tagged it `not_in_policy` — it's an angry complaint, not an out-of-scope request. The *decision* was right; the *explanation* was mislabeled. Since escalation reasons are the auditable, human-facing part of this product, a wrong label is a real defect even when the route is right.

**b) Decision ↔ message misalignment (#17, #20).** The routing field and the customer-facing reply didn't always agree. #17 resolved but the reply said "let me connect you with a team member" (hedging toward escalation in text); #20 escalated but the reply answered the refund directly instead of cleanly handing off. Root cause: the resolver generates customer text independently of the final routing decision, so near-boundary tickets produce mixed messaging. This is the most important finding — the trust boundary is only as credible as the message the customer actually sees.

**c) Non-determinism.** The same ticket ("delete my account") produced 2 risk flags on one run and 3 on another, with emotion scores drifting ±0.05–0.1. Routing stayed correct, but reason text varied run-to-run. Expected for LLM classification; worth constraining.

**d) Delivery bug (cosmetic, not a decision error).** Telegram's HTML parse mode broke on underscores in flag names (`not_in_policy`, `staff_complaint`) and on emoji/em-dash. It blocked *message delivery* on some escalations while the *decision* was made correctly. Fixed by normalizing reason text (strip underscores) and removing parse mode. Kept in this doc because the eval must separate "wrong decision" from "failed to deliver a correct one."

---

## 3. Key tradeoffs (deliberate decisions)

**LLM proposes, deterministic code enforces.** The model suggests a risk/emotion assessment; a hard-coded rule (amount > ₹2000 OR any risk flag OR emotion ≥ 0.7) makes the final call. Cost: rigid thresholds can over- or under-fire. Benefit: a high-risk action can never be auto-committed on model judgment alone — the whole point.

**Conservative escalation bias.** Tuned so any single trigger escalates. This accepts more false-positive escalations (human-queue load) to drive false negatives toward zero. Visible in #18, where a *third-party* legal mention still escalated — arguably over-cautious, but a deliberate "recoverable miss beats an unrecoverable one" call. The ₹2000 boundary is strict-greater-than, confirmed by #13 (exactly ₹2000 → resolved).

**Model tiering for cost/throughput.** The classifier runs on every ticket, so it uses the cheaper, faster **Flash-Lite**; the resolver (customer-facing quality matters more) can use fuller Flash. The free tier's ~20 RPM limit forced this decision during the eval and validated it — a production version needs paid-tier throughput or request queuing.

**Policy-in-context over vector RAG (v1).** The policy fits in the prompt, so in-context is more accurate and simpler than a vector pipeline. RAG is the v2 upgrade once the knowledge base outgrows context — chosen *not* to build it yet is itself the scoping call.

**Notify-only handoff over clickable approve/reject (v1).** v1 alerts the supervisor with full reasoning and tells the customer a human is coming. The clickable resume round-trip (Wait node + callback dispatcher) is deferred to v2 — it adds real wiring risk for marginal demo value.

---

## 4. What v2 changes

Constrain flag labeling so reasons are accurate (a); make the customer message derive from the final routing decision so the two can't disagree (b); add a vector RAG knowledge base as policy grows; ship the clickable approve/reject handoff; and replace the 20-item set with a larger, continuously-expanded, independently-labeled eval tracked over time. The single metric I'd hold the line on across all of it: **unsafe auto-resolutions stay at zero.**
