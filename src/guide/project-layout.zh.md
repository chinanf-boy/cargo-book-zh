## 项目布局

Cargo 使用文件放置惯例，以便轻松进入新的 Cargo 项目:

```text
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

- `Cargo.toml`和`Cargo.lock`存储在项目的根目录中.
- 源代码进入`src`目录
- 默认库文件是`src/lib.rs`
- 默认的可执行文件是`src/main.rs`
- 其他可执行文件，可以放入`src/bin/*.rs`
- 集成测试进入`tests`目录
- 示例进入`examples`目录
- 基准进入`benches`目录

如果一个 binary, example, bench 或是集成测试由多个源文件组成，那么在`src/bin`, `examples`, `benches`, or `tests`的子目录里，放一个带有额外[_modules_][def-module] 的`main.rs`文件。执行名为目录名。

关于 Rust's 模块系统在这本书 [the book][book-modules]。

也可以看看 [配置一个 target][configuring a target] 关于手动配置 targets 的方法。
还可以看看 [自动发现 Target][Target auto-discovery] ，关于操纵 Cargo 自动推断 target 名称。

[book-modules]: ../../book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[configuring a target]: ../reference/cargo-targets.zh.md#configuring-a-target
[def-package]: ../appendix/glossary.zh.md#package '"package" (glossary entry)'
[def-module]: ../appendix/glossary.zh.md#module '"module" (glossary entry)'
[target auto-discovery]: ../reference/cargo-targets.zh.md#target-auto-discovery
