## The Manifest Format

> 清单格式

每个包的这个`Cargo.toml`文件称为*清单*. 每个清单文件由一个或多个部分(表格)组成.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [`[package]` 部分](#the-package-section)
- [依赖项 部分](#dependency-sections)
- [`[profile.*]` 部分](#the-profile-sections)
- [`[features]` 部分](#the-features-section)
- [`[workspace]` 部分](#the-workspace-section)
- [项目布局](#the-project-layout)
- [Rust 示例](#examples)
- [Rust 测试](#tests)
- [配置一个 target](#configuring-a-target)
- [`[patch]` 部分](#the-patch-section)
- [`[replace]` 部分](#the-replace-section)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

### The `[package]` section

> `[package]`部分

`Cargo.toml`的第一部分是`[package]`.

```toml
[package]
name = "hello_world" # the name of the package
version = "0.1.0"    # the current version, obeying semver
authors = ["Alice <a@example.com>", "Bob <b@example.com>"]
```

所有这三个字段都是必要性的.

#### The `version` field

> `version` 字段

Cargo 烘烤的概念是[语义版本控制](http://semver.org/)，所以确保你遵循一些基本规则:

- 在您达到 1.0.0 之前,任何事情都会发生,但是如果您进行了重大变化的更新,则增加次要(minor)版本。在 Rust 语言中,重大变化包括，向结构添加字段，或增加变量到枚举。
- 在 1.0.0 之后,只在增加主要(major)版本时进行重大变化。不要破坏建筑.
- 在 1.0.0 之后,不要在补丁级别(patch)的版本添加任何新的公共 API(没有任何新的`pub`)。如果添加`pub`结构、特性、字段、类型、函数、方法或其他任何东东，则总是增加次要版本。
- 使用具有三个数字部分的版本号,如 1.0.0，而不是 1.0。

#### The `edition` field (optional)

> `edition` 字段 (可选)

您可以在`Cargo.toml`中的`edition`字段，选择一个特定的 Rust 版本，用于您的包。 如果没有指定版本,它将默认为 2015。

```toml
[package]
# ...
edition = '2018'
```

这个`edition`字段会影响到您的包编译的版本。若是通过`cargo new`得来的项目，Cargo 将始终让`edition`字段设置为最新版本。设置`[package]`下的`edition`字段将影响包中的所有目标/箱，包括测试套件、基准、二进制文件、示例等。

#### The `build` field (optional)

> `build` 字段 (可选)

此字段指定包根目录中的文件,该文件是[构建脚本][1]，用于生成本机代码。可以在构建脚本[指导][1]中找到更多信息..

[1]: ./build-scripts.zh.md

```toml
[package]
# ...
build = "build.rs"
```

#### The `links` field (optional)

> `links` 字段 (可选)

此字段指定，要链接到的本机库名，更多信息可以在构建脚本指南的[`links`][links]部分.

[links]: ./build-scripts.zh.md#the-links-manifest-key

```toml
[package]
# ...
links = "foo"
build = "build.rs"
```

#### The `documentation` field (optional)

> `documentation` 字段 (可选)

此字段指定托管箱(crate)文档的网站的 URL。如果清单文件中没有指定 URL，[crates.io][cratesio]自动将你的箱子连接到相应的箱子的[docs.rs][docsrs]页.

来自特定主机的文档链接被列入黑名单。如果已知主机不承载文档，并且可能具有恶意意图,例如广告跟踪网络，则主机被添加到黑名单中。下列主机的 URL 就被列入黑名单:

- rust-ci.org

来自黑名单主机的文档 URL 将不会出现在 crates.io 上，并且可能被 docs.rs 链接替换。

[docsrs]: https://docs.rs/
[cratesio]: https://crates.io/

#### The `exclude` and `include` fields (optional)

> `exclude` 和 `include` 字段 (可选)

出于打包和重建包的目的，您可以显式地指定一组[globs][globs]模式,匹配项应被忽略或包含。如`exclude`字段标识了在发布包时，不包括的一组文件，以及检测何时重建包时，应该忽略的文件，而`include`就是显式指定一定包含的文件。

如果一个 VCS 被用于一个包,则`exclude`字段将被植入 VCS 的忽略设置(例如 Git 的`.gitignore`)。

```toml
[package]
# ...
exclude = ["build/**/*.o", "doc/**/*.md"]
```

```toml
[package]
# ...
include = ["src/**/*", "Cargo.toml"]
```

选项是相互排斥的: `include`设置覆盖`exclude`。 注意`include`必须是文件的详尽列表,否则可能不包括必要的源文件。

[globs]: https://docs.rs/glob/0.2.11/glob/struct.Pattern.md

#### Migrating to `gitignore`-like pattern matching

> 转移成 类`gitignore` 模式匹配

这些配置的当前解释实现都基于 UNIX Globs，如[`glob`箱](https://crates.io/crates/glob)。 若是我们想要 Cargo 的`include`和`exclude`尽可能配置为类似于`gitignore`。可看看[这个`gitignore`规范](https://git-scm.com/docs/gitignore)，其也是基于 Globs 的，但是还有许多其他的特性,这些特性使模式编写更容易,控制也更多。因此,我们正在迁移这些配置规则的解释实现,以使用[`ignore`箱](https://crates.io/crates/ignore)，并认真对待`gitignore`文件的每一条行规则。见[跟踪问题](https://github.com/rust-lang/cargo/issues/4268)有关迁移的更多细节。

#### The `publish` field (optional)

> `publish` 字段 (可选)

这个`publish`字段通过错误，防止将包(crate)，发布到包注册中心(如*crates.io*)。

```toml
[package]
# ...
publish = false
```

#### The `workspace` field (optional)

> `workspace` 字段 (可选)

这个`workspace`字段可用于配置此包将属于的工作区。如果没有指定,这将被推断为文件系统中第一个 Cargo.toml 的`[workspace]`。

```toml
[package]
# ...
workspace = "path/to/workspace/root"
```

有关更多信息,请参见下面的工作区(workspace)表格的文档.

#### Package metadata

> 包 元信息

`[package]`部分会接受许多可选的元数据字段:

```toml
[package]
# ...

# A short blurb about the package. This is not rendered in any format when
# uploaded to crates.io (aka this is not markdown).
description = "..."

# These URLs point to more information about the package. These are
# intended to be webviews of the relevant data, not necessarily compatible
# with VCS tools and the like.
documentation = "..."
homepage = "..."
repository = "..."

# This points to a file under the package root (relative to this `Cargo.toml`).
# The contents of this file are stored and indexed in the registry.
# crates.io will render this file and place the result on the crate's page.
readme = "..."

# This is a list of up to five keywords that describe this crate. Keywords
# are searchable on crates.io, and you may choose any words that would
# help someone find this crate.
keywords = ["...", "..."]

# This is a list of up to five categories where this crate would fit.
# Categories are a fixed list available at crates.io/category_slugs, and
# they must match exactly.
categories = ["...", "..."]

# This is an SPDX 2.1 license expression for this package.  Currently
# crates.io will validate the license provided against a whitelist of
# known license and exception identifiers from the SPDX license list
# 2.4.  Parentheses are not currently supported.
#
# Multiple licenses can be separated with a `/`, although that usage
# is deprecated.  Instead, use a license expression with AND and OR
# operators to get more explicit semantics.
license = "..."

# If a package is using a nonstandard license, then this key may be specified in
# lieu of the above key and must point to a file relative to this manifest
# (similar to the readme key).
license-file = "..."

# Optional specification of badges to be displayed on crates.io.
#
# - The badges pertaining to build status that are currently available are
#   Appveyor, CircleCI, GitLab, and TravisCI.
# - Available badges pertaining to code test coverage are Codecov and
#   Coveralls.
# - There are also maintenance-related badges based on isitmaintained.com
#   which state the issue resolution time, percent of open issues, and future
#   maintenance intentions.
#
# If a `repository` key is required, this refers to a repository in
# `user/repo` format.
[badges]

# Appveyor: `repository` is required. `branch` is optional; default is `master`
# `service` is optional; valid values are `github` (default), `bitbucket`, and
# `gitlab`; `id` is optional; you can specify the appveyor project id if you
# want to use that instead. `project_name` is optional; use when the repository
# name differs from the appveyor project name.
appveyor = { repository = "...", branch = "master", service = "github" }

# Circle CI: `repository` is required. `branch` is optional; default is `master`
circle-ci = { repository = "...", branch = "master" }

# GitLab: `repository` is required. `branch` is optional; default is `master`
gitlab = { repository = "...", branch = "master" }

# Travis CI: `repository` in format "<user>/<project>" is required.
# `branch` is optional; default is `master`
travis-ci = { repository = "...", branch = "master" }

# Codecov: `repository` is required. `branch` is optional; default is `master`
# `service` is optional; valid values are `github` (default), `bitbucket`, and
# `gitlab`.
codecov = { repository = "...", branch = "master", service = "github" }

# Coveralls: `repository` is required. `branch` is optional; default is `master`
# `service` is optional; valid values are `github` (default) and `bitbucket`.
coveralls = { repository = "...", branch = "master", service = "github" }

# Is it maintained resolution time: `repository` is required.
is-it-maintained-issue-resolution = { repository = "..." }

# Is it maintained percentage of open issues: `repository` is required.
is-it-maintained-open-issues = { repository = "..." }

# Maintenance: `status` is required. Available options are `actively-developed`,
# `passively-maintained`, `as-is`, `experimental`, `looking-for-maintainer`,
# `deprecated`, and the default `none`, which displays no badge on crates.io.
maintenance = { status = "..." }
```

这个[crates.io](https://crates.io)注册中心将呈现描述、显示许可证、链接到三个 URL 并根据关键字进行分类。这些字段为注册表的用户提供有用的信息，并且还影响箱子的搜索排名。在发布箱的'展示栏'，省略任何东西都是非常令人沮丧的。

SPDX 2.1 许可证表达式被记录在案[在这里][spdx-2.1-license-expressions]。 许可证列表的当前版本可用的，[在这里][spdx-license-list]，版本 2.4 是可用的，[在这里][spdx-license-list-2.4].

#### The `metadata` table (optional)

> `metadata` 表格 (可选)

默认情况下,Cargo 将对`Cargo.toml`不使用的字段发出警告，协助检测错别字等。就像这个`package.metadata`表格，但是,完全不写了的话， Cargo 将不会被警告。这个表格可在`Cargo.toml`，用于将包配置存储好。 例如:

```toml
[package]
name = "..."
# ...

# 当要生成一个 Android APK，这个元信息会被使用, 例如.
[package.metadata.android]
package-name = "my-awesome-android-app"
assets = "path/to/static"
```

### Dependency sections

> 依赖 部分

见[指定依赖-那页](./specifying-dependencies.zh.md)有关`[dependencies]`,`[dev-dependencies]`,`[build-dependencies]`和特定目标的`[target.*.dependencies]`部分的信息。

### The `[profile.*]` sections

> `[profile.*]` 部分

Cargo 支持了，可通过顶层 配置文件(profile) 调用 rustc 的自定义配置。任何清单都可以声明一个配置文件，但是实际上只读取顶级包的配置文件。所有依赖项的配置文件都将被重写，这样做是为了让顶级包能够控制，其依赖项如何编译的。

目前有四个受支持的配置文件名称,它们都具有相同的配置。下面列出了可用的配置,以及每个配置文件的默认设置.

```toml
# The development profile, used for `cargo build`.
[profile.dev]
opt-level = 0      # controls the `--opt-level` the compiler builds with.
                   # 0-1 is good for debugging. 2 is well-optimized. Max is 3.
                   # 's' attempts to reduce size, 'z' reduces size even more.
debug = true       # (u32 or bool) Include debug information (debug symbols).
                   # Equivalent to `-C debuginfo=2` compiler flag.
rpath = false      # controls whether compiler should set loader paths.
                   # If true, passes `-C rpath` flag to the compiler.
lto = false        # Link Time Optimization usually reduces size of binaries
                   # and static libraries. Increases compilation time.
                   # If true, passes `-C lto` flag to the compiler, and if a
                   # string is specified like 'thin' then `-C lto=thin` will
                   # be passed.
debug-assertions = true # controls whether debug assertions are enabled
                   # (e.g. debug_assert!() and arithmetic overflow checks)
codegen-units = 16 # if > 1 enables parallel code generation which improves
                   # compile times, but prevents some optimizations.
                   # Passes `-C codegen-units`.
panic = 'unwind'   # panic strategy (`-C panic=...`), can also be 'abort'
incremental = true # whether or not incremental compilation is enabled
overflow-checks = true # use overflow checks for integer arithmetic.
                   # Passes the `-C overflow-checks=...` flag to the compiler.

# The release profile, used for `cargo build --release` (and the dependencies
# for `cargo test --release`,  including the local library or binary).
[profile.release]
opt-level = 3
debug = false
rpath = false
lto = false
debug-assertions = false
codegen-units = 16
panic = 'unwind'
incremental = false
overflow-checks = false

# The testing profile, used for `cargo test` (for `cargo test --release` see
# the `release` and `bench` profiles).
[profile.test]
opt-level = 0
debug = 2
rpath = false
lto = false
debug-assertions = true
codegen-units = 16
panic = 'unwind'
incremental = true
overflow-checks = true

# The benchmarking profile, used for `cargo bench` (and the test targets and
# unit tests for `cargo test --release`).
[profile.bench]
opt-level = 3
debug = false
rpath = false
lto = false
debug-assertions = false
codegen-units = 16
panic = 'unwind'
incremental = false
overflow-checks = false
```

### The `[features]` section

> `[features]` 部分

Cargo 支持特性，允许表达:

- 条件编译选项(通过`cfg`属性);
- 可选的依赖项，增强了包，但不是必需的;还有
- 可选依赖项的簇，如`postgres`，其中就包括`postgres`包`postgres-macros`包，以及可能的其他包(如开发时的模拟库、调试工具等)。

包的特性也可以是可选的依赖项,也可以是一组其他特性。指定特性的格式是:

```toml
[package]
name = "awesome"

[features]
# The default set of optional packages. Most people will want to use these
# packages, but they are strictly optional. Note that `session` is not a package
# but rather another feature listed in this manifest.
default = ["jquery", "uglifier", "session"]

# A feature with no dependencies is used mainly for conditional compilation,
# like `#[cfg(feature = "go-faster")]`.
go-faster = []

# The `secure-password` feature depends on the bcrypt package. This aliasing
# will allow people to talk about the feature in a higher-level way and allow
# this package to add more requirements to the feature in the future.
secure-password = ["bcrypt"]

# Features can be used to reexport features of other packages. The `session`
# feature of package `awesome` will ensure that the `session` feature of the
# package `cookie` is also enabled.
session = ["cookie/session"]

[dependencies]
# These packages are mandatory and form the core of this package’s distribution.
cookie = "1.2.0"
oauth = "1.1.0"
route-recognizer = "=2.1.0"

# A list of all of the optional dependencies, some of which are included in the
# above `features`. They can be opted into by apps.
jquery = { version = "1.0.2", optional = true }
uglifier = { version = "1.5.3", optional = true }
bcrypt = { version = "*", optional = true }
civet = { version = "*", optional = true }
```

使用`awesome`包:

```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # do not include the default features, and optionally
                         # cherry-pick individual features
features = ["secure-password", "civet"]
```

#### Rules

> 规则

特性的使用遵循一些规则:

- 特性名称不能与清单中的其他包名称冲突。这是因为他们被选择加入`features = [...]`，而它只有一个命名空间。
- 除此`default`特性之外，所有的特性都是可选的。若要退出默认功能，请使用`default-features = false`，任君选择个人特性.
- 特性群组不允许周期性地相互依赖.
- 开发 依赖项不能是可选的.
- 特性群组只能引用可选的依赖项.
- 当选择一个特性时，Cargo 将调用具有`--cfg feature="${feature_name}"`的`rustc`。如果包含一个特性群组，那么它将包括所有单独的特性。这可以通过`#[cfg(feature = "foo")]`在代码中进行测试..

主要注意的是，显露的特性，实际上不激活任何可选的依赖项。这就允许包在不需要新的依赖项的情况下，于内部启用/禁用特性。

#### Usage in end products

> 生产终点的用法

该特性的一个主要用例是在最终产品中，指定可选特性。例如,Servo 包可能希望包含可选特性,人们可以在构建时，启用或禁用它。

在这种情况下,Servo 将在`Cargo.toml`描述特性，且用命令行标志来启用这些特性:

```console
$ cargo build --release --features "shumway pdf"
```

可以使用`--no-default-features`，排除默认特性。

#### Usage in packages

> 包(库)的用法

在大多数情况下，在库中*可选依赖*的概念，最好将其表示为顶级应用程序所依赖的单独包。

然而,像 Iron 或 Piston 这样的高级软件包会需要排布多个软件包以便于安装。当前的 Cargo 系统允许它们将一些强制依赖项，整合到一个包中,以便于安装。

在某些情况下,包可能希望为可选依赖项，提供额外的管理:

- 将多个低层可选依赖项，组合到一个单独的高级特性中;
- 由包用户指定推荐(或建议)要包括的包;
- 包括特性(类似`secure-password`在激励示例中)，这只在可选的依赖项可用时才能工作，并且很难实现为单独的包(例如,设计一个与 OpenSSL 完全解耦的 IO 包可能过于困难，那这时，就可通过包含单独的包来选择相关特性)。

在几乎所有情况下，在设计牢固的高级包之外，使用这些特性都是反模式的。如果某个特性是可选的，那么它几乎可以肯定地表示为单独的包。

### The `[workspace]` section

> `[workspace]` 部分

包可以定义一个工作区,它是一组箱，所有箱将共享相同`Cargo.lock`和输出目录。这个`[workspace]`表格可以定义为:

```toml
[workspace]

# Optional key, inferred from path dependencies if not present.
# Additional non-path dependencies that should be included must be given here.
# In particular, for a virtual manifest, all members have to be listed.
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]

# Optional key, empty if not present.
exclude = ["path1", "path/to/dir2"]
```

工作区作为 Cargo 的[RFC 1525]一部分被添加到 Cargo 中，并具有许多属性:

- 工作区可以包含多个箱,其中一个是*根箱*.
- 这个*根箱*的`Cargo.toml`包含`[workspace]`表格，但不要求必有其他配置.
- 每当编译工作区中的任何箱时，输出被放置在*工作区根*。 即紧挨着*根箱*的`Cargo.toml`.
- 工作区中所有箱的那个锁定文件驻留在*工作区根*.
- 在`Cargo.toml`的`[patch]`,`[replace]`和`[profile.*]`部分，只认*根箱*的清单，而忽略成员箱的。

[rfc 1525]: https://github.com/rust-lang/rfcs/blob/master/text/1525-cargo-workspace.md

这个工作区的*根箱*，由其清单中存在的`[workspace]`指定，并负责定义整个工作区。所有驻留在工作区目录中的`path`依赖项都变成成员。您可以通过`members`字段将附加包添加到工作区中。请注意，显式列出的工作区成员，也在工作区中包含了它们的路径依赖项。有时候，一个包可能有很多工作区成员，并且都保持最新会很麻烦。

路径依赖也可以使用[globs][globs]匹配多个路径。
最后，`exclude`字段 可以用于将工作路径中的路径列入黑名单。如果根本不希望某些路径依赖项存在于工作区中，那么这非常有用.

这个`package.workspace`清单字段(如上所述)用于成员箱中，以指向工作区的根箱。如果省略此字段，则推断它是文件系统(向上的父目录)中，清单包含`[workspace]`的第一个箱。

箱可以指定`package.workspace`或指定`[workspace]`。 也就是说,箱不能同时作为工作区中的根箱(包含`[workspace]`)，和另一个工作区的成员箱(包含`package.workspace`)

大多数时间工作区都不需要处理。因`cargo new`和`cargo init`将自动处理工作区配置。

#### Virtual Manifest

> 虚拟清单

在工作区清单中,如果`package`表格存在,则工作区根箱将被视为普通包和工作区。如果`package`表格不存在工作区清单中,那它被称为*虚拟清单*。

#### Package selection

> Package 部分

在工作区中,与包相关的 Cargo 命令，如`cargo build`，会应用`-p` / `--package`或`--all`命令行参数选定的包。当未指定时,可选`default-members`配置被使用:

```toml
[workspace]
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]
default-members = ["path/to/member2", "path/to/member3/foo"]
```

`default-members`指定时，必会扩展到子集的`members`中.

若是`default-members`未指定，如果它是包,则默认为根清单，或者若是虚拟工作区，就为每个成员的清单(如同`--all`在命令行上).

### The project layout

> 项目布局

如果包是可执行文件，则将主源文件命名为`src/main.rs`。 如果它是一个库，请命名主源文件`src/lib.rs`。

Cargo 也将处理位于`src/bin/*.rs`任何文件作为可执行文件。如果可执行文件包含不止一个源文件，则可以使用`src/bin`目录下，又一个包含`main.rs`文件的目录，而该目录将被视为具有父目录名称的可执行文件。但是，一旦添加了`[[bin]]`部分[见下文](#configuring-a-target)，Cargo 将不再自动建立`src/bin/*.rs`文件。 相反,你必须创建一个`[[bin]]`部分，给出你想要生成的每个文件。

<!-- HERE -->

您的包可以(可选地)包含命名为`examples`,`tests`和`benches`文件夹,Cargo 将分别将其视为包含示例、集成测试和基准。类似于`bin`目标，它们可以由单个文件或拥有`main.rs`文件的目录组成。

```
▾ src/           # directory containing source files
  lib.rs         # the main entry point for libraries and packages
  main.rs        # the main entry point for packages producing executables
  ▾ bin/         # (optional) directory containing additional executables
    *.rs
  ▾ */           # (optional) directories containing multi-file executables
    main.rs
▾ examples/      # (optional) examples
  *.rs
  ▾ */           # (optional) directories containing multi-file examples
    main.rs
▾ tests/         # (optional) integration tests
  *.rs
  ▾ */           # (optional) directories containing multi-file tests
    main.rs
▾ benches/       # (optional) benchmarks
  *.rs
  ▾ */           # (optional) directories containing multi-file benchmarks
    main.rs
```

为了在创建文件和文件夹之后，为包构造代码，应该记住使用 Rust 的模块系统，您可以在这本[书](https://doc.rust-lang.org/book/crates-and-modules.md)找到。

> (译)：[中文](https://kaisery.github.io/trpl-zh-cn/ch07-00-packages-crates-and-modules.html)

### Examples

> 示例

位于`examples`下方的文件，是库提供的功能示例用法。编译时,它们被放置在`target/examples`目录。

它们可以编译为可执行文件(用`main()`函数)或，库。和可通过使用`extern crate <library-name>`导入库。 当您运行测试以保护它们免遭篡改时,它们会被编译。

可以使用命令`cargo run --example <example-name>`运行单个可执行示例.

指定`crate-type`将示例编译为库(有关箱类型的附加信息可在[Rust 参考](https://doc.rust-lang.org/reference/linkage.html)找到):

```toml
[[example]]
name = "foo"
crate-type = ["staticlib"]
```

可以使用命令`cargo build --example <example-name>`构建单个库实例.

### Tests

> 测试

当你运行`cargo test`，Cargo 会:

- 编译并运行库的单元测试，这些测试位于`lib.rs`(当然,任何标记为`#[cfg(test)]`部分将考虑为同个阶段);
- 编译并运行嵌入到文档区块内部的库的文档测试;
- 编译并运行您库的[集成测试](#integration-tests)和
- 编译你库的例子.

#### Integration tests

> 集成测试

在`tests/*.rs`的每个文件是一个集成测试。当你运行`cargo test`，Cargo 将编译每个文件作为一个单独的箱子。箱可以通过使用`extern crate <library-name>`链接(导入)您的库，就像其他导入项一样。

Cargo 不会自动编译`tests`子目录内的文件，但是，集成测试可以像往常一样从这些目录导入模块。例如,如果希望多个集成测试共享一些代码，可以将共享代码放入`tests/common/mod.rs`，然后为每个测试文件添加`mod common;`。

### Configuring a target

> 配置为一个目标

所有的`[[bin]]`,`[lib]`,`[[bench]]`,`[[test]]`和`[[example]]`部分都支持类似的配置，用于指定应该如何构建目标。双括号`[[bin]]`部分，是[TOML](https://github.com/toml-lang/toml#array-of-tables)格式的数组。这意味着你可以在您的箱中写多个`[[bin]]`，这样就会生成几个可执行文件。

下面的例子使用`[lib]`，但它也适用于所有其他部分。除非另有说明,下面所有列出的值都是对应选项的**默认值**。

```toml
[package]
# ...

[lib]
# The name of a target is the name of the library that will be generated. This
# is defaulted to the name of the package, with any dashes replaced
# with underscores. (Rust `extern crate` declarations reference this name;
# therefore the value must be a valid Rust identifier to be usable.)
name = "foo"

# This field points at where the crate is located, relative to the `Cargo.toml`.
path = "src/lib.rs"

# A flag for enabling unit tests for this target. This is used by `cargo test`.
test = true

# A flag for enabling documentation tests for this target. This is only relevant
# for libraries, it has no effect on other sections. This is used by
# `cargo test`.
doctest = true

# A flag for enabling benchmarks for this target. This is used by `cargo bench`.
bench = true

# A flag for enabling documentation of this target. This is used by `cargo doc`.
doc = true

# If the target is meant to be a compiler plugin, this field must be set to true
# for Cargo to correctly compile it and make it available for all dependencies.
plugin = false

# If the target is meant to be a "macros 1.1" procedural macro, this field must
# be set to true.
proc-macro = false

# If set to false, `cargo test` will omit the `--test` flag to rustc, which
# stops it from generating a test harness. This is useful when the binary being
# built manages the test runner itself.
harness = true

# If set then a target can be configured to use a different edition than the
# `[package]` is configured to use, perhaps only compiling a library with the
# 2018 edition or only compiling one unit test with the 2015 edition. By default
# all targets are compiled with the edition specified in `[package]`.
edition = '2015'
```

这个`[package]`还包括可选的`autobins`,`autoexamples`,`autotests`和`autobenches`，来明确 进入/退出 自动发现特定的目标种类。

#### The `required-features` field (optional)

> `required-features` 字段 (可选)

这个`required-features`字段指定目标需要构建的特性。如果未选择任何所需的特性,则将跳过目标。这只与`[[bin]]`,`[[bench]]`,`[[test]]`和`[[example]]`部分有影响，它没有影响`[lib]`。

```toml
[features]
# ...
postgres = []
sqlite = []
tools = []

[[bin]]
# ...
required-features = ["postgres", "tools"]
```

#### Building dynamic or static libraries

> 构建 动态 或 静态 库

如果您的包生成一个库,则可以通过在`Cargo.toml`显式地指明构建的库类型:

```toml
# ...

[lib]
name = "..."
crate-type = ["dylib"] # 也能是 `staticlib`
```

可用的选项是`dylib`,`rlib`,`staticlib`,`cdylib`和`proc-macro`。 您应该只在包中使用一次此选项。Cargo 总是根据(包括的)包的要求来编译包(依赖项)。

您可以阅读[Rust 参考手册](https://doc.rust-lang.org/reference/linkage.html)中更多关于不同的箱类型

### The `[patch]` Section

> `[patch]` 部分

这部分可以用来[重写其他副本的依赖项][replace]。语法类似于`[dependencies]`部分:

```toml
[patch.crates-io]
foo = { git = 'https://github.com/example/foo' }
bar = { path = 'my/local/bar' }

[dependencies.baz]
git = 'https://github.com/example/baz'

[patch.'https://github.com/example/baz']
baz = { git = 'https://github.com/example/patched-baz', branch = 'my-branch' }
```

这个`[patch]`表格由，类似依赖表格的子表组成。`[patch]`后的每个字段是正在修补的源 URL，或者`crates-io`(如果你正在修改[HTTPS://CRATESIO](https://crates.io)注册表)。在上面的例子中，`crates-io`可以用 Git URL 替换，例如`https://github.com/rust-lang-nursery/log`；第二个示例中的`[patch]`部分使用此来指定一个名为`baz`的源。

这些表格中的每个项都是一个正常的依赖关系规范，与`[dependencies]`清单的部分一样。`[patch]`部分中列出的依赖项，被解析并用于在指定的 URL 上对源进行补丁。上面的清单片段补丁`crates-io`源(例如 crates.io 本身)的`foo`箱和`bar`箱。它也用一个来自其他地方的`my-branch`分支修补了`https://github.com/example/baz`源。

可以用不存在的箱版本来修补源，也可以用已经存在的箱版本来修补源。如果用源中已经存在的箱版本对源进行修补，则会替换源的原始箱。

有关重写依赖关系的更多信息，可阅读本文档的[重写依赖项][replace]章节和对于这一特性的[RFC 1969]技术规范说明。

[rfc 1969]: https://github.com/rust-lang/rfcs/pull/1969
[replace]: ./specifying-dependencies.zh.md#overriding-dependencies

### The `[replace]` Section

> `[replace]` 部分

这部分可以用来[重写其他副本的依赖项][replace]。语法类似于`[dependencies]`部分:

```toml
[replace]
"foo:0.1.0" = { git = 'https://github.com/example/foo' }
"bar:1.0.2" = { path = 'my/local/bar' }
```

`[replace]`表格的每个字段都是[包标识规范](./pkgid-spec.zh.md)，它任意选择依赖图中的节点来重写。每个字段值与`[dependencies]指定依赖关系的语法是一样，除了不能指定特性。注意,当覆盖一个箱时,覆盖它的副本必须具有相同的名称和版本，但它可以来自不同的源(例如,git 或本地路径).

有关重写依赖关系的更多信息，可阅读本文档的[重写依赖项][replace]章节。

[spdx-2.1-license-expressions]: https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60
[spdx-license-list]: https://spdx.org/licenses/
[spdx-license-list-2.4]: https://github.com/spdx/license-list-data/tree/v2.4
