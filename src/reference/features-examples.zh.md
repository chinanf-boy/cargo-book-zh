## Features Examples

下面展示了一些实际的功能示例。

### Minimizing build times and file sizes

> 让构建时间的软件大小最小化

有些软件包使用功能，因此如果未启用这些功能，它会减小箱子的大小，并缩短编译时间。例如：

- [`syn`]是一种用于解析 Rust 代码的流行箱子。由于它非常流行，因此有助于减少编译时间，因为它会影响很多项目。它有一个[清晰的功能列表文档][syn-features]，可用于最小化其包含的代码量的功能。
- [`regex`]有一个[几项功能][regex-features]，那是[很好的功能文档][regex-docs]。删除 Unicode 支持可以减少生成的文件大小，因为它可以删除一些大型表格。
- [`winapi`]有[一长串的功能][winapi-features]，用来限制那些 Windows API 绑定。
- [`web-sys`]是另一个例子，与`winapi`类似，它提供了一个[超大的范围][web-sys-features]，控制功能 API 绑定。

[`winapi`]: https://crates.io/crates/winapi
[winapi-features]: https://github.com/retep998/winapi-rs/blob/0.3.9/Cargo.toml#L25-L431
[`regex`]: https://crates.io/crates/regex
[`syn`]: https://crates.io/crates/syn
[syn-features]: https://docs.rs/syn/1.0.54/syn/#optional-features
[regex-features]: https://github.com/rust-lang/regex/blob/1.4.2/Cargo.toml#L33-L101
[regex-docs]: https://docs.rs/regex/1.4.2/regex/#crate-features
[`web-sys`]: https://crates.io/crates/web-sys
[web-sys-features]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/crates/web-sys/Cargo.toml#L32-L1395

### Extending behavior

> 行为是可扩展得

[`serde_json`]包上有一个[`preserve_order` 功能][serde_json-preserve_order]，它会[更改][serde_json-code]JSON maps 行为，以保留 keys 插入的顺序。注意到的是，它启用了可选的依赖项[`indexmap`]，来实现这个新的行为。

当改变这样的行为时，要小心确保这些改变是正确的，[SemVer 兼容的][semver compatible]。也就是说，启用该功能不应破坏通常在关闭该功能的情况下，生成的代码。

[`serde_json`]: https://crates.io/crates/serde_json
[serde_json-preserve_order]: https://github.com/serde-rs/json/blob/v1.0.60/Cargo.toml#L53-L56
[semver compatible]: features.md#semver-compatibility
[serde_json-code]: https://github.com/serde-rs/json/blob/v1.0.60/src/map.rs#L23-L26
[`indexmap`]: https://crates.io/crates/indexmap

### `no_std` support

有些软件包希望同时支持[`no_std`]和`std`环境。这对于支持嵌入式和资源受限的平台很有用，但仍要允许，它可以扩展为支持完整标准库的平台的功能。

这个[`wasm-bindgen`]包是会定义[`std` 作为功能][wasm-bindgen-std]，其是[默认启用的][wasm-bindgen-default]。在 library 的顶层[必然启用 `no_std` 属性][wasm-bindgen-no_std]。之后，要确保`std`还有[`std` prelude][`std` 预载]不自动在（构建）范围内。然后，在代码的各个地方(比如：[example1][wasm-bindgen-cfg1], [example2][wasm-bindgen-cfg2])，它使用`#[cfg(feature = "std")]`属性，有条件地启用需要的额外功能`std`。

[`no_std`]: ../../reference/names/preludes.html#the-no_std-attribute
[`wasm-bindgen`]: https://crates.io/crates/wasm-bindgen
[`std` prelude]: ../../std/prelude/index.html
[wasm-bindgen-std]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L25
[wasm-bindgen-default]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L23
[wasm-bindgen-no_std]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L8
[wasm-bindgen-cfg1]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L270-L273
[wasm-bindgen-cfg2]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L67-L75

### Re-exporting dependency features

> 重导出依赖项功能

有种方便的做法，就是将一个依赖项的功能，重新导出。用户控制这些功能，就无需直接指定这些依赖项。例如，[`regex`]从[`regex_syntax`][regex_syntax-features]包进行[重导出][regex-re-export]，而用户不需要知道`regex_syntax`包，但他们仍然可以访问它包含的功能。

[regex-re-export]: https://github.com/rust-lang/regex/blob/1.4.2/Cargo.toml#L65-L89
[regex_syntax-features]: https://github.com/rust-lang/regex/blob/1.4.2/regex-syntax/Cargo.toml#L17-L32

### Vendoring of C libraries

> C 库的供给

有些软件包提供了，对通用 C 库的绑定（有时，称为["sys" crates][sys])。有时，这些软件包让您可以选择，使用系统上安装的 C 库，或者从源代码构建它。例如，[`openssl`]包上有一个[`vendored` 功能][openssl-vendored]，启用[`openssl-sys`]相应的`vendored`功能，而`openssl-sys`构建脚本有一些[条件性逻辑][openssl-sys-cfg]，导致它从 OpenSSL 源代码的本地副本构建，而不是使用系统中的版本。

