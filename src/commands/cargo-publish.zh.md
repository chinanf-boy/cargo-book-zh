# cargo-publish(1)

## NAME

Cargo发布-将包上载到注册表

## SYNOPSIS

`cargo publish` [_options_]

## DESCRIPTION

此命令将创建一个可分发的、压缩的`.crate`将包的源代码保存在当前目录中，并将其上载到注册表。默认注册表是<https://crates.io>。这将执行以下步骤：

1.  执行一些检查，包括：
    -   检查`package.publish`在清单中输入允许发布到哪些注册表的限制。
2.  创建一个`.crate`按照中的步骤进行归档[cargo-package(1)](cargo-package.html).
3.  将箱子上载到注册表。请注意，服务器将对箱子执行其他检查。

此命令要求您通过`--token`选择或使用[cargo-login(1)](cargo-login.html).

查看[the reference](../reference/publishing.html)有关打包和发布的更多详细信息。

## OPTIONS

### Publish Options

<dl>

<dt class="option-term" id="option-cargo-publish---dry-run"><a class="option-anchor" href="#option-cargo-publish---dry-run"></a><code>--dry-run</code></dt>
<dd class="option-desc">Perform all checks without uploading.</dd>

<dt class="option-term" id="option-cargo-publish---token"><a class="option-anchor" href="#option-cargo-publish---token"></a><code>--token</code> <em>token</em></dt>
<dd class="option-desc">API token to use when authenticating. This overrides the token stored in
the credentials file (which is created by <a href="cargo-login.html">cargo-login(1)</a>).</p>
<p><a href="../reference/config.html">Cargo config</a> environment variables can be
used to override the tokens stored in the credentials file. The token for
crates.io may be specified with the <code>CARGO_REGISTRY_TOKEN</code> environment
variable. Tokens for other registries may be specified with environment
variables of the form <code>CARGO_REGISTRIES_NAME_TOKEN</code> where <code>NAME</code> is the name
of the registry in all capital letters.</dd>

<dt class="option-term" id="option-cargo-publish---no-verify"><a class="option-anchor" href="#option-cargo-publish---no-verify"></a><code>--no-verify</code></dt>
<dd class="option-desc">Don't verify the contents by building them.</dd>

<dt class="option-term" id="option-cargo-publish---allow-dirty"><a class="option-anchor" href="#option-cargo-publish---allow-dirty"></a><code>--allow-dirty</code></dt>
<dd class="option-desc">Allow working directories with uncommitted VCS changes to be packaged.</dd>

<dt class="option-term" id="option-cargo-publish---index"><a class="option-anchor" href="#option-cargo-publish---index"></a><code>--index</code> <em>index</em></dt>
<dd class="option-desc">The URL of the registry index to use.</dd>

<dt class="option-term" id="option-cargo-publish---registry"><a class="option-anchor" href="#option-cargo-publish---registry"></a><code>--registry</code> <em>registry</em></dt>
<dd class="option-desc">Name of the registry to publish to. Registry names are defined in <a href="../reference/config.html">Cargo
config files</a>. If not specified, and there is a
<a href="../reference/manifest.html#the-publish-field"><code>package.publish</code></a> field in
<code>Cargo.toml</code> with a single registry, then it will publish to that registry.
Otherwise it will use the default registry, which is defined by the
<a href="../reference/config.html#registrydefault"><code>registry.default</code></a> config key
which defaults to <code>crates-io</code>.</dd>

</dl>

### Package Selection

默认情况下，选择当前工作目录中的包。这个`-p`标志可用于在工作区中选择不同的包。

<dl>

<dt class="option-term" id="option-cargo-publish--p"><a class="option-anchor" href="#option-cargo-publish--p"></a><code>-p</code> <em>spec</em></dt>
<dt class="option-term" id="option-cargo-publish---package"><a class="option-anchor" href="#option-cargo-publish---package"></a><code>--package</code> <em>spec</em></dt>
<dd class="option-desc">The package to publish. See <a href="cargo-pkgid.html">cargo-pkgid(1)</a> for the SPEC
format.</dd>

</dl>

### Compilation Options

<dl>

<dt class="option-term" id="option-cargo-publish---target"><a class="option-anchor" href="#option-cargo-publish---target"></a><code>--target</code> <em>triple</em></dt>
<dd class="option-desc">Publish for the given architecture. The default is the host
architecture. The general format of the triple is
<code>&lt;arch&gt;&lt;sub&gt;-&lt;vendor&gt;-&lt;sys&gt;-&lt;abi&gt;</code>. Run <code>rustc --print target-list</code> for a
list of supported targets.</p>
<p>This may also be specified with the <code>build.target</code>
<a href="../reference/config.html">config value</a>.</p>
<p>Note that specifying this flag makes Cargo run in a different mode where the
target artifacts are placed in a separate directory. See the
<a href="../guide/build-cache.html">build cache</a> documentation for more details.</dd>

<dt class="option-term" id="option-cargo-publish---target-dir"><a class="option-anchor" href="#option-cargo-publish---target-dir"></a><code>--target-dir</code> <em>directory</em></dt>
<dd class="option-desc">Directory for all generated artifacts and intermediate files. May also be
specified with the <code>CARGO_TARGET_DIR</code> environment variable, or the
<code>build.target-dir</code> <a href="../reference/config.html">config value</a>.
Defaults to <code>target</code> in the root of the workspace.</dd>

