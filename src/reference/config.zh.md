## Configuration

本文档将解释Cargo的配置系统如何工作,以及可用的密钥或配置.有关通过其清单配置程序包的信息,请参阅[清单格式](reference/manifest.md).

### Hierarchical structure

Cargo允许特定包的本地配置以及全局配置,如git.Cargo将其扩展为分层策略.例如,如果调用Cargo`/projects/foo/bar/baz`,然后将按以下顺序探测和统一以下配置文件:

-   `/projects/foo/bar/baz/.cargo/config`
-   `/projects/foo/bar/.cargo/config`
-   `/projects/foo/.cargo/config`
-   `/projects/.cargo/config`
-   `/.cargo/config`
-   `$HOME/.cargo/config`

使用此结构,您可以指定每个包的配置,甚至可以将其检入版本控制.您还可以在主目录中使用配置文件指定个人默认值.

### Configuration format

所有配置目前都在[TOML格式][toml](与清单一样),在部分(表)内部使用简单的键值对,它们都被合并在一起.

[toml]: https://github.com/toml-lang/toml

### Configuration keys

以下所有键都是可选的,除非另有说明,否则它们的默认值将列为其值.

指定工具的键值可以作为绝对路径,相对路径或无路径工具名称给出.绝对路径和无路径工具名称用于给定.相对路径相对于父目录解析`.cargo`值所在的配置文件的目录.

```toml
# An array of paths to local repositories which are to be used as overrides for
# dependencies. For more information see the Specifying Dependencies guide.
paths = ["/path/to/override"]

[cargo-new]
# This is your name/email to place in the `authors` section of a new Cargo.toml
# that is generated. If not present, then `git` will be probed, and if that is
# not present then `$USER` and `$EMAIL` will be used.
name = "..."
email = "..."

# By default `cargo new` will initialize a new Git repository. This key can be
# set to `hg` to create a Mercurial repository, or `none` to disable this
# behavior.
vcs = "none"

# For the following sections, $triple refers to any valid target triple, not the
# literal string "$triple", and it will apply whenever that target triple is
# being compiled to. 'cfg(...)' refers to the Rust-like `#[cfg]` syntax for
# conditional compilation.
[target.$triple]
# This is the linker which is passed to rustc (via `-C linker=`) when the `$triple`
# is being compiled for. By default this flag is not passed to the compiler.
linker = ".."
# Same but for the library archiver which is passed to rustc via `-C ar=`.
ar = ".."
# If a runner is provided, compiled targets for the `$triple` will be executed
# by invoking the specified runner executable with actual target as first argument.
# This applies to `cargo run`, `cargo test` and `cargo bench` commands.
# By default compiled targets are executed directly.
runner = ".."
# custom flags to pass to all compiler invocations that target $triple
# this value overrides build.rustflags when both are present
rustflags = ["..", ".."]

[target.'cfg(...)']
# Similar for the $triple configuration, but using the `cfg` syntax.
# If several `cfg` and $triple targets are candidates, then the rustflags
# are concatenated. The `cfg` syntax only applies to rustflags, and not to
# linker.
rustflags = ["..", ".."]
# Similar for the $triple configuration, but using the `cfg` syntax.
# If one or more `cfg`s, and a $triple target are candidates, then the $triple
# will be used
# If several `cfg` are candidates, then the build will error
runner = ".."

# Configuration keys related to the registry
[registry]
index = "..."   # URL of the registry index (defaults to the central repository)
token = "..."   # Access token (found on the central repo’s website)
default = "..." # Default alternative registry to use (can be overriden with --registry)

[http]
proxy = "host:port" # HTTP proxy to use for HTTP requests (defaults to none)
                    # in libcurl format, e.g. "socks5h://host:port"
timeout = 30        # Timeout for each HTTP request, in seconds
cainfo = "cert.pem" # Path to Certificate Authority (CA) bundle (optional)
check-revoke = true # Indicates whether SSL certs are checked for revocation
low-speed-limit = 5 # Lower threshold for bytes/sec (10 = default, 0 = disabled)
multiplexing = true # whether or not to use HTTP/2 multiplexing where possible

# This setting can be used to help debug what's going on with HTTP requests made
# by Cargo. When set to `true` then Cargo's normal debug logging will be filled
# in with HTTP information, which you can extract with
# `RUST_LOG=cargo::ops::registry=debug` (and `trace` may print more).
#
# Be wary when posting these logs elsewhere though, it may be the case that a
# header has an authentication token in it you don't want leaked! Be sure to
# briefly review logs before posting them.
debug = false

[build]
jobs = 1                  # number of parallel jobs, defaults to # of CPUs
rustc = "rustc"           # the rust compiler tool
rustdoc = "rustdoc"       # the doc generator tool
target = "triple"         # build for the target triple (ignored by `cargo install`)
target-dir = "target"     # path of where to place all generated artifacts
rustflags = ["..", ".."]  # custom flags to pass to all compiler invocations
incremental = true        # whether or not to enable incremental compilation
dep-info-basedir = ".."   # full path for the base directory for targets in depfiles

[term]
verbose = false        # whether cargo provides verbose output
color = 'auto'         # whether cargo colorizes output

# Network configuration
[net]
retry = 2 # number of times a network call will automatically retried
git-fetch-with-cli = false  # if `true` we'll use `git`-the-CLI to fetch git repos

# Alias cargo commands. The first 3 aliases are built in. If your
# command requires grouped whitespace use the list format.
[alias]
b = "build"
t = "test"
r = "run"
rr = "run --release"
space_example = ["run", "--release", "--", "\"command list\""]
```

### Environment variables

除了上面的TOML语法之外,还可以通过环境变量配置Cargo.对于表单上方的每个配置键`foo.bar`环境变量`CARGO_FOO_BAR`也可以用来定义值.比如说`build.jobs`key也可以定义`CARGO_BUILD_JOBS`.

环境变量将优先于TOML配置,并且当前仅支持由环境变量定义的整数,布尔和字符串键.这意味着[来源更换][source],由表表示,不能通过环境变量配置.

除上述系统外,Cargo还认可其他一些特定的系统[环境变量][env].

[env]: reference/environment-variables.md

[source]: reference/source-replacement.md
