## Profiles

配置文件提供了一种改变编译器设置的方法，从而影响优化和调试符号等内容。

Cargo有4个内置配置文件：`dev`, `release`, `test`和`bench`。如果未在命令行上指定配置文件，则会根据正在运行的命令自动选择配置文件。除了内置配置文件外，还可以指定自定义用户定义的配置文件。

配置文件设置可以在中更改[`Cargo.toml`](manifest.md)和`[profile]`桌子在每个命名的配置文件中，可以使用如下键/值对更改各个设置：

```toml
[profile.dev]
opt-level = 1               # Use slightly better optimizations.
overflow-checks = false     # Disable integer overflow checks.
```

Cargo只查看`Cargo.toml`在工作区的根目录下显示。依赖项中定义的配置文件设置将被忽略。

此外，配置文件可以从[config]释义在配置文件或环境变量中指定配置文件将覆盖`Cargo.toml`.

[config]: config.md

### Profile settings

以下是可在配置文件中控制的设置列表。

#### opt-level

这个`opt-level`设置控制[`-C opt-level` flag]它控制着优化的水平。更高的优化级别可能会以更长的编译时间为代价生成更快的运行时代码。更高级别也可能会更改和重新排列编译后的代码，这可能会使其更难与调试器一起使用。

有效选项包括：

-   `0`：没有优化
-   `1`：基本优化
-   `2`：一些优化
-   `3`：所有优化
-   `"s"`：优化二进制大小
-   `"z"`：优化二进制大小，但也关闭循环矢量化。

建议尝试不同的级别，为项目找到合适的平衡点。可能会有令人惊讶的结果，比如`3`慢于`2`，或`"s"`和`"z"`水平不一定更小。随着时间的推移，您可能还希望重新评估您的设置，使其成为新版本的`rustc`改变优化行为。

另见[Profile Guided Optimization]获取更高级的优化技术。

[`-c opt-level` flag]: ../../rustc/codegen-options/index.html#opt-level

[profile guided optimization]: ../../rustc/profile-guided-optimization.html

#### debug

这个`debug`设置控制[`-C debuginfo` flag]它控制编译的二进制文件中包含的调试信息量。

有效选项包括：

-   `0`或`false`：完全没有调试信息
-   `1`：仅限行表
-   `2`或`true`：完整调试信息

