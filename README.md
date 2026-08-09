# reviewed-writer

[![Latest release][release-badge]][releases]
[![License: MIT][license-badge]][license]

A [Claude Code] plugin for authoring documents through a panel of fixed-persona
reviewers, with every unit of content declared in one of the four [Diátaxis]
forms (the plugin's own files call them quadrants): tutorial, how-to, reference,
explanation. Contributions follow the PR-title convention in
[CONTRIBUTING.md][contributing].

## 📋 Skills and agent

| Component          | Kind  | Purpose                                                                                                                                               |
| ------------------ | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `write-doc`        | skill | Author or substantially revise a document the profile names: rubric, three diverse drafts by default, persona panel, fact-check, synthesis.           |
| `persona-review`   | skill | Run one round of persona review — invoked by `write-doc` at its review steps, or standalone for a report-only round.                                  |
| `persona-reviewer` | agent | Reviews as one fixed persona supplied at the start of its task prompt; launched by `persona-review`, one read-only reviewer per persona, in parallel. |

The re-review loop is bounded: `write-doc` iterates until every persona returns
a "ship" verdict, up to the run's re-review cap; if the cap is reached with
dissent remaining, the run stops and reports the unresolved verdicts. That cap
and the number of drafts default to five rounds and three drafts. An invocation
overrides either, and a repository that wants standing values states them in the
wrapper skill through which it invokes `write-doc`.

The plugin's Diátaxis review core (reader questions, per-quadrant guidance,
restructuring rules, and the reviewers' self-check) ships at
`${CLAUDE_PLUGIN_ROOT}/skills/persona-review/references/diataxis-review.md` — in
the source tree, `skills/persona-review/references/diataxis-review.md`.

Invocations are namespaced: `/reviewed-writer:write-doc` and
`/reviewed-writer:persona-review`. A repository's own wrapper skills in
`.claude/skills/`, when defined, are the entry points — this repository defines
`/write-docs` and `/review-docs`; the namespaced names stay available.

## 📋 What the consuming repository provides

The plugin is repository-agnostic; every repository-specific value lives in
files the consuming repository checks in, and the plugin's skills and reviewer
agent read the content files by name — never from the plugin's own directory:

| File                                                           | Supplies                                                                                                               |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `.claude/rules/persona-review-profile.md`                      | The profile; its eleven `##` headings are the contract (below).                                                        |
| `.claude/rules/diataxis-declaration.md`                        | Marker syntax for the four forms, the section or page shapes, the record.                                              |
| `.claude/personas/*.md` (for example `plugin-user.md`)         | One reviewer each: context, scope, goals, reading style, pain points, lens; the profile's Personas section lists them. |
| A voice-rules file (for example `.claude/rules/docs-voice.md`) | Editorial rules; the profile's Voice rules section names the path.                                                     |
| `.claude/settings.json`                                        | The checked-in marketplace pin and the `enabledPlugins` entry.                                                         |
| `.claude/skills/*` (optional; for example `write-docs`)        | Thin wrappers keeping the repository's established entry points.                                                       |

The first two rows are read at those literal paths — the skills read the profile
at `.claude/rules/persona-review-profile.md`, and the reviewer agent reads the
declaration file at `.claude/rules/diataxis-declaration.md`; the persona files
and the voice-rules file live wherever the profile names them.

The profile's eleven `##` headings, read by name — a missing or renamed heading
breaks the run:

- `Document` — the document(s) the workflow authors, the unit of work, and
  whether the section set is an output of the run.
- `Personas` — the persona head files, and how the primary personas (whose
  verdicts outweigh the others) are chosen.
- `Declaration mechanism` — how units declare their form (points at the
  declaration file), and where out-of-scope content is routed.
- `Premise to pin` — when a premise must be settled before drafting, what it is,
  and against which authority.
- `Sources` — the raw material a run collects.
- `Fact-check targets` — what claims are verified against, with checking notes.
- `Status dimension` — enabled or disabled; when enabled, units carry an
  implemented-or-spec status, and the section names the platform spec content
  must be implementable on.
- `Verification` — one-time wiring and the checks re-run every round.
- `Record` — how a run is recorded.
- `Voice rules` — names the voice-rules file.
- `Extra guidelines` — free-form additional rules.

The two provenance repositories, [legendary-octo-happiness] and
[hypothesis-awkward], are worked examples of a complete set. What this inventory
costs to author and maintain is stated under Why this workflow.

## 🔧 Install on one machine

To try the plugin on one machine — for a repository shared with collaborators,
pin instead (next section) — in the Claude Code prompt:

```text
/plugin marketplace add TaiSakuma/reviewed-writer
/plugin install reviewed-writer@reviewed-writer
```

Check the install summary: if it reports `Run /reload-plugins to activate.`, run
that command. The install prompts for a scope; choose user to keep the install
personal to this machine. Project scope writes the enablement into the
repository's checked-in `.claude/settings.json`, and local scope into your
gitignored `.claude/settings.local.json`. The route is unpinned: the marketplace
tracks the repository's default branch.

## 🔧 Pin the plugin for the repository

Add these keys to the repository's `.claude/settings.json` and commit the file,
merging into `extraKnownMarketplaces` and `enabledPlugins` when either key
already exists (create the file if it does not). Set `ref` to a release tag from
the [releases] page or to the rolling `latest` tag:

```json
{
  "extraKnownMarketplaces": {
    "reviewed-writer": {
      "source": {
        "source": "github",
        "repo": "TaiSakuma/reviewed-writer",
        "ref": "v0.2.0"
      }
    }
  },
  "enabledPlugins": { "reviewed-writer@reviewed-writer": true }
}
```

`extraKnownMarketplaces` declares the pinned source; `enabledPlugins` turns the
plugin on for this repository. The declaration alone installs nothing: Claude
Code offers to register the marketplace when a user trusts the repository
folder, and components load at the next session start (or after
`/reload-plugins`). A collaborator who trusted the folder before the pin was
added, or declined the offer, gets no further prompt; they register the declared
source explicitly:

```text
/plugin marketplace add TaiSakuma/reviewed-writer@v0.2.0
```

with the `@` suffix matching the checked-in `ref` — the bare
`TaiSakuma/reviewed-writer` source string registers the default branch instead
and overrides the pin. The same command also serves a machine that already
registered the marketplace — unpinned, or pinned at another tag: the add
overwrites the recorded source. The registration behavior in this section was
verified on Claude Code 2.1.221; the `@` pin suffix is the documented
`/plugin marketplace add` syntax for the `owner/repo` form.

## 🔧 Confirm and upgrade

After either route — at the next session start, or after `/reload-plugins` — the
plugin appears in `/plugin` as `reviewed-writer@reviewed-writer`, and
`/reviewed-writer:write-doc` becomes invocable. It does useful work only once
the checked-in files listed under What the consuming repository provides exist.
The running version is the one `/plugin` shows for the plugin.

To move a pinned repository to another release: edit `ref` to the new tag and
commit. The edit does not re-point machines that already registered the
marketplace — each such machine re-registers, with the new tag in the `@`
suffix, and the add overwrites the old registration:

```text
/plugin marketplace add TaiSakuma/reviewed-writer@<new-tag>
```

Components load at the next session start (or after `/reload-plugins`). A
machine that never registered the marketplace picks up the new pin through the
trust-prompt flow as usual. A repository pinned at `latest` skips this step: the
tag itself moves with releases, and each machine follows when its copy of the
marketplace refreshes — `/plugin marketplace update reviewed-writer`, or
automatically when auto-update is enabled for the marketplace (it is off by
default for marketplaces outside Anthropic's own).

## 📋 Versioning

Releases are tagged `v<version>` (for example `v0.2.0`), and
`.claude-plugin/plugin.json` carries the matching version (`0.2.0`); the
`u<version>` tags on the repository are CI triggers, not pin targets. The
rolling `latest` tag points at the highest released version. The release runbook
and the PR-title convention are in [CONTRIBUTING.md][contributing]; the
per-release record is [CHANGELOG.md][changelog].

The profile's path and its `##` section headings, the declaration file's path
and role, and the namespaced invocation names are the plugin's interface to a
consuming repository. Before 1.0, any release may change that interface; the
release notes record such changes, so moving a pin can oblige edits to the
checked-in files.

## 📖 Why this workflow

Documentation review by a single generic reviewer drifts toward style, and its
feedback changes with the reviewer. This plugin holds the reader fixed instead.
The reviewers are subagents that adopt persona definitions checked into the
repository — a deliberate proxy for the document's audience, not the audience
itself, complementing human review rather than replacing it. Each unit of
content declares its Diátaxis form, and the declaration is the fixed point of a
review: content is judged against the declared form, never the reverse, and an
ask that would pull a unit toward another form is routed to the unit that owns
that form. That discipline is what separates the workflow from the alternatives
it resembles: a docs linter checks structure and style and never asks whether
the reader's question was answered, and a Diátaxis structure maintained by hand
leaves the classification an intention that erodes — here it is a marker the
panel re-reads every round. Structurally distinct drafts precede the panel
because structure is the decision hardest to reverse once a text exists; the
orchestrator writes the final text, personas' wording is advisory, and accuracy
beats style.

The exchange is real. A consuming repository authors and maintains the profile,
the declaration rules, the persona head files, and the voice rules; they age
with the documents, and a persona nobody updates does not fail loudly — it keeps
shipping confident verdicts from a reader who no longer exists. A run's cost
scales with the panel, which is whatever the profile's Personas section lists.
Each round launches one read-only reviewer per persona, and a full `write-doc`
run adds up: a panel pass over the drafts, then re-review rounds — including any
spent on a flaw all the drafts share — up to the cap. At the default three
drafts and five rounds, with six personas, that is at most six panel rounds and
36 reviews for one document. Lowering the cap at invocation lowers that ceiling;
lowering the draft count lowers how much each reviewer reads, not how many
reviews run. The workflow fits documents revised deliberately for distinct
audiences; it is a poor fit for documentation that changes daily or a repository
unwilling to keep persona definitions current. The exit is bounded, though: the
profile, the declaration rules, the personas, the voice rules, and the documents
themselves all stay in the consuming repository, so dropping the plugin forfeits
the machinery, not the content.

## 📖 Provenance

The workflow was converged in use, not designed on paper: five refactoring
iterations ([issue #49]) across two repositories with little in common —
[legendary-octo-happiness], a reference implementation of changelog-and-release
automation, and [hypothesis-awkward], a [Scikit-HEP] project whose documentation
runs the workflow — preceded its extraction into this plugin. The two consuming
setups differ exactly where the profile lets them differ (five personas in one,
six in the other; a single README versus a growing page set), which is what the
convergence settled: the machinery is fixed, and everything repository-specific
lives in the checked-in files the profile names. This repository now consumes
its own plugin through the pin in its checked-in `.claude/settings.json`; the
README you are reading is an output of the workflow it describes.

[Claude Code]: https://claude.com/claude-code
[Diátaxis]: https://diataxis.fr/
[contributing]: CONTRIBUTING.md
[changelog]: CHANGELOG.md
[license]: LICENSE
[releases]: https://github.com/TaiSakuma/reviewed-writer/releases
[release-badge]:
  https://img.shields.io/github/v/release/TaiSakuma/reviewed-writer
[license-badge]: https://img.shields.io/github/license/TaiSakuma/reviewed-writer
[Scikit-HEP]: https://scikit-hep.org/
[legendary-octo-happiness]:
  https://github.com/TaiSakuma/legendary-octo-happiness
[hypothesis-awkward]: https://github.com/scikit-hep/hypothesis-awkward
[issue #49]: https://github.com/TaiSakuma/legendary-octo-happiness/issues/49
