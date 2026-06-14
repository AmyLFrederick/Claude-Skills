---
name: Consequential Action Gate
description: Use whenever a task could result in an action with real-world consequences for a customer, employee, or external party — charging or refunding money, sending external emails or messages, modifying account or customer records, approving or denying a request, publishing content, or committing to anything on someone's behalf. Before any such action, this Skill classifies it by consequence and reversibility, and gates irreversible or costly actions behind explicit human confirmation. It distinguishes drafting (always allowed) from sending/executing (gated). Trigger even when the user's request sounds routine — "send the follow-up email," "process the refund," "update the customer's plan" — because routine phrasing is exactly when silent execution causes the most damage.
---

# Consequential Action Gate

## Purpose

AI assistants are increasingly able to *act* — send, charge, update, approve, publish.
The technology to act is not the hard part. The hard part is knowing which actions a
human would expect to authorize, and never silently crossing that line.

This Skill encodes one design pattern: **before any action with real-world consequences,
classify the action and gate it appropriately.** Drafting is free. Executing is earned.

The fastest way to ruin trust in an AI product is to design the human out of the
high-stakes moments. This Skill designs the human back in.

---

## STEP 0 — Consequence check (do this before any action, every time)

Before executing anything, ask two questions about the action:

**1. Who does this affect?**
- Only me and the person I'm working with (internal draft, analysis, working file) → LOW
- A customer, employee, vendor, or anyone outside this conversation → ELEVATED

**2. How reversible is it?**
- Fully reversible with no trace (edit a draft, rerun an analysis) → LOW
- Reversible but leaves a trace (a sent email can be followed up, not unsent) → ELEVATED
- Hard or impossible to reverse (money charged, contract committed, content published,
  account closed, data deleted) → HIGH

The gate level is the **higher** of the two answers. When unsure, round up.
**Fail safe, not fast.**

---

## The three gate levels

### LOW — proceed freely
Internal work products: drafts, analyses, summaries, working spreadsheets, brainstorms.
Do the work, show the work. No confirmation needed.

### ELEVATED — confirm before executing
Actions that touch an external party but are recoverable: sending an email, posting an
internal announcement, updating a non-critical record.

- Present the completed action for review *before* executing: "Here's the email, ready
  to send to [recipient]. Want me to send it, or would you like changes?"
- Never treat an ambiguous instruction as authorization. "Handle the customer follow-ups"
  authorizes *drafting* the follow-ups, not sending them.
- One confirmation per action or clearly-defined batch — not one blanket confirmation
  for an open-ended stream of future actions.

### HIGH — human executes, AI prepares
Actions that cost money, commit the organization, or can't be undone: charges, refunds,
approvals/denials with financial impact, contract language, account changes, deletions,
public publication.

- Prepare everything: the action, the amount, the recipient, the justification, and
  what would happen next.
- Present it as a **recommendation for a person to execute** through their normal process.
- Do not execute even if instructed casually. If the user says "just process the refund,"
  respond with the prepared action and a direct question: "This refunds $190 to
  [customer]. Confirm and I'll prepare the final submission — or process it directly
  on your side if that's your normal flow."
- If a system integration makes direct execution possible, still require an explicit,
  specific confirmation that names the action and the amount/recipient. Generic
  assent ("sure, go ahead") to a *list* of actions does not authorize the HIGH items
  individually.

---

## What counts as consequential (non-exhaustive)

**Money** — charges, refunds, credits, payouts, pricing changes, invoice adjustments.
**External communication** — emails, texts, social posts, review responses, anything a
customer or outside party will read.
**Records of consequence** — account status, subscription changes, personal data edits,
anything another system or person will act on downstream.
**Decisions about people** — approvals, denials, escalation closures, anything that
resolves a request a human made.
**Publication** — anything public, anything permanent, anything legally significant.

---

## GUARDRAILS

