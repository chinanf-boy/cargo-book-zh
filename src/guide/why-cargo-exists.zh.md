## 为什么 Cargo 存在

### 前言

在 Rust, 正如你所了解的，一个库或执行程序被叫做 箱子。而箱子是通过 Rust 编译器`rustc`所编译的。当我们与 Rust 相遇，第一个源代码回事著名的 Hello World 程序，它可以直接用`rustc`编译。

```console
$ rustc hello.rs
$ ./hello
Hello, world!
```

注意到，上面的命令，是需要指明文件名的。如果，我们直接使用 `rustc`，那么，命令行的调用，就会夹杂着不同的命令。更别说，特殊(更详细)的命令行选项的指定，去包括外部依赖...

更重要的是，当项目具有各种外部依赖，你要管理它们的版本，确保正确性与方便性。

为了可以做到人性化，不直接操作`rustc`，我们引入了更高的抽象工具 —— 包管理器

### 出场：Cargo

Cargo 就是一个 Rust 的包管理器。允许 Rust 项目声明其各种依赖项，并确保您始终获得可重复的构建。

为了实现这一目标,Cargo 做了四件事:

- 引入两个，包含各种项目信息的元数据文件。
- 获取，并构建项目的依赖项.
- 正确使用参数，以调用`rustc`或其他构建工具，构建你的项目。
- 介绍，更容易使用 Rust 项目的约定(规范/风格)。

往大的说，Cargo 给构建一个程序，一个库，带来了一种标准化的方式，不去管项目的名字，更不需要直接调用 `rustc`，我们可以更常规地调用`cargo build`，然后，让 cargo 处理正确的`rustc`调用。Cargo 还会自动从[_registry_][def-registry]，获取我们的依赖项，安排得明明白白。

一语言之，有了 Cargo，你的 Rust 就有了垫脚石。

[def-artifact]: ../appendix/glossary.zh.md#artifact '"artifact" (glossary entry)'
[def-crate]: ../appendix/glossary.zh.md#crate '"crate" (glossary entry)'
[def-package]: ../appendix/glossary.zh.md#package '"package" (glossary entry)'
[def-package-manager]: ../appendix/glossary.zh.md#package-manager '"package manager" (glossary entry)'
[def-registry]: ../appendix/glossary.zh.md#registry '"registry" (glossary entry)'
