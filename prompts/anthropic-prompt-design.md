<guide name="anthropic-prompt-design" target="Claude Sonnet 4.6 / Opus 4.7" use_case="customer support agent">

A one-pager for authoring system prompts for a customer support agent. Recent Claude models follow instructions literally — assume the model will not infer what you did not say.

<structure>
Organize the system prompt with XML tags so each section is unambiguous: `<role>`, `<tone>`, `<tools>`, `<policies>`, `<examples>`, `<output_format>`.

Set the role in one sentence: *"You are a customer support agent for Oway, a freight logistics platform. You help carriers and shippers resolve issues with orders, payments, and dispatch."*

Do not mix instructions, examples, and context in flowing prose — the model will conflate them.
</structure>

<clarity>
Write as if briefing a new hire with zero context. If a colleague would be confused, the model will be too.

State scope explicitly. *"Apply this tone to every response, not just the first turn."*

Explain the *why* behind rules. The model generalizes better:
> "Never share another user's order ID — it's PII and violates our privacy policy."

Replace vague qualifiers ("be helpful," "be professional," "handle appropriately") with concrete behavior. If it matters, say it.
</clarity>

<examples>
Include 3–5 examples inside `<example>` tags, covering the golden path **and** the edge cases (angry customer, ambiguous request, out-of-scope question). Mirror the real tone and vocabulary your users actually use. Happy-path-only examples produce brittle behavior on the cases that matter most.
</examples>

<tone>
Specify voice with positive instructions and a sample: *"Warm, concise, solutions-first. Acknowledge the issue in one line, then resolve."*

Match the prompt style to the desired output. A curt prompt produces curt responses.

Do not say *"don't be robotic"* — tell the model what to be instead. Positive instructions beat negative ones.

Do not assume the default tone matches your brand. Recent Claude models lean direct and light on emoji — prompt for warmth if you need it.
</tone>

<tool_use>
Describe **when** and **why** to call each tool, not just what it does:
> "Call `get_order_status` whenever the user references an order — even if they only mention a tracking number. Never guess the status."

Instruct parallel calls when independent: *"If you need both order details and user profile, fetch them in parallel."*

Be explicit about action vs. suggestion: *"Resolve the ticket directly when the fix is in-scope. Only escalate when [list criteria]."*

Do not use panic language like "CRITICAL: YOU MUST" — recent Claude models already follow instructions well, and this causes overtriggering.
</tool_use>

<verbosity_and_format>
Specify length and format explicitly: *"Respond in 2–4 sentences. No bullet lists unless presenting multiple options."*

Tell the model what **to** do: *"Respond in flowing prose"* beats *"don't use markdown"*.

Do not leave format open-ended for customer-facing output — the model calibrates verbosity to perceived task complexity and may over- or under-explain.
</verbosity_and_format>

<safety_and_grounding>
Require grounding: *"Never state a policy, refund amount, or ETA you haven't verified via a tool. If unknown, say so and offer to escalate."*

Define escalation triggers precisely: legal threats, chargebacks, safety issues, requests outside supported scope.

Separate PII-handling rules into a `<privacy>` block. List what is allowed; everything else escalates.

Do not hard-code refusal phrases — describe the situation and let the model phrase it naturally.
</safety_and_grounding>

<effort_and_thinking>
Set `effort: "low"` for high-volume, latency-sensitive support turns. Use `medium` for complex multi-step troubleshooting.

Enable adaptive thinking (`thinking: {type: "adaptive"}`) when reasoning spans multiple steps (e.g. diagnosing a dispatch issue across orders + payments).

Do not default to `high` effort — support agents need fast turnaround; `low` plus a clear prompt usually wins.
</effort_and_thinking>

<iteration_checklist>
1. Show the prompt to a teammate with no context — can they follow it?
2. Test against 10 real tickets, including the weird ones.
3. When the model fails, fix the prompt. Do not hack around the output.
4. When the model surprises you in a good way, capture that example and add it to your few-shot set.
</iteration_checklist>

</guide>
