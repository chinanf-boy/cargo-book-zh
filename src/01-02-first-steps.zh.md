
## 货物的第一步

要使用Cargo启动新项目,请使用`cargo new`:

```shell
$ cargo new hello_world --bin
```

我们过世了`--bin`因为我们正在制作一个二进制程序:如果我们正在创建一个库,我们就会把它关掉.

让我们来看看Cargo为我们带来了什么:

```shell
$ cd hello_world
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

这就是我们开始所需要的一切.首先,让我们看看`Cargo.toml`:

```toml
[package]
name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
```

这被称为a**表现**,它包含了Cargo编译项目所需的所有元数据.

这是什么`src/main.rs`:

```
fn main() {
    println!("Hello, world!");
}
```

货物为我们创造了一个"你好世界".我们来编译它:

```shell
$ cargo build
   Compiling hello_world v0.1.0 (file:///path/to/project/hello_world)
```

然后运行它:

```shell
$ ./target/debug/hello_world
Hello, world!
```

我们也可以使用`cargo run`编译然后运行它,一步一步:

```shell
$ cargo run
     Fresh hello_world v0.1.0 (file:///path/to/project/hello_world)
   Running `target/hello_world`
Hello, world!
```

## 走得更远

有关使用货物的更多详细信息,请查看[货物指南](guide.html)
