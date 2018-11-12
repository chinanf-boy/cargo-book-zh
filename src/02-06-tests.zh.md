
## 测试

货物可以运行您的测试`cargo test`命令.Cargo寻找在两个地方运行的测试:在你的每个地方`src`文件和任何测试`tests/`.测试你的`src`文件应该是单元测试,并在中进行测试`tests/`应该是整合式测试.因此,您需要将包装箱导入到文件中`tests`.

这是一个跑步的例子`cargo test`在我们的项目中,目前没有测试:

```shell
$ cargo test
   Compiling rand v0.1.0 (https://github.com/rust-lang-nursery/rand.git#9f35b8e)
   Compiling hello_world v0.1.0 (file:///path/to/project/hello_world)
     Running target/test/hello_world-9c2b65bbb79eabce

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

如果我们的项目有测试,我们会看到更多的输出与正确的测试数量.

您还可以通过传递过滤器来运行特定测试:

```shell
$ cargo test foo
```

这将运行任何测试`foo`在它的名字.

`cargo test`还运行其他检查.例如,它将编译您包含的任何示例,并且还将测试文档中的示例.请看看[测试指南][testing]在Rust文档中了解更多详细信息.

[testing]: https://doc.rust-lang.org/book/testing.html
