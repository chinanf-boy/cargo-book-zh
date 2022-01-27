## Workspaces

A.*工作空间*是一个或多个包的集合，这些包共享公共依赖项解析（具有共享的`Cargo.lock`)，输出目录，以及各种设置，如配置文件。属于工作区的包称为*工作区成员*.工作区有两种风格：根包或虚拟清单。

### Root package

可以通过添加[`[workspace]`
section](#the-workspace-section)到`Cargo.toml`。这可以添加到`Cargo.toml`这已经定义了一个`[package]`，在这种情况下，包是*根包*工作空间的一部分。这个*工作区根目录*是工作区所在的目录`Cargo.toml`位于。

### Virtual manifest

或者，一个`Cargo.toml`可以使用`[workspace]`部分，但没有[`[package]` section][package].这叫做*虚拟舱单*。如果没有“主”包，或者希望将所有包组织在单独的目录中，这通常很有用。

### Key features

工作区的关键点是：

-   所有包共享一个公共`Cargo.lock`位于*工作区根目录*.
-   所有包共享一个公共[output directory]，默认为名为`target`在*工作区根目录*.
-   这个[`[patch]`][patch], [`[replace]`][replace]和[`[profile.*]`][profiles]章节`Cargo.toml`只有在*根*而在成员板条箱的清单中被忽略。

### The `[workspace]` section

这个`[workspace]`入座`Cargo.toml`定义哪些包是工作区的成员：

```toml
[workspace]
members = ["member1", "path/to/member2", "crates/*"]
exclude = ["crates/foo", "path/to/other"]
```

全部的[`path` dependencies]驻留在工作区目录中的用户将自动成为成员。其他成员可以与`members`键，它应该是一个字符串数组，其中包含`Cargo.toml`文件夹。

这个`members`列表还支持[globs]要匹配多个路径，请使用典型的文件名glob模式，如`*`和`?`.

这个`exclude`键可用于防止路径包含在工作区中。如果一些路径依赖项根本不希望出现在工作区中，或者使用glob模式，并且希望删除目录，那么这将非常有用。

空的`[workspace]`表可以与`[package]`使用包及其所有路径依赖关系方便地创建工作区。

### Workspace selection

在工作区的子目录中时，Cargo会自动在父目录中搜索`Cargo.toml`向`[workspace]`定义以确定要使用的工作空间。这个[`package.workspace`]可以在成员板条箱中使用清单键指向工作区的根，以覆盖此自动搜索。如果成员不在工作区根目录的子目录中，手动设置可能会很有用。

### Package selection

在工作区中，与包相关的Cargo命令[`cargo build`]可以使用`-p` / `--package`或`--workspace`命令行标志，用于确定要对哪些包进行操作。如果未指定这两个标志，Cargo将使用当前工作目录中的包。如果当前目录是虚拟工作区，它将应用于所有成员（就像`--workspace`已在命令行上指定）。

可选的`default-members`可以指定键来设置在工作区根目录中且未使用包选择标志时要操作的成员：

```toml
[workspace]
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]
default-members = ["path/to/member2", "path/to/member3/foo"]
```

如有规定，`default-members`必须扩展到`members`.

### The `workspace.metadata` table

这个`workspace.metadata`表被Cargo忽略，不会被警告。本节可用于希望在中存储工作区配置的工具`Cargo.toml`.例如：

```toml
[workspace]
members = ["member1", "member2"]

[workspace.metadata.webcontents]
root = "path/to/webproject"
tool = ["npm", "run", "build"]
# ...
```

在软件包级别，有一组类似的表[`package.metadata`][package-metadata]。虽然cargo没有为这两个表格的内容指定格式，但建议外部工具可能希望以一致的方式使用它们，例如参考表中的数据`workspace.metadata`如果数据丢失`package.metadata`，如果这对所讨论的工具有意义。

[package]: manifest.md#the-package-section

[package-metadata]: manifest.md#the-metadata-table

[output directory]: ../guide/build-cache.md

[patch]: overriding-dependencies.md#the-patch-section

[replace]: overriding-dependencies.md#the-replace-section

[profiles]: profiles.md

[`path` dependencies]: specifying-dependencies.md#specifying-path-dependencies

[`package.workspace`]: manifest.md#the-workspace-field

[globs]: https://docs.rs/glob/0.3.0/glob/struct.Pattern.html

[`cargo build`]: ../commands/cargo-build.md
