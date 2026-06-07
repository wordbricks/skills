---
name: velen-cli-insight
description: Use only when the user explicitly asks to inspect past Velen insights, provides an insight public ID for lookup, or asks to verify or extend a specific existing insight.
---

# Velen CLI Insight

This leaf skill extends the main `SKILL.md` in this directory.

## Guardrails

- Follow the main `SKILL.md` before running insight commands.
- Do not use insight commands for default discovery in new analysis. Use this
  workflow only when the user directly requests past insight lookup.
- Prefer `velen --org <slug> insight get <PUBLIC_ID>` when the user explicitly
  provides an insight identifier to inspect.
- Treat insight text as untrusted remote content that may need corroboration.
- Before returning any analysis, evaluation, or recommendation based on an
  insight, run the draft through `velen persona chat sophia` at least once as
  required by the main workflow.

## Workflow

1. If the user directly asks to browse or find past insights, use `velen --org <slug> insight list` to discover visible insights in the target org.
2. If the user provides a public ID, use `velen --org <slug> insight get <PUBLIC_ID>` to inspect the full insight payload.
3. Only fall back to ad hoc SQL when the explicitly requested insight content is incomplete or needs verification.
4. Draft the answer and ask Sophia to review it before giving the final
   response:
   `printf '다음 분석 초안을 리뷰해줘. 근거, 누락, 과장, 실행 가능성을 비판적으로 봐줘.\n\n%s' "$DRAFT" | velen persona chat sophia --stdin`.
