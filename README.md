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

## 📋 What the plugin ships

| Component                | Kind  | Purpose                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `write-doc`              | skill | Drives a writer agent and a review-panel agent: the writer drafts, the panel reviews the drafts, the writer synthesizes, then review and revision alternate until the panel approves, up to the run's re-review cap; owns the run's numbers and counts the rounds, relays each outcome to you, and passes paths it never opens; knows nothing of the document, the profile, or the personas. |
| `diataxis-writer`        | agent | The writer `write-doc` launches and continues: reads the profile, runs the preflight, scopes the run (its open questions reach you through `write-doc`), writes structurally distinct drafts, fact-checks, synthesizes from the panel's matrix, revises, and records; launches nothing.                                                                                                      |
| `diataxis-persona-panel` | agent | The panel `write-doc` launches and continues: one review round per call over the drafts or the document, run through `persona-review` with the same persona reviewers continued across rounds; needs subagent nesting, on by default since Claude Code 2.1.219.                                                                                                                              |
| `persona-review`         | skill | Runs one review round — invoked by the panel agent at each round, or standalone for a report-only round.                                                                                                                                                                                                                                                                                     |
| `persona-reviewer`       | agent | Reviews as one fixed persona whose head file its task prompt names first; launched by `persona-review` in a run's first round, one read-only reviewer per persona, in parallel, and continued with what changed in each later round; the plugin pins no model, so a reviewer runs on the model the launching session gives its subagents.                                                    |
| `templates/`             | files | Four skeletons: the profile, the declaration file, a persona head file, the voice rules.                                                                                                                                                                                                                                                                                                     |

Invocations are namespaced: `/reviewed-writer:write-doc` and
`/reviewed-writer:persona-review`. A repository's own wrapper skills in
`.claude/skills/`, when defined, are its entry points — this repository defines
`/write-docs` and `/review-docs` — and the namespaced names stay available.

`write-doc` iterates until every persona returns a "ship" verdict, up to the
run's re-review cap; if the cap is reached with dissent remaining, the run stops
and reports the unresolved verdicts. The cap, the draft count, the writer, and
the panel default to five rounds, three drafts,
`reviewed-writer:diataxis-writer`, and `reviewed-writer:diataxis-persona-panel`;
an invocation overrides any of them, and a repository with standing values
states them in its wrapper skill.

The Diátaxis review core — the reader questions, the per-quadrant guidance, the
restructuring rules, and the reviewers' self-check — is
`${CLAUDE_PLUGIN_ROOT}/skills/persona-review/references/diataxis-review.md`.
`${CLAUDE_PLUGIN_ROOT}` is Claude Code's placeholder for the plugin's
installation directory; it resolves inside the plugin's own skill and agent
files, not in text you type at the prompt.

The four templates in the plugin's `templates/` directory start the files a
consuming repository checks in: `persona-review-profile.md` for the profile,
`diataxis-declaration.md` for the declaration file, `persona.md` for one persona
head file, and `docs-voice.md` for the voice rules. Each is the finished shape
of its file, with angle-bracket placeholders for the repository-specific values
and comments stating what each part supplies; the placeholders are replaced and
the comments deleted as the file is written. Copying a template and filling it
in is how each file is produced; the ordered path is under Set up a consuming
repository.

## 🔧 Install on one machine

To try the plugin on one machine — for a repository shared with collaborators,
pin it instead, as Pin the plugin for the repository describes — run both
commands in the Claude Code prompt:

```text
/plugin marketplace add TaiSakuma/reviewed-writer
/plugin install reviewed-writer@reviewed-writer
```

`reviewed-writer@reviewed-writer` is the `<plugin>@<marketplace>` pattern; both
halves are the same word because this repository is a marketplace hosting this
one plugin. The install prompts for a scope: user installs it for you across all
your projects, project writes the enablement into the repository's checked-in
`.claude/settings.json` for all collaborators, each of whom still installs the
plugin on their own machine, and local installs it for you in this repository
only, through `.claude/settings.local.json`. Then read the install summary as
Confirm the install describes. This route is unpinned: the marketplace tracks
the repository's default branch. A GitHub `owner/repo` source clones over SSH;
if the add fails to clone, set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` in the shell
before starting Claude Code, then run the add again.

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
        "ref": "v0.5.0"
      }
    }
  },
  "enabledPlugins": { "reviewed-writer@reviewed-writer": true }
}
```

