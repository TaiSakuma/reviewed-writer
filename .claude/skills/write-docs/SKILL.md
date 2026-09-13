---
name: write-docs
description:
  Author or substantially revise README.md or CONTRIBUTING.md via the
  persona-review workflow
argument-hint: "[README.md|CONTRIBUTING.md]"
---

Invoke the `reviewed-writer:write-doc` skill. It drives the
`reviewed-writer:diataxis-writer` and `reviewed-writer:diataxis-persona-panel`
agents, which read `.claude/rules/persona-review-profile.md`, whose Document
section names `README.md` and `CONTRIBUTING.md` and scopes each run to one of
them; the invoker names the document. The writer runs the authoring workflow —
rubric, diverse drafts, fact-check, synthesis, revision — the panel runs a
persona round on each text, and `write-doc` iterates until every persona ships
and relays the outcomes. Pass along the target document, the change driving the
revision, and any scoping the user supplied.
