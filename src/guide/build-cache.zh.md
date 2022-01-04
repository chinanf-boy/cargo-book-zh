## Build cache

Cargo 会将构建的产物，输出到 "target" 目录。默认来说，它会在你
[_workspace_][def-workspace]目录下。要想改变，你可以设置
`CARGO_TARGET_DIR` [environment variable], the [`build.target-dir`] config
value, or the `--target-dir` command-line flag。

目录的布局，取决于你是否有使用 `--target`去构建一个指定的平台。如果 `--target`没有指定，那么它会以运行环境的主机架构为蓝本。产物进到 target 目录, 而每个 [profile] 会存储在分开的子目录里:

| 目录                                                     | 描述                                                                           |
| -------------------------------------------------------- | ------------------------------------------------------------------------------ |
| <code style="white-space: nowrap">target/debug/</code>   | Contains output for the `dev` profile.                                         |
| <code style="white-space: nowrap">target/release/</code> | Contains output for the `release` profile (with the `--release` option).       |
| <code style="white-space: nowrap">target/foo/</code>     | Contains build output for the `foo` profile (with the `--profile=foo` option). |

历史原因, `dev` 和 `test` profiles 存储在
`debug` 目录, 而 `release` and `bench` profiles 存储在
`release` 目录。 用户所定义的 profiles 则放在同名目录下。

如果配置了 `--target`, 产物输出的位置也是同名的:

| 目录                                                                    | 示例                                                                           |
| ----------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| <code style="white-space: nowrap">target/&lt;triple&gt;/debug/</code>   | <code style="white-space: nowrap">target/thumbv7em-none-eabihf/debug/</code>   |
| <code style="white-space: nowrap">target/&lt;triple&gt;/release/</code> | <code style="white-space: nowrap">target/thumbv7em-none-eabihf/release/</code> |

> **注意了**: 当 `--target`没有指定的时候, 有一个后果就是， Cargo will
> 会在 build scripts and proc macros，分享你的依赖项。[`RUSTFLAGS`]
> 也会在每次 `rustc` 调用分享。而使用了 `--target` ,
> build scripts 和 proc macros 就会分离式构建 (主机架构
> ), 也不会分享 `RUSTFLAGS`。

在 profile 目录里 (例如：`debug` 或 `release`), 产物会放到下面的目录：

| 目录                                                            | 描述                                                                                             |
| --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| <code style="white-space: nowrap">target/debug/</code>          | Contains the output of the package being built (the [binary executables] and [library targets]). |
| <code style="white-space: nowrap">target/debug/examples/</code> | Contains [example targets].                                                                      |

而一些命令的产物，也放到了各自的位置:

| 目录                                                     | 描述                                                                         |
| -------------------------------------------------------- | ---------------------------------------------------------------------------- |
| <code style="white-space: nowrap">target/doc/</code>     | Contains rustdoc documentation ([`cargo doc`]).                              |
| <code style="white-space: nowrap">target/package/</code> | Contains the output of the [`cargo package`] and [`cargo publish`] commands. |

Cargo 还创建了一些构建过程，用得上的目录。这些，其实就可以说是 Cargo 内部资料了，且有改变的可能，如以下目录:

| 目录                                                               | 描述                                                                      |
| ------------------------------------------------------------------ | ------------------------------------------------------------------------- |
| <code style="white-space: nowrap">target/debug/deps/</code>        | Dependencies and other artifacts.                                         |
| <code style="white-space: nowrap">target/debug/incremental/</code> | `rustc` [incremental output], a cache used to speed up subsequent builds. |
| <code style="white-space: nowrap">target/debug/build/</code>       | Output from [build scripts].                                              |

### Dep-info files

接下来，是"dep info" 文件，它有着 `.d`
后缀。类 Makefile 语法，表明重建一个产物，所需的所有文件依赖。这些，可以让外部构建系统使用，这样他们可以知道 Cargo 是否需要重新执行。默认这些文件是绝对路径。相对路径，就看看
[`build.dep-info-basedir`] 配置。

```Makefile
# Example dep-info file found in target/debug/foo.d
/path/to/myproj/target/debug/foo: /path/to/myproj/src/lib.rs /path/to/myproj/src/main.rs
```

### Shared cache

第三方工具, [sccache], 在不同 workspace 间，分享构建依赖

启用 `sccache`,

- 安装 `cargo install sccache`
- 设置`RUSTC_WRAPPER` 环境变量 to `sccache`。如果使用 bash，将 `export RUSTC_WRAPPER=sccache` 添加到
  `.bashrc`。或者可以在 [Cargo
  configuration][config]，设置 [`build.rustc-wrapper`]。

[`rustflags`]: ../reference/config.md#buildrustflags
[`build.dep-info-basedir`]: ../reference/config.md#builddep-info-basedir
[`build.rustc-wrapper`]: ../reference/config.md#buildrustc-wrapper
[`build.target-dir`]: ../reference/config.md#buildtarget-dir
[`cargo doc`]: ../commands/cargo-doc.md
[`cargo package`]: ../commands/cargo-package.md
[`cargo publish`]: ../commands/cargo-publish.md
[build scripts]: ../reference/build-scripts.md
[config]: ../reference/config.md
[def-workspace]: ../appendix/glossary.md#workspace '"workspace" (glossary entry)'
[environment variable]: ../reference/environment-variables.md
[incremental output]: ../reference/profiles.md#incremental
[sccache]: https://github.com/mozilla/sccache
[profile]: ../reference/profiles.md
[binary executables]: ../reference/cargo-targets.md#binaries
[library targets]: ../reference/cargo-targets.md#library
[example targets]: ../reference/cargo-targets.md#examples
