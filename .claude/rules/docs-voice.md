---
paths:
  - "README.md"
  - "CONTRIBUTING.md"
---

# Documentation voice

## Voice

The audience is international developers — maintainers of consuming repositories
and contributors — reading `README.md` and `CONTRIBUTING.md`. These rules apply
to the documents' prose, not to code, manifests, or commit messages.

- No slang; no idioms or figures of speech that mainly native English speakers
  use (no sports, military, or cooking metaphors); prefer literal expressions
  over figurative ones. Established technical terms ("pipeline", "trigger",
  "squash", "pin") are fine — the rule targets decorative figures of speech, not
  standard vocabulary.
- Prefer precise words: exact file paths, invocation names, tag patterns,
  settings keys. But do not invent precision, and avoid literal values that go
  stale when a stable reference exists.
- State things directly, without hedging. Calibrated uncertainty is information,
  not hedging; do not upgrade uncertain claims to fact.
- Spell out each acronym on first use, except those the audience knows better as
  the short form (URL, API, CI, PR).
- Explain ideas in your own words rather than lightly rewording a source; put
  reused external wording in quotation marks, attribute it, and link the source.
  Verbatim reproduction is correct where exact wording is the point — commands,
  tag patterns, section-heading names, invocation names — a paraphrased name is
  a defect, not a style choice.

## Conventions

- Reference-style links (`[text][label]`), with definitions grouped at the end
  of the document.
- Prose hard-wrapped at 80 columns; pipe-aligned tables; fenced code blocks
  specify a language (MD040). All three are enforced by prettier via pre-commit.
- A concrete example alongside every pattern (`reviewed-writer@reviewed-writer`
  for `<plugin>@<marketplace>`), with a consistent placeholder style within a
  section.
- Section headings carry their Diátaxis marker per
  `.claude/rules/diataxis-declaration.md`; container headings and document front
  matter carry none.

## During persona review

Persona-suggested wording is advisory; the orchestrator writes the final text
itself. A persona ask that conflicts with this file is resolved in favor of this
file unless the user rules otherwise.
