# Dependency Resolution

Cargo的主要任务之一是根据每个包中指定的版本要求确定要使用的依赖项的版本。这个过程称为“依赖解析”，由“解析程序”执行。解析结果存储在`Cargo.lock`文件，该文件将依赖项“锁定”到特定版本，并随时间保持固定。

解析器试图在考虑可能冲突的需求的同时统一公共依赖关系。以下各节提供了有关如何处理这些约束以及如何使用解析器的一些详细信息。

参见本章[Specifying Dependencies]有关如何指定依赖项要求的更多详细信息。

这个[`cargo tree`]命令可用于可视化分解器的结果。

[specifying dependencies]: specifying-dependencies.md

[`cargo tree`]: ../commands/cargo-tree.md

## SemVer compatibility

Cargo用途[SemVer]用于指定版本号。这为不同版本的软件包之间的兼容性建立了一个通用约定。看到了吗[SemVer Compatibility]第章为“兼容”变更的指导。“兼容性”这个概念很重要，因为Cargo假设在不破坏构建的情况下，在兼容范围内更新依赖项应该是安全的。

如果最左边的非零主/次/补丁组件相同，则认为版本兼容。例如`1.0.3`和`1.1.0`被认为是兼容的，因此从旧版本更新到新版本应该是安全的。然而，来自`1.1.0`到`2.0.0`不允许自动生成。此约定也适用于前导零的版本。例如`0.1.0`和`0.1.2`是兼容的，但是`0.1.0`和`0.2.0`不是。同样地，`0.0.1`和`0.0.2`不兼容。

作为一个快速的复习*版本要求*Cargo用于依赖项的语法是：

| 要求 | 实例 | 等价 | 描述 |
| --- | --- | --- | --- |
| 插入符号 | `1.2.3`或`^1.2.3` | <code>>=1.2.3, \<2.0.0</code> | 至少给定值的任何SemVer兼容版本。 |
| 蒂尔德 | `~1.2` | <code>>=1.2.0, \<1.3.0</code> | 最低版本，兼容范围有限。 |
| 通配符 | `1.*` | <code>>=1.0.0, \<2.0.0</code> | 任何版本的`*`位置 |
| 等于 | `=1.2.3` | <code>=1.2.3</code> | 仅适用于指定的版本。 |
| 比较 | `>1.1` | <code>>=1.2.0</code> | 指定数字的简单数字比较。 |
| 复合物 | <code>>=1.2, \<1.5</code> | <code>>1.2.0, \<1.5.0</code> | 必须同时满足的多个需求。 |

当多个软件包为一个公共软件包指定依赖项时，只要它们在SemVer兼容范围内，解析程序就会尝试确保它们使用该公共软件包的相同版本。它还尝试使用该兼容范围内当前可用的最棒版本。例如，如果解析图中有两个包具有以下要求：

```toml
# Package A
[dependencies]
bitflags = "1.0"

# Package B
[dependencies]
bitflags = "1.1"
```

如果当时`Cargo.lock`生成的文件是`bitflags`是`1.2.1`，则两个包都将使用`1.2.1`因为它是兼容范围内最好的。如果`2.0.0`发布后，仍将使用`1.2.1`因为`2.0.0`这被认为是不相容的。

如果多个包与semver不兼容的版本有一个共同的依赖关系，Cargo将允许这样做，但将构建该依赖关系的两个单独副本。例如：

```toml
# Package A
[dependencies]
rand = "0.7"

# Package B
[dependencies]
rand = "0.6"
```

