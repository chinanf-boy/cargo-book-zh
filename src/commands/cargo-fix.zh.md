# cargo-fix(1)

## NAME

Cargo修复-自动修复rustc报告的线头警告

## SYNOPSIS

`cargo fix` [_options_]

## DESCRIPTION

这个Cargo子命令将自动从诸如警告之类的诊断中获取rustc的建议，并将其应用到源代码中。这是为了帮助自动化任务，而rustc本身已经知道如何告诉您修复这些任务！

执行`cargo fix`引擎盖下会执行吗[cargo-check(1)](cargo-check.html)。适用于您的箱子的任何警告都将自动修复（如果可能），并且在检查过程完成后，将显示所有剩余的警告。例如，如果要对当前包应用所有修复程序，可以运行：

```
cargo fix
```

其行为与`cargo check --all-targets`.

`cargo fix`只能修复通常使用`cargo check`。如果代码有条件地启用了可选功能，则需要启用这些功能才能分析该代码：

```
cargo fix --features foo
```

同样，其他`cfg`特定于平台的代码等表达式需要传递`--target`修复给定目标的代码。

```
cargo fix --target x86_64-pc-windows-gnu
```

如果你遇到任何问题`cargo fix`如果您有任何问题或功能要求，请随时向<https://github.com/rust-lang/cargo>.

### Edition migration

这个`cargo fix`子命令还可用于从一个[edition]到下一个。一般程序是：

1.  跑`cargo fix --edition`也可以考虑使用`--all-features`如果项目具有多个功能，请标记。你可能也想跑步`cargo fix
    --edition`多次使用不同的`--target`标记您的项目是否具有特定于平台的代码`cfg`属性。
2.  修改`Cargo.toml`设定[edition field]到新版。
3.  运行您的项目测试，以验证一切仍然有效。如果发出新的警告，您可能需要考虑运行。`cargo fix`再次（没有`--edition`标记）以应用编译器给出的任何建议。

希望就是这样！记住上面提到的注意事项`cargo fix`无法更新非活动功能的代码或`cfg`表达。此外，在某些罕见的情况下，编译器无法自动将所有代码迁移到新版本，这可能需要在使用新版本构建后进行手动更改。

[edition]: https://doc.rust-lang.org/edition-guide/editions/transitioning-an-existing-project-to-a-new-edition.html

[edition field]: ../reference/manifest.html#the-edition-field

## OPTIONS

### Fix options

<dl>

<dt class="option-term" id="option-cargo-fix---broken-code"><a class="option-anchor" href="#option-cargo-fix---broken-code"></a><code>--broken-code</code></dt>
<dd class="option-desc">Fix code even if it already has compiler errors. This is useful if <code>cargo fix</code>
fails to apply the changes. It will apply the changes and leave the broken
code in the working directory for you to inspect and manually fix.</dd>

<dt class="option-term" id="option-cargo-fix---edition"><a class="option-anchor" href="#option-cargo-fix---edition"></a><code>--edition</code></dt>
<dd class="option-desc">Apply changes that will update the code to the next edition. This will not
update the edition in the <code>Cargo.toml</code> manifest, which must be updated
manually after <code>cargo fix --edition</code> has finished.</dd>

<dt class="option-term" id="option-cargo-fix---edition-idioms"><a class="option-anchor" href="#option-cargo-fix---edition-idioms"></a><code>--edition-idioms</code></dt>
<dd class="option-desc">Apply suggestions that will update code to the preferred style for the current
edition.</dd>

<dt class="option-term" id="option-cargo-fix---allow-no-vcs"><a class="option-anchor" href="#option-cargo-fix---allow-no-vcs"></a><code>--allow-no-vcs</code></dt>
<dd class="option-desc">Fix code even if a VCS was not detected.</dd>

<dt class="option-term" id="option-cargo-fix---allow-dirty"><a class="option-anchor" href="#option-cargo-fix---allow-dirty"></a><code>--allow-dirty</code></dt>
<dd class="option-desc">Fix code even if the working directory has changes.</dd>

