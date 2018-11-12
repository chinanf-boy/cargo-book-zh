
## 从crates.io添加依赖项

[crates.io]是Rust社区的中央存储库,用作发现和下载包的位置.`cargo`默认配置为使用它来查找请求的包.

依赖托管的图书馆[crates.io],将它添加到您的`Cargo.toml`.

[crates.io]: https://crates.io/

### 添加依赖项

如果你的`Cargo.toml`还没有`[dependencies]`部分,添加,然后列出您要使用的包名称和版本.这个例子增加了一个依赖关系`time`箱:

```toml
[dependencies]
time = "0.1.12"
```

版本字符串是[semver]版本要求.该[指定依赖项](03-01-specifying-dependencies.html)docs提供了有关此处选项的更多信息.

[semver]: https://github.com/steveklabnik/semver#requirements

如果我们还想添加一个依赖`regex`箱子,我们不需要添加`[dependencies]`对于列出的每个箱子.这就是你的整体`Cargo.toml`文件看起来像依赖于`time`和`regex`包装箱:

```toml
[package]
name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
time = "0.1.12"
regex = "0.1.41"
```

重新运行`cargo build`,Cargo将获取新的依赖项及其所有依赖项,将它们全部编译,然后更新`Cargo.lock`:

```shell
$ cargo build
      Updating registry `https://github.com/rust-lang/crates.io-index`
   Downloading memchr v0.1.5
   Downloading libc v0.1.10
   Downloading regex-syntax v0.2.1
   Downloading memchr v0.1.5
   Downloading aho-corasick v0.3.0
   Downloading regex v0.1.41
     Compiling memchr v0.1.5
     Compiling libc v0.1.10
     Compiling regex-syntax v0.2.1
     Compiling memchr v0.1.5
     Compiling aho-corasick v0.3.0
     Compiling regex v0.1.41
     Compiling hello_world v0.1.0 (file:///path/to/project/hello_world)
```

我们的`Cargo.lock`包含有关我们使用的所有这些依赖项的哪个版本的确切信息.

现在,如果`regex`在我们选择之前,我们仍将使用相同的版本进行更新`cargo update`.

你现在可以使用了`regex`图书馆使用`extern crate`在`main.rs`.

```
extern crate regex;

use regex::Regex;

fn main() {
    let re = Regex::new(r"^\d{4}-\d{2}-\d{2}$").unwrap();
    println!("Did our date match? {}", re.is_match("2014-01-01"));
}
```

运行它将显示:

```shell
$ cargo run
   Running `target/hello_world`
Did our date match? true
```
