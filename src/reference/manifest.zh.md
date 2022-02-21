## The Manifest Format

> 清单格式

每个包的这个`Cargo.toml`文件称为*清单*，是 [TOML] 格式的文件。特点是：每个清单文件由一个或多个部分(表格)组成：

- [`cargo-features`](unstable.zh.md) — 不稳定, nightly-only 的特性.
- [`[package]`](#the-package-section) — 定义一个包
  - [`name`](#the-name-field) — 名称
  - [`version`](#the-version-field) — 版本
  - [`authors`](#the-authors-field) — 作者
  - [`edition`](#the-edition-field) — Rust 编写版本
  - [`rust-version`](#the-rust-version-field) — 所支持 Rust 的最小版本
  - [`description`](#the-description-field) — 描述
  - [`documentation`](#the-documentation-field) — 文档
  - [`readme`](#the-readme-field) — readme 文件的路径
  - [`homepage`](#the-homepage-field) — 主页网址
  - [`repository`](#the-repository-field) — 源代码
  - [`license`](#the-license-and-license-file-fields) — 代码协议
  - [`license-file`](#the-license-and-license-file-fields) — 协议文件的路径
  - [`keywords`](#the-keywords-field) — 包的关键字
  - [`categories`](#the-categories-field) — 分类
  - [`workspace`](#the-workspace-field) — 工作区的路径
  - [`build`](#the-build-field) — 构建脚本的路径
  - [`links`](#the-links-field) — 包要链接的原生库的名称
  - [`exclude`](#the-exclude-and-include-fields) — 不推送的文件
  - [`include`](#the-exclude-and-include-fields) — 推送的文件
  - [`publish`](#the-publish-field) — 能用来，阻止推送包
  - [`metadata`](#the-metadata-table) — 为外部工具，提供额外设置
  - [`default-run`](#the-default-run-field) — [`cargo run`]所执行的默认二进制文件
  - [`autobins`](cargo-targets.zh.md#target-auto-discovery) — 禁用 binary 自动发现
  - [`autoexamples`](cargo-targets.zh.md#target-auto-discovery) — 禁用 example 自动发现
  - [`autotests`](cargo-targets.zh.md#target-auto-discovery) — 禁用 test 自动发现
  - [`autobenches`](cargo-targets.zh.md#target-auto-discovery) — 禁用 bench 自动发现
  - [`resolver`](resolver.zh.md#resolver-versions) — 设置使用的依赖解析器
- Target tables: (查看 [configuration](cargo-targets.zh.md#configuring-a-target) 的配置)
  - [`[lib]`](cargo-targets.zh.md#library) — Library 的设置
  - [`[[bin]]`](cargo-targets.zh.md#binaries) — Binary 的设置
  - [`[[example]]`](cargo-targets.zh.md#examples) — Example 的设置
  - [`[[test]]`](cargo-targets.zh.md#tests) — Test 的设置
  - [`[[bench]]`](cargo-targets.zh.md#benchmarks) — Benchmark 的设置
- Dependency tables:
  - [`[dependencies]`](specifying-dependencies.zh.md) — 包的依赖项
  - [`[dev-dependencies]`](specifying-dependencies.zh.md#development-dependencies) — 给 examples, tests, and benchmarks 用的依赖项
  - [`[build-dependencies]`](specifying-dependencies.zh.md#build-dependencies) — 给 build scripts 用的依赖项
  - [`[target]`](specifying-dependencies.zh.md#platform-specific-dependencies) — Platform-specific dependencies 用的依赖项
- [`[badges]`](#the-badges-section) — 展示的徽标
- [`[features]`](features.zh.md) — 构建的条件式功能
- [`[patch]`](overriding-dependencies.zh.md#the-patch-section) — 覆盖依赖项
- [`[replace]`](overriding-dependencies.zh.md#the-replace-section) — Override dependencies (弃用).
- [`[profile]`](profiles.zh.md) — 编译器的设置与优化
- [`[workspace]`](workspaces.zh.md) — 工作区定义

<a id="package-metadata"></a>

### The `[package]` section

> `[package]`部分

`Cargo.toml`的第一部分是`[package]`.

```toml
[package]
name = "hello_world" # 名字
version = "0.1.0"    # 当前版本，遵循 semver
authors = ["Alice <a@example.com>", "Bob <b@example.com>"]
```

[`name`](#the-name-field) 和 [`version`](#the-version-field)是必须的。若要推送到 registry，可能会要求其他的字段。[crates.io] 的要求字段，可以看看[推送
章节][publishing]。

#### The `name` field

指代包的名称。用途是：作为其他包的依赖项；推断 lib 和 bin 的默认名称。

格式要符合 [alphanumeric] 字符 或是 `-`，或是 `_`，不能为空。
注意 [`cargo new`] 和 [`cargo init`] 加了一些额外限制，例如，强制它是有效的 Rust 标识符，不是关键字。[crates.io] 施加了更多的限制，例如
强制仅为 ASCII 字符，而不是保留名称，而不是特殊的 Windows
名称如"nul"，不要太长等。

[alphanumeric]: ../../std/primitive.char.html#method.is_alphanumeric

#### The `version` field

Cargo 烘烤的概念是[语义版本控制](http://semver.org/)，所以确保你遵循一些基本规则:

- 在您达到 1.0.0 之前，任何事情都会发生，但是如果您进行了重大变化的更新，则增加次要(minor)版本。在 Rust 语言中，重大变化包括，向结构添加字段，或增加变量到枚举。
- 在 1.0.0 之后，只在增加主要(major)版本时进行重大变化。不要破坏建筑.
- 在 1.0.0 之后，不要在补丁级别(patch)的版本添加任何新的公共 API(没有任何新的`pub`)。如果添加`pub`结构、特性、字段、类型、函数、方法或其他任何东东，则总是增加次要版本。
- 使用具有三个数字部分的版本号，如 1.0.0，而不是 1.0。

在 [Resolver] 章节，有关于依赖项版本解析的信息，还能设成自己的版本。而
[SemVer 兼容性][semver compatibility] 有破坏性变化的定义。

[resolver]: resolver.zh.md
[semver compatibility]: semver.zh.md

<a id="the-authors-field-optional"></a>

#### The `authors` field

可选的`authors` 字段，给出作者们的名称。有时，还能上邮箱。作者名跟着邮箱，邮箱写在`<>`里面就好。

该字段仅作为包的元信息，和 `build.rs` 所使用的 `CARGO_PKG_AUTHORS`
环境变量。不会再 [crates.io] 显示。

> **警告**: 一旦，包推送上去，这个字段就不能更改了。

<a id="the-edition-field-optional"></a>


#### The `edition` field

> `edition` 字段 (可选)

您可以在`Cargo.toml`中的`edition`字段，选择一个特定的 Rust 版本，用于您的包。
这个字段影响到[Rust Edition]，涉及编译的 Rust 版本。设置`[package]`下的`edition`字段将影响包中的所有目标/箱，包括测试套件、基准、二进制文件、示例等。

```toml
[package]
# ...
edition = '2021'
```

若是通过`cargo new`得来的项目，Cargo 将始终让`edition`字段设置为最新版本。目前是 2021 版本。

如果没有该字段，为了兼容性，`2015`会为默认值。

#### The `rust-version` field

> `rust-version` 字段 (可选)

让 Cargo 使用的 Rust 的构建版本。具有限制 Rust 版本的作用，如果使用的 Rust 版本号低于设定值，向用户抛出错误：什么版本是要求的。

该字段支持的最低版本是 1.56.0。
低于该版本，该字段会被忽略，并打印警告。

```toml
[package]
# ...
rust-version = "1.56"
```

版本的格式由两到三个部分组成; 不能使用 semver 的关键字符，或预发布的标记。比如 `-nightly` 就会忽略
The `rust-version` 必须大于或等于配置的 `edition`

使用 `--ignore-rust-version` 选项忽略`rust-version`。

设置`[package]`下的`rust-version`字段将影响包中的所有目标/箱，包括测试套件、基准、二进制文件、示例等。


#### The `description` field

关于包的简短介绍。[crates.io] 会展示这段文字。纯文本 (不是 Markdown).

```toml
[package]
# ...
description = "A short description of my package"
```

> **Note**: [crates.io] 要求 `description` 的设置。

<a id="the-documentation-field-optional"></a>

#### The `documentation` field

The `documentation` 字段指定箱子文档的网址。若不设置， [crates.io] 会自动链接到 [docs.rs] 的对应页面。

```toml
[package]
# ...
documentation = "https://docs.rs/bitflags"
```

#### The `readme` field

指向包根目录下的 readme 文件 (与 `Cargo.toml` 相对的)。
箱子推送时，会传输这个文件到 [crates.io]，将渲染此文件，并将结果放在包的页面上。

```toml
[package]
# ...
readme = "README.md"
```

如果没有设置，根目录下，若存在`README.md`,
`README.txt` 或 `README`，就会自动使用。设为
`false`，就会限制这个行为，而设为 `true`, 默认推测为 `README.md`。

#### The `homepage` field

`homepage` 主页网址

```toml
[package]
# ...
homepage = "https://serde.rs/"
```

#### The `repository` field

`repository` 源代码的网址。

```toml
[package]
# ...
repository = "https://github.com/rust-lang/cargo/"
```

#### The `license` and `license-file` fields

`license` 指定软件协议。The `license-file` 则是协议的路径 (相对于 `Cargo.toml`)。

[crates.io] 会将 `license` 字段，解释为一个 [SPDX 2.1 license
expression][spdx-2.1-license-expressions]。名称必须是 [SPDX license list 3.11][spdx-license-list-3.11] 的协议。现在还不支持括号。在 [SPDX site] 上，会有更多信息。

SPDX license expressions 支持 AND 和 OR 运算符，组合多个协议 [^slash]。

```toml
[package]
# ...
license = "MIT OR Apache-2.0"
```

`OR` 是选择任一个协议。Using `AND` 是选择两个协议。`WITH`则是指定 exception 下的协议。一些示例：

- `MIT OR Apache-2.0`
- `LGPL-2.1-only AND MIT AND BSD-2-Clause`
- `GPL-2.0-or-later WITH Bison-exception-2.2`

如果使用的是非标准协议，`license-file` 会是`license` 的替代。

```toml
[package]
# ...
license-file = "LICENSE.txt"
```

> **Note**: [crates.io] 需要 `license` 或 `license-file` 的设置。

[^slash]:
    Previously multiple licenses could be separated with a `/`, 弃用。

#### The `keywords` field

这是一个，描述此箱的关键字的列表。
补充 crates.io 上的搜索引擎, 帮助别人找到这个箱子。

```toml
[package]
# ...
keywords = ["gamedev", "graphics"]
```

> **Note**: [crates.io] 最多五个关键字。每个关键字必须是
> ASCII 文本，以 letter 开头，仅能包含 letters, numbers, `_` or
> `-`，最多 20个字符。

#### The `categories` field

这是此箱子最适合的类别列表。

```toml
categories = ["command-line-utilities", "development-tools::cargo-plugins"]
```

> **Note**: [crates.io] 最多 5 个。每个字符串都完全匹配 <https://crates.io/category_slugs>。

<a id="the-workspace--field-optional"></a>

#### The `workspace` field

指定所属的工作区，如果没有指定，则定为上层文件系统的第一个带`[workspace]` 的 Cargo.toml。用作非垂直目录的工作区结构。

```toml
[package]
# ...
workspace = "path/to/workspace/root"
```

如果有定义顶层的`[workspace]`表格，该字段是不能用的。因为，一个包不能既是工作区 (contain `[workspace]`) 又是另一个工作区的成员
(contain `package.workspace`)。

[workspaces 章节](workspaces.md) 有更多信息。

<a id="package-build"></a>
<a id="the-build-field-optional"></a>


#### The `build` field

> `build` 字段 (可选)

此字段指定包根目录中的文件，该文件是[构建脚本][build script]，用于生成原生代码。可以在构建脚本[指导][build script]找到更多信息。

[build script]: ./build-scripts.zh.md

```toml
[package]
# ...
build = "build.rs"
```

默认是 `"build.rs"`，位置是根目录。使用 `build = "custom_build_name.rs"`指定一个不同的文件路径，或 `build = false` 禁用构建脚本的自动检测。

#### The `links` field

> `links` 字段 (可选)

此字段指定，要链接到的原生库名，更多信息可以看，构建脚本指南的[`links`][links]部分。

[links]: ./build-scripts.zh.md#the-links-manifest-key

```toml
[package]
# ...
links = "foo"
build = "build.rs"
```

#### The `exclude` and `include` fields

> `exclude` 和 `include` 字段 (可选)

出于打包和重建包的目的，您可以显式地指定一组[globs][globs]模式，匹配项应被排除或包含。如`exclude`字段标识了在发布包时，不包括的一组文件，以及检测何时重建包时，应该忽略的文件，而`include`就是显式指定一定包含的文件。

运行[`cargo package --list`][`cargo package`] 查看哪个文件会被包括。

```toml
[package]
# ...
exclude = ["/ci", "images/", ".*"]
```

```toml
[package]
# ...
include = ["/src", "COPYRIGHT", "/examples", "!/examples/big_example"]
```

两个字段都不设置，默认是包括根目录下的所有文件，除了下面的排除项。

如果`include`是没有指定，那么下面的文件会被排除：

- 如果非 git 项目，所有以 `.`开头的隐藏文件，会被跳过。
- 如果是 git 项目，会参考项目中 [gitignore] 文件以及全局 git 配置。

不管是`exclude` 或 `include`指定，下面的文件总会排除：

- 任意 子-packages 会被跳过 (任意包含`Cargo.toml`的子目录)。
- 根目录下的`target`。

下面则为总是包括的文件:

- The `Cargo.toml`，不需要 `include` 指定。
- 一个最简化的 `Cargo.lock` 自动包括，若是项目包含一个
  binary 或 example target。[`cargo package`] 有更多信息。
- [`license-file`](#the-license-and-license-file-fields) 指定的文件。

下面的列表为人工记录; `include` 会覆盖
`exclude`。如果 `include` 的文件中，要特别跳过某些文件, 使用
`!` 运算符。

编写模式为 [gitignore]的模式。简述:

- `foo` ：具有`foo`名称文本的文件或目录。不管它在项目的哪个位置。等同于 `**/foo`。
- `/foo` 具有`foo`名称文本的文件或目录。位于根目录。
- `foo/` 具有`foo`名称文本的目录。不管它在项目的哪个位置。
- 常见的 glob 模式，比如：`*`, `?`, 和 `[]` 是支持的：
  - `*` 匹配零或多个字符，除了 `/`。例如，`*.html`
    匹配 `.html` 后缀名文件，位置不管。
  - `?` 匹配任意字符 除了 `/`。例如，`foo?` 匹配 `food`,
    不匹配 `foo`。
  - `[]` 匹配范围内。例如，`[ab]`
    匹配`a` 或 `b`。`[a-z]` 匹配从 a 到 z。
- `**/` 匹配任意位置。例如，`**/foo/bar` 匹配，任意位置下，`foo`目录的`bar`
- `/**` 匹配内部的一切。例如，`foo/**` 匹配`foo`下的所有文件，包括子目录。
- `/**/` 匹配零或多个目录。例如，例如，`a/**/b` 匹配
  `a/b`, `a/x/b`, `a/x/y/b`等等。
- `!` 是否决，例如，`src/*.rs` 加上
  `!foo.rs` ，会匹配 `src`目录下，所有以 `.rs` 结尾的文件，但会排除 `foo.rs`。

include/exclude 还能用作 更改 的跟踪。
如：构建，带有`rustdoc`的目标, 跟踪的文件关系到项目的重新构建。如果有
[build script]，它并没有指明任意的 `rerun-if-*` 命令，那么
include/exclude 的文件会关系到 build script 的重新执行。

[gitignore]: https://git-scm.com/docs/gitignore

选项是相互排斥的: `include`设置覆盖`exclude`。 注意`include`必须是文件的详尽列表，否则可能不包括必要的源文件。

[globs]: https://docs.rs/glob/0.2.11/glob/struct.Pattern.md

#### The `publish` field

> `publish` 字段 (可选)

这个`publish`字段通过错误，防止将包(crate)，发布到包注册中心(如*crates.io*)。

将包留在本地：

```toml
[package]
# ...
publish = false
```


还可以，要发布的网址名数组：

```toml
[package]
# ...
publish = ["some-registry-name"]
```

如果，仅有一个数组元素。`cargo publish` 在`--registry`没有指定的情况下，会使用它。

<a id="the-metadata-table-optional"></a>

#### The `metadata` table

> `metadata` 表格 (可选)

默认情况下，Cargo 将对`Cargo.toml`不使用的字段发出警告，协助检测错别字等。但是，这个`package.metadata`表格，完全不写了的话， Cargo 也不会警告。这个表格让其他工具，把它们的配置放在`Cargo.toml`。 例如:

```toml
[package]
name = "..."
# ...

# 当要生成一个 Android APK，这个元信息会被使用, 例如.
[package.metadata.android]
package-name = "my-awesome-android-app"
assets = "path/to/static"
```

[`workspace.metadata`][workspace-metadata]类似。当没有特定格式，获取到的数据是连续（连贯）顺序的。例如，`package.metadata`没有的数据，可以看看 `workspace.metadata` 有没有。
是否就合理很多。

[workspace-metadata]: workspaces.zh.md#the-workspacemetadata-table


#### The `default-run` field

[`cargo run`] 要指定的默认程序。当同时存在`src/bin/a.rs` 和 `src/bin/b.rs`:

```toml
[package]
default-run = "a"
```

### The `[badges]` section

`[badges]` 展示项目状态的徽标：

> 注意: [crates.io] 之前是放项目的旁边，但可以改为放在 readme.md (查看 [
> `readme` 字段](#the-readme-field)).

```toml
[badges]
# The `maintenance` table 表示项目的维护情况。crates.io 目前还没有用到。瞅瞅 https://github.com/rust-lang/crates.io/issues/2437
# 和 https://github.com/rust-lang/crates.io/issues/2438 。
#
# `status` 字段是必须的 可用选项是:
# - `actively-developed`: 有人迭代更新
# - `passively-maintained`: 有人维护
# - `as-is`: 缺人维护，项目功能完成
# - `experimental`: 作者实验项目
# - `looking-for-maintainer`: 找人维护
# - `deprecated`: 弃用
# - `none`: 无
maintenance = { status = "..." }
```

### Dependency sections

> 依赖 部分

见[指定依赖-那页](./specifying-dependencies.zh.md)有关`[dependencies]`，`[dev-dependencies]`，`[build-dependencies]`和特定目标的`[target.*.dependencies]`部分的信息。

### The `[profile.*]` sections


The `[profile]` 提供修改编译器配置的能力，如优化，调试设置。查看 [the Profiles chapter](profiles.zh.md)。

[`cargo init`]: ../commands/cargo-init.zh.md
[`cargo new`]: ../commands/cargo-new.zh.md
[`cargo package`]: ../commands/cargo-package.zh.md
[`cargo run`]: ../commands/cargo-run.zh.md
[crates.io]: https://crates.io/
[docs.rs]: https://docs.rs/
[publishing]: publishing.zh.md
[rust edition]: ../../edition-guide/index.html
[spdx-2.1-license-expressions]: https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60
[spdx-license-list-3.11]: https://github.com/spdx/license-list-data/tree/v3.11
[spdx site]: https://spdx.org/license-list
[toml]: https://toml.io/

<script>
(function() {
    var fragments = {
        "#the-project-layout": "../guide/project-layout.html",
        "#examples": "cargo-targets.html#examples",
        "#tests": "cargo-targets.html#tests",
        "#integration-tests": "cargo-targets.html#integration-tests",
        "#configuring-a-target": "cargo-targets.html#configuring-a-target",
        "#target-auto-discovery": "cargo-targets.html#target-auto-discovery",
        "#the-required-features-field-optional": "cargo-targets.html#the-required-features-field",
        "#building-dynamic-or-static-libraries": "cargo-targets.html#the-crate-type-field",
        "#the-workspace-section": "workspaces.html#the-workspace-section",
        "#virtual-manifest": "workspaces.html",
        "#package-selection": "workspaces.html#package-selection",
        "#the-features-section": "features.html#the-features-section",
        "#rules": "features.html",
        "#usage-in-end-products": "features.html",
        "#usage-in-packages": "features.html",
        "#the-patch-section": "overriding-dependencies.html#the-patch-section",
        "#using-patch-with-multiple-versions": "overriding-dependencies.html#using-patch-with-multiple-versions",
        "#the-replace-section": "overriding-dependencies.html#the-replace-section",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>