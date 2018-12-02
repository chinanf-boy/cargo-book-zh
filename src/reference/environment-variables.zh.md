## Environment Variables

Cargo设置并读取许多环境变量,代码可以检测或覆盖这些环境变量.以下是Cargo集与它们交互时组织的变量列表:

### Environment variables Cargo reads

您可以重写这些环境变量来更改Cargo在系统中的行为:

-   `CARGO_HOME`-Cargo在本地缓存注册表索引和箱子的git结账.默认情况下,这些存储在`$HOME/.cargo`但是这个变量重写了这个目录的位置.一旦机箱被缓存,它就不会被清除命令删除.
-   `CARGO_TARGET_DIR`-相对于当前工作目录,放置所有生成的工件的位置.
-   `RUSTC`代替跑步`rustc`Cargo将执行指定的编译器.
-   `RUSTC_WRAPPER`-而不是简单地运行`rustc`Cargo将执行这个指定的包装器,将rustc调用作为其命令行参数传递,第一个参数是rustc.
-   `RUSTDOC`代替跑步`rustdoc`Cargo将执行此规定`rustdoc`实例代替.
-   `RUSTDOCFLAGS`-空间分隔的自定义标志列表,以传递给所有`rustdoc`Cargo执行的调用.与之相反`cargo rustdoc`这对于传递一个标志是有用的.*全部的* `rustdoc`实例.
-   `RUSTFLAGS`-要传递给Cargo执行的所有编译器调用的自定义标记的空间分隔列表.与之相反`cargo rustc`这对于传递一个标志是有用的.*全部的*编译实例.
-   `CARGO_INCREMENTAL`-如果设置为1,则Cargo将强制在当前编译中启用增量编译,而当设置为0,则强制禁用增量编译.如果这个EV VAR不存在,否则将使用Cargo默认值.
-   `CARGO_CACHE_RUSTC_INFO`-如果这个设置为0,那么Cargo将不尝试缓存编译器版本信息.

注意,Cargo也会读取环境变量.`.cargo/config`配置值,如[那份文件][config-env]

[config-env]: reference/config.md#environment-variables

### Environment variables Cargo sets for crates

Cargo在编译时会将这些环境变量暴露在箱子中.请注意,这也适用于测试二进制文件.要在RIST程序中获得这些变量中的任何一个变量的值,请执行以下操作:

```rust
let version = env!("CARGO_PKG_VERSION");
```

`version`现在将包含的价值`CARGO_PKG_VERSION`.

-   `CARGO`的路径`cargo`二进制执行构建.
-   `CARGO_MANIFEST_DIR`-包含包的清单的目录.
-   `CARGO_PKG_VERSION`-您的包的完整版本.
-   `CARGO_PKG_VERSION_MAJOR`-你的软件包的主要版本.
-   `CARGO_PKG_VERSION_MINOR`-您的包的次要版本.
-   `CARGO_PKG_VERSION_PATCH`-包的补丁版本.
-   `CARGO_PKG_VERSION_PRE`-包的预发布版本.
-   `CARGO_PKG_AUTHORS`-冒号从程序包的清单中分离出作者列表.
-   `CARGO_PKG_NAME`-你的包裹的名字.
-   `CARGO_PKG_DESCRIPTION`-从包的清单中描述.
-   `CARGO_PKG_HOMEPAGE`-从包的清单中的主页.
-   `CARGO_PKG_REPOSITORY`-从包的清单中存储库.
-   `OUT_DIR`-如果包具有构建脚本,则将其设置为构建脚本应该在其中放置其输出的文件夹.更多信息见下文.

### Environment variables Cargo sets for build scripts

当运行构建脚本时,Cargo设置多个环境变量.因为编译生成脚本时还没有设置这些变量,所以上面的示例使用`env!`无法工作,而是在运行构建脚本时需要检索值:

```rust
use std::env;
let out_dir = env::var("OUT_DIR").unwrap();
```

`out_dir`现在将包含的价值`OUT_DIR`.

-   `CARGO`的路径`cargo`二进制执行构建.
-   `CARGO_MANIFEST_DIR`-包含正在构建的包的清单的目录(包含构建脚本的包).还要注意,这是生成脚本启动时当前工作目录的值.
-   `CARGO_MANIFEST_LINKS`-舱单`links`价值.
-   `CARGO_FEATURE_<name>`-对于正在构建的包的每个激活特性,此环境变量将显示在`<name>`上面的名称的名称是`-`翻译成`_`.
-   `CARGO_CFG_<cfg>`-为每个[配置选项][configuration]对于正在构建的包,此环境变量将包含配置的值,其中`<cfg>`配置的名称为上列并具有`-`翻译成`_`.                     如果设置了布尔配置,则存在布尔配置,否则不存在布尔配置.具有多个值的配置被连接到单个变量,该变量的值以`,`.
-   `OUT_DIR`-所有输出都应该放在其中的文件夹.此文件夹位于正在构建的包的构建目录中,并且对于所讨论的包是唯一的.
-   `TARGET`-正在编译的目标三元组.应该为这个三元组编译本机代码.关于目标三元组的更多信息可以在[克朗自己的文档][clang].
-   `HOST`-锈编译器的主机三元组.
-   `NUM_JOBS`-指定为顶层并行的并行性.这可以通过一个`-j`参数到系统`make`. 注意,在解释这个环境变量时应该小心.出于历史目的,仍然提供此功能,但是例如,最新版本的Cargo不需要运行`make
                 -j`因为它会自动发生.Cargo自行实现[JooServer]并且允许构建脚本继承这些信息,因此与GNU兼容的程序将使作业服务器已经具有适当配置的并行性.
-   `OPT_LEVEL`,`DEBUG`-当前正在构建的配置文件的相应变量的值.
-   `PROFILE` - `release`对于发布版本,`debug`用于其他构建.
-   `DEP_<name>_<key>`-有关此组环境变量的更多信息,请参阅[`links`][links].
-   `RUSTC`,`RUSTDOC`- Cargo已经决定使用的编译器和文档生成器,传递给构建脚本,以便它也可以使用它.
-   `RUSTC_LINKER`-指向链接器二进制文件的路径,如果指定,则Cargo已解析为将链接器二进制文件用于当前目标.链接器可以通过编辑更改.`.cargo/config`请参阅有关文档[Cargo配置][cargo-config]欲了解更多信息.

[links]: reference/build-scripts.md#the-links-manifest-key

[configuration]: https://doc.rust-lang.org/reference/attributes.md#conditional-compilation

[clang]: http://clang.llvm.org/docs/CrossCompilation.md#target-triple

[jobserver]: https://www.gnu.org/software/make/manual/html_node/Job-Slots.md

[cargo-config]: reference/config.md

### Environment variables Cargo sets for 3rd party subcommands

Cargo将这个环境变量公开给第三方子命令(即,名为`cargo-foobar`放置在`$PATH`):

-   `CARGO`的路径`cargo`二进制执行构建.
