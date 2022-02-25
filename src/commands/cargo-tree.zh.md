# cargo-tree(1)

## NAME

Cargo树-显示依赖关系图的树可视化

## SYNOPSIS

`cargo tree` [_options_]

## DESCRIPTION

此命令将显示终端的依赖关系树。依赖“兰德”软件包的简单项目示例：

```
myproject v0.1.0 (/myproject)
└── rand v0.7.3
    ├── getrandom v0.1.14
    │   ├── cfg-if v0.1.10
    │   └── libc v0.2.68
    ├── libc v0.2.68 (*)
    ├── rand_chacha v0.2.2
    │   ├── ppv-lite86 v0.2.6
    │   └── rand_core v0.5.1
    │       └── getrandom v0.1.14 (*)
    └── rand_core v0.5.1 (*)
[build-dependencies]
└── cc v1.0.50
```

标有`(*)`已被“消除重复”。包的依赖关系已经在图中的其他地方显示，因此不再重复。使用`--no-dedupe`选项来重复重复。

这个`-e`标志可用于选择要显示的依赖项类型。“features”类更改输出，以显示每个依赖项启用的特性。例如`cargo tree -e features`:

```
myproject v0.1.0 (/myproject)
└── log feature "serde"
    └── log v0.4.8
        ├── serde v1.0.106
        └── cfg-if feature "default"
            └── cfg-if v0.1.10
```

在这棵树上，`myproject`取决于`log`和`serde`特色`log`反过来取决于`cfg-if`具有“默认”功能。使用时`-e features`使用它可能会有所帮助`-i`标志以显示功能如何流入包中。有关更多详细信息，请参见下面的示例。

## OPTIONS

### Tree Options

<dl>

<dt class="option-term" id="option-cargo-tree--i"><a class="option-anchor" href="#option-cargo-tree--i"></a><code>-i</code> <em>spec</em></dt>
<dt class="option-term" id="option-cargo-tree---invert"><a class="option-anchor" href="#option-cargo-tree---invert"></a><code>--invert</code> <em>spec</em></dt>
<dd class="option-desc">Show the reverse dependencies for the given package. This flag will invert
the tree and display the packages that depend on the given package.</p>
<p>Note that in a workspace, by default it will only display the package's
reverse dependencies inside the tree of the workspace member in the current
directory. The <code>--workspace</code> flag can be used to extend it so that it will
show the package's reverse dependencies across the entire workspace. The <code>-p</code>
flag can be used to display the package's reverse dependencies only with the
subtree of the package given to <code>-p</code>.</dd>

<dt class="option-term" id="option-cargo-tree---prune"><a class="option-anchor" href="#option-cargo-tree---prune"></a><code>--prune</code> <em>spec</em></dt>
<dd class="option-desc">Prune the given package from the display of the dependency tree.</dd>

<dt class="option-term" id="option-cargo-tree---depth"><a class="option-anchor" href="#option-cargo-tree---depth"></a><code>--depth</code> <em>depth</em></dt>
<dd class="option-desc">Maximum display depth of the dependency tree. A depth of 1 displays the direct
dependencies, for example.</dd>

<dt class="option-term" id="option-cargo-tree---no-dedupe"><a class="option-anchor" href="#option-cargo-tree---no-dedupe"></a><code>--no-dedupe</code></dt>
<dd class="option-desc">Do not de-duplicate repeated dependencies. Usually, when a package has already
displayed its dependencies, further occurrences will not re-display its
dependencies, and will include a <code>(*)</code> to indicate it has already been shown.
This flag will cause those duplicates to be repeated.</dd>

<dt class="option-term" id="option-cargo-tree--d"><a class="option-anchor" href="#option-cargo-tree--d"></a><code>-d</code></dt>
<dt class="option-term" id="option-cargo-tree---duplicates"><a class="option-anchor" href="#option-cargo-tree---duplicates"></a><code>--duplicates</code></dt>
<dd class="option-desc">Show only dependencies which come in multiple versions (implies <code>--invert</code>).
When used with the <code>-p</code> flag, only shows duplicates within the subtree of the
given package.</p>
<p>It can be beneficial for build times and executable sizes to avoid building
that same package multiple times. This flag can help identify the offending
packages. You can then investigate if the package that depends on the
duplicate with the older version can be updated to the newer version so that
only one instance is built.</dd>

