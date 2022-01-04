## 持续集成

### Travis CI

要在 Travis CI 上测试您的项目，这里有一个`.travis.yml`文件示例:

```yaml
language: rust
rust:
  - stable
  - beta
  - nightly
matrix:
  allow_failures:
    - rust: nightly
```

这将在所有三个 rust 版本下，进行测试，但 nightly 的任何破坏，都不会使整体构建失败。请看看[Travis CI Rust 文档](https://docs.travis-ci.com/user/languages/rust/)了解更多信息.

### GitHub Actions

GitHub Actions 中，测试项目, 这里有个简单的 `.github/workflows/ci.yml` 文件:

```yaml
name: Cargo Build & Test

on:
  push:
  pull_request:

env:
  CARGO_TERM_COLOR: always

jobs:
  build_and_test:
    name: Rust project - latest
    runs-on: ubuntu-latest
    strategy:
      matrix:
        toolchain:
          - stable
          - beta
          - nightly
    steps:
      - uses: actions/checkout@v2
      - run: rustup update ${{ matrix.toolchain }} && rustup default ${{ matrix.toolchain }}
      - run: cargo build --verbose
      - run: cargo test --verbose
```

这会测试三个 release channels (注意：一个错，整个测试流程就会是错误的)。 你还可以在 Github 上面， `"Actions" > "new workflow"` ，并选择 Rust ，将 [default configuration](https://github.com/actions/starter-workflows/blob/main/ci/rust.yml) 添加到你的库。 [GitHub Actions 文档](https://docs.github.com/en/actions)有更多信息。

### GitLab CI

要在 GitLab CI 上测试您的包，这里有一个`.gitlab-ci.yml`文件示例:

```yaml
stages:
  - build

rust-latest:
  stage: build
  image: rust:latest
  script:
    - cargo build --verbose
    - cargo test --verbose

rust-nightly:
  stage: build
  image: rustlang/rust:nightly
  script:
    - cargo build --verbose
    - cargo test --verbose
  allow_failure: true
```

这将测试 stable 版本和 nightly 版本，但 nightly 的任何破损，都不会使整体构建失败。欲获得更多信息，请看[GitLab CI](https://docs.gitlab.com/ce/ci/yaml/README.md).

### builds.sr.ht

在 sr.ht 测试你的项目, 在这里有个简单 `.build.yml` 文件。
要记得，更改你 `<your repo>` 和 `<your project>`。

```yaml
image: archlinux
packages:
  - rustup
sources:
  - <your repo>
tasks:
  - setup: |
      rustup toolchain install nightly stable
      cd <your project>/
      rustup run stable cargo fetch
  - stable: |
      rustup default stable
      cd <your project>/
      cargo build --verbose
      cargo test --verbose
  - nightly: |
      rustup default nightly
      cd <your project>/
      cargo build --verbose ||:
      cargo test --verbose  ||:
  - docs: |
      cd <your project>/
      rustup run stable cargo doc --no-deps
      rustup run nightly cargo doc --no-deps ||:
```

这会在 stable channel 和 nightly
channel 上构建文档和测试, 但 nightly 版本的一个崩溃，不会对整个构建产生影响。 [builds.sr.ht 文档](https://man.sr.ht/builds.sr.ht/) 有更多的信息。

[def-package]: ../appendix/glossary.md#package '"package" (glossary entry)'
