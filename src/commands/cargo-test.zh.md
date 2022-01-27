# cargo-test(1)

## NAME

Cargo测试——执行一个包的单元测试和集成测试

## SYNOPSIS

`cargo test` [*options*][_testname_] [`--` _test-options_]

## DESCRIPTION

编译并执行单元和集成测试。

测试过滤参数`TESTNAME`以及两个破折号后面的所有论点(`--`)传递给测试二进制文件，从而传递给*利贝斯特*（rustc的内置单元测试和微观基准测试框架）。如果您同时向Cargo和二进制文件传递参数，则后面的参数`--`进入二进制，在进入Cargo之前。有关libtest参数的详细信息，请参阅`cargo test -- --help`看看rustc书中关于测试如何在实验室工作的章节<https://doc.rust-lang.org/rustc/tests/index.html>.

例如，这将筛选具有`foo`以它们的名义在3个线程上并行运行：

```
cargo test foo -- --test-threads 3
```

测试是用`--test`选择`rustc`它创建了一个具有`main`自动运行所有带注释的函数的函数`#[test]`属性在多个线程中。`#[bench]`带注释的函数也将在一次迭代中运行，以验证它们的功能。

libtest线束可以通过设置禁用`harness = false`在目标清单设置中，在这种情况下，代码需要提供自己的`main`函数来处理正在运行的测试。

文档测试也默认运行，由`rustdoc`.它从文档注释中提取代码样本并执行它们。看到了吗[rustdoc book](https://doc.rust-lang.org/rustdoc/)有关编写文档测试的更多信息。

## OPTIONS

### Test Options

<dl>

<dt class="option-term" id="option-cargo-test---no-run"><a class="option-anchor" href="#option-cargo-test---no-run"></a><code>--no-run</code></dt>
<dd class="option-desc">Compile, but don't run tests.</dd>

<dt class="option-term" id="option-cargo-test---no-fail-fast"><a class="option-anchor" href="#option-cargo-test---no-fail-fast"></a><code>--no-fail-fast</code></dt>
<dd class="option-desc">Run all tests regardless of failure. Without this flag, Cargo will exit
after the first executable fails. The Rust test harness will run all tests
within the executable to completion, this flag only applies to the executable
as a whole.</dd>

</dl>

### Package Selection

默认情况下，如果未提供包选择选项，则选择的包取决于选择的清单文件（如果需要，则基于当前工作目录）`--manifest-path`未给出）。如果清单是工作区的根目录，则会选择工作区默认成员，否则只会选择清单定义的包。

可以使用`workspace.default-members`输入根清单。如果未设置，虚拟工作区将包括所有工作区成员（相当于传递）`--workspace`)，非虚拟工作区将只包括根板条箱本身。

<dl>

<dt class="option-term" id="option-cargo-test--p"><a class="option-anchor" href="#option-cargo-test--p"></a><code>-p</code> <em>spec</em>...</dt>
<dt class="option-term" id="option-cargo-test---package"><a class="option-anchor" href="#option-cargo-test---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Test only the specified packages. See <a href="cargo-pkgid.html">cargo-pkgid(1)</a> for the
SPEC format. This flag may be specified multiple times and supports common Unix
glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell accidentally 
expanding glob patterns before Cargo handles them, you must use single quotes or
double quotes around each pattern.</dd>

<dt class="option-term" id="option-cargo-test---workspace"><a class="option-anchor" href="#option-cargo-test---workspace"></a><code>--workspace</code></dt>
<dd class="option-desc">Test all members in the workspace.</dd>

<dt class="option-term" id="option-cargo-test---all"><a class="option-anchor" href="#option-cargo-test---all"></a><code>--all</code></dt>
<dd class="option-desc">Deprecated alias for <code>--workspace</code>.</dd>

