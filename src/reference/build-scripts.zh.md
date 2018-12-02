## Build Scripts

一些包需要编译第三方非Rust代码,例如C库.其他的包需要链接到C库,这些库既可以位于系统上,也可以从源代码构建.其他人仍然需要功能工具,比如构建之前的代码生成(想想解析器生成器).

Cargo并不打算替换为这些任务进行过良好优化的其他工具,但是它与`build`配置选项.

```toml
[package]
# ...
build = "build.rs"
```

指定的锈文件`build`命令(相对于包根)将在包中编译其他内容之前被编译和调用,从而允许Rust代码依赖于所构建或生成的工件.默认情况下Cargo抬头.`"build.rs"`在包根文件中(即使您没有指定一个值)`build`)使用`build = "custom_build_name.rs"`指定自定义生成名或`build = false`禁用对生成脚本的自动检测.

Bug命令的一些用例是:

-   建立一个捆绑的C库.
-   在主机系统上找到C库.
-   从规范中生成锈蚀模块.
-   执行机箱所需的任何平台特定配置.

下面将详细介绍这些用例中的每一个,以给出构建命令如何工作的示例.

### Inputs to the Build Script

当运行构建脚本时,存在许多到构建脚本的输入,所有输入都以[环境变量][env].

除了环境变量之外,构建脚本的当前目录是构建脚本包的源目录.

[env]: reference/environment-variables.md

### Outputs of the Build Script

由构建脚本打印到STDUT的所有行都被写入文件`target/debug/build/<pkg>/output`(精确的位置可能取决于你的配置).如果您希望直接在终端中看到这样的输出,那么使用`-vv`旗帜.注意,如果既不修改构建脚本也不修改包源文件,则随后调用`-vv`将**不**打印输出到终端,因为没有执行新的构建.跑`cargo clean`如果希望确保输出始终显示在终端上,则在每次Cargo调用之前.开始的任何一行`cargo:`直接由Cargo解释.这一行必须是形式.`cargo:key=value`就像下面的例子:

```
# specially recognized by Cargo
cargo:rustc-link-lib=static=foo
cargo:rustc-link-search=native=/path/to/foo
cargo:rustc-cfg=foo
cargo:rustc-env=FOO=bar
# arbitrary user-defined metadata
cargo:root=/path/to/foo
cargo:libdir=/path/to/foo/lib
cargo:include=/path/to/foo/include
```

另一方面,打印到STDRR的行被写入文件.`target/debug/build/<pkg>/stderr`但不被Cargo解释.

Cargo识别一些特殊的钥匙,其中一些影响板条箱的构造:

-   `rustc-link-lib=[KIND=]NAME`指示指定的值是库名,应该作为`-l`旗帜.可选的`KIND`可以是其中之一`static`,`dylib`(默认值),或`framework`见`rustc --help`更多细节.
-   `rustc-link-search=[KIND=]PATH`指示指定的值是库搜索路径,应该作为`-L`旗帜.可选的`KIND`可以是其中之一`dependency`,`crate`,`native`,`framework`或`all`(默认值),请参见`rustc --help`更多细节.
-   `rustc-flags=FLAGS`是只传递给编译器的一组标志`-l`和`-L`支持标志.
-   `rustc-cfg=FEATURE`指示指定的特性将作为`--cfg`标记到编译器.这对于执行各种特征的编译时间检测通常是有用的.
-   `rustc-env=VAR=VALUE`指示指定的环境变量将被添加到编译器所在的环境中.然后,可以检索该值.`env!`宏在编译的板条箱中.这对于在板条箱的代码中嵌入额外的元数据很有用,比如Git HEAD的散列或持续集成服务器的唯一标识符.
-   `rerun-if-changed=PATH`是文件或目录的路径,指示如果生成脚本发生更改(由文件上最近修改的时间戳检测到),则应重新运行生成脚本.通常,如果板条箱根目录中的任何文件发生更改,则重新运行构建脚本,但这可用于将更改范围扩展到仅一小组文件.(如果这个路径指向一个目录,则不会遍历整个目录以进行更改——只对目录本身的时间戳进行更改(该时间戳对应于目录中的某些类型的更改,取决于平台),将触发重新构建.要请求重新运行整个目录中的任何更改,请递归地为该目录打印一行,为该目录内的所有内容打印另一行.)请注意,如果构建脚本本身(或其依赖项之一)更改,则无条件地重新构建和重新运行该脚本,因此,`cargo:rerun-if-changed=build.rs`几乎总是多余的(除非您想要忽略除了`build.rs`)
-   `rerun-if-env-changed=VAR`是环境变量的名称,它指示如果环境变量的值发生变化,则应重新运行构建脚本.这基本上是一样的.`rerun-if-changed`除了它与环境变量一起工作.注意,这里的环境变量用于全局环境变量,如`CC`这样,对于EVV VARS,不必使用它.`TARGET`那套货.还要注意,如果`rerun-if-env-changed`打印出来,然后Cargo将*只有*如果那些环境变量发生变化或者文件打印出`rerun-if-changed`改变.