`extraKnownMarketplaces` declares the pinned source; `enabledPlugins` turns the
plugin on for this repository. Once a collaborator trusts the repository folder,
Claude Code adds the declared marketplace without a further prompt — the same
GitHub source, so the SSH clone note under Install on one machine applies,
except that the remedy here is to set `CLAUDE_CODE_PLUGIN_PREFER_HTTPS=1` and
start Claude Code again, since Claude Code makes this add itself — but adding a
marketplace no longer installs its plugins: since Claude Code 2.1.195, a plugin
from an external source such as a GitHub repository, enabled only by the
project's `.claude/settings.json`, does not load until the collaborator installs
it. Until then Claude Code reports the plugin as not installed and shows the
command to run:

```text
claude plugin install reviewed-writer@reviewed-writer
```

The shell form installs at user scope unless `--scope` is passed (the three
scopes are described under Install on one machine);
`/plugin install reviewed-writer@reviewed-writer` inside a session prompts for
the scope instead. Note which scope the install used: moving the pin later needs
it.

A machine that registered this marketplace earlier — unpinned, before the pin
existed, or at another tag — re-registers it with the `@` suffix matching the
checked-in `ref`, which overwrites the recorded source:

```text
/plugin marketplace add TaiSakuma/reviewed-writer@v0.5.0
```

From a shell, `claude plugin marketplace add TaiSakuma/reviewed-writer@v0.5.0`
does the same. The bare `TaiSakuma/reviewed-writer` source string registers the
default branch instead and overrides the pin. If the plugin is already installed
from this marketplace, re-registering leaves the installed copy as it is; update
it as under Move the pin to a new release, steps 2 and 3.

## 🔧 Confirm the install

In a session opened in the repository:

1. Run `/plugin` and find `reviewed-writer@reviewed-writer` in the Installed
   tab, which groups plugins by scope. The version shown there is the installed
   version (`claude plugin list` prints the same from a shell); it is the
   version that runs once the session has loaded it.
2. If it is listed as not installed, run the install command Claude Code shows,
   as under Pin the plugin for the repository: trusting the folder added the
   marketplace and installed nothing.
3. Read the install summary. `Plugin is now active.` needs nothing further.
   `Run /reload-plugins to activate.` is followed by that reload, which Claude
   Code runs for you; if the reload warns that your next message would re-read
   the conversation, run `/reload-plugins --force`. A `claude plugin` command
   run in another terminal takes effect at the next session start, or after
   `/reload-plugins` in the open session.
4. Type `/reviewed-writer:` in the prompt; both `write-doc` and `persona-review`
   complete.

A loaded plugin does useful work only once the repository's own files exist. Set
up a consuming repository authors them and ends in the review round that
confirms them.

## 🔧 Set up a consuming repository

From a repository with the plugin installed or pinned and confirmed, and none of
the files, to its first review round. Each file starts from a template: copy it,
replace each angle-bracket placeholder with your own value, and delete the
guidance comments as you go. A subsection each, in order: the profile, the
declaration file, the document's own headings, a persona head file, the voice
rules, the optional wrapper skill, and the smoke test. What each file must
contain is stated in full under What the consuming repository provides; the
subsections say what to write now.

The templates sit in the installed copy of the plugin. On disk that is
`plugins/cache/reviewed-writer/reviewed-writer/<version>/templates/` under your
Claude Code configuration directory, `~/.claude` unless `CLAUDE_CONFIG_DIR` is
set, with `<version>` the version `/plugin` shows for
`reviewed-writer@reviewed-writer` — or, outside a session, the version
`claude plugin list` prints for it — written without the leading `v` of the
release tag. From the repository root, in bash:

```bash
v="<version>"   # the version /plugin shows, without the leading v
r="<reader>"    # your name for the reader this first persona stands for
t="${CLAUDE_CONFIG_DIR:-$HOME/.claude}"/plugins/cache/reviewed-writer/reviewed-writer/$v/templates
mkdir -p .claude/rules .claude/personas
cp "$t"/persona-review-profile.md "$t"/diataxis-declaration.md "$t"/docs-voice.md .claude/rules/
cp "$t"/persona.md .claude/personas/"$r".md
```

