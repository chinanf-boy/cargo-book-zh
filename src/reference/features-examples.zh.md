## Features Examples

下面展示了一些实际的功能示例。

### Minimizing build times and file sizes

有些软件包使用特性，因此如果未启用这些特性，它会减小机箱的大小并缩短编译时间。例如：

-   [`syn`]是一种用于解析锈迹代码的流行板条箱。由于它非常流行，因此有助于减少编译时间，因为它会影响很多项目。它有一个[clearly documented list][syn-features]可用于最小化其包含的代码量的功能。
-   [`regex`]有一个[several features][regex-features]那是[well
    documented][regex-docs].删除Unicode支持可以减少生成的文件大小，因为它可以删除一些大型表。
-   [`winapi`]有[a large number][winapi-features]限制其支持的Windows API绑定的功能。
-   [`web-sys`]另一个例子与`winapi`这提供了一个[huge
    surface area][web-sys-features]受使用功能限制的API绑定。

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

这个[`serde_json`]包裹上有一个[`preserve_order` feature][serde_json-preserve_order]哪一个[changes the behavior][serde_json-code]JSON映射，以保留键插入的顺序。请注意，它启用了可选的依赖项[`indexmap`]实施新的行为。

当改变这样的行为时，要小心确保这些改变是正确的[SemVer compatible]。也就是说，启用该功能不应破坏通常在关闭该功能的情况下生成的代码。

[`serde_json`]: https://crates.io/crates/serde_json

[serde_json-preserve_order]: https://github.com/serde-rs/json/blob/v1.0.60/Cargo.toml#L53-L56

[semver compatible]: features.md#semver-compatibility

[serde_json-code]: https://github.com/serde-rs/json/blob/v1.0.60/src/map.rs#L23-L26

[`indexmap`]: https://crates.io/crates/indexmap

### `no_std` support

有些软件包希望同时支持这两种功能[`no_std`]和`std`环境。这对于支持嵌入式和资源受限的平台很有用，但仍然允许扩展支持完整标准库的平台的功能。

这个[`wasm-bindgen`]包定义了[`std` feature][wasm-bindgen-std]就是[enabled by default][wasm-bindgen-default].在图书馆的顶层[unconditionally enables the `no_std` attribute][wasm-bindgen-no_std].这确保`std`还有[`std` prelude]不自动在范围内。然后，在代码的各个地方([example1][wasm-bindgen-cfg1], [example2][wasm-bindgen-cfg2])，它使用`#[cfg(feature = "std")]`属性有条件地启用需要的额外功能`std`.

[`no_std`]: ../../reference/names/preludes.html#the-no_std-attribute

[`wasm-bindgen`]: https://crates.io/crates/wasm-bindgen

[`std` prelude]: ../../std/prelude/index.html

[wasm-bindgen-std]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L25

[wasm-bindgen-default]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/Cargo.toml#L23

[wasm-bindgen-no_std]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L8

[wasm-bindgen-cfg1]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L270-L273

[wasm-bindgen-cfg2]: https://github.com/rustwasm/wasm-bindgen/blob/0.2.69/src/lib.rs#L67-L75

### Re-exporting dependency features

可以方便地从依赖项重新导出功能。这允许依赖于板条箱的用户控制这些特性，而无需直接指定这些依赖项。例如[`regex`][re-exports the features][regex-re-export]从[`regex_syntax`][regex_syntax-features]包裹用户`regex`不需要知道`regex_syntax`但他们仍然可以访问它包含的功能。

[regex-re-export]: https://github.com/rust-lang/regex/blob/1.4.2/Cargo.toml#L65-L89

[regex_syntax-features]: https://github.com/rust-lang/regex/blob/1.4.2/regex-syntax/Cargo.toml#L17-L32

### Vendoring of C libraries

有些软件包提供到通用C库的绑定（有时称为["sys" crates][sys])。有时，这些软件包让您可以选择使用系统上安装的C库，或者从源代码构建它。例如[`openssl`]包裹上有一个[`vendored` feature][openssl-vendored]这使得相应的`vendored`特点[`openssl-sys`]这个`openssl-sys`构建脚本有一些[conditional logic][openssl-sys-cfg]这导致它从OpenSSL源代码的本地副本构建，而不是使用系统中的版本。

