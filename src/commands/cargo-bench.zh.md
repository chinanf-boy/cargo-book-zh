# cargo-bench(1)

## NAME

cargo-bench - 执行箱子的基准测试

## SYNOPSIS

`cargo bench` [_options_] [_benchname_] [`--` _bench-options_]

## DESCRIPTION

编译并执行基准测试。

基准测试的筛选参数 `benchname` 以及，两个破折号(`--`)后面的所有参数，都传递给基准测试的二进制文件，从而传递给 `libtest`（rustc 的内置单元测试和微基准测试框架）。如果您同时向 Cargo 和二进制文件传递参数，那么，`--`后面的参数给到二进制文件，而之前的给到 Cargo 。有关 libtest 参数的详细信息，请参阅`cargo bench -- --help`看看 rustc 书中，关于测试如何工作的 <https://doc.rust-lang.org/rustc/tests/index.html>.

例如，下面的示例中，只运行名为`foo`（并跳过，其他类似名称的基准测试，如`foobar`):

```
cargo bench -- foo --exact
```

基准测试是用`rustc`的`--test`选项，它创建了一个`main`函数，自动运行所有带`#[bench]`属性函数，Cargo 传递`--bench`给测试工具，告诉它只运行基准测试。

libtest harness 可以禁用，需要通过设置`harness = false`，在这种情况下，代码需要提供自己的`main`函数，来处理运行中的基准测试。

> **注意：** [`#[bench]` attribute](https://doc.rust-lang.org/nightly/unstable-book/library-features/test.html)目前不稳定，仅在[nightly channel](https://doc.rust-lang.org/book/appendix-07-nightly-rust.html)可用。[crates.io](https://crates.io/keywords/benchmark)上会有些包，能够帮助你在 stable channel 上运行基准测试，例如，[Criterion](https://crates.io/crates/criterion)。

默认情况下，`cargo bench`使用[`bench` profile]，启用了优化，并禁用调试信息。如果需要调试一个基准测试，可以使用`--profile=dev`命令行选项，切换到 dev profile。然后，可以在调试器中，运行启用调试的基准测试。

[`bench` profile]: ../reference/profiles.html#bench

## OPTIONS

### Benchmark Options

<dl>

<dt class="option-term" id="option-cargo-bench---no-run"><a class="option-anchor" href="#option-cargo-bench---no-run"></a><code>--no-run</code></dt>
<dd class="option-desc">编译，但不运行基准测试</dd>

<dt class="option-term" id="option-cargo-bench---no-fail-fast"><a class="option-anchor" href="#option-cargo-bench---no-fail-fast"></a><code>--no-fail-fast</code></dt>
<dd class="option-desc">不管失败的，运行所有基准测试。若是没有这个选项，在第一次基准测试失败， Cargo 会退出。The Rust test harness 会运行可执行文件，直到所有的基准测试完成，这个选项仅应用到完整的可执行文件。</dd>

</dl>

### Package Selection

默认情况下，如果未提供包选择选项，则选择的包取决于选择的清单文件（如果需要，则基于当前工作目录）`--manifest-path`未给出）。如果清单是工作区的根目录，则会选择工作区默认成员，否则只会选择清单定义的包。

可以使用`workspace.default-members`输入根清单。如果未设置，虚拟工作区将包括所有工作区成员（相当于传递）`--workspace`)，非虚拟工作区将只包括根箱子本身。

<dl>

<dt class="option-term" id="option-cargo-bench--p"><a class="option-anchor" href="#option-cargo-bench--p"></a><code>-p</code> <em>spec</em>...</dt>
<dt class="option-term" id="option-cargo-bench---package"><a class="option-anchor" href="#option-cargo-bench---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Benchmark only the specified packages. See <a href="cargo-pkgid.html">cargo-pkgid(1)</a> for the
SPEC format. This flag may be specified multiple times and supports common Unix
glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell accidentally 
expanding glob patterns before Cargo handles them, you must use single quotes or
double quotes around each pattern.</dd>

<dt class="option-term" id="option-cargo-bench---workspace"><a class="option-anchor" href="#option-cargo-bench---workspace"></a><code>--workspace</code></dt>
<dd class="option-desc">Benchmark all members in the workspace.</dd>