<dt class="option-term" id="option-cargo-tree--e"><a class="option-anchor" href="#option-cargo-tree--e"></a><code>-e</code> <em>kinds</em></dt>
<dt class="option-term" id="option-cargo-tree---edges"><a class="option-anchor" href="#option-cargo-tree---edges"></a><code>--edges</code> <em>kinds</em></dt>
<dd class="option-desc">The dependency kinds to display. Takes a comma separated list of values:</p>
<ul>
<li><code>all</code> — Show all edge kinds.</li>
<li><code>normal</code> — Show normal dependencies.</li>
<li><code>build</code> — Show build dependencies.</li>
<li><code>dev</code> — Show development dependencies.</li>
<li><code>features</code> — Show features enabled by each dependency. If this is the only
kind given, then it will automatically include the other dependency kinds.</li>
<li><code>no-normal</code> — Do not include normal dependencies.</li>
<li><code>no-build</code> — Do not include build dependencies.</li>
<li><code>no-dev</code> — Do not include development dependencies.</li>
<li><code>no-proc-macro</code> — Do not include procedural macro dependencies.</li>
</ul>
<p>The <code>normal</code>, <code>build</code>, <code>dev</code>, and <code>all</code> dependency kinds cannot be mixed with
<code>no-normal</code>, <code>no-build</code>, or <code>no-dev</code> dependency kinds.</p>
<p>The default is <code>normal,build,dev</code>.</dd>

<dt class="option-term" id="option-cargo-tree---target"><a class="option-anchor" href="#option-cargo-tree---target"></a><code>--target</code> <em>triple</em></dt>
<dd class="option-desc">Filter dependencies matching the given target-triple. The default is the host
platform. Use the value <code>all</code> to include <em>all</em> targets.</dd>

</dl>

### Tree Formatting Options

<dl>

<dt class="option-term" id="option-cargo-tree---charset"><a class="option-anchor" href="#option-cargo-tree---charset"></a><code>--charset</code> <em>charset</em></dt>
<dd class="option-desc">Chooses the character set to use for the tree. Valid values are &quot;utf8&quot; or
&quot;ascii&quot;. Default is &quot;utf8&quot;.</dd>

<dt class="option-term" id="option-cargo-tree--f"><a class="option-anchor" href="#option-cargo-tree--f"></a><code>-f</code> <em>format</em></dt>
<dt class="option-term" id="option-cargo-tree---format"><a class="option-anchor" href="#option-cargo-tree---format"></a><code>--format</code> <em>format</em></dt>
<dd class="option-desc">Set the format string for each package. The default is &quot;{p}&quot;.</p>
<p>This is an arbitrary string which will be used to display each package. The following
strings will be replaced with the corresponding value:</p>
<ul>
<li><code>{p}</code> — The package name.</li>
<li><code>{l}</code> — The package license.</li>
<li><code>{r}</code> — The package repository URL.</li>
<li><code>{f}</code> — Comma-separated list of package features that are enabled.</li>
<li><code>{lib}</code> — The name, as used in a <code>use</code> statement, of the package's library.</li>
</ul></dd>

<dt class="option-term" id="option-cargo-tree---prefix"><a class="option-anchor" href="#option-cargo-tree---prefix"></a><code>--prefix</code> <em>prefix</em></dt>
<dd class="option-desc">Sets how each line is displayed. The <em>prefix</em> value can be one of:</p>
<ul>
<li><code>indent</code> (default) — Shows each line indented as a tree.</li>
<li><code>depth</code> — Show as a list, with the numeric depth printed before each entry.</li>
<li><code>none</code> — Show as a flat list.</li>
</ul></dd>

</dl>

### Package Selection

默认情况下，如果未提供包选择选项，则选择的包取决于选择的清单文件（如果需要，则基于当前工作目录）`--manifest-path`未给出）。如果清单是工作区的根目录，则会选择工作区默认成员，否则只会选择清单定义的包。

可以使用`workspace.default-members`输入根清单。如果未设置，虚拟工作区将包括所有工作区成员（相当于传递）`--workspace`)，非虚拟工作区将只包括根箱子本身。

<dl>

<dt class="option-term" id="option-cargo-tree--p"><a class="option-anchor" href="#option-cargo-tree--p"></a><code>-p</code> <em>spec</em>...</dt>
<dt class="option-term" id="option-cargo-tree---package"><a class="option-anchor" href="#option-cargo-tree---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Display only the specified packages. See <a href="cargo-pkgid.html">cargo-pkgid(1)</a> for the
SPEC format. This flag may be specified multiple times and supports common Unix
glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell accidentally 
expanding glob patterns before Cargo handles them, you must use single quotes or
double quotes around each pattern.</dd>

<dt class="option-term" id="option-cargo-tree---workspace"><a class="option-anchor" href="#option-cargo-tree---workspace"></a><code>--workspace</code></dt>
<dd class="option-desc">Display all members in the workspace.</dd>

<dt class="option-term" id="option-cargo-tree---exclude"><a class="option-anchor" href="#option-cargo-tree---exclude"></a><code>--exclude</code> <em>SPEC</em>...</dt>
<dd class="option-desc">Exclude the specified packages. Must be used in conjunction with the
<code>--workspace</code> flag. This flag may be specified multiple times and supports
common Unix glob patterns like <code>*</code>, <code>?</code> and <code>[]</code>. However, to avoid your shell
accidentally expanding glob patterns before Cargo handles them, you must use
single quotes or double quotes around each pattern.</dd>

