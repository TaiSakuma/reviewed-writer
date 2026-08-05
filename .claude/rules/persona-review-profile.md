# Persona-review profile

Repository-specific values for the `reviewed-writer` plugin's `write-doc`
engine, invoked through `.claude/skills/write-docs/SKILL.md` and, for
report-only rounds, `.claude/skills/review-docs/SKILL.md`. The engine reads this
file first and refers to its sections by name; the section structure is shared
with the counterpart profiles in legendary-octo-happiness and
hypothesis-awkward. This repository consumes its own plugin.

## Document

The documents are `README.md` and `CONTRIBUTING.md` of `reviewed-writer`, a
Claude Code plugin for authoring documents through a panel of fixed-persona
reviewers; one document per run is the unit of work, and the section set is an
output of the run. Scoping note: the README is the plugin's public face — its
statement of the profile contract must stay in sync with the skills and
`.claude/CLAUDE.md`.

## Personas

The six review personas are the persona head files in `.claude/personas/`:
`ai.md`, `claude-plugin-expert.md`, `diataxis-expert.md`, `plugin-user.md`,
`evaluator.md`, and `maintainer.md`. Default primary personas per document — for
`README.md`: `plugin-user.md` and `evaluator.md`; for `CONTRIBUTING.md`: `ai.md`
and `plugin-user.md`, plus `maintainer.md` when the run touches the Releasing
sections — confirmed or overridden when the run is scoped (step 1).

## Declaration mechanism

Each section's Diátaxis quadrant is declared by the visible marker in its
heading, as `.claude/rules/diataxis-declaration.md` specifies (marker legend,
section shapes, record). The Diátaxis rules are the shared review core in the
`reviewed-writer` plugin's `persona-review` skill
(`references/diataxis-review.md` in its directory): the reader questions, the
per-quadrant guidance, the restructuring rules, and the reviewers' self-check.

Declarations travel with the text: each draft and the shipped documents declare
their own structure through their heading markers, and the review brief carries
each section's status and reader question, not its quadrant. Out-of-scope asks
and out-of-quadrant content are routed to the owning section of the same
document — or to the other document when it belongs there (README ↔
CONTRIBUTING) — creating or removing sections as the content requires.

## Premise to pin

Trigger: revising units that state the plugin's contract — the profile section
list, the component inventory, the invocation names. Premise: the contract as
the plugin's skill files define it. Authority: `skills/write-doc/SKILL.md`,
`skills/persona-review/SKILL.md`, and `agents/persona-reviewer.md` in the
working tree.

## Sources

`skills/write-doc/SKILL.md`; `skills/persona-review/SKILL.md`;
`skills/persona-review/references/diataxis-review.md`;
`agents/persona-reviewer.md`; `.claude-plugin/plugin.json` and
`.claude-plugin/marketplace.json`; `.github/workflows/*`; `.claude/CLAUDE.md`;
and the other document.

## Fact-check targets

For every behavioral claim: the skill, agent, manifest, and workflow files
listed under Sources. Every link resolves. The install example's `ref` matches
an existing release tag. Invocation names match the skills' frontmatter names.

## Status dimension

Disabled.

## Verification

No one-time wiring. Checks, re-run each review round:
`pre-commit run --all-files` passes (prettier; it covers only git-tracked files,
so `git add -N` new files first); local links resolve; the README's component
table matches `skills/` and `agents/`; the profile section list stated in the
README matches `.claude/CLAUDE.md` and the skills.

## Record

The shipped documents' heading markers are themselves the declaration record;
there is no table to sync. List every section-set change (added, split, merged,
removed, reclassified) in the run report.

## Voice rules

`.claude/rules/docs-voice.md`.

## Extra guidelines

- The README addresses consumers of the plugin; repository-internal concerns
  belong in `CONTRIBUTING.md` or `.claude/CLAUDE.md`.
- Reclassification — changing a section's declared quadrant while keeping its
  content — is a scoping decision (step 1, or the user's call between runs),
  never a review-round outcome. The declaration is the fixed point a round
  reviews against; reviewer remedies move content, not declarations.
- This repository consumes its own plugin pinned at a release in
  `.claude/settings.json`: reviews run with the pinned version's machinery, and
  skill-contract changes in the working tree reach reviews only after a release
  and a new pin.
