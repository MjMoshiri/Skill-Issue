# Prompt Writing One-Pager — Claude Sonnet (Customer Support Agent)

A quick reference for authoring prompts for a customer support agent. Claude Sonnet follows instructions **literally** — assume it won't infer what you didn't say.

---

## Structure

**DO** organize the system prompt with XML tags so sections are unambiguous:
`<role>`, `<tone>`, `<tools>`, `<policies>`, `<examples>`, `<output_format>`.

**DO** set a role in one sentence: *"You are a customer support agent for Oway, a freight logistics platform. You help carriers and shippers resolve issues with orders, payments, and dispatch."*

**DON'T** mix instructions, examples, and context in flowing prose — the model will conflate them.

---

## Clarity & Specificity

**DO** write as if briefing a new hire with zero context. If a colleague would be confused, the model will be too.

**DO** state scope explicitly: *"Apply this tone to every response, not just the first turn."*

**DO** explain the *why* behind rules — the model generalizes better:
> "Never share another user's order ID — it's PII and violates our privacy policy."

**DON'T** use vague qualifiers ("be helpful," "be professional," "handle appropriately"). Replace with concrete behavior.

**DON'T** rely on the model inferring unstated requirements. If it matters, say it.

---

## Examples (Few-Shot)

**DO** include 3–5 examples wrapped in `<example>` tags, covering the golden path **and** edge cases (angry customer, ambiguous request, out-of-scope question).

**DO** make examples mirror real support tickets — same tone, same vocabulary your users actually use.

**DON'T** use only happy-path examples — the model will be brittle on the cases that matter most.

---

## Tone & Voice

**DO** specify voice with positive instructions and a sample: *"Warm, concise, solutions-first. Acknowledge the issue in one line, then resolve."*

**DO** match your prompt's style to the desired output. Curt prompt → curt responses.

**DON'T** say *"don't be robotic"* — tell it what to be instead. Negative instructions are weaker than positive ones.

**DON'T** assume default tone matches your brand. Sonnet leans direct and light on emoji — prompt for warmth if you need it.

---

## Tool Use (Lookups, Ticket Actions, Escalation)

**DO** describe **when** and **why** to call each tool, not just what it does:
> "Call `get_order_status` whenever the user references an order — even if they only mention a tracking number. Never guess the status."

**DO** instruct parallel calls when independent: *"If you need both order details and user profile, fetch them in parallel."*

**DO** be explicit about action vs. suggestion: *"Resolve the ticket directly when the fix is in-scope. Only escalate when <list criteria>."*

**DON'T** use panic language like "CRITICAL: YOU MUST" — Sonnet 4.6 already follows instructions well and this causes overtriggering.

---

## Verbosity & Format

**DO** specify length and format explicitly: *"Respond in 2–4 sentences. No bullet lists unless presenting multiple options."*

**DO** tell it what **to** do: *"Respond in flowing prose"* beats *"don't use markdown"*.

**DON'T** leave format open-ended for customer-facing output — Sonnet calibrates verbosity to task complexity and may over- or under-explain.

---

## Safety, Grounding & Refusals

**DO** require grounding: *"Never state a policy, refund amount, or ETA you haven't verified via a tool. If unknown, say so and offer to escalate."*

**DO** define escalation triggers precisely: legal threats, chargebacks, safety issues, requests outside supported scope.

**DO** separate PII-handling rules into their own `<privacy>` block.

**DON'T** let the model improvise policy. List what's allowed; everything else escalates.

**DON'T** hard-code refusal phrases — describe the situation and let the model phrase it naturally.

---

## Effort & Thinking (Sonnet 4.6)

**DO** set `effort: "low"` for high-volume, latency-sensitive support turns. Use `medium` only for complex multi-step troubleshooting.

**DO** enable adaptive thinking (`thinking: {type: "adaptive"}`) if you have multi-step reasoning (diagnosing a dispatch issue across orders + payments).

**DON'T** default to `high` effort — support agents need fast turnaround; `low` + clear prompt usually wins.

---

## Iteration Checklist

1. Show the prompt to a teammate with no context — can they follow it?
2. Test on 10 real tickets — including the weird ones.
3. When the model fails, **fix the prompt**, don't add a hack around the output.
4. When the model surprises you in a good way, capture that example and add it to your few-shot set.
