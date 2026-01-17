# Panic Gap Decision Guides

## Context
Users often photograph unexpected mail or documents when they feel anxious or unsure what to do. The core product value is not OCR. It is closing the "panic gap" by turning confusion into a safe first step, clear options, and a calm path forward.

## Problem Statement
A user receives a scary or surprising letter (collections, late payment, insurance denial). They do not know:
- Is this real?
- Is it too late to act?
- Will this hurt my credit?
- Should I pay, dispute, negotiate, or wait?

In panic, more information can make anxiety worse. The product must reduce ambiguity and restore agency.

## Product Goal
Anxiety -> clarity -> action, with evidence.

The app should feel like a calm friend who says:
"You are not behind. You are here. Lets take the safest next step."

## Core UX Pattern (Decision Guide)
1. Containment
   - "We saved this. Nothing will be lost." Show the doc is stored.

2. Facts vs Unknowns
   - Clearly separate detected facts from uncertain items.
   - Highlight the evidence on the document when possible.

3. Safe First Step
   - If the user is unsure, recommend the lowest risk step (often verification).

4. Rational Pathways
   - Show 3 to 4 pathways with outcomes and tradeoffs.
   - Do not force a single recommendation.

5. Gentle, clear tone
   - Calm, direct language. No hype. No shame.

## Decision Guide Schema (Draft)
```json
{
  "category": "collections_letter",
  "calm_summary": "This appears to be a debt collection notice from ACME Collections for $450. A response may be time sensitive.",
  "facts": [
    {"label": "Sender", "value": "ACME Collections", "evidence": "logo header"},
    {"label": "Amount", "value": "$450", "evidence": "paragraph 2"},
    {"label": "Due date", "value": "2024-11-15", "evidence": "footer"}
  ],
  "unknowns": [
    {"label": "Is this your debt?", "why": "No account match found"}
  ],
  "urgency": {"level": "medium", "why": "Deadline appears on page"},
  "confidence": {"level": "medium", "why": "Amount and sender detected; account number unclear"},
  "safe_first_step": "Verify the sender and request a written validation before paying.",
  "pathways": [
    {
      "id": "verify",
      "label": "Verify or dispute",
      "outcome": "Confirms if the debt is valid before paying.",
      "steps": ["Request validation", "Check account history", "Dispute inaccuracies"],
      "avoid": "Do not share sensitive info until verified"
    },
    {
      "id": "pay",
      "label": "Pay or settle",
      "outcome": "Resolve quickly but may not erase past credit marks.",
      "steps": ["Ask for payoff amount", "Get written confirmation", "Pay via traceable method"],
      "avoid": "Do not pay without a written payoff amount"
    },
    {
      "id": "plan",
      "label": "Payment plan",
      "outcome": "Spreads cost over time.",
      "steps": ["Call and request plan", "Confirm terms in writing"],
      "avoid": "Do not agree to verbal-only terms"
    },
    {
      "id": "wait",
      "label": "Wait and monitor",
      "outcome": "No action now but risk of escalation.",
      "steps": ["Set a reminder", "Monitor mailbox"],
      "avoid": "Do not miss a stated deadline"
    }
  ],
  "templates": {
    "call_script": "Hi, I received a letter dated ___. Please confirm the account and send validation details in writing.",
    "letter_template": "I am requesting written validation of the debt referenced in your letter dated ___."
  },
  "disclaimer": "Information is for general guidance, not legal advice."
}
```

## Collection Letter Example (User Facing Copy)
Calm Summary:
- "This looks like a collection letter from ACME Collections for $450. A response may be time sensitive."

Safe First Step (if unsure):
- "Verify the sender and request written validation before paying or sharing sensitive info."

Rational Pathways:
- Verify or dispute: confirm the debt first, then decide.
- Pay or settle: resolve now; confirm in writing.
- Payment plan: negotiate terms you can afford.
- Wait and monitor: possible escalation; set a reminder.

Questions to tailor guidance:
- Is this your debt?
- Is there a deadline in the letter?
- Have you received this before?

## Guardrails and Safety
- Evidence-first. Only claim what is present in the document.
- Show confidence and uncertainty explicitly.
- Never guarantee outcomes like credit impact or legal status.
- Provide neutral, conservative language.
- Offer templates and scripts, not legal advice.

## Scope and Delivery Plan
Phase 1: Decision Guide MVP
- 10 categories that cover most anxiety mail.
- Human curated templates.

Phase 2: Blueprint library
- Store category templates and rules in a table.
- Add embeddings later if needed.

Phase 3: Personalization
- Tone preference (short vs detailed).
- Reminder style (gentle vs direct).

## Notes
The "10,000 category" vision can be a roadmap, but start small and prove trust first.
Avoid auto-research loops without human review.
