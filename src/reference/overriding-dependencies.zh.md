### 依赖覆盖

许多场景，我们会想去，覆盖一个依赖，或以其他方式改变某些依赖关系。然而，他们中的大多数都可以归结为，将箱子发布到 crates.io 之前，使用箱子的(覆盖依赖)能力。例如:

- 您编写的 `crate` ，也用于您编写的更大应用程序中，并且您希望在更大应用程序内，测试你这个`crate`的 bug fix 情况。
- 不是你编写的上游包，现在其 git 存储库的主分支上，有一个新功能或 bug fix ，您要测试它。
- 您即将发布新版本的 major 版本，但您希望在整个软件包中，进行集成测试，以确保新的主要版本能够正常运行.
- 您已经为上游的软件包，提交了一个针对您找到的 bug fix，但是您希望立即让您的应用程序，依赖此程序包的 fixed 版本，以避免 bug fix 程序被拒绝合并.

这些场景目前都是通过[`[patch]` 清单部分 ][#this-patch-section]解决的

- 用例
  - [Testing a bugfix{测试一个修复}](#testing-a-bugfix)
  - [Working with an unpublished minor version{使用一个未发布的 minor 版本}](#working-with-an-unpublished-minor-version)
    - [Overriding repository URL{覆盖存储库 URL}](#overriding-repository-url)
  - [Prepublishing a breaking change{预发布一个重大更新}](#prepublishing-a-breaking-change)
  - [Using `[patch]` with multiple versions{多版本}](#using-patch-with-multiple-versions)
- Reference
  - [`[patch]` 部分](#the-patch-section)
  - [`[replace]` 部分](#the-replace-section)
  - [`paths` 覆盖](#paths-overrides)

> **注意**: 还可以看看 [多来源][multiple locations]，去指定一个依赖，覆盖单个依赖来源。

### Testing a bugfix

假设你正在使用[`uuid` crate]，但是当你正在研究它时，你会发现一个错误。但是，你很有进取心，所以你决定尝试修复这个 bug! 最初你的清单看起来像:

[`uuid` crate]: https://crates.io/crates/uuid

```toml
[package]
name = "my-library"
version = "0.1.0"

[dependencies]
uuid = "1.0"
```

我们要做的第一件事是克隆[`uuid`存储库][uuid-repository]，到本地:

```console
$ git clone https://github.com/uuid-rs/uuid
```

接下来，我们将编辑`my-library`-Cargo.toml，为:

```toml
[patch.crates-io]
uuid = { path = "../path/to/uuid" }
```

在这里，我们宣布我们是*修补(patch)*来源`crates-io`，其有一个新的依赖，这将有效地添加本地版本的`uuid`到 crates.io 注册表，指向本地包。

接下来，我们需要确保我们的锁(lock)文件已更新为，新版本的`uuid`。自此，我们的包使用本地的副本，而不是 crates.io 中的副本。`[patch]`工作方式是，它将从`../path/to/uuid`加载依赖，然后每当 crates.io 查询`uuid`的版本时，它 _也_ 会返回本地版本.

这意味着，本地签出的版本号很重要，会影响是否使用该 patch。我们的清单宣布`uuid = "1.0"`，这意味着，我们只会解析`>= 1.0.0， < 2.0.0`，还有 Cargo 的贪婪解析算法，也意味着我们将解析到该范围内的最大版本。通常情况下，这并不重要，因为 git 存储库的版本已经比之更大，或与 crates.io 上发布的最大版本相匹配，但重要的是，要记住这一点!

无论如何，现在需要做的就是:

```console
$ cargo build
   Compiling uuid v1.0.0 (.../uuid)
   Compiling my-library v0.1.0 (.../my-library)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

就是这样! 您现在正在使用本地版本`uuid`，来构建你的程序(注意，构建输出中，括号中的路径)。如果，您没有看到构建本地路径版本，那么您可能需要运行`cargo update -p uuid --precise $version`，这里`$version`是本地签出版本的`uuid`副本。

一旦，你修复了你最初发现的错误，你要做的下一件事就是将其作为拉取请求，提交给`uuid`箱子本身。一旦，你完成了这部分，你也可以更新下`[patch]`部分。`[patch]`里面的内容列表就像是`[dependencies]`部分，所以，一旦你的拉动请求合并，你就可以改变你的`path`依赖:

```toml
[patch.crates-io]
uuid = { git = 'https://github.com/rust-lang-nursery/uuid' }
```

[uuid-repository]: https://github.com/rust-lang-nursery/uuid

### Working with an unpublished minor version

> 与 一个未发布的次要版本，一起工作

现在，让我们稍微改变一下，从 bug fix ，变成要添加功能。在努力`my-library`的同时，你发现需要`uuid`箱的一个全新的功能。而您已实现`uuid`此功能，并在`[patch]`上面进行本地测试，并提交了拉取请求。让我们来看看在实际发布之前，你如何继续使用和测试它。

我们也说当前版本的`uuid`，在 crates.io 上是`1.0.0`版本，但从提交那时起，git 存储库的主分支已更新为`1.0.1`。此分支包含您之前提交的新功能。要使用此存储库，我们将编辑我们的`Cargo.toml`，看起来像

```toml
[package]
name = "my-library"
version = "0.1.0"

[dependencies]
uuid = "1.0.1"

[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid' }
```

注意，我们对本地`uuid`的依赖已更新为`1.0.1`，因为，这是我们在箱子发布后实际需要的东西。但是，这个版本在 crates.io 上不存在，所以，我们提供了`[patch]`部分，来特指。

现在，当我们的库被构建时，它将`uuid`从 git 存储库取出，并解析到存储库中的 1.0.1 ，而不是尝试从 crates.io 下载版本。一旦 1.0.1 发布在 crates.io 上，那`[patch]`部分就可以删除了。

值得注意的是，`[patch]`是*连带关系*。假设您在更大的包中，使用`my-library`，例如:

```toml
[package]
name = "my-binary"
version = "0.1.0"

[dependencies]
my-library = { git = 'https://example.com/git/my-library' }
uuid = "1.0"

[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid' }
```

记住这`[patch]`是 _连带关系_，但只能在**顶层**，所以，我们的`my-library`用户不得不重复写入`[patch]`部分(如有必要的话)。不过，在这里，新的`uuid`箱子会适用对`uuid`的依赖和`my-library -> uuid`的依赖，两个依赖**都**指定了。该`uuid`箱 将被解析为整个 crate 关系图 的 1.0.1 版本，并且它是将从 git 存储库中 pull 而来。

#### Overriding repository URL

> 覆盖 注册表 URL

如果要覆盖的依赖项，不是加载自`crates.io`，你将不得不改变一下你的`[patch]`使用方式:

```toml
[patch."https://github.com/your/repository"]
my-library = { path = "../my-library/path" }
```

就是这样!

### Prepublishing a breaking change

> 预发布一个重要变化

让我们来看看。若要使用一个新的主要版本的箱子，其通常伴随着重大变化。而要坚持使用我们以前的箱，这意味着，我们将创建 2.0.0 版本`uuid`箱。在我们提交了所有上游更改后，我们可以更新我们的`my-library`清单，看起来像:

```toml
[dependencies]
uuid = "2.0"

[patch.crates-io]
uuid = { git = "https://github.com/rust-lang-nursery/uuid", branch = "2.0.0" }
```

就是这样! 与前面的示例一样，2.0.0 版本实际上，并不存在于 crates.io 上，但我们仍然可以通过`[patch]`部分使用。作为一个思考练习，让我们再看看`my-binary`(被使用)的再次表现:

```toml
[package]
name = "my-binary"
version = "0.1.0"

[dependencies]
my-library = { git = 'https://example.com/git/my-library' }
uuid = "1.0"

[patch.crates-io]
uuid = { git = 'https://github.com/uuid-rs/uuid', branch = '2.0.0' }
```

请注意，这实际上，将解析为两个版本的`uuid`箱。该`my-binary`箱子将继续使用 1.x.y 系列的`uuid`箱子，但是`my-library`箱 会使用 2.0.0 版本`uuid`。这将允许您通过依赖关系图逐步推出对包的更改，而无需一次性更新所有内容。

### Using `[patch]` with multiple versions

你能使用同一个箱子的不同版本。是的， `package` 能帮它们重命名。例如， `serde` 的`1.*` 有个问题修复，而`2.0.0`则是，使用它的原型。那么，我们的配置如下：

```toml
[patch.crates-io]
serde = { git = 'https://github.com/serde-rs/serde' }
serde2 = { git = 'https://github.com/example/serde', package = 'serde', branch = 'v2' }
```

The first `serde = ...` 指明从 git 存储库中，拿到 `1.*`版本 (修复版) and the second `serde2 = ...` 指明从 git 存储库的 `v2` 分支中， 拿到`serde` package。我们假设，
这个分支上的 `Cargo.toml` 是为 `2.0.0` 的。

注意： `serde2`上的`package` 其实是忽视的。只是便于我们好区分。

### The `[patch]` section

`Cargo.toml`的`[patch]` 覆盖依赖。语法上，与
[`[dependencies]`][dependencies] 接近:

```toml
[patch.crates-io]
foo = { git = 'https://github.com/example/foo' }
bar = { path = 'my/local/bar' }

[dependencies.baz]
git = 'https://github.com/example/baz'

[patch.'https://github.com/example/baz']
baz = { git = 'https://github.com/example/patched-baz', branch = 'my-branch' }
```

The `[patch]` 由类似 dependencies 的子表格组成。
`[patch]`每个 key 之后，跟着 修补的 URL, 或是存储库的名字。 其中 `crates-io` 会用来覆盖覆盖默认的
[crates.io]。 第一个 `[patch]` 就展示了覆盖
[crates.io], 而第二个 `[patch]` 展示了覆盖 git。

在这些表格的格式规范，就与`[dependencies]`的类似。上面的示例，会对修改 `crates-io` 的来源 (e.g.
crates.io itself) ，也就是 `foo` 和 `bar` 。 还会修改 `https://github.com/example/baz` 来源，将其变为 `my-branch`

来源，可以指向不存在的版本, 自然也指向存在的版本。如果，来源是指向自己的某个版本，就会被替换。

### The `[replace]` section

> **Note**: `[replace]` is deprecated. You should use the
> [`[patch]`](#the-patch-section) table instead.

This section of Cargo.toml can be used to override dependencies with other
copies. The syntax is similar to the `[dependencies]` section:

```toml
[replace]
"foo:0.1.0" = { git = 'https://github.com/example/foo' }
"bar:1.0.2" = { path = 'my/local/bar' }
```

`[replace]` 下的每个 key 都是 [package ID
规范的](pkgid-spec.zh.md)，可以任意替换 (版本是写的)。也与 `[dependencies]`的定义类似。除了，你还可以不指定 feature。注意，名称与版本都需要，才能覆盖 (e.g., git or a local path).

### `paths` overrides

> 覆盖 本地依赖项

有时你只是暂时在一个箱子上工作，而你不想修改`Cargo.toml`中像上诉的`[patch]`部分。对于这个用例，Cargo 提供了更为有限的覆盖版本**路径覆盖**.

路径覆盖是通过`.cargo/config`指定，而不是`Cargo.toml`，你可以寻找[有关此配置的更多文档][config-docs]。在`.cargo/config`内，你要指定的是一个名为`paths`字段:

[config-docs]: ./config.zh.md

```toml
paths = ["/path/to/uuid"]
```

该数组应填充包含`Cargo.toml`的目录。在这种情况下，我们只是添加`uuid`，所以它将是唯一一个被覆盖的。此路径可以是包含该路径的绝对路径或相对`.cargo`文件夹的路径.

路径覆盖，比`[patch]`部分的限制更严格，但是，路径覆盖不能改变依赖图的结构。而当使用路径替换时，前一组依赖项必须完全匹配新的`Cargo.toml`规格。如此，就意味着路径覆盖不能用于向箱添加依赖项的测试，而换成`[patch]`在该种情况下使用。因此，路径覆盖的使用，通常会与快速 bug fix 分隔开来，而不是大更新分开。

注意:使用本地配置覆盖路径，仅适用于已发布到[crates.io]的包。您无法使用此功能告诉 Cargo 如何查找本地未发布的箱。

[crates.io]: https://crates.io/
[multiple locations]: specifying-dependencies.zh.md#multiple-locations
[dependencies]: specifying-dependencies.zh.md
