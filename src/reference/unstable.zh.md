## Unstable Features

实验Cargo功能仅适用于夜间通道.您通常使用其中一个`-Z`用于启用它们的标志.跑`cargo -Z help`查看可用的标志列表.

`-Z unstable-options`是用于启用其他不稳定命令行标志的通用标志.需要此选项的选项将在下面列出.

某些不稳定的功能需要您指定`cargo-features`键入`Cargo.toml`.

### Alternate Registries

-   RFC:[#2141](https://github.com/rust-lang/rfcs/blob/master/text/2141-alternative-registries.md)
-   跟踪问题:[锈琅/锈#44931](https://github.com/rust-lang/rust/issues/44931)

备用注册表允许您使用crates.io以外的注册表.

注册表的名称定义在`.cargo/config`在下面`registries`表:

```toml
[registries]
my-registry = { index = "https://my-intranet:8080/git/index" }
```

可以添加备用注册表的身份验证信息`.cargo/credentials`:

```toml
[registries.my-registry]
token = "api-token"
```

内`Cargo.toml`您可以使用指定依赖项来自哪个注册表`registry`键.首先,您需要包含适当的`cargo-features`在文件的顶部:

```toml
cargo-features = ["alternative-registries"]

[package]
...

[dependencies]
other-create = { version = "1.0", registry = "my-registry"}
```

一个`--registry`flag已添加到与注册表交互的命令中`publish`,`login`等.示例:

```
cargo +nightly publish -Z unstable-options --registry my-registry
```

该`publish`在`Cargo.toml`已扩展为接受将仅限制发布到这些注册表的注册表列表.

```toml
[package]
...
publish = ["my-registry"]
```

### publish-lockfile

-   原始问题:[#2263](https://github.com/rust-lang/cargo/issues/2263)
-   PR:[#5093](https://github.com/rust-lang/cargo/pull/5093)
-   跟踪问题:[#5654](https://github.com/rust-lang/cargo/issues/5654)

创建时`.crate`文件分发,Cargo历史上没有包括`Cargo.lock`文件.这可能会导致使用问题`cargo install`用二进制文件.您可以指定您的包应包含`Cargo.lock`使用时的文件`cargo package`要么`cargo publish`通过指定`publish-lockfile`键入`Cargo.toml`.这也需要适当的`cargo-features`:

```toml
cargo-features = ["publish-lockfile"]

[package]
...
publish-lockfile = true
```

### Offline Mode

-   原始问题:[#4686](https://github.com/rust-lang/cargo/issues/4686)
-   跟踪问题:[#5655](https://github.com/rust-lang/cargo/issues/5655)

该`-Z offline`flag可防止Cargo因任何原因尝试访问网络.通常情况下,如果Cargo要访问网络并且不可用,则会因错误而停止.

请注意,这可能会导致与在线模式不同的依赖项解析.即使可能有索引的本地副本中指示的较​​新版本,Cargo也会将自己限制在本地可用的包装箱中.

### no-index-update

-   原始问题:[#3479](https://github.com/rust-lang/cargo/issues/3479)

该`-Z no-index-update`flag确保Cargo不会尝试更新注册表索引.这适用于发布许多Cargo命令的Crater等工具,并且您希望每次都避免更新索引的网络延迟.

### avoid-dev-deps

-   原始问题:[#4988](https://github.com/rust-lang/cargo/issues/4988)
-   稳定问题:[#5133](https://github.com/rust-lang/cargo/issues/5133)

运行命令时如`cargo install`要么`cargo build`,Cargo目前需要下载dev依赖项,即使它们未被使用.该`-Z avoid-dev-deps`flag允许Cargo避免在不需要时下载dev依赖项.该`Cargo.lock`如果跳过dev-dependencies,将不会生成文件.

### minimal-versions

-   原始问题:[#4100](https://github.com/rust-lang/cargo/issues/4100)
-   跟踪问题:[#5657](https://github.com/rust-lang/cargo/issues/5657)

当一个`Cargo.lock`生成文件,`-Z minimal-versions`flag将解析依赖关系到满足要求的最小semver版本(而不是最高版本).

此标志的预期用例是在持续集成期间检查Cargo.toml中指定的版本是否是您实际使用的最低版本的正确反映.也就是说,如果Cargo.toml说`foo = "1.0.0"`您不会意外地依赖于仅添加的功能`foo 1.5.0`.

### out-dir

-   原始问题:[#4875](https://github.com/rust-lang/cargo/issues/4875)

此功能允许您指定构建工件后将复制到的目录.通常,工件只写入`target/release`要么`target/debug`目录.但是,确定确切的文件名可能很棘手,因为您需要解析JSON输出.该`--out-dir`flag可以更容易地预测访问工件.请注意,工件被复制,因此原件仍在`target`目录.例:

```
cargo +nightly build --out-dir=out -Z unstable-options
```

### Profile Overrides

-   跟踪问题:[锈琅/锈#48683](https://github.com/rust-lang/rust/issues/48683)
-   RFC:[#2282](https://github.com/rust-lang/rfcs/blob/master/text/2282-profile-dependencies.md)

可以为特定包和自定义生成脚本覆盖配置文件.一般格式如下:

```toml
cargo-features = ["profile-overrides"]

[package]
...

[profile.dev]
opt-level = 0
debug = true

# the `image` crate will be compiled with -Copt-level=3
[profile.dev.overrides.image]
opt-level = 3

# All dependencies (but not this crate itself or any workspace member)
# will be compiled with -Copt-level=2 . This includes build dependencies.
[profile.dev.overrides."*"]
opt-level = 2

# Build scripts and their dependencies will be compiled with -Copt-level=3
# By default, build scripts use the same rules as the rest of the profile
[profile.dev.build-override]
opt-level = 3
```

只能为dev和release配置文件指定覆盖.

### Config Profiles

-   跟踪问题:[锈琅/锈#48683](https://github.com/rust-lang/rust/issues/48683)
-   RFC:[#2282](https://github.com/rust-lang/rfcs/blob/master/text/2282-profile-dependencies.md)

可以在中指定配置文件`.cargo/config`文件.该`-Z config-profile`命令行标志是使用此功能所必需的.格式与a中的格式相同`Cargo.toml`表现.如果在多个配置文件中找到,则将使用常规合并设置[配置层次结构](reference/config.md#hierarchical-structure).配置设置优先于清单设置.

```toml
[profile.dev]
opt-level = 3
```

```
cargo +nightly build -Z config-profile
```

### Namespaced features

-   原始问题:[#1286](https://github.com/rust-lang/cargo/issues/1286)
-   跟踪问题:[防锈琅/Cargo#5565](https://github.com/rust-lang/cargo/issues/5565)

目前,不可能在清单中具有相同名称的功能和依赖项.如果你设置`namespaced-features`至`true`,功能和依赖项的命名空间是分开的.这样做的结果是,在功能要求中,依赖关系必须以前缀为前缀`crate:`.像这样:

```toml
[package]
namespaced-features = true

[features]
bar = ["crate:baz", "foo"]
foo = []

[dependencies]
baz = { version = "0.1", optional = true }
```

为了防止不必要的样板必须显式声明每个可选依赖项的功能,将为任何未定义相同名称的功能的可选依赖项创建隐式功能.但是,如果定义了与依赖项同名的功能,则该功能必须将依赖项作为必需项包括在内`foo = ["crate:foo"]`.

### Build-plan

-   跟踪问题:[防锈琅/Cargo#5579](https://github.com/rust-lang/cargo/issues/5579)

该`--build-plan`争论的`build`命令将输出JSON,其中包含有关将运行哪些命令而不实际执行任何操作的信息.与其他构建工具集成时,这可能很有用.例:

```
cargo +nightly build --build-plan -Z unstable-options
```

### default-run

-   原始问题:[#2200](https://github.com/rust-lang/cargo/issues/2200)

该`default-run`选项`[package]`清单的一部分可用于指定选择的默认二进制文件`cargo run`.例如,当两者兼而有之时`src/bin/a.rs`和`src/bin/b.rs`:

```toml
[package]
default-run = "a"
```

### Metabuild

-   跟踪问题:[锈琅/锈#49803](https://github.com/rust-lang/rust/issues/49803)
-   RFC:[#2196](https://github.com/rust-lang/rfcs/blob/master/text/2196-metabuild.md)

Metabuild是一个具有声明性构建脚本的功能.而不是写一个`build.rs`脚本,您在中指定构建依赖项列表`metabuild`键入`Cargo.toml`.将自动生成一个构建脚本,该脚本按顺序运行每个构建依赖项.然后,Metabuild包可以从中读取元数据`Cargo.toml`指定他们的行为.

包括`cargo-features`在顶部`Cargo.toml`, 一个`metadata`关键在于`package`,列出依赖项`build-dependencies`,并添加metabuild包所需的任何元数据.例:

```toml
cargo-features = ["metabuild"]

[package]
name = "mypackage"
version = "0.0.1"
metabuild = ["foo", "bar"]

[build-dependencies]
foo = "1.0"
bar = "1.0"

[package.metadata.foo]
extra-info = "qwerty"
```

Metabuild包应该有一个名为的公共函数`metabuild`执行与常规操作相同的操作`build.rs`脚本会执行.
