
## 构建脚本支持

有些软件包需要编译第三方非Rust代码,例如C库.其他包需要链接到C库,这些库可以位于系统上,也可能需要从源构建.其他人仍需要功能设施,例如在构建之前生成代码(想想解析器生成器).

Cargo并不打算替换那些针对这些任务进行了优化的其他工具,但它确实与它们集成在一起`build`配置选项.

```toml
[package]
# ...
build = "build.rs"
```

由Rust指定的Rust文件`build`命令(相对于包根)将在包中编译其他任何内容之前进行编译和调用,从而允许Rust代码依赖于构建或生成的工件.请注意,没有默认值`build`,必要时必须明确指定.

build命令的一些示例用例是:

-   构建捆绑的C库.
-   在主机系统上查找C库.
-   从规范生成Rust模块.
-   执行包所需的任何特定于平台的配置.

下面将详细介绍这些用例中的每一个,以提供构建命令如何工作的示例.

### 对构建脚本的输入

运行构建脚本时,构建脚本有许多输入,所有输入都以.的形式传递[环境变量][env].

除了环境变量之外,构建脚本的当前目录是构建脚本的包的源目录.

[env]: 03-04-environment-variables.html

### 构建脚本的输出

由构建脚本打印到stdout的所有行都写入文件`target/debug/build/<pkg>/output`(具体位置可能取决于您的配置).任何以.开头的行`cargo:`由Cargo直接解释.这一行必须是这种形式`cargo:key=value`,如下面的例子:

```notrust
# specially recognized by Cargo
cargo:rustc-link-lib=static=foo
cargo:rustc-link-search=native=/path/to/foo
cargo:rustc-cfg=foo
# arbitrary user-defined metadata
cargo:root=/path/to/foo
cargo:libdir=/path/to/foo/lib
cargo:include=/path/to/foo/include
```

Cargo可以识别一些特殊键,其中一些键会影响箱子的构建方式:

-   `rustc-link-lib=[KIND=]NAME`表示指定的值是库名,应作为a传递给编译器`-l`旗.可选的`KIND`可以是其中之一`static`,`dylib`(默认),或`framework`,见`rustc --help`更多细节.
-   `rustc-link-search=[KIND=]PATH`表示指定的值是库搜索路径,应作为a传递给编译器`-L`旗.可选的`KIND`可以是其中之一`dependency`,`crate`,`native`,`framework`要么`all`(默认),请参阅`rustc --help`更多细节.
-   `rustc-flags=FLAGS`是一组仅传递给编译器的标志`-l`和`-L`支持标志.
-   `rustc-cfg=FEATURE`表示指定的功能将作为a传递`--cfg`标志到编译器.这对于执行各种功能的编译时检测通常很有用.
-   `rerun-if-changed=PATH`是一个文件或目录的路径,指示构建脚本在更改时应重新运行(由文件上较新的最后修改时间戳检测到).通常,如果crate根目录中的任何文件发生更改,则会重新运行构建脚本,但这可以用于将更改范围仅限于一小组文件.(如果此路径指向目录,则不会遍历整个目录以进行更改 - 只更改目录本身的时间戳(对应于目录中的某些类型的更改,具体取决于平台)将触发重建.请求重新运行整个目录中的任何更改,为目录中的所有内容打印一行,递归地输入其中的所有内容.)\
    请注意,如果构建脚本本身(或其中一个依赖项)发生更改,则会重建并无条件地重新运行,因此`cargo:rerun-if-changed=build.rs`几乎总是多余的(除非你想忽略除了之外的所有其他文件的更改)`build.rs`).
-   `warning=MESSAGE`是一个消息,将在构建脚本运行完毕后打印到主控制台.仅针对路径依赖性(即您在本地处理的那些)显示警告,因此,例如,默认情况下不会发出在crates.io crates中打印的警告.

任何其他元素都是用户定义的元数据,将传递给依赖者.有关这方面的更多信息,请参阅[`links`][links]部分.