</dl>

### Manifest Options

<dl>

<dt class="option-term" id="option-cargo-tree---manifest-path"><a class="option-anchor" href="#option-cargo-tree---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-tree---frozen"><a class="option-anchor" href="#option-cargo-tree---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-tree---locked"><a class="option-anchor" href="#option-cargo-tree---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-tree---offline"><a class="option-anchor" href="#option-cargo-tree---offline"></a><code>--offline</code></dt>
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

### Feature Selection

功能标志允许您控制启用哪些功能。如果未提供任何功能选项，则`default`为每个选定的软件包激活该功能。

查看[the features documentation](../reference/features.html#command-line-feature-options)更多细节。

<dl>

<dt class="option-term" id="option-cargo-tree---features"><a class="option-anchor" href="#option-cargo-tree---features"></a><code>--features</code> <em>features</em></dt>
<dd class="option-desc">Space or comma separated list of features to activate. Features of workspace
members may be enabled with <code>package-name/feature-name</code> syntax. This flag may
be specified multiple times, which enables all specified features.</dd>

<dt class="option-term" id="option-cargo-tree---all-features"><a class="option-anchor" href="#option-cargo-tree---all-features"></a><code>--all-features</code></dt>
<dd class="option-desc">Activate all available features of all selected packages.</dd>

<dt class="option-term" id="option-cargo-tree---no-default-features"><a class="option-anchor" href="#option-cargo-tree---no-default-features"></a><code>--no-default-features</code></dt>
<dd class="option-desc">Do not activate the <code>default</code> feature of the selected packages.</dd>

</dl>

### Display Options

<dl>

<dt class="option-term" id="option-cargo-tree--v"><a class="option-anchor" href="#option-cargo-tree--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-tree---verbose"><a class="option-anchor" href="#option-cargo-tree---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-tree--q"><a class="option-anchor" href="#option-cargo-tree--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-tree---quiet"><a class="option-anchor" href="#option-cargo-tree---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-tree---color"><a class="option-anchor" href="#option-cargo-tree---color"></a><code>--color</code> <em>when</em></dt>
<dd class="option-desc">Control when colored output is used. Valid values:</p>
<ul>
<li><code>auto</code> (default): Automatically detect if color support is available on the
terminal.</li>
<li><code>always</code>: Always display colors.</li>
<li><code>never</code>: Never display colors.</li>
</ul>
<p>May also be specified with the <code>term.color</code>
<a href="../reference/config.html">config value</a>.</dd>

</dl>

### Common Options

<dl>

<dt class="option-term" id="option-cargo-tree-+toolchain"><a class="option-anchor" href="#option-cargo-tree-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-tree--h"><a class="option-anchor" href="#option-cargo-tree--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-tree---help"><a class="option-anchor" href="#option-cargo-tree---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-tree--Z"><a class="option-anchor" href="#option-cargo-tree--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  在当前目录中显示包的目录树：

    货运树

2.  显示所有依赖于`syn`包：

    货运树-i syn

3.  显示每个软件包上启用的功能：

    Cargo树--格式“{p}{f}”

4.  显示多次生成的所有包。如果树中出现多个semver不兼容的版本（如1.0.0和2.0.0），可能会发生这种情况。

    ```
    cargo tree -d
    ```

5.  解释为什么为`syn`包：

    ```
    cargo tree -e features -i syn
    ```

    这个`-e features`标志用于显示特征。这个`-i`标志用于反转图形，以便显示依赖于`syn`.这将显示什么的示例：

    ```
    syn v1.0.17
    ├── syn feature "clone-impls"
    │   └── syn feature "default"
    │       └── rustversion v1.0.2
    │           └── rustversion feature "default"
    │               └── myproject v0.1.0 (/myproject)
    │                   └── myproject feature "default" (command-line)
    ├── syn feature "default" (*)
    ├── syn feature "derive"
    │   └── syn feature "default" (*)
    ├── syn feature "full"
    │   └── rustversion v1.0.2 (*)
    ├── syn feature "parsing"
    │   └── syn feature "default" (*)
    ├── syn feature "printing"
    │   └── syn feature "default" (*)
    ├── syn feature "proc-macro"
    │   └── syn feature "default" (*)
    └── syn feature "quote"
        ├── syn feature "printing" (*)
        └── syn feature "proc-macro" (*)
    ```

    要阅读此图，您可以从根目录中查看每个功能的链，以了解其包含的原因。例如，“完整”功能由`rustversion`箱子，包括从`myproject`（具有默认功能），以及`myproject`是在命令行上选择的包。所有其他的`syn`功能由“默认”功能添加（“引用”由“打印”和“proc宏”添加，两者都是默认功能）。

    如果交叉引用已消除重复的`(*)`输入，用`--no-dedupe`标记以获取完整输出。

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-metadata(1)](cargo-metadata.html)
