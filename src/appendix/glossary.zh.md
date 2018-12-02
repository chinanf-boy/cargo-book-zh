# Glossary

### Artifact

一个*神器*是由编译过程创建的文件或文件集.这包括可链接库和可执行二进制文件.

### Crate

包中的每个目标都是*箱*.Crates是库或可执行二进制文件.它可能松散地引用目标的源代码或目标生成的编译工件.包也可以指从注册表中提取的压缩包.

### Edition

一个*锈版*是Rust语言的发展里程碑.该[一个包的版本][edition-field]在.中指定`Cargo.toml`清单,各个目标可以指定它们使用的版本.见[版指南]欲获得更多信息.

### Feature

一个[*特征*][feature]是一个允许条件编译的命名标志.功能可以引用可选的依赖项,或者在a中定义的任意名称`Cargo.toml`可以在源代码中检查的清单.

Cargo有[*不稳定的特征标志*][cargo-unstable]这可以用来实现Cargo本身的实验行为.Rust编译器和Rustdoc也有自己的不稳定特征标志(参见[不稳定的书][unstable-book]和[Rustdoc书][rustdoc-unstable]).

### Index

索引是注册表中可搜索的包的列表.

### Lock file

该`Cargo.lock` *锁定文件*是一个文件,用于捕获工作空间或包中使用的每个依赖项的确切版本.它由Cargo自动生成.看到[Cargo.toml与Cargo.lock].

### Manifest

一个[*表现*][manifest]是对名为的文件中的包或工作空间的描述`Cargo.toml`.

一个[*虚拟清单*][virtual]是一个`Cargo.toml`仅描述工作空间的文件,不包含包.

### Member

一个*会员*是属于工作空间的包.

### Package

一个*包*是源文件的集合和`Cargo.toml`描述包的清单.包具有名称和版本,用于指定包之间的依赖关系.包中包含多个目标,这些目标是库或可执行二进制文件.

该*包根*是包的目录`Cargo.toml`清单位于.

该[*包ID规范*][pkgid-spec], 要么*SPEC*,是一个字符串,用于唯一地从特定源引用特定版本的包.

### Project

另一个名字[包](#package).

### Registry

一个*注册处*是一种服务,包含可下载的包,可以安装或用作包的依赖项.默认注册表是[crates.io](https://crates.io).注册表有一个*指数*其中包含所有包装箱的清单,并告诉Cargo如何下载所需的包装箱.

### Source

一个*资源*是一个包含可能作为包的依赖项包含的包的提供程序.有几种来源:

-   **注册表来源**- 见[注册处](#registry).
-   **本地注册表源**- 在文件系统上存储为压缩文件的一组包.看到[本地注册表来源].
-   **目录来源**- 在文件系统上存储为未压缩文件的一组包.看到[目录来源].
-   **路径来源**- 位于文件系统上的单个包(例如[路径依赖])或一组多个包(如[路径覆盖]).
-   **Git来源**- 位于git存储库中的包(例如[git依赖]要么[git来源]).

看到[来源更换]欲获得更多信息.

### Spec

看到[包ID规范](#package).

### Target

这个词的意思*目标*取决于具体情况:

-   **Cargo目标**- Cargo包裹包括*目标*对应于将要生成的工件.包可以包含库,二进制,示例,测试和基准目标.该[目标清单][targets]配置在`Cargo.toml`清单,通常由自动推断[目录布局]的源文件.
-   **目标架构**- 构建工件的OS和机器体系结构通常称为a*目标*.
-   **目标三重**- 三元组是用于指定目标体系结构的特定格式.见[clang文档]详情.三元组可以称为a*目标三联*这是产生的工件的架构,以及*主持三联*这是编译器运行的体系结构.可以使用指定目标三元组`--target`命令行选项或`build.target` [配置选项].
-   **目标目录**-  Cargo将所有构建的工件和中间文件放入*目标*目录.默认情况下,这是一个名为的目录`target`在工作区根目录,或者包根(如果不使用工作空间).目录随着改变而改变`--target-dir`命令行选项,`CARGO_TARGET_DIR` [环境变量], 或者`build.target-dir`
    [配置选项].

### Test Targets

Cargo*测试目标*生成二进制文件,帮助验证代码的正确操作和正确性.有两种类型的测试工件:

-   **单元测试**- 一个*单元测试*是直接从库或二进制目标编译的可执行二进制文件.它包含库或二进制代码的全部内容,并运行`#[test]`注释函数,用于验证各个代码单元.
-   **集成测试目标**- 安[*集成测试目标*][integration-tests]是一个可执行的二进制文件*测试目标*这是一个独特的箱子,其来源位于`tests`目录或由.指定[`[[test]]`表][targets]在里面`Cargo.toml`表现.它旨在仅测试库的公共API,或执行二进制文件以验证其操作.

### Workspace

一个[*工作区*][workspace]是一个共享公共依赖项解析的一个或多个包的集合(具有共享`Cargo.lock`),输出目录和各种设置,如配置文件.

一个[*虚拟工作区*][virtual]是根目录的工作空间`Cargo.toml`manifest没有定义包,只列出工作区成员.

该*工作区根*是工作区的目录`Cargo.toml`清单位于.

[cargo.toml vs cargo.lock]: guide/cargo-toml-vs-cargo-lock.md

[directory sources]: reference/source-replacement.md#directory-sources

[local registry sources]: reference/source-replacement.md#local-registry-sources

[source replacement]: reference/source-replacement.md

[cargo-unstable]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.md

[clang documentation]: http://clang.llvm.org/docs/CrossCompilation.md#target-triple

[config option]: reference/config.md

[directory layout]: reference/manifest.md#the-project-layout

[edition guide]: https://rust-lang-nursery.github.io/edition-guide/

[edition-field]: reference/manifest.md#the-edition-field-optional

[environment variable]: reference/environment-variables.md

[feature]: reference/manifest.md#the-features-section

[git dependency]: reference/specifying-dependencies.md#specifying-dependencies-from-git-repositories

[git source]: reference/source-replacement.md

[integration-tests]: reference/manifest.md#integration-tests

[manifest]: reference/manifest.md

[path dependency]: reference/specifying-dependencies.md#specifying-path-dependencies

[path overrides]: reference/specifying-dependencies.md#overriding-with-local-dependencies

[pkgid-spec]: reference/pkgid-spec.md

[rustdoc-unstable]: https://doc.rust-lang.org/nightly/rustdoc/unstable-features.md

[targets]: reference/manifest.md#configuring-a-target

[unstable-book]: https://doc.rust-lang.org/nightly/unstable-book/index.md

[virtual]: reference/manifest.md#virtual-manifest

[workspace]: reference/manifest.md#the-workspace-section
