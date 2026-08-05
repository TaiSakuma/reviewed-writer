# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## What this is

A Claude Code plugin (`reviewed-writer`) that authors documents through a panel
of fixed-persona reviewers, with every unit of content declared in a
[Diátaxis](https://diataxis.fr/) quadrant. The repository is also its own
single-plugin marketplace (`.claude-plugin/marketplace.json`). There is no build
system and there are no tests; everything is markdown, the release workflows
under `.github/`, and two JSON manifests.

## Commands

- `pre-commit run --all-files` — prettier over markdown. `.prettierrc.toml` sets
  `proseWrap = "always"`, so all markdown prose is hard-wrapped (at 80 columns).
  `.prettierignore` keeps prettier away from `CHANGELOG.md` (CI generates it)
  and `.github/` (the PR template must stay single-line).
- CI runs the same pre-commit check and validates PR titles against Conventional
  Commits (see `CONTRIBUTING.md`); there are no other checks.

## Architecture

Three components chain at runtime:

1. `skills/write-doc/SKILL.md` — the authoring orchestrator: scope → sources →
   rubric → structurally distinct drafts → persona panel → fact-check →
   synthesize → re-review until every persona ships (up to the re-review cap) →
   verify → record. The draft count and the cap are the skill's own, three and
   five; an invocation overrides either, and a consuming repository's wrapper
   skill in `.claude/skills/` is where a standing override lives. The profile
   carries neither — it holds the rules for each round, not how many rounds.
2. `skills/persona-review/SKILL.md` — one review round, invoked by `write-doc`
   at its review steps or standalone. It composes a self-contained review brief
   to a temp file, then launches one `persona-reviewer` subagent per persona in
   parallel and consolidates their reviews into a matrix.
3. `agents/persona-reviewer.md` — a read-only subagent that adopts the persona
   definition its task prompt opens with, reviews each unit against its declared
   quadrant, and returns a structured review with a ship/revise verdict.

The shared review core — reader questions, per-quadrant guidance, restructuring
rules (create/remove/split/merge/relocate/reclassify), and the reviewers'
three-pass self-check — is
`skills/persona-review/references/diataxis-review.md`. Skills reference it via
`${CLAUDE_PLUGIN_ROOT}` so the path resolves in a consuming repository.

## The profile is the interface

The plugin is repository-agnostic. All repository-specific values live in files
the consuming repository checks in, and the skills read them by name:

- `.claude/rules/persona-review-profile.md` — its `##` section headings are the
  contract: Document, Personas, Declaration mechanism, Premise to pin, Sources,
  Fact-check targets, Status dimension, Verification, Record, Voice rules, Extra
  guidelines. Renaming a heading or changing what a section supplies breaks
  consuming repositories.
- `.claude/rules/diataxis-declaration.md` — quadrant marker syntax, document
  shapes, and the declaration record.
- Persona head files in `.claude/personas/`, listed in the profile.

When editing the skills, keep this contract in sync across all three components
and the README, which document the same section list.

## Design invariants

These are load-bearing rules stated in the skill files; edits must not weaken
them:

- Reviewers depend only on the repository, this plugin, and the brief — the
  brief is self-contained.
- The declared quadrant is the fixed point of a review: content is judged
  against the marker, never the marker against the content. Reclassification is
  a scoping decision, not a review outcome.
- One quadrant per unit; out-of-quadrant content is relocated or routed, never
  polished in place.
- Accuracy beats style; the orchestrator writes the final text, personas'
  wording is advisory.

## Versioning

Releases are tagged `v<version>` and `plugin.json` carries the matching semver
version. The repo-local `/bump-version` skill bumps `plugin.json` and creates
the `u<version>` trigger tag; CI creates the matching `v<version>` tag and the
GitHub Release. Consuming repositories pin the marketplace source to a release
tag (or to the rolling `latest` tag).

## Releases

Releases use a two-tag flow; a release can be cut from any commit that already
carries the workflow files, not only `main`'s head:

1. Check out the commit to release, run the repo-local `/bump-version` skill
   (`patch`, `minor`, `major`, or an explicit version — it updates
   `.claude-plugin/plugin.json`, creates the bump commit, and creates the
   annotated `u<version>` tag), and push only that tag
   (`git push origin u<version>`).
2. The Changelog workflow (triggered by the `u` tag) verifies the tag against
   `plugin.json`'s version, creates a `release/<version>` branch at the tagged
   commit, generates `CHANGELOG.md` and the `v` tag on it, then merges the
   branch back into `main` when the release is on `main`'s line and its history
   already contains the newest existing release. Otherwise it is a backport: the
   merge-back is skipped with a warning, the branch is kept, and `main` is
   untouched.
3. The Release workflow (triggered via `workflow_run` after Changelog) creates a
   GitHub Release with GitHub auto-generated notes, marking it latest and moving
   the `latest` tag only when it is the newest version.

The pipeline assumes one release in progress at a time. The full runbook,
including recovery from a failed run, is in `CONTRIBUTING.md`.
