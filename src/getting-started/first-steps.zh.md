## Cargo 的第一步

本章节提供，cargo 命令的一些常用场景。如：

1. 生成一个新的[**_package_**][def-package]
2. 在 package 内，编译 [**_crate_**][def-crate]
3. 以及，运行程序

要使用 Cargo 启动新项目，请使用`cargo new`:

```shell
$ cargo new hello_world
```

默认我们会传递`--bin`参数，表明我们正在制作一个二进制程序: 如果我们正在创建一个库(lib)，我们就会把传递`--lib`参数。

让我们来看看 Cargo 为我们带来了什么:

```shell
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

这就是我们开始所需要的一切。首先，让我们看看`Cargo.toml`:

```toml
[package]
name = "hello_world"
version = "0.1.0"
edition = "2021"

[dependencies]
```

这被称为，**manifest**元清单，它包含了 Cargo 编译项目所需的所有元数据.

那`src/main.rs`有啥:

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 为我们创造了一个"hello_world"，也就是一个[**_binary crate_**][def-crate]。让我们来编译它:

```shell
$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/project/hello_world)
```

然后，运行它:

```shell
$ ./target/debug/hello_world
Hello, world!
```

我们也可以直接使用`cargo run`，它会自行编译，然后运行它， 一步到位:

```shell
$ cargo run
     Fresh hello_world v0.1.0 (file:///path/to/project/hello_world)
   Running `target/hello_world`
Hello, world!
```

## 走得更远

有关使用 Cargo 的更多详细信息，请查看[Cargo 指南](../guide/index.zh.md)

[def-crate]: ../appendix/glossary.zh.md#crate '"crate" (glossary entry)'
[def-manifest]: ../appendix/glossary.zh.md#manifest '"manifest" (glossary entry)'
[def-package]: ../appendix/glossary.zh.md#package '"package" (glossary entry)'