</dl>

### Feature Selection

功能标志允许您控制启用哪些功能。如果未提供任何功能选项，则`default`为每个选定的软件包激活该功能。

查看[the features documentation](../reference/features.html#command-line-feature-options)更多细节。

<dl>

<dt class="option-term" id="option-cargo-publish---features"><a class="option-anchor" href="#option-cargo-publish---features"></a><code>--features</code> <em>features</em></dt>
<dd class="option-desc">Space or comma separated list of features to activate. Features of workspace
members may be enabled with <code>package-name/feature-name</code> syntax. This flag may
be specified multiple times, which enables all specified features.</dd>

<dt class="option-term" id="option-cargo-publish---all-features"><a class="option-anchor" href="#option-cargo-publish---all-features"></a><code>--all-features</code></dt>
<dd class="option-desc">Activate all available features of all selected packages.</dd>

<dt class="option-term" id="option-cargo-publish---no-default-features"><a class="option-anchor" href="#option-cargo-publish---no-default-features"></a><code>--no-default-features</code></dt>
<dd class="option-desc">Do not activate the <code>default</code> feature of the selected packages.</dd>

</dl>

### Manifest Options

<dl>

<dt class="option-term" id="option-cargo-publish---manifest-path"><a class="option-anchor" href="#option-cargo-publish---manifest-path"></a><code>--manifest-path</code> <em>path</em></dt>
<dd class="option-desc">Path to the <code>Cargo.toml</code> file. By default, Cargo searches for the
<code>Cargo.toml</code> file in the current directory or any parent directory.</dd>

<dt class="option-term" id="option-cargo-publish---frozen"><a class="option-anchor" href="#option-cargo-publish---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo-publish---locked"><a class="option-anchor" href="#option-cargo-publish---locked"></a><code>--locked</code></dt>
<dd class="option-desc">Either of these flags requires that the <code>Cargo.lock</code> file is
up-to-date. If the lock file is missing, or it needs to be updated, Cargo will
exit with an error. The <code>--frozen</code> flag also prevents Cargo from
attempting to access the network to determine if it is out-of-date.</p>
<p>These may be used in environments where you want to assert that the
<code>Cargo.lock</code> file is up-to-date (such as a CI build) or want to avoid network
access.</dd>

<dt class="option-term" id="option-cargo-publish---offline"><a class="option-anchor" href="#option-cargo-publish---offline"></a><code>--offline</code></dt>
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

### Miscellaneous Options

<dl>
<dt class="option-term" id="option-cargo-publish--j"><a class="option-anchor" href="#option-cargo-publish--j"></a><code>-j</code> <em>N</em></dt>
<dt class="option-term" id="option-cargo-publish---jobs"><a class="option-anchor" href="#option-cargo-publish---jobs"></a><code>--jobs</code> <em>N</em></dt>
<dd class="option-desc">Number of parallel jobs to run. May also be specified with the
<code>build.jobs</code> <a href="../reference/config.html">config value</a>. Defaults to
the number of CPUs.</dd>

</dl>

### Display Options

<dl>
<dt class="option-term" id="option-cargo-publish--v"><a class="option-anchor" href="#option-cargo-publish--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo-publish---verbose"><a class="option-anchor" href="#option-cargo-publish---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">Use verbose output. May be specified twice for &quot;very verbose&quot; output which
includes extra output such as dependency warnings and build script output.
May also be specified with the <code>term.verbose</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-publish--q"><a class="option-anchor" href="#option-cargo-publish--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo-publish---quiet"><a class="option-anchor" href="#option-cargo-publish---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">Do not print cargo log messages.
May also be specified with the <code>term.quiet</code>
<a href="../reference/config.html">config value</a>.</dd>

<dt class="option-term" id="option-cargo-publish---color"><a class="option-anchor" href="#option-cargo-publish---color"></a><code>--color</code> <em>when</em></dt>
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

<dt class="option-term" id="option-cargo-publish-+toolchain"><a class="option-anchor" href="#option-cargo-publish-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">If Cargo has been installed with rustup, and the first argument to <code>cargo</code>
begins with <code>+</code>, it will be interpreted as a rustup toolchain name (such
as <code>+stable</code> or <code>+nightly</code>).
See the <a href="https://rust-lang.github.io/rustup/overrides.html">rustup documentation</a>
for more information about how toolchain overrides work.</dd>

<dt class="option-term" id="option-cargo-publish--h"><a class="option-anchor" href="#option-cargo-publish--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo-publish---help"><a class="option-anchor" href="#option-cargo-publish---help"></a><code>--help</code></dt>
<dd class="option-desc">Prints help information.</dd>

<dt class="option-term" id="option-cargo-publish--Z"><a class="option-anchor" href="#option-cargo-publish--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Unstable (nightly-only) flags to Cargo. Run <code>cargo -Z help</code> for details.</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.html)有关Cargo读取的环境变量的详细信息。

## EXIT STATUS

-   `0`：Cargo成功。
-   `101`：Cargo未能完成。

## EXAMPLES

1.  发布当前包：

    Cargo发布

## SEE ALSO

[cargo(1)](cargo.zh.md), [cargo-package(1)](cargo-package.html), [cargo-login(1)](cargo-login.html)
