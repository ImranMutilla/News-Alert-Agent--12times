# Merge-ready workflow bundle (replay)

This bundle is a replay of the latest Codex adjustments so you can merge cleanly.

Included files:
- `workflows/codex-workflow-a-daily-newsletter-merge-ready.json`
- `workflows/codex-workflow-b-content-library-merge-ready.json`
- `workflows/codex-workflow-c-web-platform-merge-ready.json`

Included adjustments from prior two rounds:
1. Updated long-form prompts for Workflow A and Workflow B.
2. A->B wiring (`Trigger Codex Workflow B`) so Workflow B runs after newsletter save.
3. Workflow B reads today's newsletter (`Read Today Newsletter`) and builds `news_context` for content generation.
4. AI nodes switched to `n8n-nodes-base.openAi` with `openAiApi` credentials (to avoid unknown node type).
5. Parsers accept OpenAI response shape (`choices[0].message.content`).
6. Workflow C stays template-rendering based (no repeated AI generation).

If your previous branch had conflicts, you can merge these `*-merge-ready.json` files directly without touching existing files.
