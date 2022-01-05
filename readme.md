# rust-lang/cargo/tree/master/src/doc [![translate-svg]][translate-list]

<!-- [![explain]][source] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 Cargo 官书文档 」

[中文](./readme.md) | [english](https://github.com/istankovic/cargo-docs)

---

## 更新 🀄

<!-- doc-templite START generated -->
<!-- repo = 'rust-lang/cargo' -->
<!-- commit = '35e82eb45f10de09e2a4b8e5f15a1c9adbd34680' -->
<!-- time = '2022-1-2' -->

| 翻译的原文 | 与日期      | 最新更新 | 更多                       |
| ---------- | ----------- | -------- | -------------------------- |
| [commit]   | ⏰ 2022-1-2 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/rust-lang/cargo.svg
[commit]: https://github.com/rust-lang/cargo/tree/35e82eb45f10de09e2a4b8e5f15a1c9adbd34680

<!-- doc-templite END generated -->

# Summary

- [x] [介绍](src/index.zh.md)

- [x] [入门](src/getting-started/index.zh.md)

- [x] [Cargo Guide](src/guide/index.zh.md)

- [ ] [Cargo Reference](src/reference/index.md)

  - [x] [Specifying Dependencies](src/reference/specifying-dependencies.zh.md)
    - [ ] [Overriding Dependencies](src/reference/overriding-dependencies.md)
  - [ ] [The Manifest Format](src/reference/manifest.md)
    - [ ] [Cargo Targets](src/reference/cargo-targets.md)
  - [ ] [Workspaces](src/reference/workspaces.md)
  - [ ] [Features](src/reference/features.md)
    - [ ] [Features Examples](src/reference/features-examples.md)
  - [ ] [Profiles](src/reference/profiles.md)
  - [ ] [Configuration](src/reference/config.md)
  - [ ] [Environment Variables](src/reference/environment-variables.md)
  - [ ] [Build Scripts](src/reference/build-scripts.md)
    - [ ] [Build Script Examples](src/reference/build-script-examples.md)
  - [ ] [Publishing on crates.io](src/reference/publishing.md)
  - [ ] [Package ID Specifications](src/reference/pkgid-spec.md)
  - [ ] [Source Replacement](src/reference/source-replacement.md)
  - [ ] [External Tools](src/reference/external-tools.md)
  - [ ] [Registries](src/reference/registries.md)
  - [ ] [Dependency Resolution](src/reference/resolver.md)
  - [ ] [SemVer Compatibility](src/reference/semver.md)
  - [ ] [Future incompat report](src/reference/future-incompat-report.md)
  - [ ] [Unstable Features](src/reference/unstable.md)

- [ ] [Cargo Commands](src/commands/index.md)

  - [ ] [General Commands](src/commands/general-commands.md)
    - [ ] [cargo](src/commands/cargo.md)
    - [ ] [cargo help](src/commands/cargo-help.md)
    - [ ] [cargo version](src/commands/cargo-version.md)
  - [ ] [Build Commands](src/commands/build-commands.md)
    - [ ] [cargo bench](src/commands/cargo-bench.md)
    - [ ] [cargo build](src/commands/cargo-build.md)
    - [ ] [cargo check](src/commands/cargo-check.md)
    - [ ] [cargo clean](src/commands/cargo-clean.md)
    - [ ] [cargo doc](src/commands/cargo-doc.md)
    - [ ] [cargo fetch](src/commands/cargo-fetch.md)
    - [ ] [cargo fix](src/commands/cargo-fix.md)
    - [ ] [cargo run](src/commands/cargo-run.md)
    - [ ] [cargo rustc](src/commands/cargo-rustc.md)
    - [ ] [cargo rustdoc](src/commands/cargo-rustdoc.md)
    - [ ] [cargo test](src/commands/cargo-test.md)
    - [ ] [cargo report](src/commands/cargo-report.md)
  - [ ] [Manifest Commands](src/commands/manifest-commands.md)
    - [ ] [cargo generate-lockfile](src/commands/cargo-generate-lockfile.md)
    - [ ] [cargo locate-project](src/commands/cargo-locate-project.md)
    - [ ] [cargo metadata](src/commands/cargo-metadata.md)
    - [ ] [cargo pkgid](src/commands/cargo-pkgid.md)
    - [ ] [cargo tree](src/commands/cargo-tree.md)
    - [ ] [cargo update](src/commands/cargo-update.md)
    - [ ] [cargo vendor](src/commands/cargo-vendor.md)
    - [ ] [cargo verify-project](src/commands/cargo-verify-project.md)
  - [ ] [Package Commands](src/commands/package-commands.md)
    - [ ] [cargo init](src/commands/cargo-init.md)
    - [ ] [cargo install](src/commands/cargo-install.md)
    - [ ] [cargo new](src/commands/cargo-new.md)
    - [ ] [cargo search](src/commands/cargo-search.md)
    - [ ] [cargo uninstall](src/commands/cargo-uninstall.md)
  - [ ] [Publishing Commands](src/commands/publishing-commands.md)
    - [ ] [cargo login](src/commands/cargo-login.md)
    - [ ] [cargo owner](src/commands/cargo-owner.md)
    - [ ] [cargo package](src/commands/cargo-package.md)
    - [ ] [cargo publish](src/commands/cargo-publish.md)
    - [ ] [cargo yank](src/commands/cargo-yank.md)

- [ ] [FAQ](src/faq.md)
- [ ] [Appendix: Glossary](src/appendix/glossary.md)
- [ ] [Appendix: Git Authentication](src/appendix/git-authentication.md)


### NEW: 还有本 Cargo 贡献指南

- [ ] ./contrib 就缓一缓

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---