上述情况将导致A包使用最大的`0.7`释放(`0.7.3`在撰写本文时）和包B将使用最大的`0.6`释放(`0.6.5`例如）。这可能会导致潜在的问题，请参阅[Version-incompatibility hazards]第节了解更多详细信息。

不允许在同一兼容范围内有多个版本，如果在一个兼容范围内约束为两个不同的版本，则会导致冲突解决程序错误。例如，如果解析图中有两个包具有以下要求：

```toml
# Package A
[dependencies]
log = "=0.4.11"

# Package B
[dependencies]
log = "=0.4.8"
```

上述操作将失败，因为它不允许有两个单独的副本`0.4`发布`log`包

[semver]: https://semver.org/

[semver compatibility]: semver.md

[version-incompatibility hazards]: #version-incompatibility-hazards

### Version-incompatibility hazards

当一个箱子的多个版本出现在解析图中时，如果使用箱子的箱子暴露了这些箱子中的类型，则可能会导致问题。这是因为Rust编译器认为类型和项是不同的，即使它们具有相同的名称。库在发布SemVer不兼容的版本（例如，发布`2.0.0`之后`1.0.0`已投入使用），尤其是对于广泛使用的图书馆。

"[semver trick]“是一种解决方法，可以在发布突破性更改的同时保留与旧版本的兼容性。链接页面详细说明问题所在以及解决方法。简而言之，当库想要发布突破性版本时，可以发布新版本，也可以发布上一版本的点版本。”n从新版本重新导出类型。

这些不兼容通常表现为编译时错误，但有时它们只会表现为运行时错误行为。例如，假设有一个名为`foo`这两个版本最终都会出现`1.0.0`和`2.0.0`在解析图中。如果[`downcast_ref`]在库使用版本创建的对象上使用`1.0.0`，以及代码调用`downcast_ref`正在从版本向下转换为类型`2.0.0`，则downcast将在运行时失败。

如果你有一个库的多个版本，你必须确保你正在正确地使用它们，尤其是如果不同版本的类型有可能一起使用的话。这个[`cargo tree
-d`][`cargo tree`]命令可用于标识重复版本及其来源。同样，如果发布一个不兼容版本的流行库，那么考虑对生态系统的影响是很重要的。

[semver trick]: https://github.com/dtolnay/semver-trick

[`downcast_ref`]: ../../std/any/trait.Any.html#method.downcast_ref

### Pre-releases

SemVer有“预发布”的概念，版本中带有破折号，例如`1.0.0-alpha`或`1.0.0-beta`.除非明确要求，否则Cargo将避免自动使用预释放。例如，如果`1.0.0-alpha`包装`foo`发布后，则需要`foo = "1.0"`将*不*匹配，并将返回一个错误。必须指定预发布，例如`foo =
"1.0.0-alpha"`.同样[`cargo install`]除非明确要求安装，否则将避免预发布。

Cargo允许自动使用“更新的”预发布。例如，如果`1.0.0-beta`发布，然后是一个要求`foo = "1.0.0-alpha"`将允许更新到`beta`版本请注意，预发布版本可能不稳定，因此在使用它们时应小心。一些项目可能会选择在预发布版本之间发布突破性的更改。如果库不是预发行版，建议不要在库中使用预发行版依赖项。更新您的应用程序时也应小心`Cargo.lock`，并在发布前更新导致问题时做好准备。

预发布标签可以用句点分隔，以区分不同的组件。数字组件将使用数字比较。例如`1.0.0-alpha.4`将使用数字比较`4`组成部分这意味着如果`1.0.0-alpha.11`这将被选为最伟大的版本。非数字成分按字典顺序进行比较。

[`cargo install`]: ../commands/cargo-install.md

### Version metadata

SemVer有“版本元数据”的概念，在版本中有一个加号，例如`1.0.0+21AF26D3`。此元数据通常被忽略，不应在版本要求中使用。绝不应发布仅在元数据标记上不同的多个版本（注意，这是一个[known issue]具有[crates.io]这是目前允许的）。

[known issue]: https://github.com/rust-lang/crates.io/issues/1059

[crates.io]: https://crates.io/

## Other constraints

在选择和统一依赖项时，版本需求并不是解析器考虑的唯一约束。以下部分介绍了可能影响分辨率的其他一些约束。

### Features

为了产生`Cargo.lock`，解析器构建依赖关系图，就像所有[features]最重要的[workspace]成员已启用。这样可以确保在使用添加或删除功能时，任何可选的依赖项都可用，并与图的其余部分一起正确解析[`--features` command-line flag](features.md#command-line-feature-options)。解析程序将再次运行，以确定在以下情况下使用的实际功能：*汇编*箱子，基于命令行上选择的特征。

依赖关系是通过对其启用所有功能的联合来解决的。例如，如果一个包依赖于[`im`]打包[`serde`
dependency]已启用，另一个包依赖于[`rayon`
dependency]那么启用了吗`im`将在启用功能和`serde`和`rayon`箱子将包含在解析图中。如果没有包依赖于`im`有了这些特性，这些可选依赖项将被忽略，并且它们不会影响分辨率。

在工作区中构建多个包时（例如`--workspace`还是多重`-p`所有这些包的依赖项的特性都是统一的。如果您希望避免不同工作区成员的统一，那么需要通过单独的`cargo`召唤。

解析器将跳过缺少必需功能的软件包版本。例如，如果包取决于版本`^1`属于[`regex`]和[`perf` feature]，那么它可以选择的最旧版本是`1.3.0`，因为在此之前的版本不包含`perf`特色类似地，如果从新版本中删除了某个功能，那么需要该功能的软件包将被粘贴到包含该功能的旧版本中。不鼓励删除SemVer兼容版本中的功能。请注意，可选依赖项也会定义隐式功能，因此删除可选依赖项或使其成为非可选依赖项可能会导致问题，请参阅[removing an
optional dependency].

[`im`]: https://crates.io/crates/im

[`perf` feature]: https://github.com/rust-lang/regex/blob/1.3.0/Cargo.toml#L56

[`rayon` dependency]: https://github.com/bodil/im-rs/blob/v15.0.0/Cargo.toml#L47

[`regex`]: https://crates.io/crates/regex

[`serde` dependency]: https://github.com/bodil/im-rs/blob/v15.0.0/Cargo.toml#L46

[features]: features.md

[removing an optional dependency]: semver.md#cargo-remove-opt-dep

[workspace]: workspaces.md

#### Feature resolver version 2

什么时候`resolver = "2"`是在`Cargo.toml`（见[resolver
versions](#resolver-versions)下面），使用不同的特征分解器，该分解器使用不同的算法来统一特征。版本`"1"`无论在哪里指定，解析器都将统一软件包的功能。版本`"2"`在以下情况下，解析器将避免统一功能：

-   如果当前未生成目标，则不会启用目标特定依赖项的功能。例如：

    ```toml
    [dependency.common]
    version = "1.0"
    features = ["f1"]

    [target.'cfg(windows)'.dependencies.common]
    version = "1.0"
    features = ["f2"]
    ```

    在为非Windows平台构建此示例时`f2`特写会*不*将被启用。

-   在上启用的功能[build-dependencies]或者，当这些依赖项用作普通依赖项时，proc宏将无法统一。例如：

    ```toml
    [dependencies]
    log = "0.4"

    [build-dependencies]
    log = {version = "0.4", features=['std']}
    ```

    在构建构建脚本时`log`箱子将与`std`特色在构建软件包的库时，它不会启用该功能。

-   在上启用的功能[dev-dependencies]当这些依赖项用作普通依赖项时，将不会统一，除非这些开发人员依赖项当前正在生成。例如：

    ```toml
    [dependencies]
    serde = {version = "1.0", default-features = false}

    [dev-dependencies]
    serde = {version = "1.0", features = ["std"]}
    ```

    在本例中，库通常会链接到`serde`没有`std`特色然而，当作为测试或示例构建时，它将包括`std`特色例如`cargo test`或`cargo build --all-targets`将统一这些特征。请注意，依赖项中的开发依赖项始终被忽略，这仅与顶级包或工作区成员相关。

[build-dependencies]: specifying-dependencies.md#build-dependencies

[dev-dependencies]: specifying-dependencies.md#development-dependencies

[resolver-field]: features.md#resolver-versions

### `links`

这个[`links` field]用于确保只有本机库的一个副本链接到二进制文件中。解析程序将尝试找到一个图，其中每个图只有一个实例`links`名称如果无法找到满足该约束的图形，它将返回一个错误。

例如，如果一个包依赖于[`libgit2-sys`]版本`0.11`另一个取决于`0.12`，因为Cargo无法统一这些，但它们都与`git2`本地图书馆。由于这一要求，建议在使SemVer与`links`字段（如果您的库是常用库）。

[`links` field]: manifest.md#the-links-field

[`libgit2-sys`]: https://crates.io/crates/libgit2-sys

### Yanked versions

[Yanked releases][yank]是那些标记为不应使用的。当解析器构建图形时，它将忽略所有被拖动的释放，除非它们已经存在于图形中`Cargo.lock`文件

[yank]: publishing.md#cargo-yank

## Dependency updates

依赖关系解析由需要了解依赖关系图的所有Cargo命令自动执行。例如[`cargo build`]将运行冲突解决程序以发现要生成的所有依赖项。第一次运行后，结果存储在`Cargo.lock`文件后续命令将运行解析器，将依赖项锁定到中的版本`Cargo.lock` *如果可以的话*.

如果依赖项列表`Cargo.toml`已修改，例如，将依赖项的版本从`1.0`到`2.0`，然后解析器将为该依赖项选择与新需求匹配的新版本。如果新的依赖关系引入了新的需求，那么这些新需求也可能会触发额外的更新。这个`Cargo.lock`文件将用新结果更新。这个`--locked`或`--frozen`标志可用于更改此行为，以防止在需求更改时自动更新，并返回错误。

[`cargo update`]可用于更新中的条目`Cargo.lock`当新版本发布时。如果没有任何选项，它将尝试更新锁文件中的所有包。这个`-p`标志可用于针对特定包的更新，以及其他标志，如`--aggressive`或`--precise`可用于控制如何选择版本。

[`cargo build`]: ../commands/cargo-build.md

[`cargo update`]: ../commands/cargo-update.md

## Overrides

Cargo有几种机制来覆盖图中的依赖关系。这个[Overriding Dependencies]本章详细介绍了如何使用覆盖。覆盖显示为注册表的覆盖，用新条目替换已修补的版本。否则，分辨率将像正常情况一样执行。

[overriding dependencies]: overriding-dependencies.md

## Dependency kinds

包中有三种依赖关系：正常依赖关系，[build]和[dev][dev-dependencies].从解析器的角度来看，大多数情况下，它们都被视为相同的。一个区别是，非工作区成员的开发依赖关系总是被忽略，并且不会影响解决方案。

[Platform-specific dependencies]和`[target]`表被解析为所有平台都已启用。换句话说，解析程序忽略平台或`cfg`表示

[build]: specifying-dependencies.md#build-dependencies

[dev-dependencies]: specifying-dependencies.md#development-dependencies

[platform-specific dependencies]: specifying-dependencies.md#platform-specific-dependencies

### dev-dependency cycles

通常，解析器不允许循环出现在图中，但它允许循环出现在图中[dev-dependencies]例如，项目“foo”对“bar”有一个开发依赖项，而“bar”对“foo”有一个正常依赖项（通常是“路径”依赖项）。这是允许的，因为从构建工件的角度来看，实际上没有一个周期。在本例中，构建“foo”库（不需要“bar”，因为“bar”仅用于测试），然后根据“foo”构建“bar”，然后链接到“bar”构建“foo”测试。

请注意，这可能会导致令人困惑的错误。在构建库单元测试的情况下，实际上有两个库副本链接到最终的测试二进制文件中：一个是用“bar”链接的，另一个是包含单元测试的。与报告中强调的问题类似[Version-incompatibility hazards]这两种类型之间不兼容。在这种情况下，从“bar”中暴露“foo”类型时要小心，因为“foo”单元测试不会将它们与本地类型一样对待。

如果可能的话，试着将你的包拆分成多个包，并对其进行重组，使其保持严格的非循环。

## Resolver versions

通过在中指定分解器版本，可以使用不同的特征分解器算法`Cargo.toml`这样地：

```toml
[package]
name = "my-package"
version = "1.0.0"
resolver = "2"
```

版本`"1"`resolver是在1.50版本之前随货提供的原始resolver。默认值是`"2"`如果根包指定[`edition = "2021"`](manifest.md#the-edition-field)或者更新版本。否则默认为`"1"`.

版本`"2"`解析器引入了[feature
unification](#features).看[features chapter][features-2]更多细节。

解析器是影响整个工作空间的全局选项。这个`resolver`将忽略依赖项中的版本，只使用顶级包中的值。如果使用[virtual workspace]，版本应在`[workspace]`表，例如：

```toml
[workspace]
members = ["member1", "member2"]
resolver = "2"
```

[virtual workspace]: workspaces.md#virtual-manifest

[features-2]: features.md#feature-resolver-version-2

## Recommendations

以下是一些关于在包中设置版本以及指定依赖项要求的建议。这些是适用于常见情况的一般指南，但当然，有些情况可能需要指定不寻常的要求。

-   遵循[SemVer guidelines]在决定如何更新版本号，以及是否需要进行SemVer不兼容的版本更改时。
-   对依赖项使用插入符号要求，例如`"1.2.3"`，适用于大多数情况。这确保了解析器在选择版本时能够最大限度地灵活，同时保持构建兼容性。
    -   使用当前使用的版本指定所有三个组件。这有助于设置将要使用的最低版本，并确保其他用户不会最终使用旧版本的依赖项，而该依赖项可能会缺少软件包所需的内容。
    -   避免`*`要求，因为不允许在[crates.io]，并且他们可以在正常情况下引入一些破坏性的变化`cargo update`.
    -   避免过于宽泛的版本要求。例如`>=2.0.0`可以引入任何SemVer不兼容的版本，比如`5.0.0`，这可能会在将来导致构建中断。
    -   尽可能避免过于狭窄的版本要求。例如，如果指定一个波浪形要求，如`bar="~1.3"`，另一个包规定了`bar="1.4"`，这将无法解决，即使小版本应该兼容。
-   尽量使依赖项版本与库所需的实际最低版本保持最新。例如，如果您需要`bar="1.0.12"`，然后在未来的版本中，您将开始使用`1.1.0`发布“bar”，将您的依赖性要求更新为`bar="1.1.0"`.

    如果你做不到这一点，它可能不会立即明显，因为Cargo可以机会主义地选择最新版本时，你运行一个毯子`cargo
    update`。但是，如果另一个用户依赖于您的库并运行`cargo
    update -p your-library`会的*不*如果“栏”被锁定在其`Cargo.lock`。只有在依赖项声明也更新的情况下，才会更新“bar”。如果不这样做，可能会导致使用`cargo update -p`.

-   如果两个包紧密耦合，那么`=`依赖性要求可能有助于确保它们保持同步。例如，一个带有配套proc宏库的库有时会在两个库之间做出假设，如果这两个库不同步，这些假设将无法很好地工作（而且永远不会单独使用这两个库）。父库可以使用`=`并重新导出宏以便于访问。
-   `0.0.x`版本可用于永久不稳定的软件包。

一般来说，依赖项要求越严格，解析程序失败的可能性就越大。相反，如果您使用的需求过于宽松，那么发布的新版本可能会破坏构建。

[semver guidelines]: semver.md

## Troubleshooting

下面说明了您可能遇到的一些问题，以及一些可能的解决方案。

### SemVer-breaking patch release breaks the build

有时，一个项目可能会无意中发布一个带有严重变更的point版本。当用户使用`cargo update`，他们将选择这个新版本，然后他们的构建可能会中断。在这种情况下，建议项目[yank]发布，并删除SemVer破坏性更改，或将其发布为新的SemVer主要版本。

如果变更发生在第三方项目中，如果可能，尝试（礼貌地！）与项目合作解决问题。

在等待释放的过程中，一些变通方法取决于具体情况：

-   如果您的项目是最终产品（例如二进制可执行文件），请避免在中更新有问题的包`Cargo.lock`.这可以通过`--precise`加入[`cargo update`].
-   如果你发布一个二进制文件[crates.io]，则可以临时添加`=`强制依赖于特定良好版本的要求。
    -   二进制项目也可以推荐用户使用`--locked`选项[`cargo install`]使用原版`Cargo.lock`包含已知的好版本。
-   库还可以考虑发布一个临时的新版本，其具有更严格的要求，避免了麻烦的依赖性。您可能需要考虑使用范围要求（而不是`=`)避免与使用相同依赖项的其他包发生冲突的过于严格的要求。问题解决后，您可以发布另一个point release，将依赖关系放松为插入符号需求。
-   如果第三方项目看起来无法或不愿意删除该版本，那么一个选项是更新代码以与更改兼容，并更新依赖项要求以将最低版本设置为新版本。您还需要考虑这是否是一个违反您自己的库的更改程序，例如，如果它从依赖关系中公开类型。