<dt class="option-term" id="option-cargo-bench---all"><a class="option-anchor" href="#option-cargo-bench---all"></a><code>--all</code></dt>
<dd class="option-desc">Deprecated alias for <code>--workspace</code>.</dd>

<dt class="option-term" id="option-cargo-bench---exclude"><a class="option-anchor" href="#option-cargo-bench---exclude"></a><code>--exclude</code> <em>SPEC</em>...</dt>
<dd class="option-desc">Exclude the specified packages. Must be used in conjunction with the
<code>--workspace</code> flag. This flag may be specified multiple times and supports
common Unix glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell
accidentally expanding glob patterns before Cargo handles them, you must use
single quotes or double quotes around each pattern.</dd>

</dl>

### Target Selection

当没有给出目标选择选项时，`cargo bench`将为选定的软件包建立以下目标：

- lib-用于链接二进制文件和基准测试
- BIN（仅当建立了基准目标且所需功能可用时）
- lib 作为基准
- 以垃圾箱为基准
- 基准目标

可以通过设置`bench`清单设置中目标的标志。以身作则`bench = true`将构建并运行该示例作为基准。设定目标`bench = false`将阻止它们在默认情况下成为基准。按名称获取目标的目标选择选项忽略`bench`标记并始终对给定目标进行基准测试。

传递目标选择标志将仅对指定的目标进行基准测试。

注意`--bin`, `--example`, `--test`和`--bench`标志还支持常见的 Unix 全局模式，如`*`, `?`和`[]`但是，为了避免您的 shell 在 Cargo 处理 glob 模式之前意外地扩展 glob 模式，您必须在每个 glob 模式周围使用单引号或双引号。

<dl>

<dt class="option-term" id="option-cargo-bench---lib"><a class="option-anchor" href="#option-cargo-bench---lib"></a><code>--lib</code></dt>
<dd class="option-desc">Benchmark the package's library.</dd>

<dt class="option-term" id="option-cargo-bench---bin"><a class="option-anchor" href="#option-cargo-bench---bin"></a><code>--bin</code> <em>name</em>...</dt>
<dd class="option-desc">Benchmark the specified binary. This flag may be specified multiple times
and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-bench---bins"><a class="option-anchor" href="#option-cargo-bench---bins"></a><code>--bins</code></dt>
<dd class="option-desc">Benchmark all binary targets.</dd>

<dt class="option-term" id="option-cargo-bench---example"><a class="option-anchor" href="#option-cargo-bench---example"></a><code>--example</code> <em>name</em>...</dt>
<dd class="option-desc">Benchmark the specified example. This flag may be specified multiple times
and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-bench---examples"><a class="option-anchor" href="#option-cargo-bench---examples"></a><code>--examples</code></dt>
<dd class="option-desc">Benchmark all example targets.</dd>

<dt class="option-term" id="option-cargo-bench---test"><a class="option-anchor" href="#option-cargo-bench---test"></a><code>--test</code> <em>name</em>...</dt>
<dd class="option-desc">Benchmark the specified integration test. This flag may be specified
multiple times and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-bench---tests"><a class="option-anchor" href="#option-cargo-bench---tests"></a><code>--tests</code></dt>
<dd class="option-desc">Benchmark all targets in test mode that have the <code>test = true</code> manifest
flag set. By default this includes the library and binaries built as
unittests, and integration tests. Be aware that this will also build any
required dependencies, so the lib target may be built twice (once as a
unittest, and once as a dependency for binaries, integration tests, etc.).
Targets may be enabled or disabled by setting the <code>test</code> flag in the
manifest settings for the target.</dd>

<dt class="option-term" id="option-cargo-bench---bench"><a class="option-anchor" href="#option-cargo-bench---bench"></a><code>--bench</code> <em>name</em>...</dt>
<dd class="option-desc">Benchmark the specified benchmark. This flag may be specified multiple
times and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-bench---benches"><a class="option-anchor" href="#option-cargo-bench---benches"></a><code>--benches</code></dt>
<dd class="option-desc">Benchmark all targets in benchmark mode that have the <code>bench = true</code>
manifest flag set. By default this includes the library and binaries built
as benchmarks, and bench targets. Be aware that this will also build any
required dependencies, so the lib target may be built twice (once as a
benchmark, and once as a dependency for binaries, benchmarks, etc.).
Targets may be enabled or disabled by setting the <code>bench</code> flag in the
manifest settings for the target.</dd>

