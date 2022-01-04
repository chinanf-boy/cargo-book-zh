## Cargo Home

The "Cargo home" 的作用，就是作为下载和源代码的缓存位置。
当构建一个 [crate][def-crate] 的时候, Cargo 会将下载的 依赖项，存储到 Cargo home.
通过 `CARGO_HOME` [environmental variable][env] 设置。
如果有需要，官方提供 [home](https://crates.io/crates/home) 箱子，获取到位置.
默认情况下，位置是 `$HOME/.cargo/`。

号外， Cargo home 内部结构尚未稳定，留意留意。

The Cargo home 由以下部分组成：

## Files:

- `config.toml`
  Cargo's 全局配置文件，参考书上会更详细点 [config entry][config].

- `credentials.toml`
  来自[`cargo login`] 的私有登录证书，为了登录 [registry][def-registry].

- `.crates.toml`, `.crates2.json`
  这些隐藏的文件包含 [package][def-package] 信息 —— 通过[`cargo install`]下载的。这文件不要手动修改。

## 目录:

- `bin`
  通过 [`cargo install`] or [`rustup`](https://rust-lang.github.io/rustup/)下载，箱子的可执行文件。
  为了能够访问到这些文件，请将这个目录路径添加到`$PATH`环境变量。

- `git`
  Git sources are stored here:

  - `git/db`
    当一个箱子依赖一个 git repository, Cargo clones the repo 到这个目录；需要时则更新。

  - `git/checkouts`
    这里就是放着 `git/db` 里面，git repo 具体的 commit。
    它给编译器供货。
    同一个 git 存储库，不同 commit 的多个 checkout，是可能的

- `registry`
  箱子来源的包和元信息 (such as [crates.io](https://crates.io/)) 会在这里.

  - `registry/index`
    是一个git repo，它包含一个来源所有可用箱子的元信息 (versions, dependencies etc)。

  - `registry/cache`
    下载的依赖项就会存储在这里。箱子会被压缩成 gzip 格式，而后缀名是 `.crate`。

  - `registry/src`
    自然，如果一个 `.crate` 是被需要的, 就会解压缩，放到 `registry/src`，让`rustc`能够找到`.rs`文件

## Caching the Cargo home in CI

避免重复下载，所有的箱子依赖，在持续集成中，你可以缓存 `$CARGO_HOME` 目录。
然而，缓存整个目录非常低效率，因为它会包含两遍的下载文件。
比如：使用 `serde 1.0.92`，再缓存整个 `$CARGO_HOME`，那么实际上，我们是缓存了源代码两次：`registry/cache` 的 `serde-1.0.92.crate` ， 和`registry/src`的 `.rs` 。
自然，重复执行构建步骤，会让 CI 服务器花上些事件。 

所以答案是，只要缓存下面的就好：

- `bin/`
- `registry/index/`
- `registry/cache/`
- `git/db/`

## Vendoring all dependencies of a project

See the [`cargo vendor`] subcommand.

## Clearing the cache

理论上，你可以对缓存执行许多操作，但

最好使用 [cargo-cache](https://crates.io/crates/cargo-cache)，它提供简单的命令工具，清除缓存中的所选项，或展示大小。

[`cargo install`]: ../commands/cargo-install.md
[`cargo login`]: ../commands/cargo-login.md
[`cargo vendor`]: ../commands/cargo-vendor.md
[config]: ../reference/config.md
[def-crate]: ../appendix/glossary.md#crate '"crate" (glossary entry)'
[def-package]: ../appendix/glossary.md#package '"package" (glossary entry)'
[def-registry]: ../appendix/glossary.md#registry '"registry" (glossary entry)'
[env]: ../reference/environment-variables.md