The other route needs no path, and is the one to use if that directory does not
exist: run `/reviewed-writer:persona-review` in the repository before any file
exists; the skill stops at its preflight, naming the missing file and the
template to copy, and the session can copy that template into place, since the
plugin-root placeholder resolves for the skill and not at your prompt.

### The profile

Copy `persona-review-profile.md` to `.claude/rules/persona-review-profile.md`,
the literal path the plugin reads. Keep the wording of its eleven `##` headings
exactly, since the plugin looks them up by name, and write a prose body under
each: the table under What the consuming repository provides says what each
supplies. Name the document under `Document`, the persona head files you are
about to write under `Personas`, and the voice-rules path under `Voice rules`.
`Premise to pin` and `Extra guidelines` read `None.` when there is nothing to
state, and every other section takes a prose body, however short; write
`Disabled.` under `Status dimension` unless your units carry a status, in which
case the table under What the consuming repository provides says what to write.

### The declaration file

Copy `diataxis-declaration.md` to `.claude/rules/diataxis-declaration.md`, also
a literal path. Its legend maps one visible marker to each of the four Diátaxis
forms: keep the template's markers, or substitute the glyphs your documents use,
but keep the forms, since the review core has reader questions for those four
and no others; for a form your documents do not use, keep the row and note it as
unused, as the template does, or drop it. State where a marker sits relative to
the heading text, and keep the three section shapes the template describes and
the record.

### The document's headings

Give every `##` heading of the document the profile's `Document` section names a
marker from your legend; a heading that only groups marked subsections carries
none, and may keep one orientation sentence of its own without one. Every
reviewer reports an unmarked heading with content as undeclared, and no review
round marks a heading for you: mark existing content by hand before the first
round, or let the first `/reviewed-writer:write-doc` run settle the markers with
you at its scoping step. Which form a section is is a [Diátaxis] question; the
review core named under What the plugin ships holds the per-form guidance the
reviewers apply.

### A persona head file

Copy `persona.md` to `.claude/personas/<reader>.md` — for example
`.claude/personas/plugin-user.md` — and list the path in the profile's
`Personas` section. The reviewer subagent reads the file first, from the path
the `persona-review` skill puts at the start of its task prompt, so write it in
the second person and give it no frontmatter; keep the parts the template
carries; each file stands for one real reader. One file is enough to run the
panel; each file added puts one more reviewer in every round.

### The voice rules

Copy `docs-voice.md` to the path the profile's `Voice rules` section names —
this repository names `.claude/rules/docs-voice.md` — and write the audience,
the editorial rules, and the formatting conventions the prose follows. The
template opens with optional `paths:` frontmatter, a Claude Code rule scope that
loads a file under `.claude/rules/` into sessions touching the listed documents:
keep it at the top with your document's path when the file sits there, or delete
the block. The plugin reads the file by the path the profile names either way.

### The wrapper skill, optional

A wrapper keeps a repository's established entry point; there is no template.
Create `.claude/skills/<skill>/SKILL.md` in the shape given under What the
consuming repository provides, with the frontmatter `name` set to the
directory's name as this repository does; that name is the invocation name. This
repository's `write-docs` wraps `write-doc` and `review-docs` wraps
`persona-review`.

### The smoke test

The document the profile's `Document` section names must exist, with its
headings marked; if you are adopting the plugin to author it from nothing,
create it as a stub with marked headings, since the round reviews what is there.
Then run one report-only round in the repository:

```text
/reviewed-writer:persona-review
```

A correct result: the round reads the profile, reviews the document as it
stands, launches one read-only reviewer per persona the `Personas` section
lists, and reports a matrix carrying one ship-or-revise verdict per persona. It
modifies nothing. If it instead stops and names a missing path or profile
heading together with the template that supplies it, that is what is still to
write. The preflight, as described under What the consuming repository provides,
confirms that those four files exist and that the profile carries its eleven
headings; it does not check the document or its markers, and it reads only two
bodies, the Personas list and the Voice rules path, to find those files. A
placeholder path there stops the run; one elsewhere passes and shows up in the
reviews instead.

Commit the profile, the declaration file, the persona head files, and the voice
rules. `/reviewed-writer:write-doc`, or the wrapper skill, then authors or
revises the document.

## 🔧 Move the pin to a new release

