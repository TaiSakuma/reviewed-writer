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
| `write-doc`        | skill | Author or substantially revise a document the profile names: rubric, three structurally distinct drafts, persona panel, fact-check, synthesis.        |
| `persona-review`   | skill | Run one round of persona review — invoked by `write-doc` at its review steps, or standalone for a report-only round.                                  |
| `persona-reviewer` | agent | Reviews as one fixed persona supplied at the start of its task prompt; launched by `persona-review`, one read-only reviewer per persona, in parallel. |

The re-review loop is bounded: `write-doc` iterates until every persona returns
a "ship" verdict, capped at five rounds; if the cap is reached with dissent
remaining, the run stops and reports the unresolved verdicts.

The plugin's Diátaxis review core (reader questions, per-quadrant guidance,
restructuring rules, and the reviewers' self-check) ships at
`${CLAUDE_PLUGIN_ROOT}/skills/persona-review/references/diataxis-review.md` — in
this repository, `skills/persona-review/references/diataxis-review.md`.

Invocations are namespaced: `/reviewed-writer:write-doc` and
`/reviewed-writer:persona-review`. A repository's own wrapper skills in
`.claude/skills/`, when defined, are the entry points — this repository defines
`/write-docs` and `/review-docs`; the namespaced names are the fallback.

## 📋 What the consuming repository provides

The plugin is repository-agnostic; every repository-specific value lives in
files the consuming repository checks in, and the skills read the content files
by name — never from the plugin's own directory:

| File                                                           | Supplies                                                                                                               |
| -------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `.claude/rules/persona-review-profile.md`                      | The profile; its eleven `##` headings are the contract (below).                                                        |
| `.claude/rules/diataxis-declaration.md`                        | Marker syntax for the four forms, the section or page shapes, the record.                                              |
| `.claude/personas/*.md` (for example `plugin-user.md`)         | One reviewer each: context, scope, goals, reading style, pain points, lens; the profile's Personas section lists them. |
| A voice-rules file (for example `.claude/rules/docs-voice.md`) | Editorial rules; the profile's Voice rules section names the path.                                                     |
| `.claude/settings.json`                                        | The checked-in marketplace pin and the `enabledPlugins` entry.                                                         |
| `.claude/skills/*` (optional; for example `write-docs`)        | Thin wrappers keeping the repository's established entry points.                                                       |

The profile's eleven `##` headings, read by name — a missing or renamed heading
breaks the run:

- Document — the document(s) the workflow authors, the unit of work, and whether
  the section set is an output of the run.
- Personas — the persona head files, and how the primary personas (whose
  verdicts outweigh the others) are chosen.
- Declaration mechanism — how units declare their form (points at the
  declaration file), and where out-of-scope content is routed.
- Premise to pin — when a premise must be settled before drafting, what it is,
  and against which authority.
- Sources — the raw material a run collects.
- Fact-check targets — what claims are verified against, with checking notes.
- Status dimension — enabled or disabled; when enabled, units carry an
  implemented-or-spec status, judged against a named platform.
- Verification — one-time wiring and the checks re-run every round.
- Record — how a run is recorded.
- Voice rules — names the voice-rules file.
- Extra guidelines — free-form additional rules.

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

The install prompts for a scope; choose user to keep the install personal to
this machine. Project scope writes the enablement into the repository's
checked-in `.claude/settings.json`, and local scope into your gitignored
`.claude/settings.local.json`. The route is unpinned: the marketplace tracks the
repository's default branch.

## 🔧 Pin the plugin for the repository

Add these keys to the repository's `.claude/settings.json`, merging into
`extraKnownMarketplaces` and `enabledPlugins` when either key already exists
(create the file if it does not). Set `ref` to a release tag — replace `v0.1.1`
below with the newest tag on the [releases] page:

```json
{
  "extraKnownMarketplaces": {
    "reviewed-writer": {
      "source": {
        "source": "github",
        "repo": "TaiSakuma/reviewed-writer",
        "ref": "v0.1.1"
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
added sees no prompt; they register the declared source explicitly:

```text
/plugin marketplace add TaiSakuma/reviewed-writer#v0.1.1
```

with the `#` suffix matching the checked-in `ref` — the bare
`TaiSakuma/reviewed-writer` source string registers the default branch instead
and overrides the pin. The same command also serves a machine that already
registered the marketplace — unpinned, or pinned at another tag: the add
overwrites the recorded source. The registration behavior in this section was
verified on Claude Code 2.1.221.

## 🔧 Confirm and upgrade

After either route, the plugin appears in `/plugin` as
`reviewed-writer@reviewed-writer`, and `/reviewed-writer:write-doc` becomes
invocable. It does useful work only once the checked-in files listed under What
the consuming repository provides exist. The running version is the one
`/plugin` shows for the plugin; the cache directories (see Versioning) only show
which versions a machine has fetched.

To upgrade a pinned repository: check the [release notes][releases] for
interface changes (see Versioning), edit `ref` to the new release tag, and
commit. The edit does not re-point machines that already registered the
marketplace — each such machine re-registers, with the new tag in the `#` suffix
(`v0.2.0` stands for it below); the add overwrites the old registration:

```text
/plugin marketplace add TaiSakuma/reviewed-writer#v0.2.0
```

Components load at the next session start (or after `/reload-plugins`). A
machine that never registered the marketplace picks up the new pin through the
trust-prompt flow as usual.

## 📋 Versioning

Releases are tagged `v<version>` (for example `v0.1.1`), and
`.claude-plugin/plugin.json` carries the matching version (`0.1.1`). CI cuts
releases from `u<version>` trigger tags (for example `u0.1.1`) — `u` tags are
triggers, not pin targets. The release runbook and the PR-title convention are
in [CONTRIBUTING.md][contributing]; the per-release record is
[CHANGELOG.md][changelog]. The plugin is MIT-licensed ([LICENSE][license]).

An installed plugin runs from a cached clone under
`~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/` — for this release,
`~/.claude/plugins/cache/reviewed-writer/reviewed-writer/0.1.1/`. The cache
keeps one directory per fetched version.

The rolling `latest` tag points at the highest released version — a backport
does not move it. A consuming repository may pin `"ref": "latest"` to follow
releases automatically, at the cost of reproducibility: the follow happens at
each user's next marketplace refresh, so collaborators can run different
versions from the same checked-in file until they refresh.

The profile's `##` section headings, the declaration file's role, and the
namespaced invocation names are the plugin's interface to a consuming
repository. Before 1.0, any release may change that interface; the release notes
record such changes, so moving a pin can oblige edits to the checked-in files.

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
it resembles: a docs linter checks structure and style and never asks whether a
reader was informed, and Diátaxis applied by hand leaves the classification an
intention that erodes — here it is a marker the panel re-reads every round.
Three structurally distinct drafts precede the panel because structure is the
decision hardest to reverse once a text exists; the orchestrator writes the
final text, personas' wording is advisory, and accuracy beats style.

The exchange is real. A consuming repository authors and maintains the profile,
the declaration rules, the persona head files, and the voice rules; they age
with the documents, and a persona nobody updates does not fail loudly — it keeps
shipping confident verdicts from a reader who no longer exists. A run's cost
scales with the panel, which is whatever the profile's Personas section lists.
Each round launches one read-only reviewer per persona, and a full `write-doc`
run adds up: a panel pass over the three drafts, one or two extra passes when
the drafts share a flaw, and a re-review loop capped at five rounds — with six
personas, up to roughly fifty reviews for one document. The workflow fits
documents revised deliberately for distinct audiences; it is a poor fit for
documentation that changes daily or a repository unwilling to keep persona
definitions current. The exit is bounded, though: the profile, the declaration
rules, the personas, the voice rules, and the documents themselves all stay in
the consuming repository, so dropping the plugin forfeits the machinery, not the
content.

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