-   `warning=MESSAGE`是构建脚本运行完毕后将打印到主控制台的消息.警告只针对路径依赖项(即,您在本地工作的那些依赖项)显示,因此例如,在.es.io板条箱中打印的警告在默认情况下不会发出.

任何其他元素都是用户定义的元数据,这些元数据将传递给依赖者.关于这个的更多信息可以在[`links`][links]部分.

[links]: #the-links-manifest-key

### Build Dependencies

构建脚本也可以依赖于其他基于Cargo的板条箱.依赖关系通过`build-dependencies`清单的部分.

```toml
[build-dependencies]
foo = { git = "https://github.com/your-packages/foo" }
```

生成脚本**不**可以访问列表中的依赖项`dependencies`或`dev-dependencies`部分(它们还没有建成!)除非明确声明,否则包本身也不能使用所有构建依赖项.

### The `links` Manifest Key

除了清单密钥`build`Cargo也支持`links`声明要链接到的本地库的名称的清单密钥:

```toml
[package]
# ...
links = "foo"
build = "build.rs"
```

此清单说明包链接到`libfoo`本机库,并且它还具有用于定位和/或构建库的构建脚本.Cargo要求A`build`如果指定了`links`还指定了条目.

这个清单键的目的是让Cargo了解包所具有的本地依赖项集,并提供在包构建脚本之间传递元数据的原则系统.

首先,Cargo要求最多只有一个包裹.`links`价值.换句话说,禁止两个包链接到同一个本机库.然而,请注意[约定就位][star-sys]为了缓解这个问题.

[star-sys]: #a-sys-packages

如上面在输出格式中提到的,每个构建脚本可以以键-值对的形式生成一组任意的元数据.此元数据传递给**依赖的**包装.例如,如果`libbar`取决于`libfoo`如果`libfoo`生成`key=value`作为其元数据的一部分,则构建脚本`libbar`将有环境变量`DEP_FOO_KEY=value`.

注意,元数据只传递给直接依赖项,而不是传递依赖项.此元数据传递的动机将在以下链接到系统库案例研究中概述.

### Overriding Build Scripts

如果清单包含`links`关键字,则Cargo支持重写用自定义库指定的构建脚本.此功能的目的是防止完全运行有问题的构建脚本,而是提前提供元数据.

要覆盖构建脚本,请将下列配置放在任何可接受的Cargo中[配置位置](reference/config.md).

```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-search = ["/path/to/foo"]
rustc-link-lib = ["foo"]
root = "/path/to/foo"
key = "value"
```

