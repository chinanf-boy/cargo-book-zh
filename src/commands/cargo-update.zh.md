# cargo-update(1)

## NAME

Cargo更新-更新本地锁文件中记录的依赖项

## SYNOPSIS

`cargo update` [_options_]

## DESCRIPTION

此命令将更新中的依赖项`Cargo.lock`文件更新至最新版本。如果`Cargo.lock`文件不存在，将使用最新的可用版本创建。

## OPTIONS

### Update Options

<dl>

<dt class="option-term" id="option-cargo-update--p"><a class="option-anchor" href="#option-cargo-update--p"></a><code>-p</code> <em>spec</em>...</dt>
<dt class="option-term" id="option-cargo-update---package"><a class="option-anchor" href="#option-cargo-update---package"></a><code>--package</code> <em>spec</em>...</dt>
<dd class="option-desc">Update only the specified packages. This flag may be specified
multiple times. See <a href="cargo-pkgid.html">cargo-pkgid(1)</a> for the SPEC format.</p>
<p>If packages are specified with the <code>-p</code> flag, then a conservative update of
the lockfile will be performed. This means that only the dependency specified
by SPEC will be updated. Its transitive dependencies will be updated only if
SPEC cannot be updated without updating dependencies.  All other dependencies
will remain locked at their currently recorded versions.</p>
<p>If <code>-p</code> is not specified, all dependencies are updated.</dd>

<dt class="option-term" id="option-cargo-update---aggressive"><a class="option-anchor" href="#option-cargo-update---aggressive"></a><code>--aggressive</code></dt>
<dd class="option-desc">When used with <code>-p</code>, dependencies of <em>spec</em> are forced to update as well.
Cannot be used with <code>--precise</code>.</dd>

<dt class="option-term" id="option-cargo-update---precise"><a class="option-anchor" href="#option-cargo-update---precise"></a><code>--precise</code> <em>precise</em></dt>
<dd class="option-desc">When used with <code>-p</code>, allows you to specify a specific version number to set
the package to. If the package comes from a git repository, this can be a git
revision (such as a SHA hash or tag).</dd>

<dt class="option-term" id="option-cargo-update--w"><a class="option-anchor" href="#option-cargo-update--w"></a><code>-w</code></dt>
<dt class="option-term" id="option-cargo-update---workspace"><a class="option-anchor" href="#option-cargo-update---workspace"></a><code>--workspace</code></dt>
<dd class="option-desc">Attempt to update only packages defined in the workspace. Other packages
are updated only if they don't already exist in the lockfile. This
option is useful for updating <code>Cargo.lock</code> after you've changed version
numbers in <code>Cargo.toml</code>.</dd>

<dt class="option-term" id="option-cargo-update---dry-run"><a class="option-anchor" href="#option-cargo-update---dry-run"></a><code>--dry-run</code></dt>
<dd class="option-desc">Displays what would be updated, but doesn't actually write the lockfile.</dd>

</dl>

### Display Options

<dl>
<dt class="option-term" id="option-cargo-update--v"><a class="option-anchor" href="#option-cargo-update--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-update---verbose"><a class="option-anchor" href="#option-cargo-update---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-update--q"><a class="option-anchor" href="#option-cargo-update--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-update---quiet"><a class="option-anchor" href="#option-cargo-update---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-update---color"><a class="option-anchor" href="#option-cargo-update---color"></a><code>--color</code> <em>when</em></dt>
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

### Manifest Options

<dl>

<dt class="option-term" id="option-cargo-update---manifest-path"><a class="option-anchor" href="#option-cargo-update---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-update---frozen"><a class="option-anchor" href="#option-cargo-update---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-update---locked"><a class="option-anchor" href="#option-cargo-update---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-update---offline"><a class="option-anchor" href="#option-cargo-update---offline"></a><code>--offline</code></dt>
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

<dt class="option-term" id="option-cargo-update-+toolchain"><a class="option-anchor" href="#option-cargo-update-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-update--h"><a class="option-anchor" href="#option-cargo-update--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-update---help"><a class="option-anchor" href="#option-cargo-update---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-update--Z"><a class="option-anchor" href="#option-cargo-update--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

## ENVIRONMENT

看见[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  更新锁定文件中的所有依赖项：

    Cargo更新

2.  仅更新特定的依赖项：

    Cargo更新-p foo-p bar

3.  将特定依赖项设置为特定版本：

    Cargo更新-p foo-精确1.2.3

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-generate-lockfile(1)](cargo-generate-lockfile.html)
