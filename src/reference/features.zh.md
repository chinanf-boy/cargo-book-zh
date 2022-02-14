## Features

Cargo“特征”提供了一种表达[conditional compilation]和[optional dependencies](#optional-dependencies).一个包在`[features]`表格`Cargo.toml`，每个功能都可以启用或禁用。可以在命令行上使用以下标志启用正在生成的包的功能：`--features`。可以在中的依赖项声明中启用依赖项功能`Cargo.toml`.

另见[Features Examples]第二章介绍了如何使用功能的一些示例。

[conditional compilation]: ../../reference/conditional-compilation.md

[features examples]: features-examples.md

### The `[features]` section

特征定义在`[features]`入座`Cargo.toml`。每个功能都指定了它启用的其他功能或可选依赖项的数组。以下示例说明了如何将功能用于二维图像处理库，其中可以选择包括对不同图像格式的支持：

```toml
[features]
# Defines a feature named `webp` that does not enable any other features.
webp = []
```

定义了此功能后，[`cfg` expressions]可用于在编译时有条件地包含代码以支持请求的功能。例如，内部`lib.rs`一揽子计划的内容可能包括：

```rust
// This conditionally includes a module which implements WEBP support.
#[cfg(feature = "webp")]
pub mod webp;
```

使用`rustc` [`--cfg` flag]，代码可以通过[`cfg` attribute]或者[`cfg` macro].

功能可以列出要启用的其他功能。例如，ICO图像格式可以包含BMP和PNG图像，因此当它被启用时，应该确保其他功能也被启用：

```toml
[features]
bmp = []
png = []
ico = ["bmp", "png"]
webp = []
```

功能名称可能包括来自[Unicode XID standard]（其中包括大多数字母），并允许以`_`或数字`0`通过`9`，并且在第一个字符之后还可以包含`-`, `+`或`.`.

> **笔记**: [crates.io]对功能名称语法施加额外的约束，这些约束必须是[ASCII alphanumeric]角色或`_`, `-`或`+`.

[crates.io]: https://crates.io/

[unicode xid standard]: https://unicode.org/reports/tr31/

[ascii alphanumeric]: ../../std/primitive.char.html#method.is_ascii_alphanumeric

[`--cfg` flag]: ../../rustc/command-line-arguments.md#option-cfg

[`cfg` expressions]: ../../reference/conditional-compilation.md

[`cfg` attribute]: ../../reference/conditional-compilation.md#the-cfg-attribute

[`cfg` macro]: ../../std/macro.cfg.html

### The `default` feature

默认情况下，除非明确启用，否则所有功能都将被禁用。可以通过指定`default`特色：

```toml
[features]
default = ["ico", "webp"]
bmp = []
png = []
ico = ["bmp", "png"]
webp = []
```

构建包时`default`功能已启用，从而启用列出的功能。这种行为可以通过以下方式改变：

-   这个`--no-default-features` [command-line
    flag](#command-line-feature-options)禁用软件包的默认功能。
-   这个`default-features = false`选项可以在[dependency
    declaration](#dependency-features).

> **笔记**：在选择默认功能集时要小心。默认功能是一种方便，可以更轻松地使用软件包，而不必强迫用户仔细选择要启用哪些功能以供通用，但也有一些缺点。依赖项自动启用默认功能，除非`default-features = false`是指定的。这会导致难以确保未启用默认功能，尤其是对于依赖关系图中多次出现的依赖关系。每个包裹都必须确保`default-features = false`指定以避免启用它们。
>
> 另一个问题是，它可能是一个[SemVer incompatible
> change](#semver-compatibility)从默认设置中删除某个功能，因此您应该确信将保留这些功能。

### Optional dependencies

依赖项可以标记为“可选”，这意味着默认情况下不会编译它们。例如，假设我们的2D图像处理库使用一个外部包来处理GIF图像。这可以这样表达：

```toml
[dependencies]
gif = { version = "0.11.1", optional = true }
```

可选依赖项隐式定义与依赖项同名的功能。这意味着同样的`cfg(feature = "gif")`可以在代码中使用语法，并且可以像以下功能一样启用依赖项：`--features gif`（见[Command-line feature
options](#command-line-feature-options)下面）。

> **笔记**：中的一个功能`[feature]`表不能使用与依赖项相同的名称。在夜间频道上，可以通过[namespaced
> features](unstable.md#namespaced-features).

明确定义的功能也可以启用可选的依赖项。只需在功能列表中包含可选依赖项的名称。例如，假设为了支持AVIF图像格式，我们的库需要启用另外两个依赖项：

```toml
[dependencies]
ravif = { version = "0.6.3", optional = true }
rgb = { version = "0.8.25", optional = true }

[features]
avif = ["ravif", "rgb"]
```

在本例中`avif`特性将启用列出的两个依赖项。

> **笔记**：可选包含依赖项的另一种方法是使用[platform-specific dependencies].这些功能不是使用功能，而是基于目标平台的条件。

[platform-specific dependencies]: specifying-dependencies.md#platform-specific-dependencies

### Dependency features

依赖项的功能可以在依赖项声明中启用。这个`features`键指示要启用的功能：

```toml
[dependencies]
# Enables the `derive` feature of serde.
serde = { version = "1.0.118", features = ["derive"] }
```

这个[`default` features](#the-default-feature)可以使用禁用`default-features = false`:

```toml
[dependencies]
flate2 = { version = "1.0.3", default-features = false, features = ["zlib"] }
```

> **笔记**：这可能无法确保禁用默认功能。如果另一个依赖项包括`flate2`没有具体说明`default-features = false`，则会启用默认功能。查看[feature
> unification](#feature-unification)下面是更多细节。

还可以在中启用依赖项的功能`[features]`桌子语法是`"package-name/feature-name"`.例如：

```toml
[dependencies]
jpeg-decoder = { version = "0.1.20", default-features = false }

[features]
# Enables parallel processing support by enabling the "rayon" feature of jpeg-decoder.
parallel = ["jpeg-decoder/rayon"]
```

> **笔记**当前位置`"package-name/feature-name"`语法还将启用`package-name`如果是可选的依赖项。在夜间频道上，可以通过[weak
> dependency features](unstable.md#weak-dependency-features).

### Command-line feature options

以下命令行标志可用于控制启用了哪些功能：

-   `--features` *特征*：启用列出的功能。多个要素可以用逗号或空格分隔。如果使用空格，请确保在从外壳（例如`--features
    "foo bar"`).如果在一个系统中构建多个包[workspace]这个`package-name/feature-name`语法可用于指定特定工作区成员的功能。

-   `--all-features`：激活命令行上选定的所有软件包的所有功能。

-   `--no-default-features`：不激活[`default`
    feature](#the-default-feature)所选软件包的名称。

[workspace]: workspaces.md

### Feature unification

功能对于定义它们的包来说是独一无二的。在包上启用功能不会在其他包上启用同名功能。

当一个依赖项被多个包使用时，Cargo在构建它时将使用该依赖项上启用的所有功能的并集。这有助于确保只使用依赖项的一个副本。看到了吗[features section]有关详细信息，请参阅解析器文档。

例如，让我们看看[`winapi`]使用[large
number][winapi-features]有很多特点。如果你的包裹依赖于一个包裹`foo`它支持的“fileapi”和“handleapi”功能`winapi`，以及另一种依赖关系`bar`它启用了的“std”和“winnt”功能`winapi`然后`winapi`将在启用所有这四个功能的情况下构建。

![winapi features example](../images/winapi-features.svg)

[`winapi`]: https://crates.io/crates/winapi

[winapi-features]: https://github.com/retep998/winapi-rs/blob/0.3.9/Cargo.toml#L25-L431

这样做的一个结果是，功能应该*添加剂*.也就是说，启用功能不应禁用功能，而且启用任何功能组合通常应该是安全的。功能不应引入[SemVer-incompatible change](#semver-compatibility).

例如，如果您希望可选地支持[`no_std`]环境，**不要**使用`no_std`特色相反，使用`std`特色*使能够*
`std`.例如：

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

在极少数情况下，功能可能相互不兼容。如果可能的话，应该避免这种情况，因为它需要协调依赖关系图中包的所有使用，以避免将它们一起启用。如果不可能，请考虑添加编译错误来检测此场景。例如：

```rust,ignore
#[cfg(all(feature = "foo", feature = "bar"))]
compile_error!("feature \"foo\" and feature \"bar\" cannot be enabled at the same time");
```

不要使用互斥的特征，考虑其他选项：

-   将功能拆分为单独的包。
-   当发生冲突时，[choose one feature over
    another][feature-precedence]这个[`cfg-if`]软件包可以帮助编写更复杂的代码`cfg`表达。
-   设计代码以允许同时启用这些功能，并使用运行时选项来控制所使用的功能。例如，使用配置文件、命令行参数或环境变量来选择要启用的行为。

[`cfg-if`]: https://crates.io/crates/cfg-if

[feature-precedence]: features-examples.md#feature-precedence

#### Inspecting resolved features

在复杂的依赖关系图中，有时很难理解如何在不同的软件包上启用不同的功能。这个[`cargo tree`]命令提供了几个选项来帮助检查和可视化哪些功能已启用。一些可供尝试的选项：

-   `cargo tree -e features`：这将在依赖关系图中显示功能。每个功能将显示哪个软件包启用了它。
-   `cargo tree -f "{p} {f}"`：这是一个更紧凑的视图，显示每个软件包上启用的功能的逗号分隔列表。
-   `cargo tree -e features -i foo`：这将反转树，显示功能如何流入给定的包“foo”。这可能很有用，因为查看整个图表可能会非常大，而且会让人不知所措。当您试图找出在特定软件包上启用了哪些功能以及为什么启用时，请使用此选项。请参见本文底部的示例[`cargo tree`]关于如何阅读这一页。

[`cargo tree`]: ../commands/cargo-tree.md

### Feature resolver version 2

可以使用指定不同的功能分解程序`resolver`现场`Cargo.toml`，就像这样：

```toml
[package]
name = "my-package"
version = "1.0.0"
resolver = "2"
```

看到了吗[resolver versions]有关指定冲突解决程序版本的详细信息，请参见第节。

版本`"2"`在一些不需要统一特性的情况下，解析器可以避免统一特性。具体情况见下表[resolver chapter][resolver-v2]，但简而言之，它避免在以下情况下统一：

-   在上启用的功能[platform-specific dependencies]对于当前未构建的目标，将忽略。
-   [Build-dependencies]proc宏与普通依赖项不共享功能。
-   [Dev-dependencies]除非构建了需要的目标（如测试或示例），否则不要激活功能。

在某些情况下，避免统一是必要的。例如，如果生成依赖项启用`std`特性，并且相同的依赖项用作`no_std`环境，使能`std`会破坏建筑。

然而，一个缺点是，这可能会增加构建时间，因为依赖项被构建了多次（每个都有不同的功能）。使用该版本时`"2"`解析器，建议检查多次生成的依赖项，以减少总体生成时间。如果不是*必修的*要构建具有不同特性的复制包，请考虑将特性添加到`features`列表中的[dependency
declaration](#dependency-features)因此，复制品最终具有相同的特性（因此Cargo只构建一次）。您可以使用[`cargo tree --duplicates`][`cargo tree`]命令它将显示多次构建的包；查找列出的任何具有相同版本的条目。查看[Inspecting resolved
features](#inspecting-resolved-features)获取有关已解析功能的更多信息。对于生成依赖项，如果使用`--target`标记，因为在该场景中，生成依赖项总是与正常依赖项分开生成。

#### Resolver version 2 command-line flags

这个`resolver = "2"`设置也会更改`--features`和`--no-default-features` [command-line options](#command-line-feature-options).

有版本`"1"`，则只能在当前工作目录中为包启用功能。例如，在包含包的工作区中`foo`和`bar`，并且您位于包的目录中`foo`，然后执行命令`cargo build -p bar --features bar-feat`，这将失败，因为`--features`仅允许在上启用功能的标志`foo`.

具有`resolver = "2"`，features标志允许为在命令行上选择的任何软件包启用功能`-p`和`--workspace`选项。例如：

```sh
# This command is allowed with resolver = "2", regardless of which directory
# you are in.
cargo build -p foo -p bar --features foo-feat,bar-feat

# This explicit equivalent works with any resolver version:
cargo build -p foo -p bar --features foo/foo-feat,bar/bar-feat
```

此外`resolver = "1"`这个`--no-default-features`flag仅禁用当前目录中包的默认功能。对于版本“2”，它将禁用所有工作区成员的默认功能。

[resolver versions]: resolver.md#resolver-versions

[build-dependencies]: specifying-dependencies.md#build-dependencies

[dev-dependencies]: specifying-dependencies.md#development-dependencies

[resolver-v2]: resolver.md#feature-resolver-version-2

### Build scripts

[Build scripts]可以通过检查`CARGO_FEATURE_<name>`环境变量，在哪里`<name>`功能名称是否已转换为大写和小写`-`皈依`_`.

[build scripts]: build-scripts.md

### Required features

这个[`required-features` field]可用于禁用特定的[Cargo
targets]如果功能未启用。有关更多详细信息，请参阅链接文档。

[`required-features` field]: cargo-targets.md#the-required-features-field

[cargo targets]: cargo-targets.md

### SemVer compatibility

启用功能不应引入SemVer不兼容的更改。例如，该功能不应以可能破坏现有用途的方式更改现有API。有关哪些更改兼容的更多详细信息，请参阅[SemVer Compatibility chapter](semver.md).

添加和删除功能定义和可选依赖项时应小心，因为这些有时可能是向后不兼容的更改。更多详细信息请参见[Cargo section](semver.md#cargo)在SemVer兼容性章节中。简而言之，遵循以下规则：

-   在小版本中，通常可以安全地执行以下操作：
    -   添加一个[new feature][cargo-feature-add]或[optional dependency][cargo-dep-add].
    -   [Change the features used on a dependency][cargo-change-dep-feature].
-   通常应注意以下几点：**不**在小版本中完成：
    -   [Remove a feature][cargo-feature-remove]或[optional dependency][cargo-remove-opt-dep].
    -   [Moving existing public code behind a feature][item-remove].
    -   [Remove a feature from a feature list][cargo-feature-remove-another].

有关注意事项和示例，请参见链接。

[cargo-change-dep-feature]: semver.md#cargo-change-dep-feature

[cargo-dep-add]: semver.md#cargo-dep-add

[cargo-feature-add]: semver.md#cargo-feature-add

[item-remove]: semver.md#item-remove

[cargo-feature-remove]: semver.md#cargo-feature-remove

[cargo-remove-opt-dep]: semver.md#cargo-remove-opt-dep

[cargo-feature-remove-another]: semver.md#cargo-feature-remove-another

### Feature documentation and discovery

我们鼓励您记录软件包中提供的功能。这可以通过添加[doc comments]在顶部`lib.rs`.作为示例，请参见[regex crate source]，渲染时可以在[docs.rs][regex-docs-rs]如果您有其他文档，例如用户指南，请考虑在那里添加文档（例如，参见[serde.rs]如果您有一个二进制项目，请考虑在项目的自述文件或其他文档中记录特性（例如，参见[sccache]).

清楚地记录这些特性可以设定对被认为“不稳定”或不应该使用的特性的期望。例如，如果存在可选依赖项，但不希望用户将该可选依赖项明确列为功能，请将其从文档列表中排除。

发表于[docs.rs]可以在中使用元数据`Cargo.toml`控制在生成文档时启用哪些功能。查看[docs.rs metadata documentation]更多细节。

> **笔记**：Rustdoc实验性地支持注释文档，以指示使用某些API需要哪些功能。看到了吗[`doc_cfg`]有关更多详细信息，请参阅文档。例如[`syn`
> documentation]，您可以看到彩色框，其中注明了使用它所需的功能。

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

当特性被记录在库API中时，用户可以更容易地发现哪些特性可用以及它们的用途。如果软件包的功能文档不可用，可以查看`Cargo.toml`文件，但有时很难找到它。箱子页面[crates.io]有到源存储库的链接（如果可用）。像这样的工具[`cargo vendor`]或[cargo-clone-crate]可以用来下载源代码并检查它。

[`cargo vendor`]: ../commands/cargo-vendor.md

[cargo-clone-crate]: https://crates.io/crates/cargo-clone-crate
