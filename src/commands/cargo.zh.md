# cargo(1)

## NAME

Cargo - Rust 包管理器

## SYNOPSIS

`cargo` [_options_] _command_ [_args_]\
`cargo` [_options_] `--version`\
`cargo` [_options_] `--list`\
`cargo` [_options_] `--help`\
`cargo` [_options_] `--explain` _code_

## DESCRIPTION

该程序是 Rust 语言的包管理器和构建工具，列入<https://rust-lang.org>。

## COMMANDS

### Build Commands

[cargo-bench(1)](cargo-bench.zh.md)\
执行包的基准测试。

[cargo-build(1)](cargo-build.zh.md)\
编译一个包。

[cargo-check(1)](cargo-check.zh.md)\
检查本地包及其所有依赖项是否存在错误。

[cargo-clean(1)](cargo-clean.zh.md)\
移除 Cargo 过去生成的工件。

[cargo-doc(1)](cargo-doc.zh.md)\
构建包的文档。

[cargo-fetch(1)](cargo-fetch.zh.md)\
网络获取包的依赖项。

[cargo-fix(1)](cargo-fix.zh.md)\
自动修复 rustc 报告的 lint 警告。

[cargo-run(1)](cargo-run.zh.md)\
运行二进制文件或本地包的示例。

[cargo-rustc(1)](cargo-rustc.zh.md)\
编译一个包，并将额外的选项传递给编译器。

[cargo-rustdoc(1)](cargo-rustdoc.zh.md)\
使用指定的自定义标志，生成包的文档。

[cargo-test(1)](cargo-test.zh.md)\
执行软件包的单元测试和集成测试。

### Manifest Commands

[cargo-generate-lockfile(1)](cargo-generate-lockfile.zh.md)\
为了一个项目，生成`Cargo.lock`。

[cargo-locate-project(1)](cargo-locate-project.zh.md)\
打印 JSON 格式的`Cargo.toml`文件的位置。

[cargo-metadata(1)](cargo-metadata.zh.md)\
以机器可读的格式，输出已解析的包依赖项。

[cargo-pkgid(1)](cargo-pkgid.zh.md)\
打印完全合格的包规格。

[cargo-tree(1)](cargo-tree.zh.md)\
显示依赖关系图的树可视化。

[cargo-update(1)](cargo-update.zh.md)\
更新本地锁文件中，记录的依赖项。

[cargo-vendor(1)](cargo-vendor.zh.md)\
本地供应所有依赖项。

[cargo-verify-project(1)](cargo-verify-project.zh.md)\
检查箱子清单的正确性。

### Package Commands

[cargo-init(1)](cargo-init.zh.md)\
在现有目录中，创建新的 Cargo 包。

[cargo-install(1)](cargo-install.zh.md)\
构建并安装一个 Rust 的二进制文件。

[cargo-new(1)](cargo-new.zh.md)\
创建一个新的 Cargo 包。

[cargo-search(1)](cargo-search.zh.md)\
在 crates.io 中搜索包。

[cargo-uninstall(1)](cargo-uninstall.zh.md)\
移除一个 Rust 包二进制程序。

### Publishing Commands

[cargo-login(1)](cargo-login.zh.md)\
本地保存一个 registry（来源网站） 的 API 令牌。

[cargo-owner(1)](cargo-owner.zh.md)\
管理 registry 上，箱子的所有者。

[cargo-package(1)](cargo-package.zh.md)\
将本地包汇编成一个可分配的压缩文件（tarball）。

[cargo-publish(1)](cargo-publish.zh.md)\
将包上载到 来源网站 。

[cargo-yank(1)](cargo-yank.zh.md)\
从索引中，移除一个已推送的箱子。

### General Commands

[cargo-help(1)](cargo-help.zh.md)\
显示有关 Cargo 的帮助信息。

[cargo-version(1)](cargo-version.zh.md)\
显示版本信息。

## OPTIONS

### Special Options

<dl>

<dt class="option-term" id="option-cargo--V"><a class="option-anchor" href="#option-cargo--V"></a><code>-V</code></dt>
<dt class="option-term" id="option-cargo---version"><a class="option-anchor" href="#option-cargo---version"></a><code>--version</code></dt>
<dd class="option-desc">打印版本信息并退出。如果使用了 <code>--verbose</code>, 就会打印额外信息</dd>

<dt class="option-term" id="option-cargo---list"><a class="option-anchor" href="#option-cargo---list"></a><code>--list</code></dt>
<dd class="option-desc">列出所有安装的 Cargo 子命令。如果使用了 <code>--verbose</code>, 打印额外的信息。</dd>

<dt class="option-term" id="option-cargo---explain"><a class="option-anchor" href="#option-cargo---explain"></a><code>--explain</code> <em>code</em></dt>
<dd class="option-desc">运行 <code>rustc --explain CODE</code>，它会给出一个错误信息的详细解释 (例如 CODE 可以是, <code>E0004</code>)。</dd>

</dl>

### Display Options

<dl>

<dt class="option-term" id="option-cargo--v"><a class="option-anchor" href="#option-cargo--v"></a><code>-v</code></dt>
<dt class="option-term" id="option-cargo---verbose"><a class="option-anchor" href="#option-cargo---verbose"></a><code>--verbose</code></dt>
<dd class="option-desc">详细的输出。还可以两个 v 字符，就是 &quot;非常详细&quot; 的输出：包括额外的输出，如依赖警告和构建脚本的输出。
还可以使用 <code>term.verbose</code>
<a href="../reference/config.zh.html">配置选项</a>.</dd>

