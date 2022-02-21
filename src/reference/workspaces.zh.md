## Workspaces

一个 _工作空间_ 是一个或多个包的集合，这些包共享，公共依赖项解析（具有共享的`Cargo.lock`），输出目录，以及各种设置，如配置文件。属于工作区的包称为*工作区成员*。工作区有两种风格：根包或虚拟清单。

### Root package

添加[`[workspace]` section](#the-workspace-section)到`Cargo.toml`，创建工作空间。若已经定义了一个`[package]`，在这种情况下，该包是*根包*工作空间的一部分。*工作区根目录*是这个`Cargo.toml`的所在。

### Virtual manifest

或者，一个`Cargo.toml`可以使用`[workspace]`部分，但没有[`[package]` section][package]。这叫做*虚拟清单*。这在你没有“主”包，或仅希望将所有包分开组织时，使用。

### Key features

工作区的关键点是：

- 所有包共享一个公共`Cargo.lock`，位于*工作区根目录*。
- 所有包共享一个公共[输出目录][output directory]，默认为`target`，位在*工作区根目录*。
- 这个[`[patch]`][patch], [`[replace]`][replace]和[`[profile.*]`][profiles]仅在*根* manifest 有效，而对成员的 manifest 忽略。

### The `[workspace]` section

这个`[workspace]`表格，定义哪些包是工作区的成员：

```toml
[workspace]
members = ["member1", "path/to/member2", "crates/*"]
exclude = ["crates/foo", "path/to/other"]
```

驻留在工作区目录中，全部的[`path`依赖项][`path` dependencies]将自动成为成员。其他成员可以在`members`列出，它应该是一个字符串数组，其字符串为包含`Cargo.toml`文件的目录。

这个`members`列表还支持[globs]，去匹配多个路径 —— 典型的文件名 glob 模式，如`*`和`?`。

这个`exclude`字段，排除定义的路径，让它们不能加入或移出工作区（可用 glob 模式）。

例如：给个空`[workspace]`表，配合上`[package]`，就能以 package 与它的所有 path 依赖项，方便地创建一个工作区。

### Workspace selection

在工作区的子目录中时，Cargo 会在父目录，自动搜索`Cargo.toml`的`[workspace]`定义，以确定要使用的工作空间。[`package.workspace`]可用在成员箱子中，指向工作区的根，以覆盖此自动搜索。如果成员不在工作区根目录的子目录中，是要手动设置的。

### Package selection

在工作区中，与包相关的 Cargo 命令，比如[`cargo build`]，它可以使用`-p` / `--package`或`--workspace`命令行标志，确定要对哪些包进行操作。如果未指定这两个标志，Cargo 将使用当前工作目录中的包。如果当前目录是虚拟工作区，它将应用于所有成员（就像`--workspace`已在命令行上指定）。

可选的`default-members`，设置在工作区根目录中，要操作的成员（命令行不指定的情况下）：

```toml
[workspace]
members = ["path/to/member1", "path/to/member2", "path/to/member3/*"]
default-members = ["path/to/member2", "path/to/member3/foo"]
```

如有规定，`default-members`必须扩展到`members`.

### The `workspace.metadata` table

这个`workspace.metadata`表被 Cargo 忽略，不会被警告。`Cargo.toml`中这个表格，是给那些要存储工作区配置的工具所使用的。例如：

```toml
[workspace]
members = ["member1", "member2"]

[workspace.metadata.webcontents]
root = "path/to/webproject"
tool = ["npm", "run", "build"]
# ...
```

在包级别，有一组类似的表[`package.metadata`][package-metadata]。虽然 cargo 没有为这两个表格的内容指定格式，但，建议外部工具可能希望以一致的方式使用它们，例子：如果`package.metadata`数据丢失，就可以参考`workspace.metadata`表中的数据，对工具可能用得上。

[package]: manifest.zh.md#the-package-section
[package-metadata]: manifest.zh.md#the-metadata-table
[output directory]: ../guide/build-cache.zh.md
[patch]: overriding-dependencies.zh.md#the-patch-section
[replace]: overriding-dependencies.zh.md#the-replace-section
[profiles]: profiles.zh.md
[`path` dependencies]: specifying-dependencies.zh.md#specifying-path-dependencies
[`package.workspace`]: manifest.zh.md#the-workspace-field
[globs]: https://docs.rs/glob/0.3.0/glob/struct.Pattern.html
[`cargo build`]: ../commands/cargo-build.zh.md
