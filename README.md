# reviewed-writer

[![Latest release][release-badge]][releases]
[![License: MIT][license-badge]][license]

A [Claude Code] plugin for authoring documents through a panel of fixed-persona
reviewers, with every unit of content declared in one of the four [Diátaxis]
forms (the plugin's own files call them quadrants): tutorial, how-to guide,
reference, explanation. The plugin holds the machinery and nothing
repository-specific: a consuming repository checks in the files naming its
documents, its reviewers, its declaration rules, and its editorial rules, and
the plugin ships a template for each. Contributions follow the PR-title
convention in [CONTRIBUTING.md][contributing].

Requires Claude Code 2.1.269 or newer.

## 📋 What the plugin ships

| Component                | Kind  | Purpose                                                                    |
| ------------------------ | ----- | -------------------------------------------------------------------------- |
| `write-doc`              | skill | Drives a run and relays each outcome to you.                               |
| `diataxis-writer`        | agent | Writes the document, from scoping to the run's record.                     |
| `diataxis-persona-panel` | agent | Runs one review round per call, over the drafts or the document.           |
| `persona-review`         | skill | Runs one review round, for the panel or standalone as a report-only round. |
| `persona-reviewer`       | agent | Reviews as one fixed persona, read-only.                                   |
| `templates/`             | files | Four skeletons for the files a consuming repository checks in.             |

In a run, the writer drafts, the panel reviews the drafts, the writer
synthesizes, and review and revision then alternate until every persona returns
a "ship" verdict; if the re-review cap is reached with dissent remaining, the
run stops and reports the unresolved verdicts. Three drafts, a cap of five, the
writer `reviewed-writer:diataxis-writer`, and the panel
`reviewed-writer:diataxis-persona-panel` are the defaults, each overridden at
the invocation or, for standing values, in a repository's wrapper skill.

Invocations are namespaced — `/reviewed-writer:write-doc` and
`/reviewed-writer:persona-review` — and stay available alongside a repository's
own wrapper skills in `.claude/skills/`, which are its entry points when
defined; this repository defines `/write-docs` and `/review-docs`.

## Install

A pin checked into a repository shared with collaborators, an unpinned install
on one machine, and what it takes to move a pin to a new release.

### 🔧 Pin the plugin for the repository

Add these keys to the repository's `.claude/settings.json` and commit the file,
merging into `extraKnownMarketplaces` and `enabledPlugins` when either key
already exists. Set `ref` to a release tag from the [releases] page, which pins
one version, or to the rolling `latest` tag:

```json
{
  "extraKnownMarketplaces": {
    "reviewed-writer": {
      "source": {
        "source": "github",
        "repo": "TaiSakuma/reviewed-writer",
        "ref": "v0.5.0"
      }
    }
  },
  "enabledPlugins": { "reviewed-writer@reviewed-writer": true }
}
```

`extraKnownMarketplaces` declares the pinned source; `enabledPlugins` turns the
plugin on for this repository. Installing stays per machine: with the folder
trusted, Claude Code reports the plugin as not installed and shows the command
each collaborator runs.

### 🔧 Install on one machine

Without a pin — one machine, no repository to share — run both commands in the
Claude Code prompt:

```text
/plugin marketplace add TaiSakuma/reviewed-writer
/plugin install reviewed-writer@reviewed-writer
```

`reviewed-writer@reviewed-writer` is the `<plugin>@<marketplace>` pattern; both
halves are the same word because this repository is a marketplace hosting this
one plugin. The route is unpinned: the marketplace tracks the repository's
default branch, and the registration replaces the machine's `reviewed-writer`
source, so a repository's pin no longer governs it.

### 🔧 Move the pin to a new release

Edit `ref` to the new tag and commit it; then, on every machine that already
registered the marketplace, re-register it at the new tag, update the installed
plugin, and reload.

## 🔧 Set up a consuming repository

With the plugin installed — [pinned][pin] and installed, or installed on one
machine — run `/reviewed-writer:persona-review` in the repository before the
profile, the declaration file, the persona head files, and the voice rules
exist. The round stops at its preflight, names the missing file and the template
that supplies it, and the session copies that template into place and fills it
in with you; repeat until the round runs and returns one verdict per persona.

Four decisions are yours: the document to review, the reader each persona head
file stands for, the editorial rules the prose follows, and the Diátaxis marker
each `##` heading of the document carries. [What the consuming repository
provides][provides] states what each file must contain. Commit the filled-in
files; `/reviewed-writer:write-doc` then authors or revises the document.

## 📋 What the consuming repository provides

The writer and panel agents, the `persona-review` skill, and the reviewer agent
read the first four files by name from the consuming repository, never from the
plugin's own directory: the first two at the literal paths below, the others
wherever the profile names them. The last two configure Claude Code itself.

| File                                                            | Template it starts from     | What it must contain                                                                                             |
| --------------------------------------------------------------- | --------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `.claude/rules/persona-review-profile.md`                       | `persona-review-profile.md` | The `##` headings below, each with a prose body.                                                                 |
| `.claude/rules/diataxis-declaration.md`                         | `diataxis-declaration.md`   | The marker legend, the marker placement, the section shapes, the record.                                         |
| `.claude/personas/<reader>.md` by convention, one per reviewer  | `persona.md`                | One reader, in the second person: opening sentence, the reader's own question, six parts, flags sentence.        |
| The voice-rules file, for example `.claude/rules/docs-voice.md` | `docs-voice.md`             | The editorial rules the final text follows.                                                                      |
| `.claude/settings.json`                                         | —                           | The pinned marketplace source and the `enabledPlugins` entry, as under [Pin the plugin for the repository][pin]. |
| `.claude/skills/<skill>/SKILL.md` (optional)                    | —                           | A wrapper that invokes a namespaced skill.                                                                       |

The profile's eleven `##` headings are read by name, so a missing or renamed
heading stops a run. Each body is prose an agent reads, not a parsed field.

| Heading                 | Body supplies                                                                                                                                                                                         |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Document`              | The document(s), the unit of work per run, whether the section set is an output of the run, and any scoping notes a run must apply.                                                                   |
| `Personas`              | The persona head files, and how the primary personas — whose verdicts outweigh the others — are chosen.                                                                                               |
| `Declaration mechanism` | How units declare their form (it points at the declaration file), and where out-of-scope content is routed.                                                                                           |
| `Premise to pin`        | The trigger, the premise to settle before drafting, and the authority it is settled against.                                                                                                          |
| `Sources`               | The raw material a run collects.                                                                                                                                                                      |
| `Fact-check targets`    | What every claim is verified against, with checking notes.                                                                                                                                            |
| `Status dimension`      | `Disabled.`, or `Enabled.` plus the platform spec content must be implementable on; enabled when each unit is labeled implemented (describes current behavior) or spec (describes intended behavior). |
| `Verification`          | One-time wiring, and the checks re-run every review round.                                                                                                                                            |
| `Record`                | How a run is recorded.                                                                                                                                                                                |
| `Voice rules`           | The path of the voice-rules file.                                                                                                                                                                     |
| `Extra guidelines`      | Free-form additional rules.                                                                                                                                                                           |

Worked examples, each a filled-in set: this repository's own [`.claude/`
directory][own-claude-dir] in the source tree, [legendary-octo-happiness], and
[hypothesis-awkward].

## 📋 Versioning

Releases are tagged `v<version>` (for example `v0.5.0`), and
`.claude-plugin/plugin.json` carries the matching version (`0.5.0`). The
`u<version>` tags on the repository (for example `u0.5.0`) are CI triggers, not
pin targets. The rolling `latest` tag points at the highest released version.
The release runbook and the PR-title convention are in
[CONTRIBUTING.md][contributing], under Releasing and PR Title Convention; the
per-release record is [CHANGELOG.md][changelog].

The profile's path and its `##` section headings, the declaration file's path
and role, and the namespaced invocation names are the plugin's interface to a
consuming repository. Before 1.0, any release may change that interface, and
moving a pin can oblige edits to the checked-in files. Each release's notes on
the [releases] page list the pull requests merged since the previous release,
where an interface change appears under its title. A renamed profile heading
stops the next run before any reviewer launches, and the run names the heading.
A change in what a section must supply stops nothing; the release notes are
where it is announced.

## 📖 Why this workflow

Documentation review by a single generic reviewer drifts toward style, and its
feedback changes with the reviewer. This plugin holds the reader fixed: the
reviewers are subagents adopting persona definitions checked into the repository
— a deliberate proxy for the document's audience, not the audience itself,
complementing human review rather than replacing it. The reviews are model
output: two runs over the same text return different reviews, and a unanimous
"ship" records that the panel's questions were answered, not that the document
is good.

Each Diátaxis form answers a different question its reader arrives with, and a
unit that tries to answer two answers neither well. So each unit declares its
form, and that declaration is the fixed point of a review: content is judged
against the declared form, never the reverse, and an ask that would pull a unit
toward another form is routed to the unit that owns that form. That discipline
separates the workflow from what it resembles. A docs linter checks structure
and style, never whether the reader's question was answered. A hand-maintained
classification is an intention that erodes, as it had in
[legendary-octo-happiness], one of the two repositories the workflow came from;
here it is a marker the panel re-reads every round. Structurally distinct drafts
precede the panel because structure is the decision hardest to reverse once a
text exists. The writer agent writes the final text, personas' wording is
advisory, and accuracy beats style.

The exchange is real. A consuming repository authors and maintains the profile,
the declaration rules, the persona head files, and the voice rules; the
templates shorten the first pass, not the upkeep. They age with the documents,
and a persona nobody updates does not fail loudly — it keeps shipping confident
verdicts from a reader who no longer exists.

A run's cost scales with the panel `Personas` lists. One review is one subagent
reading its persona head file, the run's brief, and the text under review — the
whole document, or every draft in the draft round, each written in full before
the panel sees it. The panel runs in parallel. Later rounds continue the same
reviewers with what changed — a full report on the synthesized document, then a
re-read and a short reply per persona — rather than a fresh review each time,
though a continued reviewer's context grows with each round. A run adds the
draft round to the re-review rounds, up to the cap: at the default three drafts
and cap of five, with six personas, at most six panel rounds — six full reviews
of the drafts, six full reports on the synthesized text, and up to 24 follow-ups
— for one document; a panel of one is valid and costs a sixth of that. Lowering
the cap at invocation lowers the ceiling; lowering the draft count lowers how
much each reviewer reads, not how many reviews run. The checked-in files serve
the whole repository, while a run covers the document `Document` sets as its
unit of work, so the setup is authored once and the run cost repeats for each
document revised.

The workflow fits documents revised deliberately for distinct audiences; it is a
poor fit for documentation that changes daily, or a repository unwilling to keep
persona definitions current. A run is driven from a session: it settles scope
with you and hands unresolved dissent back to you, so it is an authoring step,
not a check that can gate a pull request. The exit is bounded: the profile, the
declaration rules, the personas, the voice rules, and the documents stay in the
consuming repository, so dropping the plugin forfeits the machinery, not the
content.

## 📖 Provenance

The workflow was converged in use, not designed on paper: five refactoring
iterations ([issue #49]) across two repositories with little in common —
[legendary-octo-happiness], a reference implementation of changelog-and-release
automation, and [hypothesis-awkward], a [Scikit-HEP] project whose documentation
runs the workflow — the last of them the extraction into this plugin. The two
setups differ exactly where the profile lets them differ: five personas in one,
six in the other; a single README versus a growing page set. That is what the
convergence settled — the machinery is fixed, and everything repository-specific
lives in the checked-in files the profile names. This repository now consumes
its own plugin through the pin in its `.claude/settings.json`; the README you
are reading is an output of the workflow it describes.

[Claude Code]: https://claude.com/claude-code
[Diátaxis]: https://diataxis.fr/
[contributing]: CONTRIBUTING.md
[changelog]: CHANGELOG.md
[license]: LICENSE
[own-claude-dir]: .claude/
[releases]: https://github.com/TaiSakuma/reviewed-writer/releases
[release-badge]:
  https://img.shields.io/github/v/release/TaiSakuma/reviewed-writer
[license-badge]: https://img.shields.io/github/license/TaiSakuma/reviewed-writer
[Scikit-HEP]: https://scikit-hep.org/
[legendary-octo-happiness]:
  https://github.com/TaiSakuma/legendary-octo-happiness
[hypothesis-awkward]: https://github.com/scikit-hep/hypothesis-awkward
[issue #49]: https://github.com/TaiSakuma/legendary-octo-happiness/issues/49
[pin]: #-pin-the-plugin-for-the-repository
[provides]: #-what-the-consuming-repository-provides
