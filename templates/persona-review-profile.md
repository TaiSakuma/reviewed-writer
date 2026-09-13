<!--
Copy this file to `.claude/rules/persona-review-profile.md` in the consuming
repository and fill in every section. The `reviewed-writer` engine agent and
`persona-review` skill read the eleven `##` headings below by name: keep their
exact wording. Each body is prose an agent reads, not parsed fields, so state
the value in whatever form is clearest. Delete these comments once the section
is written.
-->

# Persona-review profile

Repository-specific values for the `reviewed-writer` plugin's
`diataxis-persona-engine` agent and `persona-review` skill. Both read this file
first and refer to its sections by name.

## Document

<!--
The document(s) the workflow authors, the unit of work per run, and whether the
section set is an output of the run (sections may be added, split, merged, or
removed) or fixed. Add any scoping notes a run must apply.
-->

<The document(s), the unit of work, and whether the section set is an output of
the run.>

## Personas

<!--
The persona head files, by path, and how the primary personas — whose verdicts
outweigh the others when fixes conflict — are chosen, per document if there are
several. One reviewer subagent is launched per file listed here.
-->

<The persona head files under `.claude/personas/`, and the primary personas per
document.>

## Declaration mechanism

<!--
How each unit of content declares its Diátaxis quadrant: point at
`.claude/rules/diataxis-declaration.md` for the marker legend, the section
shapes, and the record; say how declarations travel with drafts and what the
review brief carries; and name where out-of-scope asks and out-of-quadrant
content are routed (the owning section of the same document, or another
document).
-->

<How units declare their quadrant, and where out-of-scope content is routed.>

## Premise to pin

<!--
A premise every draft inherits and that must be settled before drafting: the
trigger that makes it apply, the premise itself, and the authority it is settled
against. Write `None.` if there is none.
-->

<Trigger, premise, and authority — or `None.`>

## Sources

<!-- The raw material a run collects before drafting: files, commands, URLs. -->

<The sources a run collects.>

## Fact-check targets

<!--
What every behavioral claim and code example is verified against, with any
checking notes (for example: every link resolves; version literals match a
release tag).
-->

<The targets claims are verified against, and checking notes.>

## Status dimension

<!--
`Disabled.` — or `Enabled.` plus the platform that spec content must be
implementable on. When enabled, every unit carries a status: implemented
(verified against the fact-check targets) or spec (verified against design
decisions supplied at invocation).
-->

Disabled.

## Verification

<!--
One-time wiring a run performs, if any, and the checks re-run every review
round (formatters, link checks, consistency checks between the document and the
repository).
-->

<One-time wiring, and the checks re-run each round.>

## Record

<!--
How a run is recorded: for example, the shipped document's own markers as the
declaration record, plus what the run report must list.
-->

<How a run is recorded.>

## Voice rules

<!-- The path of the voice-rules file, for example `.claude/rules/docs-voice.md`. -->

<The path of the voice-rules file.>

## Extra guidelines

<!-- Free-form additional rules the run applies. Write `None.` if there are none. -->

<Additional rules — or `None.`>
