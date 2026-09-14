# Changelog

All notable changes to this project will be documented in this file.


## [0.5.1] - 2026-09-14

### Documentation

- Document testing plugin changes with `--plugin-dir` ([#16](https://github.com/TaiSakuma/reviewed-writer/pull/16))
- Shorten `README.md` and require Claude Code 2.1.269 ([#17](https://github.com/TaiSakuma/reviewed-writer/pull/17))

### Build & CI

- Drop the removed README commands from the bump-version sweep ([#18](https://github.com/TaiSakuma/reviewed-writer/pull/18))


## [0.5.0] - 2026-09-13

### Refactoring

- Extract an agent from the skill `write-doc` ([#14](https://github.com/TaiSakuma/reviewed-writer/pull/14))
- Split the engine into writer and panel agents ([#15](https://github.com/TaiSakuma/reviewed-writer/pull/15))


## [0.4.0] - 2026-09-13

### Performance

- Continue reviewers across rounds and compact their reports ([#13](https://github.com/TaiSakuma/reviewed-writer/pull/13))


## [0.3.0] - 2026-09-12

### Features

- Guide first-run setup with templates and a missing-file preflight ([#11](https://github.com/TaiSakuma/reviewed-writer/pull/11))


## [0.2.1] - 2026-08-09

### Bug Fixes

- Make the re-review cap budget all rounds and repeat the fact-check ([#7](https://github.com/TaiSakuma/reviewed-writer/pull/7))

### Documentation

- Trim versioning coverage and fix the pin suffix ([#8](https://github.com/TaiSakuma/reviewed-writer/pull/8))

### Build & CI

- Pin the plugin at v0.2.0 ([#6](https://github.com/TaiSakuma/reviewed-writer/pull/6))
- Sweep version literals in the bump commit ([#9](https://github.com/TaiSakuma/reviewed-writer/pull/9))


## [0.2.0] - 2026-08-07

### Features

- Make the draft count and re-review cap configurable ([#5](https://github.com/TaiSakuma/reviewed-writer/pull/5))

### Documentation

- Revise `README.md` and `CONTRIBUTING.md` ([#4](https://github.com/TaiSakuma/reviewed-writer/pull/4))

### Build & CI

- Consume this plugin in its own repository ([#2](https://github.com/TaiSakuma/reviewed-writer/pull/2))


## [0.1.1] - 2026-08-04

### Build & CI

- Add changelog and release pipeline ([#1](https://github.com/TaiSakuma/reviewed-writer/pull/1))


## [0.1.0] - 2026-08-04

### Features

- Initial reviewed-writer plugin ([c0864a1](https://github.com/TaiSakuma/reviewed-writer/commit/c0864a1eb29198d16f4b7bd291fd2aa3b3b2bc47))
