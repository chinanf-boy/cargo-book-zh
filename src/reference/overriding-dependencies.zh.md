### 依赖覆盖

Cargo 中有许多方法支持，覆盖依赖关系以及控制依赖关系图。但是，这些选项通常仅在工作区级别可用，并且不通过依赖项传播。换句话说，"应用程序"具有覆盖依赖关系的能力，但"库"却没有。

许多场景，会产生想，覆盖依赖性或以其他方式改变某些依赖关系的愿望。然而，他们中的大多数都可以归结为，将箱子发布到 crates.io 之前使用箱子(覆盖依赖)的能力。例如:

- 您编写的 `crate` ，也用于您编写的更大应用程序中，并且您希望测试在更大应用程序内，`crate`的错误修复情况。
- 不是你编写的上游包，现在其 git 存储库的主分支上，有一个新功能或错误修复，您要测试它。
- 您即将发布新版本的 major 版本，但您希望在整个软件包中进行集成测试，以确保新的主要版本能够正常运行.
- 您已经为上游的软件包提交了一个针对您找到的错误的修复程序，但是您希望立即让您的应用程序依赖，此程序包的固定修复版本，以避免错误修复程序被拒绝合并.

这些场景目前都是通过[`[patch]` 清单部分 ][patch-section]解决的，从历史上看，其中一些方案是[该`[replace]`部分][replace-section]解决的，但我们在这里会记录`[patch]`解决的部分。

[patch-section]: ./manifest.zh.md#the-patch-section
[replace-section]: ./manifest.zh.md#the-replace-section

### 测试一个错误修复

假设你正在使用[`uuid` crate]，但是当你正在研究它时，你会发现一个错误.但是，你很有进取心，所以你决定尝试修复这个 bug! 最初你的清单看起来像:

[`uuid` crate]: https://crates.io/crates/uuid

```toml
[package]
name = "my-library"
version = "0.1.0"
authors = ["..."]

[dependencies]
uuid = "1.0"
```

我们要做的第一件事是克隆[`uuid`存储库][uuid-repository]，到本地:

```console
$ git clone https://github.com/rust-lang-nursery/uuid
```

接下来我们将编辑`my-library`-Cargo.toml，为:

```toml
[patch.crates-io]
uuid = { path = "../path/to/uuid" }
```

在这里，我们宣布我们是*修补(patch)*来源`crates-io`，其有一个新的依赖，这将有效地添加本地(签出 checkout)版本`uuid`到 crates.io 注册表，指向本地包。

接下来我们需要确保我们的锁(lock)文件已更新为，使用此新版本`uuid`，所以我们的包使用本地签出的副本，而不是 crates.io 中的副本。`[patch]`工作方式是它将从`../path/to/uuid`加载依赖，然后每当 crates.io 查询`uuid`的版本时，它*也*会返回本地版本.

这意味着本地签出的版本号很重要，会影响是否使用该补丁。我们的清单宣布`uuid = "1.0"`，这意味着我们只会解析`>= 1.0.0， < 2.0.0`，和 Cargo 的贪婪解析算法，也意味着我们将解析到该范围内的最大版本。通常情况下这并不重要，因为 git 存储库的版本已经更大，或与 crates.io 上发布的最大版本相匹配，但重要的是要记住这一点!

无论如何，通常您现在需要做的就是:

```console
$ cargo build
   Compiling uuid v1.0.0 (.../uuid)
   Compiling my-library v0.1.0 (.../my-library)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

就是这样! 您现在正在使用本地版本`uuid`构建(注意构建输出中括号中的路径)。如果您没有看到构建本地路径版本，那么您可能需要运行`cargo update -p uuid --precise $version`，这里`$version`是本地签出版本的`uuid`副本。

一旦你修复了你最初发现的错误，你要做的下一件事就是将其作为拉取请求提交给`uuid`箱子本身。一旦你完成了这个，你也可以更新下`[patch]`部分。`[patch]`里面的内容列表就像是`[dependencies]`部分，所以一旦你的拉动请求合并，你就可以改变你的`path`依赖:

```toml
[patch.crates-io]
uuid = { git = 'https://github.com/rust-lang-nursery/uuid' }
```

[uuid-repository]: https://github.com/rust-lang-nursery/uuid

### Working with an unpublished minor version

> 与 一个未发布的次要版本，一起工作

现在让我们稍微改变一下，从错误修复，变成要添加功能。在努力`my-library`的同时，你发现需要`uuid`箱的一个全新的功能。而您已实现`uuid`此功能，并在`[patch]`上面进行本地测试，并提交了拉取请求。让我们来看看在实际发布之前，你如何继续使用和测试它。

我们也说当前版本的`uuid`，在 crates.io 上是`1.0.0`版本，但从提交那时起，git 存储库的主分支已更新为`1.0.1`。此分支包含您之前提交的新功能。要使用此存储库，我们将编辑我们的`Cargo.toml`，看起来像

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

注意我们对本地`uuid`的依赖已更新为`1.0.1`，因为这是我们在箱子发布后实际需要的东西。但是，这个版本在 crates.io 上不存在，所以我们提供给它清单的`[patch]`部分.

现在，当我们的库被构建时，它将`uuid`从 git 存储库取出，并解析到存储库中的 1.0.1 ，而不是尝试从 crates.io 下载版本。一旦 1.0.1 发布在 crates.io 上，那`[patch]`部分就可以删除了。

值得注意的是，`[patch]`是*连带关系*。假设您在更大的包中使用`my-library`，例如:

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

记住这`[patch]`是*连带关系*，但只能在*顶层*，所以我们的`my-library`消费者不得不重写`[patch]`部分(如有必要的话)。不过，在这里，新的`uuid`箱子会适用对`uuid`的依赖和`my-library -> uuid`的依赖，两个依赖**都**指定了。该`uuid`箱 将被解析为整个 crate 关系图 的 1.0.1 版本，并且它是将从 git 存储库中提取。

#### Overriding repository URL

> 覆盖 注册表 URL

如果要覆盖的依赖项不是加载自`crates.io`，你将不得不改变一下你的`[patch]`使用方式:

```toml
[patch."https://github.com/your/repository"]
my-library = { path = "../my-library/path" }
```

就是这样!

### Prepublishing a breaking change

> 预发布一个重要变化

让我们来看看最后一个场景。若要使用一个新的主要版本的箱子，其通常伴随着重大变化。而要坚持使用我们以前的箱，这意味着我们将创建 2.0.0 版本`uuid`箱。在我们提交了所有上游更改后，我们可以更新我们的`my-library`清单，看起来像:

```toml
[dependencies]
uuid = "2.0"

[patch.crates-io]
uuid = { git = "https://github.com/rust-lang-nursery/uuid", branch = "2.0.0" }
```

就是这样!与前面的示例一样，2.0.0 版本实际上，并不存在于 crates.io 上，但我们仍然可以通过`[patch]`部分使用。作为一个思考练习，让我们再看看`my-binary`(被使用)的再次表现:

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

请注意，这实际上将解析为两个版本的`uuid`箱。该`my-binary`箱子将继续使用 1.x.y 系列的`uuid`箱子，但是`my-library`箱 会使用 2.0.0 版本`uuid`。这将允许您通过依赖关系图逐步推出对包的更改，而无需一次性更新所有内容。

### Overriding with local dependencies

> 覆盖 本地依赖项

有时你只是暂时在一个箱子上工作，而你不想修改`Cargo.toml`中像上诉的`[patch]`部分。对于这个用例，Cargo 提供了更为有限的覆盖版本**路径覆盖**.

路径覆盖是通过`.cargo/config`指定，而不是`Cargo.toml`，你可以寻找[有关此配置的更多文档][config-docs]。在`.cargo/config`内，你要指定的是一个名为`paths`字段:

[config-docs]: ./config.zh.md

```toml
paths = ["/path/to/uuid"]
```

该数组应填充包含`Cargo.toml`的目录。在这种情况下，我们只是添加`uuid`，所以它将是唯一一个被覆盖的。此路径可以是包含该路径的绝对路径或相对`.cargo`文件夹的路径.

路径覆盖，比`[patch]`部分的限制更严格，但是，路径覆盖不能改变依赖图的结构。而当使用路径替换时，前一组依赖项必须完全匹配新的`Cargo.toml`规格。如此，就意味着路径覆盖不能用于向箱添加依赖项的测试，而换成`[patch]`在该种情况下使用。因此，路径覆盖的使用，通常会与快速错误修复分隔开来，而不是大更新分开。

注意:使用本地配置覆盖路径，仅适用于已发布到[crates.io]的包。您无法使用此功能告诉 Cargo 如何查找本地未发布的箱。