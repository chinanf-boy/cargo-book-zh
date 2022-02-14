# cargo-verify-project(1)

## NAME

Cargo验证项目-检查箱子舱单的正确性

## SYNOPSIS

`cargo verify-project` [_options_]

## DESCRIPTION

此命令将解析本地清单并检查其有效性。它会发出一个JSON对象，并给出结果。成功的验证将显示：

```
{"success":"true"}
```

无效的工作区将显示：

```
{"invalid":"human-readable error message"}
```

## OPTIONS

### Display Options

<dl>

<dt class="option-term" id="option-cargo-verify-project--v"><a class="option-anchor" href="#option-cargo-verify-project--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-verify-project---verbose"><a class="option-anchor" href="#option-cargo-verify-project---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-verify-project--q"><a class="option-anchor" href="#option-cargo-verify-project--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-verify-project---quiet"><a class="option-anchor" href="#option-cargo-verify-project---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-verify-project---color"><a class="option-anchor" href="#option-cargo-verify-project---color"></a><code>--color</code> <em>when</em></dt>
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

<dt class="option-term" id="option-cargo-verify-project---manifest-path"><a class="option-anchor" href="#option-cargo-verify-project---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-verify-project---frozen"><a class="option-anchor" href="#option-cargo-verify-project---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-verify-project---locked"><a class="option-anchor" href="#option-cargo-verify-project---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-verify-project---offline"><a class="option-anchor" href="#option-cargo-verify-project---offline"></a><code>--offline</code></dt>
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

<dt class="option-term" id="option-cargo-verify-project-+toolchain"><a class="option-anchor" href="#option-cargo-verify-project-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-verify-project--h"><a class="option-anchor" href="#option-cargo-verify-project--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-verify-project---help"><a class="option-anchor" href="#option-cargo-verify-project---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-verify-project--Z"><a class="option-anchor" href="#option-cargo-verify-project--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：工作区正常。
-   `1`：工作区无效。

## EXAMPLES

1.  检查当前工作区是否存在错误：

    Cargo验证项目

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-package(1)](cargo-package.html)
