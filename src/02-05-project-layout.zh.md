
## 项目布局

Cargo使用文件放置惯例,以便轻松进入新的货物项目:

```shell
.
├── Cargo.lock
├── Cargo.toml
├── benches
│   └── large-input.rs
├── examples
│   └── simple.rs
├── src
│   ├── bin
│   │   └── another_executable.rs
│   ├── lib.rs
│   └── main.rs
└── tests
    └── some-integration-tests.rs
```

-   `Cargo.toml`和`Cargo.lock`存储在项目的根目录中.
-   源代码进入`src`目录.
-   默认库文件是`src/lib.rs`.
-   默认的可执行文件是`src/main.rs`.
-   其他可执行文件可以放入`src/bin/*.rs`.
-   集成测试进入`tests`目录(单元测试进入他们正在测试的每个文件中).
-   示例进入`examples`目录.
-   基准进入`benches`目录.

这些将在更详细的说明中解释[清单描述](03-02-manifest.html#the-project-layout).

## Cargo.toml与Cargo.lock

`Cargo.toml`和`Cargo.lock`有两个不同的目的.在我们谈论它们之前,这是一个总结:

-   `Cargo.toml`是从广义上描述你的依赖,并由你编写.
-   `Cargo.lock`包含有关您的依赖项的确切信息.它由Cargo维护,不应手动编辑.

如果您正在构建其他项目将依赖的库,请将其放置`Cargo.lock`在你的`.gitignore`.如果您正在构建可执行文件,如命令行工具或应用程序,请检查`Cargo.lock`成`git`.如果你对这是为什么感到好奇,请参阅["为什么二进制文件有`Cargo.lock`在版本控制中,但不是库?"在FAQ中](faq.html#why-do-binaries-have-cargolock-in-version-control-but-not-libraries).

让我们再挖掘一下.

`Cargo.toml`是一个**表现**我们可以在其中指定一系列关于我们项目的不同元数据的文件.例如,我们可以说我们依赖于另一个项目:

```toml
[package]
name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
rand = { git = "https://github.com/rust-lang-nursery/rand.git" }
```

这个项目有一个依赖关系`rand`图书馆.在这种情况下,我们已经说过,我们依赖于GitHub上的特定Git存储库.由于我们尚未指定任何其他信息,因此Cargo假定我们打算使用最新的提交`master`分支建立我们的项目.

听起来不错?嗯,有一个问题:如果你今天建立这个项目,然后你发送一份副本给我,我明天建立这个项目,可能会发生一些不好的事情.可能会有更多的提交`rand`在此期间,我的构建将包括新的提交,而你的不会.因此,我们会得到不同的构建.这很糟糕,因为我们需要可重复的构建.

我们可以通过放置一个来解决这个问题`rev`在我们的行`Cargo.toml`:

```toml
[dependencies]
rand = { git = "https://github.com/rust-lang-nursery/rand.git", rev = "9f35b8e" }
```

现在我们的构建将是相同的.但是有一个很大的缺点:现在我们每次想要更新库时都必须手动考虑SHA-1.这既乏味又容易出错.

输入`Cargo.lock`.由于它的存在,我们不需要手动跟踪确切的修订:货物将为我们做.当我们有这样的清单时:

```toml
[package]
name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
rand = { git = "https://github.com/rust-lang-nursery/rand.git" }
```

货物将采取最新的承诺,并将这些信息写入我们的`Cargo.lock`当我们第一次建造时.该文件将如下所示:

```toml
[root]
name = "hello_world"
version = "0.1.0"
dependencies = [
 "rand 0.1.0 (git+https://github.com/rust-lang-nursery/rand.git#9f35b8e439eeedd60b9414c58f389bdc6a3284f9)",
]

[[package]]
name = "rand"
version = "0.1.0"
source = "git+https://github.com/rust-lang-nursery/rand.git#9f35b8e439eeedd60b9414c58f389bdc6a3284f9"
```

你可以看到这里有更多的信息,包括我们用来构建的确切修订.现在,当您将项目交给其他人时,他们将使用完全相同的SHA,即使我们没有在我们的项目中指定它`Cargo.toml`.

当我们准备选择加入新版本的库时,Cargo可以重新计算依赖关系并为我们更新内容:

```shell
$ cargo update           # updates all dependencies
$ cargo update -p rand   # updates just “rand”
```

这将写出一个新的`Cargo.lock`与新版本信息.请注意参数`cargo update`实际上是一个[包ID规范](03-07-pkgid-spec.html)和`rand`只是一个简短的规范.
