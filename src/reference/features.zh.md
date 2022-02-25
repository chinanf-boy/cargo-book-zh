## Features

Cargo “功能（features）”提供了一种[条件编译][conditional compilation]，以及[可选依赖的功能](#optional-dependencies)的机制。你可以在包中`Cargo.toml`的`[features]`表格上，定义所命名的功能，每个功能都可以启用或禁用。可以在命令行上，使用`--features`标志启用正在生成的包的功能。可以在`Cargo.toml`中的依赖项声明中，启用依赖项的功能。

另见[功能示例][features examples]第二章介绍了，如何使用功能的一些示例。

[conditional compilation]: ../../reference/conditional-compilation.md
[features examples]: features-examples.zh.md

### The `[features]` section

功能定义在`[features]`表格。每个功能都是数组，其中指定了它启用的其他功能，或可选的依赖项。以下示例说明了如何将功能用于二维图像处理库，其中可以选择，不同图像格式的支持：

```toml
[features]
# 命名为 `webp` ，不启动任何功能
webp = []
```

定义了此功能后，[`cfg` 关键表达式][`cfg` expressions]可用于在编译时，把包含功能的代码，放进编译流程。例如，`lib.rs`：

```rust
// 开启功能 webp，把 mod webp 放进编译流程
#[cfg(feature = "webp")]
pub mod webp;
```

Cargo 用 `[features]` 使用功能，而内部使用了`rustc` [`--cfg` flag]。可以通过[`cfg` attribute]或者[`cfg` macro] 测试代码的存在与否。

功能可以列出要启用的其他功能。例如，ICO 图像格式可以包含 BMP 和 PNG 图像，因此当它被启用时，应该确保其他功能也被启用：

```toml
[features]
bmp = []
png = []
ico = ["bmp", "png"]
webp = []
```

功能名称可能来自[Unicode XID standard]（其中具有大多数字母），并允许以`_`或数字`0`至`9`开头，并且在第一个字符之后还可以包含`-`, `+`或`.`。

> **注意**: [crates.io]对功能名称语法，施加额外的约束，这些约束必须是[ASCII alphanumeric]字符或`_`, `-`或`+`。

[crates.io]: https://crates.io/
[unicode xid standard]: https://unicode.org/reports/tr31/
[ascii alphanumeric]: ../../std/primitive.char.html#method.is_ascii_alphanumeric
[`--cfg` flag]: ../../rustc/command-line-arguments.md#option-cfg
[`cfg` expressions]: ../../reference/conditional-compilation.md
[`cfg` attribute]: ../../reference/conditional-compilation.md#the-cfg-attribute
[`cfg` macro]: ../../std/macro.cfg.html

### The `default` feature

默认情况下，除非明确启用，否则，所有功能都将被禁用。但，可以指定`default`功能：

```toml
[features]
default = ["ico", "webp"]
bmp = []
png = []
ico = ["bmp", "png"]
webp = []
```

构建包时，`default`功能已启用，从而启用列出的功能。这种行为可以通过以下方式改变：

- 这个`--no-default-features` ，[命令行选项][command-line flag](#command-line-feature-options)禁用包的默认功能。
- 这个`default-features = false`选项可以在[dependency
  声明](#dependency-features)。

> **注意**：在选择默认功能集合时，要小心。默认功能是一种方便，可以更轻松地使用软件包，而不必强迫用户仔细选择要启用哪些功能，但也有一些缺点。依赖项是自动启用默认功能，除非`default-features = false`是指定的，而这会导致难以确保，默认功能的禁用，尤其是对于依赖关系图中，多次出现的依赖关系。每个包都必须确保`default-features = false`的指定，以避免启用默认功能。
>
> 另一个问题是，它可能是一个[SemVer 不兼容
> 更改](#semver-compatibility)，表现为从默认设置中，删除某个功能，因此，您应该留意这些功能的安排。

### Optional dependencies

依赖项可以标记为“可选”，这意味着，默认情况下不会编译它们。例如，假设我们的 2D 图像处理库使用一个外部包，来处理 GIF 图像。可以这样表达：

```toml
[dependencies]
gif = { version = "0.11.1", optional = true }
```

可选（optional）依赖项隐式定义与依赖项同名的功能。这意味着，可以使用同样的`cfg(feature = "gif")`代码，并且可以像功能一样启用依赖项：`--features gif`（见[命令行功能选项](#command-line-feature-options)）。

> **注意**：`[feature]`表的功能名称不能与依赖项名称相同。在 nightly 版本上，可以通过[namespaced features](unstable.md#namespaced-features) 支持。

功能也可以对可选的依赖项，进行定义。只需在功能列表中，包含可选依赖项的名称。例如，假设为了支持 AVIF 图像格式，我们的库需要启用另外两个依赖项：

```toml
[dependencies]
ravif = { version = "0.6.3", optional = true }
rgb = { version = "0.8.25", optional = true }

[features]
avif = ["ravif", "rgb"]
```

在本例中，`avif`功能将启用列出的两个依赖项。

> **注意**：包括可选依赖项的另一种方法是使用[特定平台的依赖项][platform-specific dependencies]。这个方法不是使用功能，而是基于目标平台的条件编译。

[platform-specific dependencies]: specifying-dependencies.zh.md#platform-specific-dependencies

### Dependency features

依赖项的功能可以在依赖项声明中启用。这个`features`字段指示要启用的功能：

```toml
[dependencies]
# Enables the `derive` feature of serde.
serde = { version = "1.0.118", features = ["derive"] }
```

禁用[`default` 功能](#the-default-feature)，可以使用`default-features = false`:

```toml
[dependencies]
flate2 = { version = "1.0.3", default-features = false, features = ["zlib"] }
```

> **注意**：这可能无法确保禁用默认功能。如果另一个依赖项，是包括`flate2`且没有具体定义`default-features = false`，那就会启用默认功能。查看[统一功能](#feature-unification)的细节。

还可以在`[features]`中，启用依赖项的功能，语法是`"package-name/feature-name"`。例如：

```toml
[dependencies]
jpeg-decoder = { version = "0.1.20", default-features = false }

[features]
# 启动 jpeg-decoder 的 "rayon" 功能，以支持 parallel
parallel = ["jpeg-decoder/rayon"]
```

> **注意**当前的`"package-name/feature-name"`语法，还将启用`package-name`（如果是可选的依赖项）。在 nightly 版本上，可以通过[weak dependency features](unstable.zh.md#weak-dependency-features)禁用此行为。

### Command-line feature options

以下命令行标志可用于控制/启用了哪些功能：

- `--features` _功能_：启用列出的功能。多个功能可以用逗号或空格分隔。如果使用空格，请确保终端（例如`--features "foo bar"`）的括号。如果在一个[workspace]，构建多个包，这个`package-name/feature-name`语法可指定特定工作区成员的功能。

- `--all-features`：对命令行选定的所有包，激活它们的所有功能。

- `--no-default-features`：对命令行选定的包，不激活它们的[`default`
  功能](#the-default-feature)。

[workspace]: workspaces.zh.md

### Feature unification

> 功能的统一

功能对于定义它们的包来说，是独一无二的。在一个包上启用功能，不会在其他包上，启用同名功能。

当一个依赖项被多个包使用时，Cargo 在构建它时，将使用该依赖项上启用的所有功能的并集。这有助于确保只使用依赖项的一个副本。查看解析器文档的[功能章节][features section]。

例如，让我们看看[`winapi`]使用功能的[large
number][winapi-features]。如果你的包依赖于一个`foo`包，而它支持`winapi`的“fileapi”和“handleapi”功能，以及另一种依赖`bar`，它启用了`winapi`的“std”和“winnt”功能，那么`winapi`将在启用所有这四个功能的情况下，构建。

> 译者：

- winapi
  - foo
    - winapi:fileapi
    - winapi:handleapi
  - bar
    - winapi:std
    - winapi:winnt

![winapi features example](../images/winapi-features.svg)

[`winapi`]: https://crates.io/crates/winapi
[winapi-features]: https://github.com/retep998/winapi-rs/blob/0.3.9/Cargo.toml#L25-L431

这样做的一个结果是，功能应该*叠加式*的。也就是说，一个功能的弃用不会禁用它的功能性，而且启用任何功能组合通常应该是安全的。功能不应引入[SemVer-不兼容的更改](#semver-compatibility)。

例如，如果您希望可选地支持[`no_std`]环境，**不要**使用`no_std`功能，而应该是添加`std`功能：

```rust
#![no_std]

#[cfg(feature = "std")]
extern crate std;

#[cfg(feature = "std")]
pub fn function_that_requires_std() {
    // ...
}
```

[`no_std`]: ../../reference/names/preludes.html#the-no_std-attribute
[features section]: resolver.md#features

#### Mutually exclusive features

> 手动排除功能

在极少数情况下，功能可能相互不兼容。如果出现这种情况，就需要协调依赖关系图中，包的使用，以避免将它们一起启用。如果不可能，请考虑添加编译错误，来检测此场景。例如：

```rust,ignore
#[cfg(all(feature = "foo", feature = "bar"))]
compile_error!("feature \"foo\" and feature \"bar\" cannot be enabled at the same time");
```

不要使用互斥的功能，考虑其他选项：

- 将功能拆分为单独的包。
- 当发生冲突时，[决定某功能的优先级][feature-precedence]。[`cfg-if`]包可以帮助编写更复杂的`cfg`表达代码。
- 设计代码，让这些功能可以同时启动，并使用运行时选项来控制所使用的功能。例如，使用配置文件、命令行参数或环境变量来选择要启用的行为。

[`cfg-if`]: https://crates.io/crates/cfg-if
[feature-precedence]: features-examples.md#feature-precedence

#### Inspecting resolved features

> 检测解析的功能

在复杂的依赖关系图中，有时很难理解，如何在不同的软件包上，启用不同的功能。这个[`cargo tree`]命令提供了几个选项，来帮助检查与可视化哪些功能已启用。

一些可供尝试的选项：

- `cargo tree -e features`：这将在依赖关系图中，显示功能。每个功能将显示哪个软件包启用了它。
- `cargo tree -f "{p} {f}"`：这是一个更紧凑的视图，显示每个软件包上启用的功能的逗号分隔列表。
- `cargo tree -e features -i foo`：这将反转树，显示功能如何流入给定的包“foo”。这可能很有用，因为查看整个图表可能会非常大，而且会让人不知所措。当您试图找出在特定软件包上，启用了哪些功能以及为什么启用时，请使用此选项。请参见[`cargo tree`]示例，关于如何阅读这一页。

[`cargo tree`]: ../commands/cargo-tree.zh.md

### Feature resolver version 2

可以使用`Cargo.toml`下的`resolver`字段，指定不同的功能解析程序，就像这样：

```toml
[package]
name = "my-package"
version = "1.0.0"
resolver = "2"
```

[resolver versions]有关解析程序版本冲突的详细信息。

版本`"2"`在一些不需要统一功能的情况下，解析器可以避免统一功能。具体情况见[resolver 章节][resolver-v2]，但简而言之，它避免在以下情况下统一：

- [特定平台的依赖][platform-specific dependencies]上启用的功能，对于当前未构建的目标，将忽略。
- [Build-dependencies]和 proc 宏，不会与普通依赖项共享功能。
- 除非目标的构建需要（如，测试或示例），否则[Dev-dependencies]不会激活功能。

在某些情况下，避免统一是必要的。例如，如果 build-dependency 启用`std`功能，并且相同的依赖项用作`no_std`环境，那么`std`的启用会破坏构建过程。

自然，缺点是，这可能会增加构建时间，因为依赖项被构建了多次（每个都有不同的功能）。使用`"2"`解析器，建议检查多次构建的依赖项，以减少总体生成时间。如无*必要*，具有不同功能的副本包，请考虑将功能添加到`features`列表中的[dependency declaration](#dependency-features)，这样的话，副本最终具有相同的功能（因此 Cargo 只构建一次）。
您可以使用[`cargo tree --duplicates`][`cargo tree`]命令显示多次构建的包；查找列出的任何具有相同版本的条目。
查看[Inspecting resolved features](#inspecting-resolved-features) ，获取有关已解析的功能的更多信息。
对于 build-dependency 来说，如果使用`--target`标记进行跨平台编译，就没有必要了，因为在该场景中，build-dependency 总是与普通依赖项分开生成。

#### Resolver version 2 command-line flags

这个`resolver = "2"`设置也会更改`--features`和`--no-default-features` 的 [命令行选项](#command-line-feature-options)。

版本`"1"`，只能在当前工作目录中，让包启用功能。例如，在包含`foo`和`bar`软件包的工作区中，并且您位于`foo`包的目录中，然后执行命令`cargo build -p bar --features bar-feat`，这将失败，因为`--features`仅允许在`foo`上启用功能的标志。

译者：

```bash
- root workspace
  - foo $cargo build -p bar --features bar-feat
  - bar
```

具有`resolver = "2"`，features 就允许选择的任何软件包启用功能，通过`-p`和`--workspace`选项。例如：

```sh
#  resolver = "2" 可以使用下方命令，不管处于哪个目录
cargo build -p foo -p bar --features foo-feat,bar-feat

# 同等代码，但可在任一版本下，使用:
cargo build -p foo -p bar --features foo/foo-feat,bar/bar-feat
```

此外，`resolver = "1"`的`--no-default-features`标志，仅禁用当前目录中，包的默认功能。对于版本“2”，它将禁用所有工作区成员的默认功能。

[resolver versions]: resolver.zh.md#resolver-versions
[build-dependencies]: specifying-dependencies.zh.md#build-dependencies
[dev-dependencies]: specifying-dependencies.zh.md#development-dependencies
[resolver-v2]: resolver.zh.md#feature-resolver-version-2

### Build scripts

[Build scripts]可以通过检查`CARGO_FEATURE_<name>`环境变量，知道哪个功能开启，这里的`<name>`，是已转换为大写的功能名称以及`-`转成`_`。

[build scripts]: build-scripts.zh.md

### Required features

> 必要的功能

这个[`required-features` field][`required-features` 字段]可用于禁用特定的[Cargo targets][cargo targets]。有关更多详细信息，请参阅链接文档。

[`required-features` field]: cargo-targets.zh.md#the-required-features-field
[cargo targets]: cargo-targets.zh.md

### SemVer compatibility

> SemVer 不兼容性

启用功能不应引入 SemVer 不兼容的更改。例如，该功能不应以可能破坏现有用法的方式更改现有 API。有关哪些更改兼容的更多详细信息，请参阅[SemVer Compatibility chapter](semver.zh.md)。

添加和删除功能定义和可选依赖项时，应更多小心，因为这些有时可能是向后不兼容的更改。更多详细信息请在 SemVer 兼容性中，参见[Cargo 章节](semver.zh.md#cargo)。简而言之，遵循以下规则：

- 在小（minor）版本中，通常可以安全地执行以下操作：
  - 添加一个[新功能][cargo-feature-add]或[可选依赖项][cargo-dep-add]。
  - [更改一个依赖项上的功能][cargo-change-dep-feature]。
- 通常应注意以下几点：**不**在小版本中完成：
  - [移除一项功能][cargo-feature-remove]或[可选依赖项][cargo-remove-opt-dep]。
  - [移除一项功能后，现有的共有代码][item-remove]。
  - [从功能列表中，移除一项功能][cargo-feature-remove-another]。

有关注意事项和示例，请参见链接。

[cargo-change-dep-feature]: semver.zh.md#cargo-change-dep-feature
[cargo-dep-add]: semver.zh.md#cargo-dep-add
[cargo-feature-add]: semver.zh.md#cargo-feature-add
[item-remove]: semver.zh.md#item-remove
[cargo-feature-remove]: semver.zh.md#cargo-feature-remove
[cargo-remove-opt-dep]: semver.zh.md#cargo-remove-opt-dep
[cargo-feature-remove-another]: semver.zh.md#cargo-feature-remove-another

### Feature documentation and discovery

> 功能文档与发现

我们鼓励您记录软件包中，提供了哪些功能。这可以通过添加[文档注释][doc comments]完成，比如`lib.rs`的顶部。请参见[regex 的源代码][regex crate source]，渲染结果可以在[docs.rs][regex-docs-rs]查看到。如果您有其他文档，例如：用户指南，请考虑在那里添加文档（例如，参见[serde.rs]）。如果您有一个二进制项目，请考虑在项目的自述文件或其他文档中，记录功能（例如，参见[sccache]）。

清楚地记录这些功能，告诉用户功能的属性，比如：式“不稳定”的，或不应该使用的功能。例如，如果存在可选依赖项，但不希望用户将该可选依赖项明确列为功能，请将其从文档列表中排除。

[docs.rs]上的文档，可以在`Cargo.toml`中使用元数据，去控制生成文档时，启用哪些功能。查看[docs.rs 元信息文档][docs.rs metadata documentation]更多细节。

> **注意**：Rustdoc 实验性地支持注释文档，以指示使用某些 API 需要哪些功能。看[`doc_cfg`]有关更多详细信息。例如，[`syn` 文档][`syn` documentation]作为一个例子，您可以看到彩色框，其中，注明了使用它所需的功能。

[docs.rs metadata documentation]: https://docs.rs/about/metadata
[docs.rs]: https://docs.rs/
[serde.rs]: https://serde.rs/feature-flags.html
[doc comments]: ../../rustdoc/how-to-write-documentation.html
[regex crate source]: https://github.com/rust-lang/regex/blob/1.4.2/src/lib.rs#L488-L583
[regex-docs-rs]: https://docs.rs/regex/1.4.2/regex/#crate-features
[sccache]: https://github.com/mozilla/sccache/blob/0.2.13/README.md#build-requirements
[`doc_cfg`]: ../../unstable-book/language-features/doc-cfg.html
[`syn` documentation]: https://docs.rs/syn/1.0.54/syn/#modules

#### Discovering features

> 发现功能

当功能被记录在 library API 中时，用户可以更容易地发现哪些功能可用，以及它们的用途。如果软件包的功能文档不可用，可以查看`Cargo.toml`文件，但有时很难跟踪它的用途。[crates.io]上的箱子页面，有到源码存储库的链接（如果可用）。像工具[`cargo vendor`]或[cargo-clone-crate]可以用来下载源代码并检查它。

[`cargo vendor`]: ../commands/cargo-vendor.zh.md
[cargo-clone-crate]: https://crates.io/crates/cargo-clone-crate
