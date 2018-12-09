## 依赖指定

您依赖的箱子，其实有多个来源，如[crates.io],`git`的存储库或本地文件系统上的子目录。您还可以临时覆盖依赖项的位置 - 例如, 便于能够测试您在本地工作的依赖项中的错误修复。您可以为不同的平台，和仅在开发期间使用不同的依赖项。我们来看看如何做到这些.

### 指定依赖，来自 crates.io

默认情况下，Cargo 是准备好在[crates.io]上查找依赖项。在这种情况下,只需要名称和版本字符串。在[Cargo 指南](../guide/index.zh.md),我们选择了依赖-`time`箱:

```toml
[dependencies]
time = "0.1.12"
```

字符串`"0.1.12"`是一个[semver]版本格式字符串。由于此字符串中没有任何运算符,因此它的解释方式与我们指定的`"^0.1.12"`方式相同，而`^`被称为跳脱条件.

[semver]: https://github.com/steveklabnik/semver#requirements

### Caret requirements(跳脱条件)

**跳脱条件**: 允许 SemVer 兼容更新指定版本。新的版本允许更新的条件是，不修改最左边的非零数字(无论`major,minor,patch`)。在这种情况下，如果我们执行了`cargo update -p time`，Cargo 应该更新我们的`0.1.13`版本(如果是最新的`0.1.z`发布)，但不会更新为`0.2.0`。相反,我们若将版本字符串指定为`^1.0`，Cargo 应更新至`1.1`，如果是最新的`1.y`发布，但不是`2.0`版本。`0.0.x`并不与任何其他版本兼容.

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

此兼容性约定与 SemVer ，在处理 1.0.0 之前的版本方面有所不同。虽然 SemVer 说在 1.0.0 之前没有兼容性，但 Cargo 认为`0.x.y`是兼容`0.x.z`,这里`y ≥ z`和`x > 0`.

### Tilde 条件

**Tilde 条件**指定具有更新最小版本的一定能力。如果指定 major 版本,minor 版本和 patch 程序版本，或仅指定 major 版本和 minor 版本,则仅允许 patch 程序级别更改。如果仅指定 major 版本,则允许进行 minor 和 patch 级别更改.

`~1.2.3`是 Tilde 条件的一个例子.

```notrust
~1.2.3 := >=1.2.3 <1.3.0
~1.2 := >=1.2.0 <1.3.0
~1 := >=1.0.0 <2.0.0
```

### 通配符要求

**通配符条件**允许任何通配符所在的版本.

`*`,`1.*`和`1.2.*`是通配符条件的示例.

```notrust
* := >=0.0.0
1.* := >=1.0.0 <2.0.0
1.2.* := >=1.2.0 <1.3.0
```

### Inequality requirements(范围条件)

**范围条件**允许手动指定要依赖的版本范围或确切版本.

以下是范围条件的一些示例:

```notrust
>= 1.2.0
> 1
< 2
= 1.2.3
```

### 多版本条件

多个版本，要求用逗号分隔,例如`>= 1.2, < 1.5`.

### 依赖指定，来自 `git` 存储库

依赖于位于`git`存储库的库，您需要指定的最小信息，为一个`git`字段，其是存储库的`github`位置:

```toml
[dependencies]
rand = { git = "https://github.com/rust-lang-nursery/rand" }
```

Cargo 将取得`git`，然后在这个位置找到一个存储库的请求箱子的`Cargo.toml`。方式是对`git`存储库里面的任何地方(不一定在根目录) - 例如,指定工作区中的成员包名称，和设置`git`到包含工作区的存储库).

由于我们尚未指定任何其他信息，因此 Cargo 假定我们打算使用最新的提交`master`分支，来构建我们的包。你可以将`git`字段和`rev`,`tag`, 还有`branch`，这些用于指定其他内容的字段组合起来。这是一个指定您希望在名为`next`分支上，使用最新提交的示例:

```toml
[dependencies]
rand = { git = "https://github.com/rust-lang-nursery/rand", branch = "next" }
```

### 路径，依赖指定

随着时间的推移,我们来自[指南](../guide/index.zh.md)的`hello_world`示例已大幅增长! 它已经到了我们可能想分出一个单独的箱子供其他人使用的地步。为此,Cargo 支持**路径依赖**通常是位于一个存储库中的子箱。让我们开始在`hello_world`包的内部制作一个新的箱子:

```console
# inside of hello_world/
$ cargo new hello_utils
```

这将创建一个新文件夹`hello_utils`，里面有一个`Cargo.toml`和`src`文件夹已准备好进行配置。为了告诉 Cargo,请打开`hello_world/Cargo.toml`，并添加你的`hello_utils`依赖:

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