<dt class="option-term" id="option-cargo-fix---allow-staged"><a class="option-anchor" href="#option-cargo-fix---allow-staged"></a><code>--allow-staged</code></dt>
<dd class="option-desc">Fix code even if the working directory has staged changes.</dd>

</dl>

### Package Selection

默认情况下，如果未提供包选择选项，则选择的包取决于选择的清单文件（如果需要，则基于当前工作目录）`--manifest-path`未给出）。如果清单是工作区的根目录，则会选择工作区默认成员，否则只会选择清单定义的包。

可以使用`workspace.default-members`输入根清单。如果未设置，虚拟工作区将包括所有工作区成员（相当于传递）`--workspace`)，非虚拟工作区将只包括根箱子本身。

<dl>

<dt class="option-term" id="option-cargo-fix--p"><a class="option-anchor" href="#option-cargo-fix--p"></a><code>-p</code> <em>spec</em>...</dt>
<dt class="option-term" id="option-cargo-fix---package"><a class="option-anchor" href="#option-cargo-fix---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Fix only the specified packages. See <a href="cargo-pkgid.html">cargo-pkgid(1)</a> for the
SPEC format. This flag may be specified multiple times and supports common Unix
glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell accidentally 
expanding glob patterns before Cargo handles them, you must use single quotes or
double quotes around each pattern.</dd>

<dt class="option-term" id="option-cargo-fix---workspace"><a class="option-anchor" href="#option-cargo-fix---workspace"></a><code>--workspace</code></dt>
<dd class="option-desc">Fix all members in the workspace.</dd>

<dt class="option-term" id="option-cargo-fix---all"><a class="option-anchor" href="#option-cargo-fix---all"></a><code>--all</code></dt>
<dd class="option-desc">Deprecated alias for <code>--workspace</code>.</dd>

<dt class="option-term" id="option-cargo-fix---exclude"><a class="option-anchor" href="#option-cargo-fix---exclude"></a><code>--exclude</code> <em>SPEC</em>...</dt>
<dd class="option-desc">Exclude the specified packages. Must be used in conjunction with the
<code>--workspace</code> flag. This flag may be specified multiple times and supports
common Unix glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell
accidentally expanding glob patterns before Cargo handles them, you must use
single quotes or double quotes around each pattern.</dd>

</dl>

### Target Selection

