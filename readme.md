# rust-lang/cargo/tree/master/src/doc [![translate-svg]][translate-list]

<!-- [![explain]][source] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 Cargo 官书文档 」

[中文](./readme.md) | [english](https://github.com/rust-lang/cargo/tree/master/src/doc)

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

- [x] [介绍](src/index.zh.zh.md)

- [x] [入门](src/getting-started/index.zh.zh.md)

- [x] [Cargo Guide](src/guide/index.zh.zh.md)

- [ ] [Cargo Reference](src/reference/index.zh.md)

  - [x] [Specifying Dependencies](src/reference/specifying-dependencies.zh.zh.md)
    - [x] [Overriding Dependencies](src/reference/overriding-dependencies.zh.md)
  - [x] [The Manifest Format](src/reference/manifest.zh.md)
    - [x] [Cargo Targets](src/reference/cargo-targets.zh.md)
  - [x] [Workspaces](src/reference/workspaces.zh.md)
  - [x] [Features](src/reference/features.zh.md)
    - [ ] [Features Examples](src/reference/features-examples.zh.md)
  - [ ] [Profiles](src/reference/profiles.zh.md)
  - [ ] [Configuration](src/reference/config.zh.md)
  - [ ] [Environment Variables](src/reference/environment-variables.zh.md)
  - [ ] [Build Scripts](src/reference/build-scripts.zh.md)
    - [ ] [Build Script Examples](src/reference/build-script-examples.zh.md)
  - [ ] [Publishing on crates.io](src/reference/publishing.zh.md)
  - [ ] [Package ID Specifications](src/reference/pkgid-spec.zh.md)
  - [ ] [Source Replacement](src/reference/source-replacement.zh.md)
  - [ ] [External Tools](src/reference/external-tools.zh.md)
  - [ ] [Registries](src/reference/registries.zh.md)
  - [ ] [Dependency Resolution](src/reference/resolver.zh.md)
  - [ ] [SemVer Compatibility](src/reference/semver.zh.md)
  - [ ] [Future incompat report](src/reference/future-incompat-report.zh.md)
  - [ ] [Unstable Features](src/reference/unstable.zh.md)

- [x] [Cargo Commands](src/commands/index.zh.md)

  - [x] [General Commands](src/commands/general-commands.zh.md)
    - [x] [cargo](src/commands/cargo.zh.md)
    - [x] [cargo help](src/commands/cargo-help.zh.md)
    - [x] [cargo version](src/commands/cargo-version.zh.md)
  - [x] [Build Commands](src/commands/build-commands.zh.md)
    - [ ] [cargo bench](src/commands/cargo-bench.zh.md)
    - [ ] [cargo build](src/commands/cargo-build.zh.md)
    - [ ] [cargo check](src/commands/cargo-check.zh.md)
    - [ ] [cargo clean](src/commands/cargo-clean.zh.md)
    - [ ] [cargo doc](src/commands/cargo-doc.zh.md)
    - [ ] [cargo fetch](src/commands/cargo-fetch.zh.md)
    - [ ] [cargo fix](src/commands/cargo-fix.zh.md)
    - [ ] [cargo run](src/commands/cargo-run.zh.md)
    - [ ] [cargo rustc](src/commands/cargo-rustc.zh.md)
    - [ ] [cargo rustdoc](src/commands/cargo-rustdoc.zh.md)
    - [ ] [cargo test](src/commands/cargo-test.zh.md)
    - [ ] [cargo report](src/commands/cargo-report.zh.md)
  - [ ] [Manifest Commands](src/commands/manifest-commands.zh.md)
    - [ ] [cargo generate-lockfile](src/commands/cargo-generate-lockfile.zh.md)
    - [ ] [cargo locate-project](src/commands/cargo-locate-project.zh.md)
    - [ ] [cargo metadata](src/commands/cargo-metadata.zh.md)
    - [ ] [cargo pkgid](src/commands/cargo-pkgid.zh.md)
    - [ ] [cargo tree](src/commands/cargo-tree.zh.md)
    - [ ] [cargo update](src/commands/cargo-update.zh.md)
    - [ ] [cargo vendor](src/commands/cargo-vendor.zh.md)
    - [ ] [cargo verify-project](src/commands/cargo-verify-project.zh.md)
  - [ ] [Package Commands](src/commands/package-commands.zh.md)
    - [ ] [cargo init](src/commands/cargo-init.zh.md)
    - [ ] [cargo install](src/commands/cargo-install.zh.md)
    - [ ] [cargo new](src/commands/cargo-new.zh.md)
    - [ ] [cargo search](src/commands/cargo-search.zh.md)
    - [ ] [cargo uninstall](src/commands/cargo-uninstall.zh.md)
  - [ ] [Publishing Commands](src/commands/publishing-commands.zh.md)
    - [ ] [cargo login](src/commands/cargo-login.zh.md)
    - [ ] [cargo owner](src/commands/cargo-owner.zh.md)
    - [ ] [cargo package](src/commands/cargo-package.zh.md)
    - [ ] [cargo publish](src/commands/cargo-publish.zh.md)
    - [ ] [cargo yank](src/commands/cargo-yank.zh.md)

- [ ] [FAQ](src/faq.zh.md)
- [ ] [Appendix: Glossary](src/appendix/glossary.zh.md)
- [ ] [Appendix: Git Authentication](src/appendix/git-authentication.zh.md)


### NEW: 还有本 Cargo 贡献指南

- [ ] ./contrib 就缓一缓

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[If help, **buy** me coffee —— 营养跟不上了，给我来瓶营养快线吧! 💰](https://github.com/chinanf-boy/live-need-money)

---