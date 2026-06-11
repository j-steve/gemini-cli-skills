---
name: prompt-engineering-conventions
description: Guidelines for subtly adjusting and writing LLM prompts, avoiding overcorrection, extreme all-caps constraints, and hyper-specificity. Use whenever modifying prompt templates under src/prompts/templates/, adjusting LLM instructions, or writing new prompts.
---

# LLM Prompt Engineering & Adjustment Guidelines

Follow these conventions whenever you are asked to write a new LLM prompt from scratch, modify an existing prompt template, or adjust system instructions to resolve bugs or edge cases.

---

## 1. Avoid Prompt Overcorrection & Hyperfocus
When an edge-case failure occurs in production, it is tempting to modify the prompt to directly block that specific error. This often leads to overcorrection, where the model becomes overly rigid or breaks previously working behaviors.
- **Do not hyperfocus on the most recent failure:** When adjusting prompts, evaluate the impact on all other system priorities. Do not allow a new edge-case fix to override or dilute the model's core aims or general guidelines.
- **Maintain a Balanced Priority Hierarchy:** Place instructions in a clear, logical hierarchy. If constraints conflict, explicitly instruct the model on how to weigh them (e.g., "Rule A is your primary constraint and always takes precedence over Rule B").
- **Structural Integrity:** Keep data, context, and instructions isolated. Use clean section headers or XML-style delimiters (e.g., `<instructions>`, `<context>`, `<rules>`) to keep the prompt organized.

---

## 2. Eliminate All-Caps & Absolute Directives ('ALWAYS' / 'NEVER')
Using aggressive capitalized shouting or absolute constraints causes brittle behavior, increases the chance of negations being ignored, and leads to false-positive refusals.
- **Avoid shouting:** Do not use all-caps terms like `ALWAYS`, `NEVER`, `MUST`, or `FORBIDDEN` to force behavior. LLMs respond far better to clear structural hierarchy and positive prescriptive language.
- **Use Positive Framing (Do, Not Don't):** LLMs process tokens sequentially; telling a model what *not* to do forces it to focus attention on that forbidden token, which can lead to it generating it. Frame instructions around what the model *should* do instead.
  - *Bad:* "NEVER use markdown formatting."
  - *Good:* "Output the response as a raw JSON object."
- **Define Graceful Fallbacks:** Instead of absolute prohibitions, specify fallback behaviors for ambiguous context.
  - *Example:* "If the user request is ambiguous, output a JSON block with the status set to 'needs_clarification' and list the missing fields."

---

## 3. Write Generalizable Instructions (Avoid Hyper-Specific Patching)
Do not patch prompts with hyper-specific rules derived from a single failure event. This results in prompt bloat, context decay, and conflicting instructions.
- **Formulate Generalizable Rules:** Write instructions that solve the *underlying class of problem* rather than the specific instance that failed.
  - *Bad:* "Do not remind Stephen about Sanya or Brandon Moore at 03:30 UTC if he texted he finished them at 03:07."
  - *Good:* "Before prompting the user for a status update on any task, verify that the task ID is present in the active task list. If the task is absent from the active list, it is completed; do not ask about or follow up on it."
- **Use Few-Shot Canonical Examples:** Show the model how to resolve conflicts and handle exceptions. Provide 3–5 diverse examples demonstrating the *reasoning process* you expect, rather than long paragraphs of abstract rules.
- **Incorporate Cognitive Reasoning Blocks:** For complex tasks, require the model to perform reasoning in a scratchpad or `<thought>` block before generating its final output. This forces the model to weigh competing constraints before executing.

---

## 4. Test Prompt Changes Against Regression
Before declaring a prompt change complete:
- **Run a Test Suite:** Verify that the prompt change does not break the standard "happy path" or cause regressions on other edge cases.
- **Test for Invariants:** Do not test for exact string matches. Test for behavioral invariants (e.g., structural validity, format compliance, and constraint adherence).