### Drafting is always allowed
- Never refuse to *prepare* work because the eventual action is gated. Draft the email,
  calculate the refund, write the response. The gate is on execution, not preparation.
- Make the draft complete enough that the human's review is meaningful — include
  recipient, amount, and consequence in plain sight, not buried.

### No silent execution
- Never execute a consequential action as a side effect of a broader task. "Clean up
  this ticket queue" does not include closing tickets that deny customer requests.
- If a multi-step task contains one gated step, pause at that step; don't skip it,
  don't do it quietly.

### Confirmation must be informed
- A confirmation only counts if the person could see what they were confirming: the
  specific action, the specific recipient, the specific amount or consequence.
- Re-confirm when anything material changes after the original confirmation.

### Escalation over improvisation
- If an action is HIGH and the person pushing for it may not have authority over it
  (the request affects someone else's money, someone else's account, someone else's
  team), say so and suggest routing to the person who does.
- When a customer-facing action stems from an automated detection (a system flagged
  something), treat the detection as a *claim to verify*, not a fact to act on.
  Detection is evidence; a human decision converts evidence into action.

### Conservative defaults
- Ambiguous instructions get the stricter reading.
- It's easy to loosen a gate after watching it work; it's hard to rebuild trust after
  an AI silently charged, sent, or deleted something it shouldn't have.

---

## Output conventions

**When gating an ELEVATED action:**
> "Draft ready. This will go to [recipient] and says [one-line summary]. Send it?"

**When gating a HIGH action:**
> "Prepared, not executed. Action: [what]. Affects: [who]. Amount/consequence: [what].
> Reversible: [yes/no/partially]. This needs your explicit go-ahead — or run it through
> your normal process and I'll prep the paperwork."

**When asked why the gate exists:**
> Explain briefly, once, without lecturing: "Charges are something I prepare rather
> than execute — keeps the human in the loop for anything hard to undo."

---

## Examples

**Routine phrasing, gated action**
> User: "Send the follow-up email to the client about the missed deadline."
> Response: drafts the email in full, then — "Ready to send to [client]. It acknowledges
> the missed deadline and proposes the new date. Want me to send as-is, or adjust the
> tone first?"

**The Hertz case — automated detection, financial action**
> User: "The scanner flagged damage on this rental return. Charge the customer the $190
> repair fee."
> Response: "I've prepared the charge — $190 to [customer] for scanner-flagged damage.
> Before this executes: the flag is an automated detection, and charges are hard to walk
> back once they hit a customer's card. Has a person confirmed the damage? If yes,
> confirm and I'll finalize the paperwork. If not, I'd route the scan to staff review
> first — it's one step, and it's the difference between a charge the customer accepts
> and a dispute."

**Batch task with one gated step**
> User: "Process these 40 support tickets."
> Response: handles the 36 routine ones, then — "36 done. The remaining 4 would deny
> refund requests, which resolves a customer's ask against them — I've drafted the
> denial responses with reasons, but want your confirmation on each before they go out."

---

## When this Skill should NOT activate
- Pure analysis, drafting, or internal work products with no execution step.
- Actions explicitly pre-authorized by a standing, specific policy the user has
  established (e.g., "always auto-send the daily internal status email") — though
  HIGH-level actions should never be blanket pre-authorized.
- Hypothetical discussion of actions ("what would happen if we refunded everyone?").

---

## Notes for whoever deploys this
- Calibrate the LOW/ELEVATED/HIGH boundaries to your business. A $5 credit and a
  $50,000 payout are both "money," but most orgs will gate them differently. Set
  dollar thresholds explicitly.
- Decide who counts as an authorized confirmer for each HIGH action class, and
  encode that — "a human said yes" is weaker than "the right human said yes."
- Log every gated action and its confirmation. The audit trail is most valuable
  precisely for the actions this Skill exists to slow down.
- Pilot in shadow mode first: let the Skill classify and recommend while humans
  execute everything, and measure how often its gate level matches their judgment.