本节说明目标`x86_64-unknown-linux-gnu`图书馆命名`foo`具有指定的元数据.此元数据与生成脚本时生成的元数据相同,提供了许多键/值对,其中`rustc-flags`,`rustc-link-search`和`rustc-link-lib`按键有点特殊.

使用此配置,如果包声明它链接到`foo`然后构建脚本将**不**编译或运行,将使用指定的元数据.

### Case study: Code generation

由于各种原因,一些Cargo包在编译之前需要生成代码.这里我们将介绍一个简单的示例,该示例生成库调用作为构建脚本的一部分.

首先,让我们看一下这个包的目录结构:

```
.
├── Cargo.toml
├── build.rs
└── src
    └── main.rs

1 directory, 3 files
```

在这里我们可以看到我们有一个`build.rs`构建脚本和二进制文件`main.rs`. 接下来,让我们看一下清单:

```toml
# Cargo.toml

[package]
name = "hello-from-generated-code"
version = "0.1.0"
authors = ["you@example.com"]
build = "build.rs"
```

在这里,我们可以看到,我们已经指定了一个构建脚本,我们将使用它来生成一些代码.让我们看看构建脚本里面有什么:

```rust,no_run
// build.rs

use std::env;
use std::fs::File;
use std::io::Write;
use std::path::Path;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("hello.rs");
    let mut f = File::create(&dest_path).unwrap();

    f.write_all(b"
        pub fn message() -> &'static str {
            \"Hello, World!\"
        }
    ").unwrap();
}
```

这里有两点值得注意的地方:

-   脚本使用`OUT_DIR`环境变量以发现输出文件应该位于何处.它可以使用进程的当前工作目录来查找输入文件应该位于哪里,但是在这种情况下,我们没有任何输入文件.
-   一般来说,构建脚本不应该修改外部的任何文件.`OUT_DIR`. 乍看之下,它似乎不错,但当您使用这种箱子作为依赖项时,它确实会带来问题,因为*隐性的*源中的不变量`.cargo/registry`应该是不变的.`cargo`在包装时不会允许这样的脚本.
-   这个脚本相对简单,只是写出一个小生成的文件.可以想象,其他更奇特的操作也可能发生,例如从C头文件或其他语言定义生成Rust模块.

接下来,我们来看看图书馆本身:

```rust,ignore
// src/main.rs

include!(concat!(env!("OUT_DIR"), "/hello.rs"));

fn main() {
    println!("{}", message());
}
```

