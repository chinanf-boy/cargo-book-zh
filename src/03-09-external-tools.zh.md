
## 外部工具

Cargo的目标之一是与第三方工具(如IDE和其他构建系统)的简单集成.为了简化集成,Cargo有几个设施:

-   `cargo metadata`命令,以JSON格式输出项目结构和依赖项信息,

-   `--message-format`flag,输出有关特定构建的信息,

-   支持自定义子命令.

### 有关项目结构的信息

您可以使用`cargo metadata`命令获取有关项目结构和依赖项的信息.命令的输出如下所示:

```text
{
  // Integer version number of the format.
  "version": integer,

  // List of packages for this workspace, including dependencies.
  "packages": [
    {
      // Opaque package identifier.
      "id": PackageId,

      "name": string,

      "version": string,

      "source": SourceId,

      // A list of declared dependencies, see `resolve` field for actual dependencies.
      "dependencies": [ Dependency ],

      "targets: [ Target ],

      // Path to Cargo.toml
      "manifest_path": string,
    }
  ],

  "workspace_members": [ PackageId ],

  // Dependencies graph.
  "resolve": {
     "nodes": [
       {
         "id": PackageId,
         "dependencies": [ PackageId ]
       }
     ]
  }
}
```

格式稳定且版本.打电话时`cargo metadata`你应该通过`--format-version`明确标记以避免前向不兼容的危险.

如果你正在使用Rust,那就有了[cargo_metadata]箱.

[cargo_metadata]: https://crates.io/crates/cargo_metadata

### 有关构建的信息

路过时`--message=format=json`,Cargo将在构建期间输出以下信息:

-   编译器错误和警告,

-   制作的文物,

-   构建脚本的结果(例如,本机依赖项).

输出以每行格式的JSON对象转到stdout.该`reason`字段区分不同类型的消息.

有关Makefile兼容格式的依赖关系的信息存储在`.d`文件旁边的文件.

### 自定义子命令.

Cargo设计为可以使用新的子命令进行扩展,而无需修改Cargo本身.这是通过翻译货物的货物调用来实现的`(?<command>[^ ]+)`调用外部工具`cargo-${command}`然后需要出现在其中一个用户中`$PATH`目录.

自定义子命令可能会使用`CARGO`环境变量回叫货物.或者,它可以链接到`cargo`作为一个库,但这种方法有缺点:

-   货物作为库是不稳定的,API更改没有弃用,

-   Cargo库和Cargo二进制文件的版本可能不同.
