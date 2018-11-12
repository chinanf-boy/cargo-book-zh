## The Manifest Format

### The `[package]` section

A中的第一部分`Cargo.toml`是`[package]`.

```toml
[package]
name = "hello_world" # the name of the package
version = "0.1.0"    # the current version, obeying semver
authors = ["you@example.com"]
```

所有这三个字段都是强制性的.

#### The `version` field

货物烘烤的概念[Semantic
Versioning](http://semver.org/)所以一定要遵循一些基本规则:

-   在达到1.0.0之前,任何事情都会发生,但是如果你做了突破性的改变,增加次要版本.在鲁斯特,突破性改变包括向域添加结构或变量到枚举.
-   在1.0.0之后,只在增加主版本时进行断点更改.不要破坏建筑.
-   在1.0.0之后,不要添加任何新的公共API(没有新的)`pub`任何东西都是微小的版本.如果添加新的版本,则总是增加次要版本.`pub`结构、特性、字段、类型、函数、方法或其他任何东西.
-   使用具有三个数字部分的版本号,如1.0.0,而不是1.

#### The `build` field (optional)

此字段指定存储库中的文件,该文件是[构建脚本][1]用于生成本机代码.可以在生成脚本中找到更多信息.[指导][1].

[1]: 03-05-build-scripts.html

```toml
[package]
# ...
build = "build.rs"
```

#### The `documentation` field (optional)

此字段指定托管机箱文档的网站的URL.如果清单文件中没有指定URL,[克拉西奥][cratesio]自动将你的箱子连接到相应的箱子[博士学位][docsrs]页.

来自特定主机的文档链接被列入黑名单.如果已知主机不承载文档并且可能具有恶意意图,例如广告跟踪网络,则主机被添加到黑名单中.下列主机的URL被列入黑名单:

-   rust-ci.org

来自黑名单主机的文档URL将不会出现在CRATES.IO上,并且可以被DOCS.RS链接替换.

[docsrs]: https://docs.rs/

[cratesio]: https://crates.io/

#### The `exclude` and `include` fields (optional)

您可以显式地指定一组[地球仪][globs]包装和重建包装的目的应被忽略或包含.中指定的球`exclude`字段标识在发布包时不包括的一组文件,以及出于检测何时重建包的目的而忽略的文件,以及`include`指定显式包含的文件.

如果一个VCS被用于一个包,则`exclude`字段将被植入VCS的忽略设置(`.gitignore`例如Git.

```toml
[package]
# ...
exclude = ["build/**/*.o", "doc/**/*.html"]
```

```toml
[package]
# ...
include = ["src/**/*", "Cargo.toml"]
```

选项是相互排斥的:设置`include`将覆盖一个`exclude`. 注意`include`必须是一个详尽的文件列表,否则可能不包含源文件.

[globs]: http://doc.rust-lang.org/glob/glob/struct.Pattern.html

#### Migrating to `gitignore`-like pattern matching

这些配置的当前解释基于UNIX GULLS,如在[`glob` crate](https://crates.io/crates/glob). 我们想要货物`include`和`exclude`配置为类似于`gitignore`尽可能.[The `gitignore` specification](https://git-scm.com/docs/gitignore)也基于GLUPS,但它有一系列附加的特性,使模式更容易编写和控制.因此,我们正在迁移对这些配置的规则的解释,以便使用[`ignore`
crate](https://crates.io/crates/ignore)并将它们作为一行中的每一条规则对待`gitignore`文件.见[the tracking
issue](https://github.com/rust-lang/cargo/issues/4268)有关迁移的更多细节.

#### The `publish`  field (optional)

这个`publish`字段可以用来防止包被错误地发布到存储库.

```toml
[package]
# ...
publish = false
```

#### The `workspace`  field (optional)

这个`workspace`字段可以用来配置这个包将是一个成员的工作空间.如果没有指定,这将被推断为第一个CARGO.`[workspace]`文件系统中向上.

```toml
[package]
# ...
workspace = "path/to/root"
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

# These URLs point to more information about the repository. These are
# intended to be webviews of the relevant data, not necessarily compatible
# with VCS tools and the like.
documentation = "..."
homepage = "..."
repository = "..."

# This points to a file in the repository (relative to this `Cargo.toml`). The
# contents of this file are stored and indexed in the registry.
readme = "..."

# This is a list of up to five keywords that describe this crate. Keywords
# are searchable on crates.io, and you may choose any words that would
# help someone find this crate.
keywords = ["...", "..."]

# This is a list of up to five categories where this crate would fit.
# Categories are a fixed list available at crates.io/category_slugs, and
# they must match exactly.
categories = ["...", "..."]

# This is a string description of the license for this package. Currently
# crates.io will validate the license provided against a whitelist of known
# license identifiers from http://spdx.org/licenses/. Multiple licenses can be
# separated with a `/`.
license = "..."

# If a project is using a nonstandard license, then this key may be specified in
# lieu of the above key and must point to a file relative to this manifest
# (similar to the readme key).
license-file = "..."

# Optional specification of badges to be displayed on crates.io. The badges
# currently available are Travis CI, Appveyor, and GitLab latest build status,
# specified using the following parameters:
[badges]
# Travis CI: `repository` in format "<user>/<project>" is required.
# `branch` is optional; default is `master`
travis-ci = { repository = "...", branch = "master" }
# Appveyor: `repository` is required. `branch` is optional; default is `master`
# `service` is optional; valid values are `github` (default), `bitbucket`, and
# `gitlab`.
appveyor = { repository = "...", branch = "master", service = "github" }
# GitLab: `repository` is required. `branch` is optional; default is `master`
gitlab = { repository = "...", branch = "master" }
# Circle CI: `repository` is required. `branch` is optional; default is `master`
circle-ci = { repository = "...", branch = "master" }
# Is it maintained resolution time: `repository` is required.
is-it-maintained-issue-resolution = { repository = "..." }
# Is it maintained percentage of open issues: `repository` is required.
is-it-maintained-open-issues = { repository = "..." }
# Codecov: `repository` is required. `branch` is optional; default is `master`
# `service` is optional; valid values are `github` (default), `bitbucket`, and
# `gitlab`.
codecov = { repository = "...", branch = "master", service = "github" }
# Coveralls: `repository` is required. `branch` is optional; default is `master`
# `service` is optional; valid values are `github` (default) and `bitbucket`.
coveralls = { repository = "...", branch = "master", service = "github" }
```

这个[crates.io](https://crates.io)注册表将呈现描述,显示许可证,链接到三个URL并按关键字分类.这些密钥为注册表的用户提供有用的信息,也影响板条箱的搜索排名.在出版的板条箱里省略所有东西是非常令人沮丧的.

#### The `metadata` table (optional)

默认情况下,货物将对未使用的密钥发出警告.`Cargo.toml`协助检测错别字等.这个`package.metadata`表,但是,完全忽略了货物,将不会被警告.本节可用于存储项目配置的工具.`Cargo.toml`. 例如:

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

见[specifying dependencies page](03-01-specifying-dependencies.html)有关的信息`[dependencies]`,`[dev-dependencies]`,`[build-dependencies]`和目标特定的`[target.*.dependencies]`部分.

### The `[profile.*]` sections

货物支持自定义配置如何通过顶部的配置文件调用RUSTC.任何清单都可以声明配置文件,但实际上只读取顶级项目的配置文件.所有依赖项的配置文件将被重写.这样做,顶层项目就可以控制如何编译它的依赖关系.

目前有五个支持的配置文件名,所有这些配置文件都具有相同的配置可用.下面列出的是可用配置,以及每个配置文件的默认值.

```toml
# The development profile, used for `cargo build`.
[profile.dev]
opt-level = 0      # controls the `--opt-level` the compiler builds with.
                   # 0-1 is good for debugging. 2 is well-optimized. Max is 3.
debug = true       # include debug information (debug symbols). Equivalent to
                   # `-C debuginfo=2` compiler flag.
rpath = false      # controls whether compiler should set loader paths.
                   # If true, passes `-C rpath` flag to the compiler.
lto = false        # Link Time Optimization usually reduces size of binaries
                   # and static libraries. Increases compilation time.
                   # If true, passes `-C lto` flag to the compiler.
debug-assertions = true # controls whether debug assertions are enabled
                   # (e.g. debug_assert!() and arithmetic overflow checks)
codegen-units = 1  # if > 1 enables parallel code generation which improves
                   # compile times, but prevents some optimizations.
                   # Passes `-C codegen-units`. Ignored when `lto = true`.
panic = 'unwind'   # panic strategy (`-C panic=...`), can also be 'abort'

# The release profile, used for `cargo build --release`.
[profile.release]
opt-level = 3
debug = false
rpath = false
lto = false
debug-assertions = false
codegen-units = 1
panic = 'unwind'

# The testing profile, used for `cargo test`.
[profile.test]
opt-level = 0
debug = 2
rpath = false
lto = false
debug-assertions = true
codegen-units = 1
panic = 'unwind'

# The benchmarking profile, used for `cargo bench`.
[profile.bench]
opt-level = 3
debug = false
rpath = false
lto = false
debug-assertions = false
codegen-units = 1
panic = 'unwind'

# The documentation profile, used for `cargo doc`.
[profile.doc]
opt-level = 0
debug = 2
rpath = false
lto = false
debug-assertions = true
codegen-units = 1
panic = 'unwind'
```

### The `[features]` section

货物支持特征允许表达:

-   条件编译选项(可通过)`cfg`属性;
-   可选的依赖项,增强了包,但不是必需的;
-   可选依赖项的簇,如`postgres`这将包括`postgres`包`postgres-macros`包,以及可能的其他包(如开发时间嘲弄库、调试工具等).

包的一个特性是可选的依赖项或一组其他特征.指定特征的格式是:

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
-   当选择一个特征时,货物将调用`rustc`具有`--cfg
    feature="${feature_name}"`. 如果包含一个特征组,它将包含它的所有特征.这可以通过代码进行测试.`#[cfg(feature = "foo")]`.

注意,显式允许特性不实际激活任何可选依赖项.这允许包内部启用/禁用特征,而不需要新的依赖性.

#### Usage in end products

此功能的一个主要用例是指定最终产品中的可选特征.例如,伺服项目可能希望包括可选的特性,人们可以在其构建时启用或禁用.

在这种情况下,伺服将描述其特征.`Cargo.toml`它们可以使用命令行标志来启用:

```
$ cargo build --release --features "shumway pdf"
```

可以使用默认特性`--no-default-features`.

#### Usage in packages

在大多数情况下,概念*可选依赖*在库中最好表示为顶级应用程序所依赖的单独的包.

然而,高级别封装,如铁或活塞,可能需要的能力来策划一些包,便于安装.当前的货运系统允许他们将一组强制依赖项导入一个包中以便于安装.

在某些情况下,包可能希望为可选的依赖项提供额外的规约:

-   将多个低级别可选依赖项组合成单个高级特征;
-   指定包被用户推荐的(或建议的)包;
-   包括特征(类)`secure-password`在激励示例中,这只在可选的依赖项可用时才能工作,并且很难实现为单独的包(例如,设计一个与OpenSSL完全解耦的IO包可能过于困难,通过包含单独的包来选择进入).年龄).

在几乎所有的情况下,在设计用于高级化的软件包之外使用这些特征是一种反模式.如果一个特征是可选的,它几乎可以肯定地被表示为一个单独的包.

### The `[workspace]` section

项目可以定义一个工作空间,它是一组共享相同的板条箱.`Cargo.lock`和输出目录.这个`[workspace]`表可以定义为:

```toml
[workspace]

# Optional key, inferred if not present
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]

# Optional key, empty if not present
exclude = ["path1", "path/to/dir2"]
```

工作空间添加到货物作为一部分[RFC 1525]并具有许多属性:

-   工作空间可以包含多个板条箱,其中一个是根板条箱.
-   根部板条箱`Cargo.toml`包含`[workspace]`表,但不需要有其他配置.
-   每当编译工作空间中的任何箱时,输出都放在根箱的旁边.`Cargo.toml`.
-   工作空间中所有板条箱的锁定文件驻留在根箱的旁边.`Cargo.toml`.
-   这个`[patch]`和`[replace]`章节在`Cargo.toml`只在工作区根箱中识别,它们在成员箱的清单中被忽略.

[rfc 1525]: https://github.com/rust-lang/rfcs/blob/master/text/1525-cargo-workspace.md

工作空间的根板条,由`[workspace]`在其清单中,负责定义整个工作区.所有`path`驻留在工作区目录中的依赖项成为成员.您可以通过将它们添加到工作区中来将附加包添加到工作区中.`members`关键.注意,显式列出的工作区成员也将包含工作空间中的路径依赖项.有时一个项目可能会有很多工作空间成员,而且要跟上时代是繁重的.路径依赖也可以使用.[地球仪][globs]匹配多个路径.最后,`exclude`KEY可以用于将工作路径中的路径列入黑名单.如果一些路径依赖性根本不希望在工作空间中,这可能是有用的.

这个`package.workspace`在成员箱中使用清单密钥(如上文所述)指向工作空间的根箱.如果该密钥被省略,则推断为其清单包含的第一个板条箱.`[workspace]`文件系统中向上.

板条箱可以指定`package.workspace`或指定`[workspace]`. 也就是说,板条箱不能同时作为工作空间中的根箱(包含).`[workspace]`)也是另一个工作空间的成员箱(包含)`package.workspace`)

大多数时间工作区都不需要处理.`cargo new`和`cargo init`将自动处理工作区配置.

把它移到一个更合适的地方

### The project layout

如果项目是可执行文件,则命名主源文件`src/main.rs`. 如果它是一个库,请命名主源文件`src/lib.rs`.

货物也将处理任何文件位于`src/bin/*.rs`作为可执行文件.如果您的可执行文件不只是一个源文件,那么您也可以使用目录`src/bin`包含一个`main.rs`文件,将被视为可执行文件的父目录的名称.但是,一旦添加了`[[bin]]`剖面图(剖面)[see
below](#configuring-a-target),货物将不再自动建立文件位于`src/bin/*.rs`.  相反,你必须创建一个`[[bin]]`要生成的每个文件的节.

您的项目可以选择性地包含命名文件夹.`examples`,`tests`和`benches`这些货物将分别作为示例、集成测试和基准来处理.

```notrust
▾ src/           # directory containing source files
  lib.rs         # the main entry point for libraries and packages
  main.rs        # the main entry point for projects producing executables
  ▾ bin/         # (optional) directory containing additional executables
    *.rs
  ▾ */           # (optional) directories containing multi-file executables
    main.rs
▾ examples/      # (optional) examples
  *.rs
▾ tests/         # (optional) integration tests
  *.rs
▾ benches/       # (optional) benchmarks
  *.rs
```

在为项目创建文件和文件夹之后,要构造代码,您应该记住使用RISE的模块系统,您可以阅读[the book](https://doc.rust-lang.org/book/crates-and-modules.html).

### Examples

位于下方的文件`examples`是库提供的功能的示例使用.编译时,它们被放置在`target/examples`目录.

它们可以编译为可执行文件(用`main()`函数或库并通过使用来拉入库`extern crate <library-name>`. 当您运行测试以保护它们免遭篡改时,它们会被编译.

可以使用命令运行单个可执行示例.`cargo run --example
<example-name>`.

指定`crate-type`把一个例子编译成一个库:

```toml
[[example]]
name = "foo"
crate-type = ["staticlib"]
```

可以使用命令构建单个库实例.`cargo build
--example <example-name>`.

### Tests

当你奔跑`cargo test`货物将:

-   编译并运行库的单元测试,这些单元测试可从`lib.rs`(当然,任何部分标记为`#[cfg(test)]`将在现阶段考虑);
-   编译并运行您的库的文档测试,这些测试被嵌入到文档块内;
-   编译并运行您的库[integration tests](#integration-tests)和
-   编译你的库的例子.

#### Integration tests

每个文件在`tests/*.rs`是一个集成测试.当你奔跑`cargo test`货物将作为一个单独的箱子来编译这些文件中的每一个.板条箱可以通过使用链接链接到您的库.`extern crate <library-name>`就像其他依赖它的代码一样.

货物不会自动编译子目录内的文件`tests`但是,集成测试可以像往常一样从这些目录导入模块.例如,如果希望多个集成测试共享一些代码,则可以将共享代码放入`tests/common/mod.rs`然后放`mod common;`在每个测试文件中.

### Configuring a target

所有的`[[bin]]`,`[lib]`,`[[bench]]`,`[[test]]`和`[[example]]`节支持类似的配置,用于指定如何构建目标.双括号截面`[[bin]]`数组的表[TOML](https://github.com/toml-lang/toml#array-of-tables)这意味着你可以写多于一个`[[bin]]`在您的箱中生成几个可执行文件.

下面的例子使用`[lib]`,但它也适用于所有其他部分.除非另有说明,所有列出的值都是该选项的默认值.

```toml
[package]
# ...

[lib]
# The name of a target is the name of the library that will be generated. This
# is defaulted to the name of the package or project, with any dashes replaced
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
```

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

如果项目生成了一个库,则可以通过显式列出您的库来指定要构建哪种库.`Cargo.toml`:

```toml
# ...

[lib]
name = "..."
crate-type = ["dylib"] # could be `staticlib` as well
```

可用的选项是`dylib`,`rlib`,`staticlib`,`cdylib`和`proc-macro`. 您只应该在项目中使用此选项.货物总是根据包含它们的项目的要求编译包(依赖项).

您可以阅读更多关于不同的板条类型[Rust Reference Manual](https://doc.rust-lang.org/reference/linkage.html)

### The `[patch]` Section

这段车厢可以用[重写依赖项][replace]与其他副本.语法类似于`[dependencies]`章节:

```toml
[patch.crates-io]
foo = { git = 'https://github.com/example/foo' }
bar = { path = 'my/local/bar' }
```

这个`[patch]`表由依赖性的子表组成.每个键后`[patch]`是正在修补的源的URL,或者`crates-io`如果你正在修改<https://crates.io>注册表.在上面的例子中`crates-io`可以用Git URL替换,例如`https://github.com/rust-lang-nursery/log`.

这些表中的每个条目都是正常的依赖性规范,与在`[dependencies]`清单的部分.中列出的依赖项`[patch]`部分被解析并用于在指定的URL上对源进行补丁.上面的显示片段补丁`crates-io`源(例如CRATESIO本身)与`foo`板条箱`bar`机箱.

可以用不存在的板条箱版本来修补源,也可以用已经存在的板条箱版本来修补源.如果源已经用源代码中已经存在的板条版本修补,那么源的原始板条被替换.

有关重写依赖关系的更多信息可以在[重写依赖项][replace]文件和部分[RFC 1969]对于这一特性的技术规范.请注意`[patch]`特性将首先成为锈1.20,可在2017至0831发布.

[rfc 1969]: https://github.com/rust-lang/rfcs/pull/1969

[replace]: 03-01-specifying-dependencies.html#overriding-dependencies

### The `[replace]` Section

这段车厢可以用[重写依赖项][replace]与其他副本.语法类似于`[dependencies]`章节:

```toml
[replace]
"foo:0.1.0" = { git = 'https://github.com/example/foo' }
"bar:1.0.2" = { path = 'my/local/bar' }
```

中的每个键`[replace]`表是[package id
specification](03-07-pkgid-spec.html)它允许任意选择依赖图中的节点来重写.每个密钥的值与`[dependencies]`指定依赖关系的语法,除了不能指定特性外.注意,当覆盖一个板条箱时,它被覆盖的副本必须具有相同的名称和版本,但是它可以来自不同的源(例如,g it或本地路径).

有关重写依赖关系的更多信息可以在[重写依赖项][replace]部分文件.