<dt class="option-term" id="option-cargo-test---exclude"><a class="option-anchor" href="#option-cargo-test---exclude"></a><code>--exclude</code> <em>SPEC</em>...</dt>
<dd class="option-desc">Exclude the specified packages. Must be used in conjunction with the
<code>--workspace</code> flag. This flag may be specified multiple times and supports
common Unix glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell
accidentally expanding glob patterns before Cargo handles them, you must use
single quotes or double quotes around each pattern.</dd>

</dl>

### Target Selection

当没有给出目标选择选项时，`cargo test`将为选定的软件包建立以下目标：

-   lib-用于链接二进制文件、示例、集成测试和文档测试
-   垃圾箱（仅当集成测试已构建且所需功能可用时）
-   示例-确保它们能够编译
-   作为单元测试的lib
-   作为单元测试的箱子
-   集成测试
-   lib目标的doc测试

可以通过设置`test`清单设置中目标的标志。以身作则`test = true`将生成并运行示例作为测试。设定目标`test = false`将阻止默认情况下对其进行测试。按名称获取目标的目标选择选项忽略`test`标记并始终测试给定的目标。

可以通过设置禁用库的文档测试`doctest = false`对于清单中的库。

如果存在集成测试或基准测试，则会自动构建二进制目标。这允许集成测试执行二进制文件来练习和测试其行为。这个`CARGO_BIN_EXE_<name>`
[environment variable](../reference/environment-variables.html#environment-variables-cargo-sets-for-crates)在构建集成测试时设置，以便它可以使用[`env` macro](https://doc.rust-lang.org/std/macro.env.html)找到可执行文件。

传递目标选择标志将仅测试指定的目标。

注意`--bin`, `--example`, `--test`和`--bench`标志还支持常见的Unix全局模式，如`*`, `?`和`[]`但是，为了避免您的shell在Cargo处理glob模式之前意外地扩展glob模式，您必须在每个glob模式周围使用单引号或双引号。

<dl>

<dt class="option-term" id="option-cargo-test---lib"><a class="option-anchor" href="#option-cargo-test---lib"></a><code>--lib</code></dt>
<dd class="option-desc">Test the package's library.</dd>

<dt class="option-term" id="option-cargo-test---bin"><a class="option-anchor" href="#option-cargo-test---bin"></a><code>--bin</code> <em>name</em>...</dt>
<dd class="option-desc">Test the specified binary. This flag may be specified multiple times
and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-test---bins"><a class="option-anchor" href="#option-cargo-test---bins"></a><code>--bins</code></dt>
<dd class="option-desc">Test all binary targets.</dd>

<dt class="option-term" id="option-cargo-test---example"><a class="option-anchor" href="#option-cargo-test---example"></a><code>--example</code> <em>name</em>...</dt>
<dd class="option-desc">Test the specified example. This flag may be specified multiple times
and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-test---examples"><a class="option-anchor" href="#option-cargo-test---examples"></a><code>--examples</code></dt>
<dd class="option-desc">Test all example targets.</dd>

<dt class="option-term" id="option-cargo-test---test"><a class="option-anchor" href="#option-cargo-test---test"></a><code>--test</code> <em>name</em>...</dt>
<dd class="option-desc">Test the specified integration test. This flag may be specified
multiple times and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-test---tests"><a class="option-anchor" href="#option-cargo-test---tests"></a><code>--tests</code></dt>
<dd class="option-desc">Test all targets in test mode that have the <code>test = true</code> manifest
flag set. By default this includes the library and binaries built as
unittests, and integration tests. Be aware that this will also build any
required dependencies, so the lib target may be built twice (once as a
unittest, and once as a dependency for binaries, integration tests, etc.).
Targets may be enabled or disabled by setting the <code>test</code> flag in the
manifest settings for the target.</dd>

<dt class="option-term" id="option-cargo-test---bench"><a class="option-anchor" href="#option-cargo-test---bench"></a><code>--bench</code> <em>name</em>...</dt>
<dd class="option-desc">Test the specified benchmark. This flag may be specified multiple
times and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-test---benches"><a class="option-anchor" href="#option-cargo-test---benches"></a><code>--benches</code></dt>
<dd class="option-desc">Test all targets in benchmark mode that have the <code>bench = true</code>
manifest flag set. By default this includes the library and binaries built
as benchmarks, and bench targets. Be aware that this will also build any
required dependencies, so the lib target may be built twice (once as a
benchmark, and once as a dependency for binaries, benchmarks, etc.).
Targets may be enabled or disabled by setting the <code>bench</code> flag in the
manifest settings for the target.</dd>

<dt class="option-term" id="option-cargo-test---all-targets"><a class="option-anchor" href="#option-cargo-test---all-targets"></a><code>--all-targets</code></dt>
<dd class="option-desc">Test all targets. This is equivalent to specifying <code>--lib --bins --tests --benches --examples</code>.</dd>

</dl>

<dl>

<dt class="option-term" id="option-cargo-test---doc"><a class="option-anchor" href="#option-cargo-test---doc"></a><code>--doc</code></dt>
<dd class="option-desc">Test only the library's documentation. This cannot be mixed with other
target options.</dd>

</dl>

### Feature Selection

功能标志允许您控制启用哪些功能。如果未提供任何功能选项，则`default`为每个选定的软件包激活该功能。

看见[the features documentation](../reference/features.html#command-line-feature-options)更多细节。

<dl>

<dt class="option-term" id="option-cargo-test---features"><a class="option-anchor" href="#option-cargo-test---features"></a><code>--features</code> <em>features</em></dt>
<dd class="option-desc">Space or comma separated list of features to activate. Features of workspace
members may be enabled with <code>package-name/feature-name</code> syntax. This flag may
be specified multiple times, which enables all specified features.</dd>

<dt class="option-term" id="option-cargo-test---all-features"><a class="option-anchor" href="#option-cargo-test---all-features"></a><code>--all-features</code></dt>
<dd class="option-desc">Activate all available features of all selected packages.</dd>

<dt class="option-term" id="option-cargo-test---no-default-features"><a class="option-anchor" href="#option-cargo-test---no-default-features"></a><code>--no-default-features</code></dt>
<dd class="option-desc">Do not activate the <code>default</code> feature of the selected packages.</dd>

</dl>

### Compilation Options

<dl>

<dt class="option-term" id="option-cargo-test---target"><a class="option-anchor" href="#option-cargo-test---target"></a><code>--target</code> <em>triple</em></dt>
<dd class="option-desc">Test for the given architecture. The default is the host
architecture. The general format of the triple is
<code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>. Run <code>rustc --print target-list</code> for a
list of supported targets.</p>
<p>This may also be specified with the <code>build.target</code>
<a href="../reference/config.html">config value</a>.</p>
<p>Note that specifying this flag makes Cargo run in a different mode where the
target artifacts are placed in a separate directory. See the
<a href="../guide/build-cache.html">build cache</a> documentation for more details.</dd>

<dt class="option-term" id="option-cargo-test--r"><a class="option-anchor" href="#option-cargo-test--r"></a><code>-r</code></dt>
<dt class="option-term" id="option-cargo-test---release"><a class="option-anchor" href="#option-cargo-test---release"></a><code>--release</code></dt>
<dd class="option-desc">Test optimized artifacts with the <code>release</code> profile.
See also the <code>--profile</code> option for choosing a specific profile by name.</dd>

<dt class="option-term" id="option-cargo-test---profile"><a class="option-anchor" href="#option-cargo-test---profile"></a><code>--profile</code> <em>name</em></dt>
<dd class="option-desc">Test with the given profile.
See the <a href="../reference/profiles.html">the reference</a> for more details on profiles.</dd>

<dt class="option-term" id="option-cargo-test---ignore-rust-version"><a class="option-anchor" href="#option-cargo-test---ignore-rust-version"></a><code>--ignore-rust-version</code></dt>
<dd class="option-desc">Test the target even if the selected Rust compiler is older than the
required Rust version as configured in the project's <code>rust-version</code> field.</dd>

</dl>

### Output Options

<dl>
<dt class="option-term" id="option-cargo-test---target-dir"><a class="option-anchor" href="#option-cargo-test---target-dir"></a><code>--target-dir</code> <em>directory</em></dt>
<dd class="option-desc">Directory for all generated artifacts and intermediate files. May also be
specified with the <code>CARGO_TARGET_DIR</code> environment variable, or the
<code>build.target-dir</code> <a href="../reference/config.html">config value</a>.
Defaults to <code>target</code> in the root of the workspace.</dd>

</dl>

### Display Options

默认情况下，Rust测试线束会隐藏测试执行的输出，以保持结果的可读性。测试输出可以通过以下方式恢复（例如，用于调试）`--nocapture`对于测试二进制文件：

```
cargo test -- --nocapture
```

<dl>

<dt class="option-term" id="option-cargo-test--v"><a class="option-anchor" href="#option-cargo-test--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-test---verbose"><a class="option-anchor" href="#option-cargo-test---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-test--q"><a class="option-anchor" href="#option-cargo-test--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-test---quiet"><a class="option-anchor" href="#option-cargo-test---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-test---color"><a class="option-anchor" href="#option-cargo-test---color"></a><code>--color</code> <em>when</em></dt>
<dd class="option-desc">Control when colored output is used. Valid values:</p>
<ul>
<li><code>auto</code> (default): Automatically detect if color support is available on the
terminal.</li>
<li><code>always</code>: Always display colors.</li>
<li><code>never</code>: Never display colors.</li>
</ul>
<p>May also be specified with the <code>term.color</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-test---message-format"><a class="option-anchor" href="#option-cargo-test---message-format"></a><code>--message-format</code> <em>fmt</em></dt>
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

<dt class="option-term" id="option-cargo-test---manifest-path"><a class="option-anchor" href="#option-cargo-test---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-test---frozen"><a class="option-anchor" href="#option-cargo-test---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-test---locked"><a class="option-anchor" href="#option-cargo-test---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-test---offline"><a class="option-anchor" href="#option-cargo-test---offline"></a><code>--offline</code></dt>
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

<dt class="option-term" id="option-cargo-test-+toolchain"><a class="option-anchor" href="#option-cargo-test-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-test--h"><a class="option-anchor" href="#option-cargo-test--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-test---help"><a class="option-anchor" href="#option-cargo-test---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-test--Z"><a class="option-anchor" href="#option-cargo-test--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

### Miscellaneous Options

这个`--jobs`参数影响测试可执行文件的构建，但不影响运行测试时使用的线程数。防锈测试线束包括一个控制所用螺纹数量的选项：

```
cargo test -j 2 -- --test-threads=2
```

<dl>

<dt class="option-term" id="option-cargo-test--j"><a class="option-anchor" href="#option-cargo-test--j"></a><code>-j</code> <em>N</em></dt>
<dt class="option-term" id="option-cargo-test---jobs"><a class="option-anchor" href="#option-cargo-test---jobs"></a><code>--jobs</code> <em>N</em></dt>
<dd class="option-desc">Number of parallel jobs to run. May also be specified with the
<code>build.jobs</code> <a href="../reference/config.html">config value</a>. Defaults to
the number of CPUs.</dd>

<dt class="option-term" id="option-cargo-test---future-incompat-report"><a class="option-anchor" href="#option-cargo-test---future-incompat-report"></a><code>--future-incompat-report</code></dt>
<dd class="option-desc">Displays a future-incompat report for any future-incompatible warnings
produced during execution of this command</p>
<p>See <a href="cargo-report.html">cargo-report(1)</a></dd>

</dl>

## ENVIRONMENT

看见[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  执行当前软件包的所有单元和集成测试：

    Cargo测试

2.  仅运行名称与筛选器字符串匹配的测试：

    Cargo测试名称\\u过滤器

3.  仅在特定集成测试中运行特定测试：

    Cargo测试--test int_test_name--modname:：test_name

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-bench(1)](cargo-bench.html)
