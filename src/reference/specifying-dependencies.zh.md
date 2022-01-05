## 依赖指定

您的箱子，可以依赖多个来源的库，如[crates.io]，或是`git`的存储库或本地文件系统上的子目录。您还可以临时覆盖依赖项的位置 - 例如，便于能够测试，本地工作的错误修复。您可以为不同的平台，和或仅在开发期间，使用不同的依赖项。我们来看看如何做到这些.

### 指定依赖，来自 crates.io

默认情况下，Cargo 是准备好，在[crates.io]上查找依赖项。在这种情况下，只需要名称和版本字符串。在[Cargo 指南](../guide/index.zh.md)，我们选择了一个依赖项-`time`箱:

```toml
[dependencies]
time = "0.1.12"
```

字符串`"0.1.12"`是一个版本。虽然看起来是一个规定的版本，但其实是范围性的版本，并且允许[SemVer]版本更新。看起来，此字符串中没有任何运算符，但它的解释方式与我们指定的`"^0.1.12"`方式相同，而`^`被称为跳脱条件.

[semver]: https://semver.org

### Caret requirements(跳脱条件)

**跳脱条件**: 允许 SemVer 兼容更新指定版本。新的版本允许更新的条件是，不修改最左边的非零数字(无论`major，minor，patch`)。

在这种情况下，如果我们执行了`cargo update -p time`，Cargo 应该更新我们的`0.1.13`版本(如果是最新的`0.1.z`版本)，但不会更新为`0.2.0`。相反，我们若将版本字符串指定为`^1.0`，Cargo 应更新至`1.1`，如果是最新的`1.y`发布，但不是`2.0`版本。

`0.0.x`并不与任何其他版本兼容.

以下是一些跳脱条件的例子以及它们允许的版本:

```notrust
^1.2.3 := >=1.2.3 <2.0.0
^1.2 := >=1.2.0 <2.0.0
^1 := >=1.0.0 <2.0.0
^0.2.3 := >=0.2.3 <0.3.0
^0.2 := >= 0.2.0 < 0.3.0
^0.0.3 := >=0.0.3 <0.0.4
^0.0 := >=0.0.0 <0.1.0
^0 := >=0.0.0 <1.0.0
```

此兼容性约定与 SemVer，在处理 1.0.0 之前的版本方面有所不同。虽然 SemVer 说在 1.0.0 之前没有兼容性，但 Cargo 认为`0.x.y`是兼容`0.x.z`，这里`y ≥ z`和`x > 0`。

更灵活地调整版本，是有可能，但何苦呢，是吧。

### Caret requirements

**Caret requirements** 会是默认策略的一个备选项,
`^1.2.3` 其实就等于 `1.2.3`.

### Tilde 条件

**Tilde 条件**指定具有更新最小版本的一定能力。如果指定 major 版本，minor 版本和 patch 程序版本，或仅指定 major 版本和 minor 版本，则仅允许 patch 程序级别更改。如果仅指定 major 版本，则允许进行 minor 和 patch 级别更改.

`~1.2.3`是 Tilde 条件的一个例子.

```notrust
~1.2.3 := >=1.2.3 <1.3.0
~1.2 := >=1.2.0 <1.3.0
~1 := >=1.0.0 <2.0.0
```

### 通配符要求

**通配符条件**允许任何通配符所在的版本.

`*`，`1.*`和`1.2.*`是通配符条件的示例.

```notrust
* := >=0.0.0
1.* := >=1.0.0 <2.0.0
1.2.* := >=1.2.0 <1.3.0
```

### 比较条件

**比较条件**允许手动指定要依赖的版本范围或确切版本.

以下是比较条件的一些示例:

```notrust
>= 1.2.0
> 1
< 2
= 1.2.3
```

### 多版本条件

多个版本，要求用逗号分隔，例如`>= 1.2， < 1.5`.

### 其他来源的依赖项

要指定其他来源的依赖项，而不是 [crates.io], 第一步：配置`.cargo/config.toml` file. See the [registries
文档] 有更多信息。 `registry` 字段设置来源名称

```toml
[dependencies]
some-crate = { version = "1.0", registry = "my-registry" }
```

> **Note**: [crates.io] 是不允许有其他来源依赖项的项目，上传平台的。

[registries documentation]: registries.zh.md

### 依赖指定，来自 `git` 存储库