[links]: #the-links-manifest-key

### 构建依赖关系

构建脚本也允许依赖于其他基于货物的板条箱.依赖关系是通过`build-dependencies`清单的一部分.

```toml
[build-dependencies]
foo = { git = "https://github.com/your-packages/foo" }
```

构建脚本**才不是**有权访问中列出的依赖项`dependencies`要么`dev-dependencies`部分(它们还没有建成!).除非明确说明,否则所有构建依赖项也将无法用于程序包本身.

### 该`links`清单密钥

除了清单键`build`,Cargo也支持一个`links`manifest键,用于声明链接到的本机库的名称:

```toml
[package]
# ...
links = "foo"
build = "build.rs"
```

此清单表明程序包链接到`libfoo`本机库,它还有一个用于定位和/或构建库的构建脚本.货物需要一个`build`如果指定了命令`links`还指定了条目.

此清单键的目的是让Cargo了解包具有的本机依赖关系集,以及提供在包构建脚本之间传递元数据的原则系统.

首先,货物要求每个最多一个包裹`links`值.换句话说,禁止将两个包链接到同一本机库.但请注意,有[惯例到位][star-sys]缓解这个.

[star-sys]: #-sys-packages

如上所述,在输出格式中,每个构建脚本都可以以键值对的形式生成任意一组元数据.此元数据将传递给构建脚本**依赖的**包.例如,如果`libbar`依赖于取决于`libfoo`,如果`libfoo`生成`key=value`作为其元数据的一部分,然后是构建脚本`libbar`将有环境变量`DEP_FOO_KEY=value`.

请注意,元数据仅传递给直接依赖者,而不是传递依赖者.下面链接到系统库案例研究中概述了此元数据传递的动机.

### 覆盖构建脚本

如果清单包含一个`links`key,然后Cargo支持覆盖使用自定义库指定的构建脚本.此功能的目的是防止完全运行有问题的构建脚本,而是提前提供元数据.

要覆盖构建脚本,请将以下配置放在任何可接受的货物中[配置位置](03-03-config.html).

```toml
[target.x86_64-unknown-linux-gnu.foo]
rustc-link-search = ["/path/to/foo"]
rustc-link-lib = ["foo"]
root = "/path/to/foo"
key = "value"
```

本节说明了目标`x86_64-unknown-linux-gnu`图书馆命名`foo`已指定元数据.此元数据与生成的元数据相同,就像构建脚本已运行一样,提供了许多键/值对`rustc-flags`,`rustc-link-search`,和`rustc-link-lib`钥匙有点特别.

使用此配置,如果包声明它链接到`foo`然后构建脚本将**不**编译或运行,而是使用指定的元数据.

### 案例研究:代码生成

由于各种原因,一些货物包需要在编译之前生成代码.在这里,我们将介绍一个简单的示例,它将生成库调用作为构建脚本的一部分.

首先,我们来看看这个包的目录结构:

```notrust
.
├── Cargo.toml
├── build.rs
└── src
    └── main.rs

1 directory, 3 files
```

在这里我们可以看到我们有一个`build.rs`构建脚本和我们的二进制文件`main.rs`.接下来,我们来看看清单:

```toml
# Cargo.toml

[package]
name = "hello-from-generated-code"
version = "0.1.0"
authors = ["you@example.com"]
build = "build.rs"
```

在这里,我们可以看到我们已经指定了一个构建脚本,我们将使用它来生成一些代码.让我们看看构建脚本中的内容:

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

这里有几点需要注意:

-   该脚本使用`OUT_DIR`环境变量,用于发现输出文件的位置.它可以使用进程的当前工作目录来查找输入文件的位置,但在这种情况下,我们没有任何输入文件.
-   这个脚本相对简单,因为它只写出一个小的生成文件.可以想象,可以进行其他更奇特的操作,例如,从C头文件或其他语言定义生成Rust模块.

接下来,我们来看看图书馆本身:

```rust,ignore
// src/main.rs

include!(concat!(env!("OUT_DIR"), "/hello.rs"));

fn main() {
    println!("{}", message());
}
```

这是真正的魔法发生的地方.该库正在使用rustc定义的`include!`宏与结合`concat!`和`env!`宏包含生成的文件(`hello.rs`)进入箱子的汇编.

使用此处显示的结构,crates可以包含构建脚本本身生成的任意数量的文件.

### 案例研究:构建一些本机代码

有时需要构建一些本机C或C ++代码作为包的一部分.这是利用构建脚本在Rust crate之前构建本机库的另一个很好的用例.作为一个例子,我们将创建一个Rust库,调用C来打印"Hello,World!".

如上所述,让我们先来看看项目布局:

```notrust
.
├── Cargo.toml
├── build.rs
└── src
    ├── hello.c
    └── main.rs

1 directory, 4 files
```

和以前很相似!接下来,清单:

```toml
# Cargo.toml

[package]
name = "hello-world-from-c"
version = "0.1.0"
authors = ["you@example.com"]
build = "build.rs"
```

现在我们不打算使用任何构建依赖项,所以现在让我们看一下构建脚本:

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

这个构建脚本首先将我们的C文件编译成目标文件(通过调用`gcc`)然后将此目标文件转换为静态库(通过调用`ar`).最后一步是向Cargo本身反馈说我们的输出是在`out_dir`并且编译器应该将箱子链接到`libhello.a`静静地通过`-l static=hello`旗.

请注意,这种硬编码方法存在许多缺点:

-   该`gcc`命令本身不能跨平台移植.例如,Windows平台不太可能`gcc`,甚至所有Unix平台都没有`gcc`.该`ar`命令也处于类似情况.
-   这些命令不考虑交叉编译.如果我们正在为Android等平台进行交叉编译,则不太可能`gcc`将生成一个ARM可执行文件.

但不要害怕,这是一个地方`build-dependencies`进入会有所帮助!Cargo生态系统有许多软件包可以使这类任务变得更加容易,便携和标准化.例如,构建脚本可以写成:

```rust,ignore
// build.rs

// Bring in a dependency on an externally maintained `gcc` package which manages
// invoking the C compiler.
extern crate gcc;

fn main() {
    gcc::compile_library("libhello.a", &["src/hello.c"]);
}
```

在上添加构建时依赖项`gcc`箱子里面还有以下内容`Cargo.toml`:

```toml
[build-dependencies]
gcc = "0.3"
```

