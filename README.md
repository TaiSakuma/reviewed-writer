# reviewed-writer

A [Claude Code] plugin for authoring documents through a panel of fixed-persona
reviewers, with every unit of content declared in a [Diátaxis] quadrant.

## Skills and agent

| Component          | Kind  | Purpose                                                                                                                                                            |
| ------------------ | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `write-doc`        | skill | Author or substantially revise the repository's document: rubric, three diverse drafts, persona panel, fact-check, synthesis, re-review until every persona ships. |
| `persona-review`   | skill | Run one round of persona review — invoked by `write-doc` at its review steps, or standalone for a report-only round.                                               |
| `persona-reviewer` | agent | Reviews as one fixed persona supplied at the start of its task prompt; launched by `persona-review`, one per persona.                                              |

The Diátaxis review core (reader questions, per-quadrant guidance, restructuring
rules, and the reviewers' self-check) ships inside the `persona-review` skill at
`skills/persona-review/references/diataxis-review.md`.

Invocations are namespaced: `/reviewed-writer:write-doc` and
`/reviewed-writer:persona-review`. A consuming repository typically wraps them
in thin repository-local skills that keep its established entry points.

## What the consuming repository provides

The plugin is repository-agnostic; every repository-specific value lives in
files the consuming repository checks in:

- `.claude/rules/persona-review-profile.md` — the profile. Its `##` section
  headings are the interface the plugin reads by name: Document, Personas,
  Declaration mechanism, Premise to pin, Sources, Fact-check targets, Status
  dimension, Verification, Record, Voice rules, and Extra guidelines.
- `.claude/rules/diataxis-declaration.md` — how units of content declare their
  Diátaxis quadrants (marker syntax, section or page shapes, and the declaration
  record).
- Persona head files in `.claude/personas/`, listed in the profile's Personas
  section; each defines one reviewer's context, scope, goals, reading style,
  pain points, and lens.
- The voice-rules file the profile's Voice rules section names.

## Install

Add the marketplace and install the plugin:

```text
/plugin marketplace add TaiSakuma/reviewed-writer
/plugin install reviewed-writer@reviewed-writer
```

Or pin it in the repository's checked-in `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "reviewed-writer": {
      "source": {
        "source": "github",
        "repo": "TaiSakuma/reviewed-writer",
        "ref": "v0.1.0"
      }
    }
  },
  "enabledPlugins": { "reviewed-writer@reviewed-writer": true }
}
```

## Versioning

Releases are tagged `v<version>`, and `plugin.json` carries the matching semver
version. A consuming repository pins the marketplace source to a release tag;
upgrading is an edit to that pin.

Releases are cut by CI: pushing a `u<version>` trigger tag runs a pipeline that
generates `CHANGELOG.md` with [git-cliff] from Conventional-Commit PR titles,
creates the `v<version>` tag, and publishes the GitHub Release. A rolling
`latest` tag always points at the newest release; a consuming repository may pin
`"ref": "latest"` to follow releases automatically, at the cost of
reproducibility. The release runbook and the PR-title convention are in
[CONTRIBUTING.md](CONTRIBUTING.md).

## Provenance

Extracted from [legendary-octo-happiness] and [hypothesis-awkward], where the
workflow was converged over five refactoring iterations ([issue #49]).

[Claude Code]: https://claude.com/claude-code
[Diátaxis]: https://diataxis.fr/
[git-cliff]: https://git-cliff.org/
[legendary-octo-happiness]:
  https://github.com/TaiSakuma/legendary-octo-happiness
[hypothesis-awkward]: https://github.com/scikit-hep/hypothesis-awkward
[issue #49]: https://github.com/TaiSakuma/legendary-octo-happiness/issues/49
