# cargo-uninstall(1)

## NAME

卸载Cargo-清除 Rust 的二进制文件

## SYNOPSIS

`cargo uninstall` [*options*][_spec_...]

## DESCRIPTION

此命令将删除随一起安装的软件包[cargo-install(1)](cargo-install.html)这个*规格*参数是要删除的包的包ID规范（请参阅[cargo-pkgid(1)](cargo-pkgid.html)).

默认情况下，箱子中的所有二进制文件都会被删除，但`--bin`和`--example`标志只能用于删除特定的二进制文件。

按照优先顺序确定安装根目录：

-   `--root`选项
-   `CARGO_INSTALL_ROOT`环境变量
-   `install.root`Cargo[config value](../reference/config.html)
-   `CARGO_HOME`环境变量
-   `$HOME/.cargo`

## OPTIONS

### Install Options

<dl>

<dt class="option-term" id="option-cargo-uninstall--p"><a class="option-anchor" href="#option-cargo-uninstall--p"></a><code>-p</code></dt>
<dt class="option-term" id="option-cargo-uninstall---package"><a class="option-anchor" href="#option-cargo-uninstall---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Package to uninstall.</dd>

<dt class="option-term" id="option-cargo-uninstall---bin"><a class="option-anchor" href="#option-cargo-uninstall---bin"></a><code>--bin</code> <em>name</em>...</dt>
<dd class="option-desc">Only uninstall the binary <em>name</em>.</dd>

<dt class="option-term" id="option-cargo-uninstall---root"><a class="option-anchor" href="#option-cargo-uninstall---root"></a><code>--root</code> <em>dir</em></dt>
<dd class="option-desc">Directory to uninstall packages from.</dd>

</dl>

### Display Options

<dl>

<dt class="option-term" id="option-cargo-uninstall--v"><a class="option-anchor" href="#option-cargo-uninstall--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-uninstall---verbose"><a class="option-anchor" href="#option-cargo-uninstall---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-uninstall--q"><a class="option-anchor" href="#option-cargo-uninstall--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-uninstall---quiet"><a class="option-anchor" href="#option-cargo-uninstall---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-uninstall---color"><a class="option-anchor" href="#option-cargo-uninstall---color"></a><code>--color</code> <em>when</em></dt>
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

<dt class="option-term" id="option-cargo-uninstall-+toolchain"><a class="option-anchor" href="#option-cargo-uninstall-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-uninstall--h"><a class="option-anchor" href="#option-cargo-uninstall--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-uninstall---help"><a class="option-anchor" href="#option-cargo-uninstall---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-uninstall--Z"><a class="option-anchor" href="#option-cargo-uninstall--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  卸载以前安装的软件包。

    Cargo倾覆

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-install(1)](cargo-install.html)