### 依赖覆盖

Cargo 中有许多方法支持，覆盖依赖关系以及控制依赖关系图。但是，这些选项通常仅在工作区级别可用，并且不通过依赖项传播。换句话说，"应用程序"具有覆盖依赖关系的能力,但"库"却没有。

许多场景，会产生想，覆盖依赖性或以其他方式改变某些依赖关系的愿望。然而,他们中的大多数都可以归结为，将箱子发布到 crates.io 之前使用箱子(覆盖依赖)的能力。例如:

- 您编写的 `crate` ，也用于您编写的更大应用程序中，并且您希望测试在更大应用程序内，`crate`的错误修复情况。
- 不是你编写的上游包，现在其 git 存储库的主分支上，有一个新功能或错误修复，您要测试它。
- 您即将发布新版本的 major 版本，但您希望在整个软件包中进行集成测试,以确保新的主要版本能够正常运行.
- 您已经为上游的软件包提交了一个针对您找到的错误的修复程序，但是您希望立即让您的应用程序依赖，此程序包的固定修复版本，以避免错误修复程序被拒绝合并.

这些场景目前都是通过[`[patch]` 清单部分 ][patch-section]解决的，从历史上看,其中一些方案是[该`[replace]`部分][replace-section]解决的，但我们在这里会记录`[patch]`解决的部分。

[patch-section]: ./manifest.md#the-patch-section
[replace-section]: ./manifest.md#the-replace-section

### 测试一个错误修复

假设你正在使用[`uuid` crate]，但是当你正在研究它时,你会发现一个错误.但是,你很有进取心,所以你决定尝试修复这个 bug! 最初你的清单看起来像:

[`uuid` crate]: https://crates.io/crates/uuid

```toml
[package]
name = "my-library"
version = "0.1.0"
authors = ["..."]

[dependencies]
uuid = "1.0"
```

我们要做的第一件事是克隆[`uuid`存储库][uuid-repository],到本地:

```console
$ git clone https://github.com/rust-lang-nursery/uuid
```

接下来我们将编辑`my-library`-Cargo.toml，为:

```toml
[patch.crates-io]
uuid = { path = "../path/to/uuid" }
```

在这里,我们宣布我们是*修补(patch)*来源`crates-io`，其有一个新的依赖，这将有效地添加本地(签出 checkout)版本`uuid`到 crates.io 注册表，指向本地包。

接下来我们需要确保我们的锁(lock)文件已更新为，使用此新版本`uuid`，所以我们的包使用本地签出的副本，而不是 crates.io 中的副本。`[patch]`工作方式是它将从`../path/to/uuid`加载依赖，然后每当 crates.io 查询`uuid`的版本时，它*也*会返回本地版本.

这意味着本地签出的版本号很重要，会影响是否使用该补丁。我们的清单宣布`uuid = "1.0"`，这意味着我们只会解析`>= 1.0.0, < 2.0.0`，和 Cargo 的贪婪解析算法，也意味着我们将解析到该范围内的最大版本。通常情况下这并不重要，因为 git 存储库的版本已经更大，或与 crates.io 上发布的最大版本相匹配，但重要的是要记住这一点!

无论如何,通常您现在需要做的就是:

```console
$ cargo build
   Compiling uuid v1.0.0 (.../uuid)
   Compiling my-library v0.1.0 (.../my-library)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

<!--  -->

就是这样!您现在正在使用本地版本构建`uuid`(注意构建输出中括号中的路径).如果您没有看到构建本地路径版本,那么您可能需要运行`cargo update -p uuid --precise $version`哪里`$version`是本地签出的副本的版本`uuid`.

一旦你修复了你最初发现的错误,你要做的下一件事就是将其作为拉取请求提交给`uuid`箱子本身.一旦你完成了这个,你也可以更新`[patch]`部分.里面的内容`[patch]`就像是`[dependencies]`部分,所以一旦你的拉动请求合并,你可以改变你的`path`依赖:

```toml
[patch.crates-io]
uuid = { git = 'https://github.com/rust-lang-nursery/uuid' }
```

[uuid-repository]: https://github.com/rust-lang-nursery/uuid

### Working with an unpublished minor version

现在让我们从错误修复到添加功能稍微改变一下.在努力的同时`my-library`你发现需要一个全新的功能`uuid`箱.您已实现此功能,并在上面本地测试`[patch]`,并提交了拉取请求.让我们来看看在实际发布之前你如何继续使用和测试它.

我们也说当前版本的`uuid`在 crates.io 上`1.0.0`,但从那时起,git 存储库的主分支已更新为`1.0.1`.此分支包含您之前提交的新功能.要使用此存储库,我们将编辑我们的`Cargo.toml`看起来像

```toml
[package]
name = "my-library"
version = "0.1.0"
authors = ["..."]

