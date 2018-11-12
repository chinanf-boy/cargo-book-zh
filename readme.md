# istankovic/cargo-book [![explain]][source] [![translate-svg]][translate-list]

<!-- [![size-img]][size] -->

[explain]: http://llever.com/explain.svg
[source]: https://github.com/chinanf-boy/Source-Explain
[translate-svg]: http://llever.com/translate.svg
[translate-list]: https://github.com/chinanf-boy/chinese-translate-list
[size-img]: https://packagephobia.now.sh/badge?p=Name
[size]: https://packagephobia.now.sh/result?p=Name

「 Cargo 官书文档(草稿版) 」

[中文](./readme.md) | [english](https://github.com/istankovic/cargo-book)

---

## 校对 🀄️

<!-- doc-templite START generated -->
<!-- repo = 'istankovic/cargo-book' -->
<!-- commit = 'bd26d573f3044b11abfb0555cb308e5f0e3ecf64' -->
<!-- time = '2017-08-05' -->

| 翻译的原文 | 与日期        | 最新更新 | 更多                       |
| ---------- | ------------- | -------- | -------------------------- |
| [commit]   | ⏰ 2017-08-05 | ![last]  | [中文翻译][translate-list] |

[last]: https://img.shields.io/github/last-commit/istankovic/cargo-book.svg
[commit]: https://github.com/istankovic/cargo-book/tree/bd26d573f3044b11abfb0555cb308e5f0e3ecf64

<!-- doc-templite END generated -->

# 概要

- [x] readme
- [x] [SUMMARY](./src/SUMMARY.md)
- [ ] [入门](./src/getting-started.zh.md)
  - [ ] [安装](./src/01-01-installation.zh.md)
  - [ ] [Cargo 的第一步](./src/01-02-first-steps.zh.md)
- [ ] [指南](./src/guide.zh.md)
  - [ ] [为什么,Cargo 存在](./src/02-01-why-cargo-exists.zh.md)
  - [ ] [创建一个新项目](./src/02-02-creating-a-new-project.zh.md)
  - [ ] [在现有的 Cargo 项目上工作](./src/02-03-working-on-an-existing-project.zh.md)
  - [ ] [依赖](./src/02-04-dependencies.zh.md)
  - [ ] [项目布局](./src/02-05-project-layout.zh.md)
  - [ ] [测试](./src/02-06-tests.zh.md)
  - [ ] [持续集成](./src/02-07-continuous-integration.zh.md)
- [ ] [Cargo 深度](./src/cargo-in-depth.zh.md)
  - [ ] [指定依赖项](./src/03-01-specifying-dependencies.zh.md)
  - [ ] [Cargo.toml 格式](./src/03-02-manifest.zh.md)
  - [ ] [配置](./src/03-03-config.zh.md)
  - [ ] [环境变量](./src/03-04-environment-variables.zh.md)
  - [ ] [构建脚本](./src/03-05-build-scripts.zh.md)
  - [ ] [发布到 crates.io ](./src/03-06-crates-io.zh.md)
  - [ ] [包 ID 规格](./src/03-07-pkgid-spec.zh.md)
  - [ ] [源，更换](./src/03-08-source-replacement.zh.md)
  - [ ] [外部工具](./src/03-09-external-tools.zh.md)
  - [ ] [政策](./src/03-10-policies.zh.md)
- [ ] [常见问题](./src/faq.zh.md)

### 贡献

欢迎 👏 勘误/校对/更新贡献 😊 [具体贡献请看](https://github.com/chinanf-boy/chinese-translate-list#贡献)

## 生活

[help me live , live need money 💰](https://github.com/chinanf-boy/live-need-money)

---

# Cargo Book

### 要求

建立这本书需要[mdBook]。拿到并使用它:

[mdbook]: https://github.com/azerupi/mdBook

```bash
$ cargo install mdbook
```

### 构建

构建:

```bash
$ mdbook build
```

自动开启浏览器，查看:

```
mdbook serve --open
```

输出将在`book`子目录。要查看它,请在 Web 浏览器中打开它.

_火狐:_

```bash
$ firefox book/index.html                       # Linux
$ open -a "Firefox" book/index.html             # OS X
$ Start-Process "firefox.exe" .\book\index.html # Windows (PowerShell)
$ start firefox.exe .\book\index.html           # Windows (Cmd)
```

_Chrome:_

```bash
$ google-chrome book/index.html                 # Linux
$ open -a "Google Chrome" book/index.html       # OS X
$ Start-Process "chrome.exe" .\book\index.html  # Windows (PowerShell)
$ start chrome.exe .\book\index.html            # Windows (Cmd)
```

## 贡献

鉴于该书仍处于草案状态,我们非常乐意为您提供帮助! 请随时打开有关任何事情的问题,并发送 PR 以了解您想要修复或更改的内容.如果您的变化很大,请先打开一个问题,这样我们就可以确保在您完成 PR 的工作之前，我们会接受这个问题.
