## The Manifest Format

这个`Cargo.toml`每个包的文件称为*显示*. 每个清单文件由一个或多个部分组成.

### The `[package]` section

A中的第一部分`Cargo.toml`是`[package]`.

```toml
[package]
name = "hello_world" # the name of the package
version = "0.1.0"    # the current version, obeying semver
authors = ["Alice <a@example.com>", "Bob <b@example.com>"]
```

所有这三个字段都是强制性的.

#### The `version` field

Cargo烘烤的概念[语义版本控制](http://semver.org/)所以一定要遵循一些基本规则:

-   在您达到1.0.0之前,任何事情都会发生,但是如果您进行了中断更改,则增加次要版本.在鲁斯特,突破性改变包括向域添加结构或变量到枚举.
-   在1.0.0之后,只在增加主版本时进行断点更改.不要破坏建筑.
-   在1.0.0之后,不要添加任何新的公共API(没有新的)`pub`任何)补丁级别的版本.如果添加新的版本,则总是增加次要版本.`pub`结构、特性、字段、类型、函数、方法或其他任何东西.
-   使用具有三个数字部分的版本号,如1.0.0,而不是1.

#### The `edition` field (optional)

您可以选择一个特定的锈版用于您的包与`edition`键入`Cargo.toml`.  如果没有指定版本,它将默认为2015.

```toml
[package]
# ...
edition = '2018'
```

这个`edition`关键影响您的包编译的版本.Cargo将始终通过`cargo new`与`edition`密钥设置为最新版本.设置`edition`键入`[package]`将影响包中的所有目标/板条,包括测试套件、基准、二进制文件、示例等.

#### The `build` field (optional)

此字段指定包根中的文件,该文件是[构建脚本][1]用于生成本机代码.可以在生成脚本中找到更多信息.[指导][1].

[1]: reference/build-scripts.md

```toml
[package]
# ...
build = "build.rs"
```

#### The `links` field (optional)

此字段指定链接到的本机库的名称.更多信息可以在[`links`][links]生成脚本指南的一部分.

[links]: reference/build-scripts.md#the-links-manifest-key

```toml
[package]
# ...
links = "foo"
build = "build.rs"
```

#### The `documentation` field (optional)

此字段指定托管机箱文档的网站的URL.如果清单文件中没有指定URL,[克拉西奥][cratesio]自动将你的箱子连接到相应的箱子[博士学位][docsrs]页.

来自特定主机的文档链接被列入黑名单.如果已知主机不承载文档并且可能具有恶意意图,例如广告跟踪网络,则主机被添加到黑名单中.下列主机的URL被列入黑名单:

-   rust-ci.org

来自黑名单主机的文档URL将不会出现在.es.io上,并且可能被docs.rs链接替换.

[docsrs]: https://docs.rs/

[cratesio]: https://crates.io/

#### The `exclude` and `include` fields (optional)

您可以显式地指定一组[地球仪][globs]出于打包和重建包的目的,应忽略或包含.中指定的球`exclude`字段标识在发布包时不包括的一组文件,以及出于检测何时重建包的目的而忽略的文件,以及`include`指定显式包含的文件.

