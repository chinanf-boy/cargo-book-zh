# cargo-new(1)

## NAME

Cargo新建-创建新的Cargo包

## SYNOPSIS

`cargo new` [_options_] *路径*

## DESCRIPTION

此命令将在给定目录中创建一个新的Cargo包。这包括一个带有`Cargo.toml`清单、样本源文件和VCS忽略文件。如果该目录不在VCS存储库中，则会创建一个新的存储库（请参阅`--vcs`下面）。

查看[cargo-init(1)](cargo-init.html)用于在现有目录中创建新清单的类似命令。

## OPTIONS

### New Options

<dl>

<dt class="option-term" id="option-cargo-new---bin"><a class="option-anchor" href="#option-cargo-new---bin"></a><code>--bin</code></dt>
<dd class="option-desc">Create a package with a binary target (<code>src/main.rs</code>).
This is the default behavior.</dd>

<dt class="option-term" id="option-cargo-new---lib"><a class="option-anchor" href="#option-cargo-new---lib"></a><code>--lib</code></dt>
<dd class="option-desc">Create a package with a library target (<code>src/lib.rs</code>).</dd>

<dt class="option-term" id="option-cargo-new---edition"><a class="option-anchor" href="#option-cargo-new---edition"></a><code>--edition</code> <em>edition</em></dt>
<dd class="option-desc">Specify the Rust edition to use. Default is 2021.
Possible values: 2015, 2018, 2021</dd>

<dt class="option-term" id="option-cargo-new---name"><a class="option-anchor" href="#option-cargo-new---name"></a><code>--name</code> <em>name</em></dt>
<dd class="option-desc">Set the package name. Defaults to the directory name.</dd>

<dt class="option-term" id="option-cargo-new---vcs"><a class="option-anchor" href="#option-cargo-new---vcs"></a><code>--vcs</code> <em>vcs</em></dt>
<dd class="option-desc">Initialize a new VCS repository for the given version control system (git,
hg, pijul, or fossil) or do not initialize any version control at all
(none). If not specified, defaults to <code>git</code> or the configuration value
<code>cargo-new.vcs</code>, or <code>none</code> if already inside a VCS repository.</dd>

<dt class="option-term" id="option-cargo-new---registry"><a class="option-anchor" href="#option-cargo-new---registry"></a><code>--registry</code> <em>registry</em></dt>
<dd class="option-desc">This sets the <code>publish</code> field in <code>Cargo.toml</code> to the given registry name
which will restrict publishing only to that registry.</p>
<p>Registry names are defined in <a href="../reference/config.html">Cargo config files</a>.
If not specified, the default registry defined by the <code>registry.default</code>
config key is used. If the default registry is not set and <code>--registry</code> is not
used, the <code>publish</code> field will not be set which means that publishing will not
be restricted.</dd>

</dl>

### Display Options

<dl>
<dt class="option-term" id="option-cargo-new--v"><a class="option-anchor" href="#option-cargo-new--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-new---verbose"><a class="option-anchor" href="#option-cargo-new---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-new--q"><a class="option-anchor" href="#option-cargo-new--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-new---quiet"><a class="option-anchor" href="#option-cargo-new---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-new---color"><a class="option-anchor" href="#option-cargo-new---color"></a><code>--color</code> <em>when</em></dt>
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

<dt class="option-term" id="option-cargo-new-+toolchain"><a class="option-anchor" href="#option-cargo-new-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-new--h"><a class="option-anchor" href="#option-cargo-new--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-new---help"><a class="option-anchor" href="#option-cargo-new---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-new--Z"><a class="option-anchor" href="#option-cargo-new--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  在给定目录中创建二进制Cargo包：

    货运新富

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-init(1)](cargo-init.html)