[dependencies]
uuid = "1.0.1"

[patch.crates-io]
uuid = { git = 'https://github.com/rust-lang-nursery/uuid' }
```

注意我们对本地的依赖`uuid`已更新为`1.0.1`因为这是我们在箱子出版后实际需要的东西.但是,这个版本在 crates.io 上不存在,所以我们提供了它`[patch]`清单的一部分.

现在,当我们的库被构建时,它将被取出`uuid`从 git 存储库并解析到存储库中的 1.0.1 而不是尝试从 crates.io 下载版本.一旦 1.0.1 发布在 crates.io 上`[patch]`部分可以删除.

值得注意的是`[patch]`适用*及物动词*.假设您使用`my-library`在更大的包中,例如:

```toml
[package]
name = "my-binary"
version = "0.1.0"
authors = ["..."]

[dependencies]
my-library = { git = 'https://example.com/git/my-library' }
uuid = "1.0"

[patch.crates-io]
uuid = { git = 'https://github.com/rust-lang-nursery/uuid' }
```

记住这一点`[patch]`适用*及物动词*但只能在*顶层*所以我们的消费者`my-library`不得不重复`[patch]`部分如有必要.不过,在这里,新的`uuid`箱子适用于*都*我们对...的依赖`uuid`和`my-library -> uuid`依赖.该`uuid`crate 将被解析为整个 crate 图表 1.0.1 的一个版本,并且它将从 git 存储库中提取.

#### Overriding repository URL

如果未加载要覆盖的依赖项`crates.io`,你将不得不改变一下你的使用方式`[patch]`:

```toml
[patch."https://github.com/your/repository"]
my-library = { path = "../my-library/path" }
```

就是这样!

### Prepublishing a breaking change

作为最后一个场景,让我们来看看使用一个新的主要版本的箱子,通常伴随着重大变化.坚持我们以前的板条箱,这意味着我们将创建 2.0.0 版本`uuid`箱.在我们提交了所有上游更改后,我们可以更新我们的清单`my-library`看起来像:

```toml
[dependencies]
uuid = "2.0"

[patch.crates-io]
uuid = { git = "https://github.com/rust-lang-nursery/uuid", branch = "2.0.0" }
```

就是这样!与前面的示例一样,2.0.0 版本实际上并不存在于 crates.io 上,但我们仍然可以通过使用`[patch]`部分.作为一个思考练习让我们再看看`my-binary`从上面再次表现出来:

```toml
[package]
name = "my-binary"
version = "0.1.0"
authors = ["..."]

[dependencies]
my-library = { git = 'https://example.com/git/my-library' }
uuid = "1.0"

[patch.crates-io]
uuid = { git = 'https://github.com/rust-lang-nursery/uuid', branch = '2.0.0' }
```

请注意,这实际上将解析为两个版本的`uuid`箱.该`my-binary`箱子将继续使用 1.x.y 系列`uuid`箱子但是`my-library`crate 将使用 2.0.0 版本`uuid`.这将允许您通过依赖关系图逐步推出对包的更改,而无需一次性更新所有内容.

### Overriding with local dependencies

有时你只是暂时在箱子上工作而你不想修改`Cargo.toml`喜欢的`[patch]`以上部分.对于这个用例,Cargo 提供了更为有限的覆盖版本**路径覆盖**.

路径覆盖是通过指定的`.cargo/config`代替`Cargo.toml`,你可以找到[有关此配置的更多文档][config-docs].代替`.cargo/config`你将指定一个名为的密钥`paths`:

[config-docs]: reference/config.md

```toml
paths = ["/path/to/uuid"]
```

该数组应填充包含 a 的目录`Cargo.toml`.在这种情况下,我们只是添加`uuid`,所以它将是唯一一个被覆盖的人.此路径可以是包含该路径的绝对路径或相对路径`.cargo`夹.

路径覆盖比限制更严格`[patch]`但是,部分不能改变依赖图的结构.当使用路径替换时,前一组依赖项必须完全匹配新的`Cargo.toml`规格.例如,这意味着路径覆盖不能用于测试向条件箱添加依赖项`[patch]`必须在那种情况下使用.因此,路径覆盖的使用通常与快速错误修复隔离,而不是更大的更改.

注意:使用本地配置覆盖路径仅适用于已发布到的包[crates.io].您无法使用此功能告诉 Cargo 如何查找本地未发布的板条箱.

### Platform specific dependencies

特定于平台的依赖项采用相同的格式,但在 a 下列出`target`部分.通常像锈一样`#[cfg]`语法将用于定义这些部分:

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