如果一个VCS被用于一个包,则`exclude`字段将被植入VCS的忽略设置(`.gitignore`例如Git.

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

选项是相互排斥的:设置`include`将覆盖一个`exclude`. 注意`include`必须是文件的详尽列表,否则可能不包括必要的源文件.

[globs]: https://docs.rs/glob/0.2.11/glob/struct.Pattern.md

#### Migrating to `gitignore`-like pattern matching

这些配置的当前解释基于UNIX Globs,如[`glob`机箱](https://crates.io/crates/glob). 我们想要Cargo`include`和`exclude`配置为类似于`gitignore`尽可能.[这个`gitignore`规范](https://git-scm.com/docs/gitignore)也是基于Globs的,但是还有许多其他的特性,这些特性使模式编写更容易,控制也更多.因此,我们正在迁移这些配置规则的解释,以使用[`ignore`机箱](https://crates.io/crates/ignore)并将它们作为一行中的每一条规则对待`gitignore`文件.见[跟踪问题](https://github.com/rust-lang/cargo/issues/4268)有关迁移的更多细节.

#### The `publish`  field (optional)

这个`publish`字段可用于防止将包发布到包注册中心(如*克拉西奥*错了.

```toml
[package]
# ...
publish = false
```

#### The `workspace`  field (optional)

这个`workspace`字段可用于配置此包将属于的工作区.如果没有指定,这将被推断为第一个CARGO.`[workspace]`文件系统中向上.

```toml
[package]
# ...
workspace = "path/to/workspace/root"
```

有关更多信息,请参见下面的工作区表的文档.

#### Package metadata

有许多可选的元数据字段也被接受.`[package]`章节:

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

这个[克拉西奥](https://crates.io)注册中心将呈现描述、显示许可证、链接到三个URL并根据关键字进行分类.这些键为注册表的用户提供有用的信息,并且还影响箱子的搜索排名.在出版的板条箱里省略所有东西是非常令人沮丧的.

SPDX 2.1许可证表达式被记录在案[在这里][spdx-2.1-license-expressions].  许可证列表的当前版本可用.[在这里][spdx-license-list],版本2.4是可用的[在这里][spdx-license-list-2.4].

#### The `metadata` table (optional)

默认情况下,Cargo将对未使用的密钥发出警告.`Cargo.toml`协助检测错别字等.这个`package.metadata`表,但是,完全忽略了Cargo,将不会被警告.本节可用于希望将包配置存储在`Cargo.toml`. 例如:

```toml
[package]
name = "..."
# ...

# Metadata used when generating an Android APK, for example.
[package.metadata.android]
package-name = "my-awesome-android-app"
assets = "path/to/static"
```

### Dependency sections

见[指定依赖页](reference/specifying-dependencies.md)有关的信息`[dependencies]`,`[dev-dependencies]`,`[build-dependencies]`和目标特定的`[target.*.dependencies]`部分.

### The `[profile.*]` sections

Cargo支持通过顶层配置文件调用rustc的自定义配置.任何清单都可以声明概要文件,但是实际上只读取顶级包的概要文件.所有依赖项的配置文件将被重写.这样做是为了让顶级包能够控制如何编译其依赖项.

目前有四个受支持的概要文件名称,它们都具有相同的配置.下面列出了可用的配置,以及每个配置文件的默认设置.

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

Cargo支持特征允许表达:

-   条件编译选项(可通过)`cfg`属性;
-   可选的依赖项,增强了包,但不是必需的;
-   可选依赖项的簇,如`postgres`这将包括`postgres`包`postgres-macros`包,以及可能的其他包(如开发时模拟库、调试工具等).

包的特性可以是可选的依赖项,也可以是一组其他特性.指定特征的格式是:

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

使用包装`awesome`:

```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # do not include the default features, and optionally
                         # cherry-pick individual features
features = ["secure-password", "civet"]
```

#### Rules

特征的使用遵循一些规则:

-   特征名称不能与清单中的其他包名称冲突.这是因为他们被选择进入`features = [...]`,它只有一个命名空间.
-   除此之外`default`特性,所有的特征都是可选的.若要退出默认功能,请使用`default-features = false`樱桃选择个人特征.
-   特征组不允许周期性地相互依赖.
-   DEV依赖项不能是可选的.
-   特征组只能引用可选的依赖项.
-   当选择一个特征时,Cargo将调用`rustc`具有`--cfg
    feature="${feature_name}"`. 如果包含一个特性组,那么它将包括所有单独的特性.这可以通过代码进行测试.`#[cfg(feature = "foo")]`.

注意,显式地允许特性不实际激活任何可选的依赖项.这允许包在不需要新的依赖项的情况下在内部启用/禁用特性.

#### Usage in end products

该特性的一个主要用例是在最终产品中指定可选特性.例如,Servo包可能希望包含可选特性,人们可以在构建时启用或禁用它.

在这种情况下,伺服将描述其特征.`Cargo.toml`它们可以使用命令行标志来启用:

```console
$ cargo build --release --features "shumway pdf"
```

可以使用默认特性`--no-default-features`.

#### Usage in packages

在大多数情况下,概念*可选依赖*在库中,最好将其表示为顶级应用程序所依赖的单独包.

然而,像Iron或Piston这样的高级软件包可能需要能够策划多个软件包以便于安装.当前的Cargo系统允许它们将一些强制依赖关系整合到一个包中,以便于安装.

在某些情况下,包可能希望为可选依赖项提供额外的管理:

-   将多个低级可选依赖项分组到一个单独的高级特征中;
-   指定推荐(或建议)由包用户包括的包;
-   包括特征(类)`secure-password`在激励示例中,这只在可选的依赖项可用时才能工作,并且很难实现为单独的包(例如,设计一个与OpenSSL完全解耦的IO包可能过于困难,通过包含单独的包来选择进入).年龄).

在几乎所有情况下,在设计用于治疗的高级包之外使用这些特性都是反模式.如果某个特性是可选的,那么它几乎可以肯定地表示为单独的包.

### The `[workspace]` section

包可以定义一个工作区,它是一组板条箱,所有板条箱将共享相同的内容`Cargo.lock`和输出目录.这个`[workspace]`表可以定义为:

```toml
[workspace]

# Optional key, inferred from path dependencies if not present.
# Additional non-path dependencies that should be included must be given here.
# In particular, for a virtual manifest, all members have to be listed.
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]

# Optional key, empty if not present.
exclude = ["path1", "path/to/dir2"]
```

工作空间作为Cargo的一部分被添加到Cargo中.[RFC 1525]并具有许多属性:

-   工作区可以包含多个板条箱,其中一个是*根板条箱*.
-   这个*根板条箱*的`Cargo.toml`包含`[workspace]`表,但不需要有其他配置.
-   每当编译工作空间中的任何箱时,输出被放置在*工作空间根*. 即紧邻*根板条箱*的`Cargo.toml`.
-   工作空间中所有板条箱的锁定文件驻留在*工作空间根*.
-   这个`[patch]`,`[replace]`和`[profile.*]`章节在`Cargo.toml`只有在*根板条箱*在成员箱的清单中是明显的,被忽略了.

[rfc 1525]: https://github.com/rust-lang/rfcs/blob/master/text/1525-cargo-workspace.md

这个*根板条箱*工作空间的,由存在的`[workspace]`在其清单中,负责定义整个工作区.所有`path`驻留在工作区目录中的依赖项成为成员.您可以通过将它们添加到工作区中来将附加包添加到工作区中.`members`关键.请注意,显式列出的工作区的成员还将在工作区中包含它们的路径依赖项.有时候,一个包可能有很多工作区成员,并且保持最新可能很麻烦.路径依赖也可以使用.[地球仪][globs]匹配多个路径.最后,`exclude`KEY可以用于将工作路径中的路径列入黑名单.如果根本不希望某些路径依赖项存在于工作区中,那么这非常有用.

这个`package.workspace`清单键(如上所述)用于成员机箱中,以指向工作区的根机箱.如果省略此键,则推断它是清单包含的第一个板条箱`[workspace]`文件系统中向上.

板条箱可以指定`package.workspace`或指定`[workspace]`. 也就是说,板条箱不能同时作为工作空间中的根箱(包含).`[workspace]`)也是另一个工作空间的成员箱(包含)`package.workspace`)

大多数时间工作区都不需要处理.`cargo new`和`cargo init`将自动处理工作区配置.

#### Virtual Manifest

在工作区清单中,如果`package`如果存在表,则工作区根板条箱将被视为普通包和工作区.如果`package`表不存在于工作区清单中,它被称为*虚拟清单*.

#### Package selection

在工作区中,与包装相关的Cargo命令`cargo build`应用到选定的包`-p` / `--package`或`--all`命令行参数.当未指定时,可选的`default-members`使用配置:

```toml
[workspace]
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]
default-members = ["path/to/member2", "path/to/member3/foo"]
```

指定时,`default-members`必须扩展到子集`members`.

什么时候?`default-members`未指定,如果它是包,则默认为根清单,或者每个成员清单(好像`--all`在命令行上指定了虚拟工作区.

### The project layout

如果包是可执行文件,则将主源文件命名为`src/main.rs`. 如果它是一个库,请命名主源文件`src/lib.rs`.

Cargo也将处理任何文件位于`src/bin/*.rs`作为可执行文件.如果可执行文件包含不止一个源文件,则可以在其中使用目录`src/bin`包含一个`main.rs`文件将被视为具有父目录名称的可执行文件.但是,一旦添加了`[[bin]]`剖面图(剖面)[见下文](#configuring-a-target),Cargo将不再自动建立文件位于`src/bin/*.rs`.  相反,你必须创建一个`[[bin]]`要生成的每个文件的节.

您的包可以可选地包含命名文件夹.`examples`,`tests`和`benches`其中,Cargo将分别将其视为包含示例、集成测试和基准.类似于`bin`目标,它们可以由单个文件或目录组成.`main.rs`文件.

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

为了在为包创建文件和文件夹之后构造代码,应该记住使用Rust的模块系统,您可以在[书](https://doc.rust-lang.org/book/crates-and-modules.md).

### Examples

位于下方的文件`examples`是库提供的功能的示例使用.编译时,它们被放置在`target/examples`目录.

它们可以编译为可执行文件(用`main()`函数或库并通过使用来拉入库`extern crate <library-name>`. 当您运行测试以保护它们免遭篡改时,它们会被编译.

可以使用命令运行单个可执行示例.`cargo run --example
<example-name>`.

指定`crate-type`将示例编译为库(有关板条箱类型的附加信息可在[锈蚀参考](https://doc.rust-lang.org/reference/linkage.md)):

```toml
[[example]]
name = "foo"
crate-type = ["staticlib"]
```

可以使用命令构建单个库实例.`cargo build
--example <example-name>`.

### Tests

当你奔跑`cargo test`Cargo将:

-   编译并运行库的单元测试,这些测试位于`lib.rs`(当然,任何部分标记为`#[cfg(test)]`将在现阶段考虑);
-   编译并运行嵌入到文档块内部的库的文档测试;
-   编译并运行您的库[集成测试](#integration-tests)和
-   编译你的库的例子.

#### Integration tests

每个文件在`tests/*.rs`是一个集成测试.当你奔跑`cargo test`Cargo将作为一个单独的箱子来编译这些文件中的每一个.板条箱可以通过使用链接链接到您的库.`extern crate <library-name>`就像其他依赖它的代码一样.

Cargo不会自动编译子目录内的文件`tests`但是,集成测试可以像往常一样从这些目录导入模块.例如,如果希望多个集成测试共享一些代码,可以将共享代码放入`tests/common/mod.rs`然后放`mod common;`在每个测试文件中.

### Configuring a target

所有的`[[bin]]`,`[lib]`,`[[bench]]`,`[[test]]`和`[[example]]`部分支持类似的配置,用于指定应该如何构建目标.双括号截面`[[bin]]`数组的表[托姆尔](https://github.com/toml-lang/toml#array-of-tables)这意味着你可以写多于一个`[[bin]]`在您的箱中生成几个可执行文件.

下面的例子使用`[lib]`,但它也适用于所有其他部分.除非另有说明,所有列出的值都是该选项的默认值.

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

这个`[package]`还包括可选的`autobins`,`autoexamples`,`autotests`和`autobenches`明确选择或选择退出自动发现特定的目标种类.

#### The `required-features` field (optional)

这个`required-features`字段指定目标需要构建的特征.如果未选择任何所需的特征,则将跳过目标.这只与`[[bin]]`,`[[bench]]`,`[[test]]`和`[[example]]`部分,它没有影响`[lib]`.

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

如果您的包生成库,则可以通过显式地在`Cargo.toml`:

```toml
# ...

[lib]
name = "..."
crate-type = ["dylib"] # could be `staticlib` as well
```

可用的选项是`dylib`,`rlib`,`staticlib`,`cdylib`和`proc-macro`. 您应该只在包中使用此选项.Cargo总是根据包含它们的包的要求来编译包(依赖项).

您可以阅读更多关于不同的板条类型[锈蚀参考手册](https://doc.rust-lang.org/reference/linkage.md)

### The `[patch]` Section

这段车厢可以用[重写依赖项][replace]与其他副本.语法类似于`[dependencies]`章节:

```toml
[patch.crates-io]
foo = { git = 'https://github.com/example/foo' }
bar = { path = 'my/local/bar' }

[dependencies.baz]
git = 'https://github.com/example/baz'

[patch.'https://github.com/example/baz']
baz = { git = 'https://github.com/example/patched-baz', branch = 'my-branch' }
```

这个`[patch]`表由依赖性的子表组成.每个键后`[patch]`是正在修补的源的URL,或者`crates-io`如果你正在修改[HTTPS://CRATESIO](https://crates.io)注册表.在上面的例子中`crates-io`可以用Git URL替换,例如`https://github.com/rust-lang-nursery/log`第二`[patch]`示例中的部分使用此来指定调用`baz`.

这些表中的每个条目都是一个正常的依赖关系规范,与`[dependencies]`清单的部分.中列出的依赖项`[patch]`部分被解析并用于在指定的URL上对源进行补丁.上面的显示片段补丁`crates-io`源(例如CRATESIO本身)与`foo`板条箱`bar`机箱.它也修补了`https://github.com/example/baz`源与A`my-branch`这来自其他地方.

可以用不存在的板条箱版本来修补源,也可以用已经存在的板条箱版本来修补源.如果用源中已经存在的板条箱版本对源进行修补,则替换源的原始板条箱.

有关重写依赖关系的更多信息可以在[重写依赖项][replace]文件和部分[RFC 1969]对于这一特性的技术规范.

[rfc 1969]: https://github.com/rust-lang/rfcs/pull/1969

[replace]: reference/specifying-dependencies.md#overriding-dependencies

### The `[replace]` Section

这段车厢可以用[重写依赖项][replace]与其他副本.语法类似于`[dependencies]`章节:

```toml
[replace]
"foo:0.1.0" = { git = 'https://github.com/example/foo' }
"bar:1.0.2" = { path = 'my/local/bar' }
```

中的每个键`[replace]`表是[包标识规范](reference/pkgid-spec.md)它允许任意选择依赖图中的节点来重写.每个密钥的值与`[dependencies]`指定依赖关系的语法,除了不能指定特性外.注意,当覆盖一个板条箱时,它被覆盖的副本必须具有相同的名称和版本,但是它可以来自不同的源(例如,g it或本地路径).

有关重写依赖关系的更多信息可以在[重写依赖项][replace]部分文件.

[spdx-2.1-license-expressions]: https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60

[spdx-license-list]: https://spdx.org/licenses/

[spdx-license-list-2.4]: https://github.com/spdx/license-list-data/tree/v2.4