该[`gcc`箱](https://crates.io/crates/gcc)摘要C代码的一系列构建脚本要求:

-   它调用适当的编译器(Windows的MSVC,`gcc`对于MinGW,`cc`对于Unix平台等).
-   需要的`TARGET`通过将适当的标志传递给正在使用的编译器来考虑变量.
-   其他环境变量,例如`OPT_LEVEL`,`DEBUG`等等,都是自动处理的.
-   标准输出和`OUT_DIR`地点也由处理`gcc`图书馆.

在这里,我们可以开始看到农业的一些主要好处,尽可能多的功能,以及常见的构建依赖,而不是在所有构建脚本中复制逻辑!

回到案例研究,让我们快速浏览一下内容`src`目录:

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

我们走了!这应该完成我们使用构建脚本本身从Cargo包构建一些C代码的示例.这也说明了为什么在许多情况下使用构建依赖关系至关重要,甚至更简洁!

我们还看到了一个简短的例子,说明构建脚本如何将crate作为依赖纯粹用于构建过程,而不是在运行时用于crate本身.

### 案例研究:链接到系统库

这里的最后一个案例研究将调查Cargo库如何链接到系统库以及如何利用构建脚本来支持这个用例.

Rust crate通常希望链接到系统上经常提供的本机库来绑定其功能,或者只是将其用作实现细节的一部分.当以与平台无关的方式执行此操作时,这是一个非常细微的问题,并且构建脚本的目的是尽可能地尽可能多地将其尽可能地用于使消费者尽可能容易.

作为一个例子,让我们来看看其中一个[货物自己的依赖][git2-rs],[libgit2][libgit2].该库有许多约束:

[git2-rs]: https://github.com/alexcrichton/git2-rs/tree/master/libgit2-sys

[libgit2]: https://github.com/libgit2/libgit2

-   它在Unix上具有对OpenSSL的可选依赖,以实现https传输.
-   它在所有平台上都有对libssh2的可选依赖,以实现ssh传输.
-   默认情况下,它通常不会安装在所有系统上.
-   它可以从源代码构建`cmake`.

为了想象这里发生了什么,让我们来看看相关货物包装的清单.

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

如上所示,我们有一个`build`脚本指定,但值得注意的是这个例子有一个`links`条目表明箱子(`libgit2-sys`)链接到`git2`本土图书馆.

在这里我们也看到了无条件的依赖`libssh2`通过`libssh2-sys`箱子,以及特定于平台的依赖`openssl-sys`对于\*nix(现在已经省略了其他变体).表达可能看起来有点违反直觉*C依赖项*在里面*货物清单*,但这实际上是在这个领域使用Cargo的惯例之一.

### `*-sys`包

为了减轻与系统库的链接,Cargo有一个*惯例*包命名和功能.任何名为的包`foo-sys`将提供两个主要功能:

-   库包将链接到本机库`libfoo`.这通常会探测当前系统`libfoo`在从源头建立之前.
-   图书馆箱子将提供**声明**用于...的功能`libfoo`,但确实如此**不**提供绑定或更高级别的抽象.

这套`*-sys`packages提供了一组通用的依赖项,用于链接到本机库.使用本机库相关包的惯例可以带来许多好处:

-   常见的依赖关系`foo-sys`每个值减去一个包的上述规则`links`.
-   通用依赖关系允许在发现时集中逻辑`libfoo`本身(或从源头构建).
-   这些依赖项很容易被覆盖.

### 构建libgit2

现在我们已经整理了libgit2的依赖项,我们需要实际编写构建脚本.我们不打算在这里查看特定的代码片段,而只是看一下构建脚本的高级细节.`libgit2-sys`.这并不是建议所有包都遵循这一策略,而只是概述一个特定的策略.

构建脚本的第一步应该是查询主机系统上是否已安装libgit2.为此,我们将利用预先存在的工具`pkg-config`(当它可用时).我们也会用`build-dependencies`部分重构所有的`pkg-config`相关代码(或某人已经完成了!).

如果`pkg-config`找不到libgit2,或者如果找不到`pkg-config`刚刚没有安装,下一步是从捆绑的源代码构建libgit2(作为一部分分发)`libgit2-sys`本身).这样做有一些细微差别我们需要考虑,但是:

-   libgit2的构建系统,`cmake`,需要能够找到libgh2的libssh2的可选依赖项.我们确信我们已经建立了它(它是货物依赖),我们只需要传达这些信息.为此,我们利用元数据格式在构建脚本之间传递信息.在这个例子中打印出libssh2包`cargo:root=...`告诉我们libssh2的安装位置,然后我们可以将其传递给cmake`CMAKE_PREFIX_PATH`环境变量.

-   我们需要处理一些`CFLAGS`编译C代码时的值(并告诉`cmake`对这个).我们可能希望传递的一些标志是`-m64`对于64位代码,`-m32`对于32位代码,或`-fPIC`对于64位代码也是如此.

-   最后,我们将调用`cmake`将所有输出放入`OUT_DIR`环境变量,然后我们将打印必要的元数据来指示rustc如何链接到libgit2.

这个构建脚本的大多数功能都可以轻松地重构为常见的依赖项,因此我们的构建脚本并不像这个描述那么令人生畏!实际上,预期构建脚本非常简洁,通过如上所述的种植逻辑来构建依赖关系.