Edit `ref` in the repository's `.claude/settings.json` to the new tag and commit
it. That edit does not re-point a machine that already registered the
marketplace; on each such machine, three steps:

1. Re-register the marketplace at the new tag, which overwrites the recorded
   source: `/plugin marketplace add TaiSakuma/reviewed-writer@<new-tag>` in a
   session, or
   `claude plugin marketplace add TaiSakuma/reviewed-writer@<new-tag>` from a
   shell, with the tag in the `v<version>` form the pin uses.
2. Update the installed plugin — re-registering leaves the installed copy as it
   is: from a shell in the repository,
   `claude plugin update reviewed-writer@reviewed-writer --scope <scope>` with
   the scope the install used (`user`, `project`, or `local`; the command
   defaults to `user`, and the `/plugin` Installed tab groups plugins by scope).
   The update is skipped when the resolved version already matches the installed
   one; when the registration was untagged before step 1, that leaves the
   default-branch copy in place, so replace it instead:
   `claude plugin uninstall reviewed-writer@reviewed-writer --scope <scope>`,
   then `claude plugin install reviewed-writer@reviewed-writer --scope <scope>`.
   At project scope both commands rewrite `.claude/settings.json`; check
   `git diff` afterwards.
3. Load the new version: `/reload-plugins`, or restart Claude Code. `/plugin`
   then shows the new version for `reviewed-writer@reviewed-writer`.

A machine that never registered the marketplace needs none of this: it picks up
the checked-in pin when the folder is trusted, then installs as under Pin the
plugin for the repository.

A repository pinned at the rolling `latest` tag has no tag to edit: refresh the
marketplace with `/plugin marketplace update reviewed-writer`, then take steps 2
and 3; if the version does not move, re-register with
`/plugin marketplace add TaiSakuma/reviewed-writer@latest` and repeat them.
Auto-update does the refresh and the update after a session starts — Claude Code
refreshes the marketplace data and updates installed plugins in the background,
up to ten minutes after the session starts, then prompts for `/reload-plugins`;
the running session keeps the versions it loaded at launch until then. Enable it
per marketplace in `/plugin` → Marketplaces → the marketplace → Enable
auto-update; it is disabled by default for third-party marketplaces, which
includes this one.

## 📋 What the consuming repository provides

The writer and panel agents, the `persona-review` skill, and the reviewer agent
read the first four files by name from the consuming repository, never from the
plugin's own directory: the first two at the literal paths below, the others
wherever the profile names them. The last two configure Claude Code itself.

| File                                                            | Template it starts from     | What it must contain                                                                                      |
| --------------------------------------------------------------- | --------------------------- | --------------------------------------------------------------------------------------------------------- |
| `.claude/rules/persona-review-profile.md`                       | `persona-review-profile.md` | The eleven `##` headings below, each with a prose body.                                                   |
| `.claude/rules/diataxis-declaration.md`                         | `diataxis-declaration.md`   | The marker legend, the marker placement, the section shapes, the record.                                  |
| `.claude/personas/<reader>.md` by convention, one per reviewer  | `persona.md`                | One reader, in the second person: opening sentence, the reader's own question, six parts, flags sentence. |
| The voice-rules file, for example `.claude/rules/docs-voice.md` | `docs-voice.md`             | The editorial rules the final text follows.                                                               |
| `.claude/settings.json`                                         | —                           | The marketplace pin and the `enabledPlugins` entry.                                                       |
| `.claude/skills/<skill>/SKILL.md` (optional)                    | —                           | A wrapper that invokes a namespaced skill.                                                                |

**The profile.** Its eleven `##` headings are read by name, so a missing or
renamed heading stops a run. Each body is prose an agent reads, not a parsed
field.

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

**The declaration file.** Its legend maps one visible marker to each of the four
Diátaxis forms. The markers are the consuming repository's choice — the
template's four emoji are this repository's convention, and any other visible
glyph serves — but the forms are not: the review core's reader questions exist
for those four alone, and a repository drops a row only for a form its documents
do not use. The file also states the marker placement (immediately after the `#`
hashes, before the heading text, separated by single spaces), the section shapes
— a marked section, whose unmarked subheadings share its form; a container that
groups marked subsections and carries no marker of its own and at most one
orientation sentence, and unmarked front matter — and the declaration record.
Reviewers report a marker outside the legend, misplaced, or missing, and report
a marker that does not match what its section does as a proposed
reclassification: changing a declaration is yours to make, or a `write-doc`
run's at its scoping step, never a review round's.

