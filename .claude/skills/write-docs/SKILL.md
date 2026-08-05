---
name: write-docs
description:
  Author or substantially revise README.md or CONTRIBUTING.md via the
  persona-review workflow
---

Invoke the `reviewed-writer:write-doc` skill. It reads
`.claude/rules/persona-review-profile.md`, whose Document section names
`README.md` and `CONTRIBUTING.md` and scopes each run to one of them; the
invoker names the document. The engine runs the full workflow — rubric, three
diverse drafts, persona panel, fact-check, synthesis, re-review until every
persona ships. Pass along the target document, the change driving the revision,
and any scoping the user supplied.