依赖于位于`git`存储库的库，您需要指定的最小信息，为一个`git`字段，其是存储库的`github`位置:

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex" }
```

Cargo 将取得`git`，然后在这个位置找到一个存储库的请求箱子的`Cargo.toml`。方式是对`git`存储库里面的任何地方(不一定在根目录) - 例如，指定工作区中的成员包名称，和设置`git`到包含工作区的存储库).

由于我们尚未指定任何其他信息，因此 Cargo 假定我们打算使用最新的提交`master`分支，来构建我们的包。你可以将`git`字段和`rev`，`tag`， 还有`branch`，这些用于指定其他内容的字段组合起来。这是一个指定您希望在名为`next`分支上，使用最新提交的示例:

```toml
[dependencies]
regex = { git = "https://github.com/rust-lang/regex", branch = "next" }
```

其他的 `rev`。可以是 commit `rev = "4c59b707"`；远程存储库 `rev = "refs/pull/493/head"`。其中，各家 git 运营商，可能会有不同的名称方案。

一旦 `git` 依赖被添加, Cargo 锁住依赖最新的 commit。 不会自动更新到
。但可以手动`cargo update`更新。

私有的库，要看看[Git Authentication]

> **Note**: [crates.io] 不允许夹杂 `git` 依赖
> (`git` [dev-dependencies] are ignored)。你也可以看看[Multiple
> locations](#multiple-locations) ，它有个备用的选择.

[git authentication]: ../appendix/git-authentication.zh.md

### 路径，依赖指定

随着时间的推移，我们来自[指南](../guide/index.zh.md)的`hello_world`示例已大幅增长! 它已经到了我们可能想分出一个单独的箱子供其他人使用的地步。为此，Cargo 支持**路径依赖**通常是位于一个存储库中的子箱。让我们开始在`hello_world`包的内部制作一个新的箱子:

```console
# inside of hello_world/
$ cargo new hello_utils
```

这将创建一个新文件夹`hello_utils`，里面有一个`Cargo.toml`和`src`文件夹已准备好进行配置。为了告诉 Cargo，请打开`hello_world/Cargo.toml`，并添加你的`hello_utils`依赖:

```toml
[dependencies]
hello_utils = { path = "hello_utils" }
```

这告诉 Cargo 我们依赖于一个叫做`hello_utils`的箱子，这能在`hello_utils`文件夹找到(相对于，写在`Cargo.toml`路径).

就是这样! 下一步`cargo build`将自动构建`hello_utils`，以及它自己的所有依赖项，其他人也可以开始使用它。但是，[crates.io]不允许仅使用 **路径指定依赖项** 的包。如果我们想发布我们的`hello_world`箱子，我们需要发布一个版本`hello_utils`至[crates.io](https://crates.io)，并在依赖项行中指定其版本:

```toml
[dependencies]
hello_utils = { path = "hello_utils", version = "0.1.0" }
```

### Multiple locations

可以指定，将来源版本与 `git` 或 `path`
位置结合起来。`git` 或 `path` 依赖会被本地使用 (这种情况下，
会针对本地的副本，检查 `version` ), 且当上传类似[crates.io]平台时, 会使用来源的版本 。其他的组合则不被允许。例如：

```toml
[dependencies]
# 本地就用 `my-bitflags` , 发布时，就是 version 1.0。
bitflags = { path = "my-bitflags", version = "1.0" }

# 本地用 git,  发布时，就是 version 1.0。
smallvec = { git = "https://github.com/servo/rust-smallvec", version = "1.0" }

# N.B. 注意 version 不正确, Cargo 编译会失败!
```

一个实际例子就是，在同一个工作区，你把一个库分进了多个包。本地版本，上传版本一手抓。与指定一个[override](overriding-dependencies.zh.md)类似, 但那个只有一个依赖声明可用。

### Platform specific dependencies

> 平台决定依赖

特定于平台的依赖项采用相同的格式，但在`target`下列出。像正常 Rust 一样的`#[cfg]`语法，将用于定义这些部分:

```toml
[target.'cfg(windows)'.dependencies]
winhttp = "0.4.0"

[target.'cfg(unix)'.dependencies]
openssl = "1.0.1"

[target.'cfg(target_arch = "x86")'.dependencies]
native = { path = "native/i686" }

[target.'cfg(target_arch = "x86_64")'.dependencies]
native = { path = "native/x86_64" }
```

与 Rust 一样，这里的语法支持`not`，`any`，和`all`运算符组合各种 cfg 名称/值对。

如果想要知道那个 cfg targets 在你平台上可用，运行`rustc --print=cfg`。如果，你想知道其他平台的可用，比如64-bit Windows,
run `rustc --print=cfg --target=x86_64-pc-windows-msvc`.

不同 Rust, 你不能使用
`[target.'cfg(feature = "fancy-feature")'.dependencies]` 要用 [the `[features]` section](features.zh.md)
:

```toml
[dependencies]
foo = { version = "1.0", optional = true }
bar = { version = "1.0", optional = true }

[features]
fancy-feature = ["foo", "bar"]
```

同样作用在 `cfg(debug_assertions)`, `cfg(test)` 个 `cfg(proc_macro)`.
这些值都不会工作，`rustc --print=cfg`会有默认值返回。
现在，通过配置值，添加依赖的方式还没有。

除了`#[cfg]`语法，Cargo 还支持列出依赖关系适用的完整目标:

```toml
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"

[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
```


#### 自定义的 target 规范

如果你使用一个自定义的 target 规范 (such as `--target
foo/bar.json`), 使用时，不要加上`.json` 后缀:

```toml
[target.bar.dependencies]
winhttp = "0.4.0"

[target.my-special-i686-platform.dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }
```

> **Note**: 这功能是还没有在 stabel channel 落户。


### Development dependencies

> 开发(Dev)依赖项

你可以添加一个`[dev-dependencies]`表格到`Cargo.toml`，其格式相当于`[dependencies]`:

```toml
[dev-dependencies]
tempdir = "0.3"
```

编译用于构建的包时，不会使用 Dev 依赖，但用于编译测试，示例和基准。

这些依赖关系是*不会*传播到依赖于此包的其他包.

您还可以让`dev-dependencies`具有特定目标的开发依赖项，而不是`dependencies`标题。例如:

```toml
[target.'cfg(unix)'.dev-dependencies]
mio = "0.0.1"
```

> **Note**: 当一个包被发布后, 只有 dev-dependencies 指定的
> `version` 会被放进发布的箱子。更多情况下，
> dev-dependencies 仅在发布时有用, 但些许用户 (like OS
> packagers) 在箱子运行一些测试啊，所以,  `version` 能给多点信息，也挺好的。

### Build dependencies

> 构建 依赖项

您可以在构建脚本中使用，依赖其他基于 Cargo 的箱。依赖关系是由清单的`build-dependencies`部分定义:

```toml
[build-dependencies]
cc = "1.0.3"
```

构建脚本**并不是**有权访问，`dependencies`或`dev-dependencies`部分，列出的依赖项。除非也在`dependencies`部分下面列出，否则构建依赖项同样不可用于包本身。包本身及其构建脚本是分开构建的，因此它们的依赖关系不重合。通过对依赖项进行独立化，使 Cargo 更简单，更清洁。

### Choosing features

> 选择 特性

如果您依赖的包提供条件特性，您可以指定使用哪个:

```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # 不会包括默认特性， 和 任君选
                         # 单特性
features = ["secure-password", "civet"]
```

有关 features 的更多信息，请参阅[清单文档](./manifest.zh.md#the-features-section).

### Renaming dependencies in `Cargo.toml`

> 在`Cargo.toml`中的重命名依赖项

写`Cargo.toml`的`[dependencies]`部分的时候，您为依赖项编写的字段通常与您在代码中导入的包的名称相匹配。但是，对于某些项目，您可能希望在代码中引用具有不同名称的包，而不管它是如何在 crates.io 上发布的。例如，您可能希望:

- 避免在 Rust 代码常用`use foo as bar`.
- 依赖箱子的多个版本.
- 依赖来自不同注册表管理机构的同名箱.

为了支持这个 ，Cargo 在`[dependencies]`部分使用 一个`package`字段，决定应该依赖哪个包:

```toml
[package]
name = "mypackage"
version = "0.0.1"

[dependencies]
foo = "0.1"
bar = { git = "https://github.com/example/project", package = "foo" }
baz = { version = "0.1", registry = "custom", package = "foo" }
```

在此示例中，Rust 代码中现在提供了三个包:

```rust
extern crate foo; // crates.io
extern crate bar; // git repository
extern crate baz; // registry `custom`
```

所有这三个箱的包名称在他们自己`Cargo.toml`，都是`foo`，所以我们明确地告知 Cargo ，使用的是我们想要的`package`字段(如 package = "foo"包名，即我们在本地调用其他东西)。如果没有指定`package`，则默认为所请求的依赖项的名称。

请注意，如果您有一个可选的(optional)依赖项，例如:

```toml
[dependencies]
bar = { version = "0.1", package = 'foo', optional = true }
```

你依赖于一个`foo`箱子，其来自 crates.io，但你箱子有一个`bar`特性，取代了一个`foo`特性。也就是说，在重命名时，特性的名称拿掉了依赖项的名称，而不是包名称。

启用传递依赖项的工作方式类似，例如我们可以将以下内容，添加到上面的清单中:

```toml
[features]
log-debug = ['bar/log-debug'] # 使用 'foo/log-debug' 就会出现一个错误!
```

[crates.io]: https://crates.io/
[dev-dependencies]: #development-dependencies

<script>
(function() {
    var fragments = {
        "#overriding-dependencies": "overriding-dependencies.html",
        "#testing-a-bugfix": "overriding-dependencies.html#testing-a-bugfix",
        "#working-with-an-unpublished-minor-version": "overriding-dependencies.html#working-with-an-unpublished-minor-version",
        "#overriding-repository-url": "overriding-dependencies.html#overriding-repository-url",
        "#prepublishing-a-breaking-change": "overriding-dependencies.html#prepublishing-a-breaking-change",
        "#overriding-with-local-dependencies": "overriding-dependencies.html#paths-overrides",
    };
    var target = fragments[window.location.hash];
    if (target) {
        var url = window.location.toString();
        var base = url.substring(0, url.lastIndexOf('/'));
        window.location.replace(base + "/" + target);
    }
})();
</script>
