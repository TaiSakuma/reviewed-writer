# Contributing

Contributors need the Setup and PR Title Convention sections; installing the
plugin that authors these documents is optional; everything under Releasing is
the maintainers' runbook.

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
plugin, wrapped here as the `/write-docs` and `/review-docs` skills. Opening a
PR needs neither; this section is for running them.

The plugin is pinned in `.claude/settings.json`. That declaration names the
source and installs nothing by itself: Claude Code offers to install it when you
trust the repository folder. Accept the offer, and the components load at the
next session start, or immediately after `/reload-plugins`.

When no offer appears — you trusted the folder earlier, or declined — register
the pinned source yourself, with the `#` suffix set to the `ref` value in
`.claude/settings.json` (`v0.2.0` as this is written):

```text
/plugin marketplace add TaiSakuma/reviewed-writer#<ref>
```

The components load on the same terms: next session start, or `/reload-plugins`.

Registration is per machine rather than per repository, and the add overwrites
any `reviewed-writer` registration the machine already has. If you run the
plugin in your own repository at a different tag, re-add that tag when you are
done here.

The two wrapper skills load from `.claude/skills/` whether or not the plugin is
installed, so they are offered before they work: invoking `/review-docs` without
it fails with `Unknown skill: reviewed-writer:persona-review` rather than
reviewing anything.

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
   `version` field in `.claude-plugin/plugin.json`, commits, and tags the commit
   `u<version>` (annotated). By hand, where `0.1.0` and `0.2.0` stand for the
   current and new versions:

   ```bash
   # edit the "version" field in .claude-plugin/plugin.json, then:
   git add .claude-plugin/plugin.json
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
# fix the "version" field in .claude-plugin/plugin.json, then:
git add .claude-plugin/plugin.json
git commit --amend --no-edit
git tag -f -a u0.2.0 -m "Bump version 0.1.0 → 0.2.0"
git push origin u0.2.0
```

[pre-commit]: https://pre-commit.com/
[Conventional Commits]: https://www.conventionalcommits.org/
[legendary-octo-happiness]:
  https://github.com/TaiSakuma/legendary-octo-happiness#release-process
