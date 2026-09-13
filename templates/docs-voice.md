---
paths:
  - "<document path>"
---

<!--
Copy this file to the path the profile's Voice rules section names, for example
`.claude/rules/docs-voice.md`. The `write-doc` run's engine applies these rules when it writes the final text; persona wording is advisory against them. The `paths:`
frontmatter above is a Claude Code rule scope and is optional: it loads the file
into sessions that touch the listed documents, and it must stay at the top of
the file. Delete these comments once the rules are written.
-->

# Documentation voice

## Voice

<!-- Who the audience is, and the editorial rules the prose follows. -->

The audience is <who reads the documents>. These rules apply to the documents'
prose, not to code, manifests, or commit messages.

- <Rule — for example: no slang, idioms, or figures of speech; prefer literal
  expressions.>
- <Rule — for example: precise names, exact file paths, invocation names, and
  settings keys, without inventing precision.>
- <Rule — for example: state things directly; calibrated uncertainty is
  information, hedging is not.>

## Conventions

<!-- Formatting conventions a run must keep, and which of them a tool enforces. -->

- <Convention — for example: reference-style links, with definitions grouped at
  the end of the document.>
- <Convention — for example: prose hard-wrapped at 80 columns, enforced by
  prettier via pre-commit.>
- Section headings carry their Diátaxis marker per
  `.claude/rules/diataxis-declaration.md`; container headings and document front
  matter carry none.

## During persona review

Persona-suggested wording is advisory; the engine writes the final text itself.
A persona ask that conflicts with this file is resolved in favor of this file
unless the user rules otherwise.