当没有给出目标选择选项时，`cargo fix`将修复所有目标(`--all-targets`暗示的）。如果有二进制文件，则跳过它们`required-features`这些都不见了。

传递目标选择标志将只修复指定的目标。

注意`--bin`, `--example`, `--test`和`--bench`标志还支持常见的Unix全局模式，如`*`, `?`和`[]`但是，为了避免您的shell在Cargo处理glob模式之前意外地扩展glob模式，您必须在每个glob模式周围使用单引号或双引号。

<dl>

<dt class="option-term" id="option-cargo-fix---lib"><a class="option-anchor" href="#option-cargo-fix---lib"></a><code>--lib</code></dt>
<dd class="option-desc">Fix the package's library.</dd>

<dt class="option-term" id="option-cargo-fix---bin"><a class="option-anchor" href="#option-cargo-fix---bin"></a><code>--bin</code> <em>name</em>...</dt>
<dd class="option-desc">Fix the specified binary. This flag may be specified multiple times
and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-fix---bins"><a class="option-anchor" href="#option-cargo-fix---bins"></a><code>--bins</code></dt>
<dd class="option-desc">Fix all binary targets.</dd>

<dt class="option-term" id="option-cargo-fix---example"><a class="option-anchor" href="#option-cargo-fix---example"></a><code>--example</code> <em>name</em>...</dt>
<dd class="option-desc">Fix the specified example. This flag may be specified multiple times
and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-fix---examples"><a class="option-anchor" href="#option-cargo-fix---examples"></a><code>--examples</code></dt>
<dd class="option-desc">Fix all example targets.</dd>

<dt class="option-term" id="option-cargo-fix---test"><a class="option-anchor" href="#option-cargo-fix---test"></a><code>--test</code> <em>name</em>...</dt>
<dd class="option-desc">Fix the specified integration test. This flag may be specified
multiple times and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-fix---tests"><a class="option-anchor" href="#option-cargo-fix---tests"></a><code>--tests</code></dt>
<dd class="option-desc">Fix all targets in test mode that have the <code>test = true</code> manifest
flag set. By default this includes the library and binaries built as
unittests, and integration tests. Be aware that this will also build any
required dependencies, so the lib target may be built twice (once as a
unittest, and once as a dependency for binaries, integration tests, etc.).
Targets may be enabled or disabled by setting the <code>test</code> flag in the
manifest settings for the target.</dd>

<dt class="option-term" id="option-cargo-fix---bench"><a class="option-anchor" href="#option-cargo-fix---bench"></a><code>--bench</code> <em>name</em>...</dt>
<dd class="option-desc">Fix the specified benchmark. This flag may be specified multiple
times and supports common Unix glob patterns.</dd>

<dt class="option-term" id="option-cargo-fix---benches"><a class="option-anchor" href="#option-cargo-fix---benches"></a><code>--benches</code></dt>
<dd class="option-desc">Fix all targets in benchmark mode that have the <code>bench = true</code>
manifest flag set. By default this includes the library and binaries built
as benchmarks, and bench targets. Be aware that this will also build any
required dependencies, so the lib target may be built twice (once as a
benchmark, and once as a dependency for binaries, benchmarks, etc.).
Targets may be enabled or disabled by setting the <code>bench</code> flag in the
manifest settings for the target.</dd>

<dt class="option-term" id="option-cargo-fix---all-targets"><a class="option-anchor" href="#option-cargo-fix---all-targets"></a><code>--all-targets</code></dt>
<dd class="option-desc">Fix all targets. This is equivalent to specifying <code>--lib --bins --tests --benches --examples</code>.</dd>

</dl>

### Feature Selection

功能标志允许您控制启用哪些功能。如果未提供任何功能选项，则`default`为每个选定的软件包激活该功能。

查看[the features documentation](../reference/features.html#command-line-feature-options)更多细节。

<dl>

<dt class="option-term" id="option-cargo-fix---features"><a class="option-anchor" href="#option-cargo-fix---features"></a><code>--features</code> <em>features</em></dt>
<dd class="option-desc">Space or comma separated list of features to activate. Features of workspace
members may be enabled with <code>package-name/feature-name</code> syntax. This flag may
be specified multiple times, which enables all specified features.</dd>

<dt class="option-term" id="option-cargo-fix---all-features"><a class="option-anchor" href="#option-cargo-fix---all-features"></a><code>--all-features</code></dt>
<dd class="option-desc">Activate all available features of all selected packages.</dd>

<dt class="option-term" id="option-cargo-fix---no-default-features"><a class="option-anchor" href="#option-cargo-fix---no-default-features"></a><code>--no-default-features</code></dt>
<dd class="option-desc">Do not activate the <code>default</code> feature of the selected packages.</dd>

</dl>

### Compilation Options

<dl>

<dt class="option-term" id="option-cargo-fix---target"><a class="option-anchor" href="#option-cargo-fix---target"></a><code>--target</code> <em>triple</em></dt>
<dd class="option-desc">Fix for the given architecture. The default is the host
architecture. The general format of the triple is
<code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>. Run <code>rustc --print target-list</code> for a
list of supported targets.</p>
<p>This may also be specified with the <code>build.target</code>
<a href="../reference/config.html">config value</a>.</p>
<p>Note that specifying this flag makes Cargo run in a different mode where the
target artifacts are placed in a separate directory. See the
<a href="../guide/build-cache.html">build cache</a> documentation for more details.</dd>

<dt class="option-term" id="option-cargo-fix--r"><a class="option-anchor" href="#option-cargo-fix--r"></a><code>-r</code></dt>
<dt class="option-term" id="option-cargo-fix---release"><a class="option-anchor" href="#option-cargo-fix---release"></a><code>--release</code></dt>
<dd class="option-desc">Fix optimized artifacts with the <code>release</code> profile.
See also the <code>--profile</code> option for choosing a specific profile by name.</dd>

<dt class="option-term" id="option-cargo-fix---profile"><a class="option-anchor" href="#option-cargo-fix---profile"></a><code>--profile</code> <em>name</em></dt>
<dd class="option-desc">Fix with the given profile.</p>
<p>As a special case, specifying the <code>test</code> profile will also enable checking in
test mode which will enable checking tests and enable the <code>test</code> cfg option.
See <a href="https://doc.rust-lang.org/rustc/tests/index.html">rustc tests</a> for more
detail.</p>
<p>See the <a href="../reference/profiles.html">the reference</a> for more details on profiles.</dd>

<dt class="option-term" id="option-cargo-fix---ignore-rust-version"><a class="option-anchor" href="#option-cargo-fix---ignore-rust-version"></a><code>--ignore-rust-version</code></dt>
<dd class="option-desc">Fix the target even if the selected Rust compiler is older than the
required Rust version as configured in the project's <code>rust-version</code> field.</dd>

</dl>

### Output Options

<dl>
<dt class="option-term" id="option-cargo-fix---target-dir"><a class="option-anchor" href="#option-cargo-fix---target-dir"></a><code>--target-dir</code> <em>directory</em></dt>
<dd class="option-desc">Directory for all generated artifacts and intermediate files. May also be
specified with the <code>CARGO_TARGET_DIR</code> environment variable, or the
<code>build.target-dir</code> <a href="../reference/config.html">config value</a>.
Defaults to <code>target</code> in the root of the workspace.</dd>

</dl>

### Display Options

<dl>
<dt class="option-term" id="option-cargo-fix--v"><a class="option-anchor" href="#option-cargo-fix--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-fix---verbose"><a class="option-anchor" href="#option-cargo-fix---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-fix--q"><a class="option-anchor" href="#option-cargo-fix--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-fix---quiet"><a class="option-anchor" href="#option-cargo-fix---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-fix---color"><a class="option-anchor" href="#option-cargo-fix---color"></a><code>--color</code> <em>when</em></dt>
<dd class="option-desc">Control when colored output is used. Valid values:</p>
<ul>
<li><code>auto</code> (default): Automatically detect if color support is available on the
terminal.</li>
<li><code>always</code>: Always display colors.</li>
<li><code>never</code>: Never display colors.</li>
</ul>
<p>May also be specified with the <code>term.color</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-fix---message-format"><a class="option-anchor" href="#option-cargo-fix---message-format"></a><code>--message-format</code> <em>fmt</em></dt>
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
<dt class="option-term" id="option-cargo-fix---manifest-path"><a class="option-anchor" href="#option-cargo-fix---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-fix---frozen"><a class="option-anchor" href="#option-cargo-fix---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-fix---locked"><a class="option-anchor" href="#option-cargo-fix---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-fix---offline"><a class="option-anchor" href="#option-cargo-fix---offline"></a><code>--offline</code></dt>
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

<dt class="option-term" id="option-cargo-fix-+toolchain"><a class="option-anchor" href="#option-cargo-fix-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-fix--h"><a class="option-anchor" href="#option-cargo-fix--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-fix---help"><a class="option-anchor" href="#option-cargo-fix---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-fix--Z"><a class="option-anchor" href="#option-cargo-fix--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

### Miscellaneous Options

<dl>
<dt class="option-term" id="option-cargo-fix--j"><a class="option-anchor" href="#option-cargo-fix--j"></a><code>-j</code> <em>N</em></dt>
<dt class="option-term" id="option-cargo-fix---jobs"><a class="option-anchor" href="#option-cargo-fix---jobs"></a><code>--jobs</code> <em>N</em></dt>
<dd class="option-desc">Number of parallel jobs to run. May also be specified with the
<code>build.jobs</code> <a href="../reference/config.html">config value</a>. Defaults to
the number of CPUs.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  将编译器建议应用于本地包：

    Cargo修理

2.  更新软件包以准备下一版本：

    Cargo修正版

3.  为当前版本应用建议的习惯用法：

    cargo fix——版本习惯用法

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-check(1)](cargo-check.html)