**A persona head file.** The reviewer subagent reads it first, from the path at
the start of its task prompt, so it addresses the reviewer in the second person
and carries no frontmatter. Its fixed parts are an opening sentence naming the
persona, a blockquoted question in that reader's own words, six bold parts in
order — Context, Scope, Goals, How you read, Pain points / what erodes your
trust, Your lens (what you scrutinize hardest) — and a closing sentence naming
the kind of flags the reviewer reports. One persona head file is enough to run
the workflow; the panel is whatever the `Personas` section lists.

**The voice-rules file.** The editorial rules for the documents' prose, at the
path the profile's `Voice rules` section names; the writer agent applies them
when it writes the final text, and persona-suggested wording is advisory against
them. Its template carries optional `paths:` frontmatter, a Claude Code rule
scope.

**The settings pin.** `.claude/settings.json` carries the marketplace source,
with the release tag pinned, and the `enabledPlugins` entry, as under Pin the
plugin for the repository.

**The wrapper skill**, optional, is a `SKILL.md` under `.claude/skills/<skill>/`
with `name` and `description` frontmatter — plus `argument-hint` when it takes
an argument — whose body invokes the namespaced skill it wraps; a repository's
standing draft count, re-review cap, writer, and panel belong there.

**The preflight.** The writer, the panel, and `persona-review` check these files
before anything else: the profile, carrying every `##` section the profile
template lists; the declaration file; every persona head file the `Personas`
section lists; and the voice-rules file the `Voice rules` section names. A
missing file, or a profile section absent or renamed, stops the run: it reports
the missing path or heading, names the template in the plugin's `templates/`
directory to copy and fill in, and does not proceed on a guess. The check
confirms existence and headings; it reads no body beyond the Personas and Voice
rules sections, which name the files it looks for.

**Worked examples.** This repository's own [`.claude/`
directory][own-claude-dir] is a filled-in set in the source tree — the profile,
the declaration file, the persona head files, the voice rules, the pin, and two
wrapper skills; the other files under `.claude/` there are this repository's own
and not part of the interface. The plugin cache holds a copy of the directory;
the source tree is the version to read. [legendary-octo-happiness] and
[hypothesis-awkward] are two further complete sets. What the inventory costs to
maintain is under Why this workflow.

## 📋 Versioning

Releases are tagged `v<version>` (for example `v0.5.0`), and
`.claude-plugin/plugin.json` carries the matching version (`0.5.0`); the
`u<version>` tags on the repository (for example `u0.5.0`) are CI triggers, not
pin targets. The rolling `latest` tag points at the highest released version.
The release runbook and the PR-title convention are in
[CONTRIBUTING.md][contributing], under Releasing and PR Title Convention; the
per-release record is [CHANGELOG.md][changelog].

The profile's path and its `##` section headings, the declaration file's path
and role, and the namespaced invocation names are the plugin's interface to a
consuming repository. Before 1.0, any release may change that interface; each
release's notes on the [releases] page list the pull requests merged since the
previous release, so an interface change appears there under its title, and
moving a pin can oblige edits to the checked-in files. A renamed profile heading
surfaces at the next run, which stops before any reviewer launches and names the
heading; a change in what a section must supply does not stop a run, so the
release notes are the signal for it.

The plugin mechanics in this README were checked against the Claude Code
documentation and, where noted below, observed on Claude Code 2.1.269; the
install and update steps assume Claude Code 2.1.195 or newer, where adding a
marketplace stopped installing its plugins, and a `write-doc` run assumes
2.1.219 or newer, where subagents nest by default. Claims taken from the
documentation ([discovering plugins][docs-discover], [plugin
marketplaces][docs-marketplaces], and the [plugins reference][docs-reference])
include: the `@ref` suffix, the 2.1.195 change, the install summary and its
reload, the scopes and their settings files, the rule that adding a marketplace
under an existing name replaces it, the rule that an update is skipped when the
resolved version matches the installed one, the cache layout composed with the
documented configuration directory, the `claude plugin list` output, the SSH
default, and the auto-update default and toggle. Observed on 2.1.269 and not
documented: editing the checked-in `ref` does not re-point a machine that
already registered the marketplace, the bare source string overrides the pin,
re-registering leaves the installed version in place, and `claude plugin update`
then moves it. The `latest` path follows from the documented behavior of the
commands it uses and was not run.

