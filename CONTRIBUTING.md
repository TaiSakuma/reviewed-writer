# Contributing

Contributors need the Setup and PR Title Convention sections; the two sections
about the plugin that authors these documents are optional; everything under
Releasing is the maintainers' runbook.

## 🔧 Setup

After cloning, install [pre-commit] itself if the machine does not have it (its
site covers the ways), then install the hook so the checks run at commit time:

```bash
pre-commit install
```

The hook runs prettier over markdown and rewrites files that fail it: the commit
aborts, and you `git add` the rewritten files and commit again. Prose is
hard-wrapped at 80 columns (`proseWrap` in `.prettierrc.toml`), so hand-wrapped
text gets reflowed. `.prettierignore` keeps prettier away from `CHANGELOG.md`
(CI generates it) and `.github/`. CI runs the same check on every PR (the "Lint
and format" workflow), over all files rather than only the ones a PR touches.
`pre-commit run --all-files` checks the whole tree the same way, but only files
tracked by git — `git add` new files first.

## 📋 PR Title Convention

PR titles follow [Conventional Commits]. The repository squash-merges, so the PR
title becomes the final commit message. The "Validate PR title" workflow checks
the title when a PR is opened, edited, or updated — editing the title re-runs
the check, which appears in the PR's checks as "Conventional Commits".
Individual commit messages within a PR are free-form; only the title is
enforced. The "Label PR by convention" workflow labels the PR from its type, and
those labels choose the categories in the release notes.

### Format

```text
type: description
```

Scoped prefixes are rejected: `feat(parser): description` fails the check. `!`
after the type marks a breaking change: `feat!: remove get_user()`.

### Allowed Types

| Type       | Purpose                                                 |
| ---------- | ------------------------------------------------------- |
| `feat`     | A new feature                                           |
| `fix`      | A bug fix                                               |
| `docs`     | Documentation only                                      |
| `style`    | Code style (formatting, semicolons, etc.)               |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf`     | Performance improvement                                 |
| `test`     | Adding or updating tests                                |
| `build`    | Build system or external dependencies                   |
| `ci`       | CI configuration                                        |
| `chore`    | Other changes that don't modify src or test files       |
| `revert`   | Reverts a previous commit                               |

### Examples

- `feat: add user authentication`
- `fix: handle empty input`
- `docs: update installation instructions`
- `feat!: remove get_user()`

## 🔧 Install the plugin that authors these documents

`README.md` and `CONTRIBUTING.md` are authored through the `reviewed-writer`
plugin, wrapped here as `/write-docs` and `/review-docs`. Opening a PR needs
neither.

The pin is checked in; `.claude/settings.json` names the tag (`v0.5.0` as this
is written). With the folder trusted, run `/plugin` in a session here and find
`reviewed-writer@reviewed-writer` in the Installed tab:

- Not installed: install it as the [README][readme] describes under "Pin the
  plugin for the repository", install command and scope note, then "Confirm the
  install".
- Another version, or the pinned version from a registration without a tag
  (`claude plugin marketplace list --json` shows a `ref` only for a pinned one):
  the checked-in `ref` does not move an existing registration. Take steps 1 to 3
  of the README's "Move the pin to a new release" with the pinned tag as the new
  tag; an untagged registration already at the pinned version number takes step
  2's uninstall-and-install route.
- The pinned version from a pinned registration: done.

A registration you make yourself is recorded outside this repository and
replaces the machine's `reviewed-writer` source. If you run the plugin elsewhere
at another tag, re-add that tag, update, and reload there when you are done
here.

A session started without `--plugin-dir` runs the installed copy, not the
working tree: an edit to `skills/`, `agents/`, or `templates/` reaches that
session only after a release and the cases above on your machine. To start a
session against the working tree instead, see "Test a plugin change before
releasing it" below. The wrappers under `.claude/skills/` are read from the tree
without a release.

If `/write-docs` or `/review-docs` fails with an unknown-skill error naming
`reviewed-writer:write-doc` or `reviewed-writer:persona-review`, the plugin is
not loaded: run `/reload-plugins`, or go through the cases above. If the session
was started with `--plugin-dir`, check that flag first, as the next section
describes.

## 🔧 Test a plugin change before releasing it

To run `/write-docs` or `/review-docs` against an edit to `skills/`, `agents/`,
or `templates/` that is not released yet, load the plugin from the working tree
when you start the session:

```bash
claude --plugin-dir .   # run from the repository root
```

`.` is the plugin's own directory here, since `.claude-plugin/plugin.json` sits
at the repository root. The flag needs no prior install and no `enabledPlugins`
entry, and it changes nothing on disk. The directory copy takes precedence over
the installed `reviewed-writer`, so `/write-docs` and `/review-docs` drive the
working tree's `reviewed-writer:write-doc`, `reviewed-writer:persona-review`,
and the agents they launch — the run exercises the working tree's machinery, not
the pinned release's.

A path that is not a plugin directory does not fail the session; it falls back
to the installed copy. Confirm what the flag resolves to before trusting a run:

```bash
claude --plugin-dir . plugin details reviewed-writer   # from the repository root
```

The `Source:` line names the copy: `reviewed-writer@inline` for the directory
you passed, `reviewed-writer@reviewed-writer` for the installed one. The
installed one means the path you passed was not a plugin directory; start the
session again from the repository root, or pass the path to it.
`Plugin "reviewed-writer" not found.` is the third outcome — nothing is
installed to fall back to, and `/write-docs` fails with the unknown-skill error
above.

`--plugin-dir` is read when the session starts and holds for that session alone:
a running session cannot be switched over, and the next session started without
it runs whichever copy "Install the plugin that authors these documents" left
installed.

## Releasing

Maintainers only; the pipeline is adapted from [legendary-octo-happiness] (LOH),
whose README carries both the procedure ("Cut a release") and the design behind
it ("Why the pipeline is built this way", "How a release moves through git") —
only the bump differs here, and one extra guard can fail.

### 🔧 Bump the version and push the tag

Follow LOH's "Cut a release", starting at its first step: check out the commit
to release. The two steps below replace its bump and its tag push; LOH resumes
at the workflow runs, and covers the merge-back or backport and the pull that
follows.

1. **Bump the version** — LOH bumps with `hatch`; here:

   Run the repo-local `/bump-version` skill in Claude Code with `patch`,
   `minor`, `major`, or an explicit version. It checks the guards, updates the
   `version` field in `.claude-plugin/plugin.json`, refreshes the
   current-release literals in `README.md`, `CONTRIBUTING.md`, and the pin in
   `.claude/settings.json`, commits, and tags the commit `u<version>`
   (annotated). By hand, where `0.1.0` and `0.2.0` stand for the current and new
   versions:

   ```bash
   # edit the "version" field in .claude-plugin/plugin.json and refresh the
   # current-release literals in README.md, CONTRIBUTING.md, and the pin in
   # .claude/settings.json, then:
   git add .claude-plugin/plugin.json .claude/settings.json README.md CONTRIBUTING.md
   git commit -m "Bump version 0.1.0 → 0.2.0"
   git tag -a u0.2.0 -m "Bump version 0.1.0 → 0.2.0"
   ```

   Keep the message unconventional as shown, so git-cliff filters it out of
   `CHANGELOG.md`. The hand path skips the guards, so check them yourself: the
   tree is clean; HEAD carries `.github/workflows/changelog.yml` and is the
   commit to release; the version is plain `X.Y.Z` — not `0.2.0-rc.1`, which
   sorts newest and would move the rolling `latest` tag consumers pin — and
   greater than the current one; and neither `u<version>` nor `v<version>`
   exists yet, locally or on origin.

2. **Push only the tag**, never `main --tags`, which fails on a stale local
   `latest` tag:

   ```bash
   git push origin u0.2.0
   ```

If the "Generate changelog" run fails at "Verify tag matches plugin.json
version", the next section applies; recover any other failure as LOH directs.

### 🔧 Fix a version-check failure

That step is this repository's own: LOH's `hatch` bump made the tag and the
manifest match by construction, and a hand-edited manifest cannot. It runs
before the release branch is created, so the trigger tag is the only thing to
clean up. Delete it, correct the commit — amend rather than add one on top,
since the pipeline releases the tag's parent — and push again:

```bash
git push origin --delete u0.2.0
# fix the "version" field in .claude-plugin/plugin.json and re-sweep the
# current-release literals, then:
git add .claude-plugin/plugin.json .claude/settings.json README.md CONTRIBUTING.md
git commit --amend --no-edit
git tag -f -a u0.2.0 -m "Bump version 0.1.0 → 0.2.0"
git push origin u0.2.0
```

[readme]: README.md
[pre-commit]: https://pre-commit.com/
[Conventional Commits]: https://www.conventionalcommits.org/
[legendary-octo-happiness]:
  https://github.com/TaiSakuma/legendary-octo-happiness#release-process