<dt class="option-term" id="option-cargo--q"><a class="option-anchor" href="#option-cargo--q"></a><code>-q</code></dt>
<dt class="option-term" id="option-cargo---quiet"><a class="option-anchor" href="#option-cargo---quiet"></a><code>--quiet</code></dt>
<dd class="option-desc">不打印日志信息。
还可以使用 <code>term.quiet</code>
<a href="../reference/config.zh.html">配置选项</a>。</dd>

<dt class="option-term" id="option-cargo---color"><a class="option-anchor" href="#option-cargo---color"></a><code>--color</code></dt>
<dd class="option-desc">颜色化输出的配置。可用选项:</p>
<ul>
<li><code>auto</code> (default): 自动</li>
<li><code>always</code>: 有颜色</li>
<li><code>never</code>: 没颜色</li>
</ul>
<p>还可以使用 <code>term.color</code>
<a href="../reference/config.zh.html">配置选项</a>.</dd>

</dl>

### Manifest Options

<dl>
<dt class="option-term" id="option-cargo---frozen"><a class="option-anchor" href="#option-cargo---frozen"></a><code>--frozen</code></dt>
<dt class="option-term" id="option-cargo---locked"><a class="option-anchor" href="#option-cargo---locked"></a><code>--locked</code></dt>
<dd class="option-desc">两个命令选项都要求 <code>Cargo.lock</code> 文件是最新的。如果缺少了该文件，或是它需要更新，那么 Cargo 就以一个错误退出。 <code>--frozen</code> 阻止 Cargo 连网检测它是不是最新的。</p>
<p>这两个选项可用在 CI 的环境，断定
<code>Cargo.lock</code> 是最新的，或是阻止网络的访问。</dd>

<dt class="option-term" id="option-cargo---offline"><a class="option-anchor" href="#option-cargo---offline"></a><code>--offline</code></dt>
<dd class="option-desc">网络禁止。在没有这个选项的情况下，网络访问失败，Cargo 会以一个错误停止。若使用了这个选项，Cargo 会尝试不联网的状态继续执行</p>
<p>注意，大概率会产生与联网时不同的依赖解析。限制为本地已下载的箱子，即便是本地索引的新副本，都不行。
看看 <a href="cargo-fetch.zh.html">cargo-fetch(1)</a> 在断网前，下载依赖</p>
<p>还可以使用 <code>net.offline</code> <a href="../reference/config.zh.html">配置选项</a>.</dd>

</dl>

### Common Options

<dl>

<dt class="option-term" id="option-cargo-+toolchain"><a class="option-anchor" href="#option-cargo-+toolchain"></a><code>+</code><em>toolchain</em></dt>
<dd class="option-desc">若是使用 rustup 安装的 Cargo, <code>cargo</code> 的第一个参数 
<code>+</code>, 跟着工具链名称 (例如
例如 <code>+stable</code> 或 <code>+nightly</code>).
可以查看<a href="https://rust-lang.github.io/rustup/overrides.html">rustup 文档。</a>
关于工具链覆盖的详细信息。</dd>

<dt class="option-term" id="option-cargo--h"><a class="option-anchor" href="#option-cargo--h"></a><code>-h</code></dt>
<dt class="option-term" id="option-cargo---help"><a class="option-anchor" href="#option-cargo---help"></a><code>--help</code></dt>
<dd class="option-desc">打印帮助信息。</dd>

<dt class="option-term" id="option-cargo--Z"><a class="option-anchor" href="#option-cargo--Z"></a><code>-Z</code> <em>flag</em></dt>
<dd class="option-desc">Cargo 的 不稳定的 (nightly-only) 选项。运行 <code>cargo -Z help</code> 获取详细信息。</dd>

</dl>

## ENVIRONMENT

查看[the reference](../reference/environment-variables.zh.md)有关 Cargo 读取的环境变量的详细信息。

## EXIT STATUS

- `0`： Cargo 成功。
- `101`： Cargo 未能完成。

## FILES

`~/.cargo/`\
Cargo 的“主”目录的默认位置，用于存储各种文件。可以通过`CARGO_HOME`环境变量改变。

`$CARGO_HOME/bin/`\
 由[cargo-install(1)](cargo-install.zh.md)安装的二进制文件，将位于这里。如果使用[rustup]，Rust 的可执行文件也位于此处。

`$CARGO_HOME/config.toml`\
 全局配置文件。查看[the reference](../reference/config.zh.md)有关配置文件的更多信息。

`.cargo/config.toml`\
Cargo 会自动搜索，在当前目录和所有父目录中，名为`.cargo/config.toml`的文件。这些配置文件将与全局配置文件合并。

`$CARGO_HOME/credentials.toml`\
 用于登录到 来源网站 的私人身份验证信息。

`$CARGO_HOME/registry/`\
 此目录包含 来源网站 索引的缓存下载和任何下载的依赖项。

`$CARGO_HOME/git/`\
 此目录包含 git 依赖项的缓存下载。

请注意`$CARGO_HOME`目录的内部结构尚未稳定，可能会发生更改。

[rustup]: https://rust-lang.github.io/rustup/

## EXAMPLES

1. 构建本地包及其所有依赖项：

```bash
cargo build
```

2. 构建一个优化包：

```bash
cargo build --release
```

3. 对跨编译的目标运行测试：

```bash
cargo test --target i686-unknown-linux-gnu
```

4. 创建用于生成可执行文件的新包：

```bash
cargo new foobar
```

5. 在当前目录中，创建包：

```bash
mkdir foo && cd foo
cargo init .
```

6. 了解命令的选项和用法：

```bash
cargo help clean
```

## BUGS

查看<https://github.com/rust-lang/cargo/issues>关于问题。

## SEE ALSO

[rustc(1)](https://doc.rust-lang.org/rustc/index.zh.md), [rustdoc(1)](https://doc.rust-lang.org/rustdoc/index.zh.md)