您可能还希望配置[`split-debuginfo`](#split-debuginfo)根据您的需要选择。

[`-c debuginfo` flag]: ../../rustc/codegen-options/index.html#debuginfo

#### split-debuginfo

这个`split-debuginfo`设置控制[`-C split-debuginfo` flag]它控制调试信息（如果生成）是放在可执行文件本身还是与之相邻。

此选项是一个字符串，可接受的值与[compiler accepts][`-c split-debuginfo` flag]。此选项的默认值为`unpacked`在macOS上，用于启用调试信息的配置文件。否则，此选项的默认值为[documented with rustc][`-c split-debuginfo` flag]而且是特定于平台的。有些选项仅在[nightly channel].一旦再次进行测试，并且对DWARF的支持稳定，Cargo默认值可能会在未来发生变化。

[nightly channel]: ../../book/appendix-07-nightly-rust.html

[`-c split-debuginfo` flag]: ../../rustc/codegen-options/index.html#split-debuginfo

#### debug-assertions

这个`debug-assertions`设置控制[`-C debug-assertions` flag]哪个转弯`cfg(debug_assertions)` [conditional compilation]开或关。调试断言旨在包括仅在调试/开发版本中可用的运行时验证。在发布版本中，这些可能过于昂贵或不受欢迎。调试断言支持[`debug_assert!` macro]在标准库中。

有效选项包括：

-   `true`：启用
-   `false`：禁用

[`-c debug-assertions` flag]: ../../rustc/codegen-options/index.html#debug-assertions

[conditional compilation]: ../../reference/conditional-compilation.md#debug_assertions

[`debug_assert!` macro]: ../../std/macro.debug_assert.html

#### overflow-checks

这个`overflow-checks`设置控制[`-C overflow-checks` flag]它控制着[runtime integer overflow]。当启用溢出检查时，溢出将发生死机。

有效选项包括：

-   `true`：启用
-   `false`：禁用

[`-c overflow-checks` flag]: ../../rustc/codegen-options/index.html#overflow-checks

[runtime integer overflow]: ../../reference/expressions/operator-expr.md#overflow

#### lto

这个`lto`设置控制[`-C lto` flag]控制LLVM的[link
time optimizations].LTO可以使用整个程序分析，以更长的链接时间为代价，生成更好的优化代码。

有效选项包括：

-   `false`：执行“精简本地LTO”，仅在本地板条箱上的所有板条箱上执行“精简”LTO[codegen units](#codegen-units).如果codegen单位为1或1，则不执行LTO[opt-level](#opt-level)是0。
-   `true`或`"fat"`：执行“fat”LTO，尝试对依赖关系图中的所有板条箱执行优化。
-   `"thin"`：表演["thin" LTO]。这与“fat”类似，但运行时间大大缩短，同时仍能获得类似于“fat”的性能提升。
-   `"off"`：禁用LTO。

另见[`-C linker-plugin-lto`] `rustc`跨语言LTO的标志。

[`-c lto` flag]: ../../rustc/codegen-options/index.html#lto

[link time optimizations]: https://llvm.org/docs/LinkTimeOptimization.html

[`-c linker-plugin-lto`]: ../../rustc/codegen-options/index.html#linker-plugin-lto

["thin" lto]: http://blog.llvm.org/2016/06/thinlto-scalable-and-incremental-lto.html

#### panic

这个`panic`设置控制[`-C panic` flag]它控制着使用哪种恐慌策略。

有效选项包括：

-   `"unwind"`：惊慌失措时，松开堆叠。
-   `"abort"`：恐慌时终止进程。

当设置为`"unwind"`，实际值取决于目标平台的默认值。例如，NVPTX平台不支持退绕，所以它总是使用`"abort"`.

测试、基准测试、构建脚本和过程宏忽略了`panic`背景这个`rustc`测试线束目前需要`unwind`行为看到了吗[`panic-abort-tests`]不稳定的标志，使`abort`行为

此外，当使用`abort`策略和构建测试时，所有依赖项也将被迫与`unwind`策略

[`-c panic` flag]: ../../rustc/codegen-options/index.html#panic

[`panic-abort-tests`]: unstable.md#panic-abort-tests

#### incremental

这个`incremental`设置控制[`-C incremental` flag]它控制是否启用增量编译。增量编译原因`rustc`将其他信息保存到磁盘，以便在重新编译板条箱时重复使用，从而缩短重新编译时间。附加信息存储在`target`目录

有效选项包括：

-   `true`：启用
-   `false`：禁用

增量编译仅用于工作区成员和“路径”依赖项。

增量值可以用`CARGO_INCREMENTAL`
[environment variable]或者[`build.incremental`]配置变量。

[`-c incremental` flag]: ../../rustc/codegen-options/index.html#incremental

[environment variable]: environment-variables.md

[`build.incremental`]: config.md#buildincremental

#### codegen-units

这个`codegen-units`设置控制[`-C codegen-units` flag]控制一个板条箱将被分成多少个“代码生成单元”。更多的代码生成单元允许并行处理更多的板条箱，这可能会减少编译时间，但可能会生成较慢的代码。

此选项接受大于0的整数。

默认值为256[incremental](#incremental)构建，16个用于非增量构建。

[`-c codegen-units` flag]: ../../rustc/codegen-options/index.html#codegen-units

#### rpath

这个`rpath`设置控制[`-C rpath` flag]它控制着是否[`rpath`]已启用。

[`-c rpath` flag]: ../../rustc/codegen-options/index.html#rpath

[`rpath`]: https://en.wikipedia.org/wiki/Rpath

### Default profiles

#### dev

这个`dev`配置文件用于正常的开发和调试。这是构建命令的默认设置，如[`cargo build`].

默认设置为`dev`个人资料包括：

```toml
[profile.dev]
opt-level = 0
debug = true
split-debuginfo = '...'  # Platform-specific.
debug-assertions = true
overflow-checks = true
lto = false
panic = 'unwind'
incremental = true
codegen-units = 256
rpath = false
```

#### release

这个`release`概要文件旨在用于发布和生产中使用的优化工件。当`--release`将使用标志，并且是的默认值[`cargo install`].

默认设置为`release`个人资料包括：

```toml
[profile.release]
opt-level = 3
debug = false
split-debuginfo = '...'  # Platform-specific.
debug-assertions = false
overflow-checks = false
lto = false
panic = 'unwind'
incremental = false
codegen-units = 16
rpath = false
```

#### test

这个`test`profile是用户使用的默认配置文件[`cargo test`]这个`test`配置文件从[`dev`](#dev)轮廓

#### bench

这个`bench`profile是用户使用的默认配置文件[`cargo bench`]这个`bench`配置文件从[`release`](#release)轮廓

#### Build Dependencies

默认情况下，所有配置文件都不会优化生成依赖项（生成脚本、过程宏及其依赖项）。生成替代的默认设置为：

```toml
[profile.dev.build-override]
opt-level = 0
codegen-units = 256

[profile.release.build-override]
opt-level = 0
codegen-units = 256
```

生成依赖项以其他方式继承正在使用的活动配置文件中的设置，如中所述[Profile selection](#profile-selection).

### Custom profiles

除了内置配置文件外，还可以定义其他自定义配置文件。这些对于设置多个工作流和构建模式可能很有用。定义自定义配置文件时，必须指定`inherits`键指定未指定设置时自定义配置文件从哪个配置文件继承设置。

例如，假设您想要比较一个普通的发布版本和一个具有[LTO](#lto)在优化中，可以指定如下内容：`Cargo.toml`:

```toml
[profile.release-lto]
inherits = "release"
lto = true
```

这个`--profile`然后可以使用标志选择此自定义配置文件：

```console
cargo build --profile release-lto
```

每个概要文件的输出将被放置在与中的概要文件同名的目录中[`target` directory].如上例所示，输出将进入`target/release-lto`目录

[`target` directory]: ../guide/build-cache.md

### Profile selection

使用的配置文件取决于命令，命令行标志如`--release`或`--profile`，以及包裹（如果是[overrides](#overrides))。如果未指定任何配置文件，则默认配置文件为：

| 命令 | 默认配置文件 |
| --- | ------ |
| [`cargo run`], [`cargo build`],<br>[`cargo check`], [`cargo rustc`] | [`dev` profile](#dev) |
| [`cargo test`] | [`test` profile](#test) |
| [`cargo bench`] | [`bench` profile](#bench) |
| [`cargo install`] | [`release` profile](#release) |

您可以使用`--profile=NAME`将使用给定配置文件的选项。这个`--release`国旗相当于`--profile=release`.

所选配置文件适用于所有Cargo目标，包括[library](./cargo-targets.md#library), [binary](./cargo-targets.md#binaries),  [example](./cargo-targets.md#examples),  [test](./cargo-targets.md#tests)和[benchmark](./cargo-targets.md#benchmarks).

可以使用指定特定包的配置文件[overrides](#overrides)，如下所述。

[`cargo bench`]: ../commands/cargo-bench.md

[`cargo build`]: ../commands/cargo-build.md

[`cargo check`]: ../commands/cargo-check.md

[`cargo install`]: ../commands/cargo-install.md

[`cargo run`]: ../commands/cargo-run.md

[`cargo rustc`]: ../commands/cargo-rustc.md

[`cargo test`]: ../commands/cargo-test.md

### Overrides

可以覆盖特定软件包和构建时板条箱的配置文件设置。要覆盖特定软件包的设置，请使用`package`表更改命名包的设置：

```toml
# The `foo` package will use the -Copt-level=3 flag.
[profile.dev.package.foo]
opt-level = 3
```

包名实际上是一个[Package ID Spec](pkgid-spec.md)，因此您可以使用以下语法将包的各个版本作为目标：`[profile.dev.package."foo:2.1.0"]`.

要覆盖所有依赖项（但不是任何工作区成员）的设置，请使用`"*"`软件包名称：

```toml
# Set the default for dependencies.
[profile.dev.package."*"]
opt-level = 2
```

要覆盖生成脚本、过程宏及其依赖项的设置，请使用`build-override`表：

```toml
# Set the settings for build scripts and proc-macros.
[profile.dev.build-override]
opt-level = 3
```

> 注意：当一个依赖项既是普通依赖项又是构建依赖项时，Cargo将在以下情况下尝试只构建一次`--target`未指定。使用时`build-override`，依赖项可能需要生成两次，一次作为正常依赖项，一次使用重写的生成设置。这可能会增加初始构建时间。

使用值的优先顺序如下（第一场比赛获胜）：

1.  `[profile.dev.package.name]`-一个命名的包。
2.  `[profile.dev.package."*"]`-对于任何非工作区成员。
3.  `[profile.dev.build-override]`-仅适用于生成脚本、过程宏及其依赖项。
4.  `[profile.dev]`-设置在`Cargo.toml`.
5.  Cargo中内置的默认值。

覆盖不能指定`panic`, `lto`或`rpath`设置。

#### Overrides and generics

实例化泛型代码的位置将影响用于该泛型代码的优化设置。当使用配置文件覆盖更改特定板条箱的优化级别时，这可能会导致微妙的交互。如果试图提高定义泛型函数的依赖项的优化级别，则在本地板条箱中使用这些泛型函数时可能无法进行优化。这是因为代码可能会在实例化的板条箱中生成，因此可能会使用该板条箱的优化设置。

例如[nalgebra]是一个定义向量和矩阵的库，大量使用通用参数。如果您的本地代码定义了具体的nalgebra类型，比如`Vector4<f64>`并使用他们的方法，相应的nalgebra代码将被实例化并构建在您的板条箱中。因此，如果您试图提高`nalgebra`使用配置文件覆盖可能不会带来更快的性能。

使问题进一步复杂化，`rustc`有一些优化，它将尝试在板条箱之间共享单晶仿制药。如果opt级别为2或3，则板条箱将不会使用其他板条箱中的单形态仿制药，也不会出口本地定义的单形态产品与其他板条箱共享。当尝试优化依赖关系来开发时，请考虑尝试OPT级1，它将应用一些优化，同时仍允许共享单态化项。

[nalgebra]: https://crates.io/crates/nalgebra