<dt class="option-term" id="option-cargo-bench---all-targets"><a class="option-anchor" href="#option-cargo-bench---all-targets"></a><code>--all-targets</code></dt>
<dd class="option-desc">Benchmark all targets. This is equivalent to specifying <code>--lib --bins --tests --benches --examples</code>.</dd>

</dl>

### Feature Selection

功能标志允许您控制启用哪些功能。如果未提供任何功能选项，则`default`为每个选定的软件包激活该功能。

查看[the features documentation](../reference/features.html#command-line-feature-options)更多细节。

<dl>

<dt class="option-term" id="option-cargo-bench---features"><a class="option-anchor" href="#option-cargo-bench---features"></a><code>--features</code> <em>features</em></dt>
<dd class="option-desc">Space or comma separated list of features to activate. Features of workspace
members may be enabled with <code>package-name/feature-name</code> syntax. This flag may
be specified multiple times, which enables all specified features.</dd>

<dt class="option-term" id="option-cargo-bench---all-features"><a class="option-anchor" href="#option-cargo-bench---all-features"></a><code>--all-features</code></dt>
<dd class="option-desc">Activate all available features of all selected packages.</dd>

<dt class="option-term" id="option-cargo-bench---no-default-features"><a class="option-anchor" href="#option-cargo-bench---no-default-features"></a><code>--no-default-features</code></dt>
<dd class="option-desc">Do not activate the <code>default</code> feature of the selected packages.</dd>

</dl>

### Compilation Options

<dl>

<dt class="option-term" id="option-cargo-bench---target"><a class="option-anchor" href="#option-cargo-bench---target"></a><code>--target</code> <em>triple</em></dt>
<dd class="option-desc">Benchmark for the given architecture. The default is the host
architecture. The general format of the triple is
<code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>. Run <code>rustc --print target-list</code> for a
list of supported targets.</p>
<p>This may also be specified with the <code>build.target</code>
<a href="../reference/config.html">config value</a>.</p>
<p>Note that specifying this flag makes Cargo run in a different mode where the
target artifacts are placed in a separate directory. See the
<a href="../guide/build-cache.html">build cache</a> documentation for more details.</dd>

<dt class="option-term" id="option-cargo-bench---profile"><a class="option-anchor" href="#option-cargo-bench---profile"></a><code>--profile</code> <em>name</em></dt>
<dd class="option-desc">Benchmark with the given profile.
See the <a href="../reference/profiles.html">the reference</a> for more details on profiles.</dd>

<dt class="option-term" id="option-cargo-bench---ignore-rust-version"><a class="option-anchor" href="#option-cargo-bench---ignore-rust-version"></a><code>--ignore-rust-version</code></dt>
<dd class="option-desc">Benchmark the target even if the selected Rust compiler is older than the
required Rust version as configured in the project's <code>rust-version</code> field.</dd>

</dl>

### Output Options

<dl>
<dt class="option-term" id="option-cargo-bench---target-dir"><a class="option-anchor" href="#option-cargo-bench---target-dir"></a><code>--target-dir</code> <em>directory</em></dt>
<dd class="option-desc">Directory for all generated artifacts and intermediate files. May also be
specified with the <code>CARGO_TARGET_DIR</code> environment variable, or the
<code>build.target-dir</code> <a href="../reference/config.html">config value</a>.
Defaults to <code>target</code> in the root of the workspace.</dd>

</dl>

### Display Options

默认情况下，Rust 测试工具会隐藏基准测试执行的输出，以保持结果的可读性。基准测试输出可以通过以下方式恢复（例如，用于调试）`--nocapture`对于基准二进制文件：

```
cargo bench -- --nocapture
```

<dl>

<dt class="option-term" id="option-cargo-bench--v"><a class="option-anchor" href="#option-cargo-bench--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-bench---verbose"><a class="option-anchor" href="#option-cargo-bench---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-bench--q"><a class="option-anchor" href="#option-cargo-bench--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-bench---quiet"><a class="option-anchor" href="#option-cargo-bench---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-bench---color"><a class="option-anchor" href="#option-cargo-bench---color"></a><code>--color</code> <em>when</em></dt>
<dd class="option-desc">Control when colored output is used. Valid values:</p>
<ul>
<li><code>auto</code> (default): Automatically detect if color support is available on the
terminal.</li>
<li><code>always</code>: Always display colors.</li>
<li><code>never</code>: Never display colors.</li>
</ul>
<p>May also be specified with the <code>term.color</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-bench---message-format"><a class="option-anchor" href="#option-cargo-bench---message-format"></a><code>--message-format</code> <em>fmt</em></dt>
<dd class="option-desc">The output format for diagnostic messages. Can be specified multiple times
and consists of comma-separated values. Valid values:</p>
<ul>
<li><code>human</code> (default): Display in a human-readable text format. Conflicts with
<code>short</code> and <code>json</code>.</li>
<li><code>short</code>: Emit shorter, human-readable text messages. Conflicts with <code>human</code>
and <code>json</code>.</li>
<li><code>json</code>: Emit JSON messages to stdout. See
<a href="../reference/external-tools.html#json-messages">the reference</a>
for more details. Conflicts with <code>human</code> and <code>short</code>.</li>
<li><code>json-diagnostic-short</code>: Ensure the <code>rendered</code> field of JSON messages contains
the &quot;short&quot; rendering from rustc. Cannot be used with <code>human</code> or <code>short</code>.</li>
<li><code>json-diagnostic-rendered-ansi</code>: Ensure the <code>rendered</code> field of JSON messages
contains embedded ANSI color codes for respecting rustc's default color
scheme. Cannot be used with <code>human</code> or <code>short</code>.</li>
<li><code>json-render-diagnostics</code>: Instruct Cargo to not include rustc diagnostics in
in JSON messages printed, but instead Cargo itself should render the
JSON diagnostics coming from rustc. Cargo's own JSON diagnostics and others
coming from rustc are still emitted. Cannot be used with <code>human</code> or <code>short</code>.</li>
</ul></dd>

</dl>

### Manifest Options

<dl>
<dt class="option-term" id="option-cargo-bench---manifest-path"><a class="option-anchor" href="#option-cargo-bench---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-bench---frozen"><a class="option-anchor" href="#option-cargo-bench---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-bench---locked"><a class="option-anchor" href="#option-cargo-bench---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-bench---offline"><a class="option-anchor" href="#option-cargo-bench---offline"></a><code>--offline</code></dt>
<dd class="option-desc">Prevents Cargo from accessing the network for any reason. Without this
flag, Cargo will stop with an error if it needs to access the network and
the network is not available. With this flag, Cargo will attempt to
proceed without the network if possible.</p>
<p>Beware that this may result in different dependency resolution than online
mode. Cargo will restrict itself to crates that are downloaded locally, even
if there might be a newer version as indicated in the local copy of the index.
See the <a href="cargo-fetch.html">cargo-fetch(1)</a> command to download dependencies before going
offline.</p>
<p>May also be specified with the <code>net.offline</code> <a href="../reference/config.html">config value</a>.</dd>

</dl>

### Common Options

<dl>

<dt class="option-term" id="option-cargo-bench-+toolchain"><a class="option-anchor" href="#option-cargo-bench-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-bench--h"><a class="option-anchor" href="#option-cargo-bench--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-bench---help"><a class="option-anchor" href="#option-cargo-bench---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-bench--Z"><a class="option-anchor" href="#option-cargo-bench--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

### Miscellaneous Options

这个`--jobs`参数影响基准可执行文件的构建，但不影响运行基准时使用的线程数。防锈测试 harness 在单螺纹中连续运行。

<dl>
<dt class="option-term" id="option-cargo-bench--j"><a class="option-anchor" href="#option-cargo-bench--j"></a><code>-j</code> <em>N</em></dt>
<dt class="option-term" id="option-cargo-bench---jobs"><a class="option-anchor" href="#option-cargo-bench---jobs"></a><code>--jobs</code> <em>N</em></dt>
<dd class="option-desc">Number of parallel jobs to run. May also be specified with the
<code>build.jobs</code> <a href="../reference/config.html">config value</a>. Defaults to
the number of CPUs.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关 Cargo 读取的环境变量的详细信息。

## EXIT STATUS

- `0`：Cargo 成功。
- `101`：Cargo 未能完成。

## EXAMPLES

1.  构建并执行当前软件包的所有基准：

    货台

2.  仅在特定基准目标内运行特定基准：

    货梯——梯梯台\_名称——modname:：一些\_基准

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-test(1)](cargo-test.html)