## 📖 Why this workflow

Documentation review by a single generic reviewer drifts toward style, and its
feedback changes with the reviewer. This plugin holds the reader fixed instead:
the reviewers are subagents that adopt persona definitions checked into the
repository — a deliberate proxy for the document's audience, not the audience
itself, complementing human review rather than replacing it. The reviews are
model output: two runs over the same text return different reviews, and a
unanimous "ship" records that the panel's questions were answered, not that the
document is good.

Each Diátaxis form answers a different question its reader arrives with, and a
unit that tries to answer two answers neither well; the plugin makes that
separation explicit. Each unit of content declares its form, and the declaration
is the fixed point of a review: content is judged against the declared form,
never the reverse, and an ask that would pull a unit toward another form is
routed to the unit that owns that form. That discipline separates the workflow
from what it resembles: a docs linter checks structure and style and never asks
whether the reader's question was answered, and a classification maintained by
hand is an intention that erodes, as it had in [legendary-octo-happiness], one
of the two repositories it came from, where here it is a marker the panel
re-reads every round. Structurally distinct drafts precede the panel because
structure is the decision hardest to reverse once a text exists; the writer
agent writes the final text, personas' wording is advisory, and accuracy beats
style.

The exchange is real. A consuming repository authors and maintains the profile,
the declaration rules, the persona head files, and the voice rules; the
templates shorten the first pass, not the upkeep. They age with the documents,
and a persona nobody updates does not fail loudly — it keeps shipping confident
verdicts from a reader who no longer exists. A run's cost scales with the panel
the profile's `Personas` section lists: the draft round launches one read-only
reviewer per persona, and one review is one subagent reading its persona head
file, the run's brief, and the text under review — the whole document, or every
draft in the draft round; each later round continues the same reviewers with
what changed: a full report on the synthesized document, then a re-read and a
short reply per persona, rather than a fresh review each time, though a
continued reviewer's context grows with each round. The panel runs in parallel,
and the writer agent writes each draft in full before the panel sees it. A full
`write-doc` run adds a panel pass over the drafts to the re-review rounds, up to
the cap. At the default draft count and cap — three and five — with six
personas, that is at most six panel rounds — six full reviews of the drafts, six
full reports on the synthesized text, and up to 24 follow-ups — for one
document; a panel of one is valid and costs a sixth of that. The checked-in
files serve the whole repository, while a run covers the document the profile's
`Document` section sets as its unit of work, so the setup is authored once and
the run cost repeats for each document revised. Lowering the cap at invocation
lowers the ceiling; lowering the draft count lowers how much each reviewer
reads, not how many reviews run. The workflow fits documents revised
deliberately for distinct audiences; it is a poor fit for documentation that
changes daily, or a repository unwilling to keep persona definitions current. A
run is driven from a session: it settles scope with you and hands unresolved
dissent back to you, so it is an authoring step, not a check that can gate a
pull request. The exit is bounded: the profile, the declaration rules, the
personas, the voice rules, and the documents stay in the consuming repository,
so dropping the plugin forfeits the machinery, not the content.

## 📖 Provenance

The workflow was converged in use, not designed on paper: five refactoring
iterations ([issue #49]) across two repositories with little in common —
[legendary-octo-happiness], a reference implementation of changelog-and-release
automation, and [hypothesis-awkward], a [Scikit-HEP] project whose documentation
runs the workflow — the last of them the extraction into this plugin. The two
setups differ exactly where the profile lets them differ (five personas in one,
six in the other; a single README versus a growing page set), which is what the
convergence settled: the machinery is fixed, and everything repository-specific
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
[docs-discover]: https://code.claude.com/docs/en/discover-plugins
[docs-marketplaces]: https://code.claude.com/docs/en/plugin-marketplaces
[docs-reference]: https://code.claude.com/docs/en/plugins-reference
[Scikit-HEP]: https://scikit-hep.org/
[legendary-octo-happiness]:
  https://github.com/TaiSakuma/legendary-octo-happiness
[hypothesis-awkward]: https://github.com/scikit-hep/hypothesis-awkward
[issue #49]: https://github.com/TaiSakuma/legendary-octo-happiness/issues/49