与 Rust 一样,这里的语法支持`not`,`any`,和`all`运算符组合各种 cfg 名称/值对.请注意`cfg`语法仅在 Cargo 0.9.0(Rust 1.8.0)之后可用.

此外`#[cfg]`语法,Cargo 还支持列出依赖关系适用的完整目标:

```toml
[target.x86_64-pc-windows-gnu.dependencies]
winhttp = "0.4.0"

[target.i686-unknown-linux-gnu.dependencies]
openssl = "1.0.1"
```

如果您使用的是自定义目标规范,请引用完整路径和文件名:

```toml
[target."x86_64/windows.json".dependencies]
winhttp = "0.4.0"

[target."i686/linux.json".dependencies]
openssl = "1.0.1"
native = { path = "native/i686" }

[target."x86_64/linux.json".dependencies]
openssl = "1.0.1"
native = { path = "native/x86_64" }
```

### Development dependencies

你可以添加一个`[dev-dependencies]`你的部分`Cargo.toml`其格式相当于`[dependencies]`:

```toml
[dev-dependencies]
tempdir = "0.3"
```

编译用于构建的包时不使用 Dev 依赖性,但用于编译测试,示例和基准.

这些依赖关系是*不*传播到依赖于此包的其他包.

您还可以使用具有特定于目标的开发依赖项`dev-dependencies`在目标节标题而不是`dependencies`.例如:

```toml
[target.'cfg(unix)'.dev-dependencies]
mio = "0.0.1"
```

[crates.io]: https://crates.io/

### Build dependencies

您可以依赖其他基于 Cargo 的板条箱在构建脚本中使用.依赖关系是通过`build-dependencies`清单的一部分:

```toml
[build-dependencies]
cc = "1.0.3"
```

构建脚本**才不是**有权访问中列出的依赖项`dependencies`要么`dev-dependencies`部分.除非在下面列出,否则构建依赖项同样不可用于包本身`dependencies`部分也是如此.包本身及其构建脚本是单独构建的,因此它们的依赖关系不必重合.通过将独立依赖性用于独立目的,使 Cargo 更简单,更清洁.

### Choosing features

如果您依赖的包提供条件功能,您可以指定使用哪个:

```toml
[dependencies.awesome]
version = "1.3.5"
default-features = false # do not include the default features, and optionally
                         # cherry-pick individual features
features = ["secure-password", "civet"]
```

有关功能的更多信息,请参阅[清单文件](reference/manifest.md#the-features-section).

### Renaming dependencies in `Cargo.toml`

写作的时候`[dependencies]`部分`Cargo.toml`您为依赖项编写的密钥通常与您在代码中导入的包的名称相匹配.但是,对于某些项目,您可能希望在代码中引用具有不同名称的包,而不管它是如何在 crates.io 上发布的.例如,您可能希望:

- 避免需要`use foo as bar`在 Rust 来源.
- 取决于箱子的多个版本.
- 依赖来自不同注册管理机构的同名包装箱.

为了支持这个 Cargo 支持 a`package`关键在于`[dependencies]`应该依赖哪个包的部分:

```toml
[package]
name = "mypackage"
version = "0.0.1"

[dependencies]
foo = "0.1"
bar = { git = "https://github.com/example/project", package = "foo" }
baz = { version = "0.1", registry = "custom", package = "foo" }
```

在此示例中,Rust 代码中现在提供了三个包:

```rust
extern crate foo; // crates.io
extern crate bar; // git repository
extern crate baz; // registry `custom`
```

所有这三个板条箱的包装名称都是`foo`在他们自己`Cargo.toml`所以我们明确地使用了`package`告知 Cargo 我们想要的关键`foo`包即使我们在本地调用其他东西.该`package`key,如果未指定,则默认为所请求的依赖项的名称.

请注意,如果您有可选的依赖项,例如:

```toml
[dependencies]
foo = { version = "0.1", package = 'bar', optional = true }
```

你依赖于箱子`bar`来自 crates.io,但你的箱子有一个`foo`功能而不是`bar`特征.也就是说,在重命名时,功能的名称取决于依赖项的名称,而不是包名称.

启用传递依赖项的工作方式类似,例如我们可以将以下内容添加到上面的清单中:

```toml
[features]
log-debug = ['foo/log-debug'] # using 'bar/log-debug' would be an error!
```