这个[`curl-sys`]包是[`static-curl`
feature][curl-sys-static]使其从源代码构建libcurl。请注意，它还有一个[`force-system-lib-on-osx`][curl-sys-macos]迫使它的特征[to use the system libcurl][curl-sys-macos-code]，覆盖静态卷曲设置。

[`openssl`]: https://crates.io/crates/openssl

[`openssl-sys`]: https://crates.io/crates/openssl-sys

[sys]: build-scripts.md#-sys-packages

[openssl-vendored]: https://github.com/sfackler/rust-openssl/blob/openssl-v0.10.31/openssl/Cargo.toml#L19

[build script]: build-scripts.md

[openssl-sys-cfg]: https://github.com/sfackler/rust-openssl/blob/openssl-v0.10.31/openssl-sys/build/main.rs#L47-L54

[`curl-sys`]: https://crates.io/crates/curl-sys

[curl-sys-static]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/Cargo.toml#L49

[curl-sys-macos]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/Cargo.toml#L52

[curl-sys-macos-code]: https://github.com/alexcrichton/curl-rust/blob/0.4.34/curl-sys/build.rs#L15-L20

### Feature precedence

有些软件包可能具有相互排斥的功能。处理这一问题的一个选择是选择一个特性而不是另一个。这个[`log`]包就是一个例子。是的[several features][log-features]用于在编译时选择所述的最大日志记录级别[here][log-docs].它使用[`cfg-if`]到[choose a
precedence][log-cfg-if]。如果启用了多个功能，则与较低级别相比，更高的“最大”级别将更受欢迎。

[`log`]: https://crates.io/crates/log

[log-features]: https://github.com/rust-lang/log/blob/0.4.11/Cargo.toml#L29-L42

[log-docs]: https://docs.rs/log/0.4.11/log/#compile-time-filters

[log-cfg-if]: https://github.com/rust-lang/log/blob/0.4.11/src/lib.rs#L1422-L1448

[`cfg-if`]: https://crates.io/crates/cfg-if

### Proc-macro companion package

有些软件包有一个与之密切相关的proc宏。但是，并非所有用户都需要使用proc宏。通过将proc宏设置为可选依赖项，可以方便地选择是否包含它。这很有帮助，因为有时proc宏版本必须与父包保持同步，并且您不希望强制用户必须指定这两个依赖项并保持它们同步。

例如[`serde`]它有一个[`derive`][serde-derive]使[`serde_derive`]过程宏。这个`serde_derive`板条箱被紧紧地绑在一起`serde`，所以它使用[equals version
requirement][serde-equals]以确保它们保持同步。

[`serde`]: https://crates.io/crates/serde

[`serde_derive`]: https://crates.io/crates/serde_derive

[serde-derive]: https://github.com/serde-rs/serde/blob/v1.0.118/serde/Cargo.toml#L34-L35

[serde-equals]: https://github.com/serde-rs/serde/blob/v1.0.118/serde/Cargo.toml#L17

### Nightly-only features

一些软件包想要尝试API或语言功能，这些功能只在Rust上可用[nightly channel].但是，他们可能不想要求用户也使用夜间频道。例如[`wasm-bindgen`]它有一个[`nightly` feature][wasm-bindgen-nightly]这使得[extended API][wasm-bindgen-unsize]使用[`Unsize`]在撰写本文时，只在夜间频道提供的标记特征。

请注意，在它使用的板条箱的底部[`cfg_attr` to enable the nightly
feature][wasm-bindgen-cfg_attr].记住[`feature` attribute]与Cargo特性无关，用于选择实验性语言特性。

这个[`simd_support` feature][rand-simd_support]关于[`rand`]package是另一个例子，它依赖于只在夜间频道上构建的依赖项。

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

有些软件包有新的功能，他们可能想试用，而不必保证这些API的稳定性。这些特性通常被记录为实验性的，因此将来可能会改变或中断，即使是在一个小版本中。例如[`async-std`]包，它有一个[`unstable` feature][async-std-unstable]哪一个[gates
new APIs][async-std-gate]人们可以选择使用，但可能还没有完全准备好被依赖。

[`async-std`]: https://crates.io/crates/async-std

[async-std-unstable]: https://github.com/async-rs/async-std/blob/v1.8.0/Cargo.toml#L38-L42

[async-std-gate]: https://github.com/async-rs/async-std/blob/v1.8.0/src/macros.rs#L46
