## Source Replacement

本文档是关于更换板条箱索引.您可以阅读有关重写依赖项的信息[重写依赖关系][overriding]本文档的一部分.

Cargo支持的能力**用另一个来源替换一个来源**根据镜子或销售依赖关系来表达策略.配置目前通过[`.cargo/config`组态][config]机制,像这样:

[config]: reference/config.md

```toml
# The `source` table is where all keys related to source-replacement
# are stored.
[source]

# Under the `source` table are a number of other tables whose keys are a
# name for the relevant source. For example this section defines a new
# source, called `my-awesome-source`, which comes from a directory
# located at `vendor` relative to the directory containing this `.cargo/config`
# file
[source.my-awesome-source]
directory = "vendor"

# Git sources can optionally specify a branch/tag/rev as well
git = "https://example.com/path/to/repo"
# branch = "master"
# tag = "v1.0.1"
# rev = "313f44e8"

# The crates.io default source for crates is available under the name
# "crates-io", and here we use the `replace-with` key to indicate that it's
# replaced with our source above.
[source.crates-io]
replace-with = "my-awesome-source"
```

使用此配置,Cargo会尝试在"vendor"目录中查找所有包,而不是在crates.io中查询在线注册表.使用货源更换Cargo可以表达:

-   供应 - 可以定义自定义源,它们表示本地文件系统上的包.这些源是它们正在替换的源的子集,并且如果需要可以检入包中.

-   镜像 - 源可以替换为等效版本,该版本充当crates.io本身的缓存.

Cargo有一个关于源替换的核心假设,源代码从两个源完全相同.在上面的例子中,Cargo假设所有的箱子都来自`my-awesome-source`与副本完全相同`crates-io`.请注意,这也意味着`my-awesome-source`不允许有不存在的板条箱`crates-io`资源.

因此,源替换不适用于修补依赖项或私有注册表等情况.Cargo支持通过使用修补依赖项[该`[replace]`键][replace-section],计划为未来版本的Cargo提供私人注册支持.

[replace-section]: reference/manifest.md#the-replace-section

[overriding]: reference/specifying-dependencies.md#overriding-dependencies

### Configuration

替换源的配置通过完成[`.cargo/config`][config]并且全套可用密钥是:

```toml
# Each source has its own table where the key is the name of the source
[source.the-source-name]

# Indicate that `the-source-name` will be replaced with `another-source`,
# defined elsewhere
replace-with = "another-source"

# Available kinds of sources that can be specified (described below)
registry = "https://example.com/path/to/index"
local-registry = "path/to/registry"
directory = "path/to/vendor"
```

该`crates-io`代表crates.io在线注册表(包装箱的默认来源),可以替换为:

```toml
[source.crates-io]
replace-with = 'another-source'
```

### Registry Sources

"注册表源"与crates.io本身相同.也就是说,它有一个在git存储库中提供的索引,该存储库匹配的格式[crates.io指数](https://github.com/rust-lang/crates.io-index).然后该存储库具有指示从哪里下载包的配置.

目前还没有一个已经可用的项目来设置crates.io的镜像.请继续关注!

### Local Registry Sources

"本地注册表源"旨在成为另一个注册表源的子集,但可在本地文件系统(也称为vendoring)上使用.提前下载本地注册表,通常与a同步`Cargo.lock`,并由一组`*.crate`文件和像普通注册表一样的索引.

管理和创建本地注册表源的主要方法是通过[`cargo-local-registry`][cargo-local-registry]子命令,可在crates.io上找到并可以安装`cargo install cargo-local-registry`.

[cargo-local-registry]: https://crates.io/crates/cargo-local-registry

本地注册表包含在一个目录中并包含许多`*.crate`从crates.io下载的文件以及`index`与crates.io-index项目格式相同的目录(仅填充有条件的条目).

### Directory Sources

"目录源"类似于本地注册表源,其中包含本地文件系统上可用的许多包,适用于销售依赖项.与本地注册表一样,目录源可以主要由外部子命令管理,[`cargo-vendor`][cargo-vendor],可以安装`cargo install cargo-vendor`.

[cargo-vendor]: https://crates.io/crates/cargo-vendor

目录源与本地注册表不同,但它们包含解压缩版本`*.crate`文件,使其更适合在某些情况下检查所有内容到源代码控制.目录源只是一个包含许多其他目录的目录,其中包含crates的源代码(解压缩版本的`*.crate`文件).目前,对每个目录的名称没有限制.

目录源中的每个包也有一个关联的元数据文件,指示包中每个文件的校验和,以防止意外修改.