这个[`curl-sys`]包是另一个示例，[`static-curl`
功能][curl-sys-static]，让这个包，从源代码构建 libcurl。注意，它还有一个[`force-system-lib-on-osx`][curl-sys-macos]迫使它[使用系统的 libcurl][curl-sys-macos-code]，覆盖 static-curl 设置。

[`openssl`]: https://crates.io/crates/openssl
[`openssl-sys`]: https://crates.io/crates/openssl-sys
[sys]: build-scripts.zh.md#-sys-packages
[openssl-vendored]: https://github.com/sfackler/rust-openssl/blob/openssl-v0.10.31/openssl/Cargo.toml#L19
[build script]: build-scripts.zh.md
[openssl-sys-cfg]: https://github.com/sfackler/rust-openssl/blob/openssl-v0.10.31/openssl-sys/build/main.rs#L47-L54
[`curl-sys`]: https://crates.io/crates/curl-sys
[curl-sys-static]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/Cargo.toml#L49
[curl-sys-macos]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/Cargo.toml#L52
[curl-sys-macos-code]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/build.rs#L15-L20

### Feature precedence

> 功能的优先级

有些软件包可能具有相互排斥的功能。处理这一问题的一个选择是：优先选择一个功能。这个[`log`]包就是一个例子。它有[几项功能][log-features]用于在编译时，选择最大日志记录级别，文档在[here][log-docs]。它使用[`cfg-if`]，用来[选择一个优先级][log-cfg-if]。如果启用了多个功能，则与较低级别相比，更高的“最大”级别将应用。

[`log`]: https://crates.io/crates/log
[log-features]: https://github.com/rust-lang/log/blob/0.4.11/Cargo.toml#L29-L42
[log-docs]: https://docs.rs/log/0.4.11/log/#compile-time-filters
[log-cfg-if]: https://github.com/rust-lang/log/blob/0.4.11/src/lib.rs#L1422-L1448
[`cfg-if`]: https://crates.io/crates/cfg-if

### Proc-macro companion package

> （Proc-macro）宏指令伴侣

有些软件包有一个与之密切相关的 proc 宏。但是，并非所有用户都需要使用 proc 宏。通过将 proc 宏设置为可选依赖项，可以方便地选择是否包含它。这很有帮助，因为有时 proc 宏版本，必须与父包保持同步，并且您不希望强制用户必须指定这两个依赖项，并保持它们同步。

例如[`serde`]它有一个[`derive`][serde-derive]使[`serde_derive`]过程宏。这个`serde_derive`箱子被紧紧地绑在一起`serde`，所以它使用[equals version
requirement][serde-equals]以确保它们保持同步。

[`serde`]: https://crates.io/crates/serde
[`serde_derive`]: https://crates.io/crates/serde_derive
[serde-derive]: https://github.com/serde-rs/serde/blob/v1.0.118/serde/Cargo.toml#L34-L35
[serde-equals]: https://github.com/serde-rs/serde/blob/v1.0.118/serde/Cargo.toml#L17

### Nightly-only features

<!-- TODO: 2022-2-25  -->

一些软件包想要尝试 API 或语言的实验性功能，这些功能只在 Rust [nightly channel]上可用，但是，他们可能不想要求用户也使用 nightly channel。例如[`wasm-bindgen`]有一个[`nightly` feature][wasm-bindgen-nightly]，使得[extended API][wasm-bindgen-unsize]能够使用 nightly channel 上的[`Unsize`] marker trait。

请注意，在箱子的根目录上，它使用[`cfg_attr` 启用 nightly
功能][wasm-bindgen-cfg_attr]。记住[`feature` 属性]与 Cargo 功能无关，用于实验性语言功能——(可选择性)。

[`rand`]的这个[`simd_support` feature][rand-simd_support]是另一个例子，它依赖于只在 nightly channel 上构建的依赖项。

[`wasm-bindgen`]: https://crates.io/crates/wasm-bindgen
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[wasm-bindgen-nightly]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L27
[wasm-bindgen-unsize]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/closure.rs#L257-L269
[`unsize`]: ../../std/marker/trait.Unsize.html
[wasm-bindgen-cfg_attr]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L11
[`feature` attribute]: ../../unstable-book/index.html
[`rand`]: https://crates.io/crates/rand
[rand-simd_support]: https://github.com/rust-random/rand/blob/0.7.3/Cargo.toml#L40

### Experimental features

有些软件包有新的功能，他们可能想试用，却不保证这些 API 的稳定性。这些功能通常被记录为实验性的，因为将来可能会改变或中断（即使是在一个小版本中）。例如[`async-std`]包，它有一个[`unstable` feature][async-std-unstable]，会有个[gates
new APIs][async-std-gate]能选择使用，但可能还没有完全准备好被依赖。

[`async-std`]: https://crates.io/crates/async-std
[async-std-unstable]: https://github.com/async-rs/async-std/blob/v1.8.0/Cargo.toml#L38-L42
[async-std-gate]: https://github.com/async-rs/async-std/blob/v1.8.0/src/macros.rs#L46
