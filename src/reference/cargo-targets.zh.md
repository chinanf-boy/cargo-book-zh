## Cargo Targets

_目标(Target)_ 在 Cargo 中，是指代你的 crate 想成为的目标，它可以是[library](#library), [binary](#binaries), [example](#examples), [test](#tests)和[benchmark](#benchmarks)。可以在`Cargo.toml`中，明确 target 列表。当然，常常能根据（常规的）代码库目录，[自动推断](#target-auto-discovery)由[directory layout][package layout]。

下面是有关[配置目标](#configuring-a-target)的详细信息。

### Library

Cargo 目标定义了一个`库”，可以被其他库和可执行文件使用和链接。文件名默认为`src/lib.rs`，库的名称默认为包的名称。一个包只能有一个库。库的设置可以是在`Cargo.toml`的`[lib]`下，[自定义][customized].

```toml
#  Cargo.toml 示例
[lib]
crate-type = ["cdylib"]
bench = false
```

### Binaries

二进制目标是编译后，可以运行的可执行程序。默认的二进制文件名是`src/main.rs`，默认为包的名称。其他二进制文件存储在[`src/bin/`
目录][package layout]。库的设置可以是在`Cargo.toml`的`[[bin]]`下，[自定义][customized].

二进制文件可以使用 自身 Library 的公共 API。它们还链接定义于`Cargo.toml`中的[`[dependencies]`][dependencies]。

您可以使用[`cargo run`]带上`--bin <bin-name>`选项。这个目标，可以让[`cargo install`]使用，将可执行文件复制到公共位置。

```toml
# 示例 Cargo.toml.
[[bin]]
name = "cool-tool"
test = false
bench = false

[[bin]]
name = "frobnicator"
required-features = ["frobnicate"]
```

### Examples

[`examples` 目录][package layout]下，是该库提供的使用示例。在编译时，它们被放置在[`target/debug/examples` 目录][build cache]。

示例可以使用包库的公共 API。它们还与[`[dependencies]`][dependencies]和[`[dev-dependencies]`][dev-dependencies]定义于`Cargo.toml`。

默认情况下，示例是可执行二进制文件（带有`main()`函数）。您可以指定[`crate-type` 字段](#the-crate-type-field)，将示例编译为库，如下：

```toml
[[example]]
name = "foo"
crate-type = ["staticlib"]
```

您可以使用[`cargo run`]带上`--example <example-name>`选项，运行单个示例。可以用[`cargo build`]和`--example <example-name>`选项，构建示例。可以用[`cargo install`]和`--example <example-name>`选项，将可执行二进制文件复制到公共位置。默认情况下，[`cargo test`]的执行，会让 example 被编译，以此保护它们不被腐蚀。如果，你有`#[test]`函数在你运行的 example 里，设置[`test`
字段](#the-test-field)到`true`。

### Tests

Cargo 项目中，有两种类型的测试：

- _单元测试_ 就是带有[`#[test]`
  属性][test-attribute]的函数（或是启用[`test` 字段](#the-test-field)的)。这些测试可以访问目标内的私有 API。
- _集成测试_ 就是一个单独的可执行二进制文件，也包含`#[test]`函数，该函数与项目的库链接，并可以访问其*公有* API。

测试是由[`cargo test`]命令执行。默认情况下，Cargo 和`rustc`使用[libtest harness] —— 负责收集用[`#[test]` 属性][test-attribute]，再并行执行，报告每个测试的成功和失败。如果您想使用不同测试策略，研究下[ `harness`
字段](#the-harness-field)。

[libtest harness]: ../../rustc/tests/index.html

#### Integration tests

[`tests` 目录][package layout]是集成测试。[`cargo test`]的时候，Cargo 将把这些文件编译成一个单独的箱子，并执行它们。

集成测试可以使用包库的公共 API。它们还与[`[dependencies]`][dependencies]和[`[dev-dependencies]`][dev-dependencies]链接。

如果希望在多个集成测试之间共享代码，可以将其放在单独的模块中，例如`tests/common/mod.rs`，然后在每个测试，用`mod common;`导入。

每个集成测试的结果都是一个单独的可执行二进制文件，并且，[`cargo test`] 是连续运行它们。在某些情况下，这可能效率低下，因为编译可能需要更长的时间，并且在运行测试时，可能无法充分利用多个 CPU。如果您有很多集成测试，您可能需要考虑创建一个单一的集成测试，并将测试分成多个模块。libtest harness 将自动找到所有`#[test]`函数，再并行运行它们。您还可以将模块名称传递给[`cargo test`] —— 仅在该模块内运行测试。

如果有集成测试，二进制目标会自动生成。这能让集成测试，执行二进制文件来练习和测试其行为。这个`CARGO_BIN_EXE_<name>` [environment variable]在构建集成测试时，会被设置，这样它就可以使用[`env` macro]，找到可执行文件。

[environment variable]: environment-variables.zh.md#environment-variables-cargo-sets-for-crates
[`env` macro]: ../../std/macro.env.html

### Benchmarks

基准测试[`cargo bench`]命令提供了一种方式 —— 测试你代码性能。它与[tests](#tests)的梳理流程类似，每个基准函数都用`#[bench]`属性与测试类似：

- 基准放在[`benches` 目录][package layout].
- 库和二进制文件中，定义的基准函数可以访问 _私有_ API。`benches`目录下的基准，可以使用 _公有_ API。
- [`bench` 字段](#the-bench-field)定义默认情况下，以哪些目标为基准。
- [`harness` 字段](#the-harness-field)禁用内置 harness。

> **注意**：[`#[bench]`
> 属性](../../unstable-book/library-features/test.html)目前不稳定，仅在[nightly channel]。[crates.io](https://crates.io/keywords/benchmark)也有些包，可能有助于在稳定通道上，运行基准测试，例如[Criterion](https://crates.io/crates/criterion)。

### Configuring a target

`Cargo.toml`下的`[lib]`, `[[bin]]`, `[[example]]`, `[[test]]`和`[[bench]]`，支持类似的配置，去构建目标。双括号部分，如`[[bin]]`是[TOML 语法下的数组表格](https://toml.io/en/v1.0.0-rc.3#array-of-tables)，这意味着，你可以写多个`[[bin]]`，制作几个可执行文件。而`[lib]`只能指定一个库。

以下是每个目标的 TOML 设置概述，每个字段将在下面详细描述。

```toml
[lib]
name = "foo"           # The name of the target.
path = "src/lib.rs"    # The source file of the target.
test = true            # Is tested by default.
doctest = true         # Documentation examples are tested by default.
bench = true           # Is benchmarked by default.
doc = true             # Is documented by default.
plugin = false         # Used as a compiler plugin (deprecated).
proc-macro = false     # Set to `true` for a proc-macro library.
harness = true         # Use libtest harness.
edition = "2015"       # The edition of the target.
crate-type = ["lib"]   # The crate types to generate.
required-features = [] # Features required to build this target (N/A for lib).
```

#### The `name` field

这个`name`字段指定目标的名称，该名称对应于将生成工件的文件名。对于库，这是依赖项用来引用它的箱子名称。

对于`[lib]`和默认的二进制文件(`src/main.rs`)，默认为包的名称，任何破折号都替换为下划线。其他[自动识别](#target-auto-discovery)的目标，默认为目录或文件名。

这是必需的。

#### The `path` field

这个`path`字段，指定箱子的来源，路径相对于`Cargo.toml`文件

如果未指定，则基于目标名称[推断路径](#target-auto-discovery)。

#### The `test` field

这个`test`字段表明默认情况下，是否测试目标[`cargo test`]。lib, bins, 和 tests 默认值为`true`。

> **注意**：默认情况下，[`cargo test`]是构建 example 的，以确保它们继续编译，但不会测试。设`test = true`的话，我们就将它构建为一个测试，并运行示例中，任何[`#[test]`][test-attribute]定义的函数。

#### The `doctest` field

这个`doctest`字段表明：默认情况下，[文档示例]是否由[`cargo test`]测试。这只与 libraries 有关，对其他部分没有影响。library 默认值是`true`。

#### The `bench` field

这个`bench`字段表明，默认情况下，[`cargo bench`]是否对目标进行基准测试、lib, bins, 和 benchmarks 默认值为`true` 。

#### The `doc` field

这个`doc`字段表明，默认情况下，目标是否包含在由[`cargo doc`]生成的文档中。默认值是`true`用于库和二进制文件。

> **注意**：如果二进制文件的名称与库目标相同，则将跳过该二进制文件。

#### The `plugin` field

此字段用于`rustc`使用的插件，已弃用。

#### The `proc-macro` field

这个`proc-macro`字段，表示该库是一个[procedural macro]([reference][proc-macro-reference])。这只适用于`[lib]`目标

#### The `harness` field

这个`harness`字段表示，将[`--test`]传递给`rustc`，它将自动包含 libtest 库，libtest 库是用于收集和运行，标记为[`#[test]` 属性][test-attribute]或者使用`#[bench]`属性的(函数)，所有目标默认值是`true`。

如果，设置为`false`，则您负责定义`main()`函数，来运行测试和基准测试。

测试那些[`cfg(test)` 条件语句][cfg-test]，无论 harness 是否启用。

#### The `edition` field

这个`edition`字段定义，目标将使用[Rust edition]。如果未指定，则默认为`[package]`下的[`edition` 字段][package-edition]。此字段通常不应设置，仅适用于高级场景，例如，将大型软件包增量转换为新版本。

#### The `crate-type` field

这个`crate-type`字段定义，由目标生成的[crate types]。它是一个字符串数组，允许您为单个目标指定多个箱子类型。这只能为库和示例指定。二进制文件、测试和基准测试始终是`bin`箱子类型。默认设置为：

| 目标               | 箱子类型       |
| ------------------ | -------------- |
| 普通 library       | `"lib"`        |
| Proc-macro library | `"proc-macro"` |
| Example            | `"bin"`        |

可用的选项包括，`bin`, `lib`, `rlib`, `dylib`, `cdylib`, `staticlib`和`proc-macro`。你可以在[Rust Reference Manual][crate types]上，阅读更多关于不同箱子类型的信息。

#### The `required-features` field

这个`required-features`字段，需要的[features]。如果未启用任何**必需的（required）**功能，则将跳过这个目标。这只与`[[bin]]`, `[[bench]]`, `[[test]]`和`[[example]]`有关系，但是，它对`[lib]`没有影响。

```toml
[features]
# ...
postgres = []
sqlite = []
tools = []

[[bin]]
name = "my-pg-tool"
required-features = ["postgres", "tools"]
```

### Target auto-discovery

默认情况下，Cargo 会根据[文件系统的布局][package layout]。目标配置表，例如`[lib]`, `[[bin]]`, `[[test]]`, `[[bench]]`或`[[example]]`，可用于添加，不遵循标准布局的其他目标。

可以禁用自动目标发现，以便只生成手动配置的目标。若是`[package]`下的`autobins`, `autoexamples`, `autotests`或`autobenches`设为`false`，将禁用自动发现(auto-discovery)。

```toml
[package]
# ...
autobins = false
autoexamples = false
autotests = false
autobenches = false
```

只有在特殊情况下，才需要禁用自动发现。例如，如果你有一个 library，你想要一个命名`bin`的*module*，这将带来一个问题 —— Cargo 通常会尝试编译`bin`目录作为可执行文件。以下是此场景的示例布局：

```text
├── Cargo.toml
└── src
    ├── lib.rs
    └── bin
        └── mod.rs
```

防止 Cargo 推断`src/bin/mod.rs`为可执行文件，在`Cargo.toml`里面设置`autobins = false`，就可以禁用自动发现，如下：

```toml
[package]
# …
autobins = false
```

> **注意**：对于 2015 edition 的软件包 —— 如果在`Cargo.toml`中，手动定义了至少一个目标，那么自动发现的默认设置为`false`。从 2018 版开始，默认值始终为`true`。

[build cache]: ../guide/build-cache.zh.md
[rust edition]: ../../edition-guide/index.html
[`--test` flag]: ../../rustc/command-line-arguments.html#option-test
[`cargo bench`]: ../commands/cargo-bench.zh.md
[`cargo build`]: ../commands/cargo-build.zh.md
[`cargo doc`]: ../commands/cargo-doc.zh.md
[`cargo install`]: ../commands/cargo-install.zh.md
[`cargo run`]: ../commands/cargo-run.zh.md
[`cargo test`]: ../commands/cargo-test.zh.md
[cfg-test]: ../../reference/conditional-compilation.html#test
[crate types]: ../../reference/linkage.html
[crates.io]: https://crates.io/
[customized]: #configuring-a-target
[dependencies]: specifying-dependencies.zh.md
[dev-dependencies]: specifying-dependencies.zh.md#development-dependencies
[documentation examples]: ../../rustdoc/documentation-tests.html
[features]: features.zh.md
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[package layout]: ../guide/project-layout.zh.md
[package-edition]: manifest.zh.md#the-edition-field
[proc-macro-reference]: ../../reference/procedural-macros.html
[procedural macro]: ../../book/ch19-06-macros.html
[test-attribute]: ../../reference/attributes/testing.html#the-test-attribute
