
## 创建一个新项目

要使用Cargo启动新项目,请使用`cargo new`:

```shell
$ cargo new hello_world --bin
```

我们过世了`--bin`因为我们正在制作一个二进制程序:如果我们正在创建一个库,我们就会把它关掉.这也初始化了一个新的`git`存储库默认情况下如果您不希望它这样做,请通过`--vcs none`.

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

如果我们刚刚使用过`cargo new hello_world`没有`--bin`国旗,然后我们会有一个`lib.rs`代替`main.rs`.然而,就目前而言,这就是我们开始所需要的一切.首先,让我们看看`Cargo.toml`:

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

我们也可以使用`cargo run`编译然后运行它,一步到位(你不会看到`Compiling`如果您上次编译后没有进行任何更改,请执行以下操作:

```shell
$ cargo run
   Compiling hello_world v0.1.0 (file:///path/to/project/hello_world)
     Running `target/debug/hello_world`
Hello, world!
```

您会注意到已创建了几个新文件和目录:

```shell
$ tree .
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    └── debug
        ├── build
        ├── deps
        │   └── hello_world-2386c2fd0156916f
        ├── examples
        ├── hello_world
        ├── hello_world.d
        ├── incremental
        └── native

8 directories, 6 files
```

该`Cargo.lock`file包含有关我们的依赖项的信息.既然我们还没有,那就不是很有趣了.该`target`目录包含所有构建产品,并且,可以看出,Cargo默认生成调试版本.您可以使用`cargo build --release`在打开优化的情况下编译文件:

```shell
$ cargo build --release
   Compiling hello_world v0.1.0 (file:///path/to/project/hello_world)
```

`cargo build --release`将结果二进制文件放入`target/release`代替`target/debug`.

在调试模式下编译是开发的默认设置 - 编译时间较短,因为编译器不进行优化,但代码运行速度较慢.发布模式编译需要更长时间,但代码运行速度更快.
