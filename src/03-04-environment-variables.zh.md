## Environment Variables

货物集并读取代码可以检测或重写的多个环境变量.这里有一个变量清单,货物集合,当它与它们交互时组织起来:

### Environment variables Cargo reads

您可以重写这些环境变量来更改系统上的货物行为:

-   `CARGO_HOME`-货物保持注册表索引的本地缓存和板条箱的GIT签出.默认情况下,这些存储在`$HOME/.cargo`但是这个变量重写了这个目录的位置.一旦机箱被缓存,它就不会被清除命令删除.
-   `CARGO_TARGET_DIR`-放置所有生成工件的位置,相对于当前工作目录.
-   `RUSTC`代替跑步`rustc`货物将执行指定的编译器.
-   `RUSTC_WRAPPER`-而不是简单地运行`rustc`Cargo将执行这个指定的包装器,将rustc调用作为其命令行参数传递,第一个参数是rustc.
-   `RUSTDOC`代替跑步`rustdoc`货物将执行此规定`rustdoc`实例代替.
-   `RUSTFLAGS`-空间分隔的自定义标志列表,传递给货物执行的所有编译器调用.与之相反`cargo rustc`这对于传递一个标志是有用的.*全部的*编译实例.

注意,货物也会读取环境变量.`.cargo/config`配置值,如[那份文件][config-env]

[config-env]: 03-03-config.html#environment-variables

### Environment variables Cargo sets for crates

货物在编译时会将这些环境变量暴露在箱子中.请注意,这也适用于测试二进制文件.若要在RIST程序中获取这些变量中的任何值,可以使用`env!`宏:

```
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
-   `CARGO_PKG_DESCRIPTION`-描述你的包裹.
-   `CARGO_PKG_HOMEPAGE`-你的包裹的主页.
-   `OUT_DIR`-如果包有构建脚本,则将其设置为生成脚本应放置其输出的文件夹.更多信息见下文.

### Environment variables Cargo sets for build scripts

当运行构建脚本时,货物设置多个环境变量.因为编译生成脚本时还没有设置这些变量,上述示例使用`env!`将不起作用,相反,当生成脚本运行时,您需要检索值:

```
use std::env;
let out_dir = env::var("OUT_DIR").unwrap();
```

`out_dir`现在将包含的价值`OUT_DIR`.

-   `CARGO_MANIFEST_DIR`-包含正在构建的包的清单的目录(包含构建脚本的包).还要注意,这是生成脚本启动时当前工作目录的值.
-   `CARGO_MANIFEST_LINKS`-舱单`links`价值.
-   `CARGO_FEATURE_<name>`-对于正在构建的包的每个激活特性,此环境变量将显示在`<name>`上面的名称的名称是`-`翻译成`_`.
-   `CARGO_CFG_<cfg>`-为每个[配置选项][configuration]对于正在构建的包,此环境变量将包含配置的值,其中`<cfg>`配置的名称为上列并具有`-`翻译成`_`.                     如果设置布尔配置,则不存在.具有多个值的配置与单个变量连接,其中的值由`,`.
-   `OUT_DIR`-所有输出都应该放在其中的文件夹.此文件夹位于正在构建的包的构建目录中,并且对于所讨论的包是唯一的.
-   `TARGET`-正在编译的目标三元组.应该为这个三元组编译本机代码.关于目标三元组的更多信息可以在[克朗自己的文档][clang].
-   `HOST`-锈编译器的主机三元组.
-   `NUM_JOBS`-指定为顶层并行的并行性.这可以通过一个`-j`参数到系统`make`.
-   `OPT_LEVEL`,`DEBUG`-当前正在构建的配置文件的相应变量的值.
-   `PROFILE` - `release`对于发布版本,`debug`用于其他构建.
-   `DEP_<name>_<key>`有关此一组环境变量的更多信息,请参见[`links`][links].
-   `RUSTC`,`RUSTDOC`- Cargo已经决定使用的编译器和文档生成器,传递给构建脚本,以便它也可以使用它.

[links]: 03-05-build-scripts.html#the-links-manifest-key

[profile]: 03-02-manifest.html#the-profile-sections

[configuration]: https://doc.rust-lang.org/reference/attributes.html#conditional-compilation

[clang]: http://clang.llvm.org/docs/CrossCompilation.html#target-triple

### Environment variables Cargo sets for 3rd party subcommands

货物将此环境变量暴露为第三方子命令(即程序名).`cargo-foobar`放置在`$PATH`):

-   `CARGO`的路径`cargo`二进制执行构建.
