# rust-lang/cargo/tree/master/src/doc  [![translate-svg]][translate-list]

<!-- [![explain]][source] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 Cargo 官书文档(草稿版) 」

[中文](./readme.md) | [english](https://github.com/istankovic/cargo-docs)

---

## 校对 🀄️

<!-- doc-templite START generated -->
<!-- repo = 'rust-lang/cargo' -->
<!-- commit = 'a472e7c46f6df06f86bc3032181386087a1bb8c7' -->
<!-- time = '2018-11-27' -->
翻译的原文 | 与日期 | 最新更新 | 更多
---|---|---|---
[commit] | ⏰ 2018-11-27 | ![last] | [中文翻译][translate-list]

[last]: https://img.shields.io/github/last-commit/rust-lang/cargo.svg
[commit]: https://github.com/rust-lang/cargo/tree/a472e7c46f6df06f86bc3032181386087a1bb8c7

<!-- doc-templite END generated -->

## 切换到 https://github.com/rust-lang/cargo/tree/master/src/doc 文档

- [x] [介绍](./src/index.zh.md)

- [x] [入门](./src/getting-started/index.zh.md)

  - [x] [安装](./src/getting-started/installation.zh.md)
  - [x] [Cargo 的第一步](./src/getting-started/first-steps.zh.md)

- [x] [Cargo 指南](./src/guide/index.zh.md)

  - [x] [为什么 Cargo 存在](./src/guide/why-cargo-exists.zh.md)
  - [x] [创建一个新包](./src/guide/creating-a-new-project.zh.md)
  - [x] [处理现有包](./src/guide/working-on-an-existing-project.zh.md)
  - [x] [依赖](./src/guide/dependencies.zh.md)
  - [x] [包装布局](./src/guide/project-layout.zh.md)
  - [x] [Cargo.toml 与 Cargo.lock](./src/guide/cargo-toml-vs-cargo-lock.zh.md)
  - [x] [测试](./src/guide/tests.zh.md)
  - [x] [持续集成](./src/guide/continuous-integration.zh.md)
  - [x] [构建缓存](./src/guide/build-cache.zh.md)

- [x] [Cargo 参考](./src/reference/index.zh.md)

  - [x] [指定依赖项](./src/reference/specifying-dependencies.zh.md)
  - [ ] [清单格式](./src/reference/manifest.zh.md)
  - [x] [配置](./src/reference/config.zh.md)
  - [x] [环境变量](./src/reference/environment-variables.zh.md)
  - [ ] [构建脚本](./src/reference/build-scripts.zh.md)
  - [x] [在 crates.io 上发布](./src/reference/publishing.zh.md)
  - [x] [包 ID 规范](./src/reference/pkgid-spec.zh.md)
  - [x] [来源更换](./src/reference/source-replacement.zh.md)
  - [x] [外部工具](./src/reference/external-tools.zh.md)
  - [x] [不稳定的功能](./src/reference/unstable.zh.md)

- [x] [常问问题](./src/faq.zh.md)
- [x] [附录:词汇表](./src/appendix/glossary.zh.md)


### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

# Cargo docs

### 要求

建立这本书需要[mdBook]。安装它:

[mdBook]: https://github.com/azerupi/mdBook

```bash
$ cargo install mdBook
```

### 构建

构建:

```bash
$ mdbook build
```

将输出在`docs`子目录。要查看它,可以在 Web 浏览器中打开它.

_Firefox:_

```bash
$ firefox docs/index.html                       # Linux
$ open -a "Firefox" docs/index.html             # OS X
$ Start-Process "firefox.exe" .\docs\index.html # Windows (PowerShell)
$ start firefox.exe .\docs\index.html           # Windows (Cmd)
```

_Chrome:_

```bash
$ google-chrome docs/index.html                 # Linux
$ open -a "Google Chrome" docs/index.html       # OS X
$ Start-Process "chrome.exe" .\docs\index.html  # Windows (PowerShell)
$ start chrome.exe .\docs\index.html            # Windows (Cmd)
```

## 贡献

鉴于该书仍处于草案状态,我们非常乐意为您提供帮助! 请随时打开有关任何事情的问题,并发送 PR 以了解您想要修复或更改的内容.如果您的变化很大,请先打开一个问题,这样我们就可以确保在您完成 PR 的工作之前，我们会接受这个问题.
