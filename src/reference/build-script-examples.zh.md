## Build Script Examples

以下部分说明了编写构建脚本的一些示例。

一些常见的构建脚本功能可以通过[crates.io].查看[`build-dependencies`
keyword](https://crates.io/keywords/build-dependencies)看看有什么可用的。以下是一些流行板条箱的样本[^†]\:

-   [`bindgen`](https://crates.io/crates/bindgen)-自动生成到C库的Rust FFI绑定。
-   [`cc`](https://crates.io/crates/cc)-编译C/C++/assembly。
-   [`pkg-config`](https://crates.io/crates/pkg-config)-使用`pkg-config`公用事业
-   [`cmake`](https://crates.io/crates/cmake)-运行`cmake`构建工具来构建本机库。
-   [`autocfg`](https://crates.io/crates/autocfg), [`rustc_version`](https://crates.io/crates/rustc_version), [`version_check`](https://crates.io/crates/version_check)-这些板条箱提供了基于当前数据实现条件编译的方法`rustc`例如编译器的版本。

[^†]：此列表不是背书。评估你的依赖关系，看看哪个适合你的项目。

### Code generation

由于各种原因，一些Cargo包需要在编译之前生成代码。在这里，我们将浏览一个简单的示例，该示例生成一个库调用，作为构建脚本的一部分。

首先，让我们看看这个包的目录结构：

```text
.
├── Cargo.toml
├── build.rs
└── src
    └── main.rs

1 directory, 3 files
```

在这里我们可以看到，我们有一个`build.rs`在中构建脚本和二进制文件`main.rs`.此程序包有一个基本清单：

```toml
# Cargo.toml

[package]
name = "hello-from-generated-code"
version = "0.1.0"
```

让我们看看构建脚本中的内容：

```rust,no_run
// build.rs

use std::env;
use std::fs;
use std::path::Path;

fn main() {
    let out_dir = env::var_os("OUT_DIR").unwrap();
    let dest_path = Path::new(&out_dir).join("hello.rs");
    fs::write(
        &dest_path,
        "pub fn message() -> &'static str {
            \"Hello, World!\"
        }
        "
    ).unwrap();
    println!("cargo:rerun-if-changed=build.rs");
}
```

这里有几点值得注意：

-   脚本使用`OUT_DIR`环境变量来发现输出文件应该位于何处。它可以使用进程的当前工作目录来查找输入文件的位置，但在这种情况下，我们没有任何输入文件。
-   一般来说，构建脚本不应修改外部的任何文件`OUT_DIR`.乍一看似乎没什么问题，但当你把这种板条箱当作依赖物使用时，确实会造成问题，因为有一个*含蓄的*来源于`.cargo/registry`应该是不变的。`cargo`打包时不允许使用此类脚本。
-   这个脚本相对简单，因为它只写出一个生成的小文件。人们可以想象，其他更奇特的操作可能会发生，比如从C头文件或其他语言定义生成一个Rust模块。
-   这个[`rerun-if-changed` instruction](build-scripts.md#rerun-if-changed)告诉Cargo，只有在生成脚本本身发生更改时，才需要重新运行生成脚本。如果没有这一行，如果包中的任何文件发生更改，Cargo将自动运行构建脚本。如果您的代码生成使用了一些输入文件，您可以在这里打印这些文件的列表。

接下来，让我们来看看图书馆本身：

```rust,ignore
// src/main.rs

include!(concat!(env!("OUT_DIR"), "/hello.rs"));

fn main() {
    println!("{}", message());
}
```

这才是真正神奇的地方。库正在使用rustc定义的[`include!` macro][include-macro]结合[`concat!`][concat-macro]和[`env!`][env-macro]宏来包含生成的文件(`hello.rs`)进入板条箱的汇编。

使用这里显示的结构，板条箱可以包含构建脚本本身生成的任意数量的文件。

[include-macro]: ../../std/macro.include.html

[concat-macro]: ../../std/macro.concat.html

[env-macro]: ../../std/macro.env.html

### Building a native library

有时需要将一些本地C或C++代码作为包的一部分来构建。这是另一个利用构建脚本在Rust板条箱本身之前构建本机库的优秀用例。例如，我们将创建一个Rust库，它调用C来打印“Hello，World！”。

像上面一样，让我们先看看包的布局：

```text
.
├── Cargo.toml
├── build.rs
└── src
    ├── hello.c
    └── main.rs

1 directory, 4 files
```

和以前很相似！接下来是清单：

```toml
# Cargo.toml

[package]
name = "hello-world-from-c"
version = "0.1.0"
edition = "2021"
```

现在我们不打算使用任何构建依赖项，所以现在让我们看看构建脚本：

```rust,no_run
// build.rs

use std::process::Command;
use std::env;
use std::path::Path;

fn main() {
    let out_dir = env::var("OUT_DIR").unwrap();

    // Note that there are a number of downsides to this approach, the comments
    // below detail how to improve the portability of these commands.
    Command::new("gcc").args(&["src/hello.c", "-c", "-fPIC", "-o"])
                       .arg(&format!("{}/hello.o", out_dir))
                       .status().unwrap();
    Command::new("ar").args(&["crus", "libhello.a", "hello.o"])
                      .current_dir(&Path::new(&out_dir))
                      .status().unwrap();

    println!("cargo:rustc-link-search=native={}", out_dir);
    println!("cargo:rustc-link-lib=static=hello");
    println!("cargo:rerun-if-changed=src/hello.c");
}
```

这个构建脚本首先将C文件编译成一个对象文件（通过调用`gcc`)然后将该对象文件转换为静态库（通过调用`ar`).最后一步是向Cargo公司反馈，告知我们的产品处于良好状态`out_dir`编译器应该将板条箱链接到`libhello.a`通过`-l static=hello`选项

请注意，这种硬编码方法有许多缺点：

-   这个`gcc`命令本身不能跨平台移植。例如，Windows平台不太可能`gcc`，甚至不是所有Unix平台都有`gcc`这个`ar`指挥部也处于类似的情况。
-   这些命令不考虑交叉编译。如果我们为Android这样的平台进行交叉编译，那么`gcc`将生成一个ARM可执行文件。

不过，不用担心，这就是`build-dependencies`进来会有帮助的！Cargo生态系统有许多软件包，使这类任务变得更容易、便携和标准化。让我们试试这个[`cc`
crate](https://crates.io/crates/cc)从…起[crates.io].首先，将其添加到`build-dependencies`在里面`Cargo.toml`:

```toml
[build-dependencies]
cc = "1.0"
```

然后重写构建脚本以使用此板条箱：

```rust,ignore
// build.rs

fn main() {
    cc::Build::new()
        .file("src/hello.c")
        .compile("hello");
    println!("cargo:rerun-if-changed=src/hello.c");
}
```

这个[`cc` crate]对C代码的一系列构建脚本需求进行了抽象：

-   它调用适当的编译器（MSVC for windows，`gcc`对明哥来说，`cc`用于Unix平台等）。
-   这需要时间`TARGET`通过向正在使用的编译器传递适当的标志，将变量考虑在内。
-   其他环境变量，例如`OPT_LEVEL`, `DEBUG`等等，都是自动处理的。
-   标准输出和`OUT_DIR`地点也由`cc`图书馆

在这里，我们可以开始看到尽可能多地将功能扩展到常见的构建依赖项，而不是在所有构建脚本中复制逻辑的一些主要好处！

回到案例研究，让我们快速看一下`src`目录：

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
// of selecting what to link over to the build script rather than hard-coding
// it in the source file.
extern { fn hello(); }

fn main() {
    unsafe { hello(); }
}
```

我们走了！这将完成我们使用构建脚本本身从Cargo包构建一些C代码的示例。这也说明了为什么在许多情况下，使用构建依赖项是至关重要的，甚至更简洁！

我们还看到了一个简单的例子，说明了构建脚本如何将板条箱作为一个依赖项，仅用于构建过程，而不用于运行时的板条箱本身。

[`cc` crate]: https://crates.io/crates/cc

### Linking to system libraries

此示例演示如何链接系统库，以及如何使用构建脚本来支持此用例。

Rust板条箱经常想要链接到系统上提供的本机库，以绑定其功能，或者只是将其用作实现细节的一部分。以平台无关的方式执行此操作时，这是一个相当微妙的问题。如果可能的话，最好是尽可能多地将其外包出去，让消费者尽可能容易地做到这一点。

在本例中，我们将创建一个到系统zlib库的绑定。这是一个在大多数提供数据压缩的类Unix系统上常见的库。这已经在[`libz-sys`
crate]，但对于这个例子，我们将做一个极其简化的版本。退房[the source code][libz-source]举个完整的例子。

为了便于找到图书馆的位置，我们将使用[`pkg-config` crate].这个板条箱使用系统的`pkg-config`用于发现有关库的信息的实用程序。它会自动告诉Cargo连接图书馆需要什么。这可能只适用于具有`pkg-config`安装。让我们从设置清单开始：

```toml
# Cargo.toml

[package]
name = "libz-sys"
version = "0.1.0"
edition = "2021"
links = "z"

[build-dependencies]
pkg-config = "0.3.16"
```

请注意，我们包括了`links`输入`package`桌子这告诉Cargo我们正在连接到`libz`图书馆看见["Using another sys
crate"](#using-another-sys-crate)举个例子来说明这一点。

构建脚本相当简单：

```rust,ignore
// build.rs

fn main() {
    pkg_config::Config::new().probe("zlib").unwrap();
    println!("cargo:rerun-if-changed=build.rs");
}
```

让我们用一个基本的FFI绑定来总结这个例子：

```rust,ignore
// src/lib.rs

use std::os::raw::{c_uint, c_ulong};

extern "C" {
    pub fn crc32(crc: c_ulong, buf: *const u8, len: c_uint) -> c_ulong;
}

#[test]
fn test_crc32() {
    let s = "hello";
    unsafe {
        assert_eq!(crc32(0, s.as_ptr(), s.len() as c_uint), 0x3610a686);
    }
}
```

跑`cargo build -vv`查看生成脚本的输出。在一个具有`libz`已经安装，它可能看起来像这样：

```text
[libz-sys 0.1.0] cargo:rustc-link-search=native=/usr/lib
[libz-sys 0.1.0] cargo:rustc-link-lib=z
[libz-sys 0.1.0] cargo:rerun-if-changed=build.rs
```

美好的`pkg-config`做了所有的工作，找到了图书馆，并告诉Cargo它在哪里。

软件包包含库的源代码，如果在系统中找不到源代码，或者设置了功能或环境变量，则静态地构建源代码，这种情况并不少见。例如，真实的[`libz-sys` crate]检查环境变量`LIBZ_SYS_STATIC`或者`static`功能从源代码而不是使用系统库构建它。退房[the
source][libz-source]以获取更完整的示例。

[`libz-sys` crate]: https://crates.io/crates/libz-sys

[`pkg-config` crate]: https://crates.io/crates/pkg-config

[libz-source]: https://github.com/rust-lang/libz-sys

### Using another `sys` crate

当使用`links`关键点，板条箱可以设置元数据，这些元数据可以被依赖于它的其他板条箱读取。这提供了一种在板条箱之间传递信息的机制。在本例中，我们将创建一个C库，该库使用来自真实世界的zlib[`libz-sys` crate].

如果你有一个依赖zlib的C库，你可以利用[`libz-sys`
crate]自动查找或构建它。这非常适合跨平台支持，例如通常不安装zlib的Windows。`libz-sys` [sets
the `include`
metadata](https://github.com/rust-lang/libz-sys/blob/3c594e677c79584500da673f918c4d2101ac97a1/build.rs#L156)告诉其他包在哪里可以找到zlib的头文件。我们的构建脚本可以使用`DEP_Z_INCLUDE`环境变量。下面是一个例子：

```toml
# Cargo.toml

[package]
name = "zuser"
version = "0.1.0"
edition = "2021"

[dependencies]
libz-sys = "1.0.25"

[build-dependencies]
cc = "1.0.46"
```

这里我们包括`libz-sys`这将确保只有一个`libz`在最终库中使用，并允许我们从构建脚本访问它：

```rust,ignore
// build.rs

fn main() {
    let mut cfg = cc::Build::new();
    cfg.file("src/zuser.c");
    if let Some(include) = std::env::var_os("DEP_Z_INCLUDE") {
        cfg.include(include);
    }
    cfg.compile("zuser");
    println!("cargo:rerun-if-changed=src/zuser.c");
}
```

具有`libz-sys`完成所有繁重的工作后，C源代码现在可能包括zlib头，它应该可以找到该头，即使在尚未安装它的系统上也是如此。

```c
// src/zuser.c

#include "zlib.h"

// … rest of code that makes use of zlib.
```

### Conditional compilation

构建脚本可能会发出[`rustc-cfg` instructions]它可以启用可在编译时检查的条件。在本例中，我们将了解[`openssl` crate]使用它来支持多个版本的OpenSSL库。

这个[`openssl-sys` crate]实现构建和链接OpenSSL库。它支持多个不同的实现（比如LibreSSL）和多个版本。它利用了`links`键，以便它可以将信息传递给其他构建脚本。它经过的事情之一是`version_number`密钥，即检测到的OpenSSL版本。构建脚本中的代码看起来像[like
this](https://github.com/sfackler/rust-openssl/blob/dc72a8e2c429e46c275e528b61a733a66e7877fc/openssl-sys/build/main.rs#L216):

```rust,ignore
println!("cargo:version_number={:x}", openssl_version);
```

此指令会导致`DEP_OPENSSL_VERSION_NUMBER`环境变量设置在任何板条箱中，直接取决于`openssl-sys`.

这个`openssl`提供更高级别接口的板条箱指定`openssl-sys`作为一种依赖。这个`openssl`构建脚本可以读取`openssl-sys`使用`DEP_OPENSSL_VERSION_NUMBER`环境变量。它用这个来产生一些[`cfg`
values](https://github.com/sfackler/rust-openssl/blob/dc72a8e2c429e46c275e528b61a733a66e7877fc/openssl/build.rs#L18-L36):

```rust,ignore
// (portion of build.rs)

if let Ok(version) = env::var("DEP_OPENSSL_VERSION_NUMBER") {
    let version = u64::from_str_radix(&version, 16).unwrap();

    if version >= 0x1_00_01_00_0 {
        println!("cargo:rustc-cfg=ossl101");
    }
    if version >= 0x1_00_02_00_0 {
        println!("cargo:rustc-cfg=ossl102");
    }
    if version >= 0x1_01_00_00_0 {
        println!("cargo:rustc-cfg=ossl110");
    }
    if version >= 0x1_01_00_07_0 {
        println!("cargo:rustc-cfg=ossl110g");
    }
    if version >= 0x1_01_01_00_0 {
        println!("cargo:rustc-cfg=ossl111");
    }
}
```

这些`cfg`然后可以将值与[`cfg` attribute]或者[`cfg`
macro]有条件地包含代码。例如，在OpenSSL 1.1.1中添加了SHA3支持，所以[conditionally
excluded](https://github.com/sfackler/rust-openssl/blob/dc72a8e2c429e46c275e528b61a733a66e7877fc/openssl/src/hash.rs#L67-L85)对于旧版本：

```rust,ignore
// (portion of openssl crate)

#[cfg(ossl111)]
pub fn sha3_224() -> MessageDigest {
    unsafe { MessageDigest(ffi::EVP_sha3_224()) }
}
```

当然，使用它时应该小心，因为它使生成的二进制文件更加依赖于构建环境。在本例中，如果二进制文件被分发到另一个系统，它可能没有完全相同的共享库，这可能会导致问题。

[`cfg` attribute]: ../../reference/conditional-compilation.md#the-cfg-attribute

[`cfg` macro]: ../../std/macro.cfg.html

[`rustc-cfg` instructions]: build-scripts.md#rustc-cfg

[`openssl` crate]: https://crates.io/crates/openssl

[`openssl-sys` crate]: https://crates.io/crates/openssl-sys

[crates.io]: https://crates.io/