这就是真正的魔法发生的地方.图书馆正在使用RUSTC定义`include!`宏结合`concat!`和`env!`宏包含生成的文件(`hello.rs`进入板条箱的编纂.

使用此处所示的结构,板条箱可以包括来自构建脚本本身的任何数量的生成文件.

### Case study: Building some native code

有时需要建立一些本地C或C++代码作为包的一部分.这是利用构建脚本在Rust板条箱本身之前构建本机库的另一个极好的用例.作为一个例子,我们将创建一个Rust库,它调用C来打印"Hello,World!".

和上面一样,让我们先来看看软件包的布局:

```
.
├── Cargo.toml
├── build.rs
└── src
    ├── hello.c
    └── main.rs

1 directory, 4 files
```

很像以前!下一步,清单:

```toml
# Cargo.toml

[package]
name = "hello-world-from-c"
version = "0.1.0"
authors = ["you@example.com"]
build = "build.rs"
```

现在,我们不打算使用任何构建依赖项,所以现在让我们看一下构建脚本:

```rust,no_run
// build.rs

use std::process::Command;
use std::env;
use std::path::Path;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();

    // note that there are a number of downsides to this approach, the comments
    // below detail how to improve the portability of these commands.
    Command::new("gcc").args(&["src/hello.c", "-c", "-fPIC", "-o"])
                       .arg(&format!("{}/hello.o", out_dir))
                       .status().unwrap();
    Command::new("ar").args(&["crus", "libhello.a", "hello.o"])
                      .current_dir(&Path::new(&out_dir))
                      .status().unwrap();

    println!("cargo:rustc-link-search=native={}", out_dir);
    println!("cargo:rustc-link-lib=static=hello");
}
```

此构建脚本首先将C文件编译为对象文件(通过调用`gcc`)然后将这个对象文件转换为静态库(通过调用`ar`)最后一步是反馈Cargo本身,以表示我们的产量在.`out_dir`编译器应该将机箱链接到`libhello.a`静态通过`-l static=hello`旗帜.

请注意,这种硬编码方法有许多缺点:

-   这个`gcc`命令本身不是跨平台可移植的.例如,Windows平台不太可能`gcc`,甚至不是所有UNIX平台都可能有`gcc`. 这个`ar`命令也处于类似的情况.
-   这些命令不考虑交叉编译.如果我们为Android这样的平台进行交叉编译,那就不太可能了.`gcc`将产生一个可执行的ARM.

但不要害怕,这是一个`build-dependencies`进入会有帮助的!Cargo生态系统有许多包,使得此类任务更加容易、可移植和标准化.例如,生成脚本可以写成:

```rust,ignore
// build.rs

// Bring in a dependency on an externally maintained `cc` package which manages
// invoking the C compiler.
extern crate cc;

fn main() {
    cc::Build::new()
        .file("src/hello.c")
        .compile("hello");
}
```

将生成时间依赖性添加到`cc`板条箱,以下添加到您的`Cargo.toml`:

```toml
[build-dependencies]
cc = "1.0"
```

这个[`cc`机箱](https://crates.io/crates/cc)抽象C代码的构建脚本需求范围:

-   它调用适当的编译器(Windows的MSVC),`gcc`对MinGW来说,`cc`对于UNIX平台,等等).
-   它需要`TARGET`通过向正在使用的编译器传递适当的标志来考虑变量.
-   其他环境变量,如`OPT_LEVEL`,`DEBUG`等等,都是自动处理的.
-   STDUT输出和`OUT_DIR`位置也由`cc`图书馆.

在这里,我们可以开始看到将尽可能多的功能移植到公共构建依赖项而不是在所有构建脚本之间复制逻辑的一些主要好处!

回到案例研究,让我们快速浏览一下内容.`src`目录:

```c
// src/hello.c

#include <stdio.h>

void hello() {
    printf("Hello, World!\n");
}
```

```rust,ignore
// src/main.rs

// Note the lack of the `#[link]` attribute. We’re delegating the responsibility
// of selecting what to link to over to the build script rather than hardcoding
// it in the source file.
extern { fn hello(); }

fn main() {
    unsafe { hello(); }
}
```

我们走了!这将完成使用构建脚本本身从Cargo包构建一些C代码的示例.这也说明了为什么在许多情况下使用构建依赖关系非常重要,甚至更加简洁!

我们还看到了构建脚本如何使用板条箱作为纯粹用于构建过程的依赖项,而不是在运行时用作板条箱本身的依赖项的简要示例.

### Case study: Linking to system libraries

这里的最后一个案例研究将研究Cargo库如何链接到系统库,以及构建脚本如何被利用以支持这个用例.

通常,Rust板条箱希望链接到系统上经常提供的本地库,以绑定其功能,或者只是将其用作实现细节的一部分.当以与平台无关的方式执行这个操作时,这是一个相当微妙的问题,构建脚本的目的再次是尽可能多地分配这些内容,以便让消费者尽可能容易地使用它.

作为一个例子,让我们来看一个[Cargo本身的依赖][git2-rs],[LBGIT2][libgit2]. C库有许多约束条件:

[git2-rs]: https://github.com/alexcrichton/git2-rs/tree/master/libgit2-sys

[libgit2]: https://github.com/libgit2/libgit2

-   它可选地依赖于Unix上的OpenSSL来实现https传输.
-   它可选地依赖于所有平台上的libssh2来实现ssh传输.
-   默认情况下,它通常不安装在所有系统上.
-   它可以从源代码使用`cmake`.

为了可视化这里发生的事情,让我们看一下链接到本机C库的相关Cargo包的清单.

```toml
[package]
name = "libgit2-sys"
version = "0.1.0"
authors = ["..."]
links = "git2"
build = "build.rs"

[dependencies]
libssh2-sys = { git = "https://github.com/alexcrichton/ssh2-rs" }

[target.'cfg(unix)'.dependencies]
openssl-sys = { git = "https://github.com/alexcrichton/openssl-sys" }

# ...
```

正如上面的清单所显示的,我们得到了一个`build`指定脚本,但值得注意的是,该示例具有`links`表示板条箱的条目(`libgit2-sys`链接到`git2`本地图书馆.

在这里,我们还看到,我们选择让锈箱有一个无条件的依赖`libssh2`通过`libssh2-sys`板条箱,以及特定于平台的依赖性`openssl-sys`对于\*NIX(其他变体现在被淘汰).这似乎有点违反直觉.*C依赖关系*在*Cargo舱单*但这实际上是在这个空间中使用一种Cargo的约定.

### `*-sys` Packages

为了减轻对系统库的链接,CRATES.IO有一个*惯例*包命名和功能性.任何包名`foo-sys`应该提供两个主要功能:

-   图书馆箱应链接到本地图书馆`libfoo`. 这将经常探测当前的系统.`libfoo`在从源头上诉诸建筑之前.
-   图书馆板条箱应提供**声明**函数在`libfoo`但是**不**绑定或高级抽象.

一套`*-sys`包提供了一组用于连接到本地库的公共依赖项.通过这种与本机库相关的包的约定,可以获得许多好处:

-   公共依赖性`foo-sys`减轻每一个值的一个包的上述规则`links`.
-   一个公共依赖关系允许集中逻辑发现`libfoo`本身(或者从源头构建它).
-   这些依赖关系很容易被重写.

### Building libgit2

现在我们已经整理了libgit2的依赖关系,我们需要实际编写构建脚本.我们这里不讨论特定的代码片段,而只研究构建脚本的高级细节`libgit2-sys`. 这并不是建议所有软件包都遵循这个策略,而是仅仅概述一个特定的策略.

构建脚本应该做的第一步是查询libgit2是否已经安装在主机系统上.要做到这一点,我们将利用现有的工具.`pkg-config`(当它可用时).我们也会使用`build-dependencies`部分重构所有`pkg-config`相关代码(或者有人已经这样做了!).

如果`pkg-config`找不到LIGBGIT2,或者如果`pkg-config`只是没有安装,下一步是从捆绑源代码(作为以下内容的一部分分发)构建libgit2`libgit2-sys`本身).然而,在这样做时有一些细微差别,我们需要加以考虑:

-   LBGIT2的构建系统,`cmake`需要能够找到LIGBGS2对LIPSH2的可选依赖性.我们确信我们已经建立了它(它是一个Cargo依赖项),我们只需要传递这个信息.为此,我们利用元数据格式在构建脚本之间传递信息.在这个例子中,打印出的LBSSH2包`cargo:root=...`来告诉我们libssh2安装在哪里,然后我们可以通过`CMAKE_PREFIX_PATH`环境变量.

-   我们需要处理一些`CFLAGS`编译C代码时的值`cmake`关于这个).我们想通过的一些旗帜是`-m64`对于64位代码,`-m32`对于32位代码,或`-fPIC`也适用于64位代码.

-   最后,我们将调用`cmake`将所有输出放入`OUT_DIR`环境变量,然后打印必要的元数据,以指导rustc如何链接到libgit2.

这个构建脚本的大部分功能很容易重构为常见的依赖项,因此我们的构建脚本不像这个描述那样令人生畏!实际上,通过构建依赖关系,构建脚本应该非常简单.